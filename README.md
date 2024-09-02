# AKDE Estimation and Spatial Analysis Functions

This repository contains a collection of custom functions developed for estimating and analyzing Auto-correlated Kernel Density Estimate (AKDE) home ranges using `ctmm` as the primary dependency. The functions included here are designed to simultaneously calculate AKDE for multiple individuals or groups, storing relevant information in easily manageable formats, including shapefiles and summary tables.

## Key Features

### 1. Segmented AKDE Estimation
The functions in this repository allow for the estimation of AKDE across different time periods (e.g., months, seasons, weeks), automatically segmenting the data to generate precise estimates for each time interval. The results include both shapefiles and tables that document the estimated areas and the fitted movement models.

### 2. Overlap Calculation
In addition to AKDE estimation, this repository includes functions to calculate the overlap between the resulting polygons, which is crucial for assessing interactions between individuals or groups over time.

### 3. Spatial Use Metrics
Additional functions are provided to calculate other spatial use metrics, allowing for a detailed assessment of spatio-temporal changes in tracked species. These metrics offer key insights for ecological and behavioral studies.

## Dependencies
This repository primarily depends on the `ctmm` package, along with other R packages such as `dplyr`, `sf`, and `readr` for data manipulation and shapefile generation.

## Usage
To use the functions in this repository, clone the repository to your local machine and ensure that the necessary dependencies are installed. You can start estimating AKDE and analyzing spatial overlaps by calling the included functions and adapting them to your specific telemetry data analysis needs.
