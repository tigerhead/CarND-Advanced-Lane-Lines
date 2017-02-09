

###Advance Lane Finding Project

Resource code and step by step description in root folder: p4.ipynb

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in code cells number 2 and 3 of the IPython notebook located in "p4.ipynb..  

I used 20 chessboard images provided with this project. I chose use (x, y)= (9, 6) to search corner in test images. And I created   "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time corners were successfully detected.   `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  Corners were correctly detected in 18 out of 20 test images.

Output `objpoints` and `imgpoints` were used to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  Here is undistorted test image: 

![Camera Calibration](/output_images/Calibrate_Camera.png)

And camera calibration matrix and distortion coefficeents were saved in a pickle file(/camera_cal_output/cmera_dist_pickle.p) for later use.
###Pipeline (single images)
I picked test2.jpg image in test_images folder as example to illustrate my image processing pipline:
![Original Test Image](/test_images/test2.jpg)
####1. Provide an example of a distortion-corrected image.
Camera calibration matrix and distortion coefficeents were loaded from file and cv2.undistort was used to produce a undistorted image. Code can be found in code cell 5

![Test Undistorted Image](/test_images/test_undist.jpg)

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Here is my pipepline to generate a binary image to lane line dectection:
 1) Convert RGB color space to HLS color space
 2) Get X Sobel gradient binary image on L channel using threshold = (15, 90)
 3) Get S channel binary image using threshold = (155, 255)
 4) Combine X Sobel gradent binary image and S channel threhold binary image 
Code can be found in code cell 5
Here is the combined binary image:

![Combined Binary Image](/output_images/test_combined_binary_bw.png)

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `upwarp()`, which can be found in code cell 6 `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  Different combination of source and destination points were tried and following are  source and destination points used in my transformation:

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 460      | 320, 20        | 
| 250, 720      | 320, 680      |
| 1080, 720     | 980, 680      |
| 725, 460      | 980, 20        |

Here a test image, two curved lines are almost parallel after bird-eye view transformation

![Birdeye View Image](/output_images/test_warped_bw.png)

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
On the warped bird view iamge, histogram was calculated at horizontal half line of the image. I used histogram peaks to locate the lane line position. THen use these two peaks as starting points, slice image into 10 sliding windows horizontally. In each window, find non-zero points with nearby small area. Then use those points to fit polynomial line and find lane lane  polynomial coefficient. Code can be found  `blind_lane_finder` function in code cell 7. Following are result images for sliding window technique and fitted ploynomial lines 

![Sliding Window Image](/output_images/test_window_slide.png)T

![Fitted line Image](/output_images/test_fit_lines.png)

In video frame processing, if lane lines have been found in previous frame, in currrent frame, the lane line detection can start with position of previous found lane assuming that lane position will have very small change frame by frame. And if line found in this way doesn't work well, try to use the technique above to find lane line from scratch.

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Cuvature of the lane is caculated use lane line polynomial coefficients, code can be found in is_good_dectetion function in code cell 7. 

Position of vehicle with respect to center is calcuted by finding the difference between the center pixel and avearage of two pixels on detected lane line which are on the same horizontal level. Code code can be found in is_good_dectetion function in code cell 7. 


####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

After finding lane lines from bird eye transformed binary image, lines were drawn on a blank color image and reversely warp back to camera perspective. The reversed lane line was marked on the undistorted image, which will be final output image with lane marked in green color. 

Here is an example of my result on a test image:

![output image](/output_images/out_test.png)

####7. Lane detection Sanity Check.
Follow critieria are used to check if a detection is good or not:

1) left and right curvature is close

2)lane witdth is in certain range

3)lines are clost to parallel

Code can be found in is_good_dectetion function in code cell 7. 


Here is an example of my result on a test image:

![output image](/output_images/out_test.png)

---

###Process (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://youtu.be/yWqtZVf5bBU)

Resolution video is uploaded as project_solution_video_v3.mp4

---

###Discussion

####  Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here are some challenges that I had in this project:

1. Perspective transformation. I spent a lot of time trying to find good source and destination points to transform image to bird-eye view. I tested lots of combination. The source and destination points that I used work ok for this project but I don't think they are very good. I am wondering whether there is a good way to find proper source and destination sets.

2. I found my pipeline doesn't work too well on bright area even if I used HSL color space. Maybe, my threshold can be improved. And combination of different binary image can make the pipeline more robust.

3. I used simple average method to get average fit of last 5 good frames. It works OK in project video but I am not sure if it is the right way to do it.

