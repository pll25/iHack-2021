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
