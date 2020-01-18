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
7. Define the area between the lane boundaries in the original image.
8. Display image with calculated lane boundaries and numerical estimation of lane curvature and vehicle position imposed over original  image.
9. Create a pipeline to run on a video feed.

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
[image12]: ./output_images/Binary_T2.png "Binary Test 2"
[image13]: ./output_images/Binary_T3.png "Binary Test 3"
[image14]: ./output_images/Binary_T4.png "Binary Test 4"
[image15]: ./output_images/Binary_T5.png "Binary Test 5"
[image16]: ./output_images/Binary_T6.png "Binary Test 6"
[image17]: ./output_images/Transformed_SL1.png "Transformed Straight 1"
[image18]: ./output_images/Transformed_SL2.png "Transformed Straight 2"
[image19]: ./output_images/Transformed_T1.png "Transformed Test 1"
[image20]: ./output_images/Transformed_T2.png "Transformed Test 2"
[image21]: ./output_images/Transformed_T3.png "Transformed Test 3"
[image22]: ./output_images/Transformed_T4.png "Transformed Test 4"
[image23]: ./output_images/Transformed_T5.png "Transformed Test 5"
[image24]: ./output_images/Transformed_T6.png "Transformed Test 6"
[image25]: ./output_images/Lane_Lines_Detection_Polyfit.png "Lane Line Detection and Polyfit"
[image26]: ./output_images/Lane_Area_Filled.png "Fit the lane detection back onto original image"
[image27]: ./output_images/Lane_Area_Filled_curve.png "Fit the lane detection back onto original image - Curve"
[image28]: ./output_images/Lane_Curv_Position_Fill_Curve.png "Final Processing for curvature and lateral position in lane"
[image29]: ./output_images/SourcePoints_Transform.png "Source points"
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

In order to create a thresholded binary image, i utilized all the gradient and color transform methods available. I created functions for sobel absolute magnitude and direction thresholding based binary images as well as sobel threshold in the x and directions. In addition, i generated functions for h, l and s colourspace transforms. I tuned the thresholds for each method by analyzing them one by one and then played with the combinations to obtain the best results. I created a function `binaryimg()` to return a binary image based on inputted thresholds. This can be seen in the 3rd code cell in my jupyter notebook `LaneProc.ipynb` As mentioned in the lectures, i also came to the conclusion that the L space was too noisy to use. After tuning the rest of the thresholds and playing with which methods to combine in which order, i obtained the following results:

Initial Straight Line Binary Image - Combined |  Initial Curved Line Binary Image - Combined
:-------------------------:|:-------------------------:
![alt text][image5] | ![alt text][image6]

I then fine tuned the thresholds and removed the H color space because i realized it was not helping and tweeked the order slightly to obtain a result i was satisfied with. I spent a lot of efforts here especially with the image test5. As i noticed the shadow from the tree caused a lot of noise, which then affected the lane polynomial that was fit to the curve. The resulting binary images can be seen below:

Straight Line Binary Image - Combined |  Curved Line Binary Image - Combined
:-------------------------:|:-------------------------:
![alt text][image7] | ![alt text][image8]

### Step 4: Applying a perspective transform to rectify a binary image

The 4th step would be to take the binary image and transform the image so you get a birds eye view of the lane lines. To do so, I first selected the trapezoidal area which i knew represented the straight lines and should appear in a rectangle from a birds eye view. I plotted the image and selected the corners as such:
```python
lt = (575, 460) #Left Top
lb = (255,680)  #Left Bottom
rb = (1060, 680) #Right Bottom
rt = (710, 460) #Right Top
```
I then knew that i wanted these points to be the focal point of the rectangle in the transformed image. As such, for the destination points, i added margins to the x-axis of the image and made the y-axis cordinates the top and bottom of the transformed image:

```python
tlb = (200,image.shape[0])#Left Bottom
tlt = (200, 0)#Left Top
trb = ((image.shape[1]-250), image.shape[0])#Right Bottom
trt = ((image.shape[1]-250), 0)#Right Top
```
I then used the source and destination points along with the `getPerspectiveTransform()` function to calculate the M and Minv matrices. I also created a function called `transfrm()` to take in an image with the source and destination points and return the transformed image using the `warpPerspective()` function. The results of the unwarped binary vs. the transformed binary images are below:

Binary Image - Unwarped |  Binary Image - Warped
:-------------------------:|:-------------------------:
![alt text][image9] | ![alt text][image17]
![alt text][image10] | ![alt text][image18]
![alt text][image11] | ![alt text][image19]
![alt text][image12] | ![alt text][image20]
![alt text][image13] | ![alt text][image21]
![alt text][image14] | ![alt text][image22]
![alt text][image15] | ![alt text][image23]
![alt text][image16] | ![alt text][image24]

### Step 5: Detect the lane pixels and fit a polynomial to the lane boundary

The next part of the project involved finding the lane pixels. To do so, i used a histogram approach to determine the highest concentrations of activated pixels as we move along the x - axis. I took the 2 highest peaks and selected them as my starting point. I then broke down the y axis into 9 sliding windows and recentered the window if we found a minimum number of pixels activated within a margin of the previous window. This technique is detailed in the course material. I defined a function `find_lane_pixels()` to conduct this calculation. I also wrote a function called `appendLine()` which takes left and right lane pixels from the previous frame and searches around these pixels for activated lane markings. Outputs from either of these functions are inputted into the `fit_polynomial()` function, where i fit a 2nd order polynomial to the activated left and right lane line pixels identified.

