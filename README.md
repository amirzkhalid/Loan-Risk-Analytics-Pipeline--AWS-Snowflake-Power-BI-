# Loan-Risk-Analytics-Pipeline--AWS-Snowflake-Power-BI-
Designed a modern data stack (AWS, Snowflake, Power BI) to deliver scalable loan default analytics and risk insights.


## 🔗 Dashboard
*(Insert your Power BI link here)*

---

## 📌 Problem Statement
Financial institutions face challenges in identifying high-risk borrowers and understanding loan default patterns.

This project aims to:
- Analyze loan distribution across demographics
- Identify factors contributing to loan defaults
- Provide insights into financial risk trends
- Enable data-driven decision making

---

## ⚙️ Architecture
Data Source → AWS S3 → Snowflake → Data Transformation → Power BI

---

## 🛠️ Tech Stack
- AWS S3 (Cloud Storage)
- AWS IAM (Access Control)
- Snowflake (Data Warehouse)
- SQL (Data Transformation)
- Power BI (Visualization)
- Excel

---

## 📊 Data Description

The dataset contains information about loan applicants, including their financial background, loan details, and repayment behavior.

It is structured as a **single denormalized table** for analytical purposes.

### 📁 Dataset Features

| Column Name        | Description |
|-------------------|------------|
| NumCreditLines    | Total number of active credit lines (credit cards, loans) |
| InterestRate      | Annual percentage rate (APR) charged on the loan |
| LoanTerm          | Loan duration in months |
| DTIRatio          | Debt-to-Income ratio indicating financial stress level |
| Education         | Highest education level attained |
| EmploymentType    | Employment status (Full-Time, Part-Time, Self-Employed, etc.) |
| MaritalStatus     | Borrower’s marital status |
| HasMortgage       | Indicates if borrower has an existing mortgage |
| HasDependents     | Indicates if borrower has dependents |
| LoanPurpose       | Purpose of the loan |
| HasCoSigner       | Indicates if borrower has a co-signer |
| Default           | Loan default status (Yes/No) |
| LoanDate          | Date when loan was issued |
| Age Groups (Created) | <19 "Teen", <=39 "Adults", <=59 "Middle Age Adults", "Senior Citizens" |
| Credit Scores Bins (Created) | Credit Score, <=400 "Very Low", <=450 "Low", <=650 "Medium", "High" |
| Income Bracket (Created) | Income, <30000 "Low Income", <60000 "Medium Income", >=60000 "High Income" |
| Year (Created) | Year of Loan was issued |
---

## 🔄 Data Pipeline

### 1. Data Ingestion
- Uploaded raw dataset to AWS S3
- Configured IAM roles for secure access

### 2. Data Warehousing
- Loaded data into Snowflake
- Structured into analytical tables

### 3. Data Transformation
- Cleaned missing values
- Created derived features:
  - Age Groups
  - Credit Score Bins
  - Income Brackets

### 4. Centralized Data Layer (Power BI Dataflow)
- Connected Snowflake to Dataflow
- Applied Power Query transformations
- Ensured consistent dataset across reports
- Established **Single Source of Truth (SSOT)**

### 5. Dashboard Development
- Import data from Dataflow to Power BI Desktop
- Built interactive dashboards

---

### Steps followed 

- Step 1 : Open AWS S3, Select local region, then Create a bucket and Upload dataset (Loan_Default) into it, dataset is a csv file.
- Step 2 : Open AWS IAM, Go to Access Manager and choose Roles section, then create role, select trusted entity: AWS Account, Option: Require External ID,     Extermal ID: 00000.
- Step 3 : For permission policies in IAM, choose AmazonS3FullyAccess, to ensure the third-part can access the S3, then give the role name as you wish. For me (Snowflake-Test-Role)
- Step 4 : open Snowflake add new workspace and run the syntax. Copy the Role Arn you had created and paste it into storage_aws_role_arn in the syntax. give your bucket path into storage_allowed_locations.

          create or replace storage integration s3_integration
          type =  external_stage
          storage_provider = 's3'
          enabled = true
          storage_aws_role_arn = 'Your Role Arn'
          storage_allowed_locations = ('s3://your_bucket_name/')
          comment = 'optional comment'
- Step 5 : Once the previous syntax successfully run, we need to get the property value of STORAGE_AWS_IAM_USER_ARN and STORAGE_AWS_ECTERNAL_ID by running this syntax

          //description integration object
          desc integration s3_integration;
