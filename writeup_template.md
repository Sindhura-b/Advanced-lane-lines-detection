## Advanced Lane Finding Project
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

[image1]: ./output_images/intial_camera_images.png "Undistorted"
[image2]: ./output_images/Distorted_undistorted_images.png "Road Transformed"
[image3]: ./output_images/lines_drawn.png "Binary Example"
[image4]: ./output_images/pipeline_images_output.png "Warp Example"
[image5]: ./output_images/sliding_window.png "Fit Visual"
[video1]: ./project_video_out_my.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Advanced_lane_line.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. The images whose corners are detected successfully and whose "object points" and "image points" are used for camera clibration are shown below:

![alt text][image1]

It is observed that some images are detected as they do not contain the specified number of corners ((9,6) in this case).

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

After obtaining the caliberations matrices and distortion coefficients, I built a pipeline that identifies lane lines in a given test image. When a test image is given as input to the pipeline, it first converts the distorted image to an undistorted image using `cal_undistort` fuction. 

Then gradients are used to detect the edges in an image. To do this, a fuction called `abs_sobel_thresh`, with thresholds of 0 and 250, is implemented to determine the sobel operators (S_x and S_y) that give the gradient of an image in x and y direction. Two more functions `mag_thresh` and `grad_threshold`, with thresolds 0 and 250, and 0 and 90 degrees, were implemented to determine the magnitude and gradients of image. Although gradient thresolding was able detect lane lines, there was still some noise in the resulting image. So, I have also implemented `r_thresh`, `s_thresh` and `h_thresh` functions to perform color thresholding of the test image. The next step is to use a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in code cell). I have tried various combinartions of thresholding to get the best out of both worlds. The best binary images I could obtain with minimal noise are from s-channel thresholding and r-channel thresholding. 

After having a thresholded binary image, the next is to perform a perspective transform of given image to bird eye view. Transforming image enables us to effectively view image from another perspective (bird's-eye view in this case) that is used for determining the lane curvature. Having the lane-curvature information is cruicial for a self-driving car to calculate steering angle and steer the vehicle accordingly. The steps for obatining perspective transform and image warping are implemented in `warp_corners` function. The `warp_corners` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. The source points are chosen such that they are along the lane lanes and form a rectangular box. The destination points are chose such that they display the only with lane lines from a bird's-eye view.

Following are source and destination points chosen for transforming:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 577, 463      | 450, 0        | 
| 704, 463      | 829, 720      |
| 277, 670     | 450, 720      |
| 1029, 670      | 829, 720     |

Once we have a thresholded warped image, the next step of this pipeline is to map out the lane lines. The position of lane lines in a thresholded warped image are identifying the corresponding pixel locations in the image. To do this, we first take histogram of all the columns in the lower half of the image. The peaks in the histogram signify the locations of left and right lane lines. These peak positions are used as starting points to apply sliding windows and search around that region to find locations with maximum pixels. These pixel locations are used to fit my lane lines with 2nd order polynomial and obtain its coefficients using `cv2.polyfit` function. All this is done by calling `sliding_window_polynomial_fit` The output image at this step looks similar to this:

![alt text][image5]

As discussed in Udacity's lectures, once you know where the lines are in one frame of video, a highly targeted search is done using `skip_sliding_window` function. The green area in the image below shows the region where we search for the lane lines this time.

![alt text][image3]

In the regions of sharp turns, shadows or sudden change in curvature, the pipeline might fail to detect lane lines using trageted search. In such situations where we lose the track of lane lines, the pipeline performs sliding window search. Few sanity checks such as empty lane line pixel locations and differences in the curvature of left and right lanes are used to switch between sliding window search and targeted search to map out the lane lines. The curvature of the lane lines, radius of curvature and offset of vehicle from center lane are calculated using the methods described in the lecture and the implementation of this is present in `radius_curvature` function.

The outputs of the test images across different stages of the pipeline are shown below:

![alt text][image4]

### Pipeline (video)

The pipeline is also tested on the video provided for this project and it performed reasonably well. 

Here's the [link to my video result](./project_video_out_my.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline intially failed to detect right lane line at certain locations. I figured out that the starting values of right lane line from the histogram are incorrect in those locations (eg: histogram in pipeline output image - row 3), i.e, the peak at the right lane line is smaller than the peak at the right end of the image (resulting from noise in the image). This has been fixed by doing sanity checks and performing sliding window search in those regions. However, changing threshold values and combining thresholds can be done to avoid the noise from other regions and detect lane lines properly. This pipeline also fails to detect lane lines in the challenge video. 
