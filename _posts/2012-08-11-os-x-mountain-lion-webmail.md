---
id: 100
title: OS X Mountain Lion Webmail
date: 2012-08-11T09:19:06+00:00
author: Marco Verleun
excerpt: |
  <p><span style="background-color: #ffffff; color: #565656; font-family: Helvetica, Arial, Geneva, sans-serif; font-size: 13px; line-height: 18px;">Webmail is gone in Mountain Lion. Not all users are pleased so it is time to install </span><a class="cc-route-enabled" href="http://roundcube.net/" target="_blank" style="margin: 0px; padding: 0px; border: 0px; outline: 0px; font-style: inherit; font-family: inherit; vertical-align: baseline; color: #4b8aba;">RoundCube</a><span style="background-color: #ffffff; color: #565656; font-family: Helvetica, Arial, Geneva, sans-serif; font-size: 13px; line-height: 18px;"> mail to give them their beloved webmail back. RoundCube was used with Lion as the webmail solution and can be download from </span><a class="cc-route-enabled" href="http://roundcube.net/" target="_blank" style="margin: 0px; padding: 0px; border: 0px; outline: 0px; font-style: inherit; font-family: inherit; vertical-align: baseline; color: #4b8aba;">here</a><span style="background-color: #ffffff; color: #565656; font-family: Helvetica, Arial, Geneva, sans-serif; font-size: 13px; line-height: 18px;">.</span></p>
  <p style="margin: 11px 0px 0px; padding: 0px; border: 0px; outline: 0px; font-size: 13px; font-family: Helvetica, Arial, Geneva, sans-serif; vertical-align: baseline; color: #565656; line-height: 18px; background-color: #ffffff;">But just installing it is not enough, it has to be installed as a full featured webapp in order to integrate nice with Mountain Lion. The standard security settings of the onboard PostgresQL database are pretty tight. They can be loosened, but to avoid potential update problems we'll install MySQL as a second database server. Note that installing RoundCube mail requires some experience of working at the commandline.</p>
  <p style="margin: 11px 0px 0px; padding: 0px; border: 0px; outline: 0px; font-size: 13px; font-family: Helvetica, Arial, Geneva, sans-serif; vertical-align: baseline; color: #565656; line-height: 18px; background-color: #ffffff;"><span style="font-size: 16px; line-height: 21px;"> </span></p>
layout: post
guid: http://www.marcoach.nl/wordpress/?p=100
permalink: /os-x-mountain-lion-webmail/
hits:
  - "12018"
categories:
  - Tech Tips
---
# Webmail

Webmail is gone in Mountain Lion. Not all users are pleased so it is time to install <a href="http://roundcube.net/" target="_blank">RoundCube</a> mail to give them their beloved webmail back. RoundCube was used with Lion as the webmail solution and can be download from <a href="http://roundcube.net/" target="_blank">here</a>.

But just installing it is not enough, it has to be installed as a full featured webapp in order to integrate nice with Mountain Lion. The standard security settings of the onboard PostgresQL database are pretty tight. They can be loosened, but to avoid potential update problems we&#8217;ll install MySQL as a second database server. Note that installing RoundCube mail requires some experience of working at the commandline.

