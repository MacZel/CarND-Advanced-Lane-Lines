# **Advanced Lane Finding Project**
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

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

[image1]: ./writeup_images/chessboard_corners.png "Chessboard corners drawn"
[image2]: ./writeup_images/chessboard_undistorted.png "Chessboard undistorted"
[image3]: ./writeup_images/test_images_undistorted.png "Test images undistorted"
[image4]: ./writeup_images/test_images_src_quad.png "Test images source quad drawn"
[image5]: ./writeup_images/test_images_warped.png "Test images warped"
[image6]: ./writeup_images/test_images_dst_quad.png "Test images destination quad drawn"
[image7]: ./writeup_images/test_images_hls_s.png "Test images HLS S-channel threshold"
[image8]: ./writeup_images/test_images_hls_l.png "Test images HLS L-channel threshold"
[image9]: ./writeup_images/test_images_cielab_b.png "Test images CIELab b-channel threshold"
[image10]: ./writeup_images/test_images_sobel_x.png "Test images Sobel absolute x"
[image11]: ./writeup_images/test_images_sobel_y.png "Test images Sobel absolute y"
[image12]: ./writeup_images/test_images_sobel_mag.png "Test images Sobel magnitude"
[image13]: ./writeup_images/test_images_sobel_dir.png "Test images Sobel direction"
[image14]: ./writeup_images/test_images_combined_color_thresh.png "Test images Combined color thresholds"
[image15]: ./writeup_images/test_images_combined_sobel_thresh.png "Test images Combined Sobel thresholds"
[image16]: ./writeup_images/test_images_combined_binary.png "Test images Combined binary"
[image17]: ./writeup_images/test_images_binary_warped.png "Test images Binary warped"
[image18]: ./writeup_images/test_images_sliding_windows.png "Test images Sliding windows"
[image19]: ./writeup_images/test_images_lane_drawn.png "Test images Lane drawn"
[image20]: ./writeup_images/formula.svg "Curvature formula"
[video1]: ./project_video_output.mp4 "Project video output"
[video2]: ./challenge_video_output.mp4 "Challenge video output"
[video3]: ./harder_challenge_video_output.mp4 "Harder challenge video output"
[video4]: ./private_video_1_output.mp4 "Private video 1 output"
[video5]: ./private_video_2_output.mp4 "Private video 2 output"

## Running the notebook locally

0. Assumes python and git are already installed
1. Clone the repository
2. Create virtual environment inside cloned project, run `virtualenv venv`
3. Activate virtual environment, on Windows run `venv\Scripts\activate` or on Linux `source venv/bin/activate`
4. Install packages as listed in requirements, run `pip install -r requirements.txt`
5. Launch Jupyter notebook, run `jupyter notebook`

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one:

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I did create ImageIO class which combines basic file manipulation set of methods.
Another class that is inherited by Camera class is ImageProcessor. In ImageProcessor class I did gather most of the openCV image processing methods. Finally the Camera class contains the calibrate method which is ment to run calibration step on previously initialised Camera object. Camera class is also supplied with the method to undistort all of the supplied calibration images.

I did try to use OOP while dealing with this project, since it helps to maintain the code and it's heavily logically justified. Dissection of individual components helps to match methods to certain objects, preserve their state, and keep the code less messy.

To calibrate camera we need to supply it with a bunch of distorted chessboard images taken from various angles. `Camera.calibrate()` method expects calibration imgs directory and number of inside corners in x and y direction. Method works on OpenCV functions: `cv2.findChessboardCorners()` and `cv2.calibrateCamera()`. After successful calibration the calibration and distortion coefficients are preserved as states of Camera object (`self.mtx` & `self.dist`). Therfore available for further usage. Furthermore `Camera.calibrate()` expects an optional argument of output directory, and if supplied it will make use of OpenCV `cv2.drawChessboardCorners()` and write images to this directory.

Here is the output of calibration step `Camera.calibrate()` on one of the calibration images:
![Chessboard corners drawn][image1]

Here is the example of undistorted calibration image with `Camera.undistort_calibration_imgs()`.
Undistortion makes use of `cv2.undistort`:
![Chessboard undistorted][image2]


### Pipeline (test images)

#### 1. Provide an example of a distortion-corrected image.

