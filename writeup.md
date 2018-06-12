
---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit a polynomial to them to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Rubric points are considered individually and each point's implementation is described below.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

The following is a write up of Project 3. Additionally, a README file containing the project summary is located in the main part of this repository. All code reference in this write up can be found in the IPython notebook "advanced_lane_finding.ipynb"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell, under the title "Camera Calibration Using Chessboard Images". This code was provided in Udacity Self-Driving Car, Term1, Lesson 15. 

"object points" were prepared, which were the (x, y, z) coordinates of the chessboard corners. It is assumed that the chessboard was fixed on the (x, y) plane at z=0, such that the object points were the same for each calibration image.  Thus, `objp` was just a replicated array of coordinates, and `objpoints` was appended with a copy of it every successful detection chessboard corners in a test image.  `imgpoints` was appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

The output `objpoints` and `imgpoints` were used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  This distortion correction was applied to the test image using the `cv2.undistort()` function and obtained the following results: 

#### Figure 1: Distorted Chessboard Image
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/calibration1.jpg" alt="Chess" height="120" width="240" />

#### Figure 2: Undistorted Chessboard
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/Undistorted_chessboard.jpg" height="120" width="240" />

### Pipeline (single images)

The pipeline for lane detection and marking can be found in section "Lane Finding Pipeline". The steps in the pipeline are discussed below, and include the following for advanced lane line dectection:

* Undistort image
* Get a region-of-interest
* Create a binary image from HSL colorspace
* Warp binay image
* Find lane lines by fitting polynomial to warped image
* Calculate radius of the curve
* Color the lane between the lane lines
* Warp image back to original perspective
* Write radius of the curve in text on image

#### 1. Provide an example of a distortion-corrected image.

Distortion correction was applied to Figure 3 below, by applying the distortion coefficients calculated in the chessboard images. The corrected image (Figure 4) is also shown below: 

#### Figure 3: Distorted Test Image 
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/Distorted_Image.jpg" height="120" width="240" />

#### Figure 4: Undistorted Test Image
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/Unistorted_Image.jpg" height="120" width="240" />


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

For lane line identification, images in four color spaces (RGB, YUV, HSV, and HLS) were tested to determined which was best in detecting lane lines using a gradient operator and thresholding. Of the four, HLS more consistantly yielded both left and right lane lines.

#### Figure 5: Color Space Images
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/ColorSpaces.png" height="941" width="1000" />


RGB test images were converted to HLS color-space. The function `region_of_interest()` was used to segment out a trapazoid-shaped region-of-interest (ROI) encompassing primarily the road. The size of the ROI was based on visual inpection of the test images. Code may be view under the section "Functions for Image Manipulation".

Next, a 3-D binary image was created using the funtion `get_binary()` in section "Functions for Image Manipulation". The gradient of the L-channel was performed using the Sobel operator in the X-direction. Then the absolute value of the image was calculated. The image was scaled to a values between 0 and 255, thresholded, and multiplied by 255 to yield a "binary" image, of values 0 and 255. The S-channel image was simply thresholded and multiplied by 255 to create another "binary" image (0 and 255). 

Finally, in the pipeline a single-channel binary image was created by logical ORing the L- and S-channel binary images. This resulted in visually showing both left and right lanes ( see `pipline()` in section "Lane Finding Pipeline").


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

A perspective transformation was performed on the binary image using the function `warp_img()` in section "Functions for Image Manipulation". The function takes in source points (src_pts) and destination points (dst_pts) to calculate a transormation matrix (M) using the function `cv2.getPerspectiveTransormation()`.  The matrix is then used with `cv2.warpPerspective()` to warp the binary image.

Originally, the ROI points were used for src_pts while the dst_pts were derived from src_pts values, however the transformation was not acceptable resulting in poor line fitting. Therefore, the src_pts and dst_pts were determined through trial-and-error. The original points shifted the image too far off-center resulting bad offset caclulations (see below). So, new points were determined (Table 1) to center to warped lane lines about the middle of the x-axis. The approximate pixel distance between lane lines was about 850 pixels (200 pixels to 1050 pixels). Figures 5 show a straight image and the lane lines after the perspective transformation.  


[Table 1: Source and Destination Points for Matrix Transformation]

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 500, 500      | 125, 460      | 
| 790, 510      | 1075, 480     |
| 200, 670      | 125, 670      |
| 1030, 670     | 1050, 670     |



#### Figure 5: RGB and Binary Images of Unwarped and Warped Lane Lines 
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/redo_straight.png" height="240" width="320" />
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/redo_straight_warp.png" height="240" width="320" />
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/redo_straight_bin.png" height="240" width="320" />

<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/redo_straight_bin_warp.png" height="240" width="320" />



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To determine the lane lines, the warped image was passed to the function `find_lane_lines()`, in section "Functions For Lane Detection and Marking". This function used the histogram method described in the lecture. Next, coefficients corresponding to equation 1 were obtainded using `numpy.polyfit()`. Finally, the equation [1] of the line was calculated along the y-axis. The values of the polynomial coefficients and the lines values were stored in a Line class (see section "Line Class to Store and Handle Line Fitting and Curve Radius Calculations"). 



To reduce jitter during video processing, the three most recent values of the fit lines were stored in the Line class. The average of the three were used to draw the located lines on each video frame. 

