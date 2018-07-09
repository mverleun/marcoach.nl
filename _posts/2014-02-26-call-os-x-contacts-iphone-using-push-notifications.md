---
id: 965
title: Call from OS X Contacts on iPhone using Push Notifications
date: 2014-02-26T17:16:40+00:00
author: Marco Verleun
layout: post
guid: http://www.marcoach.nl/?p=965
permalink: /call-os-x-contacts-iphone-using-push-notifications/
hits:
  - "731"
categories:
  - Tech Tips
---
# Contacts

Have you ever noticed that clicking on the grey label in front of a phone number presents you with a pop-up? One of my favourites is &#8220;Show in large type&#8221; which makes the phone number readable from quite a distance. For quite some time I wanted an entry in there which allows me to make the call from my iPhone. Finally I&#8217;ve got one using a bit of Apple Scripting and an App on my iPhone called Prowl. Now I can lookup an address on my MacBook and have my iPhone make the call using push notifications!

<!--more-->The basics are simple. Using Apple Script I&#8217;ve added a plugin to the address book. This plugin sends a specially crafted URL to my iPhone using Prowl.

## Prowl

First thing to do is to purchase (yes it is not free) Prowl and install it on your iPhone. It can be found [here](https://itunes.apple.com/app/prowl-growl-client/id320876271?mt=8). I use it for other purposes as well, but that is beside the scope.

Setup Prowl using the directions from their site (create an account and configure the App to login using this account) and generate an API key.

The API key will be used in a script to actually send a Push Notification to your phone.

On your iPhone open the Prowl app and change the following settings: turn Ask Before Redirect off and turn Open Links in Safari on if they are set differently.

## Apple Script

Place the following Apple Script in ~/Library/Address Book Plug-ins/ and make sure that you fill in your Prowl API key:

<pre class="lang:default decode:true" title="Call on iPhone.scpt">-- Script to dial a phone number from the address book on an iPhone.
-- (c) 2014 MarCoach

-- Based on an example script from www.mactech.com
-- http://www.mactech.com/articles/mactech/Vol.21/21.10/ScriptingAddressBook/index.html
-- and macsripter.net http://macscripter.net/viewtopic.php?id=35061

-- In order to use this script you have to have Prowl installed on your iPhone
-- Generate a API Key on the Prowl Website and paste it below
property apikey_verified : "" --this is VERY important

using terms from application "Contacts"
	on action property
		return "phone"
	end action property

	on should enable action for thePerson with theEntry
		if theEntry = missing value then
			return false
		else
			return true
		end if
	end should enable action

	on action title for thePerson with theEntry
		return "Call with iPhone"
	end action title

	on perform action for thePerson with theEntry
		set theProwlApiKey to "&lt;Prowl API Key goes here&gt;"
		set thePhoneNumber to value of theEntry

		sendrequest(theProwlApiKey, thePhoneNumber, "Call on iPhone", "Call this number", thePhoneNumber)
	end perform action
end using terms from

on run
	set theProwlApiKey to "&lt;Prowl API Key goes here&gt;" as string
	sendrequest(theProwlApiKey, "+1 555 2222", "Call on iPhone", "Call this number", "+1 555 2222")
end run

on sendrequest(apiKey, phoneNumber, appname, eventname, desc) -- take note of the names of the variables, this tells you what they do
	if apiKey ≠ apikey_verified then
		set i to do shell script "curl --anyauth -G https://api.prowlapp.com/publicapi/verify?apikey=" & apiKey
		if i contains "&lt;success code=\"200\"" then
			set apikey_verified to apiKey
		else if i contains "&lt;error code=\"406\"&gt;" then
			display dialog "Your IP has reached the API limit. Please wait until " & (do shell script "perl -e \"print scalar(gmtime(" & resetdate & ")), \"\\n\"\"" & ".") & ". (once you are whitelisted, you can NOT use this Applescipt-Prowl brigde again)"
		else if i contains "&lt;error code=\"500\"&gt;" then
			display dialog "Internal Server Error. Please try again later."
		else
			display dialog "The API key that was used was not valid. Please try again with another API key, or check to make sure your API key works." buttons {"OK"} default button 1
			return
		end if
	end if

	set cleanNumber to ""
	repeat with a from 1 to length of phoneNumber
		set theCharacter to character a of phoneNumber
		if "0123456789" contains theCharacter then set cleanNumber to cleanNumber & theCharacter
		if "+" contains the theCharacter then set cleanNumber to cleanNumber & "00"
	end repeat

	set theURL to " --data-urlencode url=tel://\"" & cleanNumber & "\""
	do shell script "curl  --anyauth --data-urlencode apikey=\"" & apiKey & "\" " & theURL & " --data-urlencode application=\"" & appname & "\" --data-urlencode event=\"" & eventname & "\" --data-urlencode description=\"" & desc & "\" https://api.prowlapp.com/publicapi/add"

end sendrequest</pre> Disclaimer: All information posted is written with the upmost care and valid at the time of writing. Changes in versions can supersede the provided information. Please use your own judgement.