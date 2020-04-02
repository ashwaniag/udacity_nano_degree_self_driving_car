# **Finding Lane Lines on the Road** 
---

**Overview**
---
The goal of this project is to find lanes on the road using computer vision algorithms.

![alt text][image2]

Note: Most of the code is from Udacity's nano degree program.
I modified some parts of the code and some parameter tuning to make it work with the test images.

[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[image2]: ./examples/laneLines_thirdPass.jpg
[image3]: ./examples/canny.jpg
[image4]: ./examples/masked_image.jpg
[image5]: ./examples/hough.jpg
---

### How does it work?

My pipeline consisted of following steps: 

First, Convert the images to grayscale.
![alt text][image1]

Second, Apply Gaussian filter to filter any noise in the image.

Third, Apply Canny edge detector to detect the edges in the image.
![alt text][image3]

Fourth, Mask the region of interest.
![alt text][image4]

Fifth, Use hough transform to get the lines and extend them from bottom of the image to top.
![alt text][image5]

Lastly, Blend it with the original image.
![alt text][image2]

### Modifications to the original code:
---

1) DrawLines function:
As suggested in the comment, I calculated the mean of slope and bias(intercept) of both the lines, left(slope < 0) and right (slope > 0).

Use these slopes and biases to calculate the final points of the line.

One thing I changed after experimenting is choosing the slope between 30 and 60 degress.

For the videos, I considered keeping the lines of the last frame to smooth the lines over time.


### Shortcomings of the pipeline
---

Since it doesn't use complex algorithms or machine learning to detect lanes, it fails on videos like challenge.mp4

The parameters are tuned specifically for the test images/videos and might not work in the real world.


### Possible improvements to your pipeline
---

I guess using Machine learning and using robust algorithms to predict real world scenarios can help.