The outcome of using the sliding window method to detect lane lines is shown below:
![alt text][image25]


### Step 6: Determining the curvature of the lane and vehicle position with respect to center.

In addition to fitting the polynomial to the activated lane pixels in the function `fit_polynomial()`, i also converted the lane pixels to physical distances from the origin using the provided values for meters per pixel in each axis. I then fit a polynomial to each of these real world lane polynomials to utilize in the calculations for curvature and vehicle position with respect to the center of the lane.

In order to calculate the curvature, i utilized the real world lane polynomial as function of distance in x and y axis, along with the formula:

R<sub>curve</sub> = (1+(2Ay+B)<sup>2</sup>)<sup>3/2</sup>/(|2A|),

to calculate the curvature of the left and right lane markings. I then averaged the left and right markings to get a better estimate of the desired path curvature at the center of the lane. 

In order to calculate the vehicle position with respect to center, i first defined the vehicle center as the center of the x - axis. Then i realized that at the vehicle longitudinal position, the y-axis was at its maximum point and the polynomial fit to the lane markings were defined as: x = f(y). As such, i used the real world 2nd order polynomial coeffients with the maximum Y values to calculate the lateral position of the left and right lane at Y=maxY. I determined the center of the lane to be:

(Dist<sub>LeftLane</sub> + Dist<sub>RightLane</sub>)/2 +  Dist<sub>LeftLane</sub>. 

I then defined the position error as: 

Position Error = Center of the Lane - Vehicle Center


### Step 7: Define the area between the lane boundaries in the original image.

At this stage, we have sucessfully found our lane markings, defined it with a 2nd order polynomial and determined the lane curvature and vehicle position in the lane. In order to do so, we created a binary image, warped the image and did the calculations on the warped binary image. 

In order mark the Lane we are travelling in, I first took the lane polynomials and defined left and right points. I then used the `fillPoly()` function to color the area between the two polynomials. I finally used the source and destination points defined in Step 4 and flipped them to convert the filled area from the transformed image back to the regular image. This transformed area can then be merged with the orginal image highlight the area between the lines. 

Here is some examples of my results on test images:

 Straight Line Image  |   Curved Line Image 
:-------------------------:|:-------------------------:
![alt text][image26] | ![alt text][image27]

---

### Step 8: Display image with calculated lane boundaries and numerical estimation of lane curvature and vehicle position imposed over original  image.

I then utilized the `putText()` function to write ontop of the image from Step 7 to generate the final image with curvature, position data superimposed with the lane boundary highlighted on the original image:

![alt text][image28]

### Step 9: Creat a pipeline to run on a video feed.

The first step to creating the pipeline was to define a class `Line()` with which to store some properties of the left and right lane line. 

I then created a pipeline where i replicated steps 1-8 above. In addition, I added some sanity checks to ensure the lane lines do not do anything funny. To begin with, I ensure that the absolute difference between the left and right curvature is bounded. I have different error tolerances for when the curvature is above 1000m vs. when it is below as i found the errors between the left and right side can be quite drastic in straight roads. I also calculated the lane width and utilized a lanewidth check to ensure that the we discarded any frames where the lane width did not make sense. After trying these two sanity checks, i realized that there were still some funny things happening in some parts of the video, as such i implemented sanity checks on the 1st and 2nd order lane polynomial coefficients to ensure that the right and left lane polynomials were fairly similar. This greatly improved my pipeline. In order to smooth the video stream a little bit, i implemented a simple moving average to use when the sanity checks failed. I noticed that this added some delay and caused issues with the lane lines jumping, as such i did not utilize this. My project video can be seen here: [Project Video Output](./output_images/project_video.mp4) 

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

So I ultimately found this project quite challenging. Even if the lane lines looked good in the original image, it sometimes struggled in the video feed. I realized that i likely filtered out too much information with my binary thresholds in order to reduce the noise and this has hurt my ability to detect lane lines in some challenging scenario's. I also had difficulties in implementing the `appendLine()` function with the video feed as it kept causing the script to crash. As i could not root cause this issue in the available time, i decided to stick with the histogram approach for every frame. 

Considering my implementation, i believe my pipeline will fail when there are more complex scenario's such as shadows and bad lighting, without enough good frames to reset the lane polynomial in between. Furthermore, i did not implement techniques to detect lane lines when the curvature is very steep, so i know that my pipeline will struggle with the challenge video's. 

If i am to improve the pipeline, i will do the following:

1. I will adjust the binary image thresholds to let in more information so i can get an lane polynomial estimation in harsh conditions, even if it is a poor estimate.
2. I will implement a more robust filter for the lane polynomials, as the moving average filter only introduced a delay without any notible improvement. Some ideas i have are: 
      I.  Implement a Low-Pass Filter
      II. Implement a Kalman Filter
3. I will solve the algorithm to search around the previously detected lane lines
4. I will implement an algorithm for very high curvatures
5. I will improve the sanity checks conducted on the algorithm by looking for additional data that would suggest the lane information is not trusted


All in all i quite enjoyed this project and i am fairly pleased with the results. I will definitely revisit this project in the future when i have time to deep dive into my implementation and perfect it. 
