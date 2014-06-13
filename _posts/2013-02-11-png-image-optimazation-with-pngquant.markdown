---
layout: post
status: publish
published: true
title: PNG Image optimization with pngquant
author:
  display_name: Rob Koberg
  login: rkoberg
  email: rob.koberg@gmail.com
  url: ''
author_login: rkoberg
author_email: rob.koberg@gmail.com
wordpress_id: 24
wordpress_url: http://koberg.com/?p=24
date: '2013-02-11 23:53:07 -0800'
date_gmt: '2013-02-11 23:53:07 -0800'
tags:
- pngquant
- image optimization
- command line
- bash scripting
comments: []
---
<p>Use pngquant: <a href="http://pngquant.org/" title="pngquant -  a command-line utility for converting 24/32-bit PNG images to paletted (8-bit) PNGs">http://pngquant.org/</a></p>
<blockquote><p>pngquant is a command-line utility for converting 24/32-bit PNG images to paletted (8-bit) PNGs.</p>
<p>The conversion reduces file sizes a great deal (often as much as 70%) and preserves full alpha transparency. Generated images are compatible with all modern web browsers, and have better fallback in IE6 than 24-bit PNGs.</p></blockquote>
<p>On OS X you can install it easily with <a href="http://mxcl.github.com/homebrew/" title="The missing package manager for OS X">homebrew</a>:<br />
<code>$ brew install pngquant</code></p>
<p>Now you can optimize the hell out of your PNGs. Let's say you don't have thousands of images to optimize. You can do:<br />
<code>$ cd ~/to/some/root/dir</code><br />
<code>$ find . -name '*.png' -exec pngquant -ext .png -force 256 {} \;</code><br />
This searches recursively through the current directory and greatly reduces the file size of each PNG it finds. Using the -force flag lets you overwrite the existing file.</p>
<p>Let's say you have several thousand PNGs to optimize. Using the command above froze my poor little imac. You might also want to do other things with the image (like convert it to an SWF so it can be used in an online PDF flash reader...). If we loop through the files to perform the optimization, my poor little iMac can handle it. Here is a simple bash script that basically does the same thing as the one liner above:</p>
<pre>
#!/bin/bash

for PNG in $(find . -name '*.png'); do
  
  pngquant -ext .png -force 256 ${PNG}
  echo "Optimized PNG: ${PNG}"

done
</pre>
