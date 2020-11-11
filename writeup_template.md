<!-- #region -->
## Writeup
## Artem Melnytskyi 2020
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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"





[after_transformed.png]: ./examples/documentation_src/after_transformed.png "after_transformed"
[before_transformed.png]: ./examples/documentation_src/before_transformed.png "before_transformed" 
[color_binary.png]:  ./examples/documentation_src/color_binary.png "color_binary"
[dark_binary.png]:  ./examples/documentation_src/dark_binary.png "dark_binary"
[dark_binary_wraped.png]:  ./examples/documentation_src/dark_binary_wraped.png "dark_binary_wraped"
[dark_binary_wraped_aproximate.png]:   ./examples/documentation_src/dark_binary_wraped_aproximate.png "dark_binary_wraped_aproximate"
[dark_original.png]:  ./examples/documentation_src/dark_original.png "dark_original"
[dark_result.png]:  ./examples/documentation_src/dark_result.png "dark_result"
[dir_binary.png]: ./examples/documentation_src/dir_binary.png "dir_binary"
[gradx.png]:  ./examples/documentation_src/gradx.png "gradx"
[hard_color.png]: ./examples/documentation_src/hard_color.png "hard_color"
[hard_directional.png]:  ./examples/documentation_src/hard_directional.png "hard_directional"
[hard_gradx.png]:  ./examples/documentation_src/hard_gradx.png "hard_gradx"
[hard_magnitude.png]:  ./examples/documentation_src/hard_magnitude.png "hard_magnitude"
[hard_original.png]:  ./examples/documentation_src/hard_original.png "hard_original"
[mag_binary.png]: ./examples/documentation_src/mag_binary.png "mag_binary"
[merged.png]:  ./examples/documentation_src/merged.png "merged"
[origina_.png]:  ./examples/documentation_src/origina_.png "original"
[original.png]:  ./examples/documentation_src/original.png "original"
[Points.png]:  ./examples/documentation_src/Points.png "Points"
[result.png]: ./examples/documentation_src/result.png "result"
[test_image.png]:  ./examples/documentation_src/test_image.png "test_image"
[undisorted.png]:  ./examples/documentation_src/undisorted.png "undisorted"
[combine_and_transpose.png]:  ./examples/documentation_src/combine_and_transpose.png "combine_and_transpose"
[windows_aproach.png]:  ./examples/documentation_src/windows_aproach.png "windows_aproach"
[choose_lane.png]:  ./examples/documentation_src/choose_lane.png "choose_lane"
[sobel.svg]:  ./examples/documentation_src/sobel.svg "sobel"
[magnitude_sobel.svg]:  ./examples/documentation_src/magnitude_sobel.svg "magnitude_sobel"
[direction_sobel.svg]:  ./examples/documentation_src/direction_sobel.svg "direction_sobel"

[bird_view_mathematic.svg]:  ./examples/documentation_src/bird_view_mathematic.svg "bird_view_mathematic"
[bird_view_scheme.png]:     ./examples/documentation_src/bird_view_scheme.png "bird_view_scheme"


[video1]: ./project_video.mp4 "Video"
[01_project_video.mp4]: ./output_videos/01_project_video.mp4 "Video 1 "
[02_project_video.mp4]: ./output_videos/02_project_video.mp4 "Video 2 "

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to the test image using the cv2.undistort() function and obtained this result:

For camera calibration I use standard aproach : 
1. ret, corners = cv2.findChessboardCorners(gray, (9,6),None)  - 9, 6 count of corners
2. cv2.drawChessboardCorners(img, (9,6), corners, ret) - for verify
3. ret, self.mtx, self.dist, self.rvecs, self.tvecs = cv2.calibrateCamera(self.objpoints, self.imgpoints,              gray.shape[::-1],None,None)  - self.mtx, self.dist matrix for undistord
4. undistorted = cv2.undistort(framme,self.mtx, self.dist) - undistord frame , means remove optical deformations

** Not all calibration images are ok, for example :
../camera_cal/calibration4.jpg
../camera_cal/calibration1.jpg
../camera_cal/calibration5.jpg
findChessboardCorners returne error for these samples.


All this logic you can find in class CameraCalibrator 


![alt text][image1]

### Pipeline (single images)
#### Main pipeline  

###### 1. Undisord 

original - ![original][original.png]
undisorted -  ![alt text][undisorted.png]
merged - ![alt text][merged.png] 

 
 (Usually it is part of smart camera in car)

###### 2. Use color transforms, gradients, etc., to create a thresholded binary image.

We will use Sobel operator (Sobel-Feldman). More ditailed you can find https://en.wikipedia.org/wiki/Sobel_operator
Here is examples of Sobel matrix for kernel size 3 :

gradx, grady  
![alt text][sobel.svg]


Magnitude of gradient  
![alt text][magnitude_sobel.svg]

