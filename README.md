##Report of Advanced-Lane-Lines Project

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

[image0]: ./images/corner_detection.png "Corners"
[image1]: ./images/undistorted_chessboard.png "Undistorted"
[image2]: ./images/undistorted_image.png "Straight Road"
[image3]: ./images/threshoud_image.png "Threshould Image"
[image4]: ./images/perspective_region.png "Perspective Region"
[image5]: ./images/undistorted_binary.png "Undistored Lane-line"
[image6]: ./images/window_filtering.png "Filtered Lane-line"
[image6]: ./images/windoe_filtering_poly.png "Filtered Lane-line with polynomial"
[image8]: ./images/old_window_filtering.png "Filtered Lane-line Old method"
[image9]: ./images/result.png "Result with radius and offset"
[video1]: ./output.mp4 "Video"


---
###The Code

####All the code is in the Advanced Lane Lines.ipynb file, including some explaination and demonstrations. I've also save the result to the Advanced+Lane+Lines.html file.


###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

As some cheessboard images is croped, althought the chessboard contains 9x6 corners, and change the size of `objp`` into 9x5 for detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image0]
![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at the **Threshold** section of the source code file).  

I've choosed the parameter like this:

```
gradx = abs_sobel_thresh(undistTest, orient='x', sobel_kernel=3, thresh=(50, 100))
grady = abs_sobel_thresh(undistTest, orient='y', sobel_kernel=3, thresh=(50, 100))
mag_binary = mag_thresh(undistTest, sobel_kernel=5, mag_thresh=(30, 100))
dir_binary = dir_threshold(undistTest, sobel_kernel=7, thresh=(0.7, 1.3))
```
	
Here's an example of my output for this step.  

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `undistort_img()`, which appears in the **Perspective Transform** section of the notebook. 
The `undistort_img()` function takes as inputs an image (`img`). I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([[560, 480], [720, 480],[1080, 720],[200, 720]])
    
dst = np.float32([[300, 50], [980, 50],[980, 720],[300, 720]])
    
M = cv2.getPerspectiveTransform(src, dst)

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 560, 480      | 300, 50       | 
| 720, 480      | 980, 50      |
| 1080, 720     | 980, 720      |
| 200, 720      | 300, 720        |

![alt text][image4]

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image(perspective_straight.jpg) and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image6]

![alt text][image7]

The function in the source code is called `get_filtered_points` in the **Filter lane-line pixals** section. I used the convolution method provided inside the lecture for window filtering.

Another way I've also tried is in the **Old codes** section of the ipynb file, which I write by myself based on histogram to find out centers of each window.

The result(center of each window) is founded like this:
![alt text][image8]


####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the section of **Measuring Curvature and Offset** in my code.

The parameter I choosed is:
* y: 25/720 meters/pixal
* x: 3.7/700 meters/pixal

The output was generated by inverse transform of the region formed by the polynomial

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In the **Final Code** section, I implemented this step.  Here is an example of my result on a test image:

![alt text][image9]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result][video1]

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline will fail when color and light changes at certain point. It's reasonalbe because our method is based on Color space and Sobel operator. The shadow will also affect the result.

To overcome the problem, I use several method in the **Final Code** section of the ipynb file.

1. I ignore the bad result by the following method.
	If the curvature of left and right lane lines looks like:
	* Similar curvature
	* Separated by approximately the right distance horizontally
	* Roughly parallel
	by comparing their polynomial parameters, I'll decide it as a valid result. otherwise it's not. The radius of the curvation is the average result of the left and right lane-lines
2. The result in the video was filtered by averaging the last 10 result of the iterations, the frame will be dropout if it looks like invalid

The final result looks much better than the original one.
