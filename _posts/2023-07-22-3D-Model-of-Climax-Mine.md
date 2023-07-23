---
layout: post
title:  "Animated 3D View of Climax Mine"
date:   2023-07-22 19:40
author: Joe Mason
---

<div class="col"><img src="{{ "/assets/img/Climax_Mine.jpeg" | relative_url }}" class="img-responsive" alt="</div>

I created an animated 3D visualization of the Climax Mine, near Leadville, CO, using a digital elevation model created from airborne lidar data collected in 2020-21 and National Aerial Image Program (NAIP) images from 2021. I downloaded the elevation data and NAIP images, resampled them to 3 m resolution, and mosaiced them to cover the area of the mine, using the {**rgeedim**} package (<a href="https://github.com/brownag/rgeedim" target="_blank">More on this package</a>). I used the {**rayshader**} (<a href="https://www.rayshader.com/" target="_blank">more here</a>) package to create the 3D model of topography overlain by the NAIP image, capturing it from 400 viewpoints using an R script and combining those images into an animation. An mp4 version of the animation is <a href="{{ "/assets/img/climax_mine.mp4" | relative_url }}" target="_blank"> here</a>. The animation is Copyright Joseph A. Mason; licensed under a <a href="https://creativecommons.org/licenses/by-nd/4.0/" target="_blank">CC-BY-ND License</a> 

I will update this post later with a generic R script I use to create visualizations like this using rayshader and rgeedim.
