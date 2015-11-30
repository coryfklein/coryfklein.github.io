---
layout: post
title:  "Deep Introspection of MySQL Network Connection Packets With Wireshark"
date:   2015-11-30 13:43:23
comments: true
categories:
- networking
- wireshark
- mysql
---

## Introduction

At Domo we recently needed to inspect the network connection between a service and MySQL on a low-level where individual TCP packets could be inspected. We found that when our database went down the connection on the service side would remain for up to 2 hours and wanted to gather more information about why so we could create a targeted fix that we knew would address root cause.

Wireshark is a powerful tool for doing just this, but it can be intimidating to use due to the large amount of information it displays. With Wireshark the key is learning how to filter down the data to only see what you are interested in.

At first you may think this involves composing long and complicated display filters, but this is likely an indication that you're going down the wrong path. In this post I wanted to share a couple tips and tricks we used to create some simple Wireshark display filters that showed us just what we needed and nothing more.

## The Display Filter

Ultimately the display filter we wanted looks like this:

    tcp.port == XXXXX

Where `XXXXX` was the port that our service used to connect to MySQL. This one display filter effectively removes all other traffic that is not directly associated with the connection to MySQL.

## Identify The Connection Port

We used two different methods of identifying `XXXXX`

### MySQL Deep Introspection

The first was to locate the packet containing the initial SQL query we were interested in and inspect which port it was sent on. Wireshark has [pretty advanced MySQL integration](https://www.wireshark.org/docs/dfref/m/mysql.html) and actually allows you to filter packets by the MySQL query sent within!

Here is how you search for packets containing the word "SOMETHING" as part of the MySQL query.

    mysql.query contains "SOMETHING"

This immediately filtered down all the packets to the single TCP packet initiating the MySQL query. We could then inspect the origination port and plug that into the `tcp.port == XXXXX` query from above.

Unfortunately, Wireshark is not always capable of deep introspection of `mysql.query`. In some cases, we found that our query strings got all garbled up, and this display filter didn't work. Then we moved on to the second approach.

### Check Open MySQL Connections

Since database connections are expensive to establish, all of our services here at Domo maintain a connection pool that they can borrow from to run queries when needed. This means that the TCP connections are left open perpetually and can be discovered with command line tool `lsof`.

    $ lsof -p "1428" | grep TCP | grep mysql
    java    1428 vagrant  392u  IPv6           15829583      0t0     TCP vagrant-ubuntu-trusty-64:51960->vagrant-ubuntu-trusty-64:mysql (ESTABLISHED)
    java    1428 vagrant  393u  IPv6           15812762      0t0     TCP vagrant-ubuntu-trusty-64:51834->vagrant-ubuntu-trusty-64:mysql (ESTABLISHED)
    java    1428 vagrant  394u  IPv6           15820427      0t0     TCP vagrant-ubuntu-trusty-64:51892->vagrant-ubuntu-trusty-64:mysql (ESTABLISHED)
    java    1428 vagrant  395u  IPv6           15816022      0t0     TCP vagrant-ubuntu-trusty-64:51867->vagrant-ubuntu-trusty-64:mysql (ESTABLISHED)
    java    1428 vagrant  396u  IPv6           15816024      0t0     TCP vagrant-ubuntu-trusty-64:51868->vagrant-ubuntu-trusty-64:mysql (ESTABLISHED)

This command looks for all open file descriptors of node type `TCP` and connection protocol `mysql` that are owned by process with PID 1428 - in this case that process was our service. The local port we are looking for is in the middle of the last column.

A little magic with vim and we have a list of all possible ports that our connection may be found on:

    51960
    51834
    51892
    51867
    51868

You can then join these all together to create one Wireshark display filter that covers all your MySQL connections. In our case this got our network traffic filtered down enough that we could pick out the specific connection we were interested in.

    tcp.port == 51960 or tcp.port == 51834 or tcp.port == 51892 or tcp.port == 51867 or tcp.port == 51868
