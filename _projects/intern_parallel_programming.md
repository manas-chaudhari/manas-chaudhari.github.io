---
layout: page
title: Summer Internship 2012 | Parallel Programming
date: "2012-05-01"

subtitles:
- 'Guide: <a href="http://m.i.c.h.e.l.e.free.fr/">Prof. Michèle Gouiffès</a>'
- "May - July 2012"
- '<a href="http://www.ief.u-psud.fr/">Institut d’Electronique Fondamentale</a>, Université Paris SUD XI'

links:
- '<a href="/public/docs/covariance_tracking_simd_report.pdf">project report</a>'

---

This project involved accelerating the covariance tracking code by using Single Instruction Multiple Data (SIMD) instruction set (SSE by Intel). Using SIMD, a vector data operation can be performed in a single instruction, reducing the overall computation time. Thus, an operation of adding two vectors of 4 integers element by element can be completed using only a single instruction. Different modules of the code were accelerated separately. Results have been documented in the report.
