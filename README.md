# LULC_IndiaSAT
A multi-level land use and land cover classification. 

# Input layers
To generate land use and land cover maps, we use multiple data sources as input-
- Satellite data: We use multi-spectral data from [Landsat-7](https://developers.google.com/earth-engine/datasets/catalog/LANDSAT_LE07_C02_T1_L2), [Landsat-8](https://developers.google.com/earth-engine/datasets/catalog/LANDSAT_LC08_C02_T1_L2), [Sentinel-2](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR), [MODIS](https://developers.google.com/earth-engine/datasets/catalog/modis), and [Sentinel-1](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S1_GRD) satellite constellations. Different combinations of this data are used to create feature vectors that serve as input to different machine learning algorithms. We use Google Earth Engine to access these datasets.
- Dynamic World: Google's land use and land cover product is used to identify certain land cover classes like built-up, water, etc. We access the [Dynamic World](https://developers.google.com/earth-engine/datasets/catalog/GOOGLE_DYNAMICWORLD_V1) data repository from Google Earth Engine.
- Open Street Maps: This data is used in the construction of the IndiaSAT groundtruth. [Open Street Maps](https://www.openstreetmap.org/)
- Input Shapefiles: To run LULC for any region of interest, you require to give its shapefile (uploaded as a GEE asset) as input. For example- you can access different administrative boundaries in India at [link](https://drive.google.com/drive/folders/1HJef1T8wJg8eoz4kUU8u1HHNoH3510Zv?usp=drive_link).

# Methodology
We perform a hierarchical land use and land cover classification and use different datasets and methods at each level of classification.

## Level-1 Classification
At the first level we classify pixels into 4 broad LULC classes- Greenery, Water Body, Built-up, and Barren land. We refer to the [IndiaSAT paper](https://dl.acm.org/doi/abs/10.1145/3460112.3471953?casa_token=_CBHFrHEAvIAAAAA:KVY9zoriQLArqUFlfWzU41QTySRED-DG_Eoyfr41DcL6fkgH5QhxlBrpx6y-jgp9R3cLVvSWaj7P) to produce this classification using Sentinel-2 multi-spectral data as input to a random forest classifier. We also use Dynamic World's output and aggregate their classes into these 4 categories. Then, using a rule-based method, we combine the outputs of both IndiaSAT and Dynamic World to create the final classification at this level. These rules take into account manual visualizations of results to identify strengths and weakness of both the classifiers (IndiaSAT being a per-pixel classifier and Dynamic World being an object-based classifier).

## Level-2 Classification
At the second level of classification, we classify the greenery pixels further into 2 main classes- Croplands and Forests/Trees. We use Sentinel-1 SAR data time series to perform this classification. The training data at this level is partly taken from the IndiaSAT groundtruth used in level-1 and is partly marked manually through visualization on Google Earth Pro. It is ensured that the groundtruth for both categories is geographically well distributed across different agro-climatic zones in India. 
We use a random forest classifier with 100 trees as the classification model that takes as input an annual 16-day time series of VV and VH bands from Sentinel-1.

We further use Slope information from [SRTM DEM](https://developers.google.com/earth-engine/datasets/catalog/CGIAR_SRTM90_V4) and use a threshold of 30 degrees to correct misclassifications in croplands, if any.

## Level-3 Classification
At the third level of classification, we classify cropland pixels identified at level-2 into 4 categories based on their cropping pattern- Single Kharif, Single Non-Kharif, Double, and Triple cropping. To perform this classification, we did not have access to any training data. So, we used unsupervised classification using K-nearest neighbour algorithm. We randomly sampled cropland pixels from all agro-climatic regions in India (marked from the cropland groundtruth at level-2). The feature vector for this classification is a 16-day NDVI time series that is derived from a combination of Landsat-7, Landsat-8, Sentinel-2, and MODIS data. This time series is generated on the lines of [GCI-30 paper](https://essd.copernicus.org/articles/13/4799/2021/). We do not perform Whittaker smoothing to avoid missing out important peaks in the time series. 
Initially we created 16 clusters using KNN method and then hierarchically split them into 2 whenever the distortion exceeded the threshold of 0.23. We manually labeled each cluster by interpreting the spread of time series that belonged to that cluster. These clusters are then used to assign the classification label based on euclidean distances to their centroids. The centroids are uploaded as asset on Google Earth Engine and can be accessed with the asset ID- "projects/ee-indiasat/assets/L3\_LULC\_Clusters/Non\_Padded\_Original\_Clusters\_WithLabels"

# Hosting specifications
- GDrive: https://drive.google.com/drive/folders/1qRtz4g3az_BLVKB1vUBcgIAKAy4RJRwS?usp=drive_link
- Earth engine: asset path where final outputs will be hosted 'projects/ee-indiasat/assets/LULC\_CombinedOutputs\_WithConfidence/'
- Type: raster
- Spatial resolution: 30m
- Temporal resolution: annual
- Temporal availability: 2017-2022 hydrological years

# Codebase
Execute the following scripts in sequence to generate the annual LULC output for any region of interest for all hydrological years 2017-2022.
- First step is to upload the shapefile of your region of interest as an asset to Google Earth Engine.
- Level-1 classification script: It is Google Colab script that you can run with your own earth engine credentials. In the main function enter the asset path of your region of interest. Run all the cells and the Level-1 output will get generated and automatically exported to the asset path specified.
- Level-2 classification script: It is Google Colab script that you can run with your own earth engine credentials. In the main function enter the asset path of your region of interest. Run all the cells and the Level-2 output will get generated and automatically exported to the asset path specified.
- Level-3 classification script: It is Google Colab script that you can run with your own earth engine credentials. In the main function enter the asset path of your region of interest. Run all the cells and the Level-3 output will get generated and automatically exported to the asset path specified.
- Combine Level-1,2,3 classification script: It is Google Colab script that you can run with your own earth engine credentials. Enter the path of the Level-1, Level-2, and Level-3 outputs generated in previous steps and it will export the final combined output to your assets in Google Earth Engine.    

# The final labels after classification
- 1: Greenery
- 2: Water
- 3: Builtup
- 4: Barrenland
- 5: Cropland
- 6: Forest
- 9: Single Kharif
- 10: Single Non Kharif
- 11: Double
- 12: Triple    
