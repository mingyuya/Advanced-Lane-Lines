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
[image9]: ./figures/fitted_lines.png "Fitted Lines"
[image10]: ./figures/radiusCurvature.png "Formular for Radius of Curvature"
[image11]: ./figures/detected_lane.png "Detected Lane"
[video1]: ./result_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration 
##### (Placed in the cell with the title of 'Compute the camera calibration matrix and distortion coefficients' in Advanced-Lane-Lines.ipynb

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection using the `cv2.findChessboardCorners()` function.  

I then used the output `objpoints` and `imgpoints` to compute the **camera calibration matrix** and **distortion coefficients** using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
![alt text][image1]

---

### Pipeline (Single images)

1) Load camera calibration matrix and distortion coefficients  
2) **Undistort**  
3) **Binarize** with some thresholds  
4) **Warp** binarized image to bird-eye view  
5) **Find lane lines** and **fitting** 
6) **Update** the instances  
7) **Calculate** radius of curvature and distance from lane center

#### Test Image
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one :  
![alt text][image2] 

#### Binary Image of Lane Lines

I used a combination of L-channel, S-channel and color and gradient thresholds to generate a binary image. Most of cases, it is easy to pick lane lines from an image by using combination of those specific color channels. In the other hand, using gradient shows better result, if there are shaded area or variation of background color in an image.

Here's an example of my output for this step.

![alt text][image4]
![alt text][image5]

After combining two images above by logical-OR operation, I wiped out unnecessary pixels from the image using `mask_image()` function.

All the steps described above are coded in `detect_edge()` function.

#### Perspective Transform
: in the **'Perspective Transform'** cell in `Advanced-Lane-Lines.ipynb`.

The code for my perspective transform includes a function called `warp_image()`. The function takes as inputs an image (`image`), as well as source points (`src`) and enable/disable inverse transform (`inv`).  I chose the fixed points as the source and destination points in the following manner:

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

![alt text][image6]

#### Identifying lane-line pixels and Fitting their positions with a 2nd order polynomial
: in the **'Sliding Windows and Fitting a Polynomial'** cell in `Advanced-Lane-Lines.ipynb`.

1) Finding start position  
At first, I get the histogram of the binary warped image along x-axis. After that, I chose the indices of two maximum value of the histogram as starting position, `leftx_base` and `rightx_base`. Here is the histogram of test image :  

![alt text][image7]

2) Sliding windows and get indices of lane lines  
The window its size is defined by `nwindows` and `margin` moves and searches every indices of non-zero pixels in warped binary image.  

![alt text][image8]

The `find_lines()` function performs two processes above. Its results are used for fitting 2nd order polynomials to each lane line. The fitting procedure appears in the `Test 'find_lines' function` cell. Here is its result :  

![alt text][image9]

#### Calculate the radius of curvature of the lane and the position of the vehicle with respect to center

Fisrt fo all, I assume that the test video, `project_video.mp4`, is in under the U.S. regulations so that, the length and width of the lane in the video were setted to 30m and 3.7 meter, separately. By using the assumption, it was possible to get the coefficients for  polynomials for each line in real-world scale. Finally, the radius of curvature was calculated by this formular :     

