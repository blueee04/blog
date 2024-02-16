title = "Bluetooth Keyboard Network Capture"
date = "2023-06-12"
summary = "We have a Bluetooth Network Capture as a challenge and we need to follow it's movement to get the message" 
tags = ["CTF","DFC" , "Network" , "Forensics" , "Packet Capture"]

## Description

We have another .pcap file[](https://github.com/blueee04/DFC-writeups/raw/main/evidence3/evidence3.pcap) over here which looks like a Bluetooth keyboard capture…so we take the data from the capture and pair it with their respective USB codes to generate the message typed on the keyboard.

## Tools:
* Python
* Tshark

## Code :
```py
import os
output3= os.popen("tshark -r evidence3.pcap -Y \"btatt && btatt.handle == 0x0018\" -T fields -e \"btatt.value\"").readlines()

usb_codes = {
    "0x04":['a','A'],"0x05":['b','B'], "0x06":['c','C'], "0x07":['d','D'], "0x08":['e','E'], "0x09":['f','F'],"0x0A":['g','G'],"0x0B":['h','H'], "0x0C":['i','I'], "0x0D":['j','J'], "0x0E":['k','K'], "0x0F":['l','L'],"0x10":['m','M'], "0x11":['n','N'], "0x12":['o','O'], "0x13":['p','P'], "0x14":['q','Q'], "0x15":['r','R'],"0x16":['s','S'], "0x17":['t','T'], "0x18":['u','U'], "0x19":['v','V'], "0x1A":['w','W'], "0x1B":['x','X'],"0x1C":['y','Y'], "0x1D":['z','Z'], "0x1E":['1','!'], "0x1F":['2','@'], "0x20":['3','#'], "0x21":['4','$'],"0x22":['5','%'], "0x23":['6','^'], "0x24":['7','&'], "0x25":['8','*'], "0x26":['9','('], "0x27":
    ['0',')'],"0x28":['\n','\n'], "0x29":['[ESC]','[ESC]'], "0x2A":['[BACKSPACE]','[BACKSPACE]'], "0x2B":['\t','\t'],"0x2C":[' ',' '], "0x2D":['-','_'], "0x2E":['=','+'], "0x2F":['[','{'], "0x30":[']','}'], "0x31":['\',"|'],"0x32":['#','~'], "0x33":";:", "0x34":"'\"", "0x36":",<",  "0x37":".>", "0x38":"/?","0x39":['[CAPSLOCK]','[CAPSLOCK]'], "0x3A":['F1'], "0x3B":['F2'], "0x3C":['F3'], "0x3D":['F4'], "0x3E":['F5'], "0x3F":['F6'], "0x41":['F7'], "0x42":['F8'], "0x43":['F9'], "0x44":['F10'], "0x45":['F11'],"0x46":['F12'], "0x4F":[u'→',u'→'], "0x50":[u'←',u'←'], "0x51":[u'↓',u'↓'], "0x52":[u'↑',u'↑']
   }
for _ in range(len(output3)):
    temp = output3[_].strip()
    if(temp[:2] == "00"):
        m = "0x" + temp[4:6]
        try:
            if m in usb_codes:
                print(usb_codes.get(m)[0], end='')
        except: continue
```
And we finally get the message
``` contactthepersonintheblackhat```