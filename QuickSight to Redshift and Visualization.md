## Connect QuickSight to Redshift and Create Interactive Visualization

<br>

#### Step 1: Create QuickSight Account:
- Go to QuickSight Console → Sign up (if first time)
- Account name: `DavidAbe-firstassign`
- Email: `************`
- Region: **US West (Oregon)** (must match Redshift region)
- Select **Standard** edition

<BR>

#### Step 2: Create Dataset:
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

#### Step 3: Select Table:
- Choose schema: `public`
- Select table: `zillowdata`

<BR>

  
#### Step 4: Create QuickSight Dashboard
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





