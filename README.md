## Finacial Organisation Data Analysis

## Analyzing a financial organization's complaints dataset 

### Executive Summary

A good example of a financial organization in this analysis is a bank, for every bank customer service is one of the key fundamentals to survive the competition and also to realize profits. Customers bring forward issues and complaints on different products of the bank. Some of the complaints can cause damage to the reputation of the bank if they are not addressed in time and satisfactorily. In this project, we are going to analyze a financial organization's complaint dataset to understand the distribution, response effectiveness, and regional patterns, to derive actionable insights for informed decision-making and enhancing customer satisfaction.

### Project Overview 

The project aims to understand the complaints distribution, response effectiveness, and regional patterns of the dataset to derive actionable insights to help the organization improve in areas such as  service delivery, product improvement, market demands, and turnaround time. The dataset was extracted from Real World Fake Data's data. world platform. We are going to look at the following tasks...

- What is the distribution of complaints across different products and sub-products?

- Which issues and sub-issues are most commonly reported by consumers?

- What is the company's typical response to consumer complaints?

- How does the timely response rate vary across different states?

- What is the proportion of complaints that were disputed by consumers?

- Which states have the highest and lowest ZIP code values for complaints?

- Are there any noticeable trends or patterns in the submission method (e.g., web, referral) of complaints over time?

- What are the most common issues reported by consumers who did not provide consent for further actions?

- Is there a correlation between the company's response type and the presence of monetary relief in closed cases?

- For closed cases with monetary relief, what types of issues were commonly associated with such resolutions?

### Data Analysis using Microsoft SQL Server 

##### Checking the data in the table

After loading the CSV file into the SQL server as a flat file I checked the dataset using the code below, data check is important because I get to confirm if the dataset ifs the right one and also if the data is clean and well transformed for the analysis.  If the data is not clean then data cleaning procedures must be performed first because we need clean data to get correct results.

```sql
SELECT  *
FROM    dbo.FinConsumerComplaints;
```
![sql1](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/02fa578a-92bb-4979-8664-de2034a86872)

##### Checking for duplicates in the table

I also checked for duplicates in the dataset as a data-cleaning step because duplicates can give a wrong interpretation of the data.
  
```sql
 With count_rows AS 
(
SELECT     Complaint_ID,
           Product,
           ROW_NUMBER() OVER ( PARTITION BY Complaint_ID ORDER BY (select 1)) as row_count
FROM       dbo.FinConsumerComplaints
)

SELECT     Count( * ) AS number_of_rows
FROM       count_rows
WHERE      row_count = 1;
```
![sql2](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/899f3c37-0f06-45f1-8fbe-92403b42a654)

#### Data Cleaning steps

I removed special characters from the sub-product and sub-issue columns and  'N/A' on the consumer disputed column and replaced them with 'NULL' so that the table can have the same response where there is  nothing captured.

```sql
UPDATE  dbo.FinConsumerComplaints
SET     Sub_product = 'NULL'
WHERE   Sub_product = '""';
```
```sql
UPDATE  dbo.FinConsumerComplaints
SET     Sub_issue = 'NULL'
WHERE   Sub_issue = '""'; 
```
```sql
UPDATE  dbo.FinConsumerComplaints
SET     Consumer_disputed = 'NULL'
WHERE   Consumer_disputed = 'N/A';
```

#### Analysis

##### What is the distribution of complaints across different products and sub-products?

The complaints about products had credit cards with the highest number, followed by checking of savings accounts, mortgages, prepaid cards, bank accounts, debt collection, student loans, and vehicle loans or leases in that order. The sub-products were also ranked accordingly.

```sql
 WITH cte_product AS  
          (
            SELECT     Product,
                       DENSE_RANK() OVER(ORDER BY COUNT(1) DESC) AS prdct_rank
            FROM       dbo.FinConsumerComplaints
            GROUP BY   Product
           ),

      cte_Sub_product AS
         (
           SELECT      Sub_product,
                       DENSE_RANK() OVER(ORDER BY COUNT(1) DESC) AS Sub_prdct_rnk
           FROM        dbo.FinConsumerComplaints
           GROUP BY    Sub_product
		  
          )

SELECT     DISTINCT product,
           prdct_rank,
           Sub_product,
           Sub_prdct_rnk
FROM       cte_product
CROSS JOIN cte_Sub_product;
```
![SQLONE](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/8b27d985-b299-4166-bc1a-b4507714463d)


