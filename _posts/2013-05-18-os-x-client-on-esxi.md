---
id: 483
title: OS X Client on ESXi
date: 2013-05-18T12:19:16+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=483
permalink: /os-x-client-on-esxi/
hits:
  - "1784"
categories:
  - Tech Tips
---
In a previous [post](http://www.marcoach.nl/esxi-51-on-mac-pro-31/ "ESXi 5.1 on Mac Pro 3,1") I wrote about installing ESXi on a Mac Pro 3,1 to run an OS X Client.

The installation of the client went smooth. I managed to install OS X using the &#8216;Install OS X.app&#8217; which I downloaded from the App store. Instructions can be found on the website of [VMware](http://partnerweb.vmware.com/GOSIG/MacOSX_10_8.html).

The first lesson I learned is to change the default screensaver that appears prior to logging into the system. The default one consumes all the CPU it has been granted! The one that seems to go easiest is called &#8216;Message&#8217;.<!--more-->

This has to be done as root. So I opened a terminal and typed:

<pre>sudo /Applications/System\ Preferences.app/Contents/MacOS/System\ Preferences</pre>

to start System Preferences as the user root. This is something only an administrator can type.

The next steps were to change the screensaver to &#8216;Messages&#8217; and to disable it. Disabling the screensaver is not honored by OS X 10.8.3 unfortunate.

Other settings changed:

Disabled harddisk sleep, display sleep etc. Let ESXi be in charge of saving energy.

## Things not working:

### Push notifications:

apsd is not able to register the machine. This prevents management with Profile Manager pushing settings. Manually downloading profiles works fine.

### iCloud:

&#8216;Find my Mac&#8217; seems to register fine, but when logging into www.icloud.com the device is not listed.

&#8216;Back to my Mac&#8217; is not able to register.

### Startup Modifiers:

Not possible to boot into Recovery Partition. Haven&#8217;t tried &#8216;File Vault 2&#8217; so this might work.

Not possible to boot from another volume selected with &#8216;System Preferences ➤ Startup Disk&#8217;.

Netboot doesn&#8217;t work. Nor from the VM Boot menu or System Preferences.

If someone knows how to get one of the above working, please share it with me. Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.