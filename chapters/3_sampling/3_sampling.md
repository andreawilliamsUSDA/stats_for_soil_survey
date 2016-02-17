---
html_document:
    keep_md: yes
author: Tom D'Avello, Stephen Roecker, Skye Wills
date: "Monday, February 14, 2016"
output: 
  html_document: 
    toc: yes
title: Chapter 3 - Sampling Design
---
![Statistics for pedologists course banner image](figure/logo.jpg)  

# CHAPTER 3: Sampling Design

## Introduction   

Sampling is a fundamental part of statistics. Samples are collected to achieve an understanding of the population, as it is usually not feasible to observe all members of a population. The goal is to collect samples that provide an accurate representation of the population under study. Time and money dictate that the sampling effort be efficient. Highly variable populations will require more samples to characterize their nature.  

**Define your purpose** - are you investigating soil properties, soil classes, plant productivity, etc.?   

**Expected variability** - the number of samples required increases with increasing variability.

**Acceptable variation** - the number of samples required corresponds to the acceptable confidence level. Sampling at the 85% confidence level will be less intensive than the 95% confidence level.  

## Sampling Strategies
 
### Simple random 

In simple random sampling, all samples within the region have an equal chance of being selected. A simple random selection of points can be made using either the `spsample()` function within the sp R package, or the Create Random Points tool in ArcGIS.

**Advantages**

 - Simplicity  
 - Requires little prior knowledge of the population    
       
**Disadvantages**

 - Lower accuracy                       
 - Higher Cost
 - Lower efficiency
 - Samples may be clustered spatially
 - Samples may not be representative of the feature attribute(s)  


```r
# load sp package
library(sp)

# Create a sixteen square polygon
grd <- GridTopology(c(1, 1), c(1, 1), c(4, 4))
polys <- as.SpatialPolygons.GridTopology(grd)
plot(polys, main = "Simple random sample")

# Generate simple random sample
test <- spsample(polys, n = 16, type = "random")
points(test)
```

![plot of chunk simple](figure/simple-1.png)


### Stratified random

In stratifed random sampling, the sampling region is spatially subset into different strata, and random sampling is applied to each strata. If prior information is available about the study area it can be used to develop the strata. Strata may be sampled equally or in proportion to area, however if the target or interest is rare in the population it maybe preferrable to sample the strata equally (Franklin, 2009).

**Advantages**

 - Higher accuracy
 - Lower cost 

**Disadvantages**

 - Existing knowledge used to construct strata maybe flawed


```r
plot(polys, main = "Stratified random sample")

# Generate stratified random sample
test <- spsample(polys, n = 16, type = "stratified")
points(test)
```

![plot of chunk stratified](figure/stratified-1.png)


### Multistage stratified random sampling

In multistage random sampling, the region is separated into different subsets that are randomly selected (i.e. first stage), and then the selected subsets are randomly sampled (i.e second stage). This is similar to stratified random sampling, except that with stratified random sampling each strata is sampled.

**Advantages**

 - Greater efficiency
 - Lower cost  

**Disadvantages**

 - Lower precision
 - stronger clustering than simple random sampling


```r
plot(polys, main = "Two-stage random")

# Select 8 samples from each square
s <- sapply(slot(polys, 'polygons'), function(x) spsample(x, n = 8, type= "random"))
points(sample(s, 1)[[1]]) # randomly select 1 square and plot
points(sample(s, 1)[[1]]) # randomly select 1 square and plot
```

![plot of chunk two_stage](figure/two_stage-1.png)


### Systematic

In systematic sampling, a sample is taken according to a regularized pattern. This approach insures even spatial coverage. Patterns may be rectilinear, triangular or hexagonal. This sampling strategy can be a problem if the variation of the population is cyclical.  

**Advantages**

 - Greater efficiency
 - Lower cost

**Disadvantages**

 - Lower precision


```r
plot(polys, main = "Systematic sample")

# Generate systematic random sample
test <- spsample(polys, n = 16, type = "regular")
points(test)
```

![plot of chunk systematic](figure/systematic-1.png)


### Cluster sampling

In cluster sampling, a cluster or group of points is selected at 1 or several sites. The transect is a example of this strategy, although others shapes are possible(e.g. square, triangle or cross shapes). It is common to orient the transect in direction of greatest variability.

**Advantages**

 - Greater efficiency
 - Lower cost   
 
**Disadvantages**

 - Lower precision  


```r
plot(polys, main = "Clustered (n = 3) random sample")

# Generate cluster random sample
test <- spsample(polys, n = 16, type = "clustered", nclusters = 3)
points(test)
```

![plot of chunk clustered](figure/clustered-1.png)


### Conditioned Latin hypercube (cLHS) 

