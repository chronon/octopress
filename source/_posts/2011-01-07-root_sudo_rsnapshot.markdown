---
layout: post
title: Root, sudo, and rsnapshot
description: "Instructions for configuring rsync or rsnapshot to copy restricted files between servers"
tags: [sysadmin] 
---

Introduction
------------

### The Mission: ###

To rsync and/or rsnapshot both normal and protected/restricted files from one
server to another over ssh **without** enabling remote root access to either
server while maintaining original file attributes and permissions. Whew.  

### Cast of Characters ###

* **uno:** the server which will store the backups, run rsync or
  [rsnapshot][rsnapshot].
* **zero:** the server to be backed up, with root readable files (/etc for
  example).

### Prerequisites ###

* **root** on both **uno** and **zero**, hopefully via sudo and not by remote
  root ssh access!

Configuration
-------------

The command examples here are specific to Debian (and therefore Ubuntu), so
adjust to your distribution accordingly (though little if any adjustment should
be necessary). 

### Server: uno (backup server) ###

Create new backup user, lets call it `rbackup`:
    
    $ sudo adduser rbackup

Login as rbackup, create an ssh key pair:
    
    $ su rbackup
    $ ssh-keygen -t rsa

If you are running sshd on a non-standard port (like I am), as user rbackup
create a file named `config` in ~/.ssh. In this file, enter the host and port:
    
    Host zero
        Port 12345

While still user rbackup, copy the public key `id_rsa.pub` somewhere publicly
accessible.

As your regular user (which already has ssh access to **zero**), send rbackup's
public key from **uno** to **zero**:
    
    $ scp /home/rbackup/id_rsa.pub zero:~/

Become root, `sudo -i`. If it doesn't exists, make root's ssh directory, set
the correct permissions, create a ssh config file:

    # mkdir .ssh
    # chmod 700 .ssh
    # cd .ssh
    # vim config

In root's .ssh/config file:

    Host zero-rsync
        Port 12345
        Hostname zero
        User rbackup
        IdentityFile /home/rbackup/.ssh/id_rsa

Save the file, set permissions:

    # chmod 600 config

### Server: zero (server to be backed up) ###

Create new backup user, lets call it `rbackup`:
    
    $ sudo adduser rbackup

Make **uno's** public key `id_rsa.pub` available to user rbackup.

Login as rbackup, create a .ssh directory, set permissions, create an
`authorized_keys` file:
    
    $ su rbackup
    $ cd
    $ mkdir .ssh
    $ chmod 700 .ssh
    $ cd .ssh
    $ cat /home/regularuser/id_rsa.pub > authorized_keys
    $ chmod 600 authorized_keys

Now we want to limit the use of this authorized key by allowing connections
only from **uno**, and allowing one command only. Edit the key, and add something
like this to the beginning of the key: 

    from="192.168.100.123",command="/home/rbackup/validate-rsync.sh" ssh-rsa AX 
    ...remainder of key...rbackup@uno

While still user rbackup, create a script named `validate_rsync.sh` in your
home directory:

    $ cd
    $ vim validate_rsync.sh

Contents of `validate_rsync.sh`:

{% highlight sh %}
#!/bin/sh
case "$SSH_ORIGINAL_COMMAND" in
  *\&*)
    echo "Connection closed"
    ;;
  *\;*)
    echo "Connection closed"
    ;;
    rsync*)
    $SSH_ORIGINAL_COMMAND
    ;;
  *true*)
    echo $SSH_ORIGINAL_COMMAND
    ;;
  *)
    echo "Connection closed."
    ;;
esac
{% endhighlight %}

As your regular user (which can sudo): 

    $ sudo visudo

Add this line to the bottom:

    rbackup    ALL=NOPASSWD:/usr/bin/rsync

If you have the `AllowUsers` directive set for sshd in `/etc/ssh/sshd_config`,
make sure to add the user `rbackup` to the list, and restart sshd.

As root or with sudo, create a simple rsync wrapper, named `rsync_wrapper.sh` at /usr/local/bin,
containing:

{% highlight sh %}
#!/bin/sh
/usr/bin/sudo /usr/bin/rsync "$@";
{% endhighlight %}

Testing
-------

### Test 1 ###

Become user rbackup on **uno**, attempt ssh to **zero**:

    $ ssh zero

Expected response:

    Connection closed.
    Connection to zero closed.

The "Connection closed." with the period at the end tells us the
`validate_rsync.sh` worked as expected (echoing the last Connection closed).

### Test 2 ###

Become root on **uno**, attempt to ssh to zero-rsync (the alias set in root's
.ssh/config):

    # ssh zero-rsync

Expected response:

    Connection closed.
    Connection to zero closed.

### Test 3 ###

Become root on **uno**, attempt to rsync something on **zero** that is restricted:

    # rsync -ae ssh --rsync-path='rsync_wrapper.sh' zero-rsync:/etc
    /home/regularuser/tmp/

Expected response: you should have a copy of **zero's** /etc directory in
regular users tmp directory (or wherever). Important things to note in the above
command - the --rsync-path switch and using zero-rsync as the host instead of
just **zero**.  
    
Using rsnapshot
---------------

If the above tests all work, setting up [rsnapshot][rsnapshot] is easy. Check
any other guide for general setup info, the relevant stuff for us to use in our
rsnapshot.conf is:

    rsync_long_args --rsync-path=rsync_wrapper.sh --delete --numeric-ids --relative --delete-excluded
    backup rbackup@zero-rsync:/etc zero/

Remember the rsnapshot.conf file needs tabs. The `rsync_long_args` setting is
rsnapshot's default rsync arguments, with our `--rsync-path=rsync_wrapper.sh`
added. The `backup` command has our backup user (rbackup) and the host alias
set in root@uno's .ssh/config.

References
----------

This was pieced together from various notes, blogs, and articles.
Some script source and extra-helpful information was found at:

* [alien.slackbook.org/dokuwiki/doku.php?id=linux:rsnapshot][1]
* [troy.jdmz.net/rsnapshot/][2]
* [www.cyberciti.biz/faq/linux-rsnapshot-backup-howto/][3]
* [serverfault.com/questions/136549/rsync-over-ssh-with-root-access-on-both-sides][4]

[1]: http://alien.slackbook.org/dokuwiki/doku.php?id=linux:rsnapshot
[2]: http://troy.jdmz.net/rsnapshot/
[3]: http://www.cyberciti.biz/faq/linux-rsnapshot-backup-howto/
[4]: http://serverfault.com/questions/136549/rsync-over-ssh-with-root-access-on-both-sides
[rsnapshot]: http://rsnapshot.org/
