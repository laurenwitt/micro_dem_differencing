# Using high-pass filter differences to gain insight on forest service road sediment transport

**Lauren Wittkopf**  

**CEE 467, Geospatial Data Analysis, Winter 2023**

## Summary
This repository contains code used to estimate sediment mass displacement on a small scale using digital elevation modeling (DEM) differencing. This code improves on 
the original work done by Amanda Alvis (Manaster) through the use of a high-pass Gaussian filter provided by David Shean and Shashank Bhushan. 

## Repository Navigation
The main code is within micro_dem_diff, final_project_presentation is a trimmed down, simpler version of the more robust code base in final_project. The 
best notebook to look at is final_project_np.ipynb.

All necessary information is within this Jupyter Notebook. Objectives, methods and conclusions are reproduced here, and data files are stored in MEL14_data. 

## Introduction
Erosion of gravel forest roads can have significant impacts on hydrology and sediment transport processes in forested ecosystems. Understanding the erosion processes 
occurring on forest roads is critical to mitigating sediment production in an effective and efficient way. This project is a part of Amanda Alvis' Ph.D. research 
investigating the sediment transport dynamics of mainline logging roads in western Washington and contributes by using high-resolution digital surface models (DSMs) 
to help parameterize a larger sediment transport model.

## Objectives
The objectives of Amanda's research are to identify areas of high sediment accumulation and erosion and to characterize the extent and rate of wheel rut development, 
with the overall goal being to investigate the impact of traffic and maintenance practices on forest road erosion.

The objectives of my project are to develop a data analysis product that takes an input of a GeoTiff DSM in an arbitrary coordinate system and produces figures that 
aid in sediment transport analysis 

## Methods 
### DSM Creation

I used Pix4D to create high-resolution DSMs of the study area that are in an arbitrary coordinate system. This allows me to export a GeoTiff after 3 steps of 
processing. This process was repeated for multiple flight dates to generate a time series of DSMs.

### Loading data and adding overviews

I added Gaussian overviews to these DEMs using `gdaladdo` and then opened them using rioxarray to produce xArray DataSets. I opened overview 2 to use for testing, 
and the full resolution to use in presentation. 

### Clipping, Reprojecting and Matching

These DataArrays could then be clipped to a QGIS-drawn polygon using `xda.rio.clip(mask_gdf.geometry)` where xda is the xArray DataArray and mask_gdf is the 
shapefile loaded using `gpd.read_file()`. One of these clipped DataSets also then had to be reprojected to match the other one, so that they can be differenced. I 
did so using `xda1_proj = rioxarray.rio.reproject_match(xda2)` I also extracted the extent for plotting all the data from the xArray DataSet using the [rio accessor 
base class](https://corteva.github.io/rioxarray/html/rioxarray.html) `xds.rio.bounds()`.

### Differencing

After two datasets have been loaded and reprojected to match abitrary coordinate systems, they can be differenced and plotted. Amanda did this with rasterio-powered 
masked 2D arrays, but using rioxarray-powered DataArrays allowed me to reproject and match without using gdalwarp, to load overviews that make processing time 
faster, and to keep the data in an arbitrary engineering CRS that can be located in an EPSG CRS later. Similar to the plots Amanda produced with rasterio however, 
these difference figures show long-wave deposition and erosion across the entire road (wave is oriented longitudinally), which is likely an artifact from the drone 
surveys and processing of the images to make the DSMs. This is when the high-pass filter comes in.

### High-pass Gaussian filtering

The analysis needs to be done on the change in the micro-topographical features on the road over time, so differencing that mostly shows the long wave noise from 
processing is not ideal. The high-pass filter allows me to perform another kind of difference on the simple difference I made before. This time I apply a high-pass 
filter (there are elements of Gaussian sampling and Fourier fast transform helping with the pygeotools function that can do this...I am not sure of the details) to 
the initial difference map, which produces a smoothed version of the difference map, with statistically-sound methods of smoothing the roads features. I can change 
the sigma cut-off value to change the size of features that it is smoothing. For the second overviews of these DEMs I chose to use a sigma value of 10, which 
correlates to kernel smoothing size of 61 pixels, chosen by guidance from David and because by visual inspection and comparision I can see that the smoothed version 
does not have visible ruts, and so it will enable us to see the true rut formation in the next step in high-detail. For the full-resolution of these DEMs a sigma 
value of 50 works better. The next step is to take the difference of the initial difference and the smoothed version. This results in a DataArray that can be plotted 
to clearly demonstrate only the micro-topographical changes in the road between the two time periods, without the long-wave noise artifact from processing.

##Conclusions
The main conclusion is that using the pygeotools fourier fast transform gaussian high-pass filter on these structure from motion surveys allows me to difference them 
without the influence of long-wave artifacts on the true deposition and erosion patterns. I also learned that doing the analysis on DataArray produced using 
rioxarray is an easier and more efficient way to handle these DEMs than opening them as 2D raster arrays using rasterio. Using the rioxarray objects allows me to 
difference them without orienting them in a geolocated CRS, although I did find it useful to use the GCP-guided coordinates to plot according to an accurately sized 
extent (made meters into meters, rather than being oriented in pixels). 