- Step 6 : Open AWS IAM role and choose the role you had created, click on trust relationship and edit trust policy

          {
	          "Version": "2012-10-17",  //Don't change
	           "Statement": [
		            {
			                "Effect": "Allow",    //Don't change
			                "Principal": {
		                    	"AWS": "STORAGE_AWS_IAM_USER_ARN_"    //Get it from Snowflake
	                		},
		                	"Action": "sts:AssumeRole",     //Don't change
		                	"Condition": {
			                  	"StringEquals": {
			                      		"sts:ExternalId": "STORAGE_AWS_ECTERNAL_ID" //Get it from Snowflake
		                  		}
	                		}
              		}
              	]
            }
- Step 7 : In Snowflake, create Database, Schema, and Table.

			//Create Database
			CREATE database Loan_Default;


			//Create Schema
			create schema Loan_Default_Data;


			//Create Table
			CREATE OR REPLACE TABLE loan_default_dataset (
   			LoanID STRING,
    		Age INT,
    		Income INT,
    		LoanAmount INT,
    		CreditScore INT,
    		MonthsEmployed INT,
    		NumCreditLines INT,
    		InterestRate NUMBER(5,2),
    		LoanTerm INT,
    		DTIRatio NUMBER(4,2),
    		Education STRING,
    		EmploymentType STRING,
    		MaritalStatus STRING,
    		HasMortgage STRING,
    		HasDependents STRING,
    		LoanPurpose STRING,
    		HasCoSigner STRING,
    		Default INT,
    		Loan_Date DATE
			);
- Step 8 : Format the csv file and create stage before copying it into table

			//Formatting CSV file
 			CREATE OR REPLACE FILE FORMAT loan_default_csv_format
  			TYPE = 'CSV'
  			FIELD_DELIMITER = ','
  			SKIP_HEADER = 1
  			NULL_IF = ('NULL', 'null', '')
  			EMPTY_FIELD_AS_NULL = TRUE;


			//Creating Stage to hold the link of S3 bucket
			CREATE OR REPLACE STAGE Loan_Default_Stage
			  URL = 's3://projectpowerbipunya'
			  STORAGE_INTEGRATION = s3_integration
			  FILE_FORMAT = loan_default_csv_format;


			//Chceking Files in the bucket
			LIST @Loan_Default_Stage;


			//Copy the dataset into created table
			COPY INTO loan_default_dataset
			FROM @Loan_Default_Stage
			FILES = ('Loan_default.csv')
			FILE_FORMAT = (FORMAT_NAME = loan_default_csv_format)
			ON_ERROR = 'CONTINUE';
- Step 9 : Do data cleaning to ensure the data is cleaned before connecting to dataflow in PowerBi Service. However, the "Default" column name should be changes to prevent syntax issue. In this part, we need to check for duplicate values, Null values, and data inconsistency for numerical, categorical, and date
			
			//Duplicate Cheking
			SELECT *, COUNT(*) AS cnt
			FROM loan_default_dataset
			GROUP BY ALL
			HAVING COUNT(*) > 1;


			//Null Checking
			SELECT *
			FROM loan_default_dataset
				WHERE LoanID IS NULL OR Age IS NULL OR Income IS NULL OR LoanAmount IS NULL
   				OR CreditScore IS NULL OR MonthsEmployed IS NULL OR NumCreditLines IS NULL
   				OR InterestRate IS NULL OR LoanTerm IS NULL OR DTIRatio IS NULL
   				OR Education IS NULL OR EmploymentType IS NULL OR MaritalStatus IS NULL
   				OR HasMortgage IS NULL OR HasDependents IS NULL OR LoanPurpose IS NULL
  				OR HasCoSigner IS NULL OR Loan_Default_Status IS NULL OR Loan_Date IS NULL;


			//Numerical Inconsistency Checking
			SELECT
	  			MIN(Age) AS min_age, MAX(Age) AS max_age,
  				MIN(Income) AS min_income, MAX(Income) AS max_income,
  				MIN(LoanAmount) AS min_loanamount, MAX(LoanAmount) AS max_loanamount,
  				MIN(CreditScore) AS min_creditscore, MAX(CreditScore) AS max_creditscore,
  				MIN(MonthsEmployed) AS min_monthsemployed, MAX(MonthsEmployed) AS max_monthsemployed,
  				MIN(NumCreditLines) AS min_numcreditlines, MAX(NumCreditLines) AS max_numcreditlines,
  				MIN(InterestRate) AS min_interestrate, MAX(InterestRate) AS max_interestrate,
  				MIN(LoanTerm) AS min_loanterm, MAX(LoanTerm) AS max_loanterm,
  				MIN(DTIRatio) AS min_dtiratio, MAX(DTIRatio) AS max_dtiratio,
  				MIN(Loan_Default_Status) AS min_default, MAX(Loan_Default_Status) AS max_default
			FROM loan_default_dataset;


			//Catogarical Inconsistency Checking
			SELECT DISTINCT HasMortgage FROM loan_default_dataset;
			SELECT DISTINCT HasDependents FROM loan_default_dataset;
			SELECT DISTINCT HasCoSigner FROM loan_default_dataset;
			SELECT DISTINCT Education FROM loan_default_dataset;
			SELECT DISTINCT EmploymentType FROM loan_default_dataset;
			SELECT DISTINCT MaritalStatus FROM loan_default_dataset;
			SELECT DISTINCT LoanPurpose FROM loan_default_dataset;


			//Date Inconsistency Checking
			SELECT MIN(Loan_Date), MAX(Loan_Date)
			FROM loan_default_dataset;

