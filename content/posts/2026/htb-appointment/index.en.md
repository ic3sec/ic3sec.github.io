---
weight: 4
title: "HTB Starting Point - Appointment"
date: '2026-07-08T22:52:05-07:00'
draft: false
author: "ic3sec"
authorLink: ""
description: "Writeup/Walkthrough for HTB Starting Point - Appointment"
images: []

tags: ["Writeup", "2026", "CTF", "HTB", "Hack the Box", "HTB Starting Point", "Walkthrough", "Linux", "nmap", "HTB Very Easy", "sql", "MySQL", "sql injection"]
categories: ["Writeups", "CTF"]
---

## Overview
---
Part of the "starting point"  boxes on HTB, Appointment has a set of tasks with questions that provide a framework to walk through the machine. Appointment focuses on SQL (specifically, MySQL) and guides the user towards writing their first SQL injection attack. As this machine is part of the “starting point” category, many of the tasks are fundamental knowledge questions - I highly recommend researching them a bit if you do not know the answer instead of copy/pasting.

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
What does the acronym SQL stand for?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
Structured Query Language
{{< /admonition >}}

---

### Task 2
---
{{< admonition type=question open=true >}}
What is one of the most common type of SQL vulnerabilities?
{{< /admonition >}}

While there are others, one type of attack is definitely most prevalent for SQL - so much so that a search of "sql vulnerabilities" should return this as the top result.

{{< admonition type=warning title=Answer open=false >}}
SQL Injection
{{< /admonition >}}

---

### Task 3
---

This task number was skipped over when I completed the box, leaving this here for continuity of task numbers.

---

### Task 4
---
{{< admonition type=question open=true >}}
What is the 2021 OWASP Top 10 classification for this vulnerability?
{{< /admonition >}}

OWASP's 2021 top 10 can be found [here](https://owasp.org/Top10/2021/).

{{< admonition type=warning title=Answer open=false >}}
A03:2021-Injection
{{< /admonition >}}

---

### Task 5
---
{{< admonition type=question open=true >}}
What does Nmap report as the service and version that are running on port 80 of the target?
{{< /admonition >}}

Given the port here a targetted nmap scan can be run for efficiency:
```bash
[ice@parrot]─[~/Appointment]$ nmap -sV -p 80 10.129.132.222
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-09 01:58 EDT
Nmap scan report for 10.129.132.222
Host is up (0.074s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.34 seconds
```

{{< admonition type=warning title=Answer open=false >}}
Apache httpd 2.4.38 ((Debian))
{{< /admonition >}}

---

### Task 6
---
{{< admonition type=question open=true >}}
What is the standard port used for the HTTPS protocol?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
443
{{< /admonition >}}

---

### Task 7
---
{{< admonition type=question open=true >}}
What is a folder called in web-application terminology?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
Directory
{{< /admonition >}}

---

### Task 8
---
{{< admonition type=question open=true >}}
What is the HTTP response code that is returned for `Not Found` errors?
{{< /admonition >}}

HTTP codes and their descriptions can be found [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status).

{{< admonition type=warning title=Answer open=false >}}
404
{{< /admonition >}}

---

### Task 9
---
{{< admonition type=question open=true >}}
Gobuster is one tool used to brute force directories on a webserver. What switch do we use with Gobuster to specify we're looking to discover directories, and not subdomains?
{{< /admonition >}}

gobuster's mode flags can be found on the [man page](https://manpages.debian.org/testing/gobuster/gobuster.1.en.html).

{{< admonition type=warning title=Answer open=false >}}
dir
{{< /admonition >}}

---

### Task 10
---
{{< admonition type=question open=true >}}
What single character can be used to comment out the rest of a line in MySQL?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
`#`
{{< /admonition >}}

---

### Task 11
---
{{< admonition type=question open=true >}}
If user input is not handled carefully, it could be interpreted as a comment. Use a comment to login as admin without knowing the password. What is the first word on the webpage returned?
{{< /admonition >}}

I know there is an Apache server running on port 80 from the earlier nmap scan, here's what it looks like if I navigate to the page in a browser:
![Login Page](login_page.png)

Since the question mentions logging in as admin without knowing the password and this module is about SQL injection, it's safe to assume where this is going next. Let's take a look at how the SQL query in the background may be constructed. Let's say, for a successful login, the server needs to find a result in the database that matches both the username and password input. That query could look something like this:

```mysql {title="MySQL"}
SELECT * FROM users WHERE username = '<input_username>' AND password = '<input_password>' AND <other conditions>
```

Assuming that our raw input is being taken for `<input_username>` and `input_password`, we can start to see where this might go wrong. As mentioned in the previous task, `#` comments anything after it on a line in MySQL. SQL can also evaluate equations, e.g. `1=1` which always evaluates to true.

Combining these concepts, we can try to construct a simple SQL injection attack:

**Username:** `admin`

**Password:** `' OR 1=1 #`

Again, assuming raw input and the above query example, the backend would now evaluate the following query:
```mysql {title="MySQL"}
SELECT * FROM users WHERE username = 'admin' AND password = '' OR 1=1 #AND <other conditions>
```

And that query would select the user with username = `admin` and *any* password (since we are checking the condition `OR 1=1`, which is always true), and if there were any other conditional checks as part of the query coming after that, they would be ignored with the `#` commenting out the rest of the line.

Trying it out on the webpage brings up a new page which contains the answer to this task (and the flag)!

{{< admonition type=warning title=Answer open=false >}}
Congratulations
{{< /admonition >}}

---

### Submit Single Flag
---

Flag is on the same page that was reached in the last task.

{{< admonition type=warning title=Flag open=false >}}
e3d0796d002a446c0e622226f42e9672
{{< /admonition >}}

---

## Closing Thoughts
---
Appointment is another great introductory machine for a very core fundamental in penetration testing: SQL injection. I feel as though this provided a solid guide to steer the user's hand in the right direction while not giving away the answer completely - some research into the basics of SQL injection (or SQL in general) needs to be done to understand the construction of the injection and this is just the tip of the iceberg.

Although SQL injections, at least in the form presented in this module, are *significantly* less common due to modern advancements, they are still possible to run across -- or, in other cases, there are some much more contrived forms of SQL injection that attempt to abuse the solutions (e.g. input sanitation) put in place to prevent just that.

There are numerous resources on SQL injection but if you're interested in more the [OWASP SQL injection page](https://owasp.org/www-community/attacks/SQL_Injection) is a great place to start.

---