Below is shown undistortion step `ImageProcessor.undistort()` applied to supplied test images.
The effect of this step is hardly visibly but can be seen in the difference of the car's hood shape.
![Test images undistorted][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The selection of colorspaces is based on this article [Colorspaces](https://www.learnopencv.com/color-spaces-in-opencv-cpp-python/)
I did experiment with HSV, HSL and CIELab colorspaces, and also with gradient Sobel thresholds. To generate final binary image I did combine HSL-Saturation channel, which works great for identifying both white and yellow lines, HSL-Lightness channel , which also did good on test images. I did observe that sometimes they fail to recognize yellow lines. Since it is hard to threshold specific color in RGB colorspace and both HSV and HSL, I found CIELab's b-channel to be most effective in thresholding yellows.
I also used a combination of Sobel x, y, magnitude and direction thresholds, just to get familiar with those type of filters. However I did not observe a gain of useful information by combining gradient thresholds.
Filters can be found as a methods of `ImageProcessor` object: `abs_sobel_thresh`, `mag_thresh`, `dir_thresh`, `hls_threshold`, `lab_threshold`. Functions make use of `cv2.Sobel()`, `cv2.cvtColor()`, `cvt.split()`, `cv2.merge`.
The final combined binary threshold is called in method `LaneFindingPipeline.combined_binary()`.

Below is shown HSL-S channel filtering applied to test images:
![Test images HSL-S channel filter][image7]

HSL-L channel filtering applied to test images:
![Test images HSL-L channel filter][image8]

CIELab-b channel filtering applied to test images:
![Test images HSL-L channel filter][image9]

Sobel x filtering applied to test images:
![Test images Sobel x filter][image10]

Sobel y filtering applied to test images:
![Test images Sobel y filter][image11]

Sobel magnitude filtering applied to test images:
![Test images Sobel magnitude filter][image12]

Sobel direction filtering applied to test images:
![Test images Sobel direction filter][image13]

Combined binary thresholding on test images:
![Test images combined binary][image16]

#### 3. Describe how you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `ImageProcessor.warp()`. The `warp()` function takes as inputs an image (`img`), as well as optional boolean argument `inverse` which defaults to False. If method is applied with default value of the inverse argument it will perform warping using perspective transform matrix - M, else the inverse of the perspective matrix will be used - Minv, performing unwarping of the image. Method makes use of `cv2.warpPerspective()`. To set perspective transform I did implement `ImageProcessor.set_perspective_transform()`, which takes as input list of source quad coordinates as well as offset value. Source qaud coordinates are picked experimentally. Destination quad coordinates are only dependent on offset value. The bigger the value the transformed image becomes more narrow.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. To draw `src` and `dst` quads I did implement `LaneFindingPipeline.draw_src_quad()` and `LaneFindingPipeline.draw_dst_quad()`.

Source quad drawn on test images:
![Test images source quad drawn][image4]

Warped test images:
![Test images warped][image5]

Destination quad drawn on test images:
![Test images destination quad drawn][image6]

#### 4. Describe how you identified lane-line pixels and fit their positions with a polynomial?

The functions `LaneFindingPipeline.find_lane_pixels()`, `LaneFindingPipeline.fit_polynomial()` and `LaneFindingPipeline.search_around_poly()` are the ones that perform identification of lane lines pixels. The `find_lane_pixels()` method performs calculation of a histogram of a bottom half of the image and finds base x positions of both left and right lines. The locations are taken from quartiles near the middle of the image. By observation it can be seen that lines are located close to the middle point of the image so narrowing the are we can reject surrounding disruptions from adjacent lanes.
Having the base line points set lane lines are being identified by sliding windows, each one centered at the midpoint of the pixels from previous window. After identifing all pixels corresponding to respective lanes the 2nd order polynomials are fitted `fit_polynomials()`. It is worth noting that sliding window concentrates on speeding up the algorithm by taking into account only the pixels within the windows, not the whole image.

Test images with sliding window process visualisation:
![Test images slinding window][image18]

The method `search_around_poly()` does the same task as previous method, but with the difference in initial assumption. It is highly uneffective to search for the lines once and again for every frame, since we know the position of lane line from the previous search. That is way in this method the lane line is searched within the area of previous found lane line  and the offset from the lane. Unfortunately I did not include visualisation of this step.

#### 5. Describe how you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in method `LaneFindingPipeline.measure_curvature_real()`. It is based on [Wiki article](https://pl.wikipedia.org/wiki/Krzywizna_krzywej):
![Curvature formula][image20]

second order derivative is equal to (same is also calculated for `right_fit_cr`):
`y_0'' = 2*left_fit_cr[0]*y_eval*self.ym_per_pix + left_fit_cr[1]` 
first order derivative (same is also calculated for `right_fit_cr`):
`y_0' = 2*left_fit_cr[0]`
where:
`y_eval` - the y position evaluated for the curvature withing the image
`left_fit_cr[0]` - 'a' coefficient of the 2nd order polynomial
`left_fit_cr[1]` - 'b' coefficient of the 2nd order polynomial
`ym_per_pix` - convertion value from pixels to meters

The final lane curvature is calculated by averaging both of the solutions for left and right line.
The car position calculated in `LaneFindingPipeline.measure_dist_to_lane_center()` is simply a distance from center of the image (we assume camera is located at the center of the hood), to x intercept positions of the lines at the bottom of the image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

For this purpose I did implement a `LaneFindingPipeline.draw_lane()` method.
Output of the LaneFindingPipeline is presented below:
![Lane drawn on test images][image19]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my project video result][video1]

Here are the LFP outputs applied to challenge videos and some private videos:
[Challenge video][video2]
[Harder challenge video][video3]
[Private video 1][video4]
[Private video 2][video5]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Well the biggest problem I did encounter was failing of the thresholding step to correctly recognize overbrighten and too dark images. I did try to deal with overbrightness by applying a CLAHE filter (Contrast Limited Adaptive Histogram Eqaulization). This step significantly slowed the speed of the Lane FInding Pipeline, and I did not gain any considerable improvement while applied to every frame. I think the main problem is I used hardcoded thresholds for color and gradient filters, and after applying CLAHE, thresholds would need to be changed accordingly. That is why I think CLAHE in combination with Adaptive thresholding would highly improve this step of Lane Finding algorithm. I did also experiment with applying gamma correction.

In future commits I'm planning to try implement a way to calculate an average value of brightness in image, and based on this information apply the CLAHE to only those images, also adjusting the thresholds.
