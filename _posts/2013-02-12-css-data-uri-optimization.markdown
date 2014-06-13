---
layout: post
status: publish
published: true
title: CSS data-uri optimization
author:
  display_name: Rob Koberg
  login: rkoberg
  email: rob.koberg@gmail.com
  url: ''
author_login: rkoberg
author_email: rob.koberg@gmail.com
wordpress_id: 39
wordpress_url: http://koberg.com/?p=39
date: '2013-02-12 01:03:22 -0800'
date_gmt: '2013-02-12 01:03:22 -0800'
tags:
- CSS
- Sass
- data-uri
comments: []
---
<p>One thing you can do to improve your web site's performance and cut http requests is to use data-uris in your CSS. If you can get the data-uri under 32k, it can safely be used in CSS with support for browsers down to IE8.</p>
<p>If you just have a few to deal with, <a href="http://cobbledco.de/2011/06/base64-data-uri-encoder-for-os-x/" title="Base64 Data URI encoder for OS X">here is a handy tool</a> that will offer a right+click context menu item to put the data-uri in your clipboard.</p>
<blockquote><p>Generate a base64 encoded Data URI straight from the Finder.<br />
An Automator workflow which adds an item to the ‘Services’ contextual menu. It generates the Data URI for a file and copies it to the pasteboard.</p></blockquote>
<p>Just right+click on an image in your finder and then paste it in a CSS url().</p>
<p>For a more maintainable approach (using <a href="http://koberg.com/2013/02/12/twitter-bootstrap-sass-and-compass-for-sanity/" title="Twitter Bootstrap, Sass, and Compass for sanity">Sass, Compass, and Bootstrap</a>), I like to keep two separate SCSS include files that just define variables for each image URL. One is for a URL to the real image, and one is for a generated data-uri. I do this because images will change and I can run a quick java command line app to quickly regenerate the data-uri file. Check out <a href="http://www.nczonline.net/blog/2009/11/03/automatic-data-uri-embedding-in-css-files/">Automatic data URI embedding in CSS files</a> Nicholas C. Zakas.</p>
