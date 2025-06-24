# Pudong feature store



## Import raw data

Use this following command in your terminal to import the raw data

```bash
gcloud storage cp -r gs://cn-smart-leak/data/raw data/raw
```

Load the .inp file, waiting for the final code that takes the .net file
```bash
gcloud storage cp -r gs://cn-smart-leak/data/tidy/network/230726-15min.inp data/tidy/network
```

## Codes explanation

### 1. [app/src/epanet.py](app/src/epanet.py)

This file allows to store a json file on cloud storage which indicates for each pipe of the network, all the pipes which are at a distance less than or equal to 1000 meters, as well as the relative distance between these two pipes. It is important to store this information because it is used in the API to calculate metrics.

### 2. [app/src/pipes_coordinates.py](app/src/pipes_coordinates.py)

This file stores in cloud storage a json file that indicates the coordinates of all the nodes of the network. This information is used for the front end, to display the map.

### 3. [app/src/pipes_nodes.py](app/src/pipes_nodes.py)

This file stores a json file that, for each pipe, refers to its two nodes. It is used to calculate metrics.

### 4. [app/src/pressure.py](app/src/pressure.py)

The important process of this function are the following ones :

- Reads and cleans the data: The function reads the pressure data from a CSV file, converts the 'collect_time' column to datetime format, removes duplicates, and pivots the data to a wide format. The data is then resampled to a 10-minute frequency.

- Selects the last 20 days of data: The function selects the last 20 days of data from the specified date and reshapes the data.

- Fills missing values: The function fills missing values in the data with the median of the respective row.

- Normalizes the data: The function normalizes the data by subtracting the mean and dividing by the standard deviation.

- Prepares the data for tsfresh: The function prepares the data for tsfresh by creating an 'id' column and reshaping the data to a long format.

- Extracts features with tsfresh: The function uses tsfresh to extract features from the time series data.

- Cleans the features: The function cleans the extracted features by removing columns with missing values or only one unique value.

- Renames the columns: The function renames the columns by replacing non-alphanumeric characters with underscores.

- Returns the processed features: The function returns the processed features as a pandas DataFrame.

This code returns a table for a specified date. The table uses the name of the measurement point as the key and a list of features extracted with tsfresh from the time series of the pressure measured in the 20 days preceding the date as the data. This data is then joined with the pipe information using the measurement point name.




### 5. [app/src/pipes_infos.py](app/src/pipes_infos.py)

We use the file [app/src/data/translation_pipes_infos.json](app/src/data/translation_pipes_infos.json) to translate materials names. It can be updated.


### 6. [app/src/outlet_pressure_waterplant.py](app/src/outlet_pressure_waterplant.py)

The first function 'transform_outlet_pressure_waterplant' returns the output pressure of the plants over time.
The second function 'transform_outlet_pressure_waterplant' returns the output pressure of the plants over time.

### 7. [app/src/pressure_difference.py](app/src/pressure_difference.py)

For each date, for all combinations of two measurement points, the pressure difference over the last 20 days is calculated.

### 8. [app/src/weather.py](app/src/weather.py)

This script processes and cleans a set of weather data files. It reads CSV files from a specified directory, concatenates them into a single DataFrame, and performs various transformations including creating a unified date column, calculating a rolling 20-day mean for each column, and filtering rows based on specific criteria (keep the 15th and last day for each month). It also removes columns with high correlation, missing values, or single unique values. The column names are cleaned to replace non-alphanumeric characters with underscores. The final cleaned DataFrame is then returned.

### 9. [app/src/pressure_2.py](app/src/pressure_2.py)

The important process of this function are the following ones :

Same as pressure.py with the last format of pressure given.

- Selects the last 20 days of data: The function selects the last 20 days of data from the specified date and reshapes the data.

- Fills missing values: The function fills missing values in the data with the median of the respective row.

- Normalizes the data: The function normalizes the data by subtracting the mean and dividing by the standard deviation.

- Prepares the data for tsfresh: The function prepares the data for tsfresh by creating an 'id' column and reshaping the data to a long format.

- Extracts features with tsfresh: The function uses tsfresh to extract features from the time series data.

- Cleans the features: The function cleans the extracted features by removing columns with missing values or only one unique value.

- Renames the columns: The function renames the columns by replacing non-alphanumeric characters with underscores.

- Returns the processed features: The function returns the processed features as a pandas DataFrame.

This code returns a table for a specified date. The table uses the name of the measurement point as the key and a list of features extracted with tsfresh from the time series of the pressure measured in the 20 days preceding the date as the data. This data is then joined with the pipe information using the measurement point name.


### 10. [app/create_original_dataset.ipynb](app/create_original_dataset.ipynb)

This notebook was used to create the final dataset in BigQuery.
It used the different elements of the other scripts. From now on, we will call extraction date the 15th day and the last day of each month, the dates when data is gathered.
There are three tables related to this dataset:
- `cn-ops-spdigital-datasci-dev.pudong_leak_detection.final_dataset_pipes_infos`: this table contains all the information related to each pipe (Length, diameter, ...), it also contains calculated characteristics like age, number of past leaks. Finally, the 'leaks' column is the y-label of the dataset. For each key (pipe, date), it contains the information if a leak has occurred.
- `cn-ops-spdigital-datasci-dev.pudong_leak_detection.final_dataset_pressure_ts`: this table contains the features extracted from the pressure times series. For each extraction date, we extract the features over the 20 past days.
- `cn-ops-spdigital-datasci-dev.pudong_leak_detection.final_dataset_weather`: this table contains different weather features. For each extraction date, we calculate the min/max/avg/sum of the feature over the last 20 days depending of the feature.

Join :

```SQL
CREATE TABLE cn-ops-spdigital-datasci-dev.pudong_leak_detection.final_dataset AS
SELECT *
FROM `cn-ops-spdigital-datasci-dev.pudong_leak_detection.final_dataset_pipes_infos` AS table1
LEFT JOIN `cn-ops-spdigital-datasci-dev.pudong_leak_detection.final_dataset_pressure_ts` AS table2
ON table1.date = table2.datetime AND table1.nearest_tag = table2.tag
LEFT JOIN `cn-ops-spdigital-datasci-dev.pudong_leak_detection.final_dataset_weather` AS table3
ON table1.date = table3.collection_date;
```

Create table for the API (we reduce the size of the dataset to reduce the time of training, could be change later):

```SQL
CREATE TABLE cn-ops-spdigital-datasci-dev.pudong_leak_detection.final_dataset_API AS
SELECT * FROM cn-ops-spdigital-datasci-dev.pudong_leak_detection.final_dataset
WHERE date>='2020-01-01'
```