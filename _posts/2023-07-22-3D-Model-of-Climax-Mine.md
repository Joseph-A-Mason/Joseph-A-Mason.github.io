---
layout: post
title:  "Animated 3D View of Climax Mine"
date:   2023-07-22 19:40
author: Joe Mason
---

I created an animated 3D visualization of the Climax Mine, near Leadville, CO, using a digital elevation model produced from airborne lidar data collected in 2020-21 and National Aerial Image Program (NAIP) images from 2021. I downloaded the elevation data and NAIP images, resampled them to 3 m resolution, and mosaiced them to cover the area of the mine, using the {**rgeedim**} package (<a href="https://github.com/brownag/rgeedim" target="_blank">More on this package</a>). I used the {**rayshader**} (<a href="https://www.rayshader.com/" target="_blank">more here</a>) package to create the 3D model of topography overlain by the NAIP image, capturing it from 400 viewpoints using an R script and combining those images into an animation. It can be downloaded from the viewer below. The animation is copyright Joseph A. Mason; licensed under a <a href="https://creativecommons.org/licenses/by-nd/4.0/">CC-BY-ND License</a>. 

I will update this post later with a generic R script I use to create visualizations like this using rayshader and rgeedim.

<iframe id="kaltura_player" src="https://cdnapisec.kaltura.com/p/1660902/sp/166090200/embedIframeJs/uiconf_id/25916071/partner_id/1660902?iframeembed=true&playerId=kaltura_player&entry_id=1_snrq826f&flashvars[streamerType]=auto&amp;flashvars[localizationCode]=en_US&amp;flashvars[sideBarContainer.plugin]=true&amp;flashvars[sideBarContainer.position]=left&amp;flashvars[sideBarContainer.clickToClose]=true&amp;flashvars[chapters.plugin]=true&amp;flashvars[chapters.layout]=vertical&amp;flashvars[chapters.thumbnailRotator]=false&amp;flashvars[streamSelector.plugin]=true&amp;flashvars[EmbedPlayer.SpinnerTarget]=videoHolder&amp;flashvars[dualScreen.plugin]=true&amp;flashvars[Kaltura.addCrossoriginToIframe]=true&amp;&wid=1_wz2pk02q" width="649" height="401" allowfullscreen webkitallowfullscreen mozAllowFullScreen allow="autoplay *; fullscreen *; encrypted-media *"  frameborder="0" title="Climax_Mine"></iframe>
