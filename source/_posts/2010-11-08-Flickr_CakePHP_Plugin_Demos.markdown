---
layout: post
title: Flickr CakePHP Plugin Demos
description: "Demonstrations of the CakePHP Flickr plugin"
date: 2010-11-08
categories: [dev, cakephp] 
---

This is a small collection of (mostly) [jQuery][1] based ways to use [Flickr][2] hosted photos in your own [CakePHP][3] application. The demos here use my [CakePHP Flickr plugin][10], which consists of two files, a component and a helper.

The [source code][5] for these demos is freely available, so it should be very easy to add these (or other) types of photo galleries to your own app.

Available demos:
----------------

* [Raw](http://flickrdemo.chronon.us/flickr_demos/demos/raw/)
* [Colorbox](http://flickrdemo.chronon.us/flickr_demos/demos/colorbox/)
* [Galleriffic](http://flickrdemo.chronon.us/flickr_demos/demos/galleriffic/)
* [Photostack](http://flickrdemo.chronon.us/flickr_demos/demos/photostack/)

When viewing the controller code for each demo, the $params array is merged with some defaults I've set in /app/bootstrap.php. Setting defaults for this Flickr plugin is totally optional, but makes for less code in your controller. The defaults I've set for all of these demos is:

``` php
<?php
'Flickr.defaults', array(
    'api_key' => '111122223333aaaabbbbccccdddd',
    'user_id' => '1234567@N66',
    'method' => 'flickr.photos.search',
    'format' => 'php_serial',
    'extras' => 'description, date_taken'
)
```

All defaults can be overridden/replaced in your controller. See the docs for my [Flickr plugin][10] and the [Flickr API][6] for additional information.
For more information on how to use and customize any of the demo galleries you see here, check the developer websites below (and thank them for their excellent work!):

*   [Colorbox][7]
*   [Galleriffic][8]
*   [Photostack][9]

 [1]: http://jquery.com/
 [2]: http://flickr.com/
 [3]: http://cakephp.org/
 [4]: http://github.com/chronon/flickr
 [5]: http://github.com/chronon/flickr_demos
 [6]: http://www.flickr.com/services/api/
 [7]: http://colorpowered.com/colorbox/
 [8]: http://www.twospy.com/galleriffic/
 [9]: http://tympanus.net/codrops/2010/06/27/beautiful-photo-stack-gallery-with-jquery-and-css3/
 [10]: http://technokracy.net/2010/11/07/Flickr_CakePHP_Plugin
