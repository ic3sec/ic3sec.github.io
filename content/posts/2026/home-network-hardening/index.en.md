---
weight: 4
title: "Home Network Hardening"
date: '2026-06-30T20:17:59-07:00'
draft: false
author: "ic3sec"
authorLink: ""
description: "Home network setup and hardening on MirktoTik hAP ax3"
images: []

tags: ["Writeup", "2026", "Networking", "MikroTik"]
categories: ["Writeups"]
---

## Overview
---
I have long wanted to do some home network hardening, primarily separating out IoT devices from the rest of my network, but was unable to do so with the router that I was previously using (I considered OpenWRT, but unfortunately my old router was unsupported).

This writeup will cover my journey into home network setup/hardening, from research and planning to implementation and troubleshooting.

---

## Router Selection
---
{{< admonition type=warning title="Disclaimer" open=true >}}
I am not sponsored by nor affiliated with any of the companies that will be discussed in this post. All of the following are my personal opinions based on information that I gathered through my own research and I do not believe there is a "one-size-fits-all" for home networking equipment. Everyone has their own specific needs, please do your own research to find what is best for you.
{{< /admonition >}}

While searching for a new router, my primary criteria was finding a consumer-grade product that offered the (unrestricted) ability to create VLANs and fine-tune network settings. Beyond that, I didn't need much, just decent dual-band WiFi capabilities, a few ethernet ports, and to not break the bank.

Needing VLAN capabilities cut down the available consumer product options quite heavily, but I still went through many product specification pages in my search, primarily looking at various models of Ubiquiti and TP-Link routers. 

I read great things about Ubiquiti in particular, but kept looking since their devices were on the higher end of what I was hoping to pay. I also read that they have fairly vendor-specific configuration for VLANs, which I wanted to avoid if possible.

Eventually I ran across MikroTik and I was extremely surprised by the product specifications they were offering for the price point. At first I was worried that it was too good to be true. However, I quickly found that the sentiment around MikroTik's products was not "too good to be true", but rather that many find their products to *not* be user friendly. I've never been afraid of a challenge though, and from everything I could find the hAP ax<sup>3</sup> seemed to fit my needs precisely - albeit maybe with a bit of a learning curve.

One semi-common complaint that did concern me was that the WiFi capabilities were not as good as advertised, especially for any decent range. I decided to take my chance on this since, if it were the only weak point, I'd be able to pick up an AP as well and still be under the price point of some of the other routers I was considering.

As of yet, I have had no issues with the WiFi range/performance/stability in a single-floor, couple thousand square foot house (router is on one end of the house, still getting ~50% of my max speeds on complete opposite side of the house on both bands).

Since this is my first foray into the slightly less consumer-centric network equipment space, I cannot give a proper comparison between this and other routers, but I will say that my experience with MikroTik's hAP ax<sup>3</sup> has been fantastic so far.

---

## Planning
---
As I am not the only one on the network that I will be implementing these changes on, I wanted to be sure that my plans were solid regarding both the ending setup and the means to get there as smoothly as possible.

---

### Network Plans
---
My home network is rather simple and my primary goal in planning out my subnets/VLANs was to keep it that way, while making it as secure and efficient as I could.

In order to do so, I first wrote out all the planned devices that would be on the network. Doing this allowed me to visualize the groupings so that I could map out a framework for my VLANs, which initially looked like this:
- **VLAN10 (Management)** - Router, AdGuard DNS server
- **VLAN20 (Work)** - Work PC/phone
- **VLAN30 (Trusted)** - Family PCs/phones
- **VLAN40 (Printer)** - Printer
- **VLAN50 (IoT)** - TVs
- **VLAN60 (VoIP)** - VoIP devices
- **VLAN70 (Guest)** - Guest devices and mobile gaming

At first glance this may seem overkill for a home network - and in most cases, I'd argue it may be, but I do have my reasoning for each VLAN to exist and they all serve different purposes. Specifically, there are a few "groups" here that could logically be put together, e.g. IoT and Printer, but I decided to separate them due to configuration differences (limited WAN access for IoT devices, but no WAN access for printer).

