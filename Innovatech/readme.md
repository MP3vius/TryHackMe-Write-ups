# Innovatech Write-up

## Introduction
Welcome to my first official machine on TryHackMe

## Objective
Obtain 2 flags, one located on a user account and the root flag in a more unusual location.

## Tools used
- nmap
- gobuster

## Techniques used
- enumeration
- SQL injection
- Privilege escalation

## Port Scanning
As always we start by doing an nmap scan, which came back with 3 open ports, SSH, HTTP and mongod / mongodb

![nmap scan](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/nmap.png)

## Web Enumeration
Since HTTP is one of the open ports, we start by enumerating the webpage

![gobuster scan](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/gobuster.png)

We can see that there is an admin and files page that both redirect to a login page. This seems like worth checking out!

![contact-page](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/contact.png)

Scanning the found webpages we see a mention of a junior system administrator called Remy, this might be useful information to keep in mind.

![login-page](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/login.png)

The login page looks very basic and therefore likely vulnerable to breach. Default credentials didn't work... However a simple SQL injection like ' OR 1=1-- - did the trick!
 
![todo](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/todo.png)

One of the things we find on the file page on this admin panel is this todo list with some more useful info to keep in mind. Note comment about mongodb!

![credentials](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/remy.png)

We also find a backup file which contains folders of users and SSH keys in them. Now if we still remember the mention of the junior system admin, we should try his account first, because it's more likely we can perform a priv esc through him.

## SSH and privilege escalation
We extract the id_rsa, give it the right permissions 
``chmod 600 id_rsa``
And we use it to login
``ssh -i id_rsa remy@<ip>``

![flag-sudo](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/ssh.png)

``ls -la``
``cat user.txt``

That's flag one found!

Now we can also see remy's bash history and he is using sudo rights on vim, a sudo -l confirms he can use it without a password!

This is how we elevate to root:
``sudo vim -c ':!/bin/sh'

## Root flag
Now that we are root and we see no flag in roots folder, we have to remember the nmap scan and also the todo file with the mention of mongodb and sensitive files.

First let's see what version we are dealing with:

![version](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/version.png)

We can see this is version 3.6.8, which means we have to use the older version of the shell simply called mongo, version 5.0.0 and higher uses a newer shell called mongosh

We look up some more info on how to get into a shell and we read on the configuration page that there is a setting that turns on authorization. We can assume that it's on since we are dealing with sensitive information, however as root we can check the config file and potentially turn it off!

![config](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/config.png)

Navigate to the /etc directory and nano into the config file and make these changes, save and exit out:

![auth](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/auth.png)
![noauth](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/noauth.png)

## MongoDB
For the changes to have an effect, we have to restart the service and wait a few seconds for it to boot up again
``systemctl restart mongodb``

Then we connect simply typing
``mongo``

``help``
Shows us what commands we can use

To list all databases available
``show dbs``

Admin seems the most likely to contain our root flag and so we connect to it using
``use admin``

Then we want to see its contents
``show collections``

And there we see flag and system.version, to cat out the flag we use
``db.flag.find()``

This returns the root flag and marks the machine as complete!

![flag](https://github.com/MP3vius/TryHackMe-Write-ups/blob/main/Innovatech/pictures/flag.png)

## Overview
I hope everyone enjoyed this box and hope the write up helped anyone who needed it, thank you for playing and reading!
