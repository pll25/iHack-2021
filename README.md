# iHack-2021
Write-up that I promised for the WireShark track for iHack 2021 CTF -- PCAP Exploration Par 1 & 2

### What you'll need :
-Wireshark
-Binwalk

### PCAP Exploration part 1 CTF Description : 

> " A file is hidden in this pcap, can you recover it? "

We are provided with a pcap file which is obviously a Wireshark Capture.
> See file 01.pcap

### Bite the wires.
First, let's open it and see what we can do around this.

My way of doing things with wireshark CTF is always to first open it and check for general patterns in the packets.
Let's say you would have multiple TCP, then multiple UDP and it comes back in the same order 2-3-4 times in the capture, it means there's a pattern and could be interesting in a CTF to understand it.

For this one specifically, there didn't seem to be anything.
I notice there was some SSH packets (only 2), which I've found interesting at first. Upon looking into it I couldn't see anything interesting about those.
Also, lots of TCP re-transmits (which is garbage), but I also notice that there is no encrypted packets.

As the description of the challenge is pretty straightforward, I started to look for files in the capture. 
I already doubted the files would be in the TCP part where there is no retransmission.

### How to find it ?

I spoke about patterns before. One pattern I really like to look at in wireshark is the size of the packets.
It gives a lot of information about ''anomalies'' in the packets which is usually good in CTFs. Also, it could be good in real life for multiple things (let's say DNS exfiltration, the packets would be bigger).

Looking at the packet lengths we can find that : 
> All TCP packets are 480 bytes or under, except some (starting at packet #50).
This gave me an indication that our ''file'' would be somewhere around these.

![packet-length](https://user-images.githubusercontent.com/16509773/122678081-35638d80-d1b3-11eb-83b9-d2403ddfc9f0.jpg)

Let's see what we can find in the first big Packet. The packet #50.
(Double click it to see raw packets).

As we know that we need to find a ''file'' only the Data part of the packet would be interesting to us. The data is Raw, but we can change it to HexDump.
At first, I see that the Data of the packet is starting with 7z. Well... it's a magic byte! Guess what the letters 7z, are the beginning headers of ... you guessed it a 7zip archive!!!
We now have the beginning our file. Let's see how we can extract it.

### Follow the conversation to find more.
I wrote earlier that you could see the data in HexDump or other formats. 
But in order to do this, you will need to follow the conversation to find the other parts of our file.

Right click packet #50 and hit ''Follow'' -> ''TCP Stream''.
A window will popup, and you will notice that in the background wireshark window you will have at new filter applied. It should look like :
> tcp.stream eq 11

![packet-bytes](https://user-images.githubusercontent.com/16509773/122678317-4cef4600-d1b4-11eb-8ba2-b8e25ca977b7.jpg)

If look closely our Data is shown as ASCII, but we can convert it the way we want (RAW, HexDump, YAML etc).
Usually, I often get problems when I try to extract it as raw with wireshark (I think it leaves the packet headers or to much spaces and breaks the file).

In CTF I really like to work with HexDump, as it is easier to see what type of file it could be, if there are missing parts or other stuff.
Let's see how it goes :
![magic-bytes](https://user-images.githubusercontent.com/16509773/122678418-cb4be800-d1b4-11eb-9218-b207ef0f1368.jpg)
> See that we have the exact matching magic bytes of a 7zip archive.

When I see this, I take the whole package and go to cyberchef.
CyberChef let me see if the file is corrupted in anyway, we can search the data and we can search around for hidden stuff.
> https://gchq.github.io/CyberChef/

Let's hit CTRL+A and CTRL+C To select the whole HexDump and paste it to CyberChef.
When we paste in CyberChef **it already magically tells us that we have an HexDump**.
If we hit the magic wand it will apply the ''From HexDump'' recipe.

When we do this **it magically finds a 7zip archive file**.

We can take the output and it the save button, let's save hit as a .7z as we know it's a 7z file.

When we open it, we find a new PCAP and the file where.txt that contains our **First Flag**!!
![where](https://user-images.githubusercontent.com/16509773/122678697-e2d7a080-d1b5-11eb-944d-a390fbedd201.jpg)

### Flag 2 is harder
Part 1 of the challenge is owned. Now let's go to Flag2. Which is... basically the same thing for the wireshark part.
Let's extract and open the ''pcap.pcap'' file that we have.

We can now see that the packets are not recognized as TCP from wireshark. They are either UDP or IPv4.
I still see some SSH and retransmissions which are not interesting today as we had also in part 1.

The patterns are harder to find in thise one. However, we have some big packets either UDP or Ipv4 that looks promising.

At this point I assumed that we would search for a file... again, but this was an assumption.

Looking at packet #37, we have the start of a PKIZP archive, with the word flag.txt. Well I guess I'm at the right place.
> ··09··M"PK···   · ···R_····   z·  ·   flag.txt

Let's do the same thing as before all over again. Unfortunately we can't follow the IPv4 packets as they are not resolved.
Let's see what we can get with the UDP ones, we now know that we have an archive containing flag.txt.

> Right CLick any UDP packet --> Follow UDP Stream

Bingo!
The RAW Data starts with PK (magic Bytes for an archive and are followed flag.txt) which means that the archive might contain a file named flag.txt.

Exporting it as HexDump as before... leave it some time, this file looooookkksss huuuuugeeeee.
Paste it in Cyberchef: 
The magic Wand already tells us that a PKZIP file archive is detected.
![magicwand](https://user-images.githubusercontent.com/16509773/122679364-8924a580-d1b8-11eb-98ef-b867569e3ec6.jpg)

Let's extract it (take the output and press ''Save Output to File'' as .zip archive.

Dang!
**When we try to open it with Windows archive manager it says the file is incorrect.**

I know the flag is in it!!!!!! I've seen it flag.txt.......

At this point I have messed around a bit with the file. While messing around I've found that : 
> Windows can't open it. However 7zip does.

Let's see what 7zip as in store for us : 
When we open it we see flag.txt, and logo-ihack-2020.png.
As soon as I've seen this I was super happy, and hit FLAG.TXT to find it !

Here was my deception :
![7zip1](https://user-images.githubusercontent.com/16509773/122679530-3dbec700-d1b9-11eb-9206-9c9898f1c946.jpg)

At this point I have messed around (like more than an hour) trying to find out if there was anything hidden in the png image.
It didn't have anything in store for me.

I also found out that the file was big for a simple archive with 2 files. 
I decided to send it back to CyberChef and see what was going on... 

Upon doing this I've seen that there was multiple other files in the archive.
(One good trick to find this in a CTF is that in an archive file you can see the list of files it contains at the end).

In this particular case here's what I had :
![more-in-store](https://user-images.githubusercontent.com/16509773/122679673-ce95a280-d1b9-11eb-9f61-b5c334bbe66c.jpg)

### Would it be the time to take a walk ?
I think yes! Let's ask binwalk to find if it can help uncover anything hidden.
![binwalkies](https://user-images.githubusercontent.com/16509773/122679891-af4b4500-d1ba-11eb-8aeb-d1718a62aa24.jpg)

Of course I exctracted everything in it and there was wayyyyyy more than the first archive we had.

One of the files in the main folder contained the 2nd flag the hackfest logo. (Logo.jpg).
![flag2-logo](https://user-images.githubusercontent.com/16509773/122679963-0bae6480-d1bb-11eb-935f-4d5ef2d55a1f.jpg)

At the end, I've spent few hours looking at all the files and trying to find so hard to find this flag that I've missed the obvious logo right in the first folder and spent like 2 hours on it without finding it.

It was still a great challenge & A great day! 
Thank you all the organizers and see you at HackFest, or maybe before :)

Best regards!
--pll25







