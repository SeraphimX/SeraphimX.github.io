---
layout: post
title: LazySysAdmin 1 Vulnhub Walkthrough
tags: pentesting
---
Hi again!

LazySysAdmin 1 is a boot2root vulnerable virtual machine available from https://www.vulnhub.com/entry/lazysysadmin-1,205/
It is built to teach us more about the importance of enumeration. The root dance you'll get after thorough, successful enumeration is real :)

When I first learnt that enumeration is inevitable in the pentesting industry, I was initially disgrunted. My impression of cyber attackers were high-precision, targeted attacks specially crafted with minimal intrusion. Enumeration is quite the opposite- to try and try (and try some more) until you find a way in. This probably delayed my entry into the field as it is, and anyway, in a pentest, you should be given a scope to limit yourself to anyway. The OSCP was full of
enumeration, and this LazySysAdmin box is a great introduction to enumeration for anyone

Here we go!


The first thing we want to do after discovering the ip address (via netdiscover), is to scan for open ports and services running on the victim machine. We use Sparta, which launches nmap and nikto scans for us, among a few other types.

<img src="{{ site.url }}/images/lazysysadmin/recon.PNG" >

As you can see, Sparta even launched a Nikto scan for us! We discover a robots.txt, and it's always worth checking out for hidden secrets

<img src="{{ site.url }}/images/lazysysadmin/recon2.PNG" >

Here's the front page on our browser

<img src="{{ site.url }}/images/lazysysadmin/frontpage.PNG" >

And here's the robots.txt, manually inspected

<img src="{{ site.url }}/images/lazysysadmin/frontpage2.PNG" >


Following these URLs led to nothing much being found. As a matter of fact, there was nothing much in the front page as well. No links to follow, no buttons to click. I took a (small) step back, and decided to Dirb the root URL for more web paths.

<img src="{{ site.url }}/images/lazysysadmin/dirb.PNG" >

Dirb brute-forces a webpage, and we can find hidden pages not linked from the root URL. It is "only as good as your wordlist" though, so if you don't find anything on the first Dirb, and have tried everything, you might want to consider bigger wordlists.
Here's the output from the Dirb

<img src="{{ site.url }}/images/lazysysadmin/dirb2.PNG" >

Phpmyadmin is interesting. It, among with its varient brothers like phpliteadmin, is an admin platform for managing databases like mysql or mssql running on the server. If you compromise the phpmyadmin portal, chances are high that you have gotten a foot in the door.

Unfortunately, the default credentials don't work much (Yes, do ALWAYS check for default creds on any application)

<img src="{{ site.url }}/images/lazysysadmin/failed1.PNG" >

Fortunately, Dirb did come up with more vectors to look at. We see a possible wordpress application running in either the /wordpress/ or /wp/ directory. Let's check it out!

<img src="{{ site.url }}/images/lazysysadmin/dirb3.PNG" >

Bingo! Something that looks a lot better than the initial web page we had earlier. We see a post of someone claiming to be togie (which might come in useful later).

Fun to see is also a comment by the Admin:

<img src="{{ site.url }}/images/lazysysadmin/dirb4.PNG" >

Dirb had also traversed found URL directories, and found for us some interesting paths, such as an admin portal:

<img src="{{ site.url }}/images/lazysysadmin/dirb5.PNG" >

<img src="{{ site.url }}/images/lazysysadmin/dirb6.PNG" >

Trying to log into the "Admin" account with "togie", "password", or whatever off the top of my head did not work. We can actually try to brute force these portals (and the phpmyadmin one), but we're not that desperate yet.



We have now kinda thoroughly inspected the web applications, and found nothing much yet. Thankfully, the initial Nmap scan pointed out more than just a web-server running on the victim machine

<img src="{{ site.url }}/images/lazysysadmin/recon.PNG" >

The SMB service on ports 139/445 is used to share files to other machines across the network. Along with it comes great potential to enumerate, to the extent where people created great scripts just to do so. 
I used enum4linux (tee just redirects output from the enum4linux application to both the screen and a file specified, so I can look it up further in the future)


<img src="{{ site.url }}/images/lazysysadmin/enum4linux.PNG" >

We found several shares up and running on the victim's server

<img src="{{ site.url }}/images/lazysysadmin/enum4linux2.PNG" >

And we also found a user called togie on the server! The wordpress's random post does not seem totally random now.

<img src="{{ site.url }}/images/lazysysadmin/enum4linux3.PNG" >

We can now do quite a few things to try out:
 - brute force togie's account on ssh 
 - brute force accounts on phpmyadmin and the wordpress-login
 - enumerate the SMB further
 - Take a few more steps back and look at the nmap scan for more hints

Since we're already on the SMB trail, I chose to do that. You can launch brute-forces in the background, but you run the risk of alerting Intrusion Detection Systems, and even locking out those accounts.


We try to access the SMB share on our attacker's machine, using smbclient

<img src="{{ site.url }}/images/lazysysadmin/smbclient.PNG" >

It works. We seem to have access to the www folder. 

<img src="{{ site.url }}/images/lazysysadmin/smbclient2.PNG" >

Enumerating at this stage takes the form of digging through interesting files and folders for any possible hints, such as user-created text files and scripts. Who knows, they might contain passwords!

<img src="{{ site.url }}/images/lazysysadmin/smbclient3.PNG" >

As it turns out, they do! the Deets.txt file in the smb share contains "12345".

<img src="{{ site.url }}/images/lazysysadmin/smbclient4.PNG" >

Enumerating a bit more yielded little, so we move on. 

We have some sort of password, some sort of username, and quite a few login platforms to try. The only logical thing to do follows.

Phpmyadmin and wordpress login yielded nothing. Ssh into "togie", however, was what we're looking for- a limited shell into the system

<img src="{{ site.url }}/images/lazysysadmin/ssh.PNG" >


Enumeration (yes, again) at this stage is like a fresh new book. With limited shell, your task is now to escalate privileges to the most powerful account on the system. For Linux, that is root. There are a couple of ways to do this, and all of it involves getting as much information as you can about the system

This includes the current user's permission. We can view your current permissions by doing "sudo -l"

<img src="{{ site.url }}/images/lazysysadmin/ssh2.PNG" >

(ALL : ALL) ALL means one very simple thing. Togie can execute, as sudo, ALL commands. Dropping into the root command is then trivial, with "sudo su"

Congradulations, you now have root!

<img src="{{ site.url }}/images/lazysysadmin/ssh3.PNG" >



--SeraphimX