Initially I was considering making separate SSIDs for each of these networks, but after some research (and thinking about it for a bit), I decided against it, both for maintenance simplicity and to avoid unnecessarily flooding the network with beacon management frames. Instead, I planned to split these networks into a few general categories ("Main", "IoT", "Guest") and use MAC-based access-lists to put a select few devices into their VLANs under each of the main SSIDs. The overall planned VLAN structure now looked as follows:

```goat
"Main"                                "IoT"                              "Guest"
|                                     |                                  |
'-> Default --> VLAN 30 (Trusted)     '-> Default --> VLAN50 (IoT)       '-> Default --> VLAN70 (Guest)
|                                     |
'-> MAC ACL --> VLAN10 (Management)   '-> MAC ACL --> VLAN40 (Printer)	
|                                   
'-> MAC ACL --> VLAN20 (Work)       
```

Regarding VLAN60 (VoIP), I currently use a single device over ethernet to connect to the wireless phone base, so it falls outside of any of the groupings as it will be using a port-based VLAN solely for isolation and QoS purposes.

I also planned to have a port assigned to VLAN10 as backup during setup, in case of hardware failure or if I manage to lock myself out of the router in spite of all my precautionary steps.

Now of course, VLANs on their own are doing effectively nothing in regards to network security. That's where firewall rules come in. My plan was to block all unnecessary inter-VLAN communication by default dropping all packets between VLANs, and only allow specific required protocols/services based on my household needs, e.g. IPP (TCP 631) from my work/trusted VLANs to my printer VLAN. On top of this, I planned to remove/restrict WAN access on my IoT and printer networks and set myself reminders for an update schedule to ensure the devices on those networks are not going unpatched for too long.

---

### Rollout Plans
---
My rollout plans were also relatively simple and were focused entirely around uptime/network stability and safety.

I planned a staged rollout, with the goal of being able to learn/test with a few of my own devices at each step without having any major impact on the rest of my network:
1. Swap routers, update firmware and utilize RouterOS default configuration for basic safety
2. Configure SSIDs (and passwords) identically to previous network to allow devices to reconnect smoothly and achieve basic network functionality
3. Harden device from default settings where possible (remove unnecessary protocols, change admin accounts, etc.)
4. Create a separate bridge to test and learn each component of RouterOS VLAN/DHCP/IP tables/etc. setup without impacting the main network
5. Stage network configuration by moving devices to the test bridge one at a time when not in use and checking functionality
6. Finalize any configuration changes and move to "production"

I reasoned that with the approach outlined above, I would have minimal changes at each step as well as ample time to verify that my configuration was working properly before applying everything to my main network.

I also planned to take frequent backups/export my configuration so that, if anything were to go wrong, I could always return to the most recent backup and have a stable, working network for the rest of the house while I figured out what happened.

---

## Implementation
---
Due to my lack of experience with RouterOS I took precautionary steps wherever I could in order to avoid taking my home network down. I will note that it would likely be better practice to tinker with and learn configuration within a lab environment, especially if mistakes could impact others, but for those that also have their reasons for not doing so I will outline what I did to mitigate the chances of that happening below.

---

### Initial Setup/Default Config
---
Before doing anything else I began with the default configuration, updated the OS and changed the existing SSIDs to match my old SSIDs (one 5GHz and one 2.4GHz) in both name, password and authentication types. I did this to ensure that the WiFi range was workable and that all the devices in my house had no connectivity issues, while saving myself the headache of having to reconfigure every device in order to find out.

