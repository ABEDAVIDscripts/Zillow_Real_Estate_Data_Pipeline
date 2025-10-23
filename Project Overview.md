# Zillow Real Estate Data Pipeline

<br>

### Overview
This project implements a production-grade, automated ETL pipeline that extracts Houston real estate listings from the Zillow API, transforms the data using serverless AWS Lambda functions, loads it into a Redshift data warehouse, and visualizes insights through interactive QuickSight dashboards.

#### Project Highlights:
- Fully automated data pipeline orchestrated with Apache Airflow
- Serverless architecture using AWS Lambda for scalable data transformation
- Interactive BI dashboard with 5 KPIs and 4 visualization types
- 410+ property records analyzed across Houston metro area
- Sub-80-second end-to-end pipeline execution time

<br>
<br>
<br>

## Architecture

<img width="1260" height="160" alt="Architecture Flow Diagram" src="https://github.com/user-attachments/assets/ea6ff816-f821-4c11-b8a4-a2af67d91417" />


<br>

#### Data Flow Summary:
- Extract: Zillow API to EC2 (JSON)
- Load: EC2 to S3 Landing Zone (JSON)
- Copy: Lambda 1 copies to Intermediate Zone (JSON)
- Transform: Lambda 2 converts to CSV + filters columns
- Wait: S3KeySensor monitors CSV availability
- Load: CSV loaded into Redshift table
- Visualize: QuickSight queries Redshift for insights

<br>
<br>

## Technologies Used

<br>

### Cloud & Infrastructure

|        Service    |                   Purpose|                                      Configuration|
|------------------:|-------------------------:|--------------------------------------------------:|
|            AWS EC2|              Airflow host|                                   Ubuntu t2.medium|
|             AWS S3|                 Data lake|     3 buckets (landing, intermediate, transformed)|
|         AWS Lambda| Serverless transformation|                           Python 3.10, 2 functions|
|       AWS Redshift|            Data warehouse|                          dc2.large cluster, 1 node|
|  Amazon QuickSight|     Business intelligence|                                   Standard edition|



<br>

### Orchestration & Processing
- Apache Airflow 2.x: Workflow orchestration (4 tasks: extract, load, sensor, transfer)
- Python 3.10: Core programming language
- Pandas: Data manipulation and CSV transformation
- Boto3: AWS SDK for Python

<BR>

### Data & APIs
- RapidAPI (Zillow): Real estate data source
- JSON: Raw data format
- CSV: Transformed data format
- Amazon Redshift SQL: Data warehouse queries

<BR>

### Development Tools
- VS Code: Remote SSH development
- Screen: Terminal multiplexer for session persistence
- Git: Version control (for this repository)


<br>
<br>
<br>