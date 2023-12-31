---
layout: post
title:  "Animated 3D View of Climax Mine"
date:   2023-07-22 19:40
author: Joe Mason
---

I created an animated 3D visualization of the Climax Mine, near Leadville, CO, using a digital elevation model produced from airborne lidar data collected in 2020-21 and National Aerial Image Program (NAIP) images from 2021. I downloaded the elevation data and NAIP images, resampled them to 3 m resolution, and mosaiced them to cover the area of the mine, using the {**rgeedim**} package (<a href="https://github.com/brownag/rgeedim" target="_blank">More on this package</a>). I used the {**rayshader**} (<a href="https://www.rayshader.com/" target="_blank">more here</a>) package to create the 3D model of topography overlain by the NAIP image, capturing it from 400 viewpoints using an R script and combining those images into an animation. It can be downloaded from the viewer below. The animation is copyright Joseph A. Mason; licensed under a <a href="https://creativecommons.org/licenses/by-nd/4.0/">CC-BY-ND License</a>. 

See below for a generic R script I use to create visualizations like this using rayshader and rgeedim.

<iframe id="kaltura_player" src="https://cdnapisec.kaltura.com/p/1660902/sp/166090200/embedIframeJs/uiconf_id/25916071/partner_id/1660902?iframeembed=true&playerId=kaltura_player&entry_id=1_snrq826f&flashvars[streamerType]=auto&amp;flashvars[localizationCode]=en_US&amp;flashvars[sideBarContainer.plugin]=true&amp;flashvars[sideBarContainer.position]=left&amp;flashvars[sideBarContainer.clickToClose]=true&amp;flashvars[chapters.plugin]=true&amp;flashvars[chapters.layout]=vertical&amp;flashvars[chapters.thumbnailRotator]=false&amp;flashvars[streamSelector.plugin]=true&amp;flashvars[EmbedPlayer.SpinnerTarget]=videoHolder&amp;flashvars[dualScreen.plugin]=true&amp;flashvars[Kaltura.addCrossoriginToIframe]=true&amp;&wid=1_wz2pk02q" width="649" height="401" allowfullscreen webkitallowfullscreen mozAllowFullScreen allow="autoplay *; fullscreen *; encrypted-media *"  frameborder="0" title="Climax_Mine"></iframe>

#Script Used to Create this Animation

**Please note** that much of the code in this script is adapted or copied directly from tutorials by Tyler Morgan-Wall on the <a href="https://www.rayshader.com/" target="_blank">Rayshader website</a> and by Andrew Brown on the <a href="https://github.com/brownag/rgeedim" target="_blank">rgeedim website</a>; I would be flattered if you try out this script and create something new with it, but they and their R packages really deserve most of the credit.

Start with the libraries needed. As of 12/30/2023 these can all be install from CRAN. See the explanation of Google Earth Engine authentication in the rgeedim documentation.

```
library(rgeedim)
library(terra)
library(hillshader)
library(rayshader)
library(rayrender)
library(scales)
library(av)

#gd_authenticate(auth_mode = "notebook")
gd_initialize()

#Set working directory--fill in appropriate path
setwd(' ')

```

Define an area of interest as a bbox, use rgeedim download DEM data for that area (mosaicking and adjusting resolution and coordinate system as needed), and plot to check whether it looks right. See rgeedim documentation for more details, for this block and the next

```
#Area of interest using a bbox
b <- gd_bbox(
  xmin = -106.232,
  xmax = -106.165,
  ymin = 39.357,
  ymax = 39.422
)
r <- gd_region(b)

# download 1m lidar DEM (with resampling to 3m and compositing)
dem <- "USGS/3DEP/1m" |>
  gd_collection_from_name() |>
  gd_search(region = r) |>
  gd_composite(resampling = "bilinear") |>
  gd_download(region = r,
              crs = "EPSG:26913",
              scale = 3,
              filename = "dem.tif",
              overwrite = TRUE,
              silent = FALSE) |>
  rast()

# plot to make sure it's what you want
plot(terra::terrain(dem$elevation))

```