**Update**: I just received notice that the folks at [topicdesk.com](http://topicdesk.com) have automated the process of restoring webmail functionality. If you don&#8217;t want to perform the steps written below you can use their installer. More info to be found [here](http://topicdesk.com/downloads/roundcube/roundcube-download).<!--more-->

### File locations

One of the first noticable differences between Lion and Mountain Lion is that the locations of configuration files have changed. Instead of configuring the webserver from /etc/apache2 as with Lion does Mountain Lion use /Library/Server/Web/Config/apache2/ as the configuration directory. In this directory is a subdirectory webapps that contains all the web-based applications such as calendar, wiki etc. The file com.example.mywebapp.plist will be used as a template to create the startup config plist file.

The php code from RoundCube mail will be installed outside of the standard OS X installation to ensure that the risk of loosing it with the next upgrade is minimal.

For this purpose it will be installed in /opt/webapps/webmail

### Creating the filesystem structure and downloading RoundCube mail

Open a terminal as an administrator and  issue the following command:

<pre>$ sudo mkdir -p /opt/webapps/webmail</pre>

Change to this directory:

<pre>$ cd /opt/webapps/webmail</pre>

With Safari download the RoundCube tar file from the download section on <a href="http://roundcube.net/" target="_blank">http://roundcube.net/</a> as the same administrator user that you are on the command line. The downloaded file wil be installed in the Downloads folder of this user.

Extract the downloaded tar file. Replace <username> with the shortname of the admin user that download RoundCube

<pre>$ sudo tar xvf /Users/&lt;username&gt;/Downloads/roundcubemail-0.8.0.tar</pre>

Move the contents of the extracted file to the right location:

<pre>$ sudo mv roundcubemail-0.8.0/* .</pre>

And check that everything is there:

<pre>$ ls
CHANGELOG		bin			program
INSTALL			config			robots.txt
LICENSE			index.php		roundcubemail-0.8.0
README.md		installer		skins
SQL			logs			temp
UPGRADING		plugins</pre>

Next visit the MySQL website and download the MySQL installer. The URL is <a href="http://www.mysql.com/downloads/mysql/" target="_blank">http://www.mysql.com/downloads/mysql/</a>, a direct download link is <a href="ftp://ftp.easynet.be/mysql//Downloads/MySQL-5.5/mysql-5.5.27-osx10.6-x86_64.dmg" target="_blank">ftp://ftp.easynet.be/mysql//Downloads/MySQL-5.5/mysql-5.5.27-osx10.6-x86_64.dmg</a>

Select the Mac OS X version, download it and install it as any other package. Note that the version indicates that it is for 10.6 (Snow Leopard), but it works fine with Mountain Lion (10.8)

The MySQL files are installed in /usr/local/mysql-5.5.27-osx10.6-x86_64. This path could be different if you download a newer version of MySQL. Don&#8217;t forget to start the MySQL server through System Preferences.

<div>
</div>

With all the files in place it is time to create the database

### Create and configure the database

First go to the MySQL binary directory and then create a database user that will access the database:

<pre>$ cd /usr/local/mysql-5.5.27-osx10.6-x86_64/bin/</pre>

Next create the database:

<pre>$ ./mysqladmin create roundcube</pre>

or, if your database is password protected:

<pre>$ ./mysqladmin -u root -p create roundcube</pre>

Assign a user and  password to the database and import the SQL statements to create the database

<pre>$ ./mysql roundcube
mysql&gt; GRANT ALL ON roundcube.* TO 'roundcube'@'localhost' IDENTIFIED BY 'the_new_password';
mysql&gt; exit
$ ./mysql -p roundcube &lt; /opt/webapps/webmail/SQL/mysql.initial.sql</pre>

Replace &#8216;the\_new\_password&#8217; with the password that you want to assign.

Descend into the config directory and copy the template database config file to the file that will be used:

<pre>cd /opt/webapps/webmail/config
sudo cp db.inc.php.dist db.inc.php</pre>

Edit the file db.inc.php with your favorite editor and look for the line that starts with $rcmail_config. Don&#8217;t forget to use sudo before the name of your editor otherwise you can&#8217;t save the changes.

Change this line:

<pre>$rcmail_config['db_dsnw'] = 'mysql://roundcube:pass@localhost/roundcube';</pre>

into this one, replacing the\_new\_password with the password created above:

<pre>$rcmail_config['db_dsnw'] = 'mysql://roundcube:the_new_password@127.0.0.1/roundcube';</pre>

<span style="color: #ff0000;">Important: The ip address should be used, not the word &#8216;localhost&#8217;!</span>

<div>
  <h3>
    Complete website configuration
  </h3>
  
  <p>
    First of all give RoundCube mail a default configuration. This file can be changed later to customize and debug your installation.
  </p>
  
  <p>
    Still in the confg directory type
  </p>
  
  <pre>sudo cp main.inc.php.dist main.inc.php</pre>
  
  <p>
    Find the line that starts with $rcmail_config[&#8216;default_host&#8217;] and alter it so it looks like this:
  </p>
  
  <pre>$rcmail_config['default_host'] = 'tls://127.0.0.1/';</pre>
  
  <p>
    Also alter the authentication type for imap by altering the following line from this:
  </p>
  
  <pre>$rcmail_config['imap_auth_type'] = null;</pre>
  
  <p>
    into this:
  </p>
  
  <pre>$rcmail_config['imap_auth_type'] = CRAM_MD5;</pre>
  
  <h3>
    Create the webapp to start RoundCube
  </h3>
  
  <p>
    The webapp needs an Apache config file and a plist file to work properly. In the plist file there will be a check to see if the Apache config file exists. If it does not exist it will not show up as a webapp in Server.app
  </p>
  
  <p>
    First create the Apache config file.
  </p>
  
  <pre>$ cd /opt/webapps/webmail
$ sudo vi httpd_webmail.conf</pre>
  
  <p>
    and edit this file to contain the following content:
  </p>
  
  <pre>Alias /webmail "/opt/webapps/webmail/"
Alias /Webmail "/opt/webapps/webmail/"
Alias /WebMail "/opt/webapps/webmail/"
Requestheader set x-apple-service-webmail-enabled true
&lt;Directory "/opt/webapps/webmail/"&gt;
  Options -Indexes FollowSymLinks
&lt;/Directory&gt;</pre>
  
  <p>
    Change to the webapps directory:
  </p>
  
  <pre>$ cd /Library/Server/Web/Config/apache2/webapps/</pre>
  
  <p>
    Create a webapp plist file:
  </p>
  
  <pre>$ sudo vi nl.marcoach.webapps.webmail.plist</pre>
  
  <p>
    and edit this file, once again don&#8217;t forget sudo in front of the name of your editor. Make sure it looks like this:
  </p>
  
  <pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt; 
&lt;!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"&gt; 
&lt;plist version="1.0"&gt; 
&lt;dict&gt; 
   &lt;key&gt;name&lt;/key&gt; 
   &lt;string&gt;nl.marcoach.webmail&lt;/string&gt; 
   &lt;key&gt;displayName&lt;/key&gt; &lt;string&gt;"RoundCube webmail&lt;/string&gt; 
   &lt;key&gt;proxies&lt;/key&gt; 
&lt;dict/&gt; 
&lt;key&gt;includeFiles&lt;/key&gt; 
   &lt;array&gt; &lt;string&gt;/opt/webapps/webmail/httpd_webmail.conf&lt;/string&gt; &lt;/array&gt; 
&lt;key&gt;installationIndicatorFilePath&lt;/key&gt; 
   &lt;string&gt;/opt/webapps/webmail/httpd_webmail.conf&lt;/string&gt; 
&lt;/dict&gt; 
&lt;/plist&gt;</pre>
  
  <p>
    With the webapp in place it is time to fire up Server.app and start the webapp:
  </p>
  
  <p>
    Just follow the screenshots
  </p>
  
  <div id="image-block-view-c557487a-ace1-ace1-8a81-d140da7381ad" data-guid="c557487a-ace1-ace1-8a81-d140da7381ad" data-type="image" data-file-guid="314504b3-540e-4115-ac7d-f1a0304831ef" data-file-data-guid="45034eef-8215-4bb4-ad37-10bacd988756">
    <div id="image-block-view-c557487a-ace1-ace1-8a81-d140da7381ad-wrapper">
      <div id="image-block-view-c557487a-ace1-ace1-8a81-d140da7381ad-inner">
        <div>
          <div>
            <div>
              <img id="image-block-view-c557487a-ace1-ace1-8a81-d140da7381ad-container-image-img" alt="" src="https://mac-mini.marcoach.nl/wiki/files/download/45034eef-8215-4bb4-ad37-10bacd988756" data-route-href="" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  
  <p>
    Enable PHP if it is (still) disabled. Next click on Server Website to select it and then click on the pencil:
  </p>
  
  <div id="image-block-view-c5574893-3db2-3db2-8fb2-b5f95a4f946a" data-guid="c5574893-3db2-3db2-8fb2-b5f95a4f946a" data-type="image" data-file-guid="66ad21fe-277a-47a6-ab04-a5c87fa2568c" data-file-data-guid="5dd6d0b2-c9c6-4615-b297-d2b1237bae68">
    <div id="image-block-view-c5574893-3db2-3db2-8fb2-b5f95a4f946a-wrapper">
      <div id="image-block-view-c5574893-3db2-3db2-8fb2-b5f95a4f946a-inner">
        <div>
          <div>
            <div>
              <img id="image-block-view-c5574893-3db2-3db2-8fb2-b5f95a4f946a-container-image-img" alt="" src="https://mac-mini.marcoach.nl/wiki/files/download/5dd6d0b2-c9c6-4615-b297-d2b1237bae68" data-route-href="" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  
  <p>
    Select the &#8216;Edit Advanced Settings&#8230;&#8217;
  </p>
  
  <div id="image-block-view-c55748a1-1e00-1e00-b95f-81b111ba85cc" data-guid="c55748a1-1e00-1e00-b95f-81b111ba85cc" data-type="image" data-file-guid="83ffa240-0fbe-4878-9b5e-51013ce500d1" data-file-data-guid="46e3be5d-5ed5-43ec-bed2-9671ad8732c9">
    <div id="image-block-view-c55748a1-1e00-1e00-b95f-81b111ba85cc-wrapper">
      <div id="image-block-view-c55748a1-1e00-1e00-b95f-81b111ba85cc-inner">
        <div>
          <div>
            <div>
              <img id="image-block-view-c55748a1-1e00-1e00-b95f-81b111ba85cc-container-image-img" alt="" src="https://mac-mini.marcoach.nl/wiki/files/download/46e3be5d-5ed5-43ec-bed2-9671ad8732c9" data-route-href="" />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  
  <p>
    Make sure the checkbox is activated for the newly added webapp, click OK, click done and give it a try
  </p>
</div>

<div id="image-block-view-c55748d0-04c3-04c3-53c0-bf3f0ec1e29e" data-guid="c55748d0-04c3-04c3-53c0-bf3f0ec1e29e" data-type="image" data-file-guid="3c187699-18ff-43af-8c10-203cdf80288d" data-file-data-guid="e8d9d23c-57a9-44e1-bc68-8eb39139be50">
  <div id="image-block-view-c55748d0-04c3-04c3-53c0-bf3f0ec1e29e-wrapper">
    <div id="image-block-view-c55748d0-04c3-04c3-53c0-bf3f0ec1e29e-inner">
      <div>
        <div>
          <div>
            <img id="image-block-view-c55748d0-04c3-04c3-53c0-bf3f0ec1e29e-container-image-img" alt="" src="https://mac-mini.marcoach.nl/wiki/files/download/e8d9d23c-57a9-44e1-bc68-8eb39139be50" data-route-href="" />
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

Log in with your username and password&#8230; Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.