---
id: 548
title: How to change the Model Identifier in VMWare ESXi for OS X clients
date: 2013-05-27T20:58:56+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=548
permalink: /changing-the-model-identifer-in-vmware-esxi/
hits:
  - "3112"
categories:
  - Tech Tips
---
# Model Identifier

Each Apple device has its own model identifier. My MacPro from early 2008 has is known as a MacPro3,1. This identifier can be found in System Report and is useful when troubleshooting certain issues.<!--more-->

When installing OS X under ESXi the model indentifier is different. My virtual machines are known as VMWare7,1 and show up in Finder as iMac&#8217;s. You might want to change this and it is not so difficult if you are familiar working on the command line.

# How to change the model identifier in ESXi

Changing the model identifier requires command line access to the ESXi host. I&#8217;ve enabled ssh and can remote log in as root. You also need to know where your virtual machine is stored. In my case the machine is stored in datastore2. And, last but not least, you need to master vi enough to add a line to a config file.

First shutdown the guest and wait for it to come to a full stop. When the guest has come to a full stop locate the vmx file of the guest.

Before you continue consider making a backup of the vmx file.

Add the following line to the .vmx file of the virtual machine:

<pre>hw.model="MacPro3,1"</pre>

Replace the model identifier with the one that you want to use. An extensive list can be found [here](http://www.everymac.com/systems/by_capability/mac-specs-by-machine-model-machine-id.html "Model identifiers").

You can use another option instead:

<pre>hw.model.reflectHost = "TRUE"</pre>

This will use the original model identifier from the actual hardware instead of a model identifier that you choose yourself. Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.