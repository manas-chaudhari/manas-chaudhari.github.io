---
layout: page
title: Intelligent Ground Vehicle Competition
date: "2012-12-01"

subtitles:
- ''
- "December 2012 - June 2014"

links:
- '<a href="http://www.igvc.org/">IGVC home</a>'
- '<a href="http://www.umic-iitb.org/igvc.php">team website</a>'
- '<a href="http://www.igvc.org/design/2013/Indian%20Institute%20of%20Technology%20Bombay.pdf">design report</a>'
- '<a href="http://www.youtube.com/watch?v=77CAyhHuXCg">basic arena video</a>'
---

{% include image.html src="pushpak2.jpg" title="Pushpak 2 at IGVC 2013" align="right" size="medium" %}
I was part of my institute’s team for Intelligent Ground Vehicle Competition, an international robotics competition, conducted by Oakland University, Michigan. The goal of this competition is to build a vehicle which will autonomously navigate through an obstacle course by detecting white lines, avoiding objects, following GPS waypoints. The competition is held in June every year.

#### 2013
We secured 7th position in the AutoNav Challenge with our vehicle “Pushpak 2”. My work involved implementing modules for communicating with our autonomous vehicle based on Joint Architecture for Unmanned Systems (JAUS). The base platform for our software was LabVIEW. The module for JAUS was implemented in C++ and it was called by importing the DLL in LabVIEW.

#### 2014
We shifted our platform to ROS. We rebuilt all components of the software. We made major improvements in the navigation strategy. Our work involved implementing:

- Localization using sensor data from GPS, IMU, LIDAR, camera
- Create a global Mapping using LIDAR to detect obstacles and camera to detect white lines
- Path planning which also considers the known characteristics of the surroundings
