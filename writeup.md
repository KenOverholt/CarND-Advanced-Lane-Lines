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
- [x] Discuss issues and possible improvements.

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
Method name: `calibrate_camera()`

The code for this step is contained in the method calibrate_camera() in the first code cell of the IPython notebook located in "./Advanced Lane Finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1a]![alt text][image1b]

### Pipeline

#### 0. Jupyter Structure

The Jupyter Notebook (./Advanced Lane Finding.ipynb) is structured such that cells should be run in order from top to bottom.  The first cell calibrates the camera.  The next few cells define methods needed by the cells towards the end.  The final two cells create and display a video.  The next cell up creates an image and displays intermediate images created during the process.  The next cell up is called by the video and image creation cells below it.  It calls the cells above it which is where all the image processing is defined. 

#### 1. Provide an example of a distortion-corrected image.
Method name: `correct_distortion()`

To correct a distorted image, I use the undistort method from OpenCV2 passing in the distortion coefficients and camera matrix calculated from the camera calibration step above.  Below are the original image and the undistorted image calculated from the undistort method.
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
Method name: `apply_thresholding()`

I used a combination of color and gradient thresholds to generate a binary image.  The cell titled "Use color transforms, gradients, etc., to create a thresholded binary image." contains this code.  abs_sobel_thresh is used to apply the sobel method in the x or y direction.  mag_thresh applied a magnitude gradient to the image.  dir_threshold applies a directional threshold.  hsl_S_select applies a threshold on the S channel of an HLS image.  hls_L_select applies a threshold on the L channel of an HLS image.  apply_thresholding combines the various threshold methods to produce one the final thresholding binary image.  Here are examples of each thresholding method along with an image combining all of them.

![alt text][image3a]
![alt text][image3b]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
Method name: `warp()`

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
Method name: `detect_lane_pixels_and_find_lane_boundary()`

The next method in the IPython notebook, detect_lane_pixels_and_find_lane_boundary, detects lane pixels and finds the left and right boundaries.  The method takes in the warped image and creates a histogram along the columns of the image.  The two lane lines should be the highest points in the histogram.  The method creates a window around those points and then moves up to the space above that window.  It adjusts to center of that window based on the concentration of points in the area.  In this manner, it moves up the image identifying the points in of each lane.  Lines 86-88 fit the lane lines with a 2nd order polynomial.  Here is a visualization of the completed process:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
Method name: `determine_curvature_and_position()`

The next cell, titled "Determine the curvature of the lane and vehicle position with respect to center.", determines the curvature of the line and identifies the vehicle position with respect to the center of the lane.  It takes in the line points (left_fitx and right_fitx) generated from the previous method and uses numpy's `polyfit` to generate lines in real-world space and in meters.  Next it determines car's position between the two lane lines and subtracts the middle of the image from that to determine how far off the car is from the lane center in pixels.  It returns these values to be used by the next method which will display them on the final image using OpenCV's `putText()` method.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
Method name: `warp_lanes_onto_original_image()`

The next IPython cell contains `warp_lanes_onto_original_image()` which takes in the undistorted image drawing a polygon on it which covers the detected lane in front of the car.  The binary_warped images is converted to a color image and the lane drawn on it using OpenCV's `fillPolly()`.  OpenCV's `warpPerspective()` takes Minv (which was generated by 'warp()') to get the lane onto the original image.  Here is an example of my result on a test image:

![alt text][image6]

#### 7. Putting it all together
Method name: `process_image()`

The `process_image()` method calls each of the previous methods, in turn, to find the lane and draw it on the image.  The same pipeline is used whether prcessing an stand-alone image or a frame of a video.  A flag is passed in to identify which type of processing so the pipeline can be customized.  For example, intermediate images are generated for the stand-alone image while it would be unrealistic to display all the intermediate images for each frame of a video so those are suppressed.  This could also be used in enhancements to project which might process frames based on some values from the previous frame whereas a stand-alone image couldn't use that enhanced processing.

The "Create a single image" cell reads in a single image using matplotlib's `imread()` and calls `process_image()` to find and display the lane on it.  It also generates the intermediate images seen throughout this writeup.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The cell titled "Create a video" takes in a video and runs `process_image()` on each frame.  It uses `VideoFileClip()` from moviepy to generate a video overlayed with the lane image.

Here's a [link to my project video result](./project_video_result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

To improve lane detection, I experimented with different gradient values and thresholds for the L and S detectors.  I also tried different methods of combining them and ended up the valued and methods provided in my code.  It runs reasonable well on a relatively clean road but goes astray with heavy shadows.  You can see it start to stray in the project video but it quickly recovers even on a road surface that is a lighter color than the previous section of the road.

For next steps in improving the project, I would implement memory from frame-to-frame.  Maintaining recent history of values for key elements like lane points could help identify when shadows start washing out lane markings or when bright sun reduces differences in lane markings and the road surface. 

I have a link to the challenge video.  The pipeline requires more tweaking to get the challenge video to correctly detect the lane but here's the [link anyway](./challenge_video_result.mp4).  You can see it has difficulty when the left lane line gets close to the wall and its shadow.  The lack of separation fools the detector into thinking the lane line is wider than it actually is.  Narrowing the sliding window width might improve that bit.

The challenge video also starts losing the lane when the road surface color slowly darkens away from the line.  However, it does recognize what is happening and gets back on course.  But that still needs improvement.
