---
id: 1022
title: Too many devices when connecting to email server?
date: 2014-07-05T10:35:40+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=1022
permalink: /too-many-devices-when-connecting-to-email-server/
hits:
  - "2822"
categories:
  - Tech Tips
---
# Mail

Lately I&#8217;m experiencing temporary issues when trying to access my mail which is hosted on my OS X server. For a brief moment (several minutes) I can&#8217;t get my mailbox online on my MacBook and my mobile devices won&#8217;t update &#8216;their&#8217;  inbox. This is mainly annoying, no mail is getting lost and after a while I do see the messages, but this time I decided to have a look at the cause. Do I have too many devices?

# OS X Server

I launched Server.app and inspected the logging for Mailserver. It was full with the following messages:

<pre class="wrap:true lang:default decode:true">Jul 05 09:34:40 imap-login: Info: Maximum number of connections from user+IP exceeded (mail_max_userip_connections=20): user=&lt;*****&gt;, method=CRAM-MD5, rip=213.***.***.***, lip=213.***.***.***, TLS</pre>

This is a clear message. And no, I don&#8217;t have 20 devices! But I do have several and it seems that they sometimes simultaneously access the imap server with multiple connections.

This is normal behaviour, a mail client has to check the inbox, sent items. It also has to move messages to folders etc.. So technically nothing is wrong, it is just annoying.

# Dovecot imap

On the server I ran the command sudo serveradmin settings mail:imap and noticed that there is a global setting for the maximum number of imap connections:

<pre class="wrap:true lang:default decode:true">$ sudo serveradmin settings mail:imap
Password:
...
mail:imap:max_imap_connections = 1000
...
server:~ ladmin$</pre>

Unfortunate, this means that we can&#8217;t change the number of simultaneous connections per client with the serveradmin command. Time to make changes low-level.

Open a terminal on the server and edit the relevant config file with your favourite editor as root:

<pre class="wrap:true lang:default decode:true">server:~ ladmin$ sudo nano /Library/Server/Mail/Config/dovecot/conf.d/20-imap.conf</pre>

In this file you&#8217;ll find a section that looks like this:

<pre class="lang:default decode:true ">protocol imap {
  # Space separated list of plugins to load (default is global mail_plugins).
  # stats (in global mail_plugins) and imap_stats (here) are also available
  mail_plugins = $mail_plugins imap_acl imap_quota imap_zlib

  # Maximum number of IMAP connections allowed for a user from each IP address.
  # NOTE: The username is compared case-sensitively.
  mail_max_userip_connections = 20
}</pre>

Change the value 20 on the last line into a higher number, e.g. 30, save the file.

Tell dovecot to reload its configuration:

<pre class="wrap:true lang:default decode:true ">server:~ ladmin$ sudo doveadm reload</pre>

&nbsp;

and you should be fine.

Let&#8217;s hope that this is a sticky value which doesn&#8217;t get lost when updating the server.

&nbsp;

&nbsp; Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.