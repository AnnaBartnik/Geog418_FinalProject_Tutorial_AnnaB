Geog418_FinalProject_Tutorial_AnnaB
# Lab 4Final Project: Spatial Analysis of Wildfires in BC
Author: Anna Bartnik

# 1. Introduction
## Background
British Columbia (BC) is prone to various climate-mediated events, including wildfires, which pose significant risks to both people and infrastructure. In recent years, the province has seen an increase in wildfire activity, largely attributed to rising temperatures, drought conditions, and changing precipitation patterns driven by climate change. Understanding the spatial distribution of these fires, as well as the climate factors that contribute to their occurrence, is essential for developing effective emergency management strategies aimed at minimizing property loss, injuries, and fatalities.

Temperature, in particular, plays a critical role in wildfire occurrence and intensity. For instance, higher temperatures can increase the likelihood of fire ignition and accelerate fire spread. However, analyzing the relationship between climate factors (such as temperature) and wildfire events requires careful spatial analysis. This tutorial aims to guide users through the process of spatial analysis using poorly distributed climate data and wildfire event data, showing how such data issues can lead to misleading conclusions when applying spatial regression and interpolation techniques.

## Problem Statement
Spatial analysis of climate data, particularly when investigating events like wildfires, is often challenged by the following issues:

Data clustering: When data points are overly concentrated in certain areas (e.g., wildfire occurrences clustered near urban areas), it can distort the analysis, leading to inaccurate results. Clustering can mask underlying trends and biases spatial models.

Data quality: Poor-quality data, such as missing values, inconsistencies, or low resolution, can undermine the reliability of spatial regression and interpolation methods, making it difficult to draw meaningful conclusions from spatial analyses.

## Research Question:
How does spatial autocorrelation in poorly distributed datasets affect the assumptions and outcomes of regression and spatial interpolation techniques used to analyze climate impacts on wildfire events in British Columbia?

## Objectives
This tutorial aims to provide a step-by-step guide on performing spatial analyses while highlighting how data clustering and poor data quality can influence the results. The specific objectives include:

* Reading spatial data: Learn how to read in different types of spatial data (e.g., shapefiles, CSV files) and clean multiple datasets.

* Point pattern analysis and spatial autocorrelation: Understand how to conduct point pattern analysis and assess spatial autocorrelation in poorly distributed datasets.

* Spatial interpolation: Learn how to apply spatial interpolation techniques and explore their limitations in the context of data clustering.Techniques such as Inverse Distance Weighting, and Kriging/ shoudl i inclue ordinary kriging?

* Regression analysis: Explore how spatial autocorrelation impacts regression models, and assess the validity of these models when the underlying data quality is low.

* Evaluating results: understand the impact of clustering and data quality on spatial analyses.

Through these steps, the tutorial will demonstrate not only how to perform the analyses but also how poor data quality can lead to incorrect assumptions and biased outcomes, encouraging a more critical approach to spatial analysis in climate studies.

# Getting Started:
For this tutorial, you will need to install and load the following libraries
```{r setup workspace, include=FALSE}
# Ensure installation of these packages before proceeding. 
install.packages(tmap)
install.packages(spdep)
install.packages(raster)
install.packages(sf)
install.packages(lubridate)
install.packages(dplyr)
install.packages(gstat)
install.packages(ggplot2)
install.packages(maps)
install.packages(viridis)
install.packages(spgwr)

# Next Load all necessary Packages:
library(tmap)
library(spdep)
library(raster)
library(sf)
library(lubridate)
library(dplyr)
library(gstat)
library(ggplot2)
library(maps)
library(viridis)
library(spgwr)

# Set working directory
dir <- "C:/Users/aabar/Desktop/university/Year5/Geog418_LAB/Lab4" #Change the directory based on the location you save your files to
setwd(dir)
```

# 2. Data  
## Data description 

In this study, three primary datasets are used to analyze the spatial distribution of wildfires in British Columbia, focusing on the influence of climate data and the challenges posed by clustering in poorly distributed datasets.

## Shapefiles: BC Boundary
The first dataset consists of a shapefile of British Columbia, which was downloaded from [insert source here]. Once the shapefile was imported into R, I noticed that the coordinate reference system (CRS) was initially set to EPSG: 3347. For consistency with the climate and wildfire data, the CRS was transformed to EPSG: 3005 (BC Albers), which is suitable for large-scale spatial analysis in British Columbia. This step is crucial when working with shapefiles to ensure proper alignment with other datasets, especially when comparing spatial data across different layers.

The code to load the BC shp boundary and transform the CRS is as follows:
```{r Load BC Boundary 1st time, echo = TRUE, results= 'hide', warning= FALSE, message= FALSE}
# Load BC boundary shapefile
bc_SHP_boundary <- st_read("BC.shp")
#check the crs of the shp
st_crs(bc_SHP_boundary)
# Transform the BC boundary to EPSG: 3005 to ensure a consistent CRS
bc_SHP_boundary <- st_transform(bc_SHP_boundary, crs = 3005)
```

