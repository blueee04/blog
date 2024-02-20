+++
title ="Finger Controlled Volume Control On Windows"
description = "Controlling Volume of The System Using Mediapipe and OpenCV"
date = "2023-10-28"
tags = ["Object Detection", "Computer Vision", "OpenCV", "Mediapipe", "Machine Learning", "Algorithim"]
summary = "Controlling Volume of The System Using Mediapipe and OpenCV"
+++
# Code
```py import cv2
import mediapipe as mp
from math import hypot
import _ctypes
from _ctypes import pointer
import os
from ctypes import *
from comtypes import HRESULT
from pycaw.pycaw import IAudioEndpointVolume
import numpy as np
from ctypes import cast, POINTER
from comtypes import CLSCTX_ALL
from pycaw.pycaw import AudioUtilities, IAudioEndpointVolume
import numpy as np

capture = cv2.VideoCapture(0)

mpHands = mp.solutions.hands
hands = mpHands.Hands()
mp_draw = mp.solutions.drawing_utils

devices = AudioUtilities.GetSpeakers()
interface = devices.Activate(IAudioEndpointVolume._iid_, CLSCTX_ALL, None)
volume = cast(interface, POINTER(IAudioEndpointVolume))
volbar = 200
volper = 0

volmin,volmax = volume.GetVolumeRange()[0:2]

while True:
  success, img = capture.read()
  imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

  result = hands.process(imgRGB)

  llist = []
  if result.multi_hand_landmarks:
    for handLms in result.multi_hand_landmarks:
      for id, lm in enumerate(handLms.landmark):
        h, w, c = img.shape
        cx, cy = int(lm.x * w), int(lm.y * h)
        llist.append([id, cx, cy])
      mp_draw.draw_landmarks(img, handLms, mpHands.HAND_CONNECTIONS)

  if llist != []:

    x1, y1 = llist[4][1], llist[4][2]
    x2, y2 = llist[8][1], llist[8][2]
    cv2.circle(img, (x1, y1), 15, (255, 0, 255), cv2.FILLED)
    cv2.circle(img, (x2, y2), 15, (255, 0, 255), cv2.FILLED)
    length = hypot(x2 - x1, y2 - y1)

    vol = np.interp(length, [50, 300], [volmin, volmax])
    volbar = np.interp(length, [50, 300], [400, 150])
    volper= np.interp(length, [50, 300], [0, 100])

    print(int(length), vol)
    volume.SetMasterVolumeLevel(vol, None)

    cv2.rectangle(img, (50, 150), (85, 400), (0, 255, 0), 3)
    cv2.rectangle(img, (50, int(volbar)), (85, 400), (0, 255, 0), cv2.FILLED)
    cv2.putText(img, f'{int(volper)}%', (40, 450), cv2.FONT_HERSHEY_COMPLEX, 1, (0, 255, 0), 3)

  cv2.imshow("Image", img)
  if cv2.waitKey(1) & 0xFF == ord('q'):
    break

capture.release()
cv2.destroyAllWindows()
```
# Explanation:

### Imports Necessary Libraries:

* cv2 for video capture and image processing
* mediapipe for hand tracking
* math for calculating distances
* pycaw for controlling system volume
* numpy for array manipulation

### Initializes Camera and MediaPipe Hands:

* Starts webcam capture using cv2.VideoCapture(0)
* Creates a MediaPipe Hands object to detect hands

### Sets Up Volume Control:

* Gets the default audio device's speakers
* Activates the IAudioEndpointVolume interface for volume control

### Captures and Processes Video Frames in a Loop:

* Continuously reads frames from the webcam
* Converts frames to RGB format for MediaPipe
* Detects hands in the frame using MediaPipe

### Calculates Fingertip Distance and Maps to Volume:

* If hands are detected:
* Gets coordinates of thumb and index fingertips
* Calculates the distance between them
* Maps this distance to a volume level using numpy.interp
* Sets the system volume using the IAudioEndpointVolume interface

### Visualizes Volume Control:

* Draws landmarks on the hand using MediaPipe's drawing utilities
* Draws a volume bar on the frame, indicating current volume level
* Displays the frame with visualizations

### Exits the Loop When 'q' Is Pressed:

* Breaks the loop if the 'q' key is pressed

### Cleans Up Resources:

* Releases the webcam capture
* Closes all open windows