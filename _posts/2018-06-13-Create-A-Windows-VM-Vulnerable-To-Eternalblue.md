---
layout: post
title: Create a Windows 7 VM Vulnerable to MS17-010 EternalBlue
tags: pentesting
---
Recently in one of the CTF challenges conducted in my school's Security module, my team decided to build up a challenge involving MS17-010. We wanted to create a Windows Virtual Machine that is vulnerable to the MS17-010 EternalBlue exploit.

The steps involving the creation of the VM are provided below. Take note that this VM should be used for educational and research purposes only. Also, Windows 10 prevents the trivial selection and disabling of specific updates to be installed, so we will be using Windows 7 instead. 

The basic process involves preventing the automatic update from patching the Windows VM, and setting up the environment for SMB to be active.
 
<b>Steps to create the vulnerable VM </b>

1) Download the IE8 on Win7 x86 (Virtualbox) and extract it. You can download the zipped OVA file here: https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/

<img src="{{ site.url }}/images/2018-06-13/downloadVM.PNG" >

2) Launch Virtualbox, and import the downloaded VM into Virtualbox using “File > import appliance”

<img src="{{ site.url }}/images/2018-06-13/virtualboxImport.PNG" >

<img src="{{ site.url }}/images/2018-06-13/virtualboxImport2.PNG" >

Select the unzipped ova file, hit Next, and import.


3) Before you start the VM, we want to prevent the VM from automatically update itself. Go to the VM Settings and disable the NAT connection, by unattaching the network adapter.

<img src="{{ site.url }}/images/2018-06-13/virtualboxImport5.PNG" >


4) Boot up the VM. When you reach the Desktop, proceed to change the Windows Update setting to “check for updates but let me choose when to download and install them”

Do a simple search for windows update settings




5) Re-enable the NAT adapter

6) Install every important update except for those from Mar 2017 onwards (https://stackoverflow.com/questions/47531732/how-to-make-a-win7-pc-vulnerable-to-ms17-010-eternalblue)

7) Disable the Nat adapter, and enable the Host-only adapter (with DHCP client on Virtualbox)

8) Enable Windows Firewall inbound traffic for File and Printer Sharing (SMB-In) (Private and Domain) (and also Echo Request-ICMPv4-In Private and Domain for ping)

9) Change the Default username and password for IEUSER to be something not brute-forcible

10) Set Task Scheduler to launch explorer.exe with trigger as system startup, and without logon

11) Enable password on boot (https://www.sevenforums.com/tutorials/377-log-automatically-startup.html)

12) Take a snapshot, export the appliance


Test the MS17-010 on it! Try https://github.com/ElevenPaths/Eternalblue-Doublepulsar-Metasploit or something similar.

--SeraphimX