Conditioned Latin hypercube sampling is a stratified random sampling technique that strives to obtain representative samples from feature (attribute) space (Minasny and McBratney, 2006). For example, assume you have prior knowledge of a study area, have the time and resources to collect 120 points and know the following variables (strata), represented as coregistered raster datasets, to be of importance to the soil property or class being investigated:  

 - Normalized Difference Vegetation Index (NDVI)  
 - Topographic Wetness Index (aka, Wetness Index, Compound topographic index)  
 - Solar insolation (Potential incoming solar radiation)  
 - Relative elevation (aka relative position, normalized slope height)  
 
The cLHS procedure will iteratively select samples from the strata variables such that they replicate the range of values from each stratum. Obtaining a sample that is representative of the feature space becomes increasingly difficult as the number of variables (strata) increase, unless one employs a technique such as cLHS.


```r
library(clhs)
library(raster)

data(volcano) # http://geomorphometry.org/content/volcano-maungawhau
volcano_r <- raster(as.matrix(volcano[87:1, 61:1]), crs = CRS("+init=epsg:27200"), xmn = 2667405, xmx = 2667405 + 61*10, ymn = 6478705, ymx = 6478705 + 87*10)
names(volcano_r) <- "elev"
slope_r <- terrain(volcano_r, opt = "slope", unit = "radians")

rs <- stack(volcano_r, slope_r)

s <- clhs(rs, size = 20, progress = FALSE, simple = FALSE)

plot(volcano_r)
points(s$sampled_data)
```

![plot of chunk clhs](figure/clhs-1.png)

```r
# Summary of clhs object
summary(s$sampled_data)
```

```
## Object of class SpatialPointsDataFrame
## Coordinates:
##       min     max
## x 2667420 2667980
## y 6478740 6479560
## Is projected: TRUE 
## proj4string :
## [+init=epsg:27200 +proj=nzmg +lat_0=-41 +lon_0=173 +x_0=2510000
## +y_0=6023150 +datum=nzgd49 +units=m +no_defs +ellps=intl
## +towgs84=59.47,-5.04,187.44,0.47,-0.1,1.024,-4.5993]
## Number of points: 20
## Data attributes:
##       elev           slope        
##  Min.   : 96.0   Min.   :0.01768  
##  1st Qu.:109.5   1st Qu.:0.14679  
##  Median :124.0   Median :0.25028  
##  Mean   :130.5   Mean   :0.26558  
##  3rd Qu.:150.2   3rd Qu.:0.37999  
##  Max.   :184.0   Max.   :0.55860
```

```r
# Summary of raster objects
cbind(summary(volcano_r), summary(slope_r))
```

```
##         elev       slope
## Min.      94   0.0000000
## 1st Qu.  108   0.1231178
## Median   124   0.2461519
## 3rd Qu.  150   0.3781388
## Max.     195   0.7510583
## NA's       0 292.0000000
```


## Tools for selecting Random features  

