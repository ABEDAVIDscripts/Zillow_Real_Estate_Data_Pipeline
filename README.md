# Zillow Real Estate Data Pipeline

<br>

#### Table of Contents

- [Project Overview](#overview)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Project Workflow](#project-workflow)
- [Key Features](#key-features)
- [Setup & Installation](#setup-and-installation)
  - [Step 1: Infrastructure Setup](#step-1-infrastructure-setup)
  - [Step 2: Create S3 Buckets](#step-2-create-s3-buckets)
  - [Step 3: Set Up IAM Roles](#step-3-set-up-iam-roles)
  - [Step 4: Set Up Lambda Functions](#step-4-set-up-lambda-functions)
  - [Step 5: Add S3KeySensor for Monitoring](#step-5-add-s3keysensor-for-monitoring)
  - [Step 6: Configure Airflow DAG in Visual Studio Code](#step-6-configure-airflow-dag-in-visual-studio-code)
  - [Step 7: Redshift Setup](#step-7-redshift-setup)
  - [Step 8: Access Redshift Query Editor](#step-8-access-redshift-query-editor)
  - [Step 9: Connect QuickSight to Redshift](#step-9-connect-quickSight-to-redshift)
  - [Step 10: Create QuickSight Dashboard](#step-10-create-quickSight-dashboard)


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

## Project Workflow

<br>

<img width="800" height="300" src="https://github.com/user-attachments/assets/beac20e2-e68e-49ed-b9b9-01be70b5a156">

<br>

### Phase i: Data Extraction
1. Airflow Task 1 (tsk_extract_zillow_data_var) calls Zillow API via RapidAPI
2. Filters for Houston, TX properties with status "FOR_SALE"
3. Saves raw JSON response with timestamp to EC2

<br>

<img width="800" height="300" src="https://github.com/user-attachments/assets/9f658101-2ce6-4977-a6c0-8cd4b2b81ab6">


### Phase ii: Data Loading
4. Airflow Task 2 (tsk_load_to_s3) moves JSON to S3 landing bucket
5. S3 Event Trigger invokes Lambda 1 automatically

<br>

### Phase iii: Data Transformation

<div align="center">
  <img src="https://github.com/user-attachments/assets/b5da08d5-fde7-4474-bd8e-d76a44c1f0a4" width="45%">
  <img src="https://github.com/user-attachments/assets/435b9b0d-c90f-4e3d-a0c2-717f68e21a07" width="45%">
</div>

<br>

6. Lambda 1 (CopyFirstAssignedJsonFile-LambdaFunction) copies JSON to intermediate bucket
7. S3 Event Trigger invokes Lambda 2 automatically

<br>

<div align="center">

  <!-- Row 1 -->
  <div>
    <img src="https://github.com/user-attachments/assets/f40ca90d-9975-4360-a4b5-9d226df472c8" width="45%">
    <img src="https://github.com/user-attachments/assets/f10e6403-d063-4080-b1a7-2693f244f909" width="45%">
  </div>

  <!-- Row 2 -->
  <div>
    <img src="https://github.com/user-attachments/assets/9cd8d533-013f-421c-82c2-73d8970c8591" width="45%">
    <img src="https://github.com/user-attachments/assets/0cc93340-6482-441a-b7c0-b35ffc234eab" width="45%">
  </div>

</div>


8. Lambda 2 (FirstAssignedTransformData-LambdaFunction): 
    - Parses JSON and extracts property listings
    - Filters to 11 essential columns (from original 40+)
    - Converts to Pandas DataFrame
    - Exports as CSV to transformed bucket

> #### CloudWatch Logs

<p align="center">
  <img src="https://github.com/user-attachments/assets/99004eae-cdfc-4960-9d2e-2d8eb8b28703" width="48%" />
  <img src="https://github.com/user-attachments/assets/1bc91eed-ceb1-405d-80a4-3233d071e4a8" width="48%" />
</p>


<br>

### Phase iv: Data Warehousing
9. Airflow Task 3 (tsk_is_file_in_s3_available) monitors for CSV availability
10. Airflow Task 4 (tsk_transfer_s3_to_redshift) loads CSV into Redshift table

<br>

### Phase v: Airflow DAG/Tasks

<div align="center">

  <!-- Row 1 -->
  <div>
    <img src="https://github.com/user-attachments/assets/1e219da2-e81d-44f6-8aae-59718a11bfc9" width="40%">
    <img src="https://github.com/user-attachments/assets/b4f3da44-9a67-40e7-8073-ab2c36d40014" width="40%">
  </div>

  <!-- Row 2 -->
  <div>
    <img src="https://github.com/user-attachments/assets/65e93a62-f9d0-4198-a2b0-4de3794a5291" width="40%">
    <img src="https://github.com/user-attachments/assets/862c1f6e-80ae-4b40-b22a-8eff480c8140" width="40%">
  </div>

  <!-- Row 3 -->
  <div>
    <img src="https://github.com/user-attachments/assets/bddb3876-f72f-495b-9db1-39252db4b5b4" width="40%">
    <img src="https://github.com/user-attachments/assets/a27c7e22-8a55-4599-a255-0954554cffeb" width="40%">
  </div>

</div>
<br>

### Phase vi: Visualization
11. QuickSight connects to Amazon Redshift using Direct Query for real-time visualization and analysis
12. Interactive dashboard provides real-time market insights

<br>
<br>
<br>

## Key Features

### Automated Pipeline
- Scheduled Execution: Daily runs at midnight (@daily schedule)
- Event-Driven: S3 triggers automatically invoke Lambda functions
- Self-Healing: 2 retry attempts with 15-second delays on failures
- Monitoring: S3KeySensor ensures data availability before downstream processing

<br>

### Serverless Transformation
- Cost-Effective: Pay only for Lambda execution time
- Scalable: Auto-scales with data volume
- Maintainable: No server management required
- Fast: Typical transformation completes in 10-30 seconds

<br>

### Data Quality
- Column Filtering: Reduces 40+ columns to 11 essential fields
- Schema Validation: Ensures CSV matches Redshift table structure
- Error Handling: Comprehensive logging via CloudWatch
- Data Preservation: Raw data maintained in landing zone

<br>

### Business Intelligence
- 5 KPI Metrics: Total properties, median price, price range, price/sqft
- 4 Visualization Types: Bar charts, scatter plots, donut charts, horizontal bars
- Real-Time Updates: Live connection with Direct Query for always up-to-date property insights


<br>
<br>
<br>

## Setup and Installation

### Prerequisites
- AWS Account with admin access
- EC2 instance (t2.medium, Ubuntu)
- RapidAPI account with Zillow API access
- Basic Python, SQL, and AWS

<BR>
<BR>

### Step 1: Infrastructure Setup
<img width="800" height="300" alt="ec2 instance" src="https://github.com/user-attachments/assets/fb350415-ee84-4d34-ae40-e08e746de247" />

- Launch EC2 instance (Ubuntu, t2.medium)
- SSH into instance:
```
ssh -i your-key.pem ubuntu@your-ec2-ip
```

- Update system: <br>
```
sudo apt update <br>
sudo apt install python3-pip python3-venv -y
```

- Create virtual environment
```
python3 -m venv firstassigned_env
source firstassigned_env/bin/activate
```

- Install Airflow and providers
```
pip install apache-airflow
pip install apache-airflow-providers-amazon
```

- Install AWS CLI
```
sudo snap install aws-cli --classic
```

- Initialize Airflow databasee
```
export AIRFLOW_HOME=~/airflow
airflow db migrate
```

- Start Airflow Services in Separate Terminals <br>
Create 3 terminals using Screen for Session Persistence:

```
#Terminal 1 - API Server

screen -S airflow-api
source firstassigned_env/bin/activate
export AIRFLOW_HOME=~/airflow
airflow api-server --port 8080
```


```
#Terminal 2 - Scheduler

screen -S airflow-scheduler
source firstassigned_env/bin/activate
export AIRFLOW_HOME=~/airflow
airflow scheduler
```


 ```
#Terminal 3 - DAG Processor

screen -S airflow-dag
source firstassigned_env/bin/activate
export AIRFLOW_HOME=~/airflow
airflow dag-processor
```

<BR>
<BR>
<br>

### Step 2: Create S3 Buckets
<p align="center" style="display: flex; flex-direction: column; align-items: center;">

  <!-- Top Row -->
  <span>
    <img src="https://github.com/user-attachments/assets/6660f350-c71f-4a93-9b71-4f39c99a71be" width="48%" height="280" style="object-fit:cover; margin-right:1%;">
    <img src="https://github.com/user-attachments/assets/cedf6215-a059-4003-878c-0ff341ecdaaa" width="48%" height="280" style="object-fit:cover;">
  </span>

  <br>

  <!-- Bottom Row -->
  <span>
    <img src="https://github.com/user-attachments/assets/56f82303-7eb8-453f-b028-021c28ecd8f7" width="48%" height="280" style="object-fit:cover; margin-right:1%;">
    <img src="https://github.com/user-attachments/assets/4f4b6c21-9de9-44c7-8d33-2bb7e8db6f67" width="48%" height="280" style="object-fit:cover;">
  </span>

</p>



Create three buckets in us-west-2 region
```
aws s3 mb s3://first-assigned-bucket --region us-west-2
aws s3 mb s3://copy-of-raw-jsonfile-bucket --region us-west-2
aws s3 mb s3://first-assigned-transformed-bucket --region us-west-2
```

<BR>
<BR>
<br>

### Step 3: Set Up IAM Roles

#### Step 3.1: For EC2 (Airflow access to S3):
- Go to IAM → Roles → Create Role
- Trusted entity: AWS service → EC2
- Attach policy: AmazonS3FullAccess
- Role name: first_assigned_ec2_access
- Attach role to EC2 instance

<br>

#### Step 3.2: For Lambda (S3 access + CloudWatch logs):
- Go to IAM → Roles → Create Role
- Trusted entity: AWS service → Lambda
- Attach policies:
```
AmazonS3FullAccess
AWSLambdaBasicExecutionRole
```
- Role name: lambda_function_s3_access_cloudwatch
- IAM Role Policy Breakdown:
```
  AmazonS3FullAccess – Enables read/write access to S3 buckets
  AWSLambdaBasicExecutionRole – Allows Lambda to write logs to CloudWatch
```


<BR>
<BR>
<br>

### Step 4: Set Up Lambda Functions
<img width="800" height="300" src= "https://github.com/user-attachments/assets/93a35e70-2d69-494c-af32-7217b0178032">

#### Step 4.1: Create Lambda 1
#### i: Create Lambda 1 Copy Function
- Go to Lambda Console → Create Function → Author from scratch
- Function name: CopyFirstAssignedJsonFile-LambdaFunction
- Runtime: Python 3.10
- Permissions → Change default execution role → Use an existing role:
```
select: lambda_function_s3_access_cloudwatch
```
- Click Create Function

<br>

#### ii: Add S3 Trigger to Lambda (Automatically run Lambda when file lands in landing bucket)
- Go to Lambda function page → Add trigger → Select a source: S3
- Bucket: first-assigned-bucket
- Event type: All object create events
- Acknowledge recursive invocation warning (check the box)
- Click Add

<br>

#### iii: Write Lambda 1 Code
```
import boto3
import json

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    # Get source bucket and object details from S3 event
    source_bucket = event['Records'][0]['s3']['bucket']['name']
    object_key = event['Records'][0]['s3']['object']['key']
    
    # Define target bucket
    target_bucket = 'copy-of-raw-jsonfile-bucket'
    
    # Define copy source
    copy_source = {'Bucket': source_bucket, 'Key': object_key}
    
    # Wait for object to be available (optional but good practice)
    waiter = s3_client.get_waiter('object_exists')
    waiter.wait(Bucket=source_bucket, Key=object_key)
    
    # Copy object to target bucket
    s3_client.copy_object(Bucket=target_bucket, Key=object_key, CopySource=copy_source)
    
    return {
        'statusCode': 200,
        'body': json.dumps('Copy completed successfully')
    }
```

<br>

#### iv: Adjust Lambda Timeout
Default timeout (3 seconds) might not be enough for larger files.
- Go to Configuration tab → General configuration → Edit
- Timeout: Change to 15 seconds
- Click Save

<br>

#### v: Deploy Lambda Function

<br>
<br>

#### 4.2: Lambda 2: Transform Function
Convert JSON to CSV format using Pandas for easier analytics and database loading.

<br>

#### i: Create Lambda Transform Function
- Go to Lambda Console → Create Function → Author from scratch
- Function name: FirstAssignedTransformData-LambdaFunction
- Runtime: Python 3.10
- Permissions: 
```
Use existing role: lambda_function_s3_access_cloudwatch
```
- Click Create Function

<br>

#### ii: Add S3 Trigger
- Click Add trigger → Source: S3
- Bucket: copy-of-raw-jsonfile-bucket
- Event type: All object create events
- Check recursive invocation acknowledgment
- Click Add

<br>

#### iii: Add Pandas Layer to Lambda (Using ARN)
Pandas is not included by default in Lambda 
- Click Add a layer
- Select Specify an ARN
- Enter ARN for us-west-2:
```
arn:aws:lambda:us-west-2:336392948345:layer:AWSSDKPandas-Python310:26
```
- Click Add

<br>

#### iv: Write Lambda Transform Code
```
import boto3
import json
import pandas as pd

s3_client = boto3.client('s3')

def lambda_handler(event, context):
    # Get source bucket and object details from S3 event
    source_bucket = event['Records'][0]['s3']['bucket']['name']
    object_key = event['Records'][0]['s3']['object']['key']
    
    print(f"Source bucket: {source_bucket}")
    print(f"Object key: {object_key}")
    
    # Define target bucket
    target_bucket = 'first-assigned-transformed-bucket'
    
    # Generate CSV filename (remove .json extension)
    target_file_name = object_key[:-5]  # Removes last 5 characters (.json)
    
    print(f"Target file name: {target_file_name}")
    
    # Wait for object to exist
    waiter = s3_client.get_waiter('object_exists')
    waiter.wait(Bucket=source_bucket, Key=object_key)
    
    # Get JSON file from intermediate bucket
    response = s3_client.get_object(Bucket=source_bucket, Key=object_key)
    print(f"Response: {response}")
    
    # Read and decode the file content
    data = response['Body']
    print(f"Data body: {data}")
    
    data = response['Body'].read().decode('utf-8')
    print(f"Decoded data: {data}")
    
    # Parse JSON
    data = json.loads(data)
    print(f"Parsed JSON data: {data}")
    
    # Extract results from Zillow API response
    f = []
    for i in data["results"]:
        f.append(i)

    # Convert to pandas DataFrame
    df = pd.DataFrame(f)

    # Define columns to keep (matching Redshift table)
    columns_to_keep = [
        'bathrooms', 'bedrooms', 'city', 'homeStatus', 'homeType',
        'livingArea', 'price', 'rentZestimate', 'state', 'streetAddress', 'zipcode'
    ]

    # Filter DataFrame to only include needed columns
    df = df.reindex(columns=columns_to_keep)

    # Convert DataFrame to CSV
    csv_data = df.to_csv(index=False)
    
    # Upload CSV to transformed bucket
    s3_client.put_object(
        Bucket=target_bucket,
        Key=f"{target_file_name}.csv",
        Body=csv_data,
        ContentType='text/csv'
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps(f'Successfully transformed {object_key} to CSV')
    }
```

<br>

#### v: Increase Lambda Timeout
Pandas processing takes longer than simple copy operations.
- Click Configuration tab → General configuration → Edit
- Timeout: Change to 1 minute
- Click Save

<br>

#### vi: Deploy Lambda Function

<BR>
<BR>
<BR>

### Step 5: Add S3KeySensor for Monitoring
Purpose: Wait for CSV file to appear in transformed bucket before proceeding to next pipeline stage (loading to Redshift).

<br>

#### Step 5.1: Install AWS Provider in Airflow
- Connect to EC2 terminal:
```
source firstassigned_env/bin/activate
export AIRFLOW_HOME=~/airflow
pip install apache-airflow-providers-amazon
```

<br>

#### Step 5.2: Restart Airflow Components
Restart all 3 screen sessions to load new provider:

- Restart API Server
```
screen -X -S airflow-api quit
screen -S airflow-api
source firstassigned_env/bin/activate
export AIRFLOW_HOME=~/airflow
airflow api-server --port 8080
# Ctrl+A then D
```

- Restart Scheduler
```
screen -X -S airflow-scheduler quit
screen -S airflow-scheduler
source firstassigned_env/bin/activate
export AIRFLOW_HOME=~/airflow
airflow scheduler
# Ctrl+A then D
```

- Restart DAG Processor
```
screen -X -S airflow-dag quit
screen -S airflow-dag
source firstassigned_env/bin/activate
export AIRFLOW_HOME=~/airflow
airflow dag-processor
# Ctrl+A then D
```

<br>

#### Step 5.3: Create AWS Connection in Airflow
- Go to Airflow UI: http://your-ec2-ip:8080
- Click Admin → Connections → Add Connection
- Fill in:
```
Connection Id: aws_s3_conn
Connection Type: Amazon Web Services
Leave AWS Access Key and Secret blank (using IAM role)
Extra: {"region_name": "us-west-2"}
```
- Save

<br>

#### Step 5.4: Update DAG Code with S3KeySensor
- Open zillowanalytics.py in VS Code and update:
- Add new import:
```
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor
```

- Add S3 bucket variable:
```
# Define the S3 bucket for transformed data
s3_bucket = 'first-assigned-transformed-bucket'
```

- Add Task 3 (after load_to_s3 task):
```
# Task 3: Wait for CSV file to appear in transformed bucket
is_file_in_s3_available = S3KeySensor(
    task_id='tsk_is_file_in_s3_available',
    bucket_key='{{ ti.xcom_pull("tsk_extract_zillow_data_var")[1] }}',
    bucket_name=s3_bucket,
    aws_conn_id='aws_s3_conn',
    wildcard_match=False,
    timeout=120,  # Wait up to 2 minutes
    poke_interval=5,  # Check every 5 seconds
)
```

- Update task dependencies:
```
# Define task execution order
extract_zillow_data_var >> load_to_s3 >> is_file_in_s3_available
```

<BR>
<BR>
<BR>

### Step 6: Configure Airflow DAG in Visual Studio Code
- Create file: ~/airflow/dags/zillowanalytics.py
- Script:
```
from airflow import DAG
from datetime import timedelta, datetime
import json
import requests
from airflow.providers.standard.operators.python import PythonOperator
from airflow.providers.standard.operators.bash import BashOperator
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor
from airflow.providers.amazon.aws.transfers.s3_to_redshift import S3ToRedshiftOperator



#Load JSON config file
with open('/home/ubuntu/airflow/config_api.json', 'r') as config_file:
    api_host_key = json.load(config_file)


# Generate timestamp for file naming
now = datetime.now()
dt_now_string = now.strftime("%d%m%Y%H%M%S")


# Define the S3 bucket for transformed data
s3_bucket = 'first-assigned-transformed-bucket'



def extract_zillow_data(**kwargs):
    """
    Extracts real estate data from Zillow API and saves to JSON file
    
    Args:
        **kwargs: Dictionary containing url, headers, querystring, date_string
    
    Returns:
        List containing [json_file_path, csv_file_name]
    """

    url = kwargs['url']
    headers = kwargs['headers']
    querystring = kwargs['querystring']
    dt_string = kwargs['date_string']
    
    # Make the API/GET request
    response = requests.get(url, headers=headers, params=querystring)
    response_data = response.json()

    # specify the output file path (JSON)
    output_file_path = f"/home/ubuntu/response_data_{dt_string}.json"
    file_str = f'response_data_{dt_string}.csv'

    # Save the JSON response to a file
    with open(output_file_path, "w") as output_file:
        json.dump(response_data, output_file, indent=4)  # formatted

    # return paths for downstream tasks
    output_list = [output_file_path, file_str]
    return output_list



# Default DAG configuration
default_args = {
    'owner': 'david', 
    'depends_on_past': False,
    'start_date': datetime(2025, 9, 1),  
    'email': ['20.davidabe@gmail.com'], 
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 2,
    'retry_delay': timedelta(seconds=15)
}    



with DAG(
    'zillow_analytics_dag',
    default_args = default_args,
    schedule = '@daily',
    catchup = False
) as dag:


    # Task 1: Extract data from Zillow API
    extract_zillow_data_var = PythonOperator(
        task_id= 'tsk_extract_zillow_data_var',
        python_callable=extract_zillow_data,
        op_kwargs={
            'url': 'https://zillow56.p.rapidapi.com/search', 
            'querystring': {
                "location":"houston, tx",
                "output": "json",
                "status": "forSale",
                "sortSelection": "priorityscore",
                "listing_type": "by_agent",
                "doz": "any"
            }, 
            'headers': api_host_key, 
            'date_string': dt_now_string
        }
    )


    # Task 2: Upload JSON to S3 Landing Zone
    load_to_s3 = BashOperator(
        task_id='tsk_load_to_s3',
        bash_command='aws s3 mv {{ ti.xcom_pull("tsk_extract_zillow_data_var")[0] }} s3://first-assigned-bucket/'
    )


    # Task 3: Wait for CSV file to appear in transformed bucket
    is_file_in_s3_available = S3KeySensor(
        task_id='tsk_is_file_in_s3_available',
        bucket_key='{{ ti.xcom_pull("tsk_extract_zillow_data_var")[1] }}',
        bucket_name=s3_bucket,
        aws_conn_id='aws_s3_conn',
        wildcard_match=False,
        timeout=120,  # Wait up to 2 minutes
        poke_interval=5,  # Check every 5 seconds
    )


    # Task 4: Transfer to Redshift
    transfer_s3_to_redshift = S3ToRedshiftOperator(
        task_id='tsk_transfer_s3_to_redshift',
        aws_conn_id='aws_s3_conn',
        redshift_conn_id='conn_id_redshift',
        s3_bucket=s3_bucket,
        s3_key='{{ ti.xcom_pull("tsk_extract_zillow_data_var")[1] }}',
        schema='PUBLIC',
        table='zillowdata',
        copy_options=['csv IGNOREHEADER 1']
    )



# Task order
extract_zillow_data_var >> load_to_s3 >> is_file_in_s3_available >> transfer_s3_to_redshift

```

- Create API config file: ~/airflow/config_api.json
- script:
```
{
    "x-rapidapi-key": "your-rapidapi-key-here",
    "x-rapidapi-host": "zillow56.p.rapidapi.com"
}
```

<br>
<br>
<BR>

### Step 7: Redshift Setup
#### Step 7.1: Create Redshift Cluster:
- AWS Console → Search: Redshift
- Redshift Console → Create cluster
- Cluster identifier: redshift-cluster-1
- Node type: dc2.large
- Number of nodes: 1
- Database name: dev
- Database port: 5439 
- Admin username: awsuserzillow
- Admin password: (create strong password)
- Create cluster

<br>

#### Step 7.2: Configure Security Group
Allow Airflow and QuickSight to Connect:

- Go to Redshift Console → click `redshift-cluster-1`
- Properties tab → Network and security section
- Click on the VPC security group link
- Click Inbound rules tab → Edit inbound rules
- **Add 8 rules total** (1 for Airflow + 7 for QuickSight):
- Rule 1: Allow from EC2 (Airflow)
```
Type: Custom TCP
Port: 5439
Source: EC2 security group (sg-xxxxx) OR EC2 private IP
Description: Airflow access
```

- Rules 2-8: Allow from QuickSight (us-west-2 region)
```
Type: Custom TCP
Port: 5439
Source: 52.10.0.0/15
Description: QuickSight access (1 of 7)

Type: Custom TCP
Port: 5439
Source: 52.12.0.0/15
Description: QuickSight access (2 of 7)

Type: Custom TCP
Port: 5439
Source: 52.24.0.0/14
Description: QuickSight access (3 of 7)

Type: Custom TCP
Port: 5439
Source: 52.32.0.0/14
Description: QuickSight access (4 of 7)

Type: Custom TCP
Port: 5439
Source: 52.40.0.0/16
Description: QuickSight access (5 of 7)

Type: Custom TCP
Port: 5439
Source: 54.68.0.0/14
Description: QuickSight access (6 of 7)

Type: Custom TCP
Port: 5439
Source: 54.184.0.0/13
Description: QuickSight access (7 of 7)
```

- Save rules

<br>

#### Step 7.3: Create IAM Role for Redshift (S3 access for COPY command):
- Go to IAM → Roles → Create Role
- Trusted entity: AWS service → Redshift
- Attach policy: AmazonS3ReadOnlyAccess
- Role name: `redshift_firstassigned_s3_access_role`
- Attach to Redshift cluster

<br>

#### Step 7.4: Attach IAM Role to Cluster
- Go to Redshift Console → redshift-cluster-1
- Click Actions → Manage IAM roles
- Available IAM roles: Select `redshift_firstassigned_s3_access_role`
- Click Associate IAM role → Click Save changes
- Wait for status to show "In-sync" or "Active"

<BR>
<BR>
<br>

### Step 8: Access Redshift Query Editor
<p align="center">
  <img src="https://github.com/user-attachments/assets/c2d187f1-9097-4082-bbce-add18b98ca79" width="48%" height="300" style="object-fit:cover; margin-right:1%;">
  <img src="https://github.com/user-attachments/assets/c37364dd-78d6-4459-b39e-39743e7a7161" width="48%" height="300" style="object-fit:cover;">
</p>

#### Step 8.1: Connect to Database:
- Go to Redshift console → Query editor v2
- Select cluster: `redshift-cluster-1`
- Authentication:
```
Database name: dev
User name
Password
```
- Create connection

<br>

#### Step 8.2: Create Table in Query Editor v2
In Query Editor v2, run this SQL:
```
CREATE TABLE IF NOT EXISTS zillowdata(
    bathrooms NUMERIC,
    bedrooms NUMERIC,
    city VARCHAR(225),
    homeStatus VARCHAR(225),
    homeType VARCHAR(225),
    livingArea NUMERIC,
    price NUMERIC,
    rentZestimate NUMERIC,
    state VARCHAR(225),
    streetAddress VARCHAR(225),
    zipcode INT
);
```

<br>

#### Step 8.3: Verify Table Created
```
SELECT
 *
FROM zillowdata;
```

<br>

#### Step 8.4: Create Redshift Connection in Airflow
- Airflow UI → Admin → Connections
- Add Connection
```
Connection Id: conn_id_redshift
Connection Type: Amazon Redshift
Host (Get from Redshift console → Cluster → Endpoint, remove :5439/dev)
Schema: dev
Login: awsuserzillow
Password
Port: 5439
Extra: {"region": "us-west-2"}
```
- Save

<br>
<br>
<BR>

### Step 9: Connect QuickSight to Redshift

#### Step 9.1: Create QuickSight Account:
- Go to QuickSight Console → Sign up (if first time)
- Account name: `DavidAbe-firstassign`
- Email: `************`
- Region: **US West (Oregon)** (must match Redshift region)
- Select **Standard** edition

<BR>

#### Step 9.2: Create Dataset:
- QuickSight Console → Datasets → New dataset
- Redshift (Auto-discovered)
- Configure connection:
```
   Data source name: `zillowdataset`
   Database name: `dev`
   Username: `awsuserzillow`
   Password: (Redshift password)
```
- Validate connection → Create data source

<BR>

#### Step 9.3: Select Table:
- Choose schema: `public`
- Select table: `zillowdata`

<BR>
<BR>
<BR>

  
### Step 10: Create QuickSight Dashboard
<img height="400" alt="dashboard img" src="https://github.com/user-attachments/assets/1c1ab9f7-d2d7-458b-ab83-66757c0ab707" />

**Build Interactive Visualizations:**

- **KPIs** 
   - Total Properties: 410 listings
   - Median Price: $327.7K
   - Most Expensive: $60.0M (luxury segment)
   - Cheapest: $30.0K (entry-level)
   - Price Per Sqft: $221.2

- **Metrics Visualizations:**
   - **A. Avg Price by Bedrooms (Vertical Bar Chart)**
     - X-axis: Number of bedrooms (1-8)
     - Y-axis: Average price
     - **Insight:** 8-bedroom properties command ~$60M average, showing luxury market segment

   - **B. Price vs Living Area (Scatter Plot)**
     - X-axis: Living area (square footage)
     - Y-axis: Property price
     - **Insight:** Strong positive correlation between size and price
     - Outlier detection: 8-bedroom, 220,000 sqft property priced at $600M (luxury mansion segment)

  - **C. Properties by Home Type (Donut Chart)**
     - Shows market distribution
     - Single Family: 362 properties (88%)
     - Townhouse: 37 properties (9%)
     - Condo: 11 properties (3%)
     - **Insight:** Single-family homes dominate Houston real estate market

   - **D. Avg Price by City (Horizontal Bar Chart)**
     - Compares average prices across Houston metro areas
     - Houston city has the highest average Price(~$2.1M)
     - While Stafford has the lowest average price
     - **Insight:** Significant geographic price variation within metro area


<BR>


