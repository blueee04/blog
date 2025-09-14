+++
title = "What in the world is VAE"
date = 2025-08-17
summary = "Legendary Algorithm Pull"
tags = ["Deep Learning", "Computer Vision", "U-Net", "Medical Imaging", "Image Segmentation", "CNN", "Neural Networks", "Machine Learning"]
+++

Now what is VAE....VAE is basicaly Variational Auto Encoders which was actually build on the limmitations of normal Auto Encoders.
Let me give you a summary on what Auto Encoder is...

It consists of an Encoder, a Decoder and finally a bottle-neck which is used to compute something called the latent space...
Now what is latent space, Well let's say it's a box that takes in input from the encoder and extracts the most useful features out of it.

It's more like a compressed representation...

The Autoencoder's role is to take an image as an imput, the encoder extracts the required features, the bottlenect retains those features and at the end the decoder tries to recreate the same image from the features given by the bottleneck. Preserves critical information making sure the original image can be achieved again.

The error can be calculated through mean squared error....as we change the weights the autoencoder gets better at recreating the image