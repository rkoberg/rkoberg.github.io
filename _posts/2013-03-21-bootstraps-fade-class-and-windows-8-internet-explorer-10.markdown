---
layout: post
status: publish
published: true
title: Bootstrap's fade class and Windows 8, Internet Explorer 10
author:
  display_name: Rob Koberg
  login: rkoberg
  email: rob.koberg@gmail.com
  url: ''
author_login: rkoberg
author_email: rob.koberg@gmail.com
wordpress_id: 59
wordpress_url: http://www.koberg.com/?p=59
date: '2013-03-21 00:07:06 -0700'
date_gmt: '2013-03-21 00:07:06 -0700'
tags: []
comments: []
---
<p>If content is not show because you are using fade transitions:</p>
<p><code>jQuery(document).ready(function($) {</p>
<p>  if ( $.browser.msie && 10 >= $.browser.version ) {<br />
    $('.fade').removeClass('fade');<br />
  }</code></p>
<p>before anything else...</p>
