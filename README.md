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

I spoke about patterns before. One pattern I really like to look at in wireshark is the size of the packts.
It gives a lot of information about ''anomalies'' in the packets which is usually good in CTFs. Also, it could be good in real life for multiple things (let's say DNS exfiltration, the packets would be bigger).

Looking at the packet lengths we can find that : 
> All TCP packets are 480 bytes or under, except some (starting at packet #50).
This gave me an indication that our ''file'' would be somewhere around these.

![packet-length](https://user-images.githubusercontent.com/16509773/122678081-35638d80-d1b3-11eb-83b9-d2403ddfc9f0.jpg)

Let's say what we can find in the first big Packet. The packet #50.
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
Usually, I often get problems when I try to extract it as raw with wireshark (I think it leaves the packet headers and breaks the file).

In CTF I really like to work with HexDump, as it is easier to see what type of file it could be, if there are missing parts or other stuff.
Let's see how it goes :
![magic-bytes](https://user-images.githubusercontent.com/16509773/122678418-cb4be800-d1b4-11eb-9218-b207ef0f1368.jpg)
> See that we have the exact matching magic bytes of a 7zip archive.

When I see this, I take the whole package and go to cyberchef.
CyberChef lets me see if the file is corrupted in anyway, we can search the data and we can search around for hidden stuff.
> https://gchq.github.io/CyberChef/

Let's hit CTRL+A and CTRL+C To select the whole HexDump and paste it to CyberChef.
When we paste in CyberChef **it already magically tells us that we have an HexDump**.
If we hit the magic wand it will apply the ''From HexDump'' recipe.

When we do this **it magically finds a 7zip archive file**
We can take the output and it the save button, let's save hit as a .7z as we know it's a 7z file.

When we open it, we find a new PCAP and the file where.txt that contains our **First Flag**!!
![where](https://user-images.githubusercontent.com/16509773/122678697-e2d7a080-d1b5-11eb-944d-a390fbedd201.jpg)