```html
<blockquote>
  <p><span id="mathId0"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mstyle scriptlevel="0" displaystyle="true"><mtext><mi mathvariant="normal">R</mi><mi mathvariant="normal">a</mi><mi mathvariant="normal">d</mi><mi mathvariant="normal">i</mi><mi mathvariant="normal">u</mi><mi mathvariant="normal">s</mi><mtext>&nbsp;</mtext><mi mathvariant="normal">o</mi><mi mathvariant="normal">f</mi><mtext>&nbsp;</mtext><mi mathvariant="normal">c</mi><mi mathvariant="normal">u</mi><mi mathvariant="normal">r</mi><mi mathvariant="normal">v</mi><mi mathvariant="normal">a</mi><mi mathvariant="normal">t</mi><mi mathvariant="normal">u</mi><mi mathvariant="normal">r</mi><mi mathvariant="normal">e</mi></mtext><mo>=</mo><mfrac><mrow><mrow><msup><mrow><mrow><mo fence="true">[</mo><mrow><mn>1</mn></mrow><mo>+</mo><msup><mrow><mrow><mo fence="true">(</mo><mfrac><mrow><mrow><mrow><mrow><mrow><mi>d</mi></mrow><mrow><mi>y</mi></mrow></mrow></mrow></mrow></mrow><mrow><mrow><mrow><mrow><mrow><mi>d</mi></mrow><mrow><mi>x</mi></mrow></mrow></mrow></mrow></mrow></mfrac><mo fence="true">)</mo></mrow></mrow><mrow><mn>2</mn></mrow></msup><mo fence="true">]</mo></mrow></mrow><mrow><mrow><mrow><mn>3</mn></mrow><mtext><mi mathvariant="normal">/</mi></mtext><mrow><mn>2</mn></mrow></mrow></mrow></msup></mrow></mrow><mrow><mrow><mrow><mrow><mo fence="true">∣</mo><mfrac><mrow><mrow><msup><mrow><mi>d</mi></mrow><mrow><mn>2</mn></mrow></msup><mrow><mi>y</mi></mrow></mrow></mrow><mrow><mrow><msup><mrow><mrow><mrow><mi>d</mi></mrow><mrow><mi>x</mi></mrow></mrow></mrow><mrow><mn>2</mn></mrow></msup></mrow></mrow></mfrac><mo fence="true">∣</mo></mrow></mrow></mrow></mrow></mfrac></mstyle></mrow><annotation encoding="application/x-tex">\displaystyle\text{Radius of curvature}=\frac{{{\left[{1}+{\left(\frac{{{\left.{d}{y}\right.}}}{{{\left.{d}{x}\right.}}}\right)}^{2}\right]}^{{{3}\text{/}{2}}}}}{{{\left|\frac{{{d}^{2}{y}}}{{{\left.{d}{x}\right.}^{2}}}\right|}}}</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="strut" style="height: 2.39413em;"></span><span class="strut bottom" style="height: 4.10213em; vertical-align: -1.708em;"></span><span class="base textstyle uncramped"><span class="mord text displaystyle textstyle uncramped reset-textstyle displaystyle textstyle uncramped"><span class="mord mathrm">Radius&nbsp;of&nbsp;curvature</span></span><span class="mrel reset-textstyle displaystyle textstyle uncramped">=</span><span class="mord reset-textstyle displaystyle textstyle uncramped"><span class="mopen sizing reset-size5 size5 reset-textstyle textstyle uncramped nulldelimiter"></span><span class="mfrac"><span class="vlist"><span class="" style="top: 1.05798em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 1em;">​</span></span><span class="reset-textstyle textstyle cramped"><span class="mord textstyle cramped"><span class="mord textstyle cramped"><span class="mord textstyle cramped"><span class="minner textstyle cramped"><span class="mopen style-wrap reset-textstyle textstyle uncramped"><span class="delimsizing mult"><span class="vlist"><span class="" style="top: 0.650015em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="delimsizinginner delim-size1"><span class="">∣</span></span></span><span class="" style="top: 0.044015em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="delimsizinginner delim-size1"><span class="">∣</span></span></span><span class="" style="top: -0.561985em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="delimsizinginner delim-size1"><span class="">∣</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span>​</span></span></span></span><span class="mord reset-textstyle textstyle cramped"><span class="mopen sizing reset-size5 size5 reset-textstyle textstyle uncramped nulldelimiter"></span><span class="mfrac"><span class="vlist"><span class="" style="top: 0.371228em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-textstyle scriptstyle cramped mtight"><span class="mord scriptstyle cramped mtight"><span class="mord scriptstyle cramped mtight"><span class="mord mtight"><span class="mord scriptstyle cramped mtight"><span class="minner scriptstyle cramped mtight"><span class="mopen sizing reset-size5 size5 reset-scriptstyle textstyle uncramped nulldelimiter"></span><span class="mord scriptstyle cramped mtight"><span class="mord mathit mtight">d</span></span><span class="mord scriptstyle cramped mtight"><span class="mord mathit mtight">x</span></span><span class="mclose sizing reset-size5 size5 reset-scriptstyle textstyle uncramped nulldelimiter"></span></span></span><span class="msupsub"><span class="vlist"><span class="" style="top: -0.34144em; margin-right: 0.0714286em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-scriptstyle scriptscriptstyle cramped mtight"><span class="mord scriptscriptstyle cramped mtight"><span class="mord mathrm mtight">2</span></span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span>​</span></span></span></span></span></span></span></span><span class="" style="top: -0.23em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-textstyle textstyle uncramped frac-line"></span></span><span class="" style="top: -0.446108em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-textstyle scriptstyle cramped mtight"><span class="mord scriptstyle cramped mtight"><span class="mord scriptstyle cramped mtight"><span class="mord mtight"><span class="mord scriptstyle cramped mtight"><span class="mord mathit mtight">d</span></span><span class="msupsub"><span class="vlist"><span class="" style="top: -0.286em; margin-right: 0.0714286em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-scriptstyle scriptscriptstyle cramped mtight"><span class="mord scriptscriptstyle cramped mtight"><span class="mord mathrm mtight">2</span></span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span>​</span></span></span></span><span class="mord scriptstyle cramped mtight"><span class="mord mathit mtight" style="margin-right: 0.03588em;">y</span></span></span></span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span>​</span></span></span><span class="mclose sizing reset-size5 size5 reset-textstyle textstyle uncramped nulldelimiter"></span></span><span class="mclose style-wrap reset-textstyle textstyle uncramped"><span class="delimsizing mult"><span class="vlist"><span class="" style="top: 0.650015em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="delimsizinginner delim-size1"><span class="">∣</span></span></span><span class="" style="top: 0.044015em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="delimsizinginner delim-size1"><span class="">∣</span></span></span><span class="" style="top: -0.561985em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="delimsizinginner delim-size1"><span class="">∣</span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span>​</span></span></span></span></span></span></span></span></span></span><span class="" style="top: -0.23em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 1em;">​</span></span><span class="reset-textstyle textstyle uncramped frac-line"></span></span><span class="" style="top: -1.04002em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 1em;">​</span></span><span class="reset-textstyle textstyle uncramped"><span class="mord textstyle uncramped"><span class="mord textstyle uncramped"><span class="mord"><span class="mord textstyle uncramped"><span class="minner textstyle uncramped"><span class="mopen style-wrap reset-textstyle textstyle uncramped" style="top: 0em;"><span class="delimsizing size2">[</span></span><span class="mord textstyle uncramped"><span class="mord mathrm">1</span></span><span class="mbin">+</span><span class="mord"><span class="mord textstyle uncramped"><span class="minner textstyle uncramped"><span class="mopen style-wrap reset-textstyle textstyle uncramped" style="top: 0em;"><span class="delimsizing size2">(</span></span><span class="mord reset-textstyle textstyle uncramped"><span class="mopen sizing reset-size5 size5 reset-textstyle textstyle uncramped nulldelimiter"></span><span class="mfrac"><span class="vlist"><span class="" style="top: 0.345em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-textstyle scriptstyle cramped mtight"><span class="mord scriptstyle cramped mtight"><span class="mord scriptstyle cramped mtight"><span class="mord scriptstyle cramped mtight"><span class="minner scriptstyle cramped mtight"><span class="mopen sizing reset-size5 size5 reset-scriptstyle textstyle uncramped nulldelimiter"></span><span class="mord scriptstyle cramped mtight"><span class="mord mathit mtight">d</span></span><span class="mord scriptstyle cramped mtight"><span class="mord mathit mtight">x</span></span><span class="mclose sizing reset-size5 size5 reset-scriptstyle textstyle uncramped nulldelimiter"></span></span></span></span></span></span></span><span class="" style="top: -0.23em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-textstyle textstyle uncramped frac-line"></span></span><span class="" style="top: -0.446108em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-textstyle scriptstyle uncramped mtight"><span class="mord scriptstyle uncramped mtight"><span class="mord scriptstyle uncramped mtight"><span class="mord scriptstyle uncramped mtight"><span class="minner scriptstyle uncramped mtight"><span class="mopen sizing reset-size5 size5 reset-scriptstyle textstyle uncramped nulldelimiter"></span><span class="mord scriptstyle uncramped mtight"><span class="mord mathit mtight">d</span></span><span class="mord scriptstyle uncramped mtight"><span class="mord mathit mtight" style="margin-right: 0.03588em;">y</span></span><span class="mclose sizing reset-size5 size5 reset-scriptstyle textstyle uncramped nulldelimiter"></span></span></span></span></span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span>​</span></span></span><span class="mclose sizing reset-size5 size5 reset-textstyle textstyle uncramped nulldelimiter"></span></span><span class="mclose style-wrap reset-textstyle textstyle uncramped" style="top: 0em;"><span class="delimsizing size2">)</span></span></span></span><span class="msupsub"><span class="vlist"><span class="" style="top: -0.764em; margin-right: 0.05em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-textstyle scriptstyle uncramped mtight"><span class="mord scriptstyle uncramped mtight"><span class="mord mathrm mtight">2</span></span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span>​</span></span></span></span><span class="mclose style-wrap reset-textstyle textstyle uncramped" style="top: 0em;"><span class="delimsizing size2">]</span></span></span></span><span class="msupsub"><span class="vlist"><span class="" style="top: -0.829108em; margin-right: 0.05em;"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span><span class="reset-textstyle scriptstyle uncramped mtight"><span class="mord scriptstyle uncramped mtight"><span class="mord scriptstyle uncramped mtight"><span class="mord scriptstyle uncramped mtight"><span class="mord mathrm mtight">3</span></span><span class="mord text scriptstyle uncramped mtight"><span class="mord mathrm mtight">/</span></span><span class="mord scriptstyle uncramped mtight"><span class="mord mathrm mtight">2</span></span></span></span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 0em;">​</span></span>​</span></span></span></span></span></span></span></span><span class="baseline-fix"><span class="fontsize-ensurer reset-size5 size5"><span class="" style="font-size: 1em;">​</span></span>​</span></span></span><span class="mclose sizing reset-size5 size5 reset-textstyle textstyle uncramped nulldelimiter"></span></span></span></span></span></span></p>
</blockquote>
```