## Wildfire Data: Fire Incident Locations
The wildfire data for this study was obtained from the BC Wildfire Service, which tracks historical wildfire incident locations across the province. The data includes all types of incidents, such as actual fires, suspected fires, and nuisance fires, and is updated annually with new data from the previous fire season.

To use this dataset, I downloaded the KMZ files containing historical wildfire incident points and converted them into shapefiles using an online KMZ-to-SHP conversion tool. As with the BC boundary shapefile, it was essential to ensure that the wildfire data was in the correct CRS for the analysis. After loading the wildfire points shapefile into R, I transformed the CRS to EPSG: 3005 to match the boundary shapefile and climate data.

The code for transforming the wildfire points CRS is as follows:
```{r Load Wild Fire Points 1st time, echo = TRUE, results= 'hide', warning= FALSE, message= FALSE}
# Load the wildfire shapefile
fire_shapefile <- "C:/Users/aabar/Desktop/university/Year5/Geog418_LAB/Lab4/BC_Fire_Points_2023-point.shp"  # Adjust path
fire_points <- st_read(fire_shapefile)
# Check and transform fire points CRS to match BC boundary
st_crs(fire_points)  # Check CRS of fire points
fire_points <- st_transform(fire_points, crs = 3005)  # Transform to EPSG 3005
```
## Climate Data: Temperature
The climate data used for this analysis was downloaded from the Pacific Climate Impacts Consortium’s (PCIC) BC Station Data Portal. For the purposes of this tutorial, I selected data from the BC Hydro station network (station ID: BCH), which includes over 150 stations across the province. The dataset contains daily and hourly observations for the year 2023, with the temperature variable being the daily maximum temperature "MAX_TEMP" in °C. This station was chosen specifically to demonstrate highly clustered data, which is important for understanding how spatial autocorrelation and clustering issues can affect spatial analysis.

##Data Cleaning and Aggregation:
The raw data was cleaned by merging station metadata with temperature observations, removing outliers, and filtering for unrealistic values. Temperature data was aggregated to daily, monthly, and seasonal averages to facilitate spatial analysis.


## Data Preprocessing and Formatting
Before conducting any spatial analysis, it was necessary to preprocess the data to ensure compatibility between the datasets. The following steps were performed:

### Wildfire Data: 
After converting the KMZ to shapefile, I ensured that the fire incident points were correctly georeferenced and transformed to match the BC boundary shapefile's CRS.

### Climate Data: 
The temperature data was aggregated to daily observations and merged with the station metadata to create a spatial dataset. This dataset was then cleaned for missing or inconsistent values to ensure accuracy in the analysis.

### Visualizing Data Clustering: 
One of the key goals of this tutorial is to explore how poorly distributed climate data can lead to clustering issues. To illustrate this, scatterplots or density maps will be used to visualize the spatial distribution of wildfire incidents and climate data points. This visualization helps in understanding the challenges of data clustering and how it may affect the results of spatial regression and interpolation.

Here is an example of the initial setup for the climate data:
```{r Climate Data File, echo = TRUE, results= 'hide', warning= FALSE, message= FALSE}
# Initialize an empty data frame
empty_data <- data.frame(Native.ID = character(), TEMP = numeric(), 
                         Longitude = numeric(), Latitude = numeric(), 
                         stringsAsFactors = FALSE)

# Write to a CSV file
# Check if the file already exists to avoid overwriting
csv_file_name <- "BC_AVG_TEMP.csv"
if (!file.exists(csv_file_name)) {
  write.csv(empty_data, file = csv_file_name, row.names = FALSE)
}

```

By transforming and formatting the data in this way, we can ensure a consistent and accurate analysis, while also identifying the challenges posed by poorly distributed datasets when using spatial regression and interpolation techniques.

### Second, Aggregate Temperature Data
Read and Process Climate Files
Aggregate temperature data from hourly observations into daily, monthly, and seasonal averages.

