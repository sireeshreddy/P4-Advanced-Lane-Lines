## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  

The Project
---

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

[image1]: ./camera_cal/calibration1.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## Camera Calibration

The camera was calibrated using the chessboard images in the 'camera_cal' folder
1. The images were converted to grayscale
2. Corners were found Using ''' cv2.findChessboardCorners() ''' and drawn using cv2.drawChessboardCorners()
3. Using image points and object points the cv2.calibrateCamera() method was used to calibrate the camera

## Lane Detection Pipeline

Steps taken to create a lane detection pipeline are as follows

### 1. Distortion correction

  a. Original image
  ![alt text][image1]
  b. Undistorted image generated using the cv2.undistort() method ('insert undistorted image')

### 2. Creating a Binary image to identify lane lines
  a. Convert to HLS space
  b. Apply absolute Sobel operator on the image
  c. Apply sobel operator in horizontal direction
  d. Threshold saturation and lightness
  e. Combine binary images to create a final binary image like the one below

('insert binary thresholded image here')

### 3. Apply a perspective transform
  a. A perspective transform is applied to give a bird's eye view of the road in order to identify the lane lines
  b. the source and desitnation points were hardcoded after obtaining the points manually through visual inspection
  c. cv2.getPerspectiveTransform() and cv2.warpPerspective() methods were used to transform the perspective of the image and obtain a birds-eye view
  d. A region of interest is created by applying a mask to only focus on the lane lines and ignore other portions of the image like the image below

The source and destination points are as follows:
Source | Destination
------------ | -------------
190, 720 | 340, 720
589, 457 | 340, 0
698, 457 | 995, 0
1145, 720 | 995, 720

('Insert perspective transform')

### 4. Detect lane pixels
  a. I used the binary warped image obtained in the previous step and took the following steps
  b. Starting from the bottom half, computed a histogram of detected pixel values. Peaks are detected following the application of a gaussian filter
  c. The image is partitioned into 6 horizontal slices
  d. Starting from the bottom slice, a 300 pixel wide window is used around the left peak and right peak of the histogram to determine points that could possible be the lane line
  e. This window is moved upwards to find pixels that could possibly be part of the lane line while recentering based on the center of the previous window
 
All of the above code is defined in cell 14 containing the ''' class Line() ''' as well as the method ''' get_binary_lane_image() '''
  
A fit to the current lane candidate is saved in the Line.current_fit_xvals attribute, together with the corresponding coefficients. The result of a fit for two lines is shown below.

### 5. Finding the curvature of the line
  a. For a second order polynomial f(y)=A y^2 +B y + C the radius of curvature is given by R = [(1+(2 Ay +B)^2 )^3/2]/|2A|
  b. Fixed lane width of 3.7m is assumed
  c. The ''' set_radius_of_curvature() ''' method computes this
  d. The distance from the center of the lane is computed in the ''' set_line_base_pos() ''' method, which measures the distance to each lane and computes the position assuming the lane has a given fixed width of 3.7m.

### 6. Warping lane boundaries back onto original image
With all the data collected above, all that is left is to annotate the original image with information about lane curvature and distance from center. This is done by
  a. Draw polyfit lines on a blank image
  b. Fill area between the lines with a green color
  c. Undistort and unwarp the image using the project_lane_lines() function
  d. Overlay the information on the original image using the ''' process_image() ''' function
  
### 7. Output visual display
  a. The code to process the video is in cell 19
  b. The annotated video is saved in './output_images/processed_project_video.mp4'
