---
title: "USB Recon Tool: 1st year project"
date: 2021-08-06T16:39:46+01:00
draft: false
---

-AEC- ALL Notes:

# Contents

- [Recon](#Reconnaisance)
	- [Useful information](#Useful_Information)
	- [Purpose of information](#Purpose_of_info)
- [Communication of Data](#Communication_of_data)
	- [HTTP](#HTTP)
	- [FTP](#FTP)
		- [Server](#Server)
			- [FTP Server Types](#FTP_Server_types)
			- [Configuration](#Configuration)
				- [Update Linux](#Update_Linux)
				- [Install and Start VSFTPD](#Install_and_Start_VSFTPD)
				- [Configuring VSFTPD](#Configuring_VSFTPD)
					- [Anon FTP](#Anonymous_FTP)
				- [Inotify and Linux services](#Inotify_and_Linux_services)
			- Commands (Status)
			- [Server-side permissions](#Server-side_permissions)
				- [chroot](#chroot)
		- [Client (winSCP)](#Client_(winSCP))
	- [Dealing with the Data](#Dealing_with_the_Data)
- [Windows Structure](#Windows_Structure)
	- [File System](#File_System)
	- Log Files
	- Comparison to Linux
	- [Autorun Feature](#AutoRun_Feature)
	- (EXT) Privelege escalation
	- [XP](#XP)
		- Service packs
		- Why did we choose this?	
- Covering tracks
	- Concept
	- Within Windows
	- Forensic explanations	
- (EXT) Website
	- Design
	- Implementations


---------------------------------------------
# Reconnaisance

For this project we have decided to create a reconnaisance tool that will enumerate the information from a windows
(XP) machine. There are two things that immediately come to mind when thinking the information gathered from reconnaisance.
What information is actually useful to us, and what is the purpose of the information. They do come hand in hand.

## Useful_Information

What is useful information? What is information? What information is within our scope/reach?

Within any system there will always be information that is used by the machine, whether this is to keep track
of hardware, software, versions, running processes etc. This information can be used to manipulate, access and steal from
the target machine/network.

In windows there is a useful tool known as the 'Event Viewer'. This is where any system events are logged.
> Use this to learn more about event viewer: https://www.howtogeek.com/123646/htg-explains-what-the-windows-event-viewer-is-and-how-you-can-use-it/

Another primary area to look on a windows machine is directly through the command line. There are many commands that
can be used to pull up system info. Alot of the time this doesnt even need admin privelege to do this.

> The terminal below shows an example of running the 'systeminfo' command.

~~~term
C:\Users\cueh>systeminfo

Host Name:                 EH_7-2
OS Name:                   Microsoft Windows 10 Pro Education
OS Version:                10.0.18362 N/A Build 18362
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   cueh
Product ID:                00380-00000-00001-AA804
Original Install Date:     07/03/2020, 01:51:36
System Boot Time:          07/03/2020, 01:50:11
System Manufacturer:       HP
System Model:              HP Z240 Tower Workstation
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 94 Stepping 3 GenuineIntel ~3201 Mhz
BIOS Version:              HP N51 Ver. 01.68, 04/05/2018
Windows Directory:         C:\windows
System Directory:          C:\windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-gb;English (United Kingdom)
Input Locale:              en-gb;English (United Kingdom)
Time Zone:                 (UTC+00:00) Dublin, Edinburgh, Lisbon, London
Total Physical Memory:     8,113 MB
Available Physical Memory: 4,861 MB
Virtual Memory: Max Size:  10,033 MB
Virtual Memory: Available: 5,029 MB
Virtual Memory: In Use:    5,004 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              \\EH_7-2
Hotfix(s):                 5 Hotfix(s) Installed.
                           [01]: KB4511555
                           [02]: KB4497727
                           [03]: KB4503308
                           [04]: KB4515530
                           [05]: KB4512941
Network Card(s):           5 NIC(s) Installed.
                           [01]: Intel(R) Ethernet I210-T1 GbE NIC
                                 Connection Name: Ethernet
                                 DHCP Enabled:    Yes
                                 DHCP Server:     10.0.128.252
                                 IP address(es)
                                 [01]: 10.0.128.40
                                 [02]: fe80::d936:e5c2:94d7:b31a
                           [02]: Intel(R) Ethernet Connection (2) I219-LM
                                 Connection Name: Ethernet 2
                                 DHCP Enabled:    Yes
                                 DHCP Server:     192.168.1.2
                                 IP address(es)
                                 [01]: 192.168.3.118
                                 [02]: fe80::81d2:1d00:c480:e830
                           [03]: Microsoft KM-TEST Loopback Adapter
                                 Connection Name: Ethernet 4
                                 DHCP Enabled:    Yes
                                 DHCP Server:     255.255.255.255
                                 IP address(es)
                                 [01]: 169.254.99.50
                                 [02]: fe80::601e:1e0f:eea0:6332
                           [04]: VMware Virtual Ethernet Adapter for VMnet1
                                 Connection Name: Ethernet 3
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 192.168.101.1
                                 [02]: fe80::8134:5f8a:8a1:f3dd
                           [05]: VMware Virtual Ethernet Adapter for VMnet8
                                 Connection Name: Ethernet 5
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 192.168.162.1
                                 [02]: fe80::18d8:d3e2:424a:2bd0
Hyper-V Requirements:      VM Monitor Mode Extensions: Yes
                           Virtualization Enabled In Firmware: Yes
                           Second Level Address Translation: Yes
                           Data Execution Prevention Available: Yes
~~~

> This is one command of many, the output of all our currently planned commands is 8000 lines. This needs to be sorted into different areas.

There are many more commands obviously, these fall under mainly system and network commands.
> More information on enumerating windows machines can be found here: http://www.defenceindepth.net/2009/10/enumerating-windows-information.html

## Purpose of info

The purpose of enumerating information is that we will use it to establish an 'active connection' to the target
host to discover potential attack vectors within the system

Enumeration is used to gather the below:

- Usernames, Group names
- Hostnames
- Network shares and services
- IP tables and routing tables
- Service settings and Audit configurations
- Application and banners
- SNMP and DNS Details
-----------------------------------
# Communication_of_data:

When it comes to communicating the enumerated information (we need to turn it into data) there are many methods within
the capabilites of a windows machine. The main ones that we will look at are HTTP, FTP and the variations that come under
them.

## HTTP:

HTTP (Hypertext Transfer Protocol) is used to structure requests and responses over the internet. HTTP requires data to be
transferred from one point to another using the network capabilites of the protocol.

When HTTP performs the transfer it used TCP (Transmission control protocol). This protocol is used to manage the channels
between the browser and the HTTP server being used. 

> (I would recommend learning more about TCP) - https://searchnetworking.techtarget.com/definition/TCP

The main methods used by HTTP are POST, PUT and GET. There are more but I wont talk about them here.

- GET:
    - One of the most common HTTP methods
    - Remain in browser history
    - Can be cached
    - Can be bookmarked
    - Should not be used for dealing with sensetive data
    - Only used to request data (not modify)

- POST:
    - Also common
    - Never cached
    - Cant be bookmarked

- PUT:
    - Used to send data to a server to create/update a resource
    - Difference from POST is that calling PUT always produces the same result
    - POST will have side effects of creating multiple resourses

## FTP

FTP (File Transfer Protocol)

For this task we are going to need to set up a FTP server, this will be used to store the files and data that we are collecting from the target machine.
We're using a beagle bone black ([specs](https://beagleboard.org/black "Beagle Bone Black Specs")) to host our FTP server.

FTP has many useful features and functions we can use to access the files from within the machine.
(Provided that we have succussfully implemented autorun then we can):
- Run the scripts
	- This will gather the data
	- Enable FTP
	- Send the data back to our server
	- Clear any tracks and log files
	
Some of the commands that can be used within FTP are:

- Most of what you can use from LINUX. However I have found restrictions within these.
- get : Retrieve file and store it on local machine
- put : store a local file on the remote machine
- (TO EXIT FTP: bye) Literally just bye.

> Look into these commands more.

### FTP_Server_types

#### ProFTPd
The most feature rich FTP version currently available to us. All large control panels support ProFTPd.
This is part of the reason we haven't chosen to run ProFTPd on our server. It has the most known vulnerabilities of all
the options we discussed and considered due to it's widespread use.

#### PureFTPd
The more security focused FTP solution with the lowest number of vulnerabilities found out of the three systems. Configuration of
PureFTPd is simple as it contains a no-config file option to run out of the box.

#### Vsftpd
Very Secure FTP is the system we will be running on the beagle bone to capture target data. We have come to this conclusion as
vsftpd is a good middle ground between ProFTPd and PureFTPd in terms of both security and features. As vsftpd is also a widely used FTP server it has
its fair share of known vulnerabilities but with fewer than ProFTPd and more than PureFTPd. We believe that the extra security risk in vsftpd 
is outweighed by its ability to scale efficiently, future proofing the project.

### Configuration

#### Update_Linux
~~~term
root@kali:~# sudo apt-get update && sudo apt-get upgrade
~~~
> Using this command will install available upgrades of all the packages installed on the system.

#### Install_and_Start_VSFTPD
Once you have got the upgrades, install VSFTPD, this is the FTP daemon we will use in this project. 

~~~term
root@kali:~# sudo apt-get install vsftpd
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  gir1.2-clutter-gst-3.0 gir1.2-gtkclutter-1.0 libayatana-ido3-0.4-0 libdumbnet1 libgssglue1
  libtagc0 libxdot4 python-asn1crypto
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  vsftpd
0 upgraded, 1 newly installed, 0 to remove and 496 not upgraded.
Need to get 0 B/153 kB of archives.
After this operation, 358 kB of additional disk space will be used.
Preconfiguring packages ...
Selecting previously unselected package vsftpd.
(Reading database ... 356802 files and directories currently installed.)
Preparing to unpack .../vsftpd_3.0.3-12+b1_amd64.deb ...
Unpacking vsftpd (3.0.3-12+b1) ...
Setting up vsftpd (3.0.3-12+b1) ...
vsftpd.conf:1: Line references path below legacy directory /var/run/, updating /var/run/vsftpd/empty → /run/vsftpd/empty; please update the tmpfiles.d/ drop-in file accordingly.
Processing triggers for man-db (2.9.0-2) ...
Processing triggers for systemd (244.3-1) ...
~~~
> Install log for VSFTPD

Now that we have installed the appropriate packages, we can begin to look at the service and configuration.

~~~term
root@kali:~# service vsftpd status
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; disabled; vendor preset: disabled)
     Active: inactive (dead)
~~~
> This code "service vsftpd status". As you can see, the server is inactive

We can then turn the server on, and check the status again, so we know that it is done:

~~~term
root@kali:~# service vsftpd start
root@kali:~# service vsftpd status
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; disabled; vendor preset: disabled)
     Active: active (running) since Sun 2020-03-01 07:07:51 EST; 2s ago
    Process: 2277 ExecStartPre=/bin/mkdir -p /var/run/vsftpd/empty (code=exited, status=0/SUCCESS)
   Main PID: 2278 (vsftpd)
      Tasks: 1 (limit: 7140)
     Memory: 868.0K
     CGroup: /system.slice/vsftpd.service
             └─2278 /usr/sbin/vsftpd /etc/vsftpd.conf

Mar 01 07:07:51 kali systemd[1]: Starting vsftpd FTP server...
Mar 01 07:07:51 kali systemd[1]: Started vsftpd FTP server
~~~
> Using the <service> [name] <arguement> with start and status, we achieve the server online.

#### Configuring_VSFTPD
Now we must begin to configure the server. We need to move into the /etc/ directory, this is where the
configuration file is. We need to then use vim or nano to open the vsftpd.conf.

~~~term
root@kali:/etc# ls -la | grep vsftpd.conf 
-rw-r--r--   1 root     root       5950 Feb 29 12:04 vsftpd.conf
~~~

Once opened there are many settings we need to look at.

https://www.wikihow.com/Set-up-an-FTP-Server-in-Ubuntu-Linux
>Settings within VSFTPD.conf:

##### Anonymous_FTP:

In order to use FTP anonymously then you can activate it within the configuration file by uncommenting the line.

In this project annonymous FTP will be disabled as it is very insecure.


#### Inotify_and_Linux_services

To organise the files we recieve from target machines automatically the best option we have found is Inotify-tools. This is a set of command line programs desgined to interface with Inotify. Inotify is a linux kernal subsystem that recognises changes to a file system and reports that data back to applications.

~~~bash
    inotifywait -m home/debian/ -e create -e moved_to | # -m (monitor mode) means the script will not end after it has been called once. -e (event) watching for creation and movement of directories.
        while read path action file; do # Captures output of inotify command and assigns each line to file.
            if [[ "$file" =~ .*txt$ ]]; then # Does the file end with .txt?
                python3 /home/debian/strip.py # Run python script
            fi
        done
~~~
> This is the bash script we place in /usr/bin/
> Set executable perms with chmod +x

We need this script to run constantly, our options are to either manually run the script on every boot or create a custom systemd service to run it on every boot.
To create a new service we need a unit file to give the system information about it.

~~~unicode
[Unit]
Description=A service to watch ftp destination folder.

[Service]
Type=simple
ExecStart=/bin/bash /usr/bin/watch_ftp.sh

[Install]
WantedBy=multi-user.target
~~~
>Unit file places in /etc/systemd/system/
>Set perms with chmod 644

Now we can interact with it as any other linux service.

~~~term
root@beaglebone:/home/debian# systemctl enable watch_ftp.service
root@beaglebone:/home/debian# systemctl start watch_ftp.service
root@beaglebone:/home/debian# systemctl status watch_ftp.service
● watch_ftp.service
   Loaded: loaded (/etc/systemd/system/watch_ftp.service; disabled; vendor preset: enabled)
   Active: active (running) since Sun 2020-03-15 11:19:37 UTC; 13min ago
 Main PID: 1547 (bash)
    Tasks: 3 (limit: 4915)
   CGroup: /system.slice/watch_ftp.service
           ├─1547 /bin/bash /usr/bin/watch_ftp.sh
           ├─1548 inotifywait -m home/debian/ -e create -e moved_to
           └─1549 /bin/bash /usr/bin/watch_ftp.sh

Mar 15 11:19:37 beaglebone systemd[1]: Started watch_ftp.service.
Mar 15 11:19:38 beaglebone bash[1547]: Setting up watches.
Mar 15 11:19:38 beaglebone bash[1547]: Watches established.
~~~
> Status of the service shows all watched are running.

> Enable allows the service to start on every boot.

An FTP transfer is then initiated from the target machine and we check the status again.
~~~term
root@beaglebone:/home/debian# systemctl status watch_ftp.service
● watch_ftp.service
   Loaded: loaded (/etc/systemd/system/watch_ftp.service; disabled; vendor preset: enabled)
   Active: active (running) since Sun 2020-03-15 11:19:37 UTC; 17min ago
 Main PID: 1547 (bash)
    Tasks: 3 (limit: 4915)
   CGroup: /system.slice/watch_ftp.service
           ├─1547 /bin/bash /usr/bin/watch_ftp.sh
           ├─1548 inotifywait -m home/debian/ -e create -e moved_to
           └─1549 /bin/bash /usr/bin/watch_ftp.sh

Mar 15 11:19:37 beaglebone systemd[1]: Started watch_ftp.service.
Mar 15 11:19:38 beaglebone bash[1547]: Setting up watches.
Mar 15 11:19:38 beaglebone bash[1547]: Watches established.
Mar 15 11:36:02 beaglebone bash[1547]: ftp transfer recieved
~~~
As we can see the service is working correctly and identifying new files on the system.



##### Chroot

Chroot is a command in linux which changes the apparent root directory for a user. This prevents any user establishing an FTP connection and then having full access to the system file system. 

On any new connection to the server, the user is immediately chrooted into their home directory, now no matter what they do all they can see of the ftp servers file system is their home directory.



## Client_(WinSCP)

Ideally we wouldn't be using a 3rd party FTP client to send data, this is because it bulks out the usb payload and makes the process less streamlined. However Windows XP SP3 is unable to send the text file to the server as the built in FTP client cannot access passive mode with the server. This functionality was introduced in a Windows 8 patch.

That is why we have to use WinSCP to send the data. It is the best choice of the 3rd party options in our case. This is because it is available in a portable version so we can run it directly from a USB, it has a command line version of the executable (winSCP.com) that can take a script as an argument. All of this lets us 100% automatically send our collected information back to the server.

## Dealing_with_the_Data:

When we send the data back to the server its in a 277 line txt document called "Information.txt". This is fairly useless as we have no idea which machine its for or what it contains.

This is where strip.py comes in, its the script that is triggered whenever an FTP transfer is detected by our custom service.

~~~python
import os
import datetime

now = datetime.datetime.now()
dt_string = now.strftime("%d%m%Y%H%M%S") # Format datetime into string in format: DMYHMS
file_names = ["OS_Info", "System_Info",  "IP_Info", "Firewall_Info", "Connection_Info", "Task_and_Service_Info" ]

with open("/home/debian/Information.txt", 'r') as f:
    lines = f.readlines() # Creates list of strings, each line of file is a value in the list.

ipline = lines[6] # Stores line in file with IP address 
ipaddr = ipline[44:59] # Strips string down to just IP address

path = f"/home/debian/{dt_string}_{ipaddr}/" # Creates variable with custom file path: (Current time and date)_(IP of target) 

# Breaks down Information.txt into different chuncks by line numbers.
block1 = lines[8:18]
block2 = lines[19:30]
block3 = lines[31:87]
block4 = lines[88:144]
block5 = lines[145:184]
block6 = lines[185:]

blocks = [block1, block2, block3, block4, block5, block6]

os.mkdir(path) # New directory made in FTP directory with custom name.

for n,b in zip(file_names, blocks): # Iterates through blocks and file_names simultaniously. Zip meants the operation will stop when the smallest list is complete.
        with open(f"{path}/{n}.txt", 'w') as d: # Opens file specified in file_names
            for i in b:
                d.write(i) # Writes the data from the corresponding block into its file.

os.remove("/home/debian/Information.txt") # Delete master Information file 

~~~
> Script was originally written in c++ but couldnt get inotify service to run the compiled file. Python files ran straight away so we've use that.


This script creates 6 files populated with subsections of the Information.txt file, in a directory with a name compromised of the targets systems IP address and the time stamp of the transfer.

For example: 15032020161227_192.168.243.131
Is a folder full of information about a system with IP 192.168.243.131, taken on 15/03/2020 @ 16:12:27

--------------------------
# Windows_Structure

As we are looking at windows for this project we should make sure that we understand how windows is structured. I am 
referring to things like (File system, logs, XP Service packs) aswell as comparing to what we already know about the Linux
system. 

> My research involves heavy references to XP however most of the content is the same for all versions, i will try to mention when something is different between OS Versions.

## File_system

Windows XP Professional supports the three major computer file systems of File Allocation Table (FAT/FAT16, FAT32, NTFS).

FAT is allocatted in clusters, the size of these clusters is determined by the size of the partition. The larger the partition, the larger the cluster size. The larger cluster size, the more space is required when using it to store data.

> A good explanation is here: https://www.mcmcse.com/microsoft/guides/filesystems.shtml OR this video: https://www.youtube.com/watch?v=HjVktRd35G8

## Log_Files

For the project we should look at how the log files and system events are recorded. This will heavily incorporate the forensics module we have been doing. The main areas we should look into are:

- Registry Files
      - USB
      - Command line
      - System

> Could we manage to take forensic images of the target machine before and after.

As known from forensics, we wont actually be able to see much of the file systems heirarchy, including the important registry files like the SAM file. If we were to take a forensic image and use the Lab software to view it, then we can analyse the effects within the system of our malware.

I spoke to the lecturer Russ and he gave some advice and pointers as to the places that may be affected by our USB.

Sys32 APISetup.log                              : 1st time plug-in, contains drivers for setup 
Control set in System Registry                  : ???
Registry Editor /HKEY_LOCAL/.../Enum/USBSTOR    : All USB devices that have been in the system
USB Plug-in                                     : System event log ID 20001??
Logs have been auto-off on XP                   : He mentioned this, could be interesting
Command line log                                : Where is it? We are running commands

## Comparison_to_Linux

## Autorun_feature:

One major factor in our project is the autorun feature that was introduced in Windows 95. It was used to ease application installation for non-technical users, basically to help people who are computer illiterate. These people would call the support services provided by microsoft so this feature helped to reduce the cost of these calls.

Autorun.inf must be included within the the root directory of a volume in order to be viewed. It must be at the "front" of the USB/CD.

We are going to be using this feature in order to run our scripts.
Here is an implementation of it:

~~~term
[autorun]
;Open=info_gather.bat
ShellExecute=info_gather.bat
UseAutoPlay=1
~~~

## XP

XP was introduced in 2001. This included two versions XP Home and XP Professional. The main focus for these was mobility, such as technology allowed for in 2001, and included plug and play features for connecting to wireless networks. This operating system utilizes the 802.11x wireless security standard.

https://www.webopedia.com/TERM/W/Windows_XP.html

### Service Packs

What is a service pack? To explain this you must understand that windows will regularly bring out updates, bug fixes and enhancements for the operating system of for a software program. THese updates are important in maintaining the security of the systems using the OS (XP). 
Once many of these have been bought out,Microsoft will gather all the updates, aswell as adding more into a batch of updates. This is a service pack.

http://www.compukiss.com/tutorials/service-packs-explained-and-needed.html

### Why did we choose XP?

We have decided to go for XP machines in our project. There are many reasons for this but the main ones are:

- It is still commonly in use
	- Runs well older and less powerful hardware
	- Runs older legacy apps that people still use
	- Many govenment and private offices in developing countries use it
	- Upgrading larger amounts of machines from big businesses isnt always possible
	- Some PC's dont meet the system requirements of later versions of windows.
	- Hospitals may still use this software (This is a basic example but you get the idea)
- AutoRun Feature
	- This is a big part of our project goal

https://www.windowslatest.com/2018/04/04/windows-xp-is-still-going-strong/
https://www.quora.com/Why-do-people-still-use-Windows-XP

---------------------------

# Covering tracks

As we are building a reconaissance app we should make sure that we look into covering our tracks, meaning being stealthy/undetectable forensically. There are many things that come into play when talking about covering tracks, we will look into the main ones for windows.
https://resources.infosecinstitute.com/covering-tracks-of-attacks/#gref
https://resources.infosecinstitute.com/penetration-testing-covering-tracks/#gref