```{r Aggregate Temperature Data, echo = FALSE, message = FALSE, warning = FALSE, results = "hide"}

# List all CSV files in the BCH folder
# The BCH folder contains data from the weather station network
csv_files <- list.files(path = "C:/Users/aabar/Desktop/university/Year5/Geog418_LAB/Lab4/BCH", 
                        pattern = "\\.csv$", 
                        full.names = TRUE)

# Display the list of files to confirm (do this by removing the "#" on the next line:)
#print(csv_files)

# Loop through each CSV file and perform calculations for daily, monthly, and seasonal temperatures
for (file in csv_files) {
  #--- Step 1: Cleaning Climate data ---#
  # Read each CSV file
  # Read the CSV file, skipping the first line (header = TRUE ensures column names are read correctly)
  hourly_data <- read.csv(file, skip = 1, header = TRUE)
  file_name <- file
  
  # Step 2: Adjust the date/time column
  # Convert the time column into a datetime format usable for calculations
  hourly_data$time <- lubridate::ymd_hms(hourly_data$time)
  # Check the class of the time column (debugging/verification)
  class(hourly_data$time)
  
  # Step 3: Clean the MAX_TEMP column
  # Convert MAX_TEMP to numeric and filter out missing values (NA)
  hourly_data$MAX_TEMP <- as.numeric(hourly_data$MAX_TEMP)
  hourly_data <- hourly_data %>%
    filter(!is.na(MAX_TEMP))
  
  # Remove extreme outliers: values above 80°C or below -40°C
  hourly_data <- hourly_data %>%
    filter(MAX_TEMP <= 80 & MAX_TEMP >= -40)
  
# Step 4: Calculate daily average temperature
# Group data by date and calculate the mean MAX_TEMP for each day
daily_avg_temp <- hourly_data %>%
  group_by(date = as.Date(time)) %>%
  summarize(daily_avg_temp = mean(MAX_TEMP, na.rm = TRUE))

# Display daily average temperatures (debugging/verification)
print(daily_avg_temp)

# Step 5: Calculate monthly average temperature
# Group data by year and month, then calculate the monthly mean MAX_TEMP
monthly_avg_temp <- hourly_data %>%
  group_by(year = year(time), month = month(time)) %>%
  summarize(monthly_avg_temp = mean(MAX_TEMP, na.rm = TRUE)) %>%
  ungroup() # Ensure data is not grouped for subsequent operations

# Display monthly average temperatures (debugging/verification)
print(monthly_avg_temp)

# Step 6: Calculate seasonal average temperature (May to October)
# First, filter data for the months from May to October (Fire Season)
average_temp_may_october <- hourly_data %>%
  filter(month(time) >= 5 & month(time) <= 10) %>%
  summarize(TEMP = mean(MAX_TEMP, na.rm = TRUE))  # Replace 'temperature' with your column name

# Display the average temperature for May to October
print(average_temp_may_october)

# --- Part 2: Prepare climate data for merging ---

  
  # Extract the filename (station ID) from the file path
  file_name <- basename(file_name)                # Get the file name with extension
  file_name_no_ext <- sub("\\.[^.]*$", "", file_name)  # Remove the file extension
  
  # Display the weather station ID (debugging/verification)
  print(file_name_no_ext)
  
  # Step 7: Read the existing climate data file
  # Load the CSV file containing previously stored climate data
  file_path <- csv_file_name
  data <- read.csv(file_path)
  
  # Display the original data (debugging/verification)
  cat("Original Data:\n")
  print(head(data))
  
  # Step 8: Round seasonal average temperature
  # Round the seasonal average temperature to two decimal places
  Roundedtemp <- round(average_temp_may_october, 2)
  
  # Ensure the Native.ID column in the data is of type character
  data$Native.ID <- as.character(data$Native.ID)
  
  # --- Part 3: Append new rows to the climate data ---
  
  # Create a new row with the weather station ID and seasonal average temperature
  new_values <- data.frame(
    Native.ID = file_name_no_ext, 
    TEMP = Roundedtemp, 
    stringsAsFactors = FALSE
  )
  
  # Append the new row to the existing data
  data <- bind_rows(data, new_values)
  print(head(data))
  
  # Display the updated data for verification (optional debugging)
if (interactive()) {
  cat("Updated Data:\n")
  print(head(data))
}
  
  # Step 9: Save the updated data back to the CSV file
  output_file_path <- csv_file_name
  write.csv(data, file = output_file_path, row.names = FALSE)
}


```

### Third, Merge Climate and Metadata
Combine temperature data with station metadata for spatial analysis.

```{r Merge Climate and Metadata, echo = FALSE, message = FALSE}
# Read metadata and climate data
metadata <- read.csv("C:/Users/aabar/Desktop/university/Year5/Geog418_LAB/Lab4/station-metadata-by-history.csv")
climate_data <- read.csv("BC_AVG_TEMP.csv")

# Merge datasets on station ID
merged_data <- merge(metadata, climate_data, by = "Native.ID")

# Clean merged dataset
merged_data <- merged_data %>%
    filter(TEMP <= 100) %>%                                  # Filter out unrealistic temperature values
  rename(Longitude = Longitude.x, Latitude = Latitude.x) %>% # Rename columns for clarity
  select(-ends_with(".y")) %>%                               # Remove redundant columns (with .y suffix)
  na.omit()                                                  # Remove rows with NA values


# Save cleaned data to CSV
write.csv(merged_data, file = "ClimateData.csv", row.names = FALSE)
```

## Results and Visualization
Include analyses such as mapping temperature patterns or assessing spatial autocorrelation here.

