---
layout: post
title: Extended attributes, ACLs, and Time Machine
description: "Understanding Mac OS X extended attributes and ACLs"
tags: [os-x]
---

Copying from a Time Machine backup...
-------------------------------------

So...through some git experimentation trying to erase the history of a file, I
committed and pushed to a local repository, which turned out not to be my
intention. Instead of figuring out how to fix it the git way, I thought I'd
just copy yesterday's Time Machine backup of the repository. Instead of using 
Time Machine app, I mounted the backup disk, and `cp -R`'d the directory. A
`ls-l` told me permissions and user:group ownership were correct...though I did
notice an `@` tacked onto the end `drwxr-xr-x@`...

Insufficient Permission?
------------------------

Returning to the project a few days later, git pushing a new tag to the local
repository didn't work, with a permission error:

>> insufficient permission for adding an object to repository database etc.

WTF, permissions and ownership are **right**. Google, Google, git, permissions,
etc. and no luck. Then, brilliant insight - don't ignore the information
presented to me. What exactly does the `@` on the end of `drwxr-xr-x@` mean
anyway? 

OS X Extended Attributes
------------------------

Copying directly from the Time Machine backup also copied extended attributes
and ACLs, which I definitely didn't want. The command

    xattr my_repository.git

told me things like `com.apple.metadata:_kTimeMachineOldestSnapshot` and 
`com.apple.metadata:_kTimeMachineNewestSnapshot` were attached to the directory 
and it's files. To remove:

    sudo xattr -d -r 'com.apple.metadata:_kTimeMachineOldestSnapshot' my_repository.git
    sudo xattr -d -r 'com.apple.metadata:_kTimeMachineNewestSnapshot' my_repository.git

To learn more about xattr:
    
    xattr -h

OS X ACLs
---------

Now `ls -l` gave me `drwxr-xr-x+`, with the `+` meaning the directory has ACL
(access control lists) attached to it. Needless to say, I still couldn't push
to the repository, even though the standard user:group permissions said I
could. To remove the ACL restrictions:

    sudo chmod -R -a# 0 my_repository

Finally, `ls -l` returns plain old `drwxr-xr-x`, and I could push to my
repository. What a wonderful waste of time!
    
