---
id: 108
title: Extending OS X Server backups
date: 2013-04-06T09:53:42+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/wordpress/?p=108
permalink: /extending-os-x-server-backups/
hits:
  - "1086"
ipt_kb_like_article:
  - "1"
categories:
  - Tech Tips
---
# Server Backup

OS X Server has the possibility to use Time Machine for backup purposes.

The backup process is extended with the aid of ServerBackup. A command line utility to create, verify and restore backups of critical service processes such as Open Directory, but also the important server settings.

Server Backup is a very flexible and extensible mechanism that can also be used to make proper backups of MySQL databases or other services by writing simple scripts.<!--more-->

# The tool

ServerBackup resides in /Applications/Server.app/Contents/ServerRoot/usr/sbin/ServerBackup at the time of this writing with OS X Server 2.x . It can be run from the command line and takes several options. Please check the man page (man ServerBackup) for more details. It&#8217;s worth reading.

When updating Server.app the contents of this folder gets replaced with default settings from the new version. You&#8217;ll have to make sure that you recreate the files yourself since this is not update-proof.

ServerBackup -cmd services shows which services will get the &#8216;special&#8217; treatment, in my case the output is as follows:

<pre class="lang:sh decode:true" title="ServerBackup -cmd services"># ServerBackup -cmd services
MessageServer
openDirectory
postgresql
webServer
serverSettings
sharePoints</pre>

Note that the command is entered as root.

## The services plist files

For all the listed services above there are plist files informing ServerBackup how to backup these services. These plist files reside in Applications/Server.app/Contents/ServerRoot/etc/server_backup/. A ls of this directory shows the following on my system:

<pre># ls /Applications/Server.app/Contents/ServerRoot/etc/server_backup/
40-openDirectory.plist    70-webServer.plist
45-serverSettings.plist   75-MessageServer.plist
46-postgresql.plist       com.apple.ServerBackup.plist
55-sharePoints.plist</pre>

So this is were the party starts. The files are numbered. This indicates a specific order in which they are processed. Let zoom in on the file that handles the postgresql backup. Some interesting parts have been indicated, others have been skipped.

<pre class="lang:default decode:true"># cat /Applications/Server.app/Contents/ServerRoot/etc/server_backup/46-postgresql.plist 
... 
ServiceNamepostgresqlBackupBinaryPath 
... 
RestoreBinaryPath 
... 
VerifyBinaryPath 
...</pre>

<span style="line-height: 1.5;">In the file are three references to binaries used for Backup, Restore and Verification. All the work is done using external commands. The plist file above refers to commands in /Applications/Server.app/Contents/ServerRoot/usr/libexec/server_backup/ which has the following content:</span>

<pre class="lang:default decode:true"># ls /Applications/Server.app/Contents/ServerRoot/usr/libexec/server_backup/
MessageServer_backup    openDirectory_verify     sharePoints_backup
MessageServer_restore   opendirectorybackup      sharePoints_restore
MessageServer_verify    postgresql_backup.rb     sharePoints_verify
backuptool.rb           serverSettings_backup    sysexits.rb
openDirectory_backup    serverSettings_restore   webServer_backup.rb
openDirectory_restore   serverSettings_verify</pre>

Some commands, such as sharePoints\_back have multiple binaries for the three tasks. Others, like postgresql\_backup only have a single one which is controlled using options.

Now that we know this it can&#8217;t be that hard to include a script that will backup our mysql database using the proper commands from mysql.

The steps we need to take are the following:
  
1. Write scripts to backup, restore and verify the mysql backup
  
2. Write a plist file that will integrate these scripts into the ServerBackup process.
  
3. Add the new service to ServerBackup

## 1. Scripts

I&#8217;ve placed the scripts in /opt/server_backup to keep OS X as clean as possible.
  
Her is my MySQL backup script:

<pre># cat /opt/server_backup/mysql-backup.sh 
#!/bin/bash
# Backup MySQL tables
# 6 april 2013
# Version 1.0
#
# (c) Marco Veleun, marco@marcoach.nl
#
# This script assumes a mysql installation with the dmg from the mysql website
# check the following path on your system to make sure that the binaries are 
# included in the PATH
PATH=$PATH:/usr/local/mysql/bin
MYSQLUSER="MySQLRootUser"
MYSQLPASS="PasswordForMySQLRootUser"
# Destionation for the backup.
# Should be /.ServerBackups if used with ServerBackup
DESTDIR="/.ServerBackups"
if [ -d $DESTDIR ]
then
 mysqldump --user="$MYSQLUSER" --password="$MYSQLPASS"\
 --all-databases &gt; $DESTDIR/mysql-backup.sql
else
 logger "Backup Destination does not exist"
fi</pre>

## 2. Plist file

The plist file is added to /Applications/Server.app/Contents/ServerRoot/etc/server_backup/. I started numbering at 90 to make sure it is the last service to be back-upped.

Note that this file only creates a backup. There is **NO** script to restore. **Recovering MySQL is NOT part of this document**, the focus is on explaining ServerBackup, not to create the perfect tool.

## Plist file for MySQL backup

