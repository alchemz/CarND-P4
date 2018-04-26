## Writeup of Advanced Lane Finding


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

[image1]: ./output_images/undistorted_img1.png "Undistorted1"
[image2]: ./output_images/undistorted_img2.png "Undistorted2"
[image3]: ./test_images/test2.jpg "Road Transformed"
[image4]: ./output_images/test_undistorted.png "Undistorted3"
[image5]: ./output_images/test2_unwarped.png "Unwarped"
[image6]: ./output_images/example_o.png "Example Output"
[image7]: ./output_images/undistorted_img3.png "Undistorted3"
[image8]: ./output_images/undistorted_img4.png "Undistorted4"
[image9]: ./output_images/undistorted_img5.png "Undistorted5"
[image10]: ./output_images/undistorted_img6.png "Undistorted6"
[image11]: ./output_images/channel.png "RGB Channel"
[image12]: ./output_images/sobel.png "RGB Channel"
[image13]: ./output_images/example_r.png "Example Output"
[image14]: ./output_images/failed.png "Example Output"
[image15]: ./output_images/updated.png "Success Output"
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "Advanced Lane Finding.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]
![alt text][image2]
![alt text][image7]
![alt text][image8]
![alt text][image9]
![alt text][image10]

For some reasons, the undistorted function obtained best result when applied to the calibration1.jpg image. And the effect on calibration2.jpg image is very minimum. I would assume it is because the calibration1.jpg image is less distorted than the calibration2.jpg image.



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image4]
Apply a perspective transform to rectify binary image ("birds-eye view").

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I have tried to use sobel gradient thresholds and sobel magnitude, yet the result obtained from sobel magnitude does not work well. So later on, I did not combine this two together, but used sobel gradient only.

The result is:
![alt text][image11]
![alt text][image12]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp(img, src, dst)`.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 460      | 450, 0      | 
| 707, 720      | w-450, 720  |
| 260,  680     | 450, h      |
| 1050, 680     | w-450, h    |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

At this point I was able to use the combined binary image to isolate only the pixels belonging to lane lines. The next step was to fit a polynomial to each lane line, which was done by:

- Identifying peaks in a histogram of the image to determine location of lane lines.
- Identifying all non zero pixels around histogram peaks using the numpy function numpy.nonzero().
- Fitting a polynomial to each lane using the numpy function numpy.polyfit().

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I used the following code to calculate the radius of curvature for each lane line in meters:

- ym_per_pix = 3.048/100 # meters per pixel in y dimension, lane line is 10 ft = 3.048 meters
- xm_per_pix = 3.7/378 # meters per pixel in x dimension, lane width is 12 ft = 3.7 meters

- left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
- right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)

- left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
- right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])

source: https://www.intmath.com/applications-differentiation/8-radius-curvature.php

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is an example of my result on a test image:

![alt text][image13]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://www.youtube.com/watch?v=1GAZnkWhImk)

Here's a [link to my challenge video result](https://www.youtube.com/watch?v=7y_rS15v4ck)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
Overall, the pipeline design has done comparatively robust job of detecting. And I would like to talk about some problems I have met. For example, here is a example frame captured from the video result:
![alt text][image14]

When there is a bigging turning point when driving, or things like shadows, light conditions would also affect the tracking, the result tracked is not very good. 


For the this submission, I discovered the B channel of the LAB colorspace, which isolates the yellow lines very well.
Here is the same frame captured use, which has a better tracking result.
![alt text][image15]	