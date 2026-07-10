---
weight: 4
title: "HTB Starting Point - Redeemer"
date: '2026-07-08T21:02:14-07:00'
draft: false
author: "ic3sec"
authorLink: ""
description: "Writeup/Walkthrough for HTB Starting Point - Redeemer"
images: []

tags: ["Writeup", "2026", "CTF", "HTB", "Hack the Box", "HTB Starting Point", "Walkthrough", "Redis", "redis-cli", "Linux", "nmap", "HTB Very Easy", "nc", "ncat"]
categories: ["Writeups", "CTF"]
---

## Overview
---
Part of the "starting point"  boxes on HTB, Redeemer has a set of tasks with questions that provide a framework to walk through the machine. Redeemer focuses on the basics of Redis database servers, how to interact with them and how to enumerate/exploit them. As this machine is part of the “starting point” category, many of the tasks are fundamental knowledge questions - I highly recommend researching them a bit if you do not know the answer instead of copy/pasting.

|                  |               |
| ---------------- | ------------- |
| **Release Date** | 11 May, 2022  |
| **Difficulty**   | Very Easy     |
| **OS**           | Linux         |
| **Created By**   | [ch4p](https://app.hackthebox.com/users/1) |

---

## Tasks

---

### Task 1
---
{{< admonition type=question open=true >}}
Which TCP port is open on the machine?
{{< /admonition >}}

Starting out with a quick nmap scan of open ports, it looks like there is a Redis service running on port 6379
```bash
[ice@parrot]─[~/Redeemer]$ nmap -p- --reason --min-rate 5000 10.129.132.119
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-08 22:43 EDT
Nmap scan report for 10.129.132.119
Host is up, received echo-reply ttl 63 (0.073s latency).
Not shown: 65534 closed tcp ports (reset)
PORT     STATE SERVICE REASON
6379/tcp open  redis   syn-ack ttl 63

Nmap done: 1 IP address (1 host up) scanned in 13.59 seconds
```

{{< admonition type=warning title=Answer open=false >}}
6379
{{< /admonition >}}

---

### Task 2
---
{{< admonition type=question open=true >}}
Which service is running on the port that is open on the machine?
{{< /admonition >}}

The service running on the port is included in the result of the previous nmap scan.

{{< admonition type=warning title=Answer open=false >}}
redis
{{< /admonition >}}

---

### Task 3
---
{{< admonition type=question open=true >}}
What type of database is Redis? Choose from the following options: (i) In-memory Database, (ii) Traditional Database
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
In-memory Database
{{< /admonition >}}

---

### Task 4
---
{{< admonition type=question open=true >}}
Which command-line utility is used to interact with the Redis server? Enter the program name you would enter into the terminal without any arguments.
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
redis-cli
{{< /admonition >}}

{{< admonition type=note open=true >}}
This is not the only way to connect to redis servers from a terminal, and since I do not want to install `redis-tools`, I will instead be using `nc` to connect. You can do this using `nc -v [server_ip] [port]` (or, for servers with SSL/TLS, you can instead use `ncat` with `ncat -v --ssl [server_ip] [port]`)
{{< /admonition >}}

---

### Task 5
---
{{< admonition type=question open=true >}}
Which flag is used with the Redis command-line utility to specify the hostname?
{{< /admonition >}}

As per the [redis-cli documentation](https://redis.io/docs/latest/develop/tools/cli/) the hostname can be specified using the -h flag.

{{< admonition type=warning title=Answer open=false >}}
-h
{{< /admonition >}}

---

### Task 6
---
{{< admonition type=question open=true >}}
Once connected to a Redis server, which command is used to obtain the information and statistics about the Redis server?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
INFO
{{< /admonition >}}

---

### Task 7
---
{{< admonition type=question open=true >}}
What is the version of the Redis server being used on the target machine?
{{< /admonition >}}

We can start by connecting to the redis server using `nc`:
```bash
[ice@parrot]─[~/Redeemer]$ nc -v 10.129.132.119 6379
10.129.132.119: inverse host lookup failed: Unknown host
(UNKNOWN) [10.129.132.119] 6379 (redis) open
```

Then we can run 'info' to figure out the redis version:
```bash
info 
$3289         
# Server          
redis_version:5.0.7
```

{{< admonition type=warning title=Answer open=false >}}
5.0.7
{{< /admonition >}}

---

### Task 8
---
{{< admonition type=question open=true >}}
Which command is used to select the desired database in Redis?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
SELECT
{{< /admonition >}}

---

### Task 9
---
{{< admonition type=question open=true >}}
How many keys are present inside the database with index 0?
{{< /admonition >}}

At the end of the `info` command from earlier is the `# Keyspace` section:
```bash
# Keyspace
db0:keys=4,expires=0,avg_ttl=0
```

{{< admonition type=warning title=Answer open=false >}}
4
{{< /admonition >}}

---

### Task 10
---
{{< admonition type=question open=true >}}
Which command is used to obtain all the keys in a database?
{{< /admonition >}}

There are a couple commands for this, namely `KEYS *` and `SCAN 0`, both will return the same results, however the difference between them is that `KEYS *` runs as a single operation, so if you are working with a large database it can stall other database operations heavily (since redis databases are single-threaded). `SCAN 0`, on the other hand, gets data in smaller "chunks" instead of as one contiguous operation, meaning it can take slightly longer if other operations are happening, but it will not stall those other database operations.

{{< admonition type=warning title=Answer open=false >}}
KEYS *
{{< /admonition >}}

---

### Submit Single Flag
---
Running the command from the last task returns the keys available in this database:
```bash
keys *
*4
$4
stor
$4
numb
$4
flag
$4
temp
```

That `flag` key sure looks suspicious, since it is presumably a simple string the command `get flag` can be used to print out the value.

{{< admonition type=warning title=Flag open=false >}}
03e1d2b376c37ab3f5319922053953eb
{{< /admonition >}}

---

## Closing Thoughts
---
Redeemer provides some good information on Redis databases and walks through a few important commands for enumerating them. Although I would like to have seen mention of alternative tools for these purposes (e.g. `nc` or `ncat` instead of only `redis-cli`), I do understand giving specific information and letting students research/discover things on their own. However, I do think it is more important to mention alternative commands, e.g. `SCAN 0` vs `KEYS *`, since those can have a noticeable impact when iterating through a live database and are not implicity something that one might question if they've only learned about one.

Great learning experience no less!

---