As A result: There is no any duplicates row and null values



Numerical and Date Inconsistency Checkin Result
| Column Name        | Minimum | Maximum |
|-------------------|------------|------------|
| Age    | 18 | 69 |
| Income   | 15000 | 149999 |
| LoanAmount    | 5000 | 249999 |
| CreditScore    | 300 | 849 |
| MonthsEmployed    | 0 | 119 |
| NumCreditLines    | 1 | 4 |
| InterestRate    | 2.00 | 25.00 |
| LoanTerm    | 12 | 60 |
| DTIRatio    | 0.10 | 0.90 |
| Loan_Default_Status    | 0 | 1 |
| Loan_Date | 2013-01-01 | 2018-12-31 |

Categorical Inconsistency Checking Result
| Column Name        | Category 1 | Category 2 | Category 3 | Category 4 | Category 5 |
|-------------------|------------|------------|------------|------------|------------|
| HasMortgage | No | Yes | - | - | - |
| HasDependents | No | Yes | - | - | - |
| HasCoSigner | No | Yes |  - | - | - |
| Education | High School | Bachelor's | Master's | PhD | - |
| EmploymentType | Unemployed | Part-time | Self-employed | Full-time | - |
| MaritalStatus | Married | Single | Divorced | - | - |
| LoanPurpose | Auto | Home | Other | Education | Business |

- Step 10 : Feature Engineering. Create Age_groups, Income_Bracket, and year to transform numerical to categorical for future analysis

			//Create Age_Groups column
			ALTER TABLE loan_default_dataset ADD COLUMN Age_Groups VARCHAR(20);
			UPDATE loan_default_dataset
			SET Age_Groups = CASE
			    WHEN Age <= 19 THEN 'Teen'
			    WHEN Age <= 39 THEN 'Adults'
			    WHEN Age <= 59 THEN 'Middle Age Adults'
			    ELSE 'Senior Citizens'
			  END;


 			 //Create Income_Bracket column
			ALTER TABLE loan_default_dataset ADD COLUMN Income_Bracket VARCHAR(20);
			UPDATE loan_default_dataset
			SET Income_Bracket = CASE
			    WHEN Income < 30000 THEN 'Low Income'
			    WHEN Income >= 30000 AND Income < 60000 THEN 'Medium Income'
			    WHEN Income >= 60000 THEN 'High Income'
			  END;


			  //create year column
			ALTER TABLE loan_default_dataset ADD COLUMN Loan_Year INT;
			UPDATE loan_default_dataset
			SET Loan_Year = YEAR(Loan_Date);
- Step 11 : Ratings Visual was used to represent different ratings mentioned below,


## 📊 Dashboard Features

### 1. Loan Default & Overview
- Loan amount by purpose
- Default rate by employment type
- Average income comparison
- Average loan amount amount
- Default rate by year

