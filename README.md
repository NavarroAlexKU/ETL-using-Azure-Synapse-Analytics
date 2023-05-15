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
I need to create a database that will read the data in as UTF8 for all columns if applicable: Otherwise the data could come in wrong:
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

## Create External Data Source:
We want to create an external data source for this allows us to switch between different enviornments without having to change out our connection string each time:
```
-- create external data source: raw
CREATE EXTERNAL DATA SOURCE nyc_taxi_data_raw
WITH (
    LOCATION = 'abfss://nyc-taxi-data@synpasecoursejayhawkdl.dfs.core.windows.net/raw' -- point to container level:
);
```

You can confirm that the External Data Source was created by checking in the Data tab:

![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/Create%20External%20Database.png?raw=true)

## Plot and Join data:
Next, maybe we want to plot the data in a aggregated view:
* first I'll join the taxi_zone and borough data:
```
------ join data ---------
SELECT
    taxi_zone.borough,
    COUNT(1) AS number_of_trips
FROM
    OPENROWSET(
        BULK 'trip_data_green_parquet/year=2020/month=01/',
        FORMAT = 'PARQUET',
        DATA_SOURCE = 'nyc_taxi_data_raw'
    ) AS trip_data
JOIN
    OPENROWSET(
        -- set BULK to file path:
        BULK 'abfss://nyc-taxi-data@synpasecoursejayhawkdl.dfs.core.windows.net/raw/taxi_zone.csv',
        -- set format of file:
        FORMAT = 'CSV',
        -- set parser_version to 2.0 for performance:
        PARSER_VERSION = '2.0',
        --set FIRSTROW parameter:
        FIRSTROW = 2
    )
    WITH (
        location_id SMALLINT 1,
        borough VARCHAR(15) 2,
        zone VARCHAR(50) 3,
        service_zone VARCHAR(15) 4
    ) AS taxi_zone
ON trip_data.PULocationID = taxi_zone.location_id
GROUP BY taxi_zone.borough
-- order by:
ORDER BY number_of_trips ASC;
```
* Then we can use the "Chart" tab to visual the data:

![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/EDA%208.png?raw=true)

![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/EDA%209.png?raw=true)

