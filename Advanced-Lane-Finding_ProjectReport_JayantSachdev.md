## Advanced Lane Finding

### Jayant Sachdev

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

[image1]: ./output_images/Camera_Calibration/distorted/calibration1.jpg "Distorted CheckerBoard Pattern"
[image2]: ./output_images/Camera_Calibration/undistorted/calibration1_undist.jpg "Undistorted CheckerBoard Pattern"
[image3]: ./output_images/Camera_Calibration/distorted/straight_lines1.jpg "Distorted Lane Image"
[image4]: ./output_images/Camera_Calibration/distorted/straight_lines1.jpg "Undistorted Lane Image"
[image5]: ./output_images/binary_combined1.png "Initial Combined Binary Image - Straight"
[image6]: ./output_images/binary_combined1_curved.png "Initial Combined Binary Image - Curve"
[image7]: ./output_images/binary_final.png "Final Combined Binary Image - Straight"
[image8]: ./output_images/binary_final_curved.png "Final Combined Binary Image - Curve"
[image9]: ./output_images/Binary_SL1.png "Binary Straight 1"
[image10]: ./output_images/Binary_SL2.png "Binary Straight 1"
[image11]: ./output_images/Binary_T1.png "Binary Test 1"
[image12]: ./output_images/Binary_T1.png "Binary Test 2"
[image13]: ./output_images/Binary_T1.png "Binary Test 3"
[image14]: ./output_images/Binary_T1.png "Binary Test 4"
[image15]: ./output_images/Binary_T1.png "Binary Test 5"
[image16]: ./output_images/Binary_T1.png "Binary Test 6"
[image17]: ./output_images/Transformed_SL1.png "Transformed Straight 1"
[image18]: ./output_images/Transformed_SL2.png "Transformed Straight 2"
[image19]: ./output_images/Transformed_T1.png "Transformed Test 1"
[image20]: ./output_images/Transformed_T1.png "Transformed Test 2"
[image21]: ./output_images/Transformed_T1.png "Transformed Test 3"
[image22]: ./output_images/Transformed_T1.png "Transformed Test 4"
[image23]: ./output_images/Transformed_T1.png "Transformed Test 5"
[image24]: ./output_images/Transformed_T1.png "Transformed Test 6"
[image25]: ./output_images/Lane_Lines_Detection_Polyfit.png "Lane Line Detection and Polyfit"
[image26]: ./output_images/Lane_Area_Filled.png "Fit the lane detection back onto original image"
[image27]: ./output_images/Lane_Area_Filled_curve.png "Fit the lane detection back onto original image - Curve"
[image28]: ./output_images/Lane_Curv_Position_Fill_Curve.png "Final Processing for curvature and lateral position in lane"
[video1]: ./output_images/project_video.mp4 "Video Processing"

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

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
