# Project 1 - Advanced Lane Detection

![image](https://user-images.githubusercontent.com/40880896/120982233-179b2f00-c796-11eb-8fe8-12b68f4c24fc.png)

## Overview
Detect lanes using computer vision techniques. 
The following steps were performed for lane detection:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

## Dependencies
* Python 3.5
* Numpy
* OpenCV-Python
* Matplotlib
* Pickle

## Camera calibration
The camera was calibrated using the chessboard images in 'camera_cal/*.jpg'. The following steps were performed for each calibration image:

* Convert to grayscale
* Find chessboard corners with OpenCV's `findChessboardCorners()` function, assuming a 9x6 board

After the above steps were executed for all calibration images, we used OpenCV's `calibrateCamera()` function to calculate the distortion matrices. Using the distortion matrices, we undistort images using OpenCV's `undistort()` function.

To illustrate, here is the comparison between the distorted and undistorted image via camera calibration:

![image](https://user-images.githubusercontent.com/40880896/120970998-c84f0180-c789-11eb-8fcd-35cec5702a18.png)

The final calibration matrices are saved in the pickle file 'calibrate_camera.p'

## Lane detection pipeline
The following describes and illustrates the steps involved in the lane detection pipeline. For illustration, below is the original image we will use as an example:

![image](https://user-images.githubusercontent.com/40880896/120971186-fa606380-c789-11eb-848b-4e1ebc65deae.png)

### Undistort image
Using the camera calibration matrices in 'calibrate_camera.p', we undistort the input image. Below is the example image above, undistorted:

![image](https://user-images.githubusercontent.com/40880896/120981399-4369e500-c795-11eb-80d4-8a162d700233.png)

The code to perform camera calibration is in **CameraCalibration** class. 

### Thresholded binary image
The next step is to create a thresholded binary image, taking the undistorted image as input. The goal is to identify pixels that are likely to be part of the lane lines. In particular, we perform the following:

* Apply the following filters with thresholding, to create separate "binary images" corresponding to each individual filter
  * Absolute horizontal Sobel operator on the image
  * Sobel operator in both horizontal and vertical directions and calculate its magnitude
  * Sobel operator to calculate the direction of the gradient
  * Convert the image from RGB space to HLS space, and threshold the S channel
* Combine the above binary images to create the final binary image

Here is the example image, transformed into a binary image by combining the above thresholded binary filters:

![image](https://user-images.githubusercontent.com/40880896/120981571-6b594880-c795-11eb-8aa2-722304a28b20.png)

The code to generate the thresholded binary image is in **Thresholding** class, in particular the function `combined_thresh()`.

### Perspective transform
Given the thresholded binary image, the next step is to perform a perspective transform. The goal is to transform the image such that we get a "bird's eye view" of the lane, which enables us to fit a curved line to the lane lines (e.g. polynomial fit). Another thing this accomplishes is to "crop" an area of the original image that is most likely to have the lane line pixels.

To accomplish the perspective transform, we use OpenCV's `getPerspectiveTransform()` and `warpPerspective()` functions. We hard-code the source and destination points for the perspective transform. The source and destination points were visually determined by manual inspection, although an important enhancement would be to algorithmically determine these points.

Here is the example image, after applying perspective transform:

![image](https://user-images.githubusercontent.com/40880896/120981651-7f04af00-c795-11eb-80a3-2c9988196c20.png)

The code to perform perspective transform is in **PerspectiveTransformation** class, in particular the function `perspective_transform()`. 

### Polynomial fit
Given the warped binary image from the previous step, we now fit a 2nd order polynomial to both left and right lane lines. In particular, we perform the following:

* Calculate a histogram of the bottom half of the image
* Partition the image into 9 horizontal slices
* Starting from the bottom slice, enclose a 200 pixel wide window around the left peak and right peak of the histogram (split the histogram in half vertically)
* Go up the horizontal window slices to find pixels that are likely to be part of the left and right lanes, recentering the sliding windows opportunistically
* Given 2 groups of pixels (left and right lane line candidate pixels), fit a 2nd order polynomial to each group, which represents the estimated left and right lane lines

Since our goal is to find lane lines from a video stream, we can take advantage of the temporal correlation between video frames.

Below is an illustration of the output of the polynomial fit, for our original example image. 

![image](https://user-images.githubusercontent.com/40880896/120981695-8deb6180-c795-11eb-9984-f750eb58a41c.png)

### Radius of curvature
Given the polynomial fit for the left and right lane lines, we calculated the radius of curvature for each line according to formulas presented [here](http://www.intmath.com/applications-differentiation/8-radius-curvature.php). We also converted the distance units from pixels to meters, assuming 30 meters per 720 pixels in the vertical direction, and 3.7 meters per 700 pixels in the horizontal direction.

Finally, we averaged the radius of curvature for the left and right lane lines, and reported this value in the final video's annotation.

### Vehicle offset from lane center
Given the polynomial fit for the left and right lane lines, we calculated the vehicle's offset from the lane center. The vehicle's offset from the center is annotated in the final video. We made the same assumptions as before when converting from pixels to meters.

To calculate the vehicle's offset from the center of the lane line, we assumed the vehicle's center is the center of the image. We calculated the lane's center as the mean x value of the bottom x value of the left lane line, and bottom x value of the right lane line. The offset is simply the vehicle's center x value (i.e. center x value of the image) minus the lane's center x value.


### Annotate original image with lane area
Given all the above, we can annotate the original image with the lane area, and information about the lane curvature and vehicle offset. Below are the steps to do so:

* Create a blank image, and draw our polyfit lines (estimated left and right lane lines)
* Fill the area between the lines (with green color)
* Use the inverse warp matrix calculated from the perspective transform, to "unwarp" the above such that it is aligned with the original image's perspective
* Overlay the above annotation on the original image
* Add text to the original image to display lane curvature and vehicle offset

Below is the final annotated version of our original image.

![image](https://user-images.githubusercontent.com/40880896/120981980-d6a31a80-c795-11eb-81e5-abfb0028dbe4.png)

## Discussion
This is an initial version of advanced computer-vision-based lane finding. There are multiple scenarios where this lane finder would not work. For example, the hard challenge video includes roads with cracks which could be mistaken as lane lines (see 'harderchallenge_video.mp4'). Also, it is possible that other vehicles in front would trick the lane finder into thinking it was part of the lane. More work can be done to make the lane detector more robust, e.g. [deep-learning-based semantic segmentation](https://arxiv.org/pdf/1605.06211.pdf) to find pixels that are likely to be lane markers (then performing polyfit on only those pixels).
