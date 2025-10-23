
## Project Workflow

<br>

<img width="800" height="300" src="https://github.com/user-attachments/assets/beac20e2-e68e-49ed-b9b9-01be70b5a156">

<br>

#### Phase i: Data Extraction
1. Airflow Task 1 (tsk_extract_zillow_data_var) calls Zillow API via RapidAPI
2. Filters for Houston, TX properties with status "FOR_SALE"
3. Saves raw JSON response with timestamp to EC2

<br>
<br>

<img width="800" height="300" src="https://github.com/user-attachments/assets/9f658101-2ce6-4977-a6c0-8cd4b2b81ab6">


#### Phase ii: Data Loading
4. Airflow Task 2 (tsk_load_to_s3) moves JSON to S3 landing bucket
5. S3 Event Trigger invokes Lambda 1 automatically

<br>
<br>

#### Phase iii: Data Transformation

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
<br>

#### Phase iv: Data Warehousing
9. Airflow Task 3 (tsk_is_file_in_s3_available) monitors for CSV availability
10. Airflow Task 4 (tsk_transfer_s3_to_redshift) loads CSV into Redshift table

<br>
<br>

#### Phase v: Airflow DAG/Tasks

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
<br>

#### Phase vi: Visualization
11. QuickSight connects to Amazon Redshift using Direct Query for real-time visualization and analysis
12. Interactive dashboard provides real-time market insights

<br>
<br>
<br>

## Key Features 

#### i: Automated Pipeline
- Scheduled Execution: Daily runs at midnight (@daily schedule)
- Event-Driven: S3 triggers automatically invoke Lambda functions
- Self-Healing: 2 retry attempts with 15-second delays on failures
- Monitoring: S3KeySensor ensures data availability before downstream processing

<br>

#### ii: Serverless Transformation
- Cost-Effective: Pay only for Lambda execution time
- Scalable: Auto-scales with data volume
- Maintainable: No server management required
- Fast: Typical transformation completes in 10-30 seconds

<br>

#### iii: Data Quality
- Column Filtering: Reduces 40+ columns to 11 essential fields
- Schema Validation: Ensures CSV matches Redshift table structure
- Error Handling: Comprehensive logging via CloudWatch
- Data Preservation: Raw data maintained in landing zone

<br>

#### iv: Business Intelligence
- 5 KPI Metrics: Total properties, median price, price range, price/sqft
- 4 Visualization Types: Bar charts, scatter plots, donut charts, horizontal bars
- Real-Time Updates: Live connection with Direct Query for always up-to-date property insights


<br>
<br>
<br>


