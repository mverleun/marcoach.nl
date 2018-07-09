---
id: 392
title: 'Increasing size of &#8220;physical&#8221; disk with Logical Volumes on VMWare ESX'
date: 2013-05-12T13:05:40+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=392
permalink: /increasing-size-of-physical-disk-with-logical-volumes-on-vmware-esx/
hits:
  - "4638"
ipt_kb_like_article:
  - "0"
categories:
  - Tech Tips
---
With [ESXi](http://www.marcoach.nl/esxi-51-on-mac-pro-31/ "ESXi 5.1 on Mac Pro 3,1") you have to decide on the initial amount of storage when you create a Virtual Machine. When you run out of space you can easily add a second disk to the VM and some data to the new disk. Or you can increase the size of the existing disk if your guest OS supports this. This would simplify things since there is no need to move data around, adjust configurations etc.

Linux with LVM (Logical Volume Manager) supports this. This is how to extend the size of the &#8220;disk&#8221;.<!--more-->

**<span style="color: #ff0000;">Warning:</span>** The following steps can cause dataloss beyond recovery. Before you begin backup your data and read the article in whole first. Proceed on your own risk.

First increase  the &#8220;Provisioned Size&#8221; of your  &#8220;Hard disk&#8221; within VMWare. (Open vSphere Client, Select the VM for which you want to increase the size of the disk, click &#8220;Edit Settings&#8221;, choose the &#8220;Hard disk&#8221; and enter the new  &#8220;Provisioned Size&#8221;.)

Once this is done the VM has to be informed about the increase in size. In the example above the &#8220;Hard disk&#8221; is 60 Gb, but when logged in (as root), the OS still believes that the size of the disk is roughly 40Gb (which was the original size). This can be checked with the command fdisk:

<pre class="">[root@nms ~]# fdisk -l /dev/sda
Disk /dev/sda: 42.9 GB, 42949672960 bytes
255 heads, 63 sectors/track, 5221 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00007890
Device Boot Start End Blocks Id System
/dev/sda1 * 1 13 102400 83 Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2 13 5222 41839616 8e Linux LVM</pre>

After a reboot of the system the disk size has increased but the partition table remains the same.

<pre>[root@nms ~]# fdisk -l /dev/sda
Disk /dev/sda: 64.4 GB, 64424509440 bytes
255 heads, 63 sectors/track, 7832 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00007890
Device Boot Start End Blocks Id System
/dev/sda1 * 1 13 102400 83 Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2 13 5222 41839616 8e Linux LVM</pre>

The size of the physical volume used for LVM is slightly less

<pre>[root@nms ~]# pvscan
 PV /dev/sda2 VG vg_centos lvm2 [39.90 GiB / 28.16 GiB free]
 Total: 1 [39.90 GiB] / in use: 1 [39.90 GiB] / in no VG: 0 [0 ]</pre>

Time to change the partition scheme on the disk, the most dangerous part of the exercise since we  could loose all data if things go wrong. Document your settings very carefully during the next steps.

First document the current partition scheme, turn off DOS compatibilty mode (command c) and switch display units to sectors (command u), display current layout (command p). Nothing is written to disk until the moment you issue the write (w) command.

<pre class="lang:default decode:true">[root@nms ~]# fdisk /dev/sda
WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
 switch off the mode (command 'c') and change display units to
 sectors (command 'u').
Command (m for help): c
DOS Compatibility flag is not set
Command (m for help): u
Changing display/entry units to sectors
Command (m for help): p
Disk /dev/sda: 64.4 GB, 64424509440 bytes
255 heads, 63 sectors/track, 7832 cylinders, total 125829120 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00007890
Device Boot Start End Blocks Id System
/dev/sda1 * 2048 206847 102400 83 Linux
/dev/sda2 206848 83886079 41839616 8e Linux LVM</pre>

In this example partition 2 (/dev/sda2) holds the Linux LVM (type 8e) and starts at sector 206848

Then delete the partition that contains the LVM while still in fdisk. This doesn&#8217;t delete content from the partition but only removes the information about the partition. If recreated correctly nothing gets lost.

<pre class="lang:default decode:true">Command (m for help): d
Partition number (1-4): 2</pre>

Recreate this partition with exact the same starting sector but bigger in size. All the defaults match. A new primary partition is created, number 2, with starting sector 206848 ending at the end of the disk. Make sure that everything matches your settings as displayed above.

<pre class="lang:default decode:true">Command (m for help): n
Command action
 e extended
 p primary partition (1-4)
p
Partition number (1-4): 2
First sector (206848-125829119, default 206848): 
Using default value 206848
Last sector, +sectors or +size{K,M,G} (206848-125829119, default 125829119): 
Using default value 125829119</pre>

Change the type of the partition to 8e!

<pre class="lang:default decode:true">Command (m for help): t
Partition number (1-4): 2
Hex code (type L to list codes): 8e
Changed system type of partition 2 to 8e (Linux LVM)</pre>

Check the new settings prior to writing them to disk

<pre class="lang:default decode:true">Command (m for help): p

Disk /dev/sda: 64.4 GB, 64424509440 bytes
255 heads, 63 sectors/track, 7832 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00007890

Device Boot Start End Blocks Id System
/dev/sda1 * 1 13 102400 83 Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2 13 7833 62811136 8e Linux LVM
Partition 2 does not end on cylinder boundary.</pre>

Write the new partition table if you are confident. Otherwise quit fdisk and start over again.

<pre class="lang:default decode:true ">Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
[root@nms ~]#</pre>

Reboot the system to re-read the new partition table.

After rebooting check the current PV size

<pre>[root@nms ~]# pvdisplay
--- Physical volume ---
PV Name /dev/sda2
VG Name vg_centos
PV Size 39,90 GiB / not usable 2,00 MiB
Allocatable yes
PE Size 4,00 MiB
Total PE 10214
Free PE 7208
Allocated PE 3006
PV UUID b2E4Nb-tRJK-RCy8-wHok-zeih-MwVX-l0EEy8</pre>

Increase the size of the PV (/dev/sda2)

<pre>[root@nms ~]# pvresize -v /dev/sda2
Using physical volume(s) on command line
Archiving volume group "vg_centos" metadata (seqno 9).
Resizing volume "/dev/sda2" to 83677184 sectors.
Resizing physical volume /dev/sda2 from 0 to 15334 extents.
Updating physical volume "/dev/sda2"
Creating volume group backup "/etc/lvm/backup/vg_centos" (seqno 10).
Physical volume "/dev/sda2" changed
1 physical volume(s) resized / 0 physical volume(s) not resized</pre>

And check the result

<pre>[root@nms ~]# pvdisplay
--- Physical volume ---
PV Name /dev/sda2
VG Name vg_centos
PV Size 59,90 GiB / not usable 2,00 MiB
Allocatable yes
PE Size 4,00 MiB
Total PE 15334
Free PE 12328
Allocated PE 3006
PV UUID b2E4Nb-tRJK-RCy8-wHok-zeih-MwVX-l0EEy8</pre>

The VG is automatically resized as well. Now you can extend your logical volumes&#8230; Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.