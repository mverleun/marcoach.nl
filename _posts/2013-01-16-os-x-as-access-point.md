---
id: 103
title: OS X as Access Point
date: 2013-01-16T18:30:50+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/wordpress/?p=103
permalink: /os-x-as-access-point/
hits:
  - "1355"
categories:
  - Tech Tips
---
OS X has a nice feature to share the internet connected to one interface with nodes connecting to another interface. This makes it easy to use your iMac or MacBook as an access point. Everything is configured automatically as you are used from Apple.

For a training however I had the need to do a similar thing on a OS X server and have control over the SSID and DHCP. Simply sharing the internet connection doesn&#8217;t work here.

After quite some time Google pointed to an undocumented feature that would allow for this. A few moments later there was a search result pointing to a project called HostAP on <a href="https://gist.github.com/3465147" target="_blank">https://gist.github.com/3465147</a>.

This turned out to be the answer I was looking for. It contains the source code of a small program that uses this undocumented feature. With XCode installed it was a matter of download the source code and compiling it using the provided Makefile.

All credits go to Yoshimasa Niwa Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.