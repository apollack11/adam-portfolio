---
layout: inner
title: 'Localizing the HTC Vive Tracker'
date: 2017-08-22 22:30:00
categories: C ros
tags: localization
featured_image: '/assets/vive_tracker_loc/testrun1_rviz.gif'
lead_text: 'Final Project for MSR Program'
---

Project to develop a software package to derive a 6 DoF pose of the HTC Vive Tracker.  

Final GitHub Repo: [vive_tracker_loc](https://github.com/apollack11/vive_tracker_loc)  
ROS Package Version: [vive_tracker_loc 'cmake_convert' branch](https://github.com/apollack11/vive_tracker_loc/tree/cmake_convert)

### Summary    
The goal of this project was to develop a software package to derive the pose of the HTC Vive Tracker. This software package needed to be capable of running on a Raspberry Pi 2 and also only require the Vive Tracker and a Lighthouse to work. This project was proposed by Professor Michael Rubenstein and is intended to be an add-on to his course focused on the creation of a quadrotor from start to finish. He wants to be able to use the pose estimate derived from the Vive Tracker to perform control of the quadrotor.  

### Background  
The HTC Vive system consists of a main headset unit, two lighthouses, and two controllers. The lighthouses are the core element of this system because they track the other components. Each lighthouse sends out sweeps and pulses of infrared light. The vive devices (e.g. the headset, controllers, and tracker) are all equipped with IR sensors which detect when they received each sweep of light, the duration, and the timestamp associated with the sweep and from that information can derive the horizontal and vertical angles of each sensor relative to the lighthouse (more info [here](https://www.youtube.com/watch?v=oqPaaMR4kY4)). HTC also released a set of sensors in a smaller package known as the [Vive Tracker](https://www.vive.com/us/vive-tracker/). The Vive Tracker is what I used for my project.  

### Approach  
#### Initial Efforts
My initial approach involved using SteamVR and OpenVR to get the pose of the Vive Tracker. However, after Professor Rubenstein made it clear he needed this to work on only a Raspberry Pi, I shifted my efforts toward a more custom version.  
#### Libsurvive  
After conducting some research to try and find a software package which could interface with the HTC Vive, I found [libsurvive](https://github.com/cnlohr/libsurvive). Libsurvive is a project which aims to reverse engineer the protocol behind the HTC Vive system in order to create a custom VR system built around a lightweight C library. I talked to some of the developers on Discord about the state of the project and they helped me get the software set up. They told me they were working on localization but had not gotten anything working well yet. However, though they had not gotten localization working, they had a way to extract the angle of each sensor on any of the Vive devices to the lighthouses, which is all I needed to determine the pose.  
#### Perspective n-Point Algorithm  
Now that I had a working piece of software to determine the angle from the lighthouse to each sensor on the tracker, I needed a way to determine the pose from this data. I conducted some research and found out that HTC uses an algorithm called Perspective n-Point (PnP) to solve this problem. I also found out that OpenCV has an implementation of this algorithm which is used for extrinsic camera calibration. With the help of one of the directors of the program, Jarvis Schultz, I was able to get the OpenCV version working with the data I had. The main challenge during this portion of the project was getting C++ code needed for OpenCV to work with the existing C code used in Libsurvive. After rewriting portions of the Makefile and modifying some of the struct definitions, I was able to pass in a set of angles to the OpenCV code ([opencv_pose_calc.cpp](https://github.com/apollack11/vive_tracker_loc/blob/master/src/opencv_pose_calc.cpp)) and return back an SE3 tranformation matrix from the lighthouse to the tracker.  
#### Executable  
After determining the pose using the PnP algorithm, I went back and rewrote the executable to return the current position and quaternion of the tracker relative to the lighthouse. The final executable is on the master branch of the repository and is called [runposer.c](https://github.com/apollack11/vive_tracker_loc/blob/master/src/runposer.c).  

### Extensions  
#### ROS Package  
Once I had a working version of my software package for pose estimation using only a lighthouse, a tracker, and a Raspberry Pi, I wanted to get a version of the software working with the Robot Operating System (ROS). This involved only a few modifications including adding the ability to use cmake for compilation, putting the package into a ROS workspace, and writing a node to publish pose data from the tracker. This edition of the software currently resides on the cmake_convert branch of the GitHub repo. Eventually, I would like to create a standalone ROS package that could be distributed for others to use.  

### Results  
To test the accuracy of the pose estimation, I mounted the Vive Tracker to the end effector of the Sawyer robot we have in the MSR lab. Using the ROS package, I was able to perform a brief calibration to determine the location of the lighthouse relative to the base of Sawyer and then visualize the pose estimation of the tracker in Rviz. I then recorded four different movements of Sawyer's arm and played them back to create four test runs. The video below shows both the Rviz visualization and the actual movement of Sawyer and the tracker. In the Rviz portion of the video, the two frames at the end of Sawyer's arm represent the position and orientation of the tracker and Sawyer's gripper (with the tracker offset slightly behind and above the gripper). The frame at the top of the Rviz portion of the video represents the lighthouse.  

<div style="text-align: center;">
  <iframe src="https://player.vimeo.com/video/231126775" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen style="width: 80%; height: 400px; max-width: 650px; padding: 10px"></iframe>
</div>

<div style="text-align: center;">
  <img src="/assets/vive_tracker_loc/sawyer_full.JPG" alt="Sawyer Setup" style="width: 80%; max-width: 650px; padding: 10px"/>
  <p>Figure 1: Sawyer setup with the Vive Tracker attached to its end effector</p>
</div>

<div style="text-align: center;">
  <img src="/assets/vive_tracker_loc/tracker_close.JPG" alt="Cancerous Images" style="width: 80%; max-width: 650px; padding: 10px"/>
  <p>Figure 2: Close up image of the Vive Tracker mounted to Sawyer's end effector</p>
</div>

<div style="text-align: center;">
  <img src="/assets/vive_tracker_loc/lighthouse.JPG" alt="Lighthouse" style="width: 80%; max-width: 650px; padding: 10px"/>
  <p>Figure 3: Lighthouse setup, above Sawyer</p>
</div>

### Future Work  
Below are a number of extensions to this project which I would like to implement in the future:
1. Eliminate the need for OpenCV as a dependency. This would require a different implementation of the PnP algorithm.  
2. Contribute this pose estimation work back to libsurvive. I would likke to help them improve their software package if I can eliminate the need for OpenCV and make my code entirely C.  
3. Get pose estimation working with more than one lighthouse.  
4. Get pose estimation working with more than one tracker.  
5. Implement position control on a quadrotor using the tracker for position estimation.  
6. Build a custom tracker with a smaller footprint for use on vehicles such as a quadrotor.  


<!-- ### Related Work  
Once I have established localization of the quadcopter, I would like to be able to control its position in space and track trajectories based on its position. In preparation for this concept, I focused the efforts of my project in ME454: Optimal Control of Non-Linear Systems on controlling a 2D quadcopter. Below is a brief description and the results of this project.

#### Optimal Control of a 2D Quadcopter  

**Description:**  
This system consists of a rectangular object in two dimensions which has controllable forces at both ends. It can be pictured as a viewing a quadcopter from the side. It is under-actuated (2 controls, 3 degrees of freedom) and therefore I thought it would be a good problem for which to utilize optimal control.  

**Dynamics:**  
ddx = (1 / m) * [(F1 + F2) * Sin(theta[t])]  
ddy = (1 / m) * [(F1 + F2) * Cos(theta[t]) – g]  
ddtheta = (1 / rotInertia) * [F2 * length/2 – F1 * length/2]  

The dynamics of the system were straightforward and were determined by drawing a free body diagram of the quadcopter. The mass, the length, the height, and the rotational inertia of the quadcopter was determined by looking at the specs of a DJI Mavic quadcopter.  

**Results:**  
<div style="text-align: center;">
  <img src="/assets/2d_quadcopter_control.gif" alt="2D Quadcopter Control" style="width: 80%; max-width: 650px; padding: 10px"/>
  <p>2D Quadcopter tracking the following trajectory: Travel to (1,2), draw a circle with a 1m radius, travel back to origin</p>
</div>

The end result of my efforts to create a 2D quadcopter system was a piece of code which takes xcenter and ycenter as inputs, moves to the (x,y) location defined by these points, tracks a circle with a 1m radius at that point, and returns to the origin. This works robustly for any points within the range x = [-1,1] and y = [-2,2]. Outside of this range, there were problems with computation and the result ends up diverging. I believe it may be possible to fix this by varying the amount of time for each movement. I will explore this concept further when applying this to a 3D quadcopter for the localization project. -->
