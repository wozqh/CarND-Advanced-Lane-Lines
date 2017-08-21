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

[image1]: ./output_images/undistort_output.jpg "Undistorted"
[image2]: ./output_images/test1.jpg "Road Transformed"
[image3]: ./output_images/binary_combo.png "Binary Example"
[image4]: ./output_images/warped_straight_lines.png "Warp Example"
[image5]: ./output_images/color_fit_lines.png "Fit Visual"
[image6]: ./output_images/output.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code of the IPython notebook 
lacated in "Adavanced Lane Finding Project.ipynb".

I start by preparing "object points", which will be the (x,y,z) coordinates 
of the chessboard corners in the world. Here I am assuming the chessboard is
fixed on the (x,y) plane at z=0, such that the object points are the same for each calibration image. Thus, 'objp' is just a replicated array of coordinates, and 'objpoints' will be appended with a copy it every time I successfully detect all chessboard corners in a test image. 'imgpoints' will be append with the (x,y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output 'objpoints' and 'imgpoints' to compute the camera calibration and distortion coefficients using the 'cv2.calibrateCamera()' function. I applied this distortion correction to the test image using the 'cv2.undistort()' function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I applied the output 'cameraMatrix' and 'distCoeffs' to the test image using the 'cv2.undistort()' function.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at 'Use color transforms，gradients,etc.,to create a thresholded binary image' part).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_M()`, which appears in 'Apply a perspective transform to rectify binary image ("birds-eye view")'.  The `warp_M()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 690, 450      | 1053, 0       | 
| 1053, 688     | 1053, 688     |
| 239, 689      | 239, 688      |
| 595, 450      | 239, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I first take a histogram along all the columns in the lower half of the image.
With this histogram I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines. From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in 'Calculation of radius of curvature after correcting for scale in x and y'part.

The radius of curvature at any point x of the function x = f(y) is given as follows:

```python
    ploty = np.linspace(0,img.shape[0]-1,img.shape[0])# to cover same y-range as image
    plotx = fit[0]*ploty**2 + fit[1]*ploty + fit[2]
    # Define y-value where we want radius of curvature
    # The maximum y-value is chosen, corresponding to the bottom of the image
    y_eval = np.max(ploty)
    # Fit new polynomials to x,y in world space
    fit_cr = np.polyfit(ploty*ym_per_pix,plotx*xm_per_pix,2)
    # Calculate the new radius of curvature
    curverad = ((1 + (2*fit_cr[0]*y_eval*ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
    curverad = np.round(curverad,3)
```


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 'Draw the image of lane result plotted back down onto the road'.Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_result.mp4)
Here's a [link to my challenge video result](./challenge_video_result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took,I combied sobel gradient color binary and HLS color space. This method can help me detect yellow lines and white lines at the same time. But when there is a shadow,the detection will be bad. Maybe I should enlarge my color threshold range. 

