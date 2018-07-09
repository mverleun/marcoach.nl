---
id: 866
title: Create your own Fusion Drive using partitions
date: 2013-11-29T17:55:57+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=866
permalink: /create-fusion-drive-using-partitions/
hits:
  - "1116"
categories:
  - Tech Tips
---
# Fusion Drive

Fusion Drive is a data storage technology developed by Apple Inc. Announced as part of an Apple event held on October 23, 2012, the technology combines a hard disk drive (of 1 TB or more) with a NAND flash storage (solid-state drive of 128 GB) and presents it as a single logical volume with the space of both drives combined. (Source: [Wikipedia](http://en.wikipedia.org/wiki/Fusion_Drive))

It combines a relative expensive (SSD) and relative cheap (Rotational) drive into a virtual drive. The system will allocate often accessed data blocks on the SSD part of the drive and less frequent blocks are allocated on the rotational drive.

# Creating your own fusion drive

Is described on many sites. The sites I&#8217;ve found so far all combine the entire SSD drive and the entire rotational drive into a fusion drive. In most cases  this makes sense, but I had the desire to have a fixed partition to work with, that is not part of the fusion drive.<!--more-->

This allows me to easily install additional versions of OS X for testing purposes and do other things.

I went ahead and removed the DVD drive from my MacBook Pro to make room for an additional disk drive.

Now I have a 120Gb SSD drive and a 1Tb (7200 rpm) rotational drive in my MacBook. I&#8217;ve created a fusion drive consisting of the 120Gb SSD and 500Gb from the rotational drive. The other 500Gb are there to use for other purposes as mentioned above. If you are curious on how I achieved this please read on. Be careful, you&#8217;ll have to make a backup first since the process will wipe data.

## 1. Time Machine Backup

I make frequent backups using an OS X server which functions as a Time Machine destination. But for this purpose I made a Time Machine backup on an external drive for the sake of performance. Once the backup had completed and I was certain that all relevant data was there I continued to the next step.

## 2. Replace DVD player with hard drive

Using a special enclosure from <a href="http://hdcaddy.com/product.php?id_product=10" class="broken_link" rel="nofollow">HDCaddy</a> I replaced the DVD drive with a 1Tb rotational harddrive.

## 3. Creating the fusion drive

Creating the fusion drive wasn&#8217;t that difficult either. I started the system from the recovery volume that was still available on my SSD drive and used the Disk Util to partition my 1Tb drive into two volumes. The name of the first volume is &#8216;Fusion&#8217; and the name of the seconde one is &#8216;Data&#8217;. (I could have done this from the command line as well, but this seemed easier. )

Next step was to open a terminal and do some good old command line work.

<span style="color: #ff0000;">Don&#8217;t type the commands I typed blindly&#8230; You&#8217;ll have to adapt them to match your specific system. I won&#8217;t take any responsibility if you mess up your system.</span>

First I gathered information about the disks and volumes in my system:

<pre class="lang:sh decode:true">-bash-3.2# diskutil list
/dev/disk0
#:                    TYPE NAME               SIZE       IDENTIFIER
0:   GUID_partition_scheme                   *121.3 GB   disk0
1:                     EFI EFI                209.7 MB   disk0s1
2:               Apple_HFS Macintosh HD       119.7 GB   disk0s2
3:              Apple_Boot Recovery HD        650.0 MB   disk0s3

/dev/disk1
#:                    TYPE NAME                SIZE      IDENTIFIER
0:  Apple_partition_scheme                    *1.3 GB    disk1
1:     Apple_partition_map                    30.7 KB    disk1s1
2:               Apple_HFS OS X Base System   1.3 GB     disk1s2

/dev/disk2
#:                    TYPE NAME               SIZE       IDENTIFIER
0:   GUID_partition_scheme                   *1.0 TB     disk2
1:                     EFI EFI              209.7 MB     disk2s1
2:               Apple_HFS Fusion           500.1 GB     disk2s2
3:               Apple_HFS Data             499.6 GB     disk2s3

/dev/disk3
#:                   TYPE NAME                SIZE       IDENTIFIER
0:                        untitled           *524.3 KB   disk3
/dev/disk4
#:                   TYPE NAME                SIZE       IDENTIFIER
0:                        untitled           *524.3 KB   disk4
-bash-3.2#</pre>

The two partitions that I want to use to create a fusion drive are disk0s2 and disk2s2, indicated in blue above. Creating the fusion drive was easy. First I had to create a Logical Volume Group consisting of the two partitions mentioned above.

<pre class="lang:sh decode:true">-bash-3.2# diskutil coreStorage create Fushion disk0s2 disk3s2
Started CoreStorage operation
Unmounting disk0s2
Touching partition type on disk0s2
Adding disk0s2 to Logical Volume Group
Unmounting disk3s2
Touching partition type on disk3s2
Adding disk3s2 to Logical Volume Group
Creating Core Storage Logical Volume Group
Switching disk0s2 to Core Storage
Switching disk3s2 to Core Storage
Waiting for Logical Volume Group to appear
Discovered new Logical Volume Group "BCF63BBB-037B-4AED-9A9D-01F3DCAD4601"
Core Storage LVG UUID: BCF63BBB-037B-4AED-9A9D-01F3DCAD4601
Finished CoreStorage operation
-bash-3.2#</pre>

It is always a good idea to check the result:

<pre class="lang:sh decode:true crayon-selected">-bash-3.2# diskutil cs list
CoreStorage logical volume groups (1 found)
+-- Logical Volume Group BCF63BBB-037B-4AED-9A9D-01F3DCAD4601
    =========================================================
  Name:      Fushion
  Status:    Online
  Size:      619791290368 B (619.8 GB)
  Free Space:614296743936 B (614.3 GB)
  |
  +-&lt; Physical Volume CC437DBA-68EF-427B-BC51-813C3D14E231
  |   ----------------------------------------------------
  | &nbsp; Index:  0
  | &nbsp; Disk:   disk0s2
  | &nbsp; Status: Online
  | &nbsp; Size:   119688847360 B (119.7 GB)
  |
  +-&lt; Physical Volume 4CA488FA-331D-4AEE-B698-FEC043A49CCB
      ----------------------------------------------------
      Index:   1
      Disk:    disk3s2
      Status:  Online
      Size:    500102443008 B (500.1 GB)
-bash-3.2#</pre>

This confirms that I have a Logical Volume Group consisting of two partitions: disk0s2 and disk3s2. The UUID of the volume group is <span style="color: #339966;">BCF63BBB-037B-4AED-9A9D-01F3DCAD4601</span><span style="color: #000000;">. This number is needed for the next step where I create the volume containing the filesystem:</span>

<pre class="lang:sh decode:true">-bash-3.2# diskutil cs createVolume BCF63BBB-037B-4AED-9A9D-01F3DCAD4601 jhfs+ Fusion 100%

The Core Storage Logical Volume Group UUID is BCF63BBB-037B-4AED-9A9D-01F3DCAD4601
Started CoreStorage operation
Waiting for Logical Volume to appear
Formatting file system for Logical Volume
Initialized /dev/rdisk2 as a 572 GB case-insensitive HFS Plus volume with a 49152k journal
Mounting disk
Core Storage LV UUID: E8AACDF2-00A0-479B-9CA2-C8A984510F21
Core Storage disk: disk2
Finished CoreStorage operation
-bash-3.2#</pre>

And there it is. A volume with the name Fusion has been created. Time to quit the terminal.

## 4. Restore Time Machine Backup

Now that the term has quitted it is time to restore the Time Machine Backup. This will rename the volume to it original name, &#8216;Macintosh HD&#8217;.

After completing the restore I was curious and opened &#8216;System Information&#8217; to look at the storage settings.  &#8216;System Information&#8217; (or &#8216;System Report&#8217;) is the only tool that I&#8217;m aware of that can display detailed information in the GUI Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.