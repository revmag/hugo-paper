+++
title = "Understanding ViViT and how Videos are being encoded"
weight = 6
+++


We have a bunch of videos, so how do we encode it and pass it through a transformer for classification task.

Here’s how:
Lets say Input video is black and white( so input channels are 1),
and each frame is 28*28, and there are 28 frames in total( in simple terms, we have each video which is ~3 seconds long)

So, here’s how we convert these videos to a sequence to sequence layer( akin to how ViViT works):

input video shape is ( 28*28*28)
Lets say kernel is 8*8*8 ( yes, we are doing 3D convolution here), and we choose 128 to be the number of output channels ( to capture more relations in different dimensions for the video)

So output is
[28 -8 + 2 * 0 ( padding) / 8( stride) + 1 ] = 3.5, and take floor of that, so 3

`Video to be inputted to the transformer encoder`
So output is 3*3*3*128 * 128 channels for each input)

And then you add positional encoding to each of the 27 tokens (3*3*3) of the input video.

So, we are inputting to the transformer:
videos reduced in the form→
32,27,128

`How attention works on the reduced video tokens`
Let’s see how transformer works in this:

`Final Output( 11 classes)`
So the final output is 32,128 to 32,11. In a Feed forward manner, which will reduce it to probability values for each of the 11 classes.