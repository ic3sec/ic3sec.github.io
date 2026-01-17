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

This was my first OSINT CTF and I found out about it last minute, so I went about it solo. I'll be covering my solutions, thought process and some retrospect on the challenges. There are plenty of spoilers ahead, but all the context you need for (most) challenges is at the start of their respective sections, so if you want to try them before looking at my solutions please do!

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

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
I knew this one had to be related to reverse image searching, but I just could not get an image that would take properly during the challenge. I kept finding more references to the exact same creator's videos but nothing about the station itself.

Admittedly, I got slightly frustrated with myself for not being able to find the answer to what appeared to be one of the *simplest* challenges of the competition.

As it turned out, all I needed to do was add some more context to the image - I was intentionally trying to keep the people out of it, cut off what I thought would be irrelevant at the bottom of the image, etc. when all I had to do was take a full screenshot of the video.

Lesson learned: Persistence and changing images (+ including maximum image context) when reverse searching is of great importance.
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

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
This one stumped me for a good while, partially due to not knowing the Swiss passport format, but how did I get to thinking that in the first place? 

Well... unfortunately, that started with me not reading the number on the photo properly. I thought it read:
- 64C0121F2801012CHE
- 64CO121F2801012CHE
- 6406121F2801012CHE

All well before I figured out that it was 640812. Up until the last one, where I realized that it may have been a birthday at the beginning of the string, I went down some deep rabbit holes of Swiss sample IDs and trying to associate this exact ID number to something I thought seemed relevant.

Lesson learned: Sometimes you just need to take a step back and reconsider simple solutions (but at the same time, getting stuck truly teaches you more than already knowing ever could).
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

### Dude Where Was This Car
---

>You leave the tunnel where the car enters at the end of the clip: https://www.youtube.com/watch?v=j6-yVoJTCo8
>
>There is a little public rest room with a green roof at the parking lot on the left side (Year 2021).
>
>A few meters further (also on the left side) there is a sign which was build between 2008 and 2009.
>
>What does the sign say?
>
>Flag Format: hpCTF{***** ***** **}

Film locations are pretty likely to be publicly known information, but how can we locate the exact tunnel?

{{< admonition type=abstract title="Steps" open=false >}}
1. A quick Google search for "The Car 1997 film location" mentions a few spots, but the most mentioned appears to be Zion National Park, Utah
2. Google searching for "tunnels at Zion National Park" and checking out Google maps lands us at the Zion-Mount Carmel Tunnel

![Zion-Mount Carmel Tunnel Map](001_dude_where_was_this_car.png)

3. Going to street view on the marker conveniently places us at one of the tunnel entrances - and the images just so happen to be from April 2008 with a sign in sight

![Zion-Mount Carmel Tunnel Street View](002_dude_where_was_this_car.png)
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
I got pretty lucky with this one as my immediate thought about Google searching the movie location produced exceptionally accurate results.

If that had not worked, however, I would have gone on to try and find a good frame of the video to screenshot and perform some reverse image searching instead.
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{SPEED LIMIT 25}
{{< /admonition >}}

---

### Ghost in the Inbox
---
>Oh-Oh, it seems like someone tries to phish people in our name 🤨😮
>
>Can you help us figure out who really sent this email?
>
>Flag Format: hpCTF{************************}
>
>[Challenge File](OS_Discount.eml)

{{< admonition type=abstract title="Steps" open=false >}}
1. We've been given a .eml file, which is one the most used raw file formats for emails - all we need to do is be able to read it
2. Upload the given file to any eml reader, such as https://www.emlreader.com/ and look for the **From Address** to reveal the true sender
![EML File Contents](001_ghost_in_the_inbox.png)
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{ctf{?@}cultoftherabbit.team}
{{< /admonition >}}

---

### Ghost in the Inbox - 2
---
>Nice job finding the real sender of the email!
>
>Now we need more information about the domain and where it comes from. Do you know where to look?
>
>Maybe you have to dig deeper ...
>
>the flag reveals itself once you are on the right spot (in other words, its obvious when you find it)
>
>Flag Format: hpCTF{\<flag\>}

Domain related challenges often have to do with information being hidden in DNS records and this one gives us a (maybe not so) subtle hint about *digging*.

