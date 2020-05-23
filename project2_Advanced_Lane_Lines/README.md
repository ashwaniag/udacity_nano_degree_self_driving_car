# **Advanced Lane Finding Project**
---

![alt text][giff1]


**Overview**
---

The goal of this project is detect lanes using Computer Vision algorithms (color transforms, gradients, sliding windows).

The steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./result_images/calibration_output.png "Undistorted"
[image2]: ./result_images/calibration_straight_lines1.png "Road Transformed"
[image3]: ./result_images/binary_image.png "Binary Example"
[image4]: ./result_images/perspective_transform.png "Warp Example"
[image5]: ./result_images/fit_polynomial.png "Fit Visual"
[image6]: ./output_images/straight_lines1.jpg "Output"
[video1]: ./project_video.mp4 "Video"
[giff1]: ./final_output.gif
## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

First, we detect calibration on the given [calibration images](./camera_cal) using OpenCV function `cv2.findChessboardCorners()`.

This function return the correspondences between 3D and 2D space for the given images. Now, we can use this correspondence to calibrate the camera using `cv2.calibrateCamera()`. This function returns camera matrix and distortion coefficients, needed to undistort our images.

Using the camera matrix and distortion coefficients, we can use `cv2.undistort()` to undistort the images.

The code is in the function `calibrate_camera()` in [code.ipynb](./code.ipynb).

I got the following result on applying the undistortion on one of the chessboard image:


![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The obtained camera matrix and distortion coefficients from `calibrate_camera()` function, can be applied to the [test images](./test_images).

I got the following result on applying the undistortion on straight_line1.jpg image:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Creating the binary image is a very important step in detecting lanes.
I first tried applying the sobal mask in the x direction to detect lanes. It worked well in cases where the lines are visible clearly, straight lines but in cases where lines are not very visible, apply the sobel mask just in x direction didn't help.

If we are not able to create a good binary image, subsequest steps in the pipeline will not be able to detect lanes successfully.

In order to get good binary image, I use a combination of color and gradient thresholds. To detect white lines, I used equalised the histogram of the input frame. For the yellow lines, I thresholded on the V channel in HSV color space. Furthermore, I also used sobel operator to get the gradients of the lines. Finally, I also used morphological closure to the fill the gaps in the lines.

The code is implemented in the `binarize()` function [here](./code.ipynb).
Here are the results:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I chose the hardcode the source and destination points in the following manner:

Code can be found in the `warp()` function [here](./code.ipynb).

```python
    src = np.array([[(200,imshape[0]),
                     ((imshape[1] /2) - 50, (imshape[0] /2) + 100),
                     ((imshape[1] /2) + 50, (imshape[0] /2) + 100),
                     (1100,imshape[0])]], dtype='float32')
    dst = np.array([[(300, imshape[0]),
                     (300,0),
                     ((imshape[1] - 300), 0),
                     ((imshape[1] - 300), imshape[0])]], dtype='float32')
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 300, 720      | 
| 200, 460      | 300, 0        |
| 590, 460      | 980, 0        |
| 1100, 720     | 980, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In order to identify which pixels of a given binary image belong to lane-lines, we have (at least) two possibilities. If we have a brand new frame, and we never identified where the lane-lines are, we must perform an exhaustive search on the frame. This search is implemented in `get_fits_by_sliding_windows()` function: starting from the bottom of the image, precisely from the peaks location of the histogram of the binary image, we slide two windows towards the upper side of the image, deciding which pixels belong to which lane-line.

On the other hand, if we're processing a video and we identified lane-lines on the previous frame, we can limit our search in the neiborhood of the lane-lines we detected before. This second approach is implemented in `get_fits_by_previous_fits()` function. In order to keep track of detected lines across successive frames, I used global variables such as `last_left_line_pixel, last_right_line_pixel, last_left_line_meter, last_right_line_meter, recent_left_fits_pixel, recent_right_fits_pixel, recent_left_fits_meter, recent_right_fits_meter, detected`

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Position of the vehicle with respect to center is computed in `compute_offset_from_center()` function [here](./code.ipynb). We can approximate the car's deviation from the lane center as the distance between the center of the image and the midpoint at the bottom of the image of the two lane-lines detected.

During the previous lane-line detection phase, a 2nd order polynomial is fitted to each lane-line using np.polyfit(). This function returns the 3 coefficients that describe the curve, namely the coefficients of both the 2nd and 1st order terms plus the bias. From this coefficients, following [this](https://www.intmath.com/applications-differentiation/8-radius-curvature.php) equation, we can compute the radius of curvature of the curve. Code can be found in the `curvature_meter()` function.

In the case of video, both the radius of curvature and offset are calculated over the average of last 20 frames.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The whole pipeline is implemented in the `pipeline()` function [here](./code.ipynb).
Here is an example of my result on a test image:

![alt text][image6]


All the results of the test images can be found [here](./output_images).
---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I feel using Computer Vision algorithms to detect lane lines is not successful of every case. Like the challenge videos, this algorithm fails clearly. The lighting conditions, shadows and curves on the road, really tests the potential of this algorithm. I think training a CNN could be more effective in detecting lanes.

### References

https://github.com/ndrplz/self-driving-car/tree/1eadaca5e39b0448385db7ac2de0732d6dd7e600/project_4_advanced_lane_finding
