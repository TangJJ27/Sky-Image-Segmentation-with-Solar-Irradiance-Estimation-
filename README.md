# Data Preparation
### Raw Data Files (Irradiance(CMP22),Zenith, Azimuth)
Manually select time interval of the raw data required at [NREL raw data](https://midcdmz.nrel.gov/apps/daily.pl?site=BMS&start=20200101&yr=2023&mo=11&dy=2).

- Select **Global CMP22 (vent/corr.), Zenith, Azimuth**
- Choose **Selected 1-Min Data (ASCII Text)**, then **Submit**

### Raw Sky Image
Images are downloaded from
[NREL raw image](https://midcdmz.nrel.gov/apps/imagergallery.pl?SRRLASI)  and processed automatically in the program.

---
# sky_image_ghi_estimation.ipynb
The .ipynb file is divided into different sections:
## Import Measured Data 
contains functions that import raw data into the program:
- ***import_data***: read raw data into the program
- ***get_data***: sliced out raw data day by day

## Image download 
contains functions for download image and creating and deleting folder:
- ***extract_img***: Download images from NREL
- ***create_folder***: Create folder for saving files and images

## Image segmentation 
### Radius Function
contains function for finding the radius of the ROI and extract out the ROI
- ***mask_circular_roi***: find the radius and mask out ROI

### Sun Position Function
contains functions for calculating and segmenting the sun postion in image
- ***segment_sun_position***: locate sun position using image segmentation 
- ***img_sun_location***: return calculated and segmented sun position in image
- ***plot_sun***: plot sky images with obtained sun position (for observing result)

### Image Luminance Function
contains functions for obtaining the average luminance of sky images
- ***whole_luminance***: return average whole image full pixel luminance and luminance image
- ***random_sampling_whole***: return average whole image random pixel luminance and sampling points coordinates
- ***whole_luminance***: return average cropped image full pixel luminance and cropped luminance image
- ***random_sampling_crop***: return average cropped image random pixel luminance and sampling points coordinates

### Functions Results
- testing and showing the results of each of the above functions

### Main Function
Main segmentation function directly applied to ***measure_data*** dataframe.

1. ***radius,directory_path and NO_SUN*** is made global to update with corresponding variable in main program.
2. ***zenith,azimuth,date*** is obtained from ***measure_data*** while the ***image*** is read into the program using *Image path*.
3.  ***mask_circuilar_roi*** is applied to the image to find the radius of the ROI and mask out the ROI with black pixels outside the ROI. The function return the ***masked image and ROI coordinates***.
4. ***img_sun_location*** is applied to the image to find the calculated (***cal_sun_x,cal_sun_y***) and segmented (***sun_x,sun_y***) sun position . If segmented sun is not found in the image, ***NO_SUN*** will be set.
5. ***crop_sun_img*** is applied to the image to crop out the sun in the image based on the sun position found. If sun is found in the image, cropping based on segmented position, else based on calculated position.
6. ***whole_luminance*** is applied to the full image and return average whole image full pixel luminance (***wLr***) and luminance image. 
7. ***random_sampling_whole*** is applied to the full image and return average whole image and random 1/100 of the ROI pixel luminance (***wLr_sam***). 
8. ***whole_luminance*** is applied to the cropped image and return average cropped image full pixel luminance (***cLr***) and cropped luminance image. 
9. ***random_sampling_crop*** is applied to the cropped image and return average cropped image and random 1/100 of the total cropped image pixel luminance (***cLr_sam***).
10. Sun position (***sun_x,sun_y,cal_sun_x, cal_sun_y***) and average luminance (***wLr, wLr_sam, cLr, cLr_sam***) are returned as series to the main program.

## Main Program
### Initialization
- set **time range** (start_date & end_date, day_interval, daytime start_hour,min & end_hour,min)
- set **path directory**  
- set ***DOWN_PROCESS*** flag to **True** = download and process image, **False** = download only
- the **manually dowloaded raw data** for a large time range will be imported into a dataframe named ***data***.
>> Remember to change path directory before running!

### Process
1. A **monthly folder** named with *yearmonth* will be created for the current month to store daily result of the month.

2. **If daily result is not exists** in the **monthly folder**, a **daily folder** named with *yearmonthdate* will be created to store downloaded images.

3. Sky images for **every 10 minutes** from *start_hour* until *end_hour* are downloaded using *extract_img* and saved in the daily folder.

#### If ***DOWN_PROCESS*** is True:
4. The raw dataof the day will be sliced out from ***data*** and merge downloaded image path with corresponding raw data. The resulting dataframe is named ***measure_data***. Raw data without image will be eliminated. 

5. The main function ***image_segment*** is applied to ***measure_data*** and return calculated & segmented sun position and average luminance (whole image full pixels, whole image sampled pixels, cropped image full pixels & cropped image sampled pixels). 

6. The result dataframe is named ***day_result*** and is saved as **{yearmonthdate}.csv** to the **monthly folder**.

7. The **daily folder** is then be deleted and continue with image download of the next day.


## Generate Estimation Model
- analyse results and generate estimation model using the results csv files
1. Read the generated processed csv files using ***read_result*** into a result dataframe.
2. Generate estimation models by inputing result dataframe and the order of the model into ***generate_models***. The function returns 4 lists of ***w_models,w_sampling_models, c_models, c_sampling_models*** with each consists of specified order of model.
3. Plot the models using ***plot_models*** and observe the behaviour of the models. 

## ADJR2 & RMSE
### R-squared
- apply ***adjR2*** using *average luminance* as the independent variable, x and the *measured GHI* as the dependent variable, y to obtain the adjusted R-squared value of each model 

### RMSE
- ***clear_sky*** to obtain theoretical GHI
- ***evaluate_model*** to estimate GHI using the models and obtain the RMSE of the estimated and measured GHI

## Time Series Plot
- using ***time_plot*** to plot time series plot of clear sky GHI, measured GHI and estimated GHI of the selected day.