An ArcGIS tool is available for selecting random features from the [Job Aids page](http://www.nrcs.usda.gov/wps/PA_NRCSConsumption/download?cid=stelprdb1258054&ext=pdf). This tool will randomly select the specified number of features from a dataset or set of selected features in ArcGIS. It would be an ideal tool for the first stage of a two stage random sample.  

### Two-stage stratified random sample design using ArcGIS

Purpose - Investigators in the Monongahela National Forest were interested in quantifying the depth of organic surface horizons in soils correlated to the Mandy soil series  that formed under red spruce canopy on back slopes in the Upper Greenbrier Watershed (HUC 8 -05050003).  

Stage 1 - randomly selected sub-watersheds in the study area  
 
![R GUI image](figure/ch3_fig7.jpg)  

Strata - Sampling based on three stratum:  

 - Mandy soil map units (MfE, MfF, MfG and other)
 - Red Spruce canopy cover (>30% canopy, other)
 - Slope (<= 35%, >= 35%)  
 
Data layers - Input layers include coregistered raster data of each stratum, reclassed as follows:  

![R GUI image](figure/ch3_fig8.jpg)  

Raster Calculator is used to add data layers together:  

![R GUI image](figure/ch3_fig9.jpg)  

The resulting raster file has the following combinations:  

![R GUI image](figure/ch3_fig10.jpg)  

A guide to verify the selection of sample numbers allocated according to the proportionate extent of the strata:  

![R GUI image](figure/ch3_fig11.jpg)  

Create Random points within each sub-watershed:  
Open the Create Random Points tool in the Data Management -> Feature Class Toolbox  

![R GUI image](figure/ch3_fig12.jpg)  

The sub-watershed layer is specified as the Constraining Feature Class, with 50 points selected  

![R GUI image](figure/ch3_fig13.jpg)  

The resulting point file has 50 points per polygon  

![R GUI image](figure/ch3_fig14.jpg)  

A check to see if the sample points adequately represent the proportionate extent of the data is made by summarizing the GRIDCODE of the points:  

![R GUI image](figure/ch3_fig15.jpg)  

The results compare well to the extent of the population:  

![R GUI image](figure/ch3_fig16.jpg)


### cLHS using TEUI  

In addition to performing zonal statistics, the TEUI toolkit also includes a tools for cLHS based on the clhs package in R (Roudier, 2011).

 - Relative Elevation (aka relative position)  
 - Northwestness  
 - Normalized Difference Vegetation Difference (aka NDVI)  
 
![R GUI image](figure/ch3_fig17.jpg)

Open ArcGIS and add the TEUI Toolkit Toolbar by selecting the Customize > Toolbars and checking TEUI  

![R GUI image](figure/ch3_fig18.jpg)  

The TEUI Toolbar looks like this  

![R GUI image](figure/ch3_fig19.jpg)  

Open the Latin Hyper Cube Generator Tool  

![R GUI image](figure/ch3_fig20.jpg)  

![R GUI image](figure/ch3_fig21.jpg)  

The Tool requires that all raster data is in Imagine format (“img” extension) and share a common projection and resolution. 

The tool adds all raster layers in the Table of Contents to the Layers section. The layers to be used are checked.  

An exclusion layer will be used in this example. An exclusion layer is a binary raster with values of 0/1. Using an exclusion layer confines the selection of points to those areas with a raster value of 1.  

The output file will be a shapefile named “samples.shp”, and the Number of Points will be 30.  

The number of iterations has been increased from the default of 100 to 300. Increasing the number of iterations increases the processing time, but also increases the likelihood that the samples selected are representative of the selected strata.  

Click on Generate and let the routine process. This could take from several minutes to several hours depending on how large the area is in terms of columns and rows and how many layers are used.  

The resulting output shows 30 points confined to the watershed of interest:  

![R GUI image](figure/ch3_fig22.jpg)  

Comparing the frequency distribution of the samples to the population shows a reasonable representation, especially considering the small sample size.  

![R GUI image](figure/ch3_fig23.jpg)  

## How many samples are needed?  

If there is prior general knowledge of the mean and variance of the property being investigated, the following equation may be used:  

Number of samples = (t value)<sup>2</sup> (variance) / (estimated mean)(preceision)<sup>2</sup>  

For example, based on 20 previous samples, the thickness of loess in the study area is estimated to be 100 cm with a variance of 25 cm. How many samples are needed to be 95% sure of being within 5% of the mean?  

The t values for 95% confidence and 20 samples is (20 - 1) degrees of freedom = 2.093    
The level of precision is 5%, or 0.05 

The calculation follows:  

Number of samples = (2.093)<sup>2</sup> (25) / (100)(0.05)<sup>2</sup>  = 438  

Reducing the level of precision to 10%, the number of samples needed falls to 110. 

## References

Franklin, J., & Miller, J. A. (2009). Mapping species distributions: Spatial inference and prediction. Cambridge: Cambridge University Press. [http://www.cambridge.org/us/academic/subjects/life-sciences/ecology-and-conservation/mapping-species-distributions-spatial-inference-and-prediction](http://www.cambridge.org/us/academic/subjects/life-sciences/ecology-and-conservation/mapping-species-distributions-spatial-inference-and-prediction)

Roudier, P. clhs: a R package for conditioned Latin hypercube sampling. 2011. [https://cran.r-project.org/web/packages/clhs/index.html](https://cran.r-project.org/web/packages/clhs/index.html)

Minasny, B., & McBratney, A. B. 2006. A conditioned Latin hypercube method for sampling in the presence of ancillary information. Computers & Geosciences, 32(9), 1378-1388. [http://www.sciencedirect.com/science/article/pii/S009830040500292X](http://www.sciencedirect.com/science/article/pii/S009830040500292X)  

TEUI. USFS. [http://www.fs.fed.us/eng/rsac/programs/teui/downloads.html](http://www.fs.fed.us/eng/rsac/programs/teui/downloads.html)

## Additional reading

de Gruijter, J., Brus, D. J., Bierkens, M. F. P., & Knotters, M. (2006). Sampling for Natural Resource Monitoring: Springer. [http://www.springer.com/us/book/9783540224860](http://www.springer.com/us/book/9783540224860)

Schreuder, H.T., R. Ernst, H. Ramirez-Maldonado, 2004. Statistical techniques for sampling and monitoring natural resources. Gen. Tech. Rep. RMRS-GTR-126. Fort Collins, CO: U.S. Department of Agriculture, Forest Service, Rocky Mountain Research Station. 111 p. [http://www.fs.fed.us/rm/pubs/rmrs_gtr126.html](http://www.fs.fed.us/rm/pubs/rmrs_gtr126.html)

U.S. Environmental Protection Agency. (2002). Guidance for choosing a
sampling design for environmental data collection. Washington, DC: US EPA. [http://www.epa.gov/quality/guidance-choosing-sampling-design-environmental-data-collection-use-developing-quality](http://www.epa.gov/quality/guidance-choosing-sampling-design-environmental-data-collection-use-developing-quality)


