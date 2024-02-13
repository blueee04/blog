+++
title = "Face Detection"
date = "2023-10-22"
summary = "Face Detection Using Haarcascade Frontal Face Classifier"
tags = ["Object Detection", "Computer Vision", "OpenCV", "Haarcascade", "Machine Learning", "Algorithim"]

+++

* In this i'll be using the Haarcascade Classifier to detect Frontal Face
* It's not really a complex project but it is a start

```py
import cv2

face_classifier = cv2.CascadeClassifier(
    cv2.data.haarcascades + "haarcascade_frontalface_default.xml"
)
video_capture = cv2.VideoCapture(0)
```
* This line of code initializes a cascade classifier for detecting frontal faces by loading a pre-trained classifier from the OpenCV library's collection of Haar cascade classifiers.
* And also initializes a video capture object using OpenCV

```py
def detect_bounding_box(vid):
    gray_image = cv2.cvtColor(vid, cv2.COLOR_BGR2GRAY)
    faces = face_classifier.detectMultiScale(gray_image, 1.1, 5, minSize=(40, 40))
    for (x, y, w, h) in faces:
        cv2.rectangle(vid, (x, y), (x + w, y + h), (0, 255, 0), 4)
    return faces
```
 * This line of code converts each frame to gray scale and faces is used to store the specfic frames

 ```py
 while True:

    result, video_frame = video_capture.read()
    if result is False:
        break  

    faces = detect_bounding_box(video_frame) 

    cv2.imshow(
        "My Face Detection Project", video_frame
    ) 

    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

video_capture.release()
cv2.destroyAllWindows()
```
* This checks if the frames can be read and then runs the function to detect the faces
* It also configures the program to stop when the key 'q' is pressed

![image](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2023-10-22-Face-Detection/Screenshot%20from%202024-02-13%2006-31-04.png)

