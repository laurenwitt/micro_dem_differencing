# Estimating sediment mass displacement using DEM differencing
  
Amanda Manaster  
GDA 2019

## Summary
Development of a Python script to estimate sediment mass displacement on a small scale using DEM differencing.

### Background
DEM differencing is a powerful tool for change detection. You can see how a glacier has retreated, how a slow-moving landslide has developed, or how a river's meanders have changed over time. Many tools exist to aid in change detection, and I'm interested in using those tools to help automate the change detection process using a simple Python script.

### Objectives
The main objective of this project is to create a Python script automates estimating sediment mass displacement on a small scale. Ultimately, this script will be used to determine sediment load coming from logging roads.

## Datasets
- DEMs from UAV flights done in Spring 2018 for Advanced Surveying at the University of Washington

## Packages
- Rasterio (https://rasterio.readthedocs.io/en/stable/index.html)
- GDAL: gdalwarp (https://www.gdal.org/gdalwarp.html)

## Approach
### Initial processing of DEMs
Using rasterio, read in DEMs.  
Using gdalwarp, warp/resample the DEMs to ensure the following:
- identical projection
- identical extent
- identical resolution  

### Change detection analysis
Once the DEMs are processed:
1. Difference the DEMs
2. Assign a noise threshold and an outlier threshold
	- Noise Threshold = abs(standard deviation(elevation difference) + mean elevation difference)
	- Outlier Threshold = abs(6 * standard deviation(elevation difference) + mean elevation difference)
	- **NOTE: More sophisticated methods of determining the thresholds will be used in future iterations of this script**
3. Create a mask that shows only data that fall between the noise threshold and the outlier threshold
4. Determine volume of sediment displacement
	- Volume = elevation difference * cell width * cell height
5. Convert the total volume displacement to total mass displacement using approximate density of sediment

## Expected outcomes
A script that automates the change detection process and is easily applied to different scenarios.

## Extra information
- Resource discussing the principles of topographic change detection:
	https://cloud.sdsc.edu/v1/AUTH_opentopography/www/shortcourses%2FA2HRT_RCN%2FA2HRT_Wheaton_ChangeDetectionPrinciples_2018.pdf