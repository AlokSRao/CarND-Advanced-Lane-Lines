#Advanced Lane Detection

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

[image1]: ./examples/undistortvsdistort.png "Undistorted vs Distorted"
[image2]: ./examples/Filtered.png "Filtered Image"
[image3]: ./examples/perspective.png "Perspective Transformation"
[image4]: ./examples/lane_detect.png "Lane Detection"
[image5]: ./examples/lane_detect_true.png "Fit Visual"



### Writeup / README

### Camera Calibration
Normally, images taken from a camera introduce a warping to the image, due to nature of the lens. Optical distortion is caused by the optical design of lenses (and is therefore often called “lens
distortion”), perspective distortion is caused by the position of the camera relative to the subject or by
the position of the subject within the image frame. This distortion can be undone, by comparing it to a
reference image with straight lines, like a chess board. We use cv2.findchessboardcorners to obtain
corner points in the distorted image, and since we know where the corner points should be for an
undistorted chess board, we can get the transformation required to un warp the image.


![alt text][image1]

### Pipeline (single images)

#### 1. Calibrate the camera.

Using the above method, undistort the image obtained from the camera

#### 2. Apply required filters

Our objective is to filter the unwarped image, such that only the lanes are visible. To achieve
this, we can use a number of techniques. For example, we can take one channel of the image,
such as the red color channel, and experiment with various thresholds (i.e. compare the pixel
value with some numbers to filter out pixels not in range of those numbers). If we set the
thresholds to 100-150, then only those pixels in the R channel whose value is within 100-150 will
remain. We can also experiment by converting the image to other types, such as HLS image,
and use thresholding. Finally, we can use sobel filter in the x direction to try to obtain all the
vertical lines in the video. Finally, when we are satisfied with the individual images, we can
combine them to form a composite filtered image. It is acceptable if there is some noise, as we
will deal with it in future steps.
![alt text][image2]

#### 3. Perspective Transform.
We obtain a top down image using perspective transform. To do this, we obtain 4 points in the
original image, and estimate where they will land in the top down image. We can use known
points to do this. Using these 4 points, we obtain a matrix that can be used to transform the
whole image to top down view. This view also removes everything other than the lane from
view.

```python
 #get 4 reference points
    source = np.float32([[585,455],[705,455],[1130,720],[190,720]])
    
    #4 points from the image
    offset = 200 # offset for dst points
    dst = np.float32([
    [offset, 0],
    [imageSize[0]-offset, 0],
    [imageSize[0]-offset, imageSize[1]], 
    [offset, imageSize[1]]
])
```


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]

#### 4. .Lane finding and curve fitting

Once we have a top down image, we take a histogram of the image along x axis. The idea is that
the two lanes will lie on either side of the mid point of the histogram. The block on the left of
the mid point is highly likely to be the left lane, and same for the right lane. But we do not do
this for the entire image. Instead we divide the image into horizontal windows, and perform this
histogram technique on each horizontal window. Thus, each window has a left lane candidate,
and right lane candidate. Finally, we join all these candidate points using a polynomial. 
![alt text][image4]

#### 5. .Final processing

The polynomial obtained is evaluated and a list of Y coordinates are calculated. Then these
points are remapped on the original image using inverse of the perspective transform matrix.

![alt text][image5]
#### 6. Final output

Here's a [link to my video result](./Advanced_Lane_Detection.mp4)

---
