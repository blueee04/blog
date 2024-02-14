+++
title = "Bluetooth Network Capture"
date = "2023-06-12"
summary = "We have a Bluetooth Network Capture as a challenge and we need to follow it's movement to get the message" 
tags = ["DFC" , "Network" , "Forensics" , "Packet Capture"]
+++

## Description

We need to Map the movement of the cursor to recieve the hidden message written on by the user.......We have a Bluetooth Network Capture as a challenge and we need to follow it's movement to get the message

## Tools Required:

1. TShark
2. Numpy 
3. Matplotlib

## Code

``` py
import os
import matplotlib.pyplot as plt
import numpy as np 

count =  0
plot_x = []
plot_y = []

X = []
Y = []
condition = []
x_pos =  0
y_pos = 0

output1= os.popen("tshark -r yourpcaphere.pcap -Y \"btl2cap && btatt.handle==0x0030 && btatt.opcode==0x1b\" -T fields -e \"btatt.value\"").readlines()

def filter_the_values(a):
    return (a if a < 127 else (a-256))

for i in range(len(output1)):
    temp = bytes.fromhex(output1[i].strip())
    indicator = temp[0]
    X.append(filter_the_values(temp[2]))
    Y.append(filter_the_values(temp[3]))
    condition.append(indicator)

for _ in range(len(X)):
    x_pos += X[_]
    y_pos += Y[_]
    plot_x.append(x_pos)
    plot_y.append(y_pos)

plot_x = np.array(plot_x) * -1
plot_y = np.array(plot_y) 

plt.figure(figsize=(100,60))
for i in range(len(plot_x)):
    if  condition[i]:
        plt.scatter(plot_x[i],plot_y[i],marker='o',color='blue',linewidth=50,alpha=1)
plt.plot()        
```
## Result

![](https://raw.githubusercontent.com/blueee04/DFC-writeups/main/evidence1/bluetoothplot.jpeg?token=GHSAT0AAAAAACOAQ7L65E2IY7SDZG2IHCW6ZOL2JSA)

**[Mapped Cursor movement]**
## Code Explanation


1. The code imports the necessary libraries: os, matplotlib.pyplot, and numpy.

2. Several variables are init
ialized, including count, plot_x, plot_y, X, Y, condition, x_pos, and y_pos.

3. The output1 variable is assigned the output of a command executed using os.popen(). The command runs tshark (a network protocol analyzer) on a pcap file specified by "yourpcaphere.pcap". It filters the output to only include packets that match specific criteria related to Bluetooth Attribute Protocol (ATT).

4. The filter_the_values() function is defined to convert values in the range 128-255 to their corresponding negative values.

5. A loop iterates over each line of output1. Each line is converted from hexadecimal to a bytes object, and relevant values are extracted and appended to X, Y, and condition lists.

6. Another loop calculates cumulative positions (x_pos and y_pos) and appends them to plot_x and plot_y lists.
plot_x and plot_y lists are converted to numpy arrays and modified (multiply plot_x by -1).

7. A figure with a large size is created using plt.figure().

8. A loop iterates over each point in plot_x and plot_y. If the corresponding condition is true, a blue circle marker is added to the plot using plt.scatter().

9. Finally, plt.plot() is called to display the plot.

**Note: Make sure you have the necessary libraries installed (os, matplotlib, numpy) and replace "yourpcaphere.pcap" with the actual path to your pcap file. Additionally, you may need to adjust the figure size (plt.figure(figsize=(100,60))) according to your needs.**
