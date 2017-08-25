## Writeup Report

---

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

[image1]: ./report_images/undistorted.png "Undistorted"
[image7]: ./report_images/drawChessboard.png "drawChessboard"
[image8]: ./report_images/undistorted_straight.png "Undistort_test"
[image9]: ./report_images/trackbars.png "trackbars"
[image11]: ./report_images/binary_and_perspective.png "pipe"
[image12]: ./report_images/fit.png "fit"
[image13]: ./report_images/unwarp.png "unwarp"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[gif1]: ./report_images/lane_detection.gif
[video1]: ./project_video.mp4 "Video"


---
### Files
This project contains the following files:
- writeup_report.md which is this file
- CarND_Advanced_Lane_Lines.ipynb containing all the code for the project
- lane_detection.mp4 which is the resulting video of this project
- report_images containing images which are used in this writeup

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I used 20 chessboard images to calibrate my camera, as can be seen in my jupyter notebook `CarND_Advanced_Lane_Lines.ipynb`. The `cv2.findChessboardCorner()` function looks for a chessboard we have specified in `objp`, that is a flat chessboard with 9x6 points and returns the position of these corners in the image to `img_points`. Since we use the only one chessboard the same `objp` is appended to `object_points`. To check if the pattern is correctly detected I use `cv2.drawChessboardCorners()`:

![alt text][image7]

To calibrate a camera we need the camera matrix and the distortion coefficients. Both are returned by the `cv2.calibrateCamera()` function. As inputs we just need to pass the image, `img_points` and `object_points`. With the `cv2.undistort()` I obtained this result:

![alt text][image1]

### Pipeline (single images)

#### Approach
Since there are many design choices and parameters to tune I used trackbars for testing. With this, testing was much faster and I could see the changes immediately. I could control the following things with my sliders:
- which test image to apply the pipeline
- kernel size of the Gaussian filter and edge detector
- edge detection method (Soblel, Laplacian, Canny)
- color space (HLS and HSV) and respective color channel
- High and low thresholds for gradients

![alt text][image9]

This part of the code can be found under 'TRACKBAR TESTING' in the jupyter notebook. After some tuning, the results are then displayed all at once 'SINGLE IMAGE PIPELINE' to get an overview. The details of the pipeline are described in the following sections.


#### 1. Undistortion
With the previous camera calibration I can undistort my input images. On the left you can see the original image. On the right side the undistorted image. It is barely noticeable but it will make our localization more precise.
![alt text][image8]

#### 2. Color transform and gradients
With the sliders I found the saturation in the HLS colorspace working well in combination with a 5x5 Sobel filter. The gradient direction was also considered and implemented as trackbars, but it was not much of a improve. But with further testing using Sobel only in x-direction showed better behaviour. Then a low threshold and a region of interest is applied to reduce noise.

#### 3. Perspective transform
For the perspective transform I use `cv2.getPerspectiveTransform()` and `cv2.warpPerspective()`. To get the transformation matrix I need to define source points and destination points. The source points are points in the image which I believe to be a rectangle in the real world. These points are chosen easily on straight lane. One must be careful with choosing the points, because points too far from the camera might result in blurry and distorted lines in the birds-view even with a calibration.

![alt text][image11]

#### 4. Sliding window and polynomial fitting

First a histogram is plotted over the lower half of the image to detect the initial position of the left and right lanes. The peaks of the histogram indicates where are starting position of the lane could be. Then I follow the lane with a sliding window and save all points to a list. All points outside of the window are discarded. After that, I fit a second order polynomial over the found points.
Once a polynomial is found, the histogram and the sliding window approach is skipped. We instead look for points which are near the previous lane. Should there be too few points for a polynomial fit or the left and right lane merge into one lane (which can happen), I fall back to the histogram and sliding window technique to reinitialize the lane.

![alt text][image12]

#### 5. Lane curvature and vehicle position
The lane curvature is calculated according to the fit polynomial at  approximately one third of the lane according to the formula: R_curve = (1+(2*A*y+B)^2)^2(3/2) / abs(2*A) given a second order polynomial of A*y^2 + B*y + C


#### 6. Plot found lanes back to image
The lanes in the image above are warped back and blended with the original image:
![alt text][image13]


---

### Pipeline (video)
![alt text][gif1]  
Here's a [link to this video result](./lane_detection.mp4)

---

### Discussion

The pipeline is still not very robust since it fails in the challenge videos. One weakness is the edge detection. If thresholding is too high lines will hardly be detected and if it is too low, too much noise is present. I would like to try an adaptive thresholding, lower for farther points and higher for points which are near the vehicle. Also smoothing the detected lane over multiple images should avoid wiggly lanes.
