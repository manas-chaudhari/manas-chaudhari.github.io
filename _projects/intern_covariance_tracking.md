---
layout: page
title: Summer Internship 2011 | Image Processing
date: "2011-05-01"

subtitles:
- 'Guides: <a href="http://m.i.c.h.e.l.e.free.fr/">Prof. Michèle Gouiffès</a>, <a href="http://www.ief.u-psud.fr/~bouchafa/">Prof. Samia Bouchafa</a>'
- "May - July 2011"
- '<a href="http://www.ief.u-psud.fr/">Institut d’Electronique Fondamentale</a>, Université Paris SUD XI'

links:
- '<a href="/public/docs/covariance_tracking_simd_report.pdf">project report</a>'

---

{% include image.html src="covariance-tracking.jpg" title="Montage of tracked objects in an image sequence" align="left" size="medium" %}
#### Covariance Tracking

This project involved developing a code to track a colored non-rigid object from an image sequence using the Covariance Tracking algorithm. The Covariance Tracking algorithm is a technique used to improve the robustness and adaptability of surveillance tracking systems in their applications in aerial surveillance and traffic management. Covariance Tracking involves the computation of covariance matrix of different features for storing the object model. The object is tracked by computing the similarity between the covariance matrix in the search window with the model covariance  matrix. One of the parts of this project was to implement model update strategies to increase the accuracy of the method.