Directional  
![alt text][direction_sobel.svg]

Implementation of each of this mathematicals operations you can find in class
**BinaryTransformer**

Main pipeline of BinaryTransformer is next :

**We will use sobel , but before it will be usefull to transform to HLS, and take S chanel. In terms of our issue changes in S chanel is most informative*.

       3.1 Lets convert to HLS, and take S channel for sobel 
       saturation = cv2.cvtColor(frame , cv2.COLOR_RGB2HLS)[:,:,2]   operations.
       
       3.2 Lets take gradx, and grady  
gradx  ![alt text][gradx.png] 
       
       3.3 Take sobel binary map by using of magnitude treshhold.
mag_binary  ![alt text][mag_binary.png] 
       
       3.4 Now we can take binary map from sobel transformation with direction treshholds (0.8, 1.3)
dir_binary  ![alt text][dir_binary.png] 
       
       3.5 And the last one thing, we can achive binary map frfom combination of colors that we expect on images
color_binary  ![alt text][color_binary.png] 


       3.6 combine all binary maps 
               combined[((gradx == 1)
                 | ((mag_binary == 1)
                    & (dir_binary == 1))) & (color_binary==1)] = 1
                    
Combine and transpose  (* to achieve it full please take a look point 3 )
 ![alt text][combine_and_transpose.png]

        
        
        Code you can find at class BinaryTransformer 

######  3. Perspective transformer

Good theory base here:   https://en.wikipedia.org/wiki/3D_projection#Perspective_projection'

and here  :

https://developer.ridgerun.com/wiki/index.php?title=Birds_Eye_View/Introduction/Research

 ![alt text][bird_view_scheme.png]:
 To obtain the output bird-s eye view image a transformation using the following projection matrix can be used. It maps the relationship between pixel (x,y) of bird's eye view image and pixel (u,v) from the input image.
 ![alt text][bird_view_mathematic.svg]:
 There are 3 assumptions when working without post transformation techniques:

1 .The camera is in a fixed position with respect to the road.

2. The road surface is planar.

3. The road surface is free of obstacles.


Points ![alt text][Points.png]
p1  = [ 210 , 680 ]  
p2  = [ 570 , 460 ]  
p3  = [ 710 , 460 ]
p4  = [ 1070, 680 ]
simple_camera_roi = np.array([p1,p2,p3,p4] , np.int32)

p1_transofrmed = [ 260 , 720  ]  
p2_transofrmed = [ 260 , 0    ]  
p3_transofrmed = [ 1020 , 0   ]  
p4_transofrmed = [ 1020 , 720 ]

For transformation you can look at the class : perspectiveTransformer

  Before  
  ![alt text][before_transformed.png]

  After  
  ![alt text][after_transformed.png]

        
###### 4 . Logic for detect initial bucket of points 
Slice window approach  ![alt text][windows_aproach.png] 
        
        Code you can find in functions :
        
        def find_lane_pixels(binary_warped)
        def fit_polynomial(binary_warped)
        
###### 5 . Detect points, build functions, detect lanes , calculate cuvatures and offset 
      Now by using of initial points we can build coridor from wich we will take points. 
      Using points what we receive from, we build new function for lanes, and next iterations. 
      Calculate curvature and offset. 
      
   ![alt text][test_image.png]     
   ![alt text][result.png] 
     
#### additional steps for more stability

We can faced with situation , when our extraction methods cannot achive needed result in some situation, here is :

   ![alt text][dark_original.png]
   ![alt text][dark_binary.png]
   ![alt text][dark_binary_wraped.png]
   

###### 1. One lane it is "enougth"

       In case when we have pretty well visible right or left lane ( it is often case ), we can calculates points for this lane and just move it to another part , by using of offset betwean lanes ( as we know it is constant ).

   ![alt text][choose_lane.png]: 

###### 2.Aproximate   
        Very often we have some gaps in road markings, but our brain can aproximate previos marks to not marked area.  
        so we can help, we can store some of images ( in range of memory window), and build polynom by using combination of images.
 Before aproximate 
![alt text][dark_binary_wraped.png]
 After aproximate 
![alt text][dark_binary_wraped_aproximate.png]
![alt text][dark_result.png]
        
<!-- #endregion -->

<!-- #region -->
### Discussion

1. This method very sensitive for brightness conditions
2. Curvature of road also has limitation in sons of usage this method
3. There are big couple of hyper parameters for tuning for some of situation, and this is not stable


If we want system that can be able for recognize from video sensor only with quality of human, it is meen that we need to have algorythm that work in same way. I mean neural networks. 
NN can reduse dependancy for :

1. Color of road
2. Form of road
3. Marking of road ( for some level)


### Results 

<!-- #endregion -->
![Video result 1 ][01_project_video.mp4] 
![Video result 1 ][02_project_video.mp4] 

or https://youtu.be/eCO6aTeSsxM
```python

```
