---
id: 959
title: 'OS X Mavericks: Removing or reinstalling Server.app'
date: 2014-02-21T10:39:35+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=959
permalink: /os-x-mavericks-removing-reinstalling-server-app/
hits:
  - "26153"
ipt_kb_like_article:
  - "1"
categories:
  - Tech Tips
---
# OS X Server

Converting a Mavericks computer in a true server is easy. Just purchase Server.app from the Mac Appstore and configure it.

Many people, like me, also install a copy of Server.app on a system that they want to use to remotely administer the server, which works great. No screen sharing required to remotely administer a server!

## Mistakes happen

Some people, like me, sometimes launch Server.app on their administrator computer and accidentally click on continue and convert their administrator computer into a Server as well. This will start quite a few additional processes on your administrator computer but in itself is harmless.

## <!--more-->Removing Server.app

It is quite simple to remove Server.app from your administrator computer. Simply moving it from the Applications folder to the Trash is enough to stop the services. You need to provide the password of an administrator to do this. As soon as the system detects that the app has been removed you&#8217;ll get the following message:

[<img class="alignleft size-full wp-image-960" alt="Server app removal" src="http://www.marcoach.nl/wp-content/uploads/2014/02/Screen-Shot-2014-02-21-at-09.26.50.png" width="534" height="289" srcset="https://www.marcoach.nl/wp-content/uploads/2014/02/Screen-Shot-2014-02-21-at-09.26.50.png 534w, https://www.marcoach.nl/wp-content/uploads/2014/02/Screen-Shot-2014-02-21-at-09.26.50-300x162.png 300w" sizes="(max-width: 534px) 100vw, 534px" />](http://www.marcoach.nl/wp-content/uploads/2014/02/Screen-Shot-2014-02-21-at-09.26.50.png)

## Service Data

As you can see the service data is not removed. This is the location where emails, calendar information, wiki data etc. are stored.

By default this is the folder /Library/Server and it has the following contents:

[<img class="alignleft size-full wp-image-962" alt="/Library/Server" src="http://www.marcoach.nl/wp-content/uploads/2014/02/Screen-Shot-2014-02-21-at-09.33.40.png" width="894" height="650" srcset="https://www.marcoach.nl/wp-content/uploads/2014/02/Screen-Shot-2014-02-21-at-09.33.40.png 894w, https://www.marcoach.nl/wp-content/uploads/2014/02/Screen-Shot-2014-02-21-at-09.33.40-300x218.png 300w" sizes="(max-width: 894px) 100vw, 894px" />](http://www.marcoach.nl/wp-content/uploads/2014/02/Screen-Shot-2014-02-21-at-09.33.40.png)

Move this folder to Trash and you&#8217;ve completed the removal of Server.app

Thanks Apple for this nice segragation of Mavericks as an OS and Server.app on top of this great OS. Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.