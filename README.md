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

[image1]: ./test_images/test1.jpg "Original Image"
[image2]: ./output_images/undistorted_test1.jpg "Undistorted Image"
[image3]: ./output_images/binary.jpg "Binary Example"
[image4]: ./output_images/straight_lines_warped.jpg "Warp Example"
[image5]: ./output_images/warped_binaryimage_roi.jpg "Warped Binary"
[image6]: ./output_images/fitted_lines.jpg "Fitted Lines"
[image7]: ./output_images/project_test5.jpg "Annotated Image"
[video1]: ./project_video.mp4 "Video"

## Camera Calibration

The camera was calibrated using the chessboard images in the 'camera_cal' folder
* The images were converted to grayscale
* Corners were found Using `cv2.findChessboardCorners()` and drawn using `cv2.drawChessboardCorners()`
* Using image points and object points the `cv2.calibrateCamera()` method was used to calibrate the camera

## Lane Detection Pipeline

Steps taken to create a lane detection pipeline are as follows

### 1. Distortion correction

a. Original image
![alt text][image1]
b. Undistorted image generated using the `cv2.undistort()` method 
![alt text][image2]

### 2. Creating a Binary image to identify lane lines
* Convert to HLS space
* Apply absolute Sobel operator on the image
* Apply sobel operator in horizontal direction
* Threshold saturation and lightness
* Combine binary images to create a final binary image like the one below
  
![alt text][image3]


### 3. Apply a perspective transform
* A perspective transform is applied to give a bird's eye view of the road in order to identify the lane lines
* the source and desitnation points were hardcoded after obtaining the points manually through visual inspection
* `cv2.getPerspectiveTransform()` and `cv2.warpPerspective()` methods were used to transform the perspective of the image and obtain a birds-eye view
* A region of interest is created by applying a mask to only focus on the lane lines and ignore other portions of the image like the image below

Warped Image
![alt text][image4]
  
The source and destination points are as follows:

Source | Destination
------------ | -------------
190, 720 | 340, 720
589, 457 | 340, 0
698, 457 | 995, 0
1145, 720 | 995, 720

Warped Binary Image
![alt text][image5]

### 4. Detect lane pixels
* I used the binary warped image obtained in the previous step and took the following steps
* Starting from the bottom half, computed a histogram of detected pixel values. Peaks are detected following the application of a gaussian filter
* The image is partitioned into 6 horizontal slices
* Starting from the bottom slice, a 300 pixel wide window is used around the left peak and right peak of the histogram to determine points that could possible be the lane line
* This window is moved upwards to find pixels that could possibly be part of the lane line while recentering based on the center of the previous window
 
All of the above code is defined in cell 14 containing the `class Line()` as well as the method `get_binary_lane_image()`
  
A fit to the current lane candidate is saved in the Line.current_fit_xvals attribute, together with the corresponding coefficients. The result of a fit for two lines is shown below.

### 5. Finding the curvature of the line
* For a second order polynomial f(y)=A y^2 +B y + C the radius of curvature is given by R = [(1+(2 Ay +B)^2 )^3/2]/|2A|
* Fixed lane width of 3.7m is assumed
* The `set_radius_of_curvature()` method computes this
* The distance from the center of the lane is computed in the `set_line_base_pos()` method, which measures the distance to each lane and computes the position assuming the lane has a given fixed width of 3.7m.

![alt text][image6]

### 6. Warping lane boundaries back onto original image
With all the data collected above, all that is left is to annotate the original image with information about lane curvature and distance from center. This is done by

* Draw polyfit lines on a blank image
* Fill area between the lines with a green color
* Undistort and unwarp the image using the `project_lane_lines()` function
* Overlay the information on the original image using the `process_image()` function

![alt text][image7]
  
### 7. Output visual display
* The code to process the video is in cell 19
* The annotated video is saved in './output_images/processed_project_video.mp4'

[Video Link](https://youtu.be/6TE0DwIk6Ls)

## Discussion

* The pipeline works pretty well on the project video. However, the areas where the lane detection is a little shaky is in changing light conditions. Playing with the saturation and lightness thresholds helped greatly with detecting the lane lines
* Another potential problem is when lane markings are not clearly visible due to debris, dust, snow, faded markings. Salt lines on the road can also easily trick the pipeline into thinking they are multiple lanes. In a situation like that a more robust deep learning network is what is needed

