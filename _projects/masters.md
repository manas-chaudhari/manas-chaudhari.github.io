---
layout: page
title: Masters Project | Energy Optimal Path Planning in Mobile Robots
date: "2013-07-01"

subtitles:
- "Guide: <a href=\"http://in.linkedin.com/pub/leena-vachhani/1/509/377\">Prof. Leena Vachhani</a>, Systems and Control Engineering, IIT Bombay"
- "July 2013 – July 2014"
- "Co-guide: <a href=\"http://www.ese.iitb.ac.in/faculty/rangan-banerjee\">Prof. Rangan Banerjee</a>, Energy Science and Engineering, IIT Bombay"

links:
- '<a href="http://www.youtube.com/watch?v=xrHlQodl8qQ">simulation demo</a>'
- '<a href="/public/docs/Manas-Chaudhari-Thesis.pdf">thesis</a>'
- '<a href="http://www.sciencedirect.com/science/article/pii/S1474667016326398">publication</a>'

---

{% include image.html src="rviz.png" title="Rviz in ROS" align="right" size="medium" %}
Objective: To develop strategies for minimizing energy consumption in autonomous mobile robot navigation.

In stage 1, I implemented the energy minimizing strategy proposed in the paper ‘Minimizing Energy Consumption of Wheeled Mobile Robots via Optimal Motion Planning’ by S. Liu and D. Sun. The method uses as A-star search algorithm variant which considers energy based constraints (friction) in its cost function and a trajectory planner which minimizes energy consumption by optimizing velocity and arrival time parameters at the waypoints. I used [ROS(Robot Operating System)](http://wiki.ros.org/) as the platform for this project. For simulation, I used [Gazebo](http://gazebosim.org/). It was found that the resultant waypoint selection from the method wasn’t optimal. Results have been published [here](http://www.sciencedirect.com/science/article/pii/S1474667016326398).

In stage 2, I improved waypoint selection by implementing A*, Theta* techniques to search on edges instead of points. Searching on edges allowed the cost function to consider costs for turning. I also developed an energy efficient trajectory planning technique which had linear computational complexity (vs exponential in existing method) and 14.8% more average energy savings.

I have used the software Gazebo for simulating a differential drive vehicle. Navigation goals are specified using Rviz from ROS. In this demo, the robot generates a smooth trajectory avoiding obstacles and follows it.

<iframe width="560" height="315" src="https://www.youtube.com/embed/xrHlQodl8qQ" frameborder="0" allowfullscreen></iframe>