## Create External Database Objects:
![ScreenShot](https://github.com/NavarroAlexKU/ETL-using-Azure-Synapse-Analytics/blob/main/External%20Table%201.png?raw=true)

You can use the OPENROWSET function in SQL queries that run in the default master database of the built-in serverless SQL pool to explore data in the data lake. However, sometimes you may want to create a custom database that contains some objects that make it easier to work with external data in the data lake that you need to query frequently.

### Create the Database:

```
-- use master database:
USE master
GO

-- create database:
CREATE DATABASE nyc_taxi_ldw
GO

-- alter database:
ALTER DATABASE nyc_taxi_ldw COLLATE Latin1_General_100_BIN2_UTF8
GO

-- use nyc_taxi_ldw:
USE nyc_taxi_ldw
GO

-- create bronze schema:
CREATE SCHEMA bronze
GO

-- create silver schema:
CREATE SCHEMA silver
GO

-- create gold schema:
CREATE SCHEMA gold
GO
```

### Create the External Data Source:

You can use the OPENROWSET function with a BULK path to query file data from your own database, just as you can in the master database; but if you plan to query data in the same location frequently, it's more efficient to define an external data source that references that location.

```
-- switch databases:
USE nyc_taxi_ldw;

-- check if data source already exists:
IF NOT EXISTS (
    SELECT *
    FROM sys.external_data_sources
    WHERE name = 'nyc_taxi_src'
)
-- create external data source:
CREATE EXTERNAL DATA SOURCE nyc_taxi_src
WITH (
    LOCATION = 'https://synpasecoursejayhawkdl.dfs.core.windows.net/nyc-taxi-data'
);
```

### Create an external file format:

While an external data source simplifies the code needed to access files with the OPENROWSET function, you still need to provide format details for the file being access; which may include multiple settings for delimited text files

```
USE nyc_taxi_ldw;

-- Create an external file format for DELIMITED (CSV/TSV) files.
CREATE EXTERNAL FILE FORMAT csv_file_format
WITH (
    FORMAT_TYPE = DELIMITEDTEXT,
    FORMAT_OPTIONS (
        FIELD_TERMINATOR = ',',
        STRING_DELIMITER = '"',
        FIRST_ROW = 2,
        USE_TYPE_DEFAULT = FALSE,
        ENCODING = 'UTF8',
        PARSER_VERSION = '2.0')
    );

--- create file format tsv_file_format for parser version 2.0:
IF NOT EXISTS (
  SELECT 
    * 
  FROM 
    sys.external_file_formats 
  WHERE 
    name = 'tsv_file_format'
) CREATE EXTERNAL FILE FORMAT tsv_file_format WITH (
  FORMAT_TYPE = DELIMITEDTEXT, 
  FORMAT_OPTIONS (
    FIELD_TERMINATOR = '\t', STRING_DELIMITER = '"', 
    FIRST_ROW = 2, USE_TYPE_DEFAULT = FALSE, 
    ENCODING = 'UTF8', PARSER_VERSION = '2.0'
  )
);

--- create file format tsv_file_format for parser version 1.0:
IF NOT EXISTS (
  SELECT 
    * 
  FROM 
    sys.external_file_formats 
  WHERE 
    name = 'tsv_file_format_pv1'
) CREATE EXTERNAL FILE FORMAT tsv_file_format_pv1 WITH (
  FORMAT_TYPE = DELIMITEDTEXT, 
  FORMAT_OPTIONS (
    FIELD_TERMINATOR = '\t', STRING_DELIMITER = '"', 
    FIRST_ROW = 2, USE_TYPE_DEFAULT = FALSE, 
    ENCODING = 'UTF8', PARSER_VERSION = '1.0'
  )
);

--- create parquet file format:
IF NOT EXISTS (
  SELECT 
    * 
  FROM 
    sys.external_file_formats 
  WHERE 
    name = 'parquet_file_format'
) CREATE EXTERNAL FILE FORMAT parquet_file_format WITH (
  FORMAT_TYPE = PARQUET, DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
);

----- create external file format for delta_file_format:
IF NOT EXISTS (
  SELECT *
  FROM sys.external_file_formats
  WHERE name = 'delta_file_format'
)
  CREATE EXTERNAL FILE FORMAT delta_file_format
  WITH (
    FORMAT_TYPE = DELTA,
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
  )
```

### Create External Table Bronze Display Layer:

When you need to perform a lot of analysis or reporting from files in the data lake, using the OPENROWSET function can result in complex code that includes data sources and file paths. To simplify access to the data, you can encapsulate the files in an external table; which users and reporting applications can query using a standard SQL SELECT statement just like any other database table. To create an external table, use the CREATE EXTERNAL TABLE statement, specifying the column schema as for a standard table, and including a WITH clause specifying the external data source, relative path, and external file format for your data.

```
---- create external table: CSV File Type
USE nyc_taxi_ldw;
-- check if table exist:
IF NOT EXISTS (
  SELECT 
    * 
  FROM 
    sys.external_file_formats 
  WHERE 
    name = 'csv_file_format'
) -- Create an external file format for DELIMITED (CSV/TSV) files.
CREATE EXTERNAL FILE FORMAT csv_file_format WITH (
  FORMAT_TYPE = DELIMITEDTEXT, 
  FORMAT_OPTIONS (
    FIELD_TERMINATOR = ',', STRING_DELIMITER = '"', 
    FIRST_ROW = 2, USE_TYPE_DEFAULT = FALSE, 
    ENCODING = 'UTF8', PARSER_VERSION = '2.0'
  )
);


-- check if table exist:
IF NOT EXISTS (
  SELECT 
    * 
  FROM 
    sys.external_file_formats 
  WHERE 
    name = 'csv_file_format_pv1'
) -- Create an external file format for DELIMITED (CSV/TSV) files.
CREATE EXTERNAL FILE FORMAT csv_file_format_pv1 WITH (
  FORMAT_TYPE = DELIMITEDTEXT, 
  FORMAT_OPTIONS (
    FIELD_TERMINATOR = ',', STRING_DELIMITER = '"', 
    FIRST_ROW = 2, USE_TYPE_DEFAULT = FALSE, 
    ENCODING = 'UTF8', PARSER_VERSION = '1.0'
  )
);


USE nyc_taxi_ldw;
-- check if table exist:
IF OBJECT_ID('bronze.taxi_zone') IS NOT NULL 
DROP 
  EXTERNAL TABLE bronze.taxi_zone -- create taxi_zone table:
  CREATE EXTERNAL TABLE bronze.taxi_zone (
    -- insert columns and data types:
    location_id SMALLINT, 
    borough VARCHAR(15), 
    zone VARCHAR(50), 
    service_zone VARCHAR(15)
  ) -- execute with clause:
  WITH (
    LOCATION = 'raw/taxi_zone.csv', DATA_SOURCE = nyc_taxi_src, 
    FILE_FORMAT = csv_file_format_pv1, 
    REJECT_VALUE = 10, REJECTED_ROW_LOCATION = 'rejections/taxi_zone'
  );

---- create Calendar external table: CSV File Type
IF OBJECT_ID('bronze.calendar') IS NOT NULL 
DROP 
  EXTERNAL TABLE bronze.calendar;
-- create EXTERNAL TABLE:
CREATE EXTERNAL TABLE bronze.calendar (
  date_key INT, 
  date DATE, 
  year SMALLINT, 
  month TINYINT, 
  day TINYINT, 
  day_name VARCHAR(10), 
  day_of_year SMALLINT, 
  week_of_month TINYINT, 
  week_of_year TINYINT, 
  month_name VARCHAR(10), 
  year_month INT, 
  year_week INT
) WITH (
  LOCATION = 'raw/calendar.csv', DATA_SOURCE = nyc_taxi_src, 
  FILE_FORMAT = csv_file_format_pv1, 
  REJECT_VALUE = 10, REJECTED_ROW_LOCATION = 'rejections/calendar'
);

---- create Vendor external table: CSV File Type
------ create vendor table
USE nyc_taxi_ldw;
-- check if table exist:
IF OBJECT_ID('bronze.vendor') IS NOT NULL 
DROP 
  EXTERNAL TABLE bronze.vendor -- create external table: vendor
  CREATE EXTERNAL TABLE bronze.vendor (
    -- insert columns and data types:
    vendor_id TINYINT, 
    vendor_name VARCHAR(50)
  ) -- execute with clause:
  WITH (
    LOCATION = 'raw/vendor.csv', DATA_SOURCE = nyc_taxi_src, 
    FILE_FORMAT = csv_file_format_pv1, 
    REJECT_VALUE = 10, REJECTED_ROW_LOCATION = 'rejections/vendor'
  );
  
USE nyc_taxi_ldw;
----- create trip type external table:
IF OBJECT_ID ('bronze.trip_type') IS NOT NULL 
DROP 
  EXTERNAL TABLE bronze.trip_type;
CREATE EXTERNAL TABLE bronze.trip_type (
  trip_type TINYINT, 
  trip_type_desc VARCHAR(50)
) WITH (
  LOCATION = 'raw/trip_type.tsv', DATA_SOURCE = nyc_taxi_src, 
  FILE_FORMAT = tsv_file_format_pv1, 
  REJECT_VALUE = 10, REJECTED_ROW_LOCATION = 'rejections/trip_type'
);
SELECT 
  * 
FROM 
  bronze.trip_type;


-- drop external table if exist:
IF OBJECT_ID('bronze.trip_data_green_csv') IS NOT NULL 
DROP 
  EXTERNAL TABLE bronze.trip_data_green_csv;
--- create external table trip_data_green_csv:
CREATE EXTERNAL TABLE bronze.trip_data_green_csv (
  VendorID INT, 
  lpep_pickup_datetime datetime2(7), 
  lpep_dropoff_datetime datetime2(7), 
  store_and_fwd_flag CHAR(1), 
  RatecodeID INT, 
  PULocationID INT, 
  DOLocationID INT, 
  passenger_count INT, 
  trip_distance FLOAT, 
  extra FLOAT, 
  mta_tax FLOAT, 
  tolls_amount FLOAT, 
  ehail_fee INT, 
  improvement_surcharge FLOAT, 
  total_amount FLOAT, 
  payment_type INT, 
  trip_type INT, 
  congestion_surcharge FLOAT
) WITH (
  LOCATION = 'raw/trip_data_green_csv/**', 
  DATA_SOURCE = nyc_taxi_src, FILE_FORMAT = csv_file_format
);

----- create external table for parquet: -------
-- drop external table if exist:
IF OBJECT_ID(
  'bronze.trip_data_green_parquet'
) IS NOT NULL 
DROP 
  EXTERNAL TABLE bronze.trip_data_green_parquet;
--- create external table trip_data_green_parquet:
CREATE EXTERNAL TABLE bronze.trip_data_green_parquet (
  VendorID INT, 
  lpep_pickup_datetime datetime2(7), 
  lpep_dropoff_datetime datetime2(7), 
  store_and_fwd_flag CHAR(1), 
  RatecodeID INT, 
  PULocationID INT, 
  DOLocationID INT, 
  passenger_count INT, 
  trip_distance FLOAT, 
  extra FLOAT, 
  mta_tax FLOAT, 
  tolls_amount FLOAT, 
  ehail_fee INT, 
  improvement_surcharge FLOAT, 
  total_amount FLOAT, 
  payment_type INT, 
  trip_type INT, 
  congestion_surcharge FLOAT
) WITH (
  LOCATION = 'raw/trip_data_green_parquet/**', 
  DATA_SOURCE = nyc_taxi_src, FILE_FORMAT = parquet_file_format
);

----- create external table for delta: -------
-- drop external table if exist:
IF OBJECT_ID('bronze.trip_data_green_delta') IS NOT NULL 
DROP 
  EXTERNAL TABLE bronze.trip_data_green_delta;
--- create external table trip_data_green_delta:
CREATE EXTERNAL TABLE bronze.trip_data_green_delta (
  VendorID INT, 
  lpep_pickup_datetime datetime2(7), 
  lpep_dropoff_datetime datetime2(7), 
  store_and_fwd_flag CHAR(1), 
  RatecodeID INT, 
  PULocationID INT, 
  DOLocationID INT, 
  passenger_count INT, 
  trip_distance FLOAT, 
  extra FLOAT, 
  mta_tax FLOAT, 
  tolls_amount FLOAT, 
  ehail_fee INT, 
  improvement_surcharge FLOAT, 
  total_amount FLOAT, 
  payment_type INT, 
  trip_type INT, 
  congestion_surcharge FLOAT
) WITH (
  LOCATION = 'raw/trip_data_green_delta/**', 
  DATA_SOURCE = nyc_taxi_src, FILE_FORMAT = delta_file_format
);

```

## Create Bronze Views:
```
USE nyc_taxi_ldw GO

-- drop view if exist:
DROP VIEW bronze.vw_rate_code
GO

-- create view:
CREATE VIEW bronze.vw_rate_code AS
  SELECT 
    rate_code_id, 
    rate_code -- from clause:
  FROM 
    OPENROWSET (
      BULK 'raw/rate_code.json', DATA_SOURCE = 'nyc_taxi_src', 
      FORMAT = 'CSV', FIELDTERMINATOR = '0x0b', FIELDQUOTE = '0x0b', 
      ROWTERMINATOR = '0x0b'
    ) WITH (
      jsonDoc NVARCHAR(MAX)
    ) AS rate_code CROSS APPLY OPENJSON(jsonDoc) WITH (
      rate_code_id TINYINT, 
      rate_code VARCHAR(20)
    )
GO

-- select first 10 rows of data:
SELECT TOP 10 * FROM bronze.vw_rate_code;


---- create view for payment type data source:
USE nyc_taxi_ldw GO

-- drop view if exist:
DROP VIEW IF EXISTS bronze.vw_payment_type
GO

-- create view:
CREATE VIEW bronze.vw_payment_type AS
-- select clause:
  SELECT
    payment_type,
    description
  -- from clause:  
  FROM 
    OPENROWSET (
      BULK 'raw/payment_type.json', DATA_SOURCE = 'nyc_taxi_src', 
      FORMAT = 'CSV', FIELDTERMINATOR = '0x0b', FIELDQUOTE = '0x0b', 
      ROWTERMINATOR = '0x0b'
    ) WITH (
      jsonDoc NVARCHAR(MAX)
    ) AS payment_type CROSS APPLY OPENJSON(jsonDoc) WITH (
      payment_type SMALLINT, 
      description VARCHAR(20) '$.payment_type_desc'
    )
GO

-- select first 10 rows of data:
SELECT TOP 10 * FROM bronze.vw_payment_type;
```

## Create External Tables for Silver Display Layer:
```
-- switch to correct database:
USE nyc_taxi_ldw
GO

--- drop if exist:
IF OBJECT_ID('silver.taxi_zone') IS NOT NULL
DROP EXTERNAL TABLE silver.taxi_zone
GO

-- create external table:
CREATE EXTERNAL TABLE silver.taxi_zone WITH (
  DATA_SOURCE = nyc_taxi_src,
  LOCATION = 'silver/taxi_zone', 
  FILE_FORMAT = parquet_file_format
) AS 
SELECT 
  * 
FROM 
  bronze.taxi_zone;


SELECT TOP 10 *
FROM silver.taxi_zone;
```

```
-- switch to correct database:
USE nyc_taxi_ldw
GO

--- drop if exist:
IF OBJECT_ID('silver.calendar') IS NOT NULL
DROP EXTERNAL TABLE silver.calendar
GO

-- create external table:
CREATE EXTERNAL TABLE silver.calendar WITH (
  DATA_SOURCE = nyc_taxi_src,
  LOCATION = 'silver/calendar', 
  FILE_FORMAT = parquet_file_format
) AS 
SELECT 
  * 
FROM 
  bronze.calendar;
```

```
-- switch to correct database:
USE nyc_taxi_ldw
GO

--- drop if exist:
IF OBJECT_ID('silver.trip_type') IS NOT NULL
DROP EXTERNAL TABLE silver.trip_type
GO

-- create external table:
CREATE EXTERNAL TABLE silver.trip_type WITH (
  DATA_SOURCE = nyc_taxi_src,
  LOCATION = 'silver/trip_type', 
  FILE_FORMAT = parquet_file_format
) AS 
SELECT 
  * 
FROM 
  bronze.trip_type;
```

```
-- switch to correct database:
USE nyc_taxi_ldw
GO

--- drop if exist:
IF OBJECT_ID('silver.vendor') IS NOT NULL
DROP EXTERNAL TABLE silver.vendor
GO

-- create external table:
CREATE EXTERNAL TABLE silver.vendor WITH (
  DATA_SOURCE = nyc_taxi_src,
  LOCATION = 'silver/vendor', 
  FILE_FORMAT = parquet_file_format
) AS 
SELECT 
  * 
FROM 
  bronze.vendor;
```

```
-- switch to correct database:
USE nyc_taxi_ldw GO --- drop if exist:
IF OBJECT_ID('silver.rate_code') IS NOT NULL 
DROP 
  EXTERNAL TABLE silver.rate_code GO -- create external table:
  CREATE EXTERNAL TABLE silver.rate_code WITH (
    DATA_SOURCE = nyc_taxi_src, LOCATION = 'silver/rate_code', 
    FILE_FORMAT = parquet_file_format
  ) AS 
SELECT 
  rate_code_id, 
  rate_code 
FROM 
  OPENROWSET (
    BULK 'raw/rate_code.json', DATA_SOURCE = 'nyc_taxi_src', 
    FORMAT = 'CSV', FIELDTERMINATOR = '0x0b', 
    FIELDQUOTE = '0x0b', ROWTERMINATOR = '0x0b'
  ) WITH (
    jsonDoc NVARCHAR(MAX)
  ) AS rate_code CROSS APPLY OPENJSON(jsonDoc) WITH (
    rate_code_id TINYINT, 
    rate_code VARCHAR(20)
  );
```

```
-- switch to correct database:
USE nyc_taxi_ldw GO --- drop if exist:
IF OBJECT_ID('silver.payment_type') IS NOT NULL 
DROP 
  EXTERNAL TABLE silver.payment_type GO -- create external table:
  CREATE EXTERNAL TABLE silver.payment_type WITH (
    DATA_SOURCE = nyc_taxi_src, LOCATION = 'silver/payment_type', 
    FILE_FORMAT = parquet_file_format
  ) AS 
SELECT
    payment_type,
    description
FROM 
  OPENROWSET (
    BULK 'raw/payment_type.json', DATA_SOURCE = 'nyc_taxi_src', 
    FORMAT = 'CSV', FIELDTERMINATOR = '0x0b', 
    FIELDQUOTE = '0x0b', ROWTERMINATOR = '0x0b'
  ) WITH (
    jsonDoc NVARCHAR(MAX)
  ) AS payment_type CROSS APPLY OPENJSON(jsonDoc) WITH (
    payment_type SMALLINT, 
    description VARCHAR(20) '$.payment_type_desc'
  );
```

```
-- SWITCH DATABASE:
USE nyc_taxi_ldw GO -- create view for trip_data_green:
DROP 
  VIEW IF EXISTS silver.vw_trip_data_green GO -- create view:
CREATE VIEW silver.vw_trip_data_green AS 
SELECT 
  result.filepath(1) AS year, 
  result.filepath(2) AS month, 
  result.* 
FROM 
  OPENROWSET(
    BULK 'silver/trip_data_green/year=*/month=*/*.parquet', 
    DATA_SOURCE = 'nyc_taxi_src', FORMAT = 'PARQUET'
  ) WITH (
    vendor_id INT, 
    lpep_pickup_datetime datetime2(7),
    lpep_dropoff_datetime datetime2(7), 
    store_and_fwd_flag CHAR(1), 
    rate_code_id INT, 
    pu_location_id INT, 
    do_location_id INT, 
    passenger_count INT, 
    trip_distance FLOAT, 
    fare_amount FLOAT, 
    extra FLOAT, 
    mta_tax FLOAT, 
    tip_amount FLOAT, 
    tolls_amount FLOAT, 
    ehail_fee INT, 
    improvement_surcharge FLOAT, 
    total_amount FLOAT, 
    payment_type INT,
    trip_type INT, 
    congestion_surcharge FLOAT
  ) AS [result]
GO
```





## Data Transformation:
![ScreenShot](https://global-uploads.webflow.com/634fa785d369cb60d80b6dd1/637f242f02ba099898c68400_Data-Transform.jpg)