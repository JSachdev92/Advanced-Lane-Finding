## Advanced Lane Finding

### Jayant Sachdev

---

**Advanced Lane Finding Project**

### Overview
The goals / steps of this project are the following:

1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
2. Apply a distortion correction to raw images.
3. Use color transforms, gradients, etc., to create a thresholded binary image.
4. Apply a perspective transform to rectify binary image ("birds-eye view").
5. Detect lane pixels and fit to find the lane boundary.
6. Determine the curvature of the lane and vehicle position with respect to center.
7. Warp the detected lane boundaries back onto the original image.
8. Display image with calculated lane boundaries and numerical estimation of lane curvature and vehicle position imposed over original  image.

[//]: # (Image References)

[image1]: ./output_images/Camera_Calibration/distorted/calibration1.jpg "Distorted CheckerBoard Pattern"
[image2]: ./output_images/Camera_Calibration/undistorted/calibration1_undist.jpg "Undistorted CheckerBoard Pattern"
[image3]: ./output_images/Camera_Calibration/distorted/straight_lines1.jpg "Distorted Lane Image"
[image4]: ./output_images/Camera_Calibration/undistorted/straight_lines1.jpg "Undistorted Lane Image"
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


### Steps 1 & 2: Compute Camera Coefficients to undistort images

For the first part of the project involved learning the camera coefficients to properly adjust for the effects for the lens on the camera and undistort any images that were taken by it. To do so, we used a series of 9x6 chessboard images taken by the camera. I then used the `findChessboardCorners()` function to find and define the chessboard corners in the image. Once all the chessboard corners and image points were found across all images, the function `calibrateCamera()` was used to learn the matrix and distortion coefficients for the camera. We can then use the coefficients and matrix to undistort images taken by the camera using the `undistort()` function.The result can be seen on a chessboard image below:

Distorted Chessboard Pattern |  Undistorted Chessboard Pattern
:-------------------------:|:-------------------------:
![alt text][image1]  |  ![alt text][image2]


Undistorting a real world image of a highway can be seen below"

Distorted Image |  Undistorted Image
:-------------------------:|:-------------------------:
![alt text][image3] | ![alt text][image4]

One can notice the effects of undistoring the image if one focuses on the 3rd lane line to the right of the vehicle. It appears much less distorted after the correction.

### Step 3: Use color transforms, gradients, etc., to create a thresholded binary image.

In order to create a thresholded binary image, i utilized all the gradient and color transform methods available. I created functions for sobel absolute magnitude and direction thresholding based binary images as well as sobel threshold in the x and directions. In addition, i generated functions for h, l and s colourspace transforms. I tuned the thresholds for each method by analyzing them one by one and then played with the combinations to obtain the best results. As mentioned in the lectures, i also came to the conclusion that the L space was too noisy to use. After tuning the rest of the thresholds and playing with which methods to combine in which order, i obtained the following results:

Straight Line Binary Image - Combined |  Curved Line Binary Image - Combined
:-------------------------:|:-------------------------:
![alt text][image5] | ![alt text][image6]

I then fine tuned the thresholds and tweeked the order slightly to obtain a result i was satisfied with:

Straight Line Binary Image - Combined |  Curved Line Binary Image - Combined
:-------------------------:|:-------------------------:
![alt text][image7] | ![alt text][image8]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)



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
