# Udacity-CarND-Advanced-Lane-Lines
A computer vision project that detects lane lines on a video of a driving car for the Udacity Self-Driving Car Engineer Nanodegree.

The code goes through the following steps:
- Step 0: Import Libraries
- Step 1: Calibrate Camera
- Step 2: Undistort Image
- Step 3: Apply Colour and Gradient Thresholding
- Step 4: Perspective Transform
- Step 5: Find Lane Lines
- Step 6: Measure Curvative
- Step 7: Draw Lane Area on Image
- Step 8: Sanity Check
- Step 9: Look-Ahead Filter
- Step 10: Image Pipeline
- Step 11: Video Pipeline

### Step 1: Calibrate Camera
Feed into this function a number of chessboard images (20 for accuracy) taken by a camera, and it returns a camera matrix (mtx) and distortion coefficients (dist), which can be used to undistort other images taken by this camera.

**How**: The corners on a chessboard image taken by a camera can be detected by the cv2.findChessboardCorners() function, and this is compared with the corners of a perfect 2D chessboard image. The camera matrix (mtx) and distortion coefficients (dist) can be computed from this comparison using the cv2.calibrateCamera() function.

The image below shows the corners on a chessboard image detected by the cv2.findChessboardCorners() function, and drawn by the cv2.drawChessboardCorners() function.

**Insert Image**

### Step 2: Undistort Image
Takes in a camera image, camera matrix and distortion coefficients (from Step 1), and returns an undistorted image using the cv2.undistort() function.

**Why**: Camera lenses cause images taken to be slightly distorted, affecting the shape and size of the objects in the images. This is a problem for lane detection, as the distortion causes the lane curvature measured from the image and eventually steering angle to be wrong.

It can be observed below that there are slight differences between the original and undistorted image.

**Insert Image**

### Step 3: Apply Colour and Gradient Thresholding
Takes in an undistorted image and returns a binary image with a combination of colour and gradient thresholding applied.

A. Gradient Thresholding:
- Sobel x using the abs_sobel_thresh(orient='x') function, which emphasizes vertical edges

**Insert Image**

- Sobel y using the abs_sobel_thresh(orient='y') function, which emphasizes horizontal edges

**Insert Image**

- Magnitude of gradient using the mag_thresh() function, which is the magnitude of Sobel x and Sobel y. This is equivalent to using the Canny edge detection filter, and is good for picking up all edges.

**Insert Image**

- Direction of gradient using the dir_thresh() function, which picks out edges of a particular orientation (in this case, orientation of lane lines)

**Insert Image**

B. Colour Thresholding:
- In the HLS space (hue, lightness, and saturation), the Saturation (S) channel picks up the lane lines well under very different color and contrast conditions, hence the S colour threshold is chosen

**Insert Image**

C. Combining Thresholds:
- In the combined_col_grad_thresh() function, the different thresholds described in (A) and (B) are combined in the following way: *{[(Sobel x and Sobel y) or (mag_thresh and dir_thresh)] or s_channel}*

**Insert Image**

### Step 4: Perspective Transform
Takes in an undistorted, thresholded binary image, and returns a warped top-down view of the image and perspective transform matrix (M) which will be used in Step 7.

**How**: 4 source (src) points from the binary image are chosen, resembling a trapezium around the lane lines boundary. 4 destination (dst) points on the resulting top-down image are chosen, to map the trapezium shape into a rectangle shape using the cv2.getPerspectiveTransform() function.

**Insert Image**

### Step 5: Find Lane Lines
Here, two functions are defined.
1) The sliding_windows() function takes in a binary top-down image, and returns the coefficients of the polynomials for the left and right lanes. This is the full algorithm used to find the lane lines from scratch, using histogram to perform a blind search.

**Insert Image**

2) The search_from_prior() function takes in a binary top-down image, previous coefficients of the polynomials for the left and right lanes, and returns updated coefficients. This is a shorter algorithm that does a targeted search in the margin around the previous line position.

**Insert Image**

The Look-Ahead Filter in Step 9 will decide which of these functions should be used.

### Step 6: Measure Curvature
Takes in a binary top-down image, and the coefficients of the polynomials for the left and right lanes. Returns the radius of curvature for the left and right lanes, and the vehicle offset from the lane centre, in metres.

### Step 7: Draw Lane Area on Image
Takes in the following:
- Undistorted image, to draw on
- Binary top-down image and coefficients of polynomials of left and right lane lines
- Perspective transform matrix (M), to transform the top-down image back to original image perspective
- Left and right lane radii and vehicle offset dimensions
Returns an image with the area between both lane lines highlighted, and left/right lane radii and vehicle offset displayed.

**Insert Image**

### Step 8: Sanity Check
Takes in the coefficients of polynomials describing both lane lines, and checks for the following:
- Checking that they have similar curvature
- Checking that they are separated by approximately the right distance horizontally
- Checking that they are roughly parallel
If either condition is not met, a boolean False is returned.

### Step 9: Look-Ahead Filter
A function that takes in an image, and chooses either the sliding_windows() or search_from_prior()functions to find the lane lines.
A class Lane() is instantiated to receive and record the characteristics of each video frame. The previous function sanity_check() is applied to check the characteristics for bad or difficult video frames. If it returns "False" for:
- A few frames: retain the previous positions from the frame prior, and step to the next frame to search again
- Several frames in a row: start from scratch using sliding_windows()

### Steps 10 & 11: Image & Video Pipeline
Apply all of the above to images or a video.

### Limitations of Pipeline
Here are some potential limitations of the pipeline:
1) Poor lighting conditions such as night time, fog or rain, which cause the lane lines to be less visible and distinct.
2) Some roads are not well maintained and the lane markings may be faint.
3) Some roads are not well maintained and the road surface is not black, but light grey. This does not look very different from the white or yellow lane markings.
4) On some roads, there may be sudden changes in lighting, such as when passing into a tunnel, or alternating trees. The camera needs time to adjust to changes in lighting, and during the transition period, the lane lines cannot be seen.

