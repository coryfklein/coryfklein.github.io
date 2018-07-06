---
layout: post
title:  "Recover From Git Force Push"
date:   2015-09-28 15:58:26
comments: true
categories: git
---

Have you done a `git push --force` and overwritten some work that you don't have backed up anywhere else? I just experienced this today but I was able to recover my work.

Fortunately, `git` is pretty good about not throwing away anything, so it is possible your work still exists on the server.

In my case, I used `git push --force` to erase some work on the server knowing that I still had the work on a branch on my local machine. Unfortunately, this doesn't protect from hardware failure or unintended deletion - which is what happened in my case.

## 1. Find the lost SHA

Is there anywhere that you can reference a SHA pointing to the lost work? Possible places to look might be:

* Email (Git messages)
* Chat
* Build service (Jenkins, etc)

I was saved by Jenkins because a previous build contained a reference the SHA that initiated it.

Once you find the SHA, place it in the URL to your GitHub repository like this:

    https://github.com/coryfklein/myrepo/commit/<SHA_HERE>

And voila! You have a link to your lost work.

You can click on "parent" links to open any other related commits you may have lost as well.

## 2. Create Patch Files

For each commit you can get a patch file by appending `.patch` to the above URL like so:

    https://github.com/coryfklein/myrepo/commit/<SHA_HERE>.patch

Save this file to disk, repeating as necessary for each commit you need to recover.

## 3. Apply the Patch Files

Use `git am` to apply your patch files to your repository.

    git am 001-work.patch 002-work.patch 003-work.patch

## 4. Save Your Work

When you're ready, push your recovered work to master:

    git push

Otherwise, save your work to a branch and push that to the server so it is saved both locally and remotely:

    git co -b my-saved-work
    git push --set-upstream origin my-saved-work
