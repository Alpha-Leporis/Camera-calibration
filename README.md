# one of the key problems in Computer vision is recovering the 3D structure of a scene from its images.

We have a scene that is defined in the world coordinate frame when we reconstructed the scene we would like to know where each point lies in the world coordinate frame say as in millimeters. but what we have is the disposal are images of the scene where points are measured in terms of pixels.
To go from images to full matrix reconstruction, we need two things - 
1. The first is the position of the camera with respect to the world coordinate frame these are referred to as the external parameters of the camera.
2. And then we wanna know how the camera maps via perspective projection point in the world onto its image plane this is referred to as internal parameters of the camera (such as its focal length).

Determining the external and internal parameters of the camera is referred to as camera calibration.

So we are gonna develop a method for estimating the external and internal parameters of the camera.

Before we can do this we first need a model of the camera and this is what we referred to as a camera model or a forward imaging model which takes from 3D to 2D.

# Linear Camera model
Which takes a point in 3D to its projection in pixels in the image. the linear model is a single matrix called a projection matrix. so now projection matrix is in place we develop a camera calibration method.


# camera calibration
We take a single picture of an object of a known geometry this is what we need to fully calibrate the camera. in other words, to determine the projection matrix.

# Extractiing Intrinsic and Extrinsic Matrices
Once we have determined the projection matrix we can tear it apart to figure out both the internal parameters and the external parameters of the camera. These are called Intrinsic and Extrinsic Matrices.

At this point, the camera is fully calibrated.

# Different types of camera calibration methods

1. **Calibration pattern:** Sometimes known as a calibration grid or a calibration target is a repeating pattern of known size and spacing. the best way to perform calibration is to capture several images of an object or pattern of known dimensions from different viewpoints. for example 
	**checkerboard pattern:** camera calibration using checkerboard pattern to get the Intrinsic Matrices depends on the camera calibration accuracy.
	We can also use **circular patterns** of known dimensions.

2. **Geometric clues:** Sometimes we have other geometric clues in the scene like straight lines and vanishing points which can be used for calibration.

3. **Deep Learning based:** When we have very little control over the imaging setup (for example we have a single image of the scene), it may still be possible to obtain calibration information of the camera using a Deep Learning-based method. 


# Calibration using OpenCV
The projection of a 3D point onto the image plane, we first need to transform the point from the world coordinate system to the camera coordinate system using the extrinsic parameters (Rotation and Translation). 

![alt text](https://github.com/itsmeaby/HackLab-Assignment/blob/main/img/intrinsic%20parameters.png)

Where, P is a 3×4 Projection matrix consisting of two parts — the intrinsic matrix (K) that contains the intrinsic parameters and the extrinsic matrix ( [R | t] ) that is the combination of 3×3 rotation matrix R and a 3×1 translation t vector.

![alt text](https://github.com/itsmeaby/HackLab-Assignment/blob/main/img/Projection%20matrix.png)

The intrinsic matrix K is upper triangular 

![alt text](https://github.com/itsmeaby/HackLab-Assignment/blob/main/img/intrinsic%20matrix%20K%20.png)

Where,

f_x, f_y are the x and y focal lengths ( yes, they are usually the same ).

c_x, c_y are the x and y coordinates of the optical center in the image plane. Using the center of the image is usually a good enough approximation.

γ is the skew between the axes. It is usually 0. 

# calibration process
The calibration process is to find the 3×3 matrix K, the 3×3 rotation matrix R, and the 3×1 translation vector t using a set of known 3D points (X_w, Y_w, Z_w) and their corresponding image coordinates (u, v). When we get the values of intrinsic and extrinsic parameters the camera is said to be calibrated. 

A camera calibration algorithm has the following inputs and outputs
1. Inputs: A collection of images with points whose 2D image coordinates and 3D world coordinates are known.
2. Outputs: The 3×3 camera intrinsic matrix, the rotation, and translation of each image. 

**In OpenCV** The camera intrinsic matrix does not have the skew parameter. 
So the matrix is of the form ![alt text](https://github.com/itsmeaby/HackLab-Assignment/blob/main/img/skew%20parameter.png)


![alt text](https://github.com/itsmeaby/HackLab-Assignment/blob/main/img/camera-calibration-flowchart.png)


# Camera Calibration can be done in a step-by-step approach:
**Step 1**: First define **real-world coordinates** of 3D points using the known size of the checkerboard pattern.

**World Coordinate System**: World coordinates are fixed by the checkerboard pattern. 3D points are the corners of the squares in the checkerboard. Any corner of the checkerboard board can be chosen to the origin of the world coordinate system. All points on the checkerboard are on the XY plane ( i.e.  = 0 ).
In the process of calibration, we calculate the camera parameters by a set of known 3D points and their corresponding pixel location in the image.
For the 3D points, we photograph a checkerboard pattern with known dimensions at many different orientations. The world coordinate is attached to the checkerboard and since all the corner points lie on a plane, we can arbitrarily choose for every point to be 0. Since points are equally spaced in the checkerboard, the coordinates of each 3D point are easily defined by taking one point as reference (0, 0) and defining remaining with respect to that reference point.

**Step 2**: Different viewpoints of the check-board image are captured.
We keep the checkerboard static and take multiple images of the checkerboard by moving the camera.
Alternatively, we can also keep the camera constant and photograph the checkerboard pattern at different orientations. The two situations are similar mathematically.

**Step 3**: Find 2D coordinates of checkerboard: We have multiple images of the checkerboard. We also know the 3D location of points on the checkerboard in world coordinates. The last thing we need is the 2D pixel locations of these checkerboard corners in the images.  
**3.1 Find checkerboard corners:** OpenCV provides a built-in function called **findChessboardCorners()** that looks for a checkerboard and returns the coordinates of the corners.  
**3.2 Refine checkerboard corners:** OpenCV’s function **cornerSubPix()** takes in the original image, and the location of corners, and looks for the best corner location inside a small neighborhood of the original location. The algorithm is iterative in nature and therefore we need to specify the termination criteria ( e.g. number of iterations and/or the accuracy )

**Step 4**: **findChessboardCorners()** is a method in OpenCV and used to find pixel coordinates (u, v) for each 3D point in different images.