#### Figure 6: Lane Lines on Binary Warped Image: Histogram Method
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/laneLines_histMethod.png" height="480" width="640" />


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

To visually mark the driving lane, the left and right fit lines were passed the  `fill_lane()` (section "Functions For Lane Detection and Marking") which used `cv2.polyfill()` and `cv2.addWeighted()` to color the lane, or area between the fitted lane lines in the warped image.  This image was transformed back into its original perspective using `warp_img()` with the original src_pts and dst_pts arguments swaped. 

#### [Equation 1]    `f(y) = A * y^2 + B * y + C`


#### Figure 7: Original Image
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/lane_noFill.png" height="315" width="460" />

#### Figure 8: Image with Lane Colored
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/lane_Fill.png" height="315" width="460" />

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of the curve was calculated during the lane line polynomial fitting using Line class funtions `left_right_fit()` and `add_radius()`. Three polynomial coefficients were obtained for both the left and right lanes using `numpy.polyfit` and correspond to A, B, and C of the equation. The average of each coeffiecient for. Then, the  The equation (below) was provided in lecture and calculated with respect to the y-axis as the points of the line along the x-axis were not monotonic. 

#### [Equation 2]    `Radius of curve =( (1+(2Ay+B)^2)^3/2 ) / ∣2A∣`

To convert pixel to meters, the variables ym_per_pix = 30/720 and self.xm_per_pix = 3.7/850 were used to multiply the x- and y-values of the lane lines during polynomial fitting.  A 3-point weighted average was used to smooth radius calculations during video processing. The radius of the curve was drawn on the each frame of the video using `cv2.putText()`.

The vehicle's offset (position of the vehicle with respect to the center of the lane) was calculated by finding the average center line then subtracting it from  the mid-point of the x-axis, assumed to be the actual center of the lane. The Line class function `calculate_offset()` used the rolling average polynomial coefficients, calculated from the radius of the curve, to calculate a center line according to Equation 1. The function used a y-value corresponding the bottom of the curve to obtain the position of the vehicle between the left and right lane lines. The y-value was converted into meters by multiplying it by `ym_per_px= 30/720`. The mid-point of the x-axis was converted to meters by mulitplying it by `xm_per_px = 3.7/850`. 

During video processing the offset was drawn on each frame using `cv2.putText()`. Offsets with a negative value indicate the vehicle's center is right of the mid-point, while positive offsets indicate the vehicle's center is to the left of the mid-point. During the video the offset values were typically near -1 meters.
 
---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

<a href="https://youtu.be/L9F9vf5eXEs" target="_blank"><img src="https://i9.ytimg.com/vi/L9F9vf5eXEs/1.jpg?sqp=CPiRltgF&rs=AOn4CLCoHE3zWpikt4utLocnua25ndqq-w" alt="Advanced Lane Finding Video" width="240" height="180" border="10" /></a>
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main issued faced during the development of the pipeline were:
* fiding the correct source and destination points for the perspective transformation (discussed briefly above)
* Finding the correct thresholding values to create a binary image
* using a faster lane finding algorithm instead of the histogram method

Improvements to the pipeline would include better lane line segmentation. In the video at about 43 seconds, there is distortion in the lane drawn on the video frames. This does not last for more that a second or two, but the distortion indicates that the fit lines ware incorrect, curving more to the left instead of to the right. 

This occured in an area where there were shadows on the road from a tree, which was the probable cause of the distortion. In the video the distortion was worse when using `find_lane_lines_quick()`. This function used the line values from previous line detection (e.g. `find_lane_line()` ) to more rapidly fit polynomials to the lane lines. For this reason `find_lane_lines_quick()` was not used in video processing. 

Another <b> probable of hypythetical </b> point of failure would be the introduction of additional lines and features on the road, such as old faded lane lines, cracks,  seams, or even shawdows cast from power lines or bridges. Because the lane line segmentation used the gradient/ derivative of the black-and-white image find lines, areas where there is sharp changes in neighboring pixel values, such as these additional lines and features,  would show up in the gradient. In fact, the challenge_video.mp4 has a seam in the road. The pipeline was tested using the challenge video and the pipeline failed to accurately find the lane lines, often finding a line to the seam instead. 

Another method to segment out the lane lines when creating a binary image was attempted. The idea was to segment lane lines based on color (e.g. yellow and white), as described in [Changing Colorspace](http://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_colorspaces/py_colorspaces.html#converting-colorspaces). The funtion `get_color_2_binary()`, in section "Functions for Image Manipulation", converted a RGB image to HSV color space. It used a lower and upper threshold for yellow (in HSV space) and white (in RGB space) to threshold the HSV and RGB images using `cv2.inRange()` to produce two binary images (0 and 255). These images were logically ORed together and passed throught the rest of the pipeline for line fitting. Figure 7 shows the driving lane successfully marked using this method.

#### [Figure 7. Lane Marking Using Color-Based Line Segmentation]

<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/LineSeg_by_Color.png" height="357" width="1167" />

While line segementaion was successful using test images, it broke down during video processing and failed to mark the lane lines altogether during the middle of the video. Further exploration of this method is warranted, to make the pipeline more robust and immune to color, shawdowing/lighting changes in the road. With more robust line segmentation the use of `find_lane_lines_quick()` would increase speed of frame processing. 



