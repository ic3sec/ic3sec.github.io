---
weight: 4
title: "HTB Starting Point - Crocodile"
date: '2026-07-10T02:24:13-07:00'
draft: false
author: "ic3sec"
authorLink: ""
description: "Writeup/Walkthrough for HTB Starting Point - Crocodile"
images: []

tags: ["Writeup", "2026", "CTF", "HTB", "Hack the Box", "HTB Starting Point", "Walkthrough", "Linux", "nmap", "HTB Very Easy", "ftp", "misconfiguration", "gobuster", "seclists"]
categories: ["Writeups", "CTF"]
---

## Overview
---
Part of the "starting point"  boxes on HTB, Crocodile has a set of tasks with questions that provide a framework to walk through the machine. Crocodile starts to combine a couple separate attacks on one server into one, beginning with a misconfigured ftp server that allows for anonymous login and ending with some directory busting to find a login page that can be logged into for the flag. As this machine is part of the “starting point” category, many of the tasks are fundamental knowledge questions - I highly recommend researching them a bit if you do not know the answer instead of copy/pasting.

|                  |               |
| ---------------- | ------------- |
| **Release Date** | 06 Oct, 2021  |
| **Difficulty**   | Very Easy     |
| **OS**           | Linux         |
| **Created By**   | [ch4p](https://app.hackthebox.com/users/1) |

---

## Tasks

---

### Task 1
---
{{< admonition type=question open=true >}}
What Nmap scanning switch employs the use of default scripts during a scan?
{{< /admonition >}}

Switches for nmap can be found on their [man page](https://linux.die.net/man/1/nmap).

{{< admonition type=warning title=Answer open=false >}}
-sC
{{< /admonition >}}

---

### Task 2
---
{{< admonition type=question open=true >}}
What service version is found to be running on port 21?
{{< /admonition >}}

Since the last task mentions `-sC` we can run a quick nmap scan on the given port using it:
```bash
[ice@parrot]─[~/Crocodile]$ nmap -sC -p 21 10.129.1.15
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-10 05:32 EDT
Nmap scan report for 10.129.1.15
Host is up (0.073s latency).

PORT   STATE SERVICE
21/tcp open  ftp
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.15.105
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd

Nmap done: 1 IP address (1 host up) scanned in 1.29 seconds
```

{{< admonition type=warning title=Answer open=false >}}
vsFTPd 3.0.3
{{< /admonition >}}

---

### Task 3
---
{{< admonition type=question open=true >}}
What FTP code is returned to us for the "Anonymous FTP login allowed" message?
{{< /admonition >}}

The FTP code is in the script results of the previous nmap scan.

{{< admonition type=warning title=Answer open=false >}}
230
{{< /admonition >}}

---

### Task 4
---
{{< admonition type=question open=true >}}
After connecting to the FTP server using the ftp client, what username do we provide when prompted to log in anonymously?
{{< /admonition >}}

In order to connect to an FTP server anonymously, follow the `ftp` command with the IP address and enter `anonymous` when prompted for a username:
```bash
[ice@parrot]─[~/Crocodile]$ ftp 10.129.1.15
Connected to 10.129.1.15.
220 (vsFTPd 3.0.3)
Name (10.129.1.15:ice): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

{{< admonition type=warning title=Answer open=false >}}
anonymous
{{< /admonition >}}

---

### Task 5
---
{{< admonition type=question open=true >}}
After connecting to the FTP server anonymously, what command can we use to download the files we find on the FTP server?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
get
{{< /admonition >}}

---

### Task 6
---
{{< admonition type=question open=true >}}
What is one of the higher-privilege sounding usernames in 'allowed.userlist' that we download from the FTP server?
{{< /admonition >}}

First the named files needs to be downloaded from the server, we can start by listing the files in the current directory:
```bash
ftp> ls
229 Entering Extended Passive Mode (|||46103|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
```

Now both files can be grabbed with `get`:
```bash
ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
229 Entering Extended Passive Mode (|||41526|)
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
100% |*****************************************************************************************************************************|    33       12.58 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.42 KiB/s)
ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
229 Entering Extended Passive Mode (|||45082|)
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
100% |*****************************************************************************************************************************|    62       23.24 KiB/s    00:00 ETA
226 Transfer complete.
62 bytes received in 00:00 (0.81 KiB/s)
```

Now we can leave the ftp session with `exit` (or open another terminal window) and check out those files, starting with the one we were asked about in this task:
```bash
[ice@parrot]─[~/Crocodile]$ cat allowed.userlist
aron
pwnmeow
egotisticalsw
admin
```

One of those sure sounds high privilege!

{{< admonition type=warning title=Answer open=false >}}
admin
{{< /admonition >}}

---

### Task 7
---
{{< admonition type=question open=true >}}
What version of Apache HTTP Server is running on the target host?
{{< /admonition >}}

Looks like we're going to need to expand that nmap scan from earlier:
```bash
[ice@parrot]─[~/Crocodile]$ nmap -p- --reason --min-rate 5000 10.129.1.15
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-10 06:04 EDT
Nmap scan report for 10.129.1.15
Host is up, received echo-reply ttl 63 (0.076s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Nmap done: 1 IP address (1 host up) scanned in 14.27 seconds
```

Looks like there's another port open, let's run nmap again with `-sV` to figure out the http server version:
```bash
[ice@parrot]─[~/Crocodile]$ nmap -p 80 -sV 10.129.1.15
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-10 06:06 EDT
Nmap scan report for 10.129.1.15
Host is up (0.075s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.36 seconds

```

{{< admonition type=warning title=Answer open=false >}}
Apache httpd 2.4.41
{{< /admonition >}}

---

### Task 8
---
{{< admonition type=question open=true >}}
What switch can we use with Gobuster to specify we are looking for specific filetypes?
{{< /admonition >}}

We might need to do a bit more digging for this one since the [gobuster man page](https://manpages.debian.org/testing/gobuster/gobuster.1.en.html) does not list it.

However, gobuster's cli help command tells us that we can use gobuster [command] --help for more specific help, we can try that with the `gobuster dir` command to look a bit deeper into available options.

{{< admonition type=warning title=Answer open=false >}}
-x
{{< /admonition >}}

---

### Task 9
---
{{< admonition type=question open=true >}}
Which PHP file can we identify with directory brute force that will provide the opportunity to authenticate to the web service?
{{< /admonition >}}

We can start by setting up dirbuster, which takes a couple options:
- `-u`, the target url
- `-w`, the wordlist for dirbuster to go through

And, since it was mentioned that we want a PHP file, we can also add `-x php` to narrow our search down.

{{< admonition type=note open=true >}}
You may have your wordlists located in a different place. It's also totally fine to use another directory wordlist if that suits your use case better. If you're missing them entirely, however, check out [SecLists](https://github.com/danielmiessler/seclists) - or, more specifically for this case, [SecLists Web-Content](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content).
{{< /admonition >}}

Let's run it:
```bash
[ice@parrot]─[~/Crocodile]$ gobuster dir -u http://10.129.1.15 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.1.15
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 276]
/login.php            (Status: 200) [Size: 1577]
Progress: 437 / 175330 (0.25%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 467 / 175330 (0.27%)
===============================================================
Finished
===============================================================
```

And almost immediately we get a response from `/login.php` - I cancelled the search there, but you can let it keep running if you want to see if there are any other interesting pages on the target. 

{{< admonition type=warning title=Answer open=false >}}
login.php
{{< /admonition >}}

---

### Submit Single Flag
---

All that we have left now is to login to that page and grab the flag!

But wait, first we need to grab the password for the user (`admin`) that we're going to try to login as, let's see:
```bash
[ice@parrot]─[~/Crocodile]$ cat allowed.userlist.passwd 
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

Based on the order (being the same as the username file) we can assume the password we're looking for here is the last one (`rKXM59ESxesUFHAd`) - now we can navigate to the page and login.

And right there on the dashboard lies the flag.

{{< admonition type=warning title=Flag open=false >}}
c7110277ac44d78b6a9fff2232434d16
{{< /admonition >}}

---

## Closing Thoughts
---
Crocodile starts to introduce multiple aspects to one attack, having the user get multiple results back from an nmap scan, both of which are services that need to be leveraged to get the flag. Starting out with a misconfigured ftp server, Crocodile leaves out enough information that the user will need to fill in a few steps to complete their goal but should have little trouble staying on track with the provided guidance.

I enjoyed the fact that the userlist and passwd files were in the same order with a small red herring thrown in of the `root` password, making sure that users were paying attention to the order of the list (or just brute forcing it since there were only 4 password options, of course).

---