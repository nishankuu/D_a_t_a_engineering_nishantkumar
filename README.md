# Youtube Data ETL 

This dataset provides a daily record of YouTube's top trending videos, selected based on user interactions such as views, shares, comments, and likes rather than overall view count. It includes data from multiple regions—initially covering the US, GB, DE, CA, and FR, later expanded to include RU, MX, KR, JP, and IN.

Each region's data is stored separately and contains key video attributes like title, channel name, publish time, tags, views, likes, dislikes, description, and comment count. The dataset also features a category_id field, with category details available in an accompanying JSON file.

The data was collected using the YouTube API and has been structurally improved for better usability. For details on specific columns, refer to the column metadata.

Source - Kaggle (One Example File Uploaded)
Link Included for download - https://www.kaggle.com/datasets/datasnaek/youtube-new

# Data Architecture/Flow
<img width="1012" alt="Image" src="https://github.com/user-attachments/assets/0045bd69-60e4-47ac-84ec-6a20d576e32c" />



# YouTube Trending Videos Data Pipeline

o begin, I used the AWS CLI to transfer data to S3 buckets, organizing the files into separate folders for JSON and CSV data.

Data Upload Using AWS CLI

JSON Files: Stored in the raw_statistics_reference_data folder.
aws s3 cp . s3://nishant-dataengineering-youtube/youtube/raw_statistics_reference_data/ --recursive --exclude "*" --include "*.json"

CSV Files: Organized by region within the raw_statistics folder.

aws s3 cp FRvideos.csv s3://nishant-dataengineering-youtube/youtube/raw_statistics/region=fr/

aws s3 cp GBvideos.csv s3://nishant-dataengineering-youtube/youtube/raw_statistics/region=gb/

aws s3 cp INvideos.csv s3://nishant-dataengineering-youtube/youtube/raw_statistics/region=in/

aws s3 cp JPvideos.csv s3://nishant-dataengineering-youtube/youtube/raw_statistics/region=jp/

aws s3 cp KRvideos.csv s3://nishant-dataengineering-youtube/youtube/raw_statistics/region=kr/

aws s3 cp MXvideos.csv s3://nishant-dataengineering-youtube/youtube/raw_statistics/region=mx/

aws s3 cp RUvideos.csv s3://nishant-dataengineering-youtube/youtube/raw_statistics/region=ru/

aws s3 cp USvideos.csv s3://nishant-dataengineering-youtube/youtube/raw_statistics/region=us/

This structured data storage ensures easy access and efficient processing in subsequent steps of the project.


# Project Overview

This project focuses on building a data pipeline for analyzing YouTube's trending videos dataset using AWS Glue, Athena, and Quicksight. The dataset is processed in multiple stages, from raw JSON extraction to a structured and optimized format for querying and visualization.

The key components of this project include:

## AWS Glue Crawler for metadata extraction
## Amazon Athena for querying structured data
## AWS Lambda for data transformation
## S3 Buckets for data storage at various processing stages
## AWS Quicksight for analytics and visualization
## Data Processing Steps




# 1. Building the Data Catalog with AWS Glue
To enable structured querying, we use an AWS Glue Crawler to extract metadata from the raw JSON files.

First, we configure an IAM role with full access to the S3 bucket and Glue services.
A database is created in AWS Glue where the crawler stores extracted metadata.
The crawler processes JSON data, identifying three attributes:
Kind (String)
Etag (String)
Items (Array - Nested JSON)


# 2. Querying Data with Amazon Athena

To analyze the extracted data:
We configure Amazon Athena, which requires an S3 bucket to store query outputs.
As Glue does not support nested JSON natively, the array fields need transformation for proper querying in Athena.


# 3. ETL Transformation Using AWS Lambda

Since Glue processes data in a single-line format, we need an ETL process to convert JSON into a structure supported by Athena.

We use AWS Lambda to clean the data and store it in a new S3 bucket.
A new IAM role is created to allow Lambda access to S3.
The Lambda function extracts the [Items] array from JSON and transforms it into a structured format.
The transformed data is stored in a "clean" S3 bucket.

n Athena, you can use SerDe libraries to deserialize JSON data. Deserialization converts the JSON data so that it can be serialized (written out) into a different format like Parquet or ORC.(Source - AWS Documentation)
Link - https://docs.aws.amazon.com/athena/latest/ug/json-serde.html

# 4. Handling AWS Lambda Limitations

AWS Lambda has constraints:
Execution timeout (15 minutes)
Limited memory and CPU

To overcome these:
We install required Python packages for processing.
Increase maximum memory allocation for better performance.

After  above steps we get clean JSON and we successfully transformed [item] - array part from JSON files.

## Table Schema in Glue DB before transformation
![Image](https://github.com/user-attachments/assets/f352abb6-c9fc-4f4e-a682-0928ac916287)

## Table Schema in Glue DB after transformation
![Image](https://github.com/user-attachments/assets/cde49d82-b9c5-487e-ba29-95c83c3d8263)


# 5. Processing CSV Data with AWS Glue
Similar to JSON, we crawl CSV files and create a partitioned table in the Glue catalog.
Files are stored in S3, organized by regions.
The schema of the CSV files in the Glue database, after crawling the data files from the raw data bucket, is as follows:
![Image](https://github.com/user-attachments/assets/fc13f2bf-8c52-4d9d-ad89-22207fb9d341)

# 6. Data Type Optimization in Athena
When joining (Different data attributes are in different tables) JSON and CSV datasets in Athena, we encountered data type mismatches (e.g., string IDs in JSON).

We can change this in query by using cast function. However, to avoid casting costs, we modify the schema at the pre-processing stage in Lambda.
Once updated, we re-run Lambda for seamless data type conversion.

# 7. Storing Cleaned Data in S3 with Parquet Format
To optimize storage and query performance:
A new Glue job is created to save cleaned data in Parquet format(Non English Speaking Regions files dropped - Code can be found in files attached)
The Glue job script generated by visual ETL is customized with:
Partitioning by region (same as source bucket)
Filtering for English-speaking regions - (predicate_pushdown = "region in ('ca','gb','us')")
Dynamic Frame processing
Coalesce function to reduce clusters

# 8. Final Data Storage & Access Control
The processed data from clean data bucket after running through glue is moved to a new S3 bucket for analytical workloads.
Access is granted to data analysts and data scientists for querying and insights.

## Glue Job saving clean data from Glue Catalog in S3 Analytical bucket
![Image](https://github.com/user-attachments/assets/3c55e333-da74-4a90-91a1-549d6c7032f2)

# 9. Data Visualization with AWS Quicksight
The cleaned data is connected to AWS Quicksight (Can be connected to any BI tool for further Analysis).
Visualizations are built to present insights to senior stakeholders.

## Example graph from QucickSight (Further Analysis comes in scope of Data Analytics)
<img width="1422" alt="Image" src="https://github.com/user-attachments/assets/fdc94bbd-0009-4e84-b488-beb701e1805e" />

# Conclusion

This pipeline enables seamless extraction, transformation, and analysis of YouTube trending videos data using AWS services. The structured data can now be efficiently queried in Amazon Athena and visualized using AWS Quicksight, making it a robust solution for data analytics.
