## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


The Project
----

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Software Requirements
-------
     The project is done using python 3.6.2.
     OpenCV version 3.1.0 is used.
Python version less than 3.6.0 will not work properly because in my code I used f-string which is a new feature.

Folder Specification
-------
    1. The images for camera calibration are stored in the folder called camera_cal.  
    2. The images in test_images are for testing pipeline on single frames.
    3. The outputs after finding the chase board corner are in camera_cal_output.
    4. The binary outputs after color and gradient transformation of test_images 
       are in test_images_transformed folder.
    5. The binary outputs after bird's eye view perspective transform of test_images 
       are in test_images_wraped folder.
    6. The test_output folder contains all the test images after applying final pipeline function.
    7. Resources folder contains some images for purpose of writing README.md file.
    8. The output video after applying the pipeline to project_video.mp4 is output.mp4.


**Note**: All codes are self explanatory. Comments and function documentation has been given where needed.
          
#### Finding chess board corner
---

**The code for this step is contained in the fourth code cell of the IPython notebook advanced_lane_finding.ipynb.**

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time `cv2.findChessboardCorners()` detects all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. 

Then draw the chess board corners by `cv2.drawChessboardCorners()`. Here is an example of chessboard corner drawn .

**Chess Board Corners**

![Image](./resources/chsbrdcrnr.png)

#### Calibrating camera and calculation of camera matrix and distortion coefficients
`undistort_image()` function is used to undistort a image, the previous output `objpoints` and `imgpoints` have been used 
to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the input image using the `cv2.undistort()` function .

**Undistorted Chess board image**

![Undistorted Chess Board](./resources/undist_chsbrd.png)


### Pipeline (single images)
---

#### Example of distortion corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

**Undistorted test image**
![Undistorted Test](./resources/undist_testimg.png)

#### Applying gradient and color thresolding
I used a combination of color and gradient thresholds to generate a binary image .
       
        1. compute the sobel magnitude  then take a thresolding range b/w 20 and 255.
        2. select_yellow() and select_white() aretwo functions to extract the yellow
           and white lane lines the combine it by bitwise or.
        3. Then combine it with sobel magnitude calculated.
       

**Transformed image after applying gradient and color thresolding**
![Thresolded Image](./resources/trnsfrm.png)

#### Perspective transformation and getting a birds eye view perspective

I used a `wrap()` function to get a birds eye view using perspective transform. It takes an image , source points and destination points and returns the wraped image transformation matrix and it's inverse .

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 710, 460      | 990,0         |  
| 1090, 720     | 990, 720      |
| 200, 720      | 300, 720      |
| 580, 460      | 300, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Perspective Transform](./resources/birdseyeview.png)

#### Finding lane line pixel and fitting polynomial

     1. Divide the thresolded binary transformed image into n=9 horizontal strips of equal height.
     2. Compute the histogram of each strip using np.sum().
     3. Identify two peaks where histogram computed are maximums.
     
**Birds eye view of a test transformed image and histogram drawn**

![Image 1st](./resources/hist.png)
![Image 2nd](./resources/hist1.png)

    
     4. Get the pixels in that horizontal strip that have x coordinates close to the two 
        peaks of x coordinates.


**Rectangles are drawn where lane line pixels are detected**

![Image 3rd](./resources/poslane.png)
 
 
     5. Fit a second order polynomial to each lane line using np.polyfit() function.


**Polynomial fitted to birds-eye-view image**

![Image 4th](./resources/poly1.png)
![Image 5th](./resources/ploy2.png)


#### Calculation of radius of curvature.

    left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5)\
                     / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5)\
                     / np.absolute(2*right_fit_cr[0])
    
    curvature = (left_curverad + right_curverad)/2

#### Example on a test image.

The final `pipeline_func()` contains all the necessary steps for finding and plotting the lane lines. Here is an example of my result on a test image:

![Output Image](./test_output/test2.jpg)

---

### Pipeline (video)
I applied my `image_pipeline()` function to the project_video. It detect's lane lines reasonably well but under a 
curved shadowed path it once showing some deviation.
**Here's the link of my [Output Video](./output.mp4)**

---

### Discussion
    This pipeline will not actually work rather than this project video. The source and destination 
    points have been chosen manually for birds eye view perspective transformation. Which indeed 
    is the most important step for finding lane line pixels.
    
    The color and gradient thresolding and identifying lane lines is a computing intensive work 
    and a maximum speed of 1.5 iteration/s achieved in these case will not work in realtime.
    
    But this pipeline detects lane lines moderately well on different colored , shaded highway path.
