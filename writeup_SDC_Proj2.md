## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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



## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

My program consisted of 2 main steps, myPipeline and image_process.
myPipeline calls the following function:

1- Camera_calib: using the chessboard pic to undistort it and calculate the undistortion parameters for other images
2- undistort: with undistortion parameters from Camera_calib function, the images would be converted to undistored images.
3- hls_conversion: to create the hsl image from RGB
4- find_sobels and combined_all: to detect the gradient form diiferent asprct (sobelx, sobely, magnitude, direction and color)
5- image_tansformation: to generate a top view of of bottom half of image
6- hist: to detect the start of lanes with help of histogram
7- find_lane_pixle: finding the lanes with help of starting points (hist) and detecting the lane lines with defining the rectngles
8- fit_polynomial: to fit a lane through the detected area in find_lane_pixle
9- curvature_not_center_calc and calc_veh_center: to calculate the curvature radius and deviation of vehicle from center

image_process calls the myPipeline and sends the its outputs to draw_on_image (I use this function from https://github.com/dkarunakaran/carnd-advanced-lane-lines-p4/blob/master/P4.ipynb) to present the images with detected lanes 



output images are save here:

   ./output_video
   ./output_images
    
    
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

to remove the distorion of images, I used a chessboard image. Function "Camera_calib" returns:
    - ret
    - mtx: camera matrix
    - dist: distortion coefficient
    - rvec: rotationl vector
    - tvec: translation vector
    
   ./output_images/Chessboard_img.png         "chessboard image"
       
in function "undistort", I've used mtx (camera matrix) and dist (distortion coefficient) for my images to undistort them.



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

here an example of one undistort image.

   ./output_images/undistort_img.png          "undistored image"
    

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

As we know, Sobel operator is at the heart of the Canny edge detection algorithm and takes the derivative of the image in different directions based on soebel function definition. Soebel function identify pixels where the gradient of an image falls within a specified threshold range. It could be in x direction, y direction, magnitude of both direction, or combination of all of them.

My function for that one are:
    - def find_sobels(): for soebel calculation
    - def combined_sobel(): for combination of all soebels
   
here the result of my soebel functions:

   ./output_images/soebel_x.png               "detecting gradient difference in x direction with soebel x"
   ./output_images/soebel_y.                  "detecting gradient difference in y direction with soebel y"
   ./output_images/soebel_magnitude.png       "detecting gradient difference using both soebel_x and soebel_y"
   ./output_images/soebel_direction.png       "detecting gradient difference using soebel for direction! 
   
for color gradient I've convert RGB to HLS and apply the soebel on this one:
   
   ./output_images/soebel_color_grad.png      "detecting gradient difference based on color"
   
finally combination of all soebels:
   ./output_images/combinel_all_soebel.png    "detecting gradient difference combining all above soebels"

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Function "image_tansformation" maps the points in a my image to image points with a top-view (bird's eye) perspective.
For this one I've used the following set points for source and destination image:

set points of source image:
   src_corner_ul = [590, 450]
   src_corner_ur = [700, 450]
   src_cornre_ll = [200, 670]
   src_corner_lr = [1100, 700]

set points of destination image:
   dst_corner_ul = [200, 0]
   dst_corner_ur = [1000, 0]
   dst_corner_ll = [200, 700]
   dst_corner_lr = [1000, 700]


here my transformation matrix:
[[ -5.70618627e-01  -1.48340464e+00   9.88807789e+02]
 [  2.06703341e-16  -1.89631666e+00   8.53342498e+02]
 [ -1.10080759e-05  -2.37878152e-03   1.00000000e+00]]


here one example of my function output done on one of project images:

   ./output_images/Top_view_half_img.png      "top view of image"


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

to detect the lanes start I've used histogram in "hist" function. here the result:

   ./output_images/histogram_lanes.png        "histogram to finde thelanes"

after thet I use the window method (function find_lane_pixle()with 9 windows for every lane and starting from detected lane by hist() ) and then calculating the polynomial coefficient for finding and drawing the polynomial which goes through all windows for every lane.

after that I used "fit_polynomial" function to fit this polynomial
Here is an example result for detected lanes:
   ./output_images/lane_detected.png          "detected lines as polynom"
   
   
#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the curvature radius (in meter) I've used the polynomial coefficients and with help of following mathematical formel, the radius could be calculated:

((1+(2*left_fit[0]*y_eval*ym_per_pix + left_fit[1])**2)**(3/2))/np.absolute(2*left_fit[0]) for left curve and the same (with right_fit for the right curve)

here the paramete ym_per_pix would be used to calculate the radius in meter instead of pixel.

to find the center of vehicle respect to center of image I find the center of image and center of lane and compare both to eachother, to identify if the vehicle is on the right side of image or left one.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

here one of the original images:
    ./output_images/original_img.png           "one of original images"

to show the images with detected anes I've find a nice function "draw_on_image()) and used it with a little changes (e.g. in parameter names).

With running "image_process()", I call "myPipeline()" as I discribed above to execute all calculation and then "draw_on_image() to show the images with detected lanes. Here are the results:

   ./output_images/Image_results.png"           "all 6 images with detected lines and information written on images"


### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Using my "image_process()" function to ally the lane detection function on sample video. Here is the result:

   ./output_video/project_video.mp4           "project video"
    
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The Project was pretty easy, because we've done all steps in excersises and I had to do all of them together, but was really funny.
These situations could cause problems:

   - I have problem with shadow!
   - if a car in front of ego vehicle change the lane (entering or leaving): here we need some algorithm to detect the moving object
   - if the lane color quality is not good, then detection would be more challenging
   - if the road quality is not good to identify the lane
   - if weather condition is not good (rainy, aquaplaning, snow, ...)
   - road construction site with differet lanes
   - changing the lane colors
   - crowded regions like city centers
   - maybe really sharp curves (montain roads)
   
To make the lane detection more robust, different algorithms are needed an a really good and robust sensor data fusion. and maybe another algorithm to plausibility check of sensor data.


