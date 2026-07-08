---
weight: 4
title: "HTB Starting Point - Fawn"
date: '2026-07-08T01:21:04-07:00'
draft: false
author: "ic3sec"
authorLink: ""
description: "Writeup/Walkthrough for HTB Starting Point - Fawn"
images: []

tags: ["Writeup", "2026", "CTF", "HTB", "Hack the Box", "HTB - Starting Point", "Walkthrough", "ftp", "Linux", "nmap"]
categories: ["Writeups", "CTF"]
---

## Overview

Part of the "starting point"  boxes on HTB, Fawn has a set of tasks with questions that provide a framework to step through the machine. Fawn focuses on an simple ftp server that has anonymous login enabled.

|                  |               |
| ---------------- | ------------- |
| **Release Date** | 30 Sept, 2021 |
| **Difficulty**   | Very Easy     |
| **OS**           | Linux         |
| **Creator(s)**   | HTB-Bot       |

## Tasks

As this machine is part of the "starting point" category, many of the tasks are fundamental knowledge questions - I highly recommend reading into them a bit if you do not know the answer instead of copy/pasting.

### Task 1
{{< admonition type=question open=true >}}
What does the 3-letter acronym FTP stand for?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
File Transfer Protocol
{{< /admonition >}}

### Task 2
{{< admonition type=question open=true >}}
Which port does the FTP service listen on usually?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
21
{{< /admonition >}}

### Task 3
{{< admonition type=question open=true >}}
FTP sends data in the clear, without any encryption. What acronym is used for a later protocol designed to provide similar functionality to FTP but securely, as an extension of the SSH protocol?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
SFTP
{{< /admonition >}}

### Task 4
{{< admonition type=question open=true >}}
What is the command we can use to send an ICMP echo request to test our connection to the target?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
ping
{{< /admonition >}}

### Task 5
{{< admonition type=question open=true >}}
From your scans, what version is FTP running on the target?
{{< /admonition >}}

Since this box is specifically asking about FTP and mentions the default port, I can run a very quick and simple nmap scan to check this, using the -sV flag to probe the service/version info:
```bash
[ice@parrot]─[~/Fawn]$ nmap -sV -p 21 10.129.129.145
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-08 03:41 EDT
Nmap scan report for 10.129.129.145
Host is up (0.076s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
Service Info: OS: Unix
```

{{< admonition type=warning title=Answer open=false >}}
vsftpd 3.0.3
{{< /admonition >}}

### Task 6
{{< admonition type=question open=true >}}
From your scans, what OS type is running on the target?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
Unix
{{< /admonition >}}

### Task 7
{{< admonition type=question open=true >}}
What is the command we need to run in order to display the 'ftp' client help menu?
{{< /admonition >}}

As noted on the [ftp man page](https://linux.die.net/man/1/ftp) `?` is a synonym for "help", so `ftp -?` serves to bring up the ftp client help menu.

{{< admonition type=warning title=Answer open=false >}}
ftp -?
{{< /admonition >}}

### Task 8
{{< admonition type=question open=true >}}
What is username that is used over FTP when you want to log in without having an account?
{{< /admonition >}}

This one is not listed on the base ftp man page but instead on the [ftpd man mage](https://linux.die.net/man/8/ftpd) - which notes that either "anonymous" or "ftp" can be used for anonymous logins.

{{< admonition type=warning title=Answer open=false >}}
anonymous
{{< /admonition >}}

### Task 9
{{< admonition type=question open=true >}}
What is the response code we get for the FTP message 'Login successful'?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
230
{{< /admonition >}}

### Task 10
{{< admonition type=question open=true >}}
There are a couple of commands we can use to list the files and directories available on the FTP server. One is dir. What is the other that is a common way to list files on a Linux system.
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
ls
{{< /admonition >}}

### Task 11
{{< admonition type=question open=true >}}
What is the command used to download the file we found on the FTP server?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
get
{{< /admonition >}}

### Submit Single Flag
Alright, the real meat and bones of this box - but really, the tasks before it establish all the knowledge that is needed to perform this task. If you were performing the task steps above on the target as it went, you should only be one step away from the answer, but here I'll walk through the few steps from the start.

First, I need to install an `ftp` client, since I did not have a standard one installed: `sudo apt-get install ftp`

Next, since the steps mentioned anonymous login, I can try connecting an ftp client to the target, providing "anonymous" as the username and nothing for the password (which, conveniently, does also show the "Login successful" response code, among a couple others):
```bash
[ice@parrot]─[~/Fawn]$ ftp 10.129.129.145
Connected to 10.129.129.145.
220 (vsFTPd 3.0.3)
Name (10.129.129.145:ice): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

Now I can use that directory listing command from earlier to see available files:
```bash
ftp> ls
229 Entering Extended Passive Mode (|||36305|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
```

And then grab `flag.txt` using `get flag.txt`, after which I can close my ftp connection using `exit`.

Finally, all that remains is so get the flag, so I can `cat flag.txt` and the box has been solved! 

{{< admonition type=warning title=Flag open=false >}}
035db21c881520061c53e0536e44f815
{{< /admonition >}}

## Closing Thoughts
Fawn is a great, very introductory box that provides a nice framework for learning about ftp, though many modern OSes will no longer have an ftp client installed (nor should they), as ftp is *generally* deprecated in favor of SFTP (and FTPS in some cases).

That being said, it's still good foundational knowledge to have and - believe it or not - some unsecure ftp servers most definitely do exist in the wild. Wild.