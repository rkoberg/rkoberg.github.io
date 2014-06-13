---
layout: post
status: publish
published: true
title: notes on setting up php apps at EngineYard (EY)
author:
  display_name: Rob Koberg
  login: rkoberg
  email: rob.koberg@gmail.com
  url: ''
author_login: rkoberg
author_email: rob.koberg@gmail.com
wordpress_id: 64
wordpress_url: http://www.koberg.com/?p=64
date: '2013-12-12 15:04:21 -0800'
date_gmt: '2013-12-12 15:04:21 -0800'
tags: []
comments: []
---
<p>EY works through deployments of branches or tags from your git repo. For now you will probably just have a deployment off of your develop branch (you are using gitflow, right <a href="http://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/">http://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/</a> ).</p>
<p>Each time you deploy, everything will sync up with whatever is in the develop branch of your repository. This means you cannot put things that will vary by the server or any uploaded files that normally live under docroot directory.</p>
<p>During the deployment process they give us deployment 'hooks' which are simple ruby evals of command line commands. For example, you probably have some settings file that should live at a specific location. Since that file should not be in version control, you need to create it on the server. On each EY instance there is a shared directory (/data/{environment-name}/shared). I suggest you create a folder (wp?) inside the shared directory. In there you can put server variable files for settings  and directories for uploaded files. Then you would need to symlink them to the correct locations in the currently deployed project (always at /data/{env-name}/current). This is where deployment hooks come in.</p>
<p>You should have a basic directory structure where you have your docroot directory and a deploy directory directly under your project root:<br />
<code>my-project<br />
-- .git<br />
-- deploy<br />
-- public (or docroot or whatever)</code></p>
<p>Inside the deploy directory will go your deploy hooks. For example, for a drupal I have a file named before_symlink.rb that will be run before EY does its symlinking. It has:<br />
<code>run "sudo ln -s #{config.shared_path}/drupal/files #{config.release_path}/docroot/sites/default/files"<br />
run "sudo ln -s #{config.shared_path}/drupal/files-private #{config.release_path}/docroot/sites/default/files-private"<br />
run "sudo ln -s #{config.shared_path}/drupal/settings.php #{config.release_path}/docroot/sites/default/settings.php"</code><br />
This links the uploadable files destinations, as well as the main settings.php from its location in the shared directory to the necessary locations in the currently deployed app.</p>
<h3>Recipes</h3>
<p>Next you will want to use recipes. First clone (not in your current working project) the ey recipes from <a href="https://github.com/engineyard/ey-cloud-recipes">https://github.com/engineyard/ey-cloud-recipes</a> and follow the 6 step quickstart. Once you are setup you just need to uncomment out some lines for the services you want to enable like varnish_frontend, memcached, ssmtp. I had a problem with varnish and needed to alter the recipe because no pages where being served outside of localhost.</p>
<p>One recipe that is not included is reverting from php54 to php53. Drupal throws tons of warnings if you use php54. To get the recipe for going down to 53, follow the instructions found here: <a href="https://support.cloud.engineyard.com/entries/23768022-PHP-5-4-is-the-version-available-by-default-if-you-need-PHP-5-3">https://support.cloud.engineyard.com/entries/23768022-PHP-5-4-is-the-version-available-by-default-if-you-need-PHP-5-3</a></p>