##### Which issues and sub-issues are most commonly reported by consumers?
The issue with the highest number of consumers is managing an account followed by deposits and withdrawals, and the third largest is for those who are having challenges with payments, the fourth issue is for those who are having challenges failing to pay their mortgages and the last one is for those who are not agreeing with balances showing on their statements. The Sub issues are many and are shown in the order of their numbers.

```sql
WITH  CTE_Issue AS
         (
           SELECT     Issue,
                      DENSE_RANK()OVER( ORDER BY COUNT(1) DESC) AS Issue_rank
           FROM       dbo.FinConsumerComplaints
           GROUP BY   Issue
         ),  

      CTE_Sub_Issue AS

         (
           SELECT     Sub_issue,
                      DENSE_RANK()OVER( ORDER BY COUNT(1)DESC ) AS Sub_Issue_rank
           FROM       dbo.FinConsumerComplaints
           GROUP BY   Sub_issue
    
          )

		   SELECT     Issue,
		              Issue_rank,
                              Sub_Issue,
                              Sub_Issue_rank
		  FROM        CTE_Issue
		  CROSS JOIN   CTE_Sub_Issue
		  WHERE  Issue_rank <= 5
		  AND    Sub_Issue_rank <= 5;
```
![SQLTWO](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/209326f8-beb1-4fd3-ac5c-add5dc6c63cc)

##### What is the company's typical response to consumer complaints?

According to the analysis code  below the company's typical response to consumer complaints are  closed with an explanation and  closed with monetary relief
    
```sql
    SELECT      Company_response_to_consumer,
                DENSE_RANK()OVER( ORDER BY COUNT(1) DESC) AS company_response_rank
    FROM        dbo.FinConsumerComplaints
    GROUP BY    Company_response_to_consumer;
```
![SQLTHREE](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/234b9c5d-bd1c-4a35-9c19-b627f4cebf52)

##### How does the timely response rate vary across different states?
The timely response rate was found between  92% and 100% with States like NH having 100% and ND state having the lowest rate of 92.59%
  
```sql
WITH CTE_response_rate AS
          (
          SELECT     State,
                     COUNT(Timely_response) AS  Timely_res_count,
                     SUM(CAST(Timely_response AS INT)) AS total_Timely_res
          FROM       dbo.FinConsumerComplaints
          GROUP BY   State
          
           )

SELECT     State,
           ROUND(CAST( total_Timely_res AS REAL) * 100 / CAST(Timely_res_count AS REAL),2)  AS response_percentage
FROM       CTE_response_rate
ORDER BY   response_percentage DESC;

```
![SQLFOUR](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/ef84d408-285a-466e-9074-58305c54e35b)

##### What is the proportion of complaints that were disputed by consumers?
  
``` sql
 
 WITH  CTE_proportition AS 
(
SELECT      Count(Consumer_disputed) AS dispute_count
FROM        dbo.FinConsumerComplaints
GROUP  BY   Consumer_disputed
HAVING      Consumer_disputed LIKE 'Yes'
)
SELECT      ROUND( CAST(dispute_count AS real) *100/(SELECT  COUNT(*) FROM dbo.FinConsumerComplaints),2) AS  dispute_proportion
FROM        CTE_proportition;

```
![SQLFIVE](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/76c3fe53-4eeb-41f5-be46-a8a7a8c35f5a)


##### Which states have the highest and lowest  ZIP code values for complaints?
  
- maximum number of complaints
  
```sql

  WITH CTE_complaints_ZIP AS
(
SELECT        State,
              ZIP_code,
              Count(Complaint_ID) AS number_complaints
FROM          dbo.FinConsumerComplaints
GROUP   BY    State,
              ZIP_code
HAVING        ZIP_code IS NOT NULL
AND           State    IS NOT NULL
)

SELECT    TOP 1 State,
              ZIP_code,
              MAX(number_complaints) AS max_number_complaints
FROM          CTE_complaints_ZIP
GROUP BY      ZIP_code,
              State,
              number_complaints
ORDER BY      number_complaints DESC;

```
![SQLSIX A](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/3f209754-cd2c-418b-9540-2b38bc145ac6)

