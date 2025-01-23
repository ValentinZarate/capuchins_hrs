# sim_akde Function

## Overview

The `sim_akde` function is designed to calculate the home ranges (AKDE) for various indiviuals based on telemetry data. The function processes multiple telemetry datasets (identified by `individual.local.identifier`), estimates movement models, and generates AKDE estimates for each group. The function outputs the results in the form of shapefiles and a summary dataframe, which includes the best-fit movement model, the 95% and 50% AKDE estimates, and the number of locations used in the estimation.

## Requirements

Before using the `sim_akde` function, ensure that the following prerequisites are met:

### 1. Data Format:
- The input `data` should be a `data.frame` containing at least the following columns:
  - `individual.local.identifier`: A unique identifier for each group's telemetry data.
  - `grupo`: The group to which the animal belongs.
  - `year`: The year of the observation.
  - `month`: The month of the observation.
  - Any additional columns required for creating a `telemetry` object.

### 2. Working Directory Structure:
- The function will create and organize output files in a specific directory structure:
  - `ctmm_fits/monthly/`: Directory where the fitted models (`fits`) will be saved as `.RDS` files.
  - `ctmm_fits/monthly/shapes/`: Subdirectory where the AKDE shapefiles (`.shp`) will be saved.
  
  If these directories do not exist, the function will create them automatically.

### 3. Dependencies:
- The following R packages are required to run the function:
  - `ctmm`: For movement model estimation and AKDE calculation.
  - `dplyr`: For data manipulation.
  - `sf`: For handling spatial data and shapefiles.
  - `readr`: For saving and reading `.RDS` files.

Install the necessary packages using:
```r
install.packages(c("ctmm", "dplyr", "sf", "readr"))
library(ctmm)
library(dplyr)
library(sf)
library(readr)

sim_akde <- function(data, output_dir = "ctmm_fits/monthly/") {
  
  shapes_dir <- file.path(output_dir, "shapes")
  
  if(!dir.exists(output_dir)) dir.create(output_dir, recursive = TRUE)
  if(!dir.exists(shapes_dir)) dir.create(shapes_dir, recursive = TRUE)
  
  results <- data.frame(
    individual.local.identifier = character(),
    best_fit = character(),
    akde_95_area = numeric(),
    akde_50_area = numeric(),
    n_locs = integer(),
    stringsAsFactors = FALSE
  )
  
  unique_ids <- unique(data$individual.local.identifier)
  
  for (id in unique_ids) {
    
    telemetry_data <- data %>% filter(individual.local.identifier == id)
    
    telemetry_object <- as.telemetry(telemetry_data, timezone = "America/Argentina/Buenos_Aires")
    
    guess <- ctmm.guess(telemetry_object, CTMM = ctmm(error = 20), interactive = FALSE)
    
    fits <- ctmm.select(telemetry_object, CTMM = guess, verbose = TRUE)
    
    saveRDS(fits, file = file.path(output_dir, paste0(id, "_fits.RDS")))
    
    akde_result <- akde(data = telemetry_object, CTMM = fits[[1]], weights = TRUE)
    
    akde_sf_95 <- as.sf(akde_result, level.UD = 0.95, level = 0.95) %>% st_transform(32721)
    akde_sf_50 <- as.sf(akde_result, level.UD = 0.5, level = 0.5) %>% st_transform(32721)
    
    akde_95 <- akde_sf_95 %>% filter(name == paste0(id, " 95% est"))
    akde_50 <- akde_sf_50 %>% filter(name == paste0(id, " 50% est"))
    
    st_write(akde_95, file.path(shapes_dir, paste0(id, "_akde_95.shp")), delete_layer = TRUE)
    st_write(akde_50, file.path(shapes_dir, paste0(id, "_akde_50.shp")), delete_layer = TRUE)
    
    best_fit_name <- if (is.list(fits) && length(fits) > 0) {
      summary(fits[[1]])$name
    } else {
      NA
    }
    
    results <- rbind(results, data.frame(
      individual.local.identifier = id,
      best_fit = best_fit_name,
      akde_95_area = as.numeric(st_area(akde_95)),
      akde_50_area = as.numeric(st_area(akde_50)),
      n_locs = nrow(telemetry_data)
    ))
  }
  
  return(results)
}
```
## Detailed Steps

### Directory Setup
The function first checks if the necessary directories exist and creates them if they do not. Specifically, it creates:

- A main output directory (`ctmm_fits/monthly/`) for storing the fitted models as `.RDS` files.
- A subdirectory for storing the AKDE shapefiles (`ctmm_fits/monthly/shapes/`).

### Loop Through Identifiers
The function iterates over each unique `individual.local.identifier` in the dataset. For each identifier:

- The relevant subset of data is extracted.
- A telemetry object is created using the `as.telemetry` function, specifying the appropriate timezone.

### Model Estimation

- An initial guess of the movement model is made using the `ctmm.guess` function.
- The best-fitting model is selected using `ctmm.select`, which returns a list of potential models. The first model in this list is assumed to be the best fit.

### AKDE Calculation

- The function calculates the AKDE for the given telemetry data and selected model using the `akde` function.
- The 95% and 50% AKDE estimates are converted to spatial objects (`sf`) and transformed to the appropriate coordinate system (`EPSG:32721`).

### Output Shapefiles

- The 95% and 50% AKDE estimates are saved as shapefiles in the `ctmm_fits/monthly/shapes/` directory, named according to their `individual.local.identifier`.

### Summary DataFrame

- For each identifier, the function stores the best-fit model, the estimated areas for 95% and 50% AKDE, and the number of locations used in a summary dataframe.

### Return

- The function returns the summary dataframe containing the results for all processed identifiers.

