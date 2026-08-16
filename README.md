# Loan-Risk-Analytics-Pipeline--AWS-Snowflake-Power-BI-
-Designed a modern data stack (AWS, Snowflake, Power BI) to deliver scalable loan default analytics and risk insights.

-**Disclaimer**: This is a synthetic dataset, as this dataset was used for practical purpose. So, the dashboard's insight may not be applied in real world cases

---

## 📌 Problem Statement
Financial institutions face significant capital losses when borrowers default on their loans. Currently, loan applicant data is siloed and unprocessed, making it difficult to assess credit risks effectively. Without a centralized and automated end-to-end data pipeline, management lacks the real-time visibility and advanced analytical metrics needed to identify high-risk borrowers early and mitigate financial exposure.

This project aims to:
- Build a Secure and Automated Data Pipeline
- To Enrich Data Quality for Deeper Analytical Insights
- To Establish a Single Source of Truth via Power BI Dataflow
- Deliver Actionable Insights with Interactive Dashboards

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
As the dataset is cleaned, we can move forward to the next step.

- Step 11 : Open PowerBI service and create a new workspace by clicking on the workspace at the left pane, click on the "+New Item" then search for dataflow Gen2/Gen1 then Create a new file. In the power query, Click on Get data and choose snowflake.
- Step 12 : In Connection Settings, key-in your server (to get your server, open snowflake and click on your account -> view account detail then copy your server URL. while you can check your warehouse at the top of snowflake pane (example: COMPUTE_WH). Your username and password is your snowflake username and password
- Step 13 : Open PowerBI desktop then click get data, and choose workspace that has been created with dataflow.


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

**Employment type is the clearest default-risk driver.** Default rate ranks cleanly by employment stability: Unemployed (3.39%) > Part-time (3.01%) > Self-employed (2.86%) > Full-time (2.36%). This is the most consistent risk signal in the dataset.

**Loan sizing doesn't scale with creditworthiness.** Median loan amount actually decreases as credit score improves, Low (128,397) down to High (127,149). Borrowers with better credit aren't receiving larger loans, suggesting loan amount isn't being risk-adjusted by credit score.

**Loan volume is concentrated in medium-to-high credit borrowers, despite no size advantage.** By total volume, Medium (4.6bn) and High (4.5bn) credit score bins carry the bulk of the loan book, versus Low (1.1bn), most of the portfolio sits with medium/high-credit borrowers even though their average loan size isn't any higher.

**Default rate has no year-over-year trend.** Across 2013–2018, default rate stays in a tight 11.5%–11.75% band, with a temporary rise in 2015–2016. On its own, year isn't a useful predictor of default risk.

**Loan growth and default growth move together.** YoY loan amount and YoY default amount rise and fall in the same years (both dip in 2014, both rise in 2015, both dip in 2017), expansion years bring proportionally more defaults along with more lending.

**High-income borrowers dominate the loan book, but employment type doesn't differentiate within that group.** The High Income bracket accounts for 21.73bn of 32.58bn total loan sum (67%), split almost evenly across Full-time, Part-time, and Self-employed (~5.44bn each).

**Mortgage and dependents status has no effect on loan amount.** Loan amounts are flat (3.1bn) regardless of whether a borrower has a mortgage or dependents, making this a weak segmentation variable for loan sizing.

---

### The Documentation is not finished yet. (Soon to be continue)



