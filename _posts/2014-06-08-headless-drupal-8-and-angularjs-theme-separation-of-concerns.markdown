---
layout: post
status: publish
published: true
title: Headless Drupal 8 and AngularJS theme - Separation of Concerns
author:
  display_name: Rob Koberg
  login: rkoberg
  email: rob.koberg@gmail.com
  url: ''
author_login: rkoberg
author_email: rob.koberg@gmail.com
date: '2014-06-08 21:34:12 -0700'
date_gmt: '2014-06-08 21:34:12 -0700'
tags:
- AngularJS
- Drupal 8
comments: []
---
<p>This post will walk you through how to get a Drupal 8 app server going with a AngularJS client side theme. The instructions were written as I went through the process on OS X, but are hopefully clear enough to work on other OSes and web servers (I used apache). And if you are worried about google crawling your site, see <a href="https://developers.google.com/webmasters/ajax-crawling/docs/getting-started">https://developers.google.com/webmasters/ajax-crawling/docs/getting-started</a>.</p>
<p>What I am hoping to get out of this post is to spark more interest in the Drupal community (see <a href="https://groups.drupal.org/headless-drupal">https://groups.drupal.org/headless-drupal</a>) about separating out the theme layer from Drupal so we can bootstrap the core to a lower layer (like DATABASE and/or SESSION) rather than loading all layers. This way we can get the best performance out of Drupal and achieve a clean Separation of Concerns (SoC). <strong>Let's make it easy for designer types to do their thing and maybe we can get pretty sites too.</strong></p>
<p>When done we will have one site running Drupal as the app/api server and another site running the client angular app (which could be hosted on a CDN). The angular app is on github at <a href="https://github.com/rkoberg/d8client/tree/develop">https://github.com/rkoberg/d8client/tree/develop</a> and can be used as a starting point. Hopefully it can build over time to include all of the core TPLs. As modules develop we can create other github(?) projects for those module's TPLs and easily load them into the angular app. Fork it or download it and build your theme.</p>
<p>From <a href="http://en.wikipedia.org/wiki/Separation_of_concerns">http://en.wikipedia.org/wiki/Separation_of_concerns</a></p>
<blockquote><p>Let me try to explain to you, what to my taste is characteristic for all intelligent thinking. It is, that one is willing to study in depth an aspect of one's subject matter in isolation for the sake of its own consistency, all the time knowing that one is occupying oneself only with one of the aspects. We know that a program must be correct and we can study it from that viewpoint only; we also know that it should be efficient and we can study its efficiency on another day, so to speak. In another mood we may ask ourselves whether, and if so: why, the program is desirable. But nothing is gained —on the contrary!— by tackling these various aspects simultaneously. It is what I sometimes have called <b>"the separation of concerns"</b>, which, even if not perfectly possible, is yet the only available technique for effective ordering of one's thoughts, that I know of. This is what I mean by "focusing one's attention upon some aspect": it does not mean ignoring the other aspects, it is just doing justice to the fact that from this aspect's point of view, the other is irrelevant. It is being one- and multiple-track minded simultaneously. - Edsger W. Dijkstra</p></blockquote>
<p>It would not be a bad thing update brew and upgrade at least the node formula:<br />
<code>$ brew update<br />
$ brew doctor</code><br />
If the doctor says you have problems, now is a good time to fix them. Next, at least upgrade node to the latest version:<br />
<code>$ brew upgrade node</code><br />
or to upgrade everything:<br />
<code>$ brew upgrade</code></p>
<p>If you already have a drupal 8 site running with REST Web Services, you skip down to the <a href="#angularjs-client">angular client section</a>.</p>
<h2>Git and install Drupal 8</h2>
<p>Get the latest Drupal 8 and put it into the d8server directory:<br />
<code>$ git clone --branch 8.x http://git.drupal.org/project/drupal.git d8server<br />
$ cd d8server</code><br />
Copy over the sites/default/default.settings.php to sites/default/settings.php and make it writable by apache.<br />
<code>$ cp sites/default/default.settings.php sites/default/settings.php<br />
$ sudo chown _www sites/default/settings.php</code><br />
Create the sites/default/files directory and make it writable by apache.<br />
<code>$ mkdir sites/default/files<br />
$ sudo chown _www sites/default/files</code><br />
Setup a virtual host (mine will be d8server.local) create a database (mine is named d8server).<br />
<code>$ sudo vi /etc/hosts</code><br />
Add a line for your new local site:<br />
<code>127.0.0.1       d8server.local</code><br />
Add a virtual host to apache:<br />
<code>$ sudo vi /etc/apache2/extra/httpd-vhosts.conf</code></p>
<pre>&lt;VirtualHost *:80&gt;
    ServerAdmin rkoberg
    DocumentRoot "/Users/rkoberg/Sites/d8server"
    ServerName d8server.local
    &lt;Directory "/Users/rkoberg/Sites/d8server"&gt;
      Options Indexes FollowSymLinks Includes ExecCGI
      AllowOverride All
      Order deny,allow
      Allow from all
    &lt;/Directory0&gt;
    ErrorLog "/private/var/log/apache2/d8server-error_log"
    CustomLog "/private/var/log/apache2/d8server-access_log" common
&lt;/VirtualHost0&gt;</pre>
<p>Restart apache:<br />
<code>$ sudo apachectl restart</code><br />
Visit the site at <a href="http://d8server.local">http://d8server.local</a>. What? I have a fatal error:<br />
<code style="color:red">Parse error: syntax error, unexpected '[' in /Users/rkoberg/Sites/d8server/core/vendor/guzzlehttp/guzzle/src/functions.php on line 20</code><br />
Oh, right... I was working on a Drupal 7 site which requires php53 and Drupal 8 requires php54 or better. So, I will install php55 using homebrew:<br />
<code>$ brew install php55</code><br />
Now make sure the apache conf is pointing to the correct lib file for php55. Hmmm... homebrew did not set that up for me. I will comment out the php53 (will need it again for D7 dev work) and add php55:</p>
<pre>#LoadModule php5_module /usr/local/Cellar/php53/5.3.28/libexec/apache2/libphp5.so
LoadModule php5_module /usr/local/Cellar/php55/5.5.12/libexec/apache2/libphp5.so</pre>
<p>and restart apache. (if copying and pasting here, make sure of the version homebrew got for you)</p>
<p><del datetime="2014-06-08T21:19:31+00:00">That was a pain. There is a better way using <a href="http://www.vagrantup.com/">vagrant</a> and <a href="https://www.virtualbox.org/">virtualbox</a> to create a virtual machine for each site. It is very handy, but I won't get into that here. But there are lots of recipes out there to get sites up and running quickly and run them on OSes that are exactly like your production environments. You can even use vagrant to provision up an AWS EC2 instance or instances.</del> Check out <a href="https://www.docker.io/">https://www.docker.io/</a></p>
<p>Visiting the local site now gives me the Drupal install wizard. (if you did not create your DB, settings.php and files directory, do that now)</p>
<p>Create/edit your first article at <a href="http://d8server.local/node/add/article">http://d8server.local/node/add/article</a>. <em>Note: I am adding an image and a tag. You should too.</em><br />
<a href="/images/2014/06/Screen-Shot-2014-06-06-at-4.40.28-PM.png"><img src="/images/2014/06/Screen-Shot-2014-06-06-at-4.40.28-PM-300x248.png" alt="My first drupal 8 article" width="300" height="248" class="alignnone size-medium wp-image-95" /></a><br />
Next enable the core REST modules at <a href="http://d8server.local/admin/modules">http://d8server.local/admin/modules</a><br />
<a href="/images/2014/06/Screen-Shot-2014-06-06-at-4.46.42-PM.png"><img src="/images/2014/06/Screen-Shot-2014-06-06-at-4.46.42-PM-300x142.png" alt="Drupal 8 Core REST modules" width="300" height="142" class="alignnone size-medium wp-image-97" /></a><br />
We will need to add a bit of code to allow our client site/app to communicate with our server site/app so we do not get cross-origin resource scripting errors in browsers. There is a D7 module for <a href="https://drupal.org/project/cors">CORS</a>, but it hasn't been ported to D8 yet. I will take a shortcut and which will enable any site to request the d8server resources. In your drupal .htaccess file, add this line:<br />
<code>Header set Access-Control-Allow-Origin "*"</code><br />
For more info, try <a href="http://enable-cors.org/server_apache.html">http://enable-cors.org/server_apache.html</a></p>
<p>Now let's try to access our node RESTfully:<br />
<code>$ curl -i -H "Accept: application/json" http://d8server.local/node/1</code><br />
Hmmm... that gives me an empty JSON object '{}'. Let's try to get HAL (Hypertext Application Language) JSON:<br />
<code>$ curl -i -H "Accept: application/hal+json" http://d8server.local/node/1</code><br />
Hmmm... that give me an error:<br />
<code style="color:red">A fatal error occurred: No authentication credentials provided.</code><br />
Hmmm... time for the google. Ahhh... of course, it is a permissions issue, so go to <a href="http://d8server.local/admin/people/permissions">http://d8server.local/admin/people/permissions</a> and check "Anonymous user" for "Access GET on Content resource" under the section labeled "RESTful Web Services". Google also tells me that application+hal is enabled by default. If we want to change these things and make other entities available for RESTful requests we can either edit sites/default/files/config_XXXX/active/rest.settings.yml or install a contrib module called <a href="https://drupal.org/project/restui">restui</a>. (See <a href="http://drupalize.me/blog/201401/introduction-restful-web-services-drupal-8">http://drupalize.me/blog/201401/introduction-restful-web-services-drupal-8</a> and <a href="https://drupal.org/node/2096019">https://drupal.org/node/2096019</a></p>
<p>Now that I have set the permissions, the last curl hal+json request returns node/1 as HAL JSON. That is enough Drupal for now. (I also installed the restui contrib module)</p>
<h2 id="angularjs-client">AngularJS Drupal client app/site</h2>
<h3>Basic setup</h3>
<p>To make this simple and get started quickly with a bunch of tools and best practices let's install some things. First, if you don't have nodejs installed on your system, do that by:<br />
<code>$ brew install node</code><br />
This also gives you the node package manager npm, which we will use to install some very useful tools. </p>
<ul>
<li><a href="http://yeoman.io/">Yeoman</a></li>
<li><a href="http://gruntjs.com/">Grunt</a></li>
<li><a href="http://bower.io/">Bower</a></li>
<li><a href="https://github.com/yeoman/generator-angular">Angular app generator</a></li>
</ul>
<p>Now that node is installed grab the tools:<br />
<code>$ npm install -g yo grunt-cli bower generator-angular</code><br />
Create a directory for your project and go into it:<br />
<code>$ mkdir d8client && cd $_</code><br />
Create the angular app:<br />
<code>$ yo angular d8client</code><br />
and hit enter (Y) through the questions to get the defaults.</p>
<p>Now you have a well thought out starting point for the client side app. Among other things you can use Grunt to serve your site. It will build your SASS, JS and others for you on change. When it sees a change, it will reload the page for you. To start the server, run:<br />
<code>$ grunt serve</code></p>
<h3>Use git, gitflow and github to version our code</h3>
<p>Let's put this in git and use gitflow. If you are not familiar with gitflow, read <a href="http://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/">http://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/</a> (just use it, it is good for you and your team). If you do not have gitflow:<br />
<code>$ brew install git-flow</code><br />
<code>yo angular</code> created a .gitignore file for you that excludes a bulk of the files so that only your app is versioned. Since I use IntelliJ IDEA, I want to add those files to the .gitignore now. I add these lines:</p>
<pre>.idea
*.iml</pre>
<p>Next, let's init the repo (just hit enter to accept the defaults):<br />
<code>$ git flow init</code><br />
This will drop you into the develop branch. Let's first check what we will be adding to the repo:<br />
<code>$ git status</code><br />
If you see anything that should not go into the repo, add it to your .gitignore. Now let's add files, commit them, and hook it up to a repo at github (my project is at <a href="https://github.com/rkoberg/d8client">https://github.com/rkoberg/d8client</a>):<br />
<code>$ git add -A .<br />
$ git commit -m "initial commit"<br />
$ git remote add origin https://github.com/rkoberg/d8client.git</code><br />
The <code>git remote add origin ...</code> is unique to my project. You can create your own project at github and copy the URL from under the "HTTPS clone URL".<br />
<a href="/images/2014/06/Screen-Shot-2014-06-07-at-4.18.32-PM.png"><img src="/images/2014/06/Screen-Shot-2014-06-07-at-4.18.32-PM-204x300.png" alt="github clone URL" class="alignnone size-medium wp-image-114" /></a><br />
Now we can push our develop branch off to github:<br />
<code>$ git push origin develop</code><br />
Refresh your github page and make sure to select the develop branch to see your files (the page will default to the master branch).</p>
<h3>Back to AngularJS and building the Single Page App (SPA)</h3>
<p>First thing we want to do is handle all of our node pages that will be served from /node/[nid]. The following can easily be done manually, but let's use yeoman and generator-angular to do it for us (checkout the other generators at <a href="https://github.com/yeoman/generator-angular#generators">https://github.com/yeoman/generator-angular#generators</a>.<br />
<code>yo angular:route node</code><br />
Which generates the following:</p>
<pre>app/scripts/controllers/node.js
test/spec/controllers/node.js
app/views/node.html</pre>
<p>and adds a route to our app/scripts/app.js that looks like:</p>
<pre>
      .when('/node', {
        templateUrl: 'views/node.html',
        controller: 'NodeCtrl'
      })</pre>
<p>We want to add the node ID as a parameter to this, which looks like:</p>
<pre>
      .when('/node/:nid', {
        templateUrl: 'views/node.html',
        controller: 'NodeCtrl'
      })</pre>
<p>In our app/scripts/controllers/node.js, change it to look like:</p>
<pre>angular.module('d8clientApp')
  .controller('NodeCtrl', ['$scope', '$routeParams', function ($scope, $routeParams) {

    var nid = $routeParams.nid;
    console.log('nid', nid);

  }]);</pre>
<p>We have changed a few things here. First we made the second argument to the controller an array and put the params to the function first as strings. You want to do this for when we build our app and the JS get minified (trust me). We added the $routeParams which gives us access to the route parameters, namely our node ID. Visit <a href="http://localhost:9000/#/node/1">http://localhost:9000/#/node/1</a> and check out the console to verify we have the node ID. Next we have to change app/scripts/app.js first to handle minification and add a global Accept header to all our requests. This can be done each time we make a request or overriden on some, if we want. Modify the app.js to:</p>
<pre>
  .config(['$routeProvider', '$httpProvider', function ($routeProvider, $httpProvider) {

    $httpProvider.defaults.headers.common.Accept = 'application/hal+json';

    $routeProvider
      .when('/', {</pre>
<p>Now let's add a request to GET the node from Drupal in the app/scripts/controllers/node.js:</p>
<pre>angular.module('d8clientApp')
  .controller('NodeCtrl', ['$scope', '$routeParams', '$http', function ($scope, $routeParams, $http) {

    var nid = $routeParams.nid;
    console.log('nid', nid);

    $http.get('http://d8server.local/node/' + nid).then(function(response) {
      console.log('NodeCtrl GET response', response);
    });

  }]);</pre>
<p>Now check out the console to see the response from Drupal. If you got an error <span style="color:red">Access-Control-Allow-Origin</span>, make sure you added <code>Header set Access-Control-Allow-Origin "*"</code> to your .htaccess as mentioned above or follow instructions <a href="http://enable-cors.org/server_apache.html">here</a>. You should be seeing the returned node hal+json in the console, which we can inspect to set up $scope variables for our template and looks like:<br />
<a href="/images/2014/06/Screen-Shot-2014-06-07-at-5.40.32-PM.png"><img src="/images/2014/06/Screen-Shot-2014-06-07-at-5.40.32-PM-286x300.png" alt="Chrome console output for node/1" width="286" height="300" class="alignnone size-medium wp-image-123" /></a><br />
Drill in and inspect the returned object. Let's make it easy for our template and 'control' what can be used in our HTML template. We will duplicate the default node view eventually, but let's keep it simple for now and just add the type, title and body:</p>
<pre>angular.module('d8clientApp')
  .controller('NodeCtrl', ['$scope', '$routeParams', '$http', function ($scope, $routeParams, $http) {

    var nid = $routeParams.nid;

    $http.get('http://d8server.local/node/' + nid).then(function(response) {
      console.log('NodeCtrl GET response', response);
      var data = response.data;
      $scope.type = data.type[0].target_id;
      $scope.title = data.title[0].value;
      $scope.body = data.body[0].value;
      console.log('NodeCtrl GET $scope', $scope);
    });
  }]);</pre>
<p>And we can set up the app/views/node.html template like:</p>
<pre>&lt;article class="{{type}}">
  &lt;h2>{{title}}&lt;/h2>
  &lt;div ng-bind-html="body">&lt;/div>
&lt;/article></pre>
<p>We need to use the ng-bind-html attribute for the body because it contains markup. If we don't use that the markup will display escaped. But there is a problem: We don't have the image data available. We also only have a reference to the taxonomy term. There has to be a way to get that information in the node request, right? I am not seeing anything in permissions related to files/images and REST services. Let's check google. </p>
<p>...Ufff.... after about 6 hours of trying to figure out how to get the image and taxonomy term data, I give up for now.</p>