The code for the formular is 
```
ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/800 # meters per pixel in x dimension

# leftx, lefty, rightx, righty are the result of find_lines() function
left_fit_m = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
right_fit_m = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)

y_eval = np.max(yvals)*ym_per_pix
left_curverad = ((1 + (2*left_fit_m[0]*y_eval + left_fit_m[1])**2)**1.5) / np.absolute(2*left_fit_m[0])
right_curverad = ((1 + (2*right_fit_m[0]*y_eval + right_fit_m[1])**2)**1.5) / np.absolute(2*right_fit_m[0])
```

This part of code is also appears in in the `Test 'find_lines' function` cell.

#### Projection of found lane onto the input image

I implemented this step in the `draw_lane()` function. The function requires the following as its input parameters :  
* Original image to be drawn  
* Source points of warped image  
* Positions of fitted lines  
* The radius of curvature  
* The distance between the vehicle and the centor of lane  

Here are **the sequence of processing those inputs** and **its result** :
1) Drawing the polygon indicating the lane  
2) Dewarping the found lane  
3) Projection of step 2 onto the original image + Writing the radius and the distance

![alt text][image11]

---

### Pipeline for Processing Video

All the processing stages mentioned above were merged to the `process_image()` function. The function not only redrawn image, but also calculates the characteristics of the lane appears in each frame and stores it to `Line` class. For the radius of curvature, I used the mean of coefficients for left and right lane lines.

Here's a [link to my video result](./result_project_video.mp4)

---

### Discussion

I'll add some pipelines to ignore black lines appears in `challenge_video.mp4`. Because it doesn't have any information about the direction that vehicle has to be ahead and makes unwanted edges

