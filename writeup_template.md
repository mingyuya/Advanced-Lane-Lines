# Writeup for P4:Advanced Lane Finding

### Matt, Min-gyu, Kim
---

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

[image1]: ./figures/calibration.png "Undistorted"
[image2]: ./figures/test_image.png "Road Transformed"
[image3]: ./figures/masked.png "Region of Interest"
[image4]: ./figures/c_binary.png "Binary_1"
[image5]: ./figures/grad.png "Binary_2"
[image6]: ./figures/warped.png "Warped"
[image7]: ./figures/histogram.png "Histogram"
[image8]: ./figures/find_lane_line.png "Finding Lines"
[image9]: ./figures/warped.png "Warped"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration 
##### (Placed in the cell with the title of 'Compute the camera calibration matrix and distortion coefficients' in Advanced-Lane-Lines.ipynb

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection using the `cv2.findChessboardCorners()` function.  

I then used the output `objpoints` and `imgpoints` to compute the **camera calibration matrix** and **distortion coefficients** using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (Single images)

1) Load camera calibration matrix and distortion coefficients  
2) **Undistort**  
3) **Binarize** with some thresholds  
4) **Warp** binarized image to bird-eye view  
5) **Find lane lines** and **fitting** 
6) **Update** the instances  
7) **Calculate** radius of curvature and distance from lane center

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2] 

##### Binary Image of Lane Lines

I used a combination of L-channel, S-channel and color and gradient thresholds to generate a binary image. Most of cases, it is easy to pick lane lines from an image by using combination of those specific color channels. In the other hand, using gradient shows better result, if there are shaded area or variation of background color in an image.

Here's an example of my output for this step.

![alt text][image4]
![alt text][image5]

After combining two images above by logical-OR operation, I wiped out unnecessary pixels from the image using `mask_image` function.

##### Perspective Transform

The code for my perspective transform includes a function called `warp_image()`, which appears in lines 1 through 8 in the cell with the title of 'Perspective Transform' in `Advanced-Lane-Lines.ipynb`.  The `warp_image()` function takes as inputs an image (`image`), as well as source points (`src`) and enable/disable inverse transform (`inv`).  I chose the fixed points as the source and destination points in the following manner:

Source Point
```
h, w = warp_in.shape[0], warp_in.shape[1]

src_LB = [200   , 700     ]   # Left-Bottom
src_LT = [w*0.44, h*0.65  ]  # Left-Top
src_RT = [w*0.557, h*0.65 ]  # Right-Top
src_RB = [1100  , 700     ]   # Right-Bottom
src = np.float32([src_LB, src_LT, src_RT, src_RB])
```
Destination Point
```
offset = 100
dst_LB = [src[0][0]+offset, shape[0]]  # Left-Bottom
dst_LT = [src[0][0]+offset, 0       ]  # Left-Top
dst_RT = [src[3][0]-offset, 0       ]  # Right-Top
dst_RB = [src[3][0]-offset, shape[0]]  # Right-Bottom
dst = np.float32([dst_LB, dst_LT, dst_RT, dst_RB])
```

The following shows the result of `warp_image` function : 

![alt text][image4]

##### Identifying lane-line pixels and Fitting their positions with a 2nd order polynomial

1) Find starting position
At first, I get the histogram of the binary warped image along x-axis. After that, I chose the indices of two maximum value of it as starting position, `leftx_base` and `rightx_base`. Here is the histogram of test image :  
![alt text][image7]

2) Slide windows and get indices of lane lines  


####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

