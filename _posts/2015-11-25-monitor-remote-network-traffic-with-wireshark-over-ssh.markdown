---
layout: post
title:  "Monitor Remote Network Traffic With Wireshark Over SSH"
date:   2015-11-25 16:21:32
comments: true
categories:
- networking
- wireshark
---

## SSH Tunelling

Wireshark is an extremely useful tool in monitoring and debugging network traffic. However, sometimes you want to view the network traffic on a remote device where using Wireshark's windowed UI is not feasible. Wireshark does have a command-line interface for doing this, but I find the UI much more approachable.

Fortunately, there is an easy way to route the network traffic over SSH to an instance of the Wireshark GUI you are running locally.

    wireshark -k -i <(ssh -l <USER> <HOST> "dumpcap -i <INTERFACE> -P -w - -f 'not tcp port 22'")

This will open a local instance of Wireshark and show all traffic on the remote interface, filtering out any traffic related to you ssh connection over port 22.

You can then use the display filter in the Wireshark UI to filter down the packets to only the ones you are interested in.

When you are done, make sure to save your data before using CTRL-C to close your connection, as that will also close your Wireshark UI.

### Narrow Your Filter

To preserve the bandwidth over your SSH connection, you may want to amend the filter. For example, if you are not interested in http packets, you can modify the above command like so:

    wireshark -k -i <(ssh -l <USER> <HOST> "dumpcap -i <INTERFACE> -P -w - -f 'not tcp port 22 and not http'")

### Filter, Transfer, View

Sometimes the above is not feasible because no matter how much you narrow down your filters you are still getting too much data. When streaming Wireshark captures you are only allowed to apply packet filters with libpcap filter syntax. This means that display filters that rely on deep packet inspection such as "mysql.query contains 'CUSTOMER'" cannot be applied.

In this case, you can instead save your network data to a file:

    dumpcap -i <INTERFACE> -P -w - -f 'not tcp port 22' -w output.pcapng

Then, you can use `tshark` to apply a display filter:

    tshark -Y "mysql.query contains 'CUSTOMER'" -r output.pcapng -w filtered.pcapng

Then, just `scp` over `filtered.pcapng` and open it in Wireshark!

Now, I know what you're thinking, why not just pipe the output of `dumpcap` to `tshark` to let it filter the data in real time, and then pipe the data over the SSH connection like before?

Actually, in previous versions of Wireshark this was actually possible. But due to other bugs that were associated with this behaviour, `tshark` was modified to disallow the use of display filters when piping packet data through it. :(

So for now, there is no way to apply Wireshark display filters to data before piping it over an SSH connection, you must first save the network data to a file, apply the display filters to that file, and then view the resulting data.
