---
weight: 4
title: "Haunted Pumpkin CTF (2025)"
date: 2025-11-05T07:28:00-00:00
draft: false
author: "ic3sec"
authorLink: ""
description: "Writeup for OSINT Switzerland's 2025 Haunted Pumpkin CTF"
images: []

tags: ["CTF", "OSINT", "Writeup", "2025"]
categories: ["Writeups"]
---

## Overview
This was a fun OSINT CTF organized and hosted by the great team over at [OSINT Switzerland](https://osintswitzerland.ch/). Contestants were given 30 hours to complete a multitude of OSINT challenges and could sign up solo or with a partner.

This was my first OSINT CTF and I found out about it last minute, so I went about it solo. I'll be covering my solutions, thought process and some retrospect on the challenges. There are plenty of spoilers ahead, but all the context you need for (most) challenges are at the start of their respective sections, so if you want to try them before looking at solutions please do!

**DISCLAIMER**: These challenges were made for educational purposes **only** and should not be used for malicious purposes.

## Challenges
### Introduction
---
>Your entry flag, brave investigator, is the key into the Realm of the Pumpkin… 🎃🔑 🎃🎃🎃 hpCTF{Th4nk5_4_F0ll0w1ng_Th3_Ru135} 🎃🎃🎃

Starting off difficult, we're tasked with the ultimate challenge: reading. Jokes aside, this flag was used as a rule agreement, and could be found at the end of the text.

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{Th4nk5_4_F0ll0w1ng_Th3_Ru135}
{{< /admonition >}}

---

### Horror Clowns
---
>A few years ago, there were some people dressing up as Horror Clowns, showing up out of nowhere scaring the hell out of people as can be seen in the following staged video. Link: https://www.youtube.com/watch?v=vaXjN3zChyE
>
>Can you find the name of the train station at minute 01:56?
>
>Flag format: hpCTF{station name}
>
>Flag example: hpCTF{******* ***********}

We've been provided a link to a YouTube video of scary clowns and are tasked with finding the name of the train station at which one particular prank was being performed.

I was unable to solve this particular challenge in time, but I found it after and got confirmation on the flag. Turns out I may need to brush up on some basics...

{{< admonition type=abstract title="Steps" open=false >}}
1. Navigate to the provided video and find a suitable frame (looking for good image context), one can be found at [2:02](https://youtu.be/vaXjN3zChyE?si=Unz7sJ8c84z2sAnY&t=122)
![Train Station Frame](001_horror_clowns.png)
2. Reverse image search the frame and look for any links which may give information on the location
![Train Station Google Reverse Search](002_horror_clowns.png)
3. Search through sites found from reverse searching to find the station name
![Train Station Name](003_horror_clowns.png)
4. Confirm station details by cross referencing with other frames from the video -- in this case, by searching the assumed station name from the previous step, we can find other images that show the bridge we can see at [1:59](https://youtu.be/vaXjN3zChyE?si=Unz7sJ8c84z2sAnY&t=119) in the video
![Train Station Bridge Comparison](004_horror_clowns.png)
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{Perugia Silvestrini}
{{< /admonition >}}

---

### My Identity
---
>We recorded a grave raider last night.
>
>Every information that can lead finding this person is a big help.
>
>Maybe start with the birthday?
>
>Flag Format: hpCTF{YYYYMMDD}
>
>Flag Example: hpCTF{19900615}
>
>Challenge image: ![My Identity](001_my_identity.png)

We've been tasked with finding the birth date of the grave raider in the image.

This one threw me for a loop at first. I overcomplicated it and went down rabbit holes that need not exist. I looked into swiss sample passports, found similar examples and even tried to figure out [how to check date of birth from a Swiss EID](https://www.ausweisapp.bund.de/en/online-identification/what-you-need). After a much needed "break" (doing other challenges) I came back to find the answer staring me right in the face.

{{< admonition type=abstract title="Steps" open=false >}}
1. Zoom in on the briefcase in the image to find a Swiss passport format
![Zoomed Briefcase](002_my_identity.png)
2. Read the date of birth information at the start of the passport in YYMMDD format: 640812
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{19640812}
{{< /admonition >}}

---

### My Identity - 2
---
>Great, the birthday is a good start!
>
>But tell me, belongs this passport really to the man in the image?
>
>Answer with yes or no.
>
>Flag Format: hpCTF{<yes|no>}

Now we must identify whether or not the passport in the image belongs to the *man* committing the robbery.

During my research on Swiss passport formats in the first part of this challenge, I learned that there is a gender identifying character included as well, as noted in the [MRZ TD1 travel document format](https://www.doubango.org/SDKs/mrz/docs/MRZ_formats.html).

{{< admonition type=abstract title="Steps" open=false >}}
1. This question asks us specifically if the pictured passport belongs to the **man** in the image 
2. Looking back at the passport, we can see that the character in the 8th position is an "F", which signifies that the owner of this passport is female and therefore not **his** passsport

![Passport Gender Character](001_my_identity_2.png)
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{no}
{{< /admonition >}}

---

### Ghost Ship
---
>A Ghost Ship was found ashore.
>
>Can you find the IMO number of the ship.
>
>Flag Format: hpCTF{IMO_nr}
>
>Challenge image: ![Ghost Ship](001_ghost_ship.png)

We've been tasked with finding the IMO number of the ship in the image.

This time around my searching skills didn't prove to be quite as rusty (unlike our ghost ship).

{{< admonition type=abstract title="Steps" open=false >}}
1. Quick Google reverse image search brings us to the MV Alta
![Ghost Ship Image Search](002_ghost_ship.png)
2. Looking at the Wikipedia page we can find the IMO number for the ship in question
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{7432305}
{{< /admonition >}}

---

### Ghost Ship - 2
---
>Can you also find out where the ship currently is ?

Let's find out.

{{< admonition type=abstract title="Steps" open=false >}}
1. Doing some quick Google dorking with "MV Alta" and "location", we can find this resource: https://www.theultimateroadtripresource.com/location/discover-new-adventures/ghost-ship-mv-alta
2. Coordinates listed on the site are 51.8091707, -8.0657690 -- plug these into Google maps and we can see the wreckage nearby
![Ghost Ship 2 Map](001_ghost_ship_2.png)
3. Grab the coordinates from the location as marked on Google maps
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{51.81125219988279, -8.056740104712809}
{{< /admonition >}}

---

### Ghost in the Inbox
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Ghost in the Inbox - 2
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Peekaboo
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Dude Where Was This Car
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Fake News
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Fake News - 2
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Distress Call
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Distress Call - 2
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Forgotten Helter-Skelter
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Pumpkin Ping
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Wiggler
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### I Like Trains
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Friday 13
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Friday 14
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Misunderstanding
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Blair Witch Resident
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Three Word Nightmare
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Twin Sister
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Follow The Money
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Halloween Party
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Follow The Car
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

## Final Thoughts
---
At the end of it all I placed at rank 23 out of ~800+ participants. I was able to maintain a top 10 position for the majority of the first ~16 hours -- not bad for a first timer! I believe I ended up being tied for first place in terms of solo players, but I'm sure many more experienced solos could have done much better than I did. Next time, I think I'll try to find a teammate. There are so many moments where having another perspective is truly invaluable.

Overall I really enjoyed this as my first foray into OSINT CTFs. Huge thanks to the team over at OSINT Switzerland for hosting and the members of their Discord community for being so welcoming and friendly - this is a truly supportive group that anyone with an interest can find their place in, no matter how (in)experienced.

Thank you for reading and have a great rest of your day/night!

-ic3sec

---