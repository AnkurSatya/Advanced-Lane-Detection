

## Project Writeup 

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

[image1]: ./output_images/chessboard_distortion.png "Original Chessboard"
[image2]: ./output_images/road_distortion.png "Original Road Lanes"
[image3]: ./output_images/gradient_color_thresh.png "Combined Thresholds"
[image4]: ./output_images/gradient_color_thresh_code.png "Code for Gradient and Color Thresholding" 
[image5]: ./output_images/sobel_thresholds.png
[image6]: ./output_images/color_space_code.png
[image7]: ./output_images/white_lines_code.png
[image8]: ./output_images/perspective_transform.png
[image9]: ./output_images/lane_marking.png
[image10]: ./output_images/final.png
[video1]: ./output_videos/video1.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./project.ipynb".

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![image2]

This image is used for all further demonstrations purposes.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. The code for the combination is provided below and individual thresholding is explained further after.

![alt text][image3]

**Code:**

![image4]

Now, explaining individual thresholding-

![image5]

* **Gradient Thresholding in Horizontal direction (X) and Vertical direction (Y)**

I used Sobel operator for evaluating the gradients. This calcultes the gradient of every pixel by convolving a kernel on the image. Further edges can be detected by setting a threshold on the gradients.
As lane lines are more inclined towards vertical than horizontal, thresholding gradients in horizontal direction (X) can detect lane lines.
But in combined code we used the union of both the gradients as it will decrease the dependency of detecting the lanes only on the basis of their inclination.


* **Gradient Magnitude and Direction Thresholding**
Magnitude of total gradient for any pixel can be calculated by taking the square of the sum square of the X and Y gradients. Direction can be known by taking the inverse tangent of (gradientx/gradienty). They remove the unnecessary lines or noise from the image.


* **Color Thresholding**
I used HLS color channel for the purpose of color thresholding. Ligthness and Saturation channel proved out to be quite useful.
* To detect yellow lines:
Yellow lines can be easily detected by using the saturation channel and the code used to detect it is given below:

![image6]

* To detect white lines:
White lines are hard to detect using saturation chanel. But lightness channel is helpful in identifying white lines.
The code is given below:

![image7]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for perspective transform is in *project.pynb* from cell 12 -13. I chose to hardcode the source and destination rectangles for the transformation. The values are given below:

```python
src=np.float32([[195,720],
                [590,460],
                [700,460],
                [1120,720]])
dst=np.float32([[350,720],
                [410,0],
                [970,0],
                [1000,720]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 195, 720      | 350, 720        | 
| 590, 460      | 410, 0      |
| 700, 460      | 970, 0     |
| 1120, 720     | 1000, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. This was sufficient to verify the warping process.

![image8]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the histogram method to find the peaks in the image to find the starting position for the sliding windows to identify the lane. As the noise was very much cleared by the morphological operations, there was no difficulty in identifying the starting positions. 

After identifying the left and right lane pixels I used cv2.polyfit() to fit the polynomial curve. With my initial threshold paramters, there was a problem in detecting the frames with shadow in them. I changed the minimum threshold for Lightness channel to a lower value which helped in improving the detection for the shadow frames. I used the past frame data if there was no detection in a frame altogether.

![alt text][image9]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for finding radius of curvature and position of vehicle w.r.t center is in cell 16 under the title "Creating a Line Class to store data" in the project.ipynb file.
The process to position of the vehicle includes fitting the left and right lane polynomial on the warped image. Then, I calculated the mean of left and right lane to find the lane's center coordinates.
Assuming the vehicle to be in the middle of the frame, I calculated the difference between the lane center and the vehcile's position(as found out by frame center).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this can be found in the cell 14 in the project.pynb file. Below is an example of the desired output.

![alt text][image10]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Please find the video in the zip folder submitted for the project submission with the name **video1.mp4**

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I am satisified with the output of my pipeline on the given test video. But, in the given video, the brightness does not change much except for a few spots. I found that my pipeline needs to be more robust when it encounters low brightness or shadows. A way to encounter this is predict the slope of the lanes in the next frame by using a small neural network and training it with the error(difference between the predicted slope - actual slope). This migth work well in case of shadows and low brightness.