Now use rgeedim to search for, select, and download NAIP (<a href="https://naip-usdaonline.hub.arcgis.com/" target="_blank">ational Agricultural Imagery Program</a>) data. These are high-resolution digital air photos, collected at regular intervals for most of the US. You may want to initially search over a longer time interval or look up the years when these images were collected for your area of interest. rgeedim documentation will help here also.

```
# search NAIP collection for date range
img <- 'USDA/NAIP/DOQQ' |>
  gd_collection_from_name() |>
  gd_search(
    start_date = '2021-01-01',
    end_date = '2022-12-31',
    cloudless_portion = 85,
    region = b
  )

# list images found with metadata, %cloudless, date, etc. 
# %fill tells you how much of the box each image fills
gd_properties(img)

#download composite image
img_comp <- img |> 
  gd_composite(
    method = "mosaic",
    start_date = '2021-08-24',
    end_date ='2021-08-24'
  ) |>
  gd_download(
    filename = "image.tif",
    region = b,
    scale = 3,
    crs = 'EPSG:26913',
    dtype = 'uint16',
    overwrite = TRUE,
    silent = FALSE
  )
plot(rast(img_comp)[[1:4]])

# download a single image (included as an option but won't work with this demo)
# img_sel <- gd_properties(img)$id[3] |> 
#   gd_image_from_id() |>
#   gd_download(
#     filename = "image.tif",
#     region = b,
#     scale = 10,
#     crs = 'EPSG:26913',
#     dtype = 'uint16',
#     overwrite = TRUE,
#     silent = FALSE
#   )
#plot(rast(img_sel)[[1:4]])

```

Now import the DEM, then import each band of the NAIP image and create an image stack, plotting it to make sure it looks right. It's not needed for this example, but in some cases image and DEM data need to be put in the same coordinate reference systems. Finally, turn the DEM raster into a matrix for use in rayshader.

```
#Import data from tif to raster object for use in rayshader
localtif = raster::raster("dem.tif")

#Not needed for demo. Only use this if the dem and image are in different CRS, or
#you're not sure Can cause problems if they are already same CRS, not sure why
#Check CRS of image and dem
# raster::crs(img_r)
# raster::crs(localtif)
# #Reproject dem to image CRS
# localtif_utm = raster::projectRaster(localtif, crs = crs(img_r), method = "bilinear")
# #Confirm new CRS of image
# crs(localtif_utm)

#Import NAIP image bands as separate rasters
img_r = raster::raster("image.tif", band = 1)
img_g = raster::raster("image.tif", band = 2)
img_b = raster::raster("image.tif", band = 3)

#create image stack and plot to see what it looks like
img_rgb = raster::stack(img_r, img_g, img_b)
raster::plotRGB(img_rgb)

#convert elevation raster to a matrix
el_matrix = rayshader::raster_to_matrix(localtif)

```

The following part is best explained in a <a href="https://www.tylermw.com/a-step-by-step-guide-to-making-3d-maps-with-satellite-imagery-in-r/" target="_blank">rayshader tutorial</a> (but note that example uses a different type of imagery). Basically, you are turning each band of the image into a matrix, combining those matrixes into an array, then transposing the array to match the elevation data. To make sure this worked right, we'll plot the image from the transposed array, before and after a contrast enhancement.

```
#create overlay from rgb image
names(img_rgb) = c("r","g","b")

img_r = rayshader::raster_to_matrix(img_rgb$r)
img_g = rayshader::raster_to_matrix(img_rgb$g)
img_b = rayshader::raster_to_matrix(img_rgb$b)

img_rgb_array = array(0,dim=c(nrow(img_r),ncol(img_r),3))

img_rgb_array[,,1] = img_r/255 #Red layer
img_rgb_array[,,2] = img_g/255 #Green layer
img_rgb_array[,,3] = img_b/255 #Blue layer

img_rgb_array = aperm(img_rgb_array, c(2,1,3))

#plot image from array before and after enhancing contrast
plot_map(img_rgb_array)
img_rgb_contrast = scales::rescale(img_rgb_array,to=c(0,1.0))
plot_map(img_rgb_contrast)

```

