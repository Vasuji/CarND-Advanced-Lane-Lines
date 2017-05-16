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

[image1]: ./pic/undistort.png "Undistorted1"
[image11]: ./pic/undistort2.png "Undistorted2"
[image2]: ./pic/pipeline.png "Pipeline binary"
[image3]: ./pic/perspective.png "Perspective transform"
[image4]: ./pic/fit-poly1.png "Fit polynomial with window search"
[image5]: ./pic/fit-poly2.png "Fit polynomial using previous window"
[image6]: ./pic/plot-lane.png "image-Output"
[video1]: ./project_video_output.mp4 "Video-output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  


***This writeup follows all work done in*** [CARND-Project4.ipynb](https://github.com/Vasuji/CarND-Advanced-Lane-Lines/blob/master/CARND-project4.ipynb)

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2,3, and 4 cell of the IPython notebook called `CARND-Project4.ipynb`. 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image11]


----------------



### Pipeline (single images)


#### 1. Provide an example of a distortion-corrected image.

The camera matrix ```mtx``` and distortion coefficients ```dst``` calculated during camera calibration is saved in pickle file which can be used whenever required. I used that pickle file and produced undistorted images using function ```undistort ``` at cell 7 and 8 . Following are the sample undistorted images I produced.

![alt text][image1]



#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I wrote a class called ```Gradient``` which includes all requied functions and shared data between them. I used a combination of color and gradient thresholds to generate a binary image. These steps are in cell:14,15,16,17. I specifically applied ``` hls_lthresh``` and ```lab_bthresh``` and combined these two to produce final output from ```pipeline``` function.  Here's an example of my output for this step.  


![alt text][image2]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()` which is in cell 9 of Ipython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
apex, apey = 360, 258
offset_far = 50
offset_near = 50


# source points
src = np.float32([[int(apex-offset_far),apey],
                  [int(apex+offset_far),apey],
                  [int(0+offset_near),390],
                  [int(720-offset_near),390]])


#destination points
dst = np.float32([[0,0],[720,0],[0,405],[720,405]])

```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I wrote a class called ``PolyFit`` by collecting all required functions and shared data to find polynomial and curvature etc. First I applied sliding window search as in function ```sliding_window_polyfit``` function in my `PolyFit` class to find initial polynomial and its starting point by using ```histogram``` and ```poly_coff``` supporting function within same class. Following is the image from this step:

![alt text][image4]

 Once the starting polynomial is found, the consecutive images in video frame get benefited from previously found polynomial and use ```polyfit_using_prev_fit``` within function ```skip_window_search```. Following is the result from this step:
 
![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In my PolyFit class at cell 17, there is ```curvature_and_center_dist``` function which takes polynomial coefficients and lane indices and calculates curvature and center. Sample curvature and center has been produced at cell 20.



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I used ```display info``` and ```draw_lane``` function along with functions from ```PolyFit``` class to produce lane plotted image. ```draw_lane``` function interestingly use inverse perspective transform using ```Minv``` which was calculated while we did perspective transform. These are shown in cell-21,22 and 23. These are few sample outpus:

![alt text][image6]


-----------------




### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
Finally I wrote Image Processing Pipeline as ```ImageProcessing``` which further uses ```Line``` class to update lane data in video frame, and use ```intercept_checker``` to validate lane found . It gos through all the steps developed so far: image pipeline, lane detection, polynomial construction, curvature and center calculation and display of lane and information.

Here's a [link to my video result](./project_video_output.mp4)

-------------------






### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1. ***Shadow***: Shadow is one of the issue which changes the pixel values suddenly, there should be some method( for example: make a quick study of neighbourhood pixel values to identify it is shadow and change threshold values) to acknowledge there is shadow now and threshold value has to be changed in this case.

2. ***Yellow line detection***: Yellow line detection is very different then white line detection. I implemented other colorspace except ```RBG``` for binary image.

3. ***More then two lanes detected***: This may occur if we ignore some validation step for two lines being a pair og lane. I used ```intercept_checker``` in cell 25 which decides whether two line detected are pair of lane.

4. ***Sharp bendings***: My pipeline could not perform good in extra challange videos. This is mainly because of many shadows and sharpbendings.

5. ***Rainy day and poor vision***: In these cases only camera photos may not provide enough information.


