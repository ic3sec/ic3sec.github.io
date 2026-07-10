---
weight: 4
title: "HTB Starting Point - Sequel"
date: '2026-07-09T21:10:01-07:00'
draft: false
author: "ic3sec"
authorLink: ""
description: "Writeup/Walkthrough for HTB Starting Point - Sequel"
images: []

tags: ["Writeup", "2026", "CTF", "HTB", "Hack the Box", "HTB Starting Point", "Walkthrough", "Linux", "nmap", "HTB Very Easy", "sql", "MySQL", "MariaDB", "misconfiguration"]
categories: ["Writeups", "CTF"]
---

## Overview
---
Part of the "starting point"  boxes on HTB, Sequel has a set of tasks with questions that provide a framework to walk through the machine. Sequel focuses on SQL (specifically, MySQL) and guides the user through some basic SQL commands after abusing a misconfiguration to login to the target database. As this machine is part of the “starting point” category, many of the tasks are fundamental knowledge questions - I highly recommend researching them a bit if you do not know the answer instead of copy/pasting.

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
During our scan, which port do we find serving MySQL?
{{< /admonition >}}

Performing a quick port scan shows one open port with MySQL running on it:
```bash
┌─[ice@parrot]─[~/Sequel]
└──╼ $nmap -p- --reason --min-rate 5000 10.129.95.232
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-10 00:12 EDT
Nmap scan report for 10.129.95.232
Host is up, received echo-reply ttl 63 (0.073s latency).
Not shown: 65534 closed tcp ports (reset)
PORT     STATE SERVICE REASON
3306/tcp open  mysql   syn-ack ttl 63

Nmap done: 1 IP address (1 host up) scanned in 14.76 seconds
```

{{< admonition type=warning title=Answer open=false >}}
3306
{{< /admonition >}}

---

### Task 2
---
{{< admonition type=question open=true >}}
What community-developed MySQL version is the target running?
{{< /admonition >}}

Now that the port is known for the MySQL server I can run another scan specifically on that port to get more details, including the MySQL community version that this target is running:
```bash
┌─[✗]─[ice@parrot]─[~/Sequel]
└──╼ $nmap -p 3306 -sC 10.129.95.232
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-10 00:17 EDT
Host is up (0.076s latency).

PORT     STATE SERVICE
3306/tcp open  mysql
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 59
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, Speaks41ProtocolOld, SupportsTransactions, IgnoreSpaceBeforeParenthesis, IgnoreSigpipes, ODBCClient, LongColumnFlag, Speaks41ProtocolNew, SupportsCompression, SupportsLoadDataLocal, InteractiveClient, FoundRows, ConnectWithDatabase, DontAllowDatabaseTableColumn, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: wt`dSV5C2nWHAM@88QmS
|_  Auth Plugin Name: mysql_native_password

