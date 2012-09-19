---
layout: post
title: PHP, MySQL, and CGIWrap on Mac OS X (pair Networks style)
description: "Compiling PHP, MySQL, and CGIWrap on Mac OS X (pair Networks style)"
date: 2005-01-06
categories: [os-x, dev] 
---

*   **Version: 1.04**
*   **Revised: 06-JAN-2005**
*   **Disclaimer: If you break your computer it's not my fault!**
*   **If you find this useful or find errors, let me know!**

This guide (hopefully) will tell you about how to set up a PHP4/php-cgiwrap/MySQL testing server using Mac OSX (10.3.x) based on pair Networks (<http://pair.com>) FreeBSD server configuration. Of course these notes could be used to set up a testing server for use with another host, just omit pair specific parts. I originally did this using FreeBSD 4.8, then again with Gentoo Linux, and then changed a few things to get it working in OS X. I'll venture to say this setup would be impossible using any version of Windows, due to path configuration and cgiwrap (PHP, Apache, MySQL no problem). Hopefully I have not forgotten any major steps, but if I have let me know.

For this OS X setup I used the included Apache installation after getting mod_ssl running and configured. SSL is not necessary to continue, but my goal was a very close replica of my pair server, and I have/use SSL. To set up SSL follow this link:

*   <http://developer.apple.com/internet/serverside/modssl.html>

**Table of contents:**

*   [Make some directories and set permissions][1]
*   [Install MySQL server][2]
*   [Compile and install PHP Apache module][3]
*   [Make a virtual directory for your user][4]
*   [Check to see if it's working so far][5]
*   [Compile and install your own cgi version of PHP][6]
*   [Compile and install cgiwrap][7]
*   [Modify /private/etc/http/http.conf to work with cgiwrap][8]
*   [Test PHP cgi using cgiwrap][9]
*   [Conclusion][10]
*   [About this page][11] 

<a name="id2244661" id="id2244661"></a>

Make some directories and set permissions
-----------------------------------------

**I strongly suggest using the same username you use on pair on your host!** Setting up my testing server using the same path structure as my host (pair Networks) has saved me a lot of time. If you are not using pair, just change the /usr/www/users/YOURUSER path to whatever is appropriate. The cd command with no arguments simply takes you to your home directory.

``` sh
$ sudo mkdir /Library/WebServer/cgi-sys
$ sudo mkdir -p /usr/www/users/YOURUSER
$ sudo chown YOURUSER /usr/www/users/YOURUSER
$ sudo chgrp YOURUSER /usr/www/users/YOURUSER
$ chmod 705 /usr/www/users/YOURUSER
$ cd
$ mkdir source
$ mkdir phpini
$ ln -s /usr/www/users/YOURUSER public_html
$ cd public_html
$ mkdir cgi-bin
```
<a name="id2244688" id="id2244688"></a>

Install MySQL server
--------------------

*   <http://dev.mysql.com/downloads/mysql/4.0.html>
*   <http://www.entropy.ch/software/MacOSx/mysql/>
*   <http://developer.apple.com/internet/opensource/osdb.html>

I used "Installer package (Mac OS X v10.3) Standard 4.0.21 (11.7M)" and just followed the instructions in the readme. Unfortunately I can't remember the exact steps...but it was easy and straightforward. 

<a name="id2244715" id="id2244715"></a>

Compile and install PHP Apache module
-------------------------------------

I'm using PHP4 (4.3.10) and pair now has 4.3.10 installed as well. The [osxpair.tar.bz2][12] file contains all of the configure scripts for PHP and cgiwrap, along with the config.* files for cgiwrap to compile.

* <http://www.php.net/downloads.php>
* <http://darwinports.opendarwin.org/>
* <http://developer.apple.com/internet/opensource/php.html>

Extract the configure script archive:

``` sh
$ mv (wherever you downloaded to)/osxpair.tar.bz2 ~/source/ 
$ mv (wherever you downloaded to)/php-4.3.10.tar.bz2 ~/source/
$ cd ~/source/
$ tar jxvf osxpair.tar.bz2
$ chmod 755 conf-*
```

Now it is time for some decisions. Use your favorite editor (Vim!) and open conf-mod-osx. These configure options are based on what pair uses, with the following differences because I don't use them or they are not enabled by default anymore. Modify to your own taste:

pair has these, this configuration does not:

```
--with-iodbc
--with-db
--without-gdbm
--with-ndbm
--without-db2
--without-dbm
--without-db3
--enable-dba
--enable-mstring
--with-pdflib*
--with-recode*
```

This configuration has, pair does not. Remove if you want!

```
--enable-exif
```
 
If you want your php.ini somewhere like /etc/php (I do)

```
$ sudo mkdir /etc/php
```
 
I don't use Fink for package management but do use Darwinports...so because pdflib and recode are not available in Darwinports I did not install or enable them. They could be installed by Fink or manually if needed.

Anyway, anything in the conf-mod-osx file with "/opt/local" means I installed it using Darwinports (for example gettext, mcrypt, gd, sablotron, etc.). Darwinports is easy to use, so check the site and install whatever you need/want first. If you don't want a feature just remove the line from the file.

``` sh
$ tar jxvf php-4.3.10.tar.bz2
$ cd php-4.3.10/
$ ln -s ../conf-mod-osx conf-mod
$ ./conf-mod
```
    
If the configure script does not complete you have probably not installed something listed in conf-mod-osx. Check the error message and see what file it could not find. Either install the package needed or remove the option...and run ./conf-mod again until it is successful. 

``` sh
$ make
$ sudo make install
```
    
I think the install script takes care of your Apache /private/etc/httpd/httpd.conf file, but check to make sure you have these lines:

``` apache
LoadModule php4_module    libexec/httpd/libphp4.so
AddModule mod_php4.c
AddType application/x-httpd-php .php
```
    
Copy your php.ini:

``` sh
$ sudo cp php.ini-recommended /etc/php/php.ini
```

Edit php.ini as needed. These are the differences between what pair uses and what is in the php.ini-recommended. First value is what pair has, second one in (parantheses) is what php.ini-recommended has.

``` ini
allow_url_fopen off (on)
error_reporting 39 (47)
max_input_time -1 (60)
output_buffering no value (4096)
register_globals on (off)
upload_max_filesize 4194304 (2M)
upload_tmp_dir /usr/tmp (no value)
y2k_compliance off (on)
mysql.allow_persistent off (on)
mysql.max_persistent 1 (Unlimited)
session.bug_compat_42 on (off)
session.gc_divisor 100 (1000)
```

<a name="id2244860" id="id2244860"></a>

Make a virtual directory for your user
--------------------------------------

Use your editor of choice...

``` sh
$ sudo vim /private/etc/httpd/httpd.conf
```

Add this part:

``` apache
<VirtualHost 127.0.0.1:80>
    DocumentRoot "/usr/www/users/YOURUSER"
    ServerName 127.0.0.1
    ServerAdmin you@localhost
    SSLEngine off
</VirtualHost>
```

Save, and then run:

``` sh
$ sudo apachectl graceful
```
     
<a name="id2244893" id="id2244893"></a>

Check to see if it's working so far
-----------------------------------

``` sh
cd
cd public_html
```

Make a file named phpinfo.php with the following:

``` php
<?php phpinfo() ?>
```
    
Open your browser, go to http://localhost/phpinfo.php and you should see the PHP info page. Notice the 4th item "Server API" says "Apache". If you get an error you did something wrong or I wrote something wrong! 

<a name="id2244919" id="id2244919"></a>

Compile and install your own cgi version of PHP
-----------------------------------------------

I try to make my cgi PHP the same as the module, which is (nearly) the same as pair's. This means I can use it when needed, and use the faster module when not without worrying about features matching. If you changed the conf-mod-osx file, go ahead and change the conf-cgi-osx to match.

**IMPORTANT!** For php-cgiwrap to work you MUST keep this line in conf-cgi-osx:

```
--enable-force-cgi-redirect=yes
```

**IMPORTANT!** Edit conf-cgi-osx and set YOURUSER on the second line!

``` sh
$ cd ~/source/php-4.3.10/
$ make distclean
$ rm config.cache
$ ln -s ../conf-cgi-osx conf-cgi
$ ./conf-cgi
```

If the configure script does not complete you have probably not installed something listed in conf-cgi-osx. Check the error message and see what file it could not find. Either install the package needed or remove the option...and run ./conf-cgi again until it is successful. 

``` sh
$ make
```
    
**DO NOT RUN make install**

``` sh
$ cd sapi/cgi
$ cp php ~/public_html/cgi-bin/php4310.cgi
$ cd ~/public_html/cgi-bin/
$ strip php4310.cgi
$ ln -s php4310.cgi php4
$ cd ~/source/php-4.3.10/
$ cp php.ini-recommended ~/phpini/php.ini
$ cd ~/phpini/
```
    
Edit php.ini as needed. These are the differences between what pair uses and what is in the php.ini-recommended. First value is what pair has, second one in (parantheses) is what php.ini-recommended has.

``` ini
allow_url_fopen off (on)
error_reporting 39 (47)
max_input_time -1 (60)
output_buffering no value (4096)
register_argc_argv off (on)
register_globals on (off)
upload_max_filesize 4194304 (2M)
upload_tmp_dir /usr/tmp (no value)
y2k_compliance off (on)
mysql.allow_persistent off (on)
mysql.max_persistent 1 (Unlimited)
session.bug_compat_42 on (off)
session.gc_divisor 100 (1000)
```
     
<a name="id2244995" id="id2244995"></a>

Compile and install cgiwrap
---------------------------

The latest version is cgiwrap-3.9.tar.gz, so that is what I am using.

*   <http://sourceforge.net/projects/cgiwrap>

The config.guess and config.sub files included with the cgiwrap distribution don't know OS X. The ones I included in the osxpair.tar.bz2 file are copies of the ones in the PHP distribution, and do know OS X, so you need to copy and overwrite the existing ones in cgiwrap-3.9 directory.

``` sh
$ mv (wherever you downloaded to)/cgiwrap-3.9.tar.gz ~/source/
$ cd ~/source/
$ tar zxvf cgiwrap-3.9.tar.gz
$ cd cgiwrap-3.9
$ cp ../config.guess .
$ cp ../config.sub .
$ ln -s ../conf-cgiwrap-osx conf-cgiwrap
$ ./conf-cgiwrap
```
    
If it completes without error:

``` sh
$ make
$ make install (this gives me an error...no problem)
$ sudo cp cgiwrap /Library/WebServer/cgi-sys/
$ cd /Library/WebServer/cgi-sys/
$ sudo strip cgiwrap
$ sudo chown root cgiwrap
$ sudo chmod 4755 cgiwrap
```
     
<a name="id2245037" id="id2245037"></a>

Modify /private/etc/http/http.conf to work with cgiwrap
-------------------------------------------------------

Put this within your VirtualHost container: 

``` apache
<VirtualHost 127.0.0.1:80>

    # other stuff you added before...

    <Directory /usr/www/users/YOURUSER>
        Options +Indexes +ExecCGI
        AllowOverride All
        Allow from all
    </Directory>

</VirtualHost>
```

If you are using SSL, put the same thing within your `<VirtualHost 127.0.0.1:443>` container.

Find this line:

``` apache
ScriptAlias /cgi-bin/ "/Library/WebServer/CGI-Executables/"
```

Add this line below it:

``` apache
ScriptAlias /cgi-sys/ "/Library/WebServer/cgi-sys/"
```

Save httpd.conf, restart Apache:

``` sh
$ sudo apachectl graceful
```
     
<a name="id2245087" id="id2245087"></a>

Test PHP cgi using cgiwrap
--------------------------

``` sh
$ cd ~/public_html/
```
    
Create a file named .htaccess with the following:

``` apache
AddHandler php-cgiwrap .php
Action php-cgiwrap /cgi-sys/cgiwrap/YOURUSER/cgi-bin/php4
```
    
Open your browser, go to http://localhost/phpinfo.php and you should see the PHP info page. Notice the 4th item "Server API" says "CGI". If it says "Apache" you messed up the .htaccess file. If you get an error, ummm, it was something else. 

<a name="id2245114" id="id2245114"></a>

Conclusion
----------

If everything is working, you should have a great test system set up! Any directory you need to use the cgiwrapped PHP cgi just put the .htaccess file in it. For pair, change the .htaccess lines to:

``` apache
Action application/x-pair-sphpx /cgi-sys/php-cgiwrap/YOURUSER/php4
AddType application/x-pair-sphpx .php
```

<a name="id2245132" id="id2245132"></a>

About this page
---------------

This page was (originally) written in XML, transformed into XHTML using [libxslt][14] and made pretty with CSS. All code edited using [Vim][15] running on [Gentoo Linux][16] and Mac OSX.

In 2011 this outdated guide was semi-successfully converted from HTML to Markdown
using [Markdownify][17]. 

 [1]: #id2244661
 [2]: #id2244688
 [3]: #id2244715
 [4]: #id2244860
 [5]: #id2244893
 [6]: #id2244919
 [7]: #id2244995
 [8]: #id2245037
 [9]: #id2245087
 [10]: #id2245114
 [11]: #id2245132
 [12]: /files/osxpair.tar.bz2
 [14]: http://www.xmlsoft.org/
 [15]: http://www.vim.org/
 [16]: http://www.gentoo.org/
 [17]: http://milianw.de/projects/markdownify/