{{< admonition type=abstract title="Steps" open=false >}}
1. Look for DNS records of the sender's domain (cultoftherabbit.team) using your favourite search tool, just make sure it supports TXT records (e.g. https://dnsdumpster.com/) or use the dig utility `dig cultoftherabbit.team TXT`
{{< admonition type=note >}}
DNS records can be altered or removed, if you do not find the flag in TXT records it is likely no longer being hosted as the challenge is over. They are an important place to check, but not the only place! It's worth a quick look over other DNS record types as well for this style of challenge.
{{< /admonition >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
I started by looking into the domain with a whois search of the domain associated to the sender: https://www.whois.com/whois/cultoftherabbit.team

![whois results](001_ghost_in_the_inbox_2.png)

I noted that their name severs were under the .de domain, which is from Germany, and tried submitting that since the question mentioned information about **where** it comes from.

However, that isn't exactly an *obvious* flag, unlike the final solution.

At this point I was stumped for a while, but ended up moving on to try other tools, including the [whois domain tools](https://whois.domaintools.com/cultoftherabbit.team) which still landed me in Germany, but more specifically in Berlin, which I again tried using desite the fact that Berlin wouldn't be any more obvious an answer than Germany.

After some more searching I found [dnsdumpster](https://dnsdumpster.com/) which, unlike the previous tools I tried using, supports TXT records -- an important difference that resulted in finding the **significantly** more obvious flag.

Next time, maybe I should just use dig...

Lesson learned: Pay attention to the question phrasing literally, *obvious* generally cannot depend on external/unrelated knowledge.
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{TH1S_15_Y0UR_DNS_F14G}
{{< /admonition >}}

---

### Peekabo
---
>The barcode for this parcel was placed on a really weird place...
>
>Where do you have to send this parcel so it finds its original destination?
>
>Flag Format: hpCTF{\<ZIP\>\_\<City\>\_\<Street\>\_\<Number\>}
>
>[Challenge File](weirdlylargefile.png)

Nothing jumps out in the image itself, but that doesn't mean the file isn't hiding something, especially given that intriguing file name.

{{< admonition type=abstract title="Steps" open=false >}}
1. Extract the embedded file using Binwalk or a similar tool - there are plenty of online options as well, e.g. https://www.unroll.ing/
2. We should get a .bin file after extracting, which we can change over to the .png file type in order to view as an image
![Extracted Image](001_peekabo.png)
3. Now that we have a barcode, we can put read it using an online tool e.g. https://online-barcode-reader.inliteresearch.com/
4. This gives us `CHE-274.572.141` which definitely does not look like an address at first glance, but a quick Google search returns results that map it to our answer

![Address Search Result](002_peekabo.png)
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
I did not manage to finish this one in time, but I researched it out after the fact.

I expected some form of file stuffing given the file name `weirdlylargefile.png` and I first tried checking it out using exiftool, which showed that there was data trailing the PNG IEND chunk.

Only issue at that point was I did not know how to properly extract it. Hexdump didn't seem to get me anywhere - the data after the IEND appeared frivilous to me. Exiftool gave me the file description as well though, which confirmed my thoughts: "hunger, ate a second file"

I ended up moving on to other challenges unsure how to extract it, but after the competition ended I read into file exfiltrating software like [binwalk](https://github.com/ReFirmLabs/binwalk) (+ online equivalents) and was able to go from there.
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{6003_Luzern_Sagenmattstrasse_7}
{{< /admonition >}}

---

### Fake News
---
>Fake news like these spread more wildely than ever.
>
>Can you find out who was actually arrested here?
>
>Flag Format: hpCTF{**** *** ***-*****}
>
>Challenge image: ![Fake News](001_fake_news.png)

{{< admonition type=abstract title="Steps" open=false >}}
1. Reverse searching the right side of the image brings up articles with images of another person (https://info.51.ca/articles/1005978)
![Article Image](002_fake_news.png)
2. Translating the headline of the article above gives us the partial name "Law Wai-kwong", but this doesn't quite match the flag format
3. Google searching with the partial name returns results with the full name (our flag) and confirms the article image we found earlier
![Google Search Result](003_fake_news.png)
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}

{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{Ryan Law Wai-kwong}
{{< /admonition >}}

---

### Fake News - 2
---
>Can you also find which age Ryan was when he was arrested?
>
>Flag Format: hpCTF{**}

{{< admonition type=abstract title="Steps" open=false >}}
1. Since we know his name, a simple Google search of "Ryan Law Wai-kwong age during arrest" gives us the result we're trying to find
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{46}
{{< /admonition >}}

---

### The Distress Call
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Distress Call - 2
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Forgotten Helter-Skelter
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Pumpkin Ping
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Wiggler
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### I Like Trains
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Friday 13
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Friday 14
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Misunderstanding
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Blair Witch Resident
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Three Word Nightmare
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Twin Sister
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Follow The Money
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### The Halloween Party
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
{{< /admonition >}}

{{< admonition type=warning title="Flag" open=false >}}
hpCTF{}
{{< /admonition >}}

---

### Follow The Car
---

{{< admonition type=abstract title="Steps" open=false >}}
{{< /admonition >}}

{{< admonition type=tip title="Thought Process/Retrospect" open=false >}}
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