It has been trimmed down and probable can even be trimmed down more. But it works for me. One day I might include a restore option as well, for now I&#8217;m happy to know that my TimeMachine backup does include a proper dump of my database.

<pre class="lang:default decode:true"># cat /Applications/Server.app/Contents/ServerRoot/etc/server_backup/90-mysql.plist 
&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"&gt;
&lt;plist version="1.0"&gt;
&lt;dict&gt;
 &lt;key&gt;ServiceName&lt;/key&gt;
 &lt;string&gt;MySQL&lt;/string&gt;
 &lt;key&gt;BackupBinaryPath&lt;/key&gt;
 &lt;string&gt;/opt/server_backup/mysql-backup.sh&lt;/string&gt;
 &lt;key&gt;BackupLog&lt;/key&gt;
 &lt;string&gt;/private/var/log/server_backup/mysql_backup.log&lt;/string&gt;
 &lt;key&gt;BackupActions&lt;/key&gt;
 &lt;dict&gt;
 &lt;key&gt;Default&lt;/key&gt;
 &lt;string&gt;all&lt;/string&gt;
 &lt;key&gt;Backup&lt;/key&gt;
 &lt;string&gt;backup&lt;/string&gt;
 &lt;key&gt;Help&lt;/key&gt;
 &lt;string&gt;help&lt;/string&gt;
 &lt;key&gt;Version&lt;/key&gt;
 &lt;string&gt;version&lt;/string&gt;
 &lt;/dict&gt;
 &lt;key&gt;PreBackupServiceCommands&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;PostBackupServiceCommands&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;RestoreBinaryPath&lt;/key&gt;
 &lt;string&gt;&lt;/string&gt;
 &lt;key&gt;RestoreLog&lt;/key&gt;
 &lt;string&gt;&lt;/string&gt;
 &lt;key&gt;RestoreActions&lt;/key&gt;
 &lt;dict&gt;
 &lt;key&gt;Default&lt;/key&gt;
 &lt;string&gt;all&lt;/string&gt;
 &lt;key&gt;Browse&lt;/key&gt;
 &lt;string&gt;browse&lt;/string&gt;
 &lt;/dict&gt;
 &lt;key&gt;PreRestoreServiceCommands&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;PostRestoreServiceCommands&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;VerifyBinaryPath&lt;/key&gt;
 &lt;string&gt;&lt;/string&gt;
 &lt;key&gt;VerifyLog&lt;/key&gt;
 &lt;string&gt;&lt;/string&gt;
 &lt;key&gt;VerifyActions&lt;/key&gt;
 &lt;dict&gt;
 &lt;key&gt;Default&lt;/key&gt;
 &lt;string&gt;all&lt;/string&gt;
 &lt;/dict&gt;
 &lt;key&gt;ConfigurationFiles&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;DataFiles&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;ExclusionPaths&lt;/key&gt;
 &lt;array/&gt;
 &lt;key&gt;Version&lt;/key&gt;
 &lt;string&gt;10.8&lt;/string&gt;
&lt;/dict&gt;
&lt;/plist&gt;</pre>

Make sure that the version number matches the major number of your OS X version (Mountain Lion = 10.8, Mavericks = 10.9)

## 3. Adding Service

We have to inform ServerBackup that we&#8217;ve created a new service. This service is added to the list of services:

<span style="font-size: 24px; font-weight: bold; line-height: 1;">4. Testing, testing, one, two&#8230;</span>

One: Check if the service is listed:

<pre class="lang:default decode:true"># ServerBackup -cmd services
MySQL
MessageServer
openDirectory
postgresql
webServer
serverSettings
sharePoints</pre>

Two: Go for it

<pre># ServerBackup -cmd backup -source /
*** nextPath := 40-openDirectory.plist
 *** nextPath := 45-serverSettings.plist
*** nextPath := 46-postgresql.plist
*** nextPath := 55-sharePoints.plist
*** nextPath := 70-webServer.plist
2013-04-06 11:35:07.044 ServerBackup[54552:707] SRC := /Library/Server/Web/Config/apache2/
DST := /.ServerBackups/webServer
2013-04-06 11:35:07.221 ServerBackup[54552:707] SRC := /etc/certificates/
DST := /.ServerBackups/webServer
*** nextPath := 75-MessageServer.plist
*** nextPath := 90-mysql.plist</pre>

And in another terminal while the backup is running:

<pre class="lang:default decode:true crayon-selected"># ls -l /.ServerBackups/
total 10371912
drwxr-xr-x 6  root wheel 204         Apr 6 11:35 MessageServer
drwxr-xr-x 2  root wheel 68          Apr 6 11:35 MySQL
-rw-r--r-- 1  root wheel 5310416786  Apr 6 11:48 mysql-backup.sql
drwxr-xr-x 2  root wheel 68          Apr 6 11:34 openDirectory
drwxr-xr-x 2  root wheel 68          Apr 6 11:35 postgresql
drwxr-xr-x 45 root wheel 1530        Apr 6 11:35 serverSettings
drwxr-xr-x 14 root wheel 476         Apr 6 11:35 sharePoints
drwxr-xr-x 9  root wheel 306         Apr 6 11:35 webServer</pre> Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.