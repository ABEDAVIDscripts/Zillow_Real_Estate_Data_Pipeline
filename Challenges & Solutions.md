## Challenges & Solutions

#### i. Challenge 1: Lambda Pandas Dependency
- Problem: Pandas not available in AWS Lambda runtime
- Solution: Added AWS SDK Pandas Layer `AWSSDKPandas-Python310`

<br>

#### ii. Challenge 2: CSV–Redshift Column Mismatch
- Problem: API returned 40+ columns, but only 11 needed
- Solution: Used df.reindex(columns=columns_to_keep) to filter columns

<br>

#### iii. Challenge 3: S3KeySensor Timeout
- Problem: Sensor failed when Lambda processing took too long
- Solution: Increased Lambda memory to 128 MB and sensor timeout to 60sec

<BR>

#### iv. Challenge 4: Redshift Connection Timeout
- Problem: EC2 instance couldn’t reach Redshift
- Solution: Added inbound rule for port 5439 from EC2 security group
<!-- Comment: 1 inbound rule added for EC2 -->

<br>

#### v. Challenge 5: QuickSight Redshift Access
- Problem: QuickSight couldn’t connect to Redshift
- Solution: Allowed QuickSight IP CIDR blocks (e.g., 52.10.0.0/15) in security group
<!-- Comment: 7 inbound rules added for Redshift -->

<br>
<br>