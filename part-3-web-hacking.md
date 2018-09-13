---
description: >-
  Web hacking is essential hacker knowledge. You are already and will be exposed
  to a ton of different web services, and a lot of them can be exploited.
---

# Part 3 - Web hacking

## What do you learn in this part?

* General web concepts
* Web vulnerabilities
* Web hacking tools
* Hacking some more boxes!

**Boxes that are suitable for this part**

* CronOS - 10.10.10.13
* Shocker - 10.10.10.56
* Bonus - Beep, Europa, Bank, Tenten, Nineveh

## **General web concepts**

Before we start hacking the web, we need to know some basics about how the web works. A web service consists of a few different components.

* **Web server** - the server that handles HTTP requests
* **Web service** - is an application run by a web server, performing tasks and returning structured data to a calling program, rather than just static content.
* **Content Management System \(CMS\)** - front end for serving content. Wordpress and Drupal are examples of CMSs.
* **Web application** - Basicalwhat you see on the screen

## Web vulnerabilities

#### OWASP Top 10 

When you start your hacking journey you will seen hear talk about [The Open Web Application Security Project \(OWASP\) Top Ten](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_2017_Project). This is an anually reviewed overview of web application security risks worldwide. The most recent update is from 2017. Here are some important vulnerabilities you will see on HTB:

