
---
**Udacity Self-Driving Car Term1**
**Project 3, Advanced Lane Finding Project**

This project uses python, numpy, and openCV to find, and mark lanes of a road. 

The repository includes:
*IPython notebook of code to identity the current driving lane of automobile
*README.md file 
*Writeup.md, write up of the project
*output_images, folder of image and video from the project


The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit a polynomial to them to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

---


The pipeline for lane detection and marking can be found in section "Lane Finding Pipeline". The steps in the pipeline are summarized below, and include the following:

* Undistort image
* Get a region-of-interest
* Create a binary image from HSL colorspace
* Warp binay image
* Find lane lines by fitting polynomial to warped image
* Calculate radius of the curve
* Color the lane between the lane lines
* Warp image back to original perspective
* Write radius of the curve in text on image

#### 1. Distortion-corrected Image.

Distortion correction was applied to Figure 3 below, by applying the distortion coefficients calculated in the chessboard images. The corrected image (Figure 4) is also shown below: 

#### Figure 3: Distorted Test Image 
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/Distorted_Image.jpg" height="120" width="240" />

#### Figure 4: Undistorted Test Image
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/Unistorted_Image.jpg" height="120" width="240" />


#### 2. Color transforms, gradients or other methods to create a thresholded binary image.  

For lane line identification, images in four color spaces (RGB, YUV, HSV, and HLS) were tested to determined which was best in detecting lane lines using a gradient operator and thresholding. Of the four, HLS more consistantly yielded both left and right lane lines.

#### Figure 5: Color Space Images
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/ColorSpaces.png" height="941" width="1000" />


RGB test images were converted to HLS color-space. The function `region_of_interest()` was used to segment out a trapazoid-shaped region-of-interest (ROI) encompassing primarily the road. The size of the ROI was based on visual inpection of the test images. Code may be view under the section "Functions for Image Manipulation".

Next, a 3-D binary image was created using the funtion `get_binary()` in section "Functions for Image Manipulation". The gradient of the L-channel was performed using the Sobel operator in the X-direction. Then the absolute value of the image was calculated. The image was scaled to a values between 0 and 255, thresholded, and multiplied by 255 to yield a "binary" image, of values 0 and 255. The S-channel image was simply thresholded and multiplied by 255 to create another "binary" image (0 and 255). 

Finally, in the pipeline a single-channel binary image was created by logical ORing the L- and S-channel binary images. This resulted in visually showing both left and right lanes ( see `pipline()` in section "Lane Finding Pipeline").


#### 3. Perspective transformation.

A perspective transformation was performed on the binary image using the function `warp_img()` in section "Functions for Image Manipulation". The function takes in source points (src_pts) and destination points (dst_pts) to calculate a transormation matrix (M) using the function `cv2.getPerspectiveTransormation()`.  The matrix is then used with `cv2.warpPerspective()` to warp the binary image.

Originally, the ROI points were used for src_pts while the dst_pts were derived from src_pts values, however the transformation was not acceptable resulting in poor line fitting. Therefore, the src_pts and dst_pts (Table 1) were determined through trial-and-error. Figures 5 show a straight image and the lane lines after the perspective transformation of it's binary image. 


[Table 1: Source and Destination Points for Matrix Transformation]

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 490, 510      | 340, 470      | 
| 790, 510      | 1180, 470     |
| 250, 670      | 375, 670      |
| 1030, 670     | 1180, 670     |



#### Figure 5: Binary Images of Unwarped and Warped Perspective 
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/bin_warp2.png" height="349" width="918" />

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To determine the lane lines, the warped image was passed to the function `find_lane_lines()`, in section "Functions For Lane Detection and Marking". This function used the histogram method described in the lecture, then fit a second order polynomial to the lane lines. The values of the lines and polynomial coefficients were stored in a Line() Class (see section "Line() Class to Store and Handle Line Fitting and Curve Radius Calculations"). To reduce jitter during video processing, the three most recent values of the fit lines were stored in the Line class. The average of the three were used to draw the located lines on each video frame. 

#### Figure 6: Lane Lines on Binary Warped Image: Histogram Method
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/laneLines_histMethod.png" height="480" width="640" />


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

To visually mark the driving lane, the left and right fit lines were passed the  `fill_lane()` (section "Functions For Lane Detection and Marking") which used `cv2.polyfill()` and `cv2.addWeighted()` to color the lane, or area between the fitted lane lines in the warped image.  This image was transformed back into its original perspective using `warp_img()` with the original src_pts and dst_pts arguments swaped. 

#### Figure 7: Original Image
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/lane_noFill.png" height="315" width="460" />

#### Figure 8: Image with Lane Colored
<img src="https://github.com/bhumphrey0x20/Advanced-Lane-Finding-Proj-3/blob/master/output_images/lane_Fill.png" height="315" width="460" />

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of the curve was calculated during the lane line polynomial fitting using Line class funtions `left_right_fit()` and `add_radius()`. The equation (below) was provided in lecture and calculated with respect to the y-axis as the points of the line along the x-axis were not monotonic. 

`Radius of curve =( (1+(2Ay+B)2)3/2 ) / ∣2A∣`

To convert pixel to meters, the variables ym_per_pix = 30/720 and self.xm_per_pix = 3.7/700 were used to multiply the x- and y-values of the lane lines during polynomial fitting.  A 3-point weighted average was used to smooth radius calculations during video processing. The radius of the curve was drawn on the each frame of the video using `cv2.putText()`.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).



<a href="https://youtu.be/yIxWpOFOdh0" target="_blank"><img src="https://i9.ytimg.com/vi/yIxWpOFOdh0/1.jpg?sqp=CIDIkdgF&rs=AOn4CLC0qLDI3wvXhNW6te2mluWc09O3BA" 
alt="Advanced Lane Finding Video" width="240" height="180" border="10" /></a>
---
