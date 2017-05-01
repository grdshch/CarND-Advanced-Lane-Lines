**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./images/cal.png "Undistorted"
[image2]: ./images/cal2.png "Undistorted"
[image3]: ./images/s.png "Binary Example"
[image4]: ./images/s_bad.png "Bad Binary Example"
[image5]: ./images/hlsrgb.png "HLSRGB"
[image6]: ./images/perspective.png "Perspective transform"
[image7]: ./images/pixels.png "Finding pixels and curve"
[image8]: ./images/frame.png "Result frame"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

You're reading it!

### Source code
All code is in [project.ipynb](https://github.com/grdshch/CarND-Advanced-Lane-Lines/blob/master/project.ipynb) jupyter notebook file. It contains all stages of experiments with images and videos and the final pipeline. Also you may see it in [html](https://github.com/grdshch/CarND-Advanced-Lane-Lines/blob/master/project.html) format.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in cells 2-4 of the Jupyter notebook project.ipynb.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
Code to undistort road image is located in cell 4 of Jupyter notebook:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
First I used a combination of S channel from HLS color map and gradient of the image. I selected thresholds to get good result on test images.
![alt text][image3]

But then found that there are two places in the video where S channel works not well. There are tree shadows which give a lot of noise in S channel.

![alt text][image4]

I tried to make S threshold stricter but this give worse results on other parts of video. After playing with frames with tree shadows I found that histogram of such frame is much higher than one for test images.

As the result I decided to calculate number of pixels on combined binary image and if it is too large then to switch from S channel to B channel from RGB map.

Here all RGB and HLS channels of that frame:

![alt text][image5]

All described actions are located in `pipeline()` function in cell 17.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is located in cell 5 of Jupyter notebook and then included into `pipeline()` function in cell 17.

To make transform I took test image with straight lines, hardcoded source and destination coordinates.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I draw histogram of pixels in combined binary image, took two peaks - for left and right lane lines and used sliding windows to detect places with more pixels storing pixels indices. When indices were detected I used `np.polyfit()` method to get second order polynomial. All that code is located in cell 12 of Jupyter notebook and then is included into `pipeline()` function in cell 17.

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Calculating the radius of curvature is done in cell 16 of Jupyter notebook. I used [this tutorial](http://www.intmath.com/applications-differentiation/8-radius-curvature.php) to get formula for radius. Calculated radius is converted to meters.

To calculate the position of the vehicle I took found x coordinates of left and right lanes in the bottom of the image, too midpoint between them, calculate distance between that midpoint and center of the image, and convert the result to meters. This is done in the end of `pipeline()` function in cell 17 of Jupyter notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is one frame from result video as an example:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://github.com/grdshch/CarND-Advanced-Lane-Lines/blob/master/project_video_result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

It was quite easy to get good result on test images. After that about 48 seconds of 50s video were correct, but two 1-second pieces has pure lane lines. I fixed that seconds using caching found lane lines to draw them if pipeline failed to find new on current frame, refinding lane line using sliding windows if pipeline failed to find lane lines on previous frame, switching to B channel instead of S channel if binary image is too noisy. The last point is the weakest place of the algorithm because it looks like hardcoding the solution for the specific video. I selected constant to detect number of pixels when need to switch  to B channel but didn't test it enough. I need more robust algorithm detecting image parameters and selecting specific way to detect lane line pixels.