![page_1](https://github.com/amirzkhalid/Loan-Risk-Analytics-Pipeline--AWS-Snowflake-Power-BI-/blob/b7bfef41d204aa6f28e5f463b55be61c027a6247/Screenshot%202026-08-10%20155024.png)

### 2. Demographics Analysis
- Median Loan Amount by Credit Score Category
- Average Loan Amount (High Credit) by Age Groups and Marital Status
- Total Loan (Adults) by Credit Score Bins
- Loan (Middle Age Adults) by Have Mortgage/Dependents
- Number of Loans by Education Type

![page_2](https://github.com/amirzkhalid/Loan-Risk-Analytics-Pipeline--AWS-Snowflake-Power-BI-/blob/1eb8f13443a4d9e9c25521e8f32b1558a55e82fc/Screenshot%202026-08-10%20160108.png)

### 3. Risk Metrics
- YOY Loan Amount Change by Year
- YOY Default Loan Change by Year
- YTD Loan Amount Change by Credit Score and Marital Status
- Decomposition Tree of Loan Amount

![page_3](https://github.com/amirzkhalid/Loan-Risk-Analytics-Pipeline--AWS-Snowflake-Power-BI-/blob/69ac27979593c164f27ea5a3b6572de4cfaa9985/Screenshot%202026-08-10%20160317.png)

---

## 📈 Key Insights
- Unemployed individuals have the highest default rate
- Full-time employees show lower risk
- Adults take the highest loan amounts
- Default trends fluctuate over time

---

### Steps followed 

- Step 1 : Load data into Power BI Desktop, dataset is a csv file.
- Step 2 : Open power query editor & in view tab under Data preview section, check "column distribution", "column quality" & "column profile" options.
- Step 3 : Also since by default, profile will be opened only for 1000 rows so you need to select "column profiling based on entire dataset".
- Step 4 : It was observed that in none of the columns errors & empty values were present except column named "Arrival Delay".
- Step 5 : For calculating average delay time, null values were not taken into account as only less than 1% values are null in this column(i.e column named "Arrival Delay") 
- Step 6 : In the report view, under the view tab, theme was selected.
- Step 7 : Since the data contains various ratings, thus in order to represent ratings, a new visual was added using the three ellipses in the visualizations pane in report view. 
- Step 8 : Visual filters (Slicers) were added for four fields named "Class", "Customer Type", "Gate Location" & "Type of travel".
- Step 9 : Two card visuals were added to the canvas, one representing average departure delay in minutes & other representing average arrival delay in minutes.
           Using visual level filter from the filters pane, basic filtering was used & null values were unselected for consideration into average calculation.
           
           Although, by default, while calculating average, blank values are ignored.
- Step 10 : A bar chart was also added to the report design area representing the number of satisfied & neutral/unsatisfied customers. While creating this visual, field named "Gender" was also added to the Legends bucket, thus number of customers are also seggregated according the gender. 
- Step 11 : Ratings Visual was used to represent different ratings mentioned below,

  (a) Baggage Handling

  (b) Check-in Services
  
  (c) Cleanliness
  
  (d) Ease of online booking
  
  (e) Food & Drink
  
  (f) In-flight Entertainment

  (g) In-flight Service
  
  (h) In-flight wifi service
  
  (i) Leg Room service
  
  (j) On-board service
  
  (k) Online boarding
  
  (l) Seat comfort
  
  (m) Departure & arrival time convenience
  
In our dataset, Some parameters were assigned value 0, representing those parameters are not applicable for some customers.

All these values have been ignored while calculating average rating for each of the parameters mentioned above.

- Step 12 : In the report view, under the insert tab, two text boxes were added to the canvas, in one of them name of the airlines was mentioned & in the other one company's tagline was written.
- Step 13 : In the report view, under the insert tab, using shapes option from elements group a rectangle was inserted & similarly using image option company's logo was added to the report design area. 
- Step 14 : Calculated column was created in which, customers were grouped into various age groups.

for creating new column following DAX expression was written;
       
        Age Group = 
        
        if(airline_passenger_satisfaction[Age]<=25, "0-25 (25 included)",
        
        if(airline_passenger_satisfaction[Age]<=50, "25-50 (50 included)",
        
        if(airline_passenger_satisfaction[Age]<=75, "50-75 (75 included)",
        
        "75-100 (100 included)")))
        
Snap of new calculated column ,

![Snap_1](https://user-images.githubusercontent.com/102996550/174089602-ab834a6b-62ce-4b62-8922-a1d241ec240e.jpg)

        
- Step 15 : New measure was created to find total count of customers.

Following DAX expression was written for the same,
        
        Count of Customers = COUNT(airline_passenger_satisfaction[ID])
        
A card visual was used to represent count of customers.

![Snap_Count](https://user-images.githubusercontent.com/102996550/174090154-424dc1a4-3ff7-41f8-9617-17a2fb205825.jpg)

        
 - Step 16 : New measure was created to find  % of customers,
 
 Following DAX expression was written to find % of customers,
 
         % Customers = (DIVIDE(airline_passenger_satisfaction[Count of Customers], 129880)*100)
 
 A card visual was used to represent this perecntage.
 
 Snap of % of customers who preferred business class
 
 ![Snap_Percentage](https://user-images.githubusercontent.com/102996550/174090653-da02feb4-4775-4a95-affb-a211ca985d07.jpg)

 
 - Step 17 : New measure was created to calculate total distance travelled by flights & a card visual was used to represent total distance.
 
 Following DAX expression was written to find total distance,
 
         Total Distance Travelled = SUM(airline_passenger_satisfaction[Flight Distance])
    
 A card visual was used to represent this total distance.
 
 
 ![Snap_3](https://user-images.githubusercontent.com/102996550/174091618-bf770d6c-34c6-44d4-9f5e-49583a6d5f68.jpg)
 
 - Step 18 : The report was then published to Power BI Service.
 
 
![Publish_Message](https://user-images.githubusercontent.com/102996550/174094520-3a845196-97e6-4d44-8760-34a64abc3e77.jpg)

# Snapshot of Dashboard (Power BI Service)

![dashboard_snapo](https://user-images.githubusercontent.com/102996550/174096257-11f1aae5-203d-44fc-bfca-25d37faf3237.jpg)

 
 # Report Snapshot (Power BI DESKTOP)

 
![Dashboard_upload](https://user-images.githubusercontent.com/102996550/174074051-4f08287a-0568-4fdf-8ac9-6762e0d8fa94.jpg)

# Insights

A single page report was created on Power BI Desktop & it was then published to Power BI Service.

Following inferences can be drawn from the dashboard;

### [1] Total Number of Customers = 129880

   Number of satisfied Customers (Male) = 28159 (21.68 %)

   Number of satisfied Customers (Female) = 28269 (21.76 %)

   Number of neutral/unsatisfied customers (Male) = 35822 (27.58 %)

   Number of neutral/unsatisfied customers (Female) = 37630 (28.97 %)


           thus, higher number of customers are neutral/unsatisfied.
           
### [2] Average Ratings

    a) Baggage Handling - 3.63/5
    b) Check-in Service - 3.31/5
    c) Cleanliness - 3.29/5
    d) Ease of online booking - 2.88/5
    e) Food & Drink - 3.21/5
    f) In-flight Entertainment - 3.36/5
    g) In-flight service - 3.64/5
    h) In-flight Wifi service - 2.81/5
    i) Leg room service - 3.37/5
    j) On-board service - 3.38/5
    k) Online boarding - 3.33/5
    l) Seat comfort - 3.44/5
    m) Departure & arrival convenience - 3.22/5
  
  while calculating average rating, null values have been ignored as they were not relevant for some customers. 
  
  These ratings will change if different visual filters will be applied.  
  
  ### [3] Average Delay 
  
      a) Average delay in arrival(minutes) - 15.09
      b) Average delay in departure(minutes) - 14.71
Average delay will change if different visual filters will be applied.

 ### [4] Some other insights
 
 ### Class
 
 1.1) 47.87 % customers travelled by Business class.
 
 1.2) 44.89 % customers travelled by Economy class.
 
 1.3) 7.25 % customers travelled by Economy plus class.
 
         thus, maximum customers travelled by Business class.
 
 ### Age Group
 
 2.1)  21.69 % customers belong to '0-25' age group.
 
 2.2)  52.44 % customers belong to '25-50' age group.
 
 2.3)  25.57 % customers belong to '50-75' age group.
 
 2.4)  0.31 % customers belong to '75-100' age group.
 
         thus, maximum customers belong to '25-50' age group.
         
### Customer Type

3.1) 18.31 % customers have customer type 'First time'.

3.2) 81.69 % customers have customer type 'returning'.
       
       thus, more customers have customer type 'returning'.

### Type of travel

4.1) 69.06 % customers have travel type 'Business'.

4.2) 30.94 % customers have travel type 'Personal'.

        thus, more customers have travel type 'Business'.