Now comes the fun part, creating an interactive 3D rgl object, trying out different perspectives on it, and plotting snapshots with title added. It's definitely worth spending some time adjusting the perspective parameters and plotting new snapshots to decide what you want to put into the animation. Edit the title as needed. rayshader documentation has more on the parameters that set the perspective.

```
#make 3D rgl object with image overlay. Usually pretty fast.
plot_3d(img_rgb_contrast, el_matrix, windowsize = c(1000,800), zscale = 2, shadow=FALSE, 
        #shadowdepth = -50, shadowcolor = "#523E2B",
        zoom=0.64, phi=30,theta=-30,fov=70, background = "#F2E1D0")

#Try out a couple of viewpoints to see how this works
#Change numbers to explore other viewpoints before setting them below
render_camera(theta = 220, phi=30,zoom= 0.80, fov= 56 )
render_camera(theta = 330, phi=40,zoom= 0.30, fov= 56 )
render_snapshot(title_text = "Climax Mine, Colorado | DEM: 3m | Image: 2021 NAIP",
title_bar_color = "#1f5214", title_color = "white", title_bar_alpha = 1)

```

The following is my own method for automating a flyover tour of the scene you are looking at, zooming in and out and circling around it however you want. The for-loop creates a series of snapshots that are then used to make an animation. Please contact me with any questions about this part, in particular.

```
#Generalized code for using the rgl object to make 
#an animation that moves through various camera viewpoints

#Decide on view points including start and end point. Each view point is a row in PathMatrix.
#Enter each view point's values for theta(0-360), phi(10-60), zoom(0.1-1.0) in
#the first three columns. Leave zeros in the other three columns. Add/remove rows as needed.
Numpoints = 6 #Count start and end point
PathMatrix = matrix(nrow=Numpoints, ncol=6, byrow=TRUE,
                    c(220, 30, 0.70, 0.0, 0.0, 0.0,
                      180, 30, 0.50, 0.0, 0.0, 0.0,
                      180, 20, 0.30, 0.0, 0.0, 0.0,
                      330, 30, 0.40, 0.0, 0.0, 0.0,
                      180, 40, 0.50, 0.0, 0.0, 0.0,
                      220, 30, 0.70, 0.0, 0.0, 0.0))

#Number of frames to get between points, including 0 for start
Speeds = c(0, 80, 80, 80, 80, 80) 

stepsize = function(startpoint, endpoint, speed) {
  if(startpoint==endpoint){
    stepsize = 0.0
  } else stepsize = (endpoint-startpoint)/speed
}

for(i in 1:Numpoints){
  if(i==1){
    PathMatrix[i,4:6]=0.0
  } else for(k in 4:6){
    PathMatrix[i,k]=stepsize(PathMatrix[i-1,k-3], PathMatrix[i,k-3], Speeds[i])
  }
}

render_camera(theta=PathMatrix[1,1], phi=PathMatrix[1,2], zoom=PathMatrix[1,3], fov=56)
TotalSteps=0
for(k in 2:Numpoints){
  theta=PathMatrix[k-1, 1]
  phi=PathMatrix[k-1, 2]
  zoom=PathMatrix[k-1, 3]
  for(j in 1:Speeds[k]) {
    theta=theta+PathMatrix[k,4]
    phi=phi+PathMatrix[k,5]
    zoom=zoom+PathMatrix[k,6]
    i=TotalSteps+j
    render_camera(theta=theta, phi=phi, zoom=zoom, fov=56)
    #in the next line replace ... with the path to where you want to save the images that will become animation frames
    render_snapshot(filename = sprintf(".../Climax%i.png", i), 
                    title_text = "Climax Mine, Colorado | DEM: 3m | Image: 2021 NAIP",
                    title_bar_color = "#1f5214", title_color = "white", title_bar_alpha = 1)
  } 
  TotalSteps=TotalSteps+Speeds[k]
}

#creates the movie; again fill in the right path to replace ...
av::av_encode_video(sprintf(".../Climax%i.png",seq(1,TotalSteps,by=1)), framerate = 10,
                    output = "Climax_Mine.mp4")

#Closes rgl object. Deprecated but still works, I guess
rgl::rgl.close()

```