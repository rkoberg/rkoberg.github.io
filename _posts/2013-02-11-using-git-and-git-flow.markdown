---
layout: post
status: publish
published: true
title: Using git and gitflow with Acquia and EngineYard
author:
  display_name: Rob Koberg
  login: rkoberg
  email: rob.koberg@gmail.com
  url: ''
author_login: rkoberg
author_email: rob.koberg@gmail.com
wordpress_id: 5
wordpress_url: http://koberg.local/?p=5
date: '2013-02-11 01:57:53 -0800'
date_gmt: '2013-02-11 01:57:53 -0800'
tags:
- git
- gitflow
- version control
comments: []
---
<p>First read: <a title="Why aren't you using git-flow?" href="http://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/">http://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/</a></p>
<p>More depth: <a title="A successful Git branching model" href="http://nvie.com/posts/a-successful-git-branching-model/">http://nvie.com/posts/a-successful-git-branching-model/</a></p>
<p>Video tutorial: <a title="A short introduction to Git Flow from Mark Derricutt" href="http://vimeo.com/16018419">http://vimeo.com/16018419</a></p>
<p>And: <a title="Smart branching with SourceTree and Git-flow" href="http://blog.sourcetreeapp.com/2012/08/01/smart-branching-with-sourcetree-and-git-flow/">http://blog.sourcetreeapp.com/2012/08/01/smart-branching-with-sourcetree-and-git-flow/</a>,&nbsp;<a title="Getting Started – Git-Flow" href="http://yakiloo.com/getting-started-git-flow/">http://yakiloo.com/getting-started-git-flow/ </a></p>
<p>I want to outline the process we are using at two separate hosting services:</p>
<ul>
<li>EngineYard - a ruby project that has multiple developers. Requires a separate deployment step</li>
<li>Acquia - multisite drupal projects, now with only me as the developer. For Acquia, we are using them as the remote repository which allows us to just push our changes to the remote repo to have them deploy.</li>
</ul>
<p>The process is similar for both projects. Each uses three separate environments: development, stage/test, production. You will see some redundancy if you deploy to dev, then to stage and then to prod.</p>
<h3>Deploying to development</h3>
<ol>
<li>Ensure you are in the develop branch:<br />
<code>$ git branch</code><br />
and/or<br />
<code>$ git checkout develop</code>
</li>
<li>Get the latest code:<br />
<code>$ git pull origin develop</code>
</li>
</ol>
<ul>
<li>
For engineyard:<br />
<code>$ ey deploy -e dev</code>
</li>
<li>
For acquia (just push to the develop branch):<br />
<code>$ git push origin develop</code>
</li>
</ul>
<h3>Deploying to stage/test</h3>
<ol>
<li>Ensure you are in the develop branch:<br />
<code>$ git branch</code><br />
and/or<br />
<code>$ git checkout develop</code>
</li>
<li>Get the latest code:<br />
<code>$ git pull origin develop</code>
</li>
<li>Switch over to the master branch:<br />
<code>$ git checkout master</code><br />
<code>$ git pull origin master</code>
</li>
<li>Merge your changes:<br />
<code>$ git merge develop</code>
</li>
<li>Push your changes to the remote repo:<br />
<code>$ git push origin master</code><br />
This deploys the master branch to acquia.
</li>
</ol>
<ul>
<li>
For engineyard:<br />
<code>$ ey deploy -e staging</code>
</li>
</ul>
<h3>Deploying to production</h3>
<p>For acquia we use their web control panel to simply drag and drop the stage code to production. This automatically tags master and deploys it to production. For engineyard we set up a slightly more elaborate (and correct) release process. In general do the following steps for both hosting services.</p>
<ol>
<li><code>$ git checkout master</code></li>
<li><code>$ git pull origin master</code></li>
<li><code>$ git checkout develop</code></li>
<li><code>$ git pull origin develop</code></li>
<li><code>$ git diff master develop</code></li>
</ol>
<p>You should not see any differences.</p>
<p>For engineyard we will create a release and publish that with gitflow. I usually just check the rails app's config/ey.yml for the latest version number, but you should also be able to do:<br />
<code>$ git branch -r</code><br />
The version number will look like: 1.1.3</p>
<p>Create a release with the next version number, e.g.<br />
<code>$ git flow release start 1.1.4</code><br />
Update the rails engineyard config by editing:<br />
<code>$ vim config/ey.yml</code><br />
Add it by being extra careful at this point:<br />
<code>$ git add -p</code><br />
Commit this changed config file:<br />
<code>$ git commit -m "Updated release branch for production."</code><br />
Now use gitflow to publish and finish the release:<br />
<code>$ git flow release publish 1.1.4</code><br />
<code>$ git flow release finish 1.1.4</code><br />
Enter a message like "releasing 1.1.4 to production."<br />
You changes are now in the master branch.<br />
<code>$ git co master</code><br />
<code>$ git push origin master</code><br />
Deploy to engineyard:<br />
<code>$ ey deploy -e production</code><br />
Finally, after the deployment, <strong>make sure you switch back to develop</strong> so you don't forget next time you make some changes.<br />
<code>$ git co develop</code></p>
