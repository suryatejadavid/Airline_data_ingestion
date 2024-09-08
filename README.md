# Ingest Daily Flight Data in Redshift Fact Table

## Overview

This project focuses on ingesting daily flight data into a Redshift fact table. The data pipeline utilizes a variety of AWS services such as S3, Glue, Redshift, and Step Functions to ensure efficient data ingestion and processing. The goal is to automate the process of ingesting daily flight data, managing the workflow with a step-by-step architecture involving notifications, event-driven triggers, and data transformation.

## Tech Stack

1. **S3:** Stores the raw flight and airport data.
2. **S3 CloudTrail Notification:** Monitors data events in S3 for changes or uploads.
3. **EventBridge Pattern Rule:** Triggers AWS services based on the events captured by CloudTrail.
4. **Glue Crawler:** Scans the data in S3 to infer schema and create a data catalog for further ETL operations.
5. **Glue Visual ETL:** Orchestrates the Extract, Transform, Load (ETL) process for transforming and preparing the data for Redshift.
6. **SNS:** Sends notifications for success or failure of the data ingestion process.
7. **Redshift:** Stores the processed flight data in fact tables for analytics and querying.
8. **Step Function:** Manages the workflow of the entire ingestion process by orchestrating the different AWS services.

## Architecture

The project consists of the following main components:

1. **S3 Bucket:**
    - Stores raw flight data (`flights.csv`) and airport information (`airports.csv`).
  
2. **Glue Crawler and Glue ETL:**
    - The **Glue Crawler** scans the raw data to infer schema and update the AWS Glue Data Catalog.
    - **Glue Visual ETL** transforms the data and prepares it for ingestion into Redshift.

3. **EventBridge & CloudTrail:**
    - **CloudTrail** monitors the S3 bucket for new uploads of daily flight data and triggers an event.
    - **EventBridge** triggers a workflow based on the CloudTrail event when new data is uploaded.

4. **Step Function & SNS:**
    - **Step Functions** orchestrate the workflow, including Glue Crawlers, Glue ETL jobs, and Redshift data loading.
    - **SNS** sends notifications for job status (success/failure).

5. **Redshift:**
    - The processed data is stored in the **Redshift** fact table for querying and analytics.

## Prerequisites

- **AWS Account** with necessary permissions for S3, Glue, Redshift, EventBridge, Step Functions, and SNS.
- **AWS CLI** and credentials configured to interact with your AWS account.

### Explanation:
- **Tech Stack**: Lists all the AWS services used for the ingestion pipeline.
- **Redshift Schema and Table Creation**: Provides the necessary SQL commands for setting up Redshift tables.
- **Setup and Deployment**: Outlines the steps for setting up the pipeline, from uploading data to S3, setting up Glue, and automating with EventBridge and Step Functions.
- **Project Structure**: Displays the folder and file structure for clarity.

### Required Python Packages, Redshift:
For running the Glue ETL job, make sure the following packages are installed in the Glue job environment:
```bash
boto3
awswrangler

-- Create the schema
CREATE SCHEMA airlines;

-- Create the airports dimension table
CREATE TABLE airlines.airports_dim (
    airport_id BIGINT,
    city VARCHAR(100),
    state VARCHAR(100),
    name VARCHAR(200)
);

-- Load data into airports dimension table
COPY airlines.airports_dim
FROM 's3://airlines-dataset-gds/airports.csv'
IAM_ROLE 'arn:aws:iam::348532040329:role/service-role/AmazonRedshift-CommandsAccessRole-20230820T081203'
DELIMITER ','
IGNOREHEADER 1
REGION 'us-east-1';

-- Create the daily flights fact table
CREATE TABLE airlines.daily_flights_fact (
    carrier VARCHAR(10),
    dep_airport VARCHAR(200),
    arr_airport VARCHAR(200),
    dep_city VARCHAR(100),
    arr_city VARCHAR(100),
    dep_state VARCHAR(100),
    arr_state VARCHAR(100),
    dep_delay BIGINT,
    arr_delay BIGINT
);

