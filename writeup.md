## Writeup

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

- [x] Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
- [x] Apply a distortion correction to raw images.
- [x] Use color transforms, gradients, etc., to create a thresholded binary image.
- [x] Apply a perspective transform to rectify binary image ("birds-eye view").
- [x] Detect lane pixels and fit to find the lane boundary.
- [x] Determine the curvature of the lane and vehicle position with respect to center.
- [x] Warp the detected lane boundaries back onto the original image.
- [x] Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
- [x] Create a pipeline for processing a video.
- [ ] Discuss issues and possible improvements.

[//]: # (Image References)

[image1a]: ./output_images/original_checkerboard.jpg "Original"
[image1b]: ./output_images/undistorted_checkerboard.jpg "Undistorted"
[image2]: ./output_images/test1_undistorted.jpg "Road Transformed"
[image3a]: ./output_images/thresholding1.jpg "Binary Example"
[image3b]: ./output_images/thresholding2.jpg "Binary Example"
[image4]: ./output_images/perspective_transform.jpg "Warp Example"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output.jpg "Output"
[video1]: ./project_video_result.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
Method name: calibrate_camera

The code for this step is contained in the method calibrate_camera() in the first code cell of the IPython notebook located in "./Advanced Lane Finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1a]![alt text][image1b]

### Pipeline

#### 0. Jupyter Structure

The Jupyter Notebook (./Advanced Lane Finding.ipynb) is structured such that cells should be run in order from top to bottom.  The first cell calibrates the camera.  The next few cells define methods needed by the cells towards the end.  The final two cells create and display a video.  The next cell up creates an image and displays intermediate images created during the process.  The next cell up is called by the video and image creation cells below it.  It calls the cells above it which is where all the image processing is defined. 

#### 1. Provide an example of a distortion-corrected image.
Method name: correct_distortion

To correct a distorted image, I use the undistort method from OpenCV2 passing in the distortion coefficients and camera matrix calculated from the camera calibration step above.  Below are the original image and the undistorted image calculated from the undistort method.
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
Method name: apply_thresholding

I used a combination of color and gradient thresholds to generate a binary image.  The cell titled "Use color transforms, gradients, etc., to create a thresholded binary image." contains this code.  abs_sobel_thresh is used to apply the sobel method in the x or y direction.  mag_thresh applied a magnitude gradient to the image.  dir_threshold applies a directional threshold.  hsl_S_select applies a threshold on the S channel of an HLS image.  hls_L_select applies a threshold on the L channel of an HLS image.  apply_thresholding combines the various threshold methods to produce one the final thresholding binary image.  Here are examples of each thresholding method along with an image combining all of them.

![alt text][image3a]
![alt text][image3b]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
Method name: warp

The code for my perspective transform includes a function called `warp()`, which appears in the next cell of the IPython notebook.  The `warp()` function takes as input, an image (`img`).  Based on the image's size, it creates source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
Method name: detect_lane_pixels_and_find_lane_boundary

The next method in the IPython notebook, detect_lane_pixels_and_find_lane_boundary, detects lane pixels and finds the left and right boundaries.  I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in cell titled "Determine the curvature of the lane and vehicle position with respect to center."

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
