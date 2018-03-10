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

[image1]: ./output_images/chessboard_undistorted.jpg "Undistorted"
[image2]: ./output_images/undistorted.jpg "Road Transformed"
[image3]: ./output_images/thresholded.jpg "Binary Example"
[image4]: ./output_images/binary_warped.jpg "Warp Example"
[image5]: ./output_images/lanes.jpg "Fit Visual"
[image6]: ./output_images/result_with_radius_of_curvature.jpg "Output"
[video1]: ./project_video.mp4 "Video"
[image7]: ./output_images/original.jpg "Original Image"
[image8]: ./output_images/histogram.jpg "Histogram"
[image9]: ./output_images/sliding_window.jpg "Sliding Window"
## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 1 and 2 cell of the IPython notebook located in "./AdvanceLaneFindings.ipynb"  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

Original image :

![alt text][image7]

The Distortion Matrix and coefficients were then used to undistort any given image taken from the camera.

Undistorted:

![alt text][image2]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is contained in the 7th cell of the IPython notebook located in "./AdvanceLaneFindings.ipynb"  

I used a combination of HLS and HSV image for color thresholds by combining the 'S' channel and 'V' Channel respectively with appropriate threshold values to remove the noise such as shadows and combined the results. Gradient thresholds (Sobelx and Sobely) is also combined with the result of color threshold to improve the results resulting in a binary image as shown:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in Block 7 through file `./AdvanceLaneFindings.ipynb`. The `warper()` function takes as inputs an image (`image`), and Perspective Matrix `M` which is fetched from the function `getPerspectiveMatrix()` which takes source and destination points as input.  I chose the hardcode the source and destination points in the following manner:

```python

src = np.float32([
           [570.,470.],
           [722.,470.],
           [1100.,image_height],
           [220,image_height]
      ])
offset = 320
dst =  np.float32([
            [offset,0],
            [image_width-offset,0],
            [image_width-offset,image_height],
            [offset,image_height]
     ])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570, 470      | 320, 0        | 
| 722, 470      | 960, 0        |
| 1100, 720     | 960, 720      |
| 220, 720      | 320, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I then began my procedure for finding the lane lines in the image, with the code provided in the classroom as my starting point. The function `findCurveFit()` in the cell 16 of file `./AdvanceLaneFindings.ipynb` is used to detect lane lines in the image. It takes as input a binary transformed image and histogram to identify the lanes. 

The code begins by drawing a histogram of the image. Since by now we have a binary image, with reduced noise, the lanes should be the highest peaks in the histogram as shown below. 

![alt text][image8]

Using this we identify the mid-point of the image. Points upto the the mid-point are classifed as left and those greater than the mid-point are considered right. 

A sliding window search is then employed and the non-zero values obtained within this region was obtained. The points to the left were saved as good left indices and the points to the right of midpoint were saved as good right indices. 

Sliding Window:

![alt text][image9]

These points were then plotted in the arrays left_fitx and right_fitx. And then finally, I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 16th cell of file `AdvanceLaneFindings.ipynb`.

For the radius of curvature, a polynomial equation was fitted through the location points of the lanes obtain to form a polynomial in y. The radius was obtained using a formula R = ((1 + (dx/dy)^2)^(3/2))/(d2x/dy2). Code provided in the measuring curvature part of classroom material served as help and position was measured comparing the center of the image i.e. 640. I finally printed radius and position at top left corner in FONT_HERSHEY_SIMPLEX font.

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

The pipeline slightly fails for the harder challenges provided. I think that using other color channel images (RGB) and thresholding with the the appropriate values might solve the issue to some extent. I'm still experimenting and researching about the techniques to make the pipeline more general and robust. 

I still have to try the following few things:

1. Technique of averaging the lines and using lines detected with previous frames to get the output of current frame.
2. Making source and the destination points dynamic helping to make our pipeline more robust.
