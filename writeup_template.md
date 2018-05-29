
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

[image1]: ./output_images/chessboard_distorted.png
[image2]: ./output_images/chessboard_undistorted.png
[image3]: ./output_images/unmasked.png
[image4]: ./output_images/masked.png
[image5]: ./output_images/before_perspective.png
[image6]: ./output_images/after_perspective.png
[image7]: ./output_images/sliding0.png
[image8]: ./output_images/sliding2.png
[image9]: ./output_images/sliding3.png
[image10]: ./output_images/pipeline_out.png

[image11]: ./output_images/no_region.png
[image12]: ./output_images/region.png



### Pipeline Description

#### 1. Camera Calibration

Image distortion occurs when a camera looks at 3D objects in the real world and transforms them into a 2D image; this transformation isn’t perfect. Distortion can:

1. Change the apparent size of an object in an image
2. Change the apparent shape of an object in an image
3. Cause an object's appearance to change depending on where it is in the field of view
4. Make objects appear closer or farther away than they actually are

So the first step in our pipeline is to calibrate the image by removing this distortion.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

Distorted
![alt text][image1]

Undistored
![alt text][image2]

#### 2. Region of Interest

A region of interest is applied to the image to remove as much extraneous information outside the lane lines as possible. Below are the results of the region of interest mask.

Original
![alt text][image11]

Region of Interest
![alt text][image12]

#### 3. Color Thresholding.

I experimented with a number of different color and gradient thresholding techniques. Ultimately I used 3 masks in my pipeline:

1. An RGB white mask
2. An RGB yellow mask
3. An HSV white mask

I found the combination of these three masks was effective at finding lane lines. Below are examples of an input image and a masked image:

Unmasked
![alt text][image3]

Masked
![alt text][image4]

#### 3. Perspective Transform

The code for my perspective transform includes a function called `warp()`.  The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[300, 650], # bottom left
     [1000, 650], # bottom right
     [600, 450], # top left
     [680, 450]]) # top right

dst = np.float32(
    [[375, 720],
     [1000, 720],
     [375, 0],
     [1000, 0]])

```

Before Perspective transform
![alt text][image5]

After Perspective Transform
![alt text][image6]

#### 4. Sliding Window and Polynomial fit

Next I used a sliding window search to find the lane lines and fit a polynomial. This code is provided by Udacity. The idea is to apply a histogram along the width of the image. The highest peak on the left side of the image should correspond to the left lane while the highest peak on the right side of the image should correspond to the right lane. Using these peak positions as a starting points we step through the height of the image adjusting the window on each side of the image as necessary. The images below show the starting image, the perspective transform, and the sliding window search performed on the perspective transform image.  

Original Image
![alt text][image7]

Threshold and Perspective Transform
![alt text][image8]

Sliding Window Search
![alt text][image9]

#### 5. Radius of Curvature and Distance from Center

For curvature measurements I used the polynomial fit the left lane line (as the right lane should have the same or very similar curvature).

Given a polynomial of the form f(y) = Ay^2 + By + C the curvature can be given as

R
curve = ( (1+(2Ay+B)^2)^3/2 ) / 2A

The vehicle's position was determined using the center of the image (assumed the camera is mounted on the center of the car). The center of the lane was calculated using the left and right lane line pixels from the bottom of the image and averaging them. Finally the distance from the center of the lane was found by subtracting lane center from the vehicle's position and converting to meters (assuming 3.7/700 meters/pixel in the x direction).

#### 6. Example Output

Image with Lane Lines
![alt text][image10]

---

### Pipeline (video)


[![Video 2](https://i.imgur.com/dm6d4Ab.png)](https://youtu.be/CmGLsZQRthg "Self Driving Car Project - Advanced Lane Lines - Click to Watch!")
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline can struggle with changes in road color and changes in lighting. Due to time constraints the current implementation performs a new sliding window search on each frame. This could be improved by performing a sliding window search on the first frame, saving the data, and doing a more targeted search on subsequent frames.
