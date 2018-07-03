
# **Advanced Lane Finding Project**

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

[image1]: ./writeup/cal_undistorted.png "Undistorted"
[image2]: ./writeup/cal_original.png "Uncalibrated"
[image3]: ./writeup/test_image_undistortion.png "Test Image Undistortion"
[image4]: ./writeup/hls_s.png "HLS-S"
[image5]: ./writeup/polyfit_rgb_r.png "Polyfit"
[image6]: ./writeup/lane_detection.png "Output"
[image7]: ./writeup/rgb_r.png "RGB-R"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook "CarND-Advanced-Lane-Lines.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Original distorted image][image2] ![Undistorted image][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the same distortion matrix calculated from chessboard pattern, I applied the distortion correction to one of the test images like this one:

![Test Image Undistortion][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I visualized all of the test images in three colorspaces: RGB, HSV and HLS. Three channels of each are saperately visualized in 5th to 13th, a total of 9 code cells in the Jupyter notebook, to see if any, or a combination, of channels can be used to effectively filter out just the lane lines in all of the test images that represent a sufficient amount of variation in lightening conditions, required for this project. And sure enough, one could clearly see that S-Channel in the HLS colorspace works well on all the the test images.

Here is the result of test images in the S-Channel of HLS colorspace:

![Test Images in HLS-S][image4]

I could also see that with the right thresholds RGB R-channel could also provide the relevant information sufficiently. So I took note of that but proceeded with the pipeline using HLS S-Channel. On the final video though, the results weren't too promising so I decided to test out RGB R-Channel instead. This worked out really well. By applying just few further constraints in the pipeline, just the RGB R-Channel filtering produced the desired result in the end. Here is how RGB R-Channel looked on the test images:

![Test Images in RGB-R][image7]

I then used the minimum and maximum thresholds of the values in the RGB R-Channel to extract just the pixels on the lane lines and create the binary image.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 14th code cell of the Jupyter notebook). The `warper()` function takes as inputs an image (`img`) and the Transformation Matrix (`M`). This way, the Matrix M is needed to be calculated just once, making processing more efficient. 

M is calucalted using `cv2.getPerspectiveTransform` function, which takes in source (`src`) and destination (`dst`) points.  I chose to calculate the source and destination points based on the image dimesions in the following way:

```python
# get image height and width
height, width = img.shape[:2]

# set source points 
bottom_left_src = (0.1 * width, 0.95 * height)
bottom_right_src = (0.9 * width, 0.95 * height)
top_left_src = (0.46 * width, 0.65 * height)
top_right_src = (0.58 * width, 0.65 * height)
source = np.float32([bottom_left_src, bottom_right_src, top_left_src, top_right_src])

# set destination pints
bottom_left_dst = (0.3 * width, height)
bottom_right_dst = (0.7 * width, height)
top_left_dst = (0.3 * width, 0) 
top_right_dst = (0.7 * width, 0)
dest = np.float32([bottom_left_dst, bottom_right_dst, top_left_dst, top_right_dst])
```

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In my pipeline, I created the binary images from the images that were first applied the percpective transform to get the bird-eye view.

In order to fit the polynomial onto the lane lines, I first filtered out the noise from the sides and top by cropping the binary image 25% from each side and 60% from the top. I then applied the histogram onto the cropped image to identify the peaks of pixel density to be used as the starting points for the left and right lanes to apply polyfit. From these starting points, I propagate up the binary image in the windowed manner, with a total of 10 windows each for left and right lanes. In each window I identify the pixels belonging to the lane. After finding the location of the lane in each of the windows, I use those points to polyfit the line onto the binary image for each of the left and right lane. The result on the test images was as follows:

![Polyfit][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Based on the result of the lane lines I computeed through polyfitting, I calculated the curvature and raius of each lane line,  left and right, in the 21st code cell of the Jupyter Notebook

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I tested the whole pipeline in the 26th code cell of my Jupyter Notebook on the test images. Here is an example:

![Pipeline Result][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline is performing reasonably well on the given video. But it can be seen that it is still a little wobbly and not perfect. I am, for the most part, leveraging only the RGB R-channnel information. So it is probably advisable to use other colospace and gradient-based filtering in combination to make the results more robust. The whole pipeline heavily relies on a single viewing perspective. So it would immediately fail if the camera angle or placement is changes a little. 