Nmap done: 1 IP address (1 host up) scanned in 43.27 seconds
```

{{< admonition type=note open=true >}}
I used `-sC` here instead of `-sV` due to the task noting that the server is running a specific community-developed MySQL version, which `-sV` can struggle with while `-sC` includes a specific script, [mysql-info](https://nmap.org/nsedoc/scripts/mysql-info.html), which generally works great for determining MySQL versions.
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
MariaDB
{{< /admonition >}}

---

### Task 3
---
{{< admonition type=question open=true >}}
When using the MySQL command line client, what switch do we need to use in order to specify a login username?
{{< /admonition >}}

This flag can be found on the [mysql man page](https://linux.die.net/man/1/mysql).

{{< admonition type=warning title=Answer open=false >}}
-u
{{< /admonition >}}

---

### Task 4
---
{{< admonition type=question open=true >}}
Which username allows us to log into this MariaDB instance without providing a password?
{{< /admonition >}}

Honestly, I just tried a few basic usernames and found this one quite quickly. In order to try connecting without a password simply use: `mysql -u <username> -h <ip_address>`

{{< admonition type=warning title=Answer open=false >}}
root
{{< /admonition >}}

---

### Task 5
---
{{< admonition type=question open=true >}}
In SQL, what symbol can we use to specify within the query that we want to display everything inside a table?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
*
{{< /admonition >}}

---

### Task 6
---
{{< admonition type=question open=true >}}
In SQL, what symbol do we need to end each query with?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
;
{{< /admonition >}}

---

### Task 7
---
{{< admonition type=question open=true >}}
There are three databases in this MySQL instance that are common across all MySQL instances. What is the name of the fourth that's unique to this host?
{{< /admonition >}}

There are a few system databases that come with all MySQL databases, including: `information_schema`, `mysql`, and `performance_schema`. All databases can be displayed with `SHOW DATABASES`:
```mysql {title="MySQL"}
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.078 sec)
```

{{< admonition type=warning title=Answer open=false >}}
htb
{{< /admonition >}}

---

### Task 8
---
{{< admonition type=question open=true >}}
What is the command in MySQL to select a database to interact with?
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
USE
{{< /admonition >}}

---

### Task 9
---
{{< admonition type=question open=true >}}
What is the command in MySQL to show the different columns for a given table?
{{< /admonition >}}

A couple commands can be used for this, but one is a shortcut specifically made for this purpose.

{{< admonition type=warning title=Answer open=false >}}
DESCRIBE
{{< /admonition >}}

---

### Task 10
---
{{< admonition type=question open=true >}}
Which table has a column named "flag"?
{{< /admonition >}}

I started by checking what tables we have available in this `htb` database:
```mysql {title="MySQL"}
MariaDB [htb]> SHOW TABLES;
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
2 rows in set (0.076 sec)
```

Now I can use the command from earlier, `DESCRIBE`, to see what columns are contained in each table.

```mysql {title="MySQL"}
MariaDB [htb]> DESCRIBE users;
+----------+---------------------+------+-----+---------+----------------+
| Field    | Type                | Null | Key | Default | Extra          |
+----------+---------------------+------+-----+---------+----------------+
| id       | bigint(20) unsigned | NO   | PRI | NULL    | auto_increment |
| username | text                | YES  |     | NULL    |                |
| email    | text                | YES  |     | NULL    |                |
+----------+---------------------+------+-----+---------+----------------+
3 rows in set (0.075 sec)

MariaDB [htb]> DESCRIBE config;
+-------+---------------------+------+-----+---------+----------------+
| Field | Type                | Null | Key | Default | Extra          |
+-------+---------------------+------+-----+---------+----------------+
| id    | bigint(20) unsigned | NO   | PRI | NULL    | auto_increment |
| name  | text                | YES  |     | NULL    |                |
| value | text                | YES  |     | NULL    |                |
+-------+---------------------+------+-----+---------+----------------+
3 rows in set (0.075 sec)
```

Interestingly, there is no *column* named flag in either of these tables. I figured this was a mistake on the part of the question and checked by looking at the contents of each table:
```mysql {title="MySQL"}
MariaDB [htb]> SELECT * FROM users;
+----+----------+------------------+
| id | username | email            |
+----+----------+------------------+
|  1 | admin    | admin@sequel.htb |
|  2 | lara     | lara@sequel.htb  |
|  3 | sam      | sam@sequel.htb   |
|  4 | mary     | mary@sequel.htb  |
+----+----------+------------------+
4 rows in set (0.073 sec)

MariaDB [htb]> SELECT * FROM config;
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
7 rows in set (0.075 sec)
```

There it is, we have a row with the name flag in one of the tables.

{{< admonition type=warning title=Answer open=false >}}
config
{{< /admonition >}}

---

### Submit Single Flag
---

The flag is contained in the 'value' column of the config table from the previous task.

{{< admonition type=warning title=Flag open=false >}}
7b4bec00d1a39e3dd4e021ec3d915da8
{{< /admonition >}}

---

## Closing Thoughts
---
Sequel is a solid introduction to some basic database enumeration in MySQL databases. It provides a simple attack path but introduces the user to database navigation fundamentals.

---