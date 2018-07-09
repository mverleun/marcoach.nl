---
id: 500
title: Whitelisting email OS X Server
date: 2013-05-21T10:36:10+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=500
permalink: /whitelisting-email-os-x-server/
hits:
  - "3196"
ipt_kb_like_article:
  - "1"
categories:
  - Tech Tips
---
Have you ever noticed that incoming emails are delayed on your OS X Server? This could be caused by a mechanism called greylisting. A very effective spam prevention technique.<!--more-->

# Greylisting

When an OS X Server receives an incoming mail it checks it to see if it is spam. The first check is against a blacklist server to see if the sender is a known spammer.

If not, the server checks to see if the sender is known by checking for previous messages. And if the sender is a known sender the message is accepted immediately. If the sender is unknown the message will be temporary rejected. If the message is offered to OS X server again after several minutes the assumption is that it is coming from a regular mail server and this time the message is accepted. All future messages from this sender will be accepted as well, they are whitelisted.

This mechanism is called greylisting and proves to be very successful, with one drawback&#8230;

# Long delays

When the greylist check is done OS X Server checks against several characteristics to determine the sender. Not only the name and the email address of the sender are checked, but also the mail server that is trying to deliver the message.

If any of these are different when the message is offered again it is considered to be a new message and it will be temporary delayed again.

This happens when providers use different outgoing mail servers. Google and Hotmail are examples of providers using different mail servers to deliver mail. This could result in long delays when receiving mail.

# Whitelisting

The solution is to whitelist email addresses that you trust on the OS X Server manually. This has to be done on the command line. So open a terminal and type:

<pre>sudo serveradmin settings mail:postfix:add_whitelist_domain =  google.com
sudo serveradmin settings mail:postfix:add_whitelist_domain =  hotmail.com</pre>

to whitelist these two domains. Of course you can whitelist more domains. Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.