## SQL_Project
### Analyzing a financial organization's complaints dataset 

A good example of a financial organization in this analysis is a bank, for every bank customer service is one of the key fundamentals to survive the competition and also to realize profits. Customers bring forward issues and complaints on different products of the bank. Some of the complaints can cause damage to the reputation of the bank if they are not addressed in time and satisfactorily. In this project, we are going to analyze a financial organization's complaint dataset to understand the distribution, response effectiveness, and regional patterns, to derive actionable insights for informed decision-making and enhancing customer satisfaction. The dataset was extracted from Real World Fake Data's data. world platform. We are going to look at the following tasks...

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


- Checking the data in the table

```sql
SELECT  *   FROM  dbo.FinConsumerComplaints;
```
- Checking for duplicates in the table
  
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

- Data Cleaning steps

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

### Financial Complaints Analysis
- What is the distribution of complaints across different products and sub-products?
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

-  Which issues and sub-issues are most commonly reported by consumers?
  
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

  - What is the company's typical response to consumer complaints?
    
```sql
    SELECT      Company_response_to_consumer,
                DENSE_RANK()OVER( ORDER BY COUNT(1) DESC) AS company_response_rank
    FROM        dbo.FinConsumerComplaints
    GROUP BY    Company_response_to_consumer;
```

- How does the timely response rate vary across different states?
  
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

-  What is the proportion of complaints that were disputed by consumers?
  
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
- Which states have the highest and lowest  ZIP code values for complaints?
  
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
              Min(number_complaints) AS max_number_complaints
FROM          CTE_complaints_ZIP
GROUP BY      ZIP_code,
              State,
			  number_complaints
ORDER BY      number_complaints ASC;

```
- Are there any noticeable trends or patterns in the submission method (e.g., web, referral) of complaints over time?
  
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

- What are the most common issues reported by consumers who did not provide consent for further actions?
  
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
- Is there a correlation between the company's response type and the presence of monetary relief in closed cases?
  
```sql

SELECT        Company_public_response,
              Company_response_to_consumer
FROM          dbo.FinConsumerComplaints
GROUP BY      Company_public_response,
              Company_response_to_consumer
HAVING        Company_response_to_consumer LIKE 'Closed with monetary relief'
AND           Company_public_response IS NOT NULL;

```
- For closed cases with monetary relief, what types of issues were commonly associated with such resolutions?
  
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

## Thank you for your time

			   
  


  