After everything was up and running on the most basic configuration, I began my research into RouterOS capabilities and syntax. Two of my early reference points for useful general information were [The Network Berg](https://www.youtube.com/c/thenetworkberg) and the official [MikroTik](https://www.youtube.com/@mikrotik) channels, and I would highly recommend checking out some of their videos if you are interested in MikroTik products or get stuck during configuration.

---

### Safe Mode
---
In my preliminary research on MikroTik/RouterOS I discovered safe mode which I utilized very heavily, as it is effectively a safety net for breaking changes (that is, locking yourself out of the router style breaking changes). It is present for both WinBox UI (top right) and terminal sessions (`Ctrl + X` or `/safe-mode/toggle`), however it can only be active in one 'session' at a time and each terminal/WinBox instance is a separate session, even if you open a terminal from within the WinBox UI. Ensure that you are using safe mode within the session that you are making changes in, e.g. if you want to make changes in WinBox UI, safe mode should be enabled *in the UI* and not in a console session instead (or vice versa).

---

### Backups and Exports
---
On top of safe mode, MikroTik offers a robust backup and config export system which can act as a checkpoint system to recover from in case of system lockout, or simply when wanting to revert large blocks of changes.

**Backups** (`/system/backup`) generate a binary configuration (`.backup`) file, which is intended to be used to provide a 1:1 recovery point on *the same hardware* with which the backup was created. They contain sensitive information including user passwords and can be encrypted/password protected on generation.

**Exports** (`/system/export`) generate a plain-text script (`.rsc`) file which acts as a configuration setup script and does not (at least, by default) include any sensitive information. Exports are not hardware tied and can be used to generate quick setup scripts to configure multiple devices, but do not act as full backups.

In the case of setting up a single device, **backups** are the superior choice since they provide a 1:1 recovery point and being hardware tied is not an issue. Before any major steps I took a backup using `/system/backup save name=[date]_backup[num]`and downloaded it locally.

---

### Management Port
---
While consistent use of safe mode alongside frequent backups is likely enough, I did set up one more fallback by creating a dedicated management port. In my setup I have the luxury of having an extra couple ethernet ports, so dedicating one to be a management port was no problem. Here's how I set it up using ether5 as my management port:

1. Remove port from bridge 
   ```text
   /interface/bridge/port remove [find interface=ether5]
   ```
2. Assign an IP and set up DHCP:
   ```text
   /ip/address add address=192.168.99.1/24 interface=ether5
   /ip/pool add name=pool-mgmt ranges=192.168.99.10-192.168.99.254
   /ip/dhcp-server add address-pool=pool-mgmt interface=ether5 name=dhcp-mgmt
   /ip/dhcp-server/network add address=192.168.99.0/24 gateway=192.168.99.1
   ```
3. Create an interface list for management and add ethernet port to list:
   ```text
   /interface/list add name=oob-mgmt
   /interface/list/member add list=oob-mgmt interface=ether5
   ```
4. Add firewall rule to permit SSH access for management (must go before dropping all non LAN, since the management port is out-of-band - for me this was rule 5 on defconf):
   ```text
   /ip/firewall/filter add chain=input action=accept protocol=tcp dst-port=22 in-interface-list=oob-mgmt src-address=192.168.99.0/24 place-before=5
   ```
5. Add firewall rules to prevent communication with WAN and the rest of the internal network (ensure these rules are *after* the SSH accept rule from above):
   ```text
   /ip/firewall/filter add chain=forward action=drop in-interface-list=oob-mgmt
   /ip/firewall/filter add chain=forward action=drop out-interface=oob-mgmt
   ```

I placed this port off the main bridge as I only ever planned to use this port for router management, therefore I was not concerned about it not having hardware offloading. I did consider keeping it on the main bridge, but I decided that having it off-bridge would be beneficial in case I were to mess up a bridge configuration on my main bridge (and not be saved by safe mode).

Finally, this meant that once I had all my router configuration done, I could reasonably swap over to entirely out-of-band network management if I ever felt the desire to do so.

---

### Hardening IP Services
---
One of the easier ways to improve network security is disabling unused services on the router and ensuring that available services are restricted to being accessed from specific IP ranges.

For myself, since I only intend to access my router configuration through either SSH or WinBox, I disabled api, api-ssl, ftp, telnet, www and www-ssl:
```text
/ip/service disable api,api-ssl,ftp,telnet,www,www-ssl
```

I then restricted access to SSH and WinBox to only the networks that I planned to access them from. Initially, I added more ranges than I intended to end up with just for ease of configuration and removed them later:
```text
/ip/service set ssh address=192.168.10.0/24,192.168.88.0/24
/ip/service set winbox address=192.168.10.0/24,192.168.88.0/24
```

---

### Configuring VLANs
---
Before setting up VLANs on my main bridge, I wanted to do some testing to ensure that I knew how to set everything up properly (VLAN tagging, DHCP pools, inter-VLAN communication rules, access lists, etc.) without risking downtime or creating a configuration headache.

I did this by creating a separate bridge and SSID to mess around with while leaving the rest of my network alone on the main bridge (passphrase as 123 is a placeholder, I only use that on my luggage):
```text
/interface/bridge add name=bridge-sandbox pvid=1 vlan-filtering=yes
/interface/wifi add name=wifi-sandbox datapath=bridge-sandbox security.authentication-types=wpa3-psk security.passphrase=123 master-interface=wifi5g configuration.ssid=wifi-sandbox disabled=no
```

I then added a VLAN and DHCP pool to test with to test with:
1. Create VLAN interface: `/interface/vlan add interface=bridge-sandbox name=vlan-sandbox vlan-id=5`
2. Create IP pool: `/ip/pool name=pool-vlan-sandbox ranges-192.168.5.10-192.168.5.254`
3. Assign IP pool to VLAN interface: `/ip/address add address=192.168.5.0/24 interface=vlan-sandbox`
4. Create DHCP server: `/ip/dhcp-server add name=dhcp-sandbox interface=vlan-sandbox pool=pool-vlan-sandbox disabled=no`
5. Define network for DHCP server: `/ip/dhcp-server/network add address=192.168.5.0/24 gateway=192.168.5.1 dns-server=192.168.5.1`

After doing the above, I tried to connect to my sandbox SSID and was met with no internet connection. I added some logging to see if I was getting to the DHCP handshake at least (`/system/logging add topics=dhcp,debug action=memory`) and immediately noticed that I was seeing an attempt to connect, but never getting to DHCP. I dug around a bit before recalling that I enabled VLAN filtering on my sandbox bridge and forgot to do any tagging for the traffic coming in on that sandbox SSID, so I added it as a datapath:
`/interface/wifi set [find where name=wifi-sandbox] datapath.vlan-id=5`

And voila! Now I was being assigned an IP and I was able to ping other devices on the network. One step in the right direction, however I still did not have WAN access.

As you may have already guessed, I forgot to add the VLAN interface to my LAN and thus any non-ICMP traffic was being dropped by the default firewall rules for being outside of the LAN. All it took to fix this was adding the sandbox VLAN interface to my LAN interface list temporarily:
`/interface/list/member add list=LAN interface=vlan-sandbox`

---

### MAC-based VLAN Steering
---
Now that I had a basic functional setup on my sandbox bridge, I wanted to expand it slightly to test some more functionality of what I was planning to do (specifically, inter-VLAN routing and MAC based access lists for VLAN steering). In order to do this I created and added one more VLAN to the bridge, `vlan2-sandbox`, with a VLAN ID of 6.

I started by testing MAC based VLAN steering by creating an ACL that, in theory, would place my laptop on the VLAN with ID 6 instead of 5, despite the SSID having `datapath.vlan-id=5`:
`/interface/wifi/access-list add action=accept vlan-id=6 mac-address=AA:BB:CC:DD:EE:FF interface=wifi-sandbox`

And that was all it took since ACL rules are checked [before the rest of the interface configuration](https://help.mikrotik.com/docs/spaces/ROS/pages/224559120/WiFi#WiFi-AccessList) for the SSID.

---

### Setting Up Inter-VLAN Firewall Rules
---
Finally, before moving over to working on my proper network, I wanted to ensure that I would have properly sectioned off VLANs, as well as the knowledge of how to allow certain connections between them if need be.

Testing this was fairly easy, since I could simply assign my laptop to one VLAN and try to reach another device anywhere on my network, then implement new firewall rules and check again. Only testing from one subnet to the rest of the network made it simple to ensure that I would not lock myself out with rules for testing. 

First, I tried cutting all communication from my sandbox network to the rest of my LAN:
`/ip/firewall/filter add place-before=13 src-address=192.168.5.0/24 out-interface-list=LAN action=drop chain=forward`

{{< admonition type=note open=true >}}
One minor thing I did to help ensure I would not randomly drop connection (on top of safe mode, etc.) is typing the action for my firewall rule *last* -- this meant I would not accidentally add a more general drop rule than I was intending.
{{< /admonition >}}

I placed this just below the "accept established, related, untracked" forward rule to ensure that established connections would still be allowed, but as expected I was unable to make any of these connections since they were getting dropped by the above rule before being established.

Great! I could still reach the WAN, but I was now unable to connect to any other device on my LAN. Now I wanted to test allowing a specific type of connection through - say, if I wanted to be able to reach the printer from my trusted VLAN (though, in this case, I'd be testing with my sandbox to my printer). Testing this was as simple as creating an accept rule, placed just above my new drop rule, that specifically let TCP connections on port 443 (https) through and attempting to connect to my printer's administrative server:
```text
/ip/firewall/filter add place-before=13 src-address=192.168.5.0/24 out-interface-list=LAN action=accept port=443 protocol=tcp chain=forward
```

I tried connecting on my laptop by navigating to `https://[printer_ip]` and found myself able to connect, with further packets being caught by the "established, related, untracked" rule mentioned earlier. And to ensure that my granular rule was working, I tried connecting to the http version of the page (port 80), and found the connection being dropped by my drop rule, exactly as expected.

Now that I had tested basic inter-VLAN firewall rules for dropping and accepting traffic, I was ready to move on to implementing this on my main network.

---

### Primary Network Implementation
---
Most of this section will be redundant from the above testing, but I'll put a rundown of the steps that I took here as an overview of the process. Essentially all that I had to do was repeat the previous steps, but with my actual network plan in mind (and, of course, on the main network), then troubleshoot any issues.

1. Create VLAN interfaces
   ```text
	/interface/vlan 
	add interface=bridge name=vlan10-mgmt vlan-id=10
	add interface=bridge name=vlan20-work vlan-id=20
	add interface=bridge name=vlan30-trusted vlan-id=30
	add interface=bridge name=vlan40-print vlan-id=40
	add interface=bridge name=vlan50-iot vlan-id=50
	add interface=bridge name=vlan60-voip vlan-id=60
	add interface=bridge name=vlan70-guest vlan-id=70
   ```
2. Create DHCP IP pools
   ```text
	/ip/pool 
	add name=pool-vlan10 ranges=192.168.10.10-192.168.10.254
	add name=pool-vlan20 ranges=192.168.20.10-192.168.20.254
	add name=pool-vlan30 ranges=192.168.30.10-192.168.30.254
	add name=pool-vlan40 ranges=192.168.40.10-192.168.40.254
	add name=pool-vlan50 ranges=192.168.50.10-192.168.50.254
	add name=pool-vlan60 ranges=192.168.60.10-192.168.60.254
	add name=pool-vlan70 ranges=192.168.70.10-192.168.70.254
   ```
3. Assign gateway IPs to VLAN interfaces
   ```text
	/ip/address 
	add address=192.168.10.1/24 interface=vlan10
	add address=192.168.20.1/24 interface=vlan20
	add address=192.168.30.1/24 interface=vlan30
	add address=192.168.40.1/24 interface=vlan40
	add address=192.168.50.1/24 interface=vlan50
	add address=192.168.60.1/24 interface=vlan60
	add address=192.168.70.1/24 interface=vlan70
   ```
4. Create DHCP servers for VLAN interfaces
   ```text
	/ip/dhcp-server 
	add name=dhcp-vlan10 interface=vlan10 pool=pool-vlan10
	add name=dhcp-vlan20 interface=vlan10 pool=pool-vlan20
	add name=dhcp-vlan30 interface=vlan10 pool=pool-vlan30
	add name=dhcp-vlan40 interface=vlan10 pool=pool-vlan40
	add name=dhcp-vlan50 interface=vlan10 pool=pool-vlan50
	add name=dhcp-vlan60 interface=vlan10 pool=pool-vlan60
	add name=dhcp-vlan70 interface=vlan10 pool=pool-vlan70
   ```
5. Define network for DHCP clients
   ```text
	/ip/dhcp-server/network 
	add address=192.168.10.0/24 gateway=192.168.10.1 dns-server=192.168.10.1
	add address=192.168.20.0/24 gateway=192.168.20.1 dns-server=192.168.20.1
	add address=192.168.30.0/24 gateway=192.168.30.1 dns-server=192.168.30.1
	add address=192.168.40.0/24 gateway=192.168.40.1 dns-server=192.168.40.1
	add address=192.168.50.0/24 gateway=192.168.50.1 dns-server=192.168.50.1
	add address=192.168.60.0/24 gateway=192.168.60.1 dns-server=192.168.60.1
	add address=192.168.70.0/24 gateway=192.168.70.1 dns-server=192.168.70.1
   ```
6. Assign VLANs to bridge interface
   ```text
	/interface/vlan 
	add name=vlan10 vlan-id=10 interface=bridge
	add name=vlan20 vlan-id=20 interface=bridge
	add name=vlan30 vlan-id=30 interface=bridge
	add name=vlan40 vlan-id=40 interface=bridge
	add name=vlan50 vlan-id=50 interface=bridge
	add name=vlan60 vlan-id=60 interface=bridge
	add name=vlan70 vlan-id=70 interface=bridge
   ```
7. Add VLANs to LAN interface list
   ```text
	/interface/list/member 
	add list=LAN interface=vlan10
	add list=LAN interface=vlan20
	add list=LAN interface=vlan30
	add list=LAN interface=vlan40
	add list=LAN interface=vlan50
	add list=LAN interface=vlan60
	add list=LAN interface=vlan70
   ```
8. Configure SSIDs
   ```text
	/interface/wifi 
	add name=wifi-main2.4g master-interface=wifi1 security.authentication-types=[auth-types] security.passphrase=[passphrase] configuration.ssid=wifimain datapath.bnidge=bridge datapath.vlan-id=30
	add name=wifi-main5g master-interface=wifi2 security.authentication-type=[auth-types] security.passphrase=[passphrase] configuration.ssid=wifimain datapath.bnidge=bridge datapath.vlan-id=30
	
	add name=wifi-iot2.4g master-interface=wifi1 security.authentication-types=[auth-types] security.passphrase=[passphrase] configuration.ssid=wifiiot datapath.bnidge=bridge datapath.vlan-id=50
	add name=wifi-iot5g master-interface=wifi2 security.authentication-type=[auth-types] security.passphrase=[passphrase] configuration.ssid=wifiiot datapath.bnidge=bridge datapath.vlan-id=50
	
	add name=wifi-guest2.4g master-interface=wifi1 security.authentication-types=[auth-types] security.passphrase=[passphrase] configuration.ssid=wifiguest datapath.bnidge=bridge datapath.vlan-id=70
	add name=wifi-guest5g master-interface=wifi2 security.authentication-type=[auth-types] security.passphrase=[passphrase] configuration.ssid=wifiguest datapath.bnidge=bridge datapath.vlan-id=70
   ```
9. Configure MAC-based access list rules (e.g. placing printer onto VLAN40)
   ```text
	/interface/wifi/access-list/add action=accept mac-address=AA:AA:AA:AA:AA:AA interface=wifi-iot2.4g vlan-id=40
   ```
10. Add firewall rule to prevent inter-VLAN routing (placed only above other drop rules)
    ```text
	/ip/firewall/filter add chain=forward action=drop in-interface-list=LAN out-interface-list=LAN place-before=15
    ```
11. Add firewall rule(s) to allow specific inter-VLAN routing (e.g. printing if printer IP is static at 192.168.40.2 - this rule will allow all traffic through *to* the printer instead of specific protocols, but only established/related will be allowed back, printer cannot establish the connection)
    ```text
	/ip/firewall/filter add chain=forward action=accept in-interface-list=print-allowed dst-address=192.168.40.2
    ```

Finally, all I had to do was enable VLAN filtering on my main bridge to see if everything would work properly. I took backups before this, of course, and was considering running a script to auto-load a backup on a timer, since I knew I'd be force disconnecting and thus could not rely on safe mode, however I decided against it since I had my backup of my off-bridge management port just in case.

Overall it went smoothly. My WiFi networks were functioning, my inter-VLAN blocking was exactly as expected, and I think it went about as well as I could have expected. I'd love to say everything went perfectly, but of course, that is not quite the case!

---

## Issues Faced and Solutions
---

### VLAN Rules Not Auto-Generating for Wired Connection
---
One of the first issues I faced upon applying my network changes to my main network was that my main PC disconnected and was not being given a new IP. Turns out I had forgot to set the PVID for my main ethernet connection - whoops!

Good thing for the management port, I was able to do this without trouble. I set the PVID of my ether2 port to 30 so my PC would land on my trusted network and checked `/interface/bridge/vlan` to see if all my tagging was set up properly and noticed VLAN30 missing from the "added by VLAN on bridge" section of untagged traffic. 

I don't exactly know what caused this, but me manually adding and then removing this tagging rule seemed to fix it, I can imagine a quick router restart probably would have done the same.

---

### TV/IoT Issues with 5GHz Channels
---
Another thing I noticed with my main network from the moment I got this new router was that my TV was *refusing* to connect to my 5GHz SSIDs. In fact, it wasn't even showing them as available.

I was curious as to why and, during my research, learned about [dynamic frequency selection (DFS)](https://en.wikipedia.org/wiki/Dynamic_frequency_selection) channels. Essentially, there are specific channel ranges that have varying mandates enforced by governing bodies to allow consumer access points (APs) to utilize some frequencies (specifically, 5250-5350 and 5470-5725 MHz) with the specification that they must adhere to DFS standards wherein, if radar signals are detected by the AP, it broadcasts a switch-channel event and automatically switches the channel outside of the range in order to not interfere.

Why does this matter? Well, this means that devices either need to be DFS compliant (e.g. be able to perform a quick channel swap if presented with a switch-channel request from an AP), or refuse to join networks on DFS channels. As it turns out, my router's default config was set to include DFS channels which is fine in general, however it was causing this specific issue for me, so I configured my 5g WiFi band to skip DFS channels `/interface/wifi/set name=wifi2 channel.skip-dfs-channels=all`

This led me into some frequency range optimization...

---

### Network Congestion
---
One thing I had done on some past routers that I wanted to repeat at some point on this one was optimizing channel selection for both my 2.4 and 5GHz bands. I am fortunate to not have too much in the way of overcrowded WiFi channels near me, but I found this a great opportunity to test out some of the tools that MikroTik provides.

Specifically, MikroTik has two main tools that are very useful for frequency testing: frequency monitor and snooper. Both are accessible from the command line but I personally used the UI for these as I preferred the look. These tools will render the interface that you run them on offline during their use, so be aware that you'll be interrupting your network, but if you can run these at a few different times during the day to gather profiles of the most and least used channels in your area, it can provide invaluable information to clear up potential network congestion.

For myself, the 2.4GHz band was relatively similar across every channel, so I left it alone. The 5GHz band, however, had a very clear frequency range winner of 5150-5240 (outside of the DFS channel ranges), so I set my parent 5GHz interface to use that and have had no network congestion issues since.

To note, this will not necessarily work well for everyone. Most routers will dynamically select channels by default and their methods for doing so vary. It can be assumed that most neighboring networks will likely not be configured manually and therefore may change to the channels that you are using at some point, bringing potential congestion back. It works well for my case, but it may be best to leave the wider range in other cases.

---

### Forcing Printer on 2.4GHz Channel
---
What would network setup be without a printer to cause a headache or ten?

Finally I thought I had everything working well, all that I had to test still was the printer (saved the best for last). I ran into a multitude of issues trying to get my printer to connect to the network properly. The printer itself was seeing excellent connection strength, but every time I tried to connect the printer thought it did and my router disagreed. No actual network connection was being established. Great.

What had I changed? Well, the main thing was that my IoT SSID was now dual-band, instead of just being the default separate SSID for one 5GHz and one 2.4GHz band as it was before. I tried to force my printer onto the 2.4GHz band in a few ways - access list rules, printer network configs, temporarily disabling the 5GHz band on the SSID, etc. but none of them stuck.

As of now I am simply broadcasting one more SSID, 2.4GHz only, just for my printer. I will be searching for a more elegant solution in the future, but the printer wins this time. At least it works, for now...

---

## Closing Thoughts
---
Overall this was a fantastic learning experience and introduction to the world of MikroTik devices. I am very happy with my home network setup now and feel much more secure having full control over every part of it - not just what the vendor feels is relevant for their consumers to have access to change. I absolutely love how configurable everything within the MikroTik ecosystem is and RouterOS feels very comfortable for someone used to a Linux terminal.

I am looking forward to digging deeper into the capabilities of this device including scripting, container hosting (though I'll likely keep this on separate devices myself), etc. and assuredly getting my hands on some more MikroTik products in the future to mess around with (albeit most probably in a lab setting instead of directly on my shared home network).

-ic3sec

---