* [Injection](https://www.owasp.org/index.php/Top_10-2017_A1-Injection)
* [Broken access control](https://www.owasp.org/index.php/Broken_Access_Control)
* [Security misconfiguration](https://www.owasp.org/index.php/Top_10-2017_A6-Security_Misconfiguration)
* [Using components with known vulnerabilities](https://www.owasp.org/index.php/Top_10-2017_A9-Using_Components_with_Known_Vulnerabilities)

**SQL Injections**

These are some examples of quite simple SQL injections. Put this in username and/or password field, and see what happens. Maybe you get lucky!

* `admin' or '1'='1`
* `admin'-- - " or ""="`
* `' or 1=1 -- - '`
* `union select 1,2,3 -- -`

**File inclusions**

A file inclusion is when we are able to access arbitrary files on the file system, through the web server.

* Local file inclusion \(LFI\) - read files on the remote file system
* Remote file inclsuion \(RFI\) - upload files to the file system 

LFI happens often in PHP. Example: [`http://10.10.10.100/file.php?path=../../etc/passwd`](http://10.10.10.100/file.php?path=../../etc/passwd)

Because the `path` parameter in the`file.php` code is not properly handled, we are now able to read arbitrary files on the filesystem, for example the passwd file whic contains usernames.

RFI gives you LFI, but LFI doesn’t necessarily give you RFI. If you think you have found an LFI you can try to verify it with [fimap](https://tools.kali.org/web-applications/fimap).

 `fimap -u http://ip/file.php?path=..`  

But what files do we look for if we find a file inclusion error? Files containing usernames, passwords, source code, PHP files, perhaps even [SSH keys](http://blakesmith.me/2010/02/08/understanding-public-key-private-key-concepts.html). But we must know the file name and path! Therefore, learning enough about the file system to know [what files to look for](https://digi.ninja/blog/when_all_you_can_do_is_read.php) and where they are usually located is crucial.

### **Domain Name System \(DNS\)**

#### **Adding a DNS entry**

For some boxes on HTB, we have to manually add a DNS entry. What does this mean? When you enter a url into your web browser, DNS is the technology that translates the text address into an IP address for you. This is so we don't have to go around and remember IP addresses for websites we like to visit. Now, we can manually specify DNS entries, that means we say that, well we like this address, e.g. `lol.htb` to translate to `10.10.10.10`. We do this in Linix, by opening up the file called /etc/hosts in a text editor. Add an entry in a new line like this: `10.10.10.10 lol.thb` and save the file. Yes, just like that and nothing more. You can then navigate to lol.htb in your web browser and it will automatically translate `lol.htb` to the IP address `10.10.10.10.` 

The screenshot shows the `/etc/hosts` file opened in the vim editor in the terminal. As you can see, a DNS entry for `lol.htb` has been added to the file. This is a fictional machine and IP for the purpose of this example.

![](.gitbook/assets/image%20%2814%29.png)

#### Zone transfer attack

DNS Zone transfer is the process where a DNS server passes a copy of part of it's database \(which is called a "zone"\) to another DNS server. It basically allows us to make the server reveal some information about itself. We can exploit this to learn more about the system, and perhaps discover URLs to investigate.

To dig up some more info about the domain, run this DNS query. This will of course only work on machines that have exposed DNS as a service externally. That basically means if you see that port 53 \(DNS\) is open, you can try this. Note that it might not always work, and you have to have a DNS entry added so your computer knows which DNS server to send the request to.

`dig axfr <url> @<server>`

![](https://lh3.googleusercontent.com/32Uh7W3TrxMLpStYF19GXML61UU-qFLdKACaqfy_Za43mn-jGnoZs7VU0Fh_8KLCg0Rmk7aHf5tD3XkgWlAcZPPuD_ewG9AlveqrLNnlWU3AcbUjmIMSypaRVDYimUq2Rwhl_rO4SDg)

## **Web hacking tools**

### Curling

`curl -i <url>` - Downloads only the header of the URL you specify

`nikto -h <url>` - A web server scanner that tests for dangerous files, outdated server software and other problems. Can find vulnerabilities in web server versions as well.

Burp

Burp Suite is a professional tool for web hacking. It contains a ton of features and is extended with several plugins. This allows us to do pretty much everything we want towards web services and is used both by amateur hackers and penetration testing professionals every single day. Learning to use Burp efficiently is very important.

#### Burp is a proxy

A proxy is when you route network traffic somewhere else Burp captures the requests that we send to a website, and allows us to manipulate them in many ways In Firefox we can adjust proxy settings This is a bit clunky so the addon FoxyProxy is recommended. Install the Foxyproxy browser extension in [Firefox ](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/)or Chromium and add a new proxy with IP 127.0.0.1 and port 8080. This will send all the requests from your web browser into Burp. This is the standard proxy for Burp, so that your web requests will be sent to Burp, where you work with them in the interface.

#### Burp features

* Proxy - intercept requests
* Spider - find hidden directories and files
* Intruder - brute force logins, etc
* Repeater - same as proxy but you can repeat requests as many times as you want to try a web request numerous times and make small changes.

A very good [Burp Tutorial series ](https://www.youtube.com/playlist?list=PLq9n8iqQJFDrwFe9AEDBlR1uSHEN7egQA)is available on Youtube

### **SQLmap**

Capture a request with Burp -&gt; copypaste and save to a file request.txt in your working folder. sqlmap -r request.txt sqlmap -r r.txt --dbms=mysql --dbs sqlmap -r r.txt --dbms=mysql -D nameofdatabase --tables sqlmap -r r.txt --dbms=mysql -D nameofdatabase -T users --dump

### **Busting directores**

Sometimes we have a web server on `www.lol.htb`, but what if all the secret passwords are stored at `www.lol.htb/secret_passwords`? How do we access them, if there is no link on the web page to the directory? Very often, web site administrators have not been able to properly hide or restrict access to every file and directory on their service. Therefore, we need to do some educated guessing, by trying common files and directories. This is a very common task, so we have very good tools for this!

#### Dirbuster

Use dirb to find directories on a web server. You can also use the GUI version called dirbuster. Just search for it in Kali. The terminal version is called dirb. We recommend you try them both.

`dirb <url_base> <wordlist_file>`

Kali comes with a lot of good wordlists. This one is very good and reliable to find most stuff.`/usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.tx`

#### Dirserach

Dirbuster isn't always the fastest. If you start noticing it's restrictions, we suggest you test dirsearch. You can clone it from Github with

`git clone` [`https://github.com/maurosoria/dirsearch`](https://github.com/maurosoria/dirsearch)\`\`

You then run it with

`python3 dirsearch.py -u` [`http://ip:port`](http://ip:port) `-w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -e php -t 100`

Remember to try different extensions like .py .pl .cgi and .sh. What this does is add an extension to every word in the wordlist in addition to just searching for the directories. So you will try both `http://lol.htb/secret` and`http://lol.htb/secret.php` You might discover a lot of interesting files this way. Also, remember to try relevant file extensions. If you are uncertain what kind of extensions to use, look at the type of web services that are running and Google around for what kind of extensions are used. For example you might learn that IIS servers very often use `asp` and `aspx` extensions, while content managers such as Wordpress relies on ~~`php`~~.

### **Bruteforce attack**

Sometimes we are out of options, so we need to break in the door. A brute force attack basically means to try every single thing in a very brutal way. This is how you get detected so beware.

Burp can be used to perform brute force attacks. Capture a request with Burp -&gt; right click -&gt; send to Intruder -&gt; Set a marker -&gt; Specify payload / wordlist -&gt; Start attack

One of the most common lists for brute force attacks is called `rockyou`. It contains 14+ million passwords and is built into Kali. You will most definitely use this one a lot`/usr/share/wordlists/rockyou.txt`

#### SQLmap

SQLmap is one of our absolute favorite hacking tools. It's immesenly powerful to the extent that even if you sat down and learnt every option in the program you would still have a lot to learn. It basically automatically performs every kind of SQL injection you can imagine and even allows you to get a shell on the system straight from the program. Quite fun indeed!

This is pretty much the route we take when working with databases and injections. It is all built in a very neat hierarchy.

1. **DBMS** = database management system. e.g. MySQL, Postgresql, MSSQL
2. **Databases** - the actual databases that contains the content. You can have many databases in one DBMS.
3. Tables - A database can have many tables
4. Columns - 
5. Rows - this is where you will find your username and password.
6. 
**How to exploit an SQL injection with SQLMap** 

Capture a request with Burp and copypaste it. Save it to a file `r.txt` Then execute sqlmap by pointing to the request. It will then automatically start injecting where it deems fit.

`sqlmap -r request.txt`

If it finds a parameter to inject, you will most likely be told so and SQLmap will reveal what kind of DBMS. From there you want to continue down the hierarchy indicated above. So next we specify the DBMS we got and say that we want to extract the databases:

 `sqlmap -r r.txt --dbms=mysql --dbs`

Fine! We got a database we want to extract. Let's specify that, and that we want to extract the tables:

`sqlmap -r r.txt--dbms=mysql -D corporate --tables`

And when we have a table, let's just dump all the content of that table.

`sqlmap -r r.txt --dbms=mysql -D nameofdatabase -T users --dump`

**We should end up with a complete dump of all the columns and rows in the** `users` **table in the** `corporate` **database that is hosted on a** `mysql` **dbms. And we barely did anything but type some lines and press enter! Magic!**