- Minimum number of complaints

``` sql
WITH CTE_complaints_ZIP AS
(
SELECT        State,
              ZIP_code,
              Count(Complaint_ID) AS number_complaints
 FROM         dbo.FinConsumerComplaints
GROUP   BY    State,
              ZIP_code
HAVING        ZIP_code IS NOT NULL
AND           State    IS NOT NULL
)

SELECT   TOP 1 State,
              ZIP_code,
              Min(number_complaints) AS min_number_complaints
FROM          CTE_complaints_ZIP
GROUP BY      ZIP_code,
              State,
              number_complaints
ORDER BY      number_complaints ASC;

```
![SQLSIX B](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/76d261bd-7087-4e94-874d-d0535ced6c1c)


##### Are there any noticeable trends or patterns in the submission method (e.g., web, referral) of complaints over time?

There are several noticeable trends in the submission methods used,  for instance, the web submission method has been rising since the year 2011 up to 2014 then it started dropping however it remained one of the most preferred methods. The referral method has been the second favoured which also has been on the rise since 2011 and the numbers started to fall again after 2014. The phone call method has been continuously rising since 2011 to 2020.
  
```sql
SELECT          Submitted_via,
                YEAR(Date_Sumbited) AS Year_submitted,
                COUNT(Submitted_via)submission_method_cnt
FROM            dbo.FinConsumerComplaints
GROUP BY        Submitted_via,
                YEAR(Date_Sumbited)
ORDER  BY       Year_submitted ASC,
                submission_method_cnt;
```
![SQLSEVEN](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/bf6d1e0a-7e82-4570-84c3-ff39f833db39)

##### What are the most common issues reported by consumers who did not provide consent for further actions?

Managing an account, problems with a purchase shown on your statement, attempts to collect debt not owed, and struggling to pay the mortgage form the most common issues reported by consumers.
  
```sql

SELECT         TOP 10 Consumer_consent_provided,
               Issue,
               COUNT(Issue) as issue_count,
               DENSE_RANK()OVER(PARTITION BY Consumer_consent_provided ORDER BY COUNT(Issue) DESC) AS rnk
FROM           dbo.FinConsumerComplaints
GROUP BY       Issue,
               Consumer_consent_provided
HAVING         Consumer_consent_provided LIKE 'Consent not provided';

```

![SQLEIGHT](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/20211f80-1044-4f12-8ee6-a993c64cffc7)

##### Is there a correlation between the company's response type and the presence of monetary relief in closed cases?

There is a strong correlation between the company's response type and the presence of monetary relief. The analysis shows that whenever the company is not providing public response a monetary relief is being proposed. 
  
```sql

SELECT        Company_public_response,
              Company_response_to_consumer
FROM          dbo.FinConsumerComplaints
GROUP BY      Company_public_response,
              Company_response_to_consumer
HAVING        Company_response_to_consumer LIKE 'Closed with monetary relief'
AND           Company_public_response IS NOT NULL;

```
![SQLNINE](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/8aa87203-2fd8-4210-b398-f6a0edbf11f6)


##### For closed cases with monetary relief, what types of issues were commonly associated with such resolutions?

Managing an account, deposits and withdrawals, billing disputes, problems caused by my funds being low and  problems with a purchase shown on your statement are the most common issues associated with monetary relief.
  
```sql

SELECT       TOP 10 Company_response_to_consumer,
             Issue,
             COUNT(Issue) AS issue_cnt,
             DENSE_RANK() OVER (PARTITION BY Company_response_to_consumer ORDER BY COUNT(Issue) DESC) AS issue_rank
FROM         dbo.FinConsumerComplaints
GROUP BY     Issue,
             Company_response_to_consumer
HAVING       Company_response_to_consumer LIKE 'Closed with monetary relief';
```
![SQLTEN](https://github.com/richardmukechiwa/SQL_Project/assets/131812176/9e0049ad-1033-495e-9436-f55a8eca7853)

## Thank you for your time

			   
  


  


