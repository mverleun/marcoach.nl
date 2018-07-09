---
id: 1100
title: Nagios Push Notifications
date: 2014-11-23T10:27:32+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=1100
permalink: /nagios-push-notifications/
hits:
  - "1432"
categories:
  - Tech Tips
---
# Nagios

I monitor my systems, services and network with [Nagios](http://www.nagios.org). I&#8217;m using different ways to deliver notifications if Nagios has the need to inform me. Until now I used [Prowl](http://www.prowlapp.com) to send a push notification. Prowl would then open [Touchmon](http://www.s-team.at), a front-end app for iOS. This is a two step process where the initial message is displayed by Prowl. It works good, but there is no good first impression.

# MobiosPush

A more elegant, and cheaper, solution is provided by [MobiosPush](http://www.isnapp.nl/mobiospush/). An app that can receive push notifications directly. This removes Prowl from the chain and gives a more clearer first impression when the notification is received.

## Installing MobiosPush

Installing is rather straight forward. First buy the app and install it on your iOS device. Enter information how this app can contact your Nagios server and press &#8216;Get configuration details&#8217;.

This will send an email with all the information that you need to configure your Nagios server.

&nbsp;

&nbsp; Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.