# Extract, Transform, and Load Data using Microsoft Azure Synapse Analytics
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/Azure%20Synapse%20Analytics%20Architecture.png?raw=true)

## Author
Alex Navarro

## 🔗 Contact Information
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/alexnavarro2/)
[![Email](https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white)](https://mail.google.com/mail/u/0/#inbox?compose=GTvVlcSBpRjxKKJtxTLNxwpsKvpfbRSRnRLcTQRMZLcKCNfrJjXfcNNKPmstkbHJpzHGNZnHvhCph)

## Project Objective:
Demonstrate how to clean and prep data using Microsoft Azure Synapse Analytics. Then visualize the output using Microsoft Power BI.

## Import Data into Azure Data Lake:
We will use "Azure Data Explorer" to import our data:

![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/Azure%20Storage%20Explorer.png?raw=true)

Step 1: Create Blob Container:
* Open Azure Data Explorer
* Go to your Azure Subscription, Storage Accounts, synapse ADLS Gen2
* Then create a new Blob Container:
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/Create%20Blob%20Container.png?raw=true)
* I called my new Blob Container "nyc-tax-data":
* Click on "Upload" then select the file path for the data you want to upload:
* You will see the upload progress at the bottom of the screen
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/Upload%20Files%20to%20Blob%20Container.png?raw=true)

## Exploratory Data Analysis (EDA):
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/EDA%207.png?raw=true)

Next, will open up Azure Synapse Analytics and perform some queries for our EDA process:
* Go to Data, Linked Tab
* Create a new SQL script for the "taxi_zone.csv" file:
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/EDA%201.png?raw=true)
* Execute the following query using the "Top 100 Rows" feature to display the first 100 rows of data:
```
-- This is auto-generated code
SELECT
    -- select top 100 rows:
    TOP 100 *
-- execute from clause:
FROM
    OPENROWSET(
        -- set BULK to file path:
        BULK 'https://synpasecoursejayhawkdl.dfs.core.windows.net/nyc-taxi-data/raw/taxi_zone.csv',
        -- set format of file:
        FORMAT = 'CSV',
        -- set parser_version to 2.0 for performance:
        PARSER_VERSION = '2.0',
        -- set header_row function to true:
        HEADER_ROW = TRUE
    -- alias as result:
    ) AS [result]
```
* Lastly, I'm going to create a folder and sub-folder to store my query in and then publish the changes:
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/EDA%203.png?raw=true)

### Check the data types of the dataset:
We can use the built in stored-procedure called "sp_describe_first_result_set" to see the data types of the data:
```
 -- examine the data types for the columns:
 EXEC sp_describe_first_result_set N'
-- This is auto-generated code
SELECT
    -- select top 100 rows:
    TOP 100 *
-- execute from clause:
FROM
    OPENROWSET(
        -- set BULK to file path:
        BULK ''https://synpasecoursejayhawkdl.dfs.core.windows.net/nyc-taxi-data/raw/taxi_zone.csv'',
        -- set format of file:
        FORMAT = ''CSV'',
        -- set parser_version to 2.0 for performance:
        PARSER_VERSION = ''2.0'',
        -- set header_row function to true:
        HEADER_ROW = TRUE
    -- alias as result:
    ) AS [result]'
```
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/EDA%204.png?raw=true)

However, in terms of creating cost efficient queries in Azure Synapse Analytics: lets check what the max length is for our columns to confirm if we need Azure Synapse Analytics reading our columns with the given data types:
```
- check max length for columns:
SELECT
    MAX(LEN(LocationId)) AS len_LocationId,
    MAX(LEN(Borough)) AS len_Borough,
    MAX(LEN(Zone)) AS len_zone,
    MAX(LEN(service_zone)) as len_service_zone
-- insert from clause:
FROM
    OPENROWSET(
        -- set BULK to file path:
        BULK 'https://synpasecoursejayhawkdl.dfs.core.windows.net/nyc-taxi-data/raw/taxi_zone.csv',
        -- set format of file:
        FORMAT = 'CSV',
        -- set parser_version to 2.0 for performance:
        PARSER_VERSION = '2.0',
        -- set header_row function to true:
        HEADER_ROW = TRUE,
        -- set fieldterminator:
        FIELDTERMINATOR = ',',
        -- set rowterminator:
        ROWTERMINATOR = '\n'
    -- alias as result:
    ) AS [result]
```
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/EDA%205.png?raw=true)
* As we can see, the length of the data types is actually a lot smaller then the default data types Synapse gave us.
* Good practice is to check the data type length before having Azure Synapse Analytics give the default data types.
  - Serverless SQL pool charges based on the amount of data processed. We don't want Synapse giving us larger data types then needed since we would be charged more then what we should be.
  - The data types also affect the query performance, we want to make sure that they're efficient data types to optimize query performance.
* Lets run the query again but this time given the explicit data types to run:
```
-- examine the data types for the columns:
EXEC sp_describe_first_result_set N'
SELECT *
-- insert from clause:
FROM
    OPENROWSET(
        -- set BULK to file path:
        BULK ''https://synpasecoursejayhawkdl.dfs.core.windows.net/nyc-taxi-data/raw/taxi_zone.csv'',
        -- set format of file:
        FORMAT = ''CSV'',
        -- set parser_version to 2.0 for performance:
        PARSER_VERSION = ''2.0'',
        -- set header_row function to true:
        HEADER_ROW = TRUE,
        -- set fieldterminator:
        FIELDTERMINATOR = '','',
        -- set rowterminator:
        ROWTERMINATOR = ''\n''
    )
    -- execute with clause:
    WITH (
        LocationID SMALLINT,
        Borough VARCHAR(15),
        Zone VARCHAR(50),
        service_zone VARCHAR(15)
    ) AS [result]' -- alias as result:
```
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/EDA%206.png?raw=true)

We can see that Azure Synapse Analytics is now reading in the correct data types when we explicity state our data types for the above columns

## Specify Data Collation:
I need to create a database that will read data in as UTF8 for all columns if applicable: Otherwise the data could come in wrong:
```
-- create database:
CREATE DATABASE nyc_taxi_discovery;

-- switch to database:
USE nyc_taxi_discovery;

-- switch to database:
USE nyc_taxi_discovery;

-- alter database:
ALTER DATABASE nyc_taxi_discovery COLLATE Latin1_General_100_CI_AI_SC_UTF8;
```

## Data Transformation:
![ScreenShot](https://global-uploads.webflow.com/634fa785d369cb60d80b6dd1/637f242f02ba099898c68400_Data-Transform.jpg)