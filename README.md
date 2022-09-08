# Agent T

> Bradley Lubow | rnbochsr, September 2022

*My notes and solutions to the TryHackMe.com's Agent T room.*

*Something seems a little off with the server.* 

## Task 1 - Find The Flag
Agent T uncovered this website, which looks innocent enough, but something seems off about how the server responds...

After deploying the vulnerable machine attached to this task, please wait a couple of minutes for it to respond.

### Recon - Enumeration 

I started with the standard scans:
* `nmap`
* `gobuster`
* `nikto`
* `ffuf`

#### NMAP
```bash
# Nmap 7.80 scan initiated Mon Sep  5 22:46:25 2022 as: nmap -sC -sV -v -oN scans/nmap.scan agent-t
Nmap scan report for agent-t (10.10.245.239)
Host is up (0.0011s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title:  Admin Dashboard
MAC Address: 02:0B:16:F2:06:F5 (Unknown)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Sep  5 22:46:32 2022 -- 1 IP address (1 host up) scanned in 7.18 seconds
```

#### Nikto
```bash
- Nikto v2.1.6/2.1.5
+ Target Host: agent-t
+ Target Port: 80
+ GET Retrieved x-powered-by header: PHP/8.1.0-dev
+ GET The anti-clickjacking X-Frame-Options header is not present.
+ GET The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ GET The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ OSVDB-44056: GET /sips/sipssys/users/a/admin/user: SIPS v0.2.2 allows user account info (including password) to be retrieved remotely.
+ OSVDB-3092: GET /demo/: This might be interesting...
+ OSVDB-18114: GET /reports/rwservlet?server=repserv+report=/tmp/hacker.rdf+destype=cache+desformat=PDF:  Oracle Reports rwservlet report Variable Arbitrary Report Executable Execution
```

These only showed a web server running and nothing much else. `Gobuster` errored and wouldn't run without a wildcard switch and `FFUF` showed me why. It reported every attempt as `200 ok`. So no help at all. 

The website was a skeleton of placeholder data and links. The page's source code showed a flat html page and no notes or references to other sites or servivces. The hint said look at the headers. 

#### Web Browser Developer Tools
I opened the developer tools and reloaded the page. I didn't see anythng of interest. There were no cookies to try to abuse. Just the javascript to run the site and the html code. Checking each tab I saw that the website was listing a development version of `PHP/8.1.0-dev`. That isn't normal practice. Being the framework for the website it should be a production version not a development one. 

Looking back at my `nmap.scan` file, I see that I already had that information. I guess I didn't read the entire line to the end. I saw port 80 and moved on. Had I taken a moment to read it I might not have needed the hint. But I learned something. So either was it works. 

I ran a search for exploits of this PHP version and found an entry in the `exploit-db.com` database. It was a Python script that is supposed to generate a pseudo-shell. All the script needs is the URL. It won't be fully interactve, but we can look around. 

### Exploit the server
* Copy the code from `exploit-db.com` into my terminal.
* Save the file.
* Run the script and I have a shell. 

It says I'm root. I can't change directories. I'm in `/var/www/html`. I can view the `/etc/passwd` file. There are no user home directories. I ran a directory list for the `/root` directory.
```bash
$ ls -la /
total 76
drwxr-xr-x   1 root root 4096 Mar  7 22:03 .
drwxr-xr-x   1 root root 4096 Mar  7 22:03 ..
-rwxr-xr-x   1 root root    0 Mar  7 22:03 .dockerenv
drwxr-xr-x   1 root root 4096 Mar 30  2021 bin
drwxr-xr-x   2 root root 4096 Nov 22  2020 boot
drwxr-xr-x   5 root root  340 Sep  5 22:40 dev
drwxr-xr-x   1 root root 4096 Mar  7 22:03 etc
-rw-rw-r--   1 root root   38 Mar  5  2022 flag.txt
drwxr-xr-x   2 root root 4096 Nov 22  2020 home
drwxr-xr-x   1 root root 4096 Mar 30  2021 lib
drwxr-xr-x   2 root root 4096 Jan 11  2021 lib64
drwxr-xr-x   2 root root 4096 Jan 11  2021 media
drwxr-xr-x   2 root root 4096 Jan 11  2021 mnt
drwxr-xr-x   2 root root 4096 Jan 11  2021 opt
dr-xr-xr-x 149 root root    0 Sep  5 22:40 proc
drwx------   2 root root 4096 Jan 11  2021 root
drwxr-xr-x   3 root root 4096 Jan 11  2021 run
drwxr-xr-x   2 root root 4096 Jan 11  2021 sbin
drwxr-xr-x   2 root root 4096 Jan 11  2021 srv
dr-xr-xr-x  13 root root    0 Sep  5 22:40 sys
drwxrwxrwt   1 root root 4096 Mar 30  2021 tmp
drwxr-xr-x   1 root root 4096 Jan 11  2021 usr
drwxr-xr-x   1 root root 4096 Mar 30  2021 var
```

There it is, the `flag.txt` file.
```bash
$ cat /root/flag.txt
flag{41[REDACTED]cb}
```

The hint was good. I'm not sure how long I would have taken to think about the page headers. I have to remember to have that in my normal workflow. In all, it was a fun challenge. 
