## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

[image1_1]: ./output_images/undistort_output.png "Original and Undistorted"
[image2_1]: ./output_images/undistort_test1.png "Original and Undistorted"
[image3_1]: ./output_images/HLScolorchannels_test1.png "HLS Channels"
[image3_2]: ./output_images/HLScolorthresholds_test1.png "HLS Binary"
[image3_3]: ./output_images/binary_test1.png "Original and Binary"
[image4_1]: ./output_images/warp_test1.png "Warp Example"

[image5_1]: ./output_images/binary_warped_test3.png "Input for Fit Visual"
[image5_2]: ./output_images/fitted_test3.png "output for Fit Visual"
[image6_1]: ./output_images/pipeline_test3.png "output for Pipeline"



## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1_1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

The code is in function `undistort()`. This function takes an image, `objpoints`, and `imgpoints` and returns an undistorted version of it. I used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image2_1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in `P2.ipynb`).  For the color threshold, I converted to HLS, and isolated each channel:

![alt text][image3_1]

Then, I tried a binary version for each channel (using lower thresholds for H). The S-channel seems like the best option, as appears here:

![alt text][image3_2]

For the gradient threshold, I used the H Channel as an input to calculate the derivative in the x direction, because lane lines are closer to vertical. Then, I took the absolute value of that in order to accentuate lines away from horizontal. Then, I converted the absolute value image to 8-bit, just to make sure my thresholds work on different scales. Then, I thresholded it.

After that, I combined both color and grandient thresholds. Here's an example of my output for this step.  

![alt text][image3_3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the same notebook under Step 3 headline.  The `warper()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

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

Then, I made use of the function `getPerspectiveTransform()` in order to calculate the perspective transform matrix, M. Then, I used the function `warpPerspective()` from OpenCV to warp the image.  I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4_1]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For this question, I prepared a binary warped version of test 3 as an input.

![alt text][image5_1]

Then, I created a function `use_windows()` which uses the sliding windows method to calculate the list of activated pixel that represent the two lanes. The sliding windows method start with searching in the lower half of the image using histogram. Since lanes in the bottom half of the image would be more like a straight line, the histogram would be likely to show two peaks, which represent the two lanes. From there, the upper half would be divided into equal-height smaller pairs of windows (each lane gets its own windows). Each window is drawn around the identified lanes: first sliding window starts from the histogram-found peaks. After that, the next window is re-centered if sufficient activated pixels were found within the current window. This function is called by another function `find_lane_pixels()`. In the primitive version of my code, this function calls the use_windows() and it eventually returns the positions of the left/right lane pixels. Later, in the video question, I add another function `search_around_poly()` which provides a faster calculation of the indices. Once we have the left/right lane pixels, we use the x and y values to calculate the 2nd order polynomial fit. The result looks like this:

![alt text][image5_2]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in Step 5 in my code in `P2.ipynb`. I applied it to the following image:

![alt text][image5_2]

The Radius of Curvature = 2311(m), and the vehicle is 0.048m left of center. The radius is chosen to be the minimum radius among the two curvatures.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in Step 6 in my code in `P2.ipynb` in the function `pipeline()`.  The order of the steps is the following: 1) calibrate camera (using object points and image points), 2) use this to undistort the image, 3) transform the image to an eagle-eye view, 4) produce the binary version of the image, 5) fit polynomials to the activated pixels in the image, and 6) calculate curviatures and distance from center. In the primitive version of my code (works for an image), I used the calculated fit and curvature for my final output. In the video version as I explain later, I considered a sanity check, and I updated my curvatures if the sanity check failed. I also used a smoothed version of the fit numbers and x-fitted values. Then, I create a filled polygon for the warped version of the image, and I inverse-transformed it to the original unwarped image, and I layered it with some weight (transparency) over the road. Finally, I print out the minimum of the two curvatures and the distance from center at the top. Here is an example of my result on test image 3:

![alt text][image6_1]

Below the code, I also show the testing  for all 8 test images.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

For this questions, I updated by previous code in the following ways:

1) I created a function `search_around_poly()` which provides a faster calculation of the indices. The function basically takes the fit lines from previous frame, and search within the area around those lines.

2) Now I have two versions of calculation: a) slow version (sliding window) and (b) fast version (search around the fit lane). The very first frame has to use the slow version. After that, lanes are calculated using the fast version, except if finding a lane fails 5 times in a row. If this happens, the calculation goes back to the slow version.

3) I added a class called `Line` (find it at the begining before Step 1). This class contains the parameters that are updated or called during the video.

4) I created a function called `sanityCheck()` that checks whether the identified lines/curvatures make sense. This function is called when the fast (search around the lanes) method is used.

5) I created a function called `updateLines()` which updates the parameters of the two `Line` objects. The function is called when the slow (sliding window) method is used or when the sanity check is passed.

6) I used a smooth version of the fit, and the x-fitted values from the last n rounds.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There are two remaining issues that I couldn't fix: 
1) It seems like the video is taking too much time, as if the fast method is not being called. I have already checked whether the fast method is being called, and I have tested the time of computation for each of the fast and slow methods, and validated that the fast is 10 times faster. I am still unsure it is doing what I expected it to do.
2) The performance on the video seems to be great for most of the time. The only exception is in the middle where the green polygon gets messed up for a second or two.
