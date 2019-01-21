## Project writeup for Advance lane detection


---

**Advanced Lane Finding Project**

The steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./camera_cal/calibration1.jpg "Distorted chessboard"
[image2]: ./camera_cal_undistort/calibration1.jpg "Undistorted Chessboard"
[image3]: ./test_images_undistort/straight_lines1.jpg "Undistorted test image"
[image4]: ./test_images_threshold/straight_lines1.jpg "Threshold Binary"
[image5]: ./test_images_warped/straight_lines1.jpg "Warped"
[image6]: ./test_images_output/straight_lines1.jpg "Final Output"
[image7]: ./test_images/straight_lines1.jpg "Distorted test image"
[image8]: ./test_images_sliding/straight_lines1.jpg "Sliding window"
[video1]: ./project_video_output.mp4 "Video"



### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

This is the first step in advance lane detection. Lenses in cameras introduce distortion in the images which is called radial distortion. Images need to be undistorted. This step involves following substeps. 

1. Creation of object points which is 3D representation of image and image points which is 2D representation. This is done by using the chessboard images. **cv2.findChessboardCorners()** is used for finding the inner corners in the chessboard image and then those corners act as image points and we have object points as well. We can draw corners on chessboard images using **cv2.drawChessboardCorners()** function.

2. Then **cv2.calibrateCamera()** function is used to calculate the camera matrix, distortion coefficients and other coefficients and camera matrix and distortion coefficients are used to undistort the image.

3. Then **cv2.undistort()** function uses matrix and distortion coefficients to give the undistorted image. Following is the examples of distorted and undistorted images.

![Distorted_image][image1]



![Undistorted_image][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Following is the input test image and distortion corrected image which has been obtained by applying the undistortion steps.


![Distorted_test_image][image7]


![Undistorted_test_image][image3]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Next step is to get the thresholded binary image which is like Canny detection of edges but this uses various color spaces and sobel functions.

This is one of the most important part in the lane detection as this decides how lanes are detected.

We have various functions like **x and y sobel, magnitude sobel, direction sobel** as various gradient methods.

We can apply various thresholds on various components of color spaces like RGB, HLS, etc color spaces.

1. x sobel : This emphasizes edges closer to vertical axis.

2. y sobel : This emphasizes edges closer to hosrizontal axis.

3. Magnitude sobel : This works on magnitude and use both x and y sobel.

4. Direction sobel : This uses arctan function and gives direction. 0 implies vertical line and +/- 90 degrees denotes horizontal line.

5. Color spaces : We have various color spaces and I tried to use RGB and HLS color spaces.

I tried various thresholds of above mentiones thresholds and finally settled on using X and magnitude sobel along with thresholded R from RGB and S from HLS color spaces.

Function **threshold()** is being created for thresholding.

Following is the image representing the tresholded binary image.





#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

After thresholded image, next step is to perform the perspective transform. Perspective transform gives bird eye view of the lanes detected in thresholded binary image.

I have created function **warping()** for this which returns perspetive transformed image as well as inverse matrix for inverse perspective transform and used following values after going through the process similar to the region selection.
Following values have been used for perspective transformation. 



| Source        | Destination   | 
|:-------------:|:-------------:| 
| 500, 500      | 0, 0          | 
| 810, 500      | 1280, 0       |
| 130, 720      | 0, 720        |
| 1250,720      | 1280, 720     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Warped_image][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

After perspective transform, I used histogram sliding window technique to fit a second order polynomial. In this method image is divided into 2 halves, right and left respectively, for right and left lanes and both the halves are scanned in smaller windows which is a hyperparameter and window width is given by a configurable hyperparameter.

Following image describes the sliding window approach to fit polynomial.

![Sliding_window][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of curvature has been calculated by formula given at [Radius of curvature](https://www.intmath.com/applications-differentiation/8-radius-curvature.php). Basic idea is to find the radius of curvature of the curve at a particular point is defined as the radius of the approximating circle.

So derivation was required but second order polynomial derivative is the slope of the line.

Function **measure_curvature_real()** is used for this purpose.



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Plotting lanes and filling the lane lines requires inverse persepective transform and then filling the polynomial between both the lane lines.

Inverse perspective transform requires Inverse matrix which can be calculated by just reversing the **src and dst** in the function **cv2.getPerspectiveTransform()** and then using the **cv2.warpPerspective()** in the similar way as perspective transform was found.

![Final_output][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Following is the link of my project video.

[Video](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

During the implementation of this project following were the challenges :

1. Thresholding the image and video pipeline was most challenging part as we have various gradient threshold and lot of color spaces like RGB, HSV, HLS, etc which can be used to make the lane detection more robust. Tuning all these parameters together and keep on checking the output was very challenging. Sometimes threshold was working on images but was not working on the video. So again retuning was required.

2. Video pipeline was also required careful consideration of steps. Project video was having change of color of road and the pipeline which was working fine but was failing on video and both lines were getting disappeared from the video and later on it was found that both lines were started having same polynomial fit co-efficients which was happening when video was entering from lighter shade to darker shade. To resolve this issue sanity check was put in place for both the lines like checking the length of the fits, checking the difference between slope of both the lines and checking the position of vehicle which made the video pipeline more robust.


Following are the points where this pipeline can fail:

1. if the first frame is not able to find the fit for either right or left lane lines then this pipeline will give error.

2. On sharper turns this can give erratice behaviour.


This pipeline can be further improved by following methods :

1. Detect both lines individually and if any of the line is found then go ahead with plottin of one line and keep on trying to detect the other line.

2. Use some other color spaces as well to make lane detection more robust in various light condition.