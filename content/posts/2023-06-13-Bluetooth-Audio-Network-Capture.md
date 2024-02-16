title = "Bluetooth Audio Device Network Capture"
date = "2023-06-12"
summary = "We have a Bluetooth Network Capture as a challenge and we need to follow it's movement to get the message" 
tags = ["CTF","DFC" , "Network" , "Forensics" , "Packet Capture"]

## Description : 

We are given a .pcap [file](https://github.com/blueee04/DFC-writeups/raw/main/evidence2/evidence2.pcap) which looks like a network capture of a audio device. So for this we import the file into WireShark to analyze it. Being a Bluetooth Audio Device let’s analyze the RTP(Real-time Transport Protocol) stream of the capture. So we get to Telephony > RTP > RTP Streams.

## Tools Used:
* WireShark
* Sonic Visualizer
* Morse Code Translator

## Method
[]()
There we can dump out the audio stream as a .wav to analyse it.
[]()
We import this wav file into Sonic Visualizer to analyze it further.
[]()
Analyzing it further we can see we have a morse code…decoding it further we have a string as follows
[]()