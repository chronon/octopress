---
layout: post
title: Building PHP with MacPorts Apache2 and MySQL
description: "Mac OS X web development server: building PHP with MacPorts Apache2 and MySQL"
tags: [os-x, dev] 
---

In the past I've always installed Apache2 via MacPorts, MySQL from source or binary, and built PHP from source. While putting this all together on Snow Leopard, I decided to install MySQL from MacPorts as well:

    sudo port -v install apache2 mysql5-server

After everything was built and installed, I then installed the dependencies I needed for PHP 5.3 from MacPorts (openssl, mcrypt, libxml, etc.). My full configure command is:

    ./configure \
    --prefix=/usr/local \
    --with-apxs2=/opt/local/apache2/bin/apxs \
    --with-config-file-path=/usr/local/etc/php5 \
    --disable-cgi \
    --enable-bcmath \
    --with-gd \
    --with-curl \
    --enable-exif \
    --enable-calendar \
    --enable-mbstring \
    --with-mysql=/usr/local/mysql \
    --with-mysqli \
    --with-pdo-mysql=/usr/local/mysql \
    --with-zlib-dir=/opt/local \
    --with-openssl=/opt/local \
    --with-mcrypt=/opt/local \
    --with-mhash=/opt/local \
    --with-freetype-dir=/opt/local \
    --with-jpeg-dir=/opt/local \
    --with-png-dir=/opt/local \
    --with-libxml-dir=/opt/local \
    --with-xsl=/opt/local \
    --with-gettext=/opt/local \
    --with-bz2 \
    --with-iconv=/opt/local

Notice --with-mysql=/usr/local/mysql? No matter how many paths I tried I could not get PHP to con- figure with MySQL. Apparently the MacPort changes the layout for some reason. My solution (inspired by a [2007 discussion on the MacPorts mailing list][1]) involved making a few symlinks:

    sudo mkdir -p /usr/local/mysql
    cd /usr/local/mysql
    sudo ln -s /opt/local/include/mysql5 include
    sudo ln -s /opt/local/lib/mysql5 lib
    sudo mkdir bin
    cd bin
    sudo ln -s /opt/local/bin/mysql_config5 mysql_config

Once the symlinks were in place, PHP configured without error.

 [1]: http://lists.macosforge.org/pipermail/macports-users/2007-December/007567.html
