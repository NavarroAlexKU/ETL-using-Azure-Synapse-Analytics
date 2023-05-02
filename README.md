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
