---
id: 99
title: Start iTunes during boot
date: 2012-07-16T08:28:58+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/wordpress/?p=99
permalink: /itunes-starten-tijdens-boot/
hits:
  - "1632"
categories:
  - Tech Tips
---
iTunes is a great application that could act very well as a server to share musis, iOS apps, books, movies etc. using homesharing.

To start iTunes a user would have to log on to the system and start it manually. Would it not be great if iTunes were started at boot time without the need for someone to logon to the system?

<!--more-->

This can be accomplished by creating a so called Launch Daemon. Launchd is the is the first process on an OS X server to be started. It&#8217;s like init on a classic Unix system. Launchd, amongst other things, starts services at boot time.

By creating a plist file in /Library/LaunchDaemons it is possible to start iTunes on behalf of a user without the need for this user to logon.

In the example below iTunes is started with the credentials of the user ladmin. Only the Library of this user is shared if homesharing is turned on by this user.

Creating the plist file requires root priviliges and some basic knowledge of working with the command line.

First decide as which user Itunes should be started. Then create the plist file on the Desktop using the free editor Text Wrangler which can be downloaded from the App Store.

The contents of the plist file should look like this:

<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"&gt;
&lt;plist version="1.0"&gt;
&lt;dict&gt;
&lt;key&gt;Label&lt;/key&gt;
&lt;string&gt;com.example.iTunes&lt;/string&gt;
&lt;key&gt;OnDemand&lt;/key&gt;
&lt;false/&gt;
&lt;key&gt;ProgramArguments&lt;/key&gt;
&lt;array&gt;
&lt;string&gt;/Applications/iTunes.app/Contents/MacOS/iTunes&lt;/string&gt;
&lt;/array&gt;
&lt;key&gt;RunAtLoad&lt;/key&gt;
&lt;true/&gt;
&lt;key&gt;UserName&lt;/key&gt;
&lt;string&gt;<em>iTunesGebruiker</em>&lt;/string&gt;
&lt;key&gt;WorkingDirectory&lt;/key&gt;
&lt;string&gt;/Users/<em>iTunesUser</em>/&lt;/string&gt;
&lt;key&gt;ServiceDescription&lt;/key&gt;
&lt;string&gt;iTunes&lt;/string&gt;
&lt;/dict&gt;
&lt;/plist&gt;</pre>

Replace &#8216;iTunesUser&#8217; with the name of the user you decided on earlier.

Vul op de plaats waar iTunesGebruiker staat de naam van de gebruiker in die wordt gebruikt om iTunes te beheren.

Move this file to the proper location by issuing the following command:

<pre>sudo mv Desktop/com.example.iTunes.plist /Library/LaunchDaemons/</pre>

Start iTunes with the command

<pre>sudo launchctl load -w com.example.iTunes.plist</pre>

<div>
  Or simply reboot&#8230;
</div> Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.