## SQL_Project
### Analyzing a financial organization's complaints dataset 

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

SELECT    Count( * ) AS number_of_rows
FROM  count_rows
WHERE   row_count = 1;

- Data Cleaning steps
  UPDATE  dbo.FinConsumerComplaints
SET   Sub_product = 'NULL'
WHERE Sub_product = '""';


UPDATE dbo.FinConsumerComplaints
SET     Sub_issue = 'NULL'
WHERE   Sub_issue = '""'; 

UPDATE dbo.FinConsumerComplaints
SET    Consumer_disputed = 'NULL'
WHERE   Consumer_disputed = 'N/A';


### Financial Complaints Analysis
- 1 What is the distribution of complaints across different products and sub-products?

   WITH cte_product AS
          (
            SELECT     Product,
            DENSE_RANK() OVER(ORDER BY COUNT(1) DESC) AS rank
            FROM       dbo.FinConsumerComplaints
            GROUP BY   Product
           ),

    cte_Sub_product AS
         (
           SELECT     Sub_product,
           DENSE_RANK() OVER(ORDER BY COUNT(1) DESC) AS s_rnk
           FROM       dbo.FinConsumerComplaints
           GROUP BY   Sub_product
		  
          )

SELECT     DISTINCT product,
           rank,
		   Sub_product,
		   s_rnk
FROM       cte_product
CROSS JOIN cte_Sub_product;
```
- 2 Which issues and sub-issues are most commonly reported by consumers?
  
```
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
		   GROUP BY  Sub_issue
    
          )

		   SELECT      Issue,
		              Issue_rank,
					  Sub_Issue,
					  Sub_Issue_rank
		  FROM         CTE_Issue
		  CROSS JOIN   CTE_Sub_Issue
		  WHERE  Issue_rank <= 5
		  AND    Sub_Issue_rank <= 5;
```
  - 3 What is the company's typical response to consumer complaints?
```
    SELECT      Company_response_to_consumer,
                DENSE_RANK()OVER( ORDER BY COUNT(1) DESC) AS company_response_rank
    FROM        dbo.FinConsumerComplaints
    GROUP BY    Company_response_to_consumer;
```
- 4 How does the timely response rate vary across different states?
```
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
- 5 What is the proportion of complaints that were disputed by consumers?
```
 
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
- 6 Which states have the highest and lowest  ZIP code values for complaints?
  
- maximum number of complaints
```
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

SELECT   TOP 1 State,
              ZIP_code,
			  MAX(number_complaints) AS max_number_complaints
FROM          CTE_complaints_ZIP
GROUP BY      ZIP_code,
              State,
			  number_complaints
ORDER BY      number_complaints DESC;
```
- Minimum number of complaints

```
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

SELECT   TOP 1 State,
              ZIP_code,
			  Min(number_complaints) AS max_number_complaints
FROM          CTE_complaints_ZIP
GROUP BY      ZIP_code,
              State,
			  number_complaints
ORDER BY      number_complaints ASC;
```
- 7 Are there any noticeable trends or patterns in the submission method (e.g., web, referral) of complaints over time?
```
SELECT          Submitted_via,
                YEAR(Date_Sumbited) AS Year_submitted,
				COUNT(Submitted_via)submission_method_cnt
FROM            dbo.FinConsumerComplaints
GROUP BY        Submitted_via,
                YEAR(Date_Sumbited)
ORDER  BY       Year_submitted ASC,
                submission_method_cnt;
```
- 8 What are the most common issues reported by consumers who did not provide consent for further actions?
```
SELECT         TOP 10 Consumer_consent_provided,
               Issue,
               COUNT(Issue) as issue_count,
			   DENSE_RANK()OVER(PARTITION BY Consumer_consent_provided ORDER BY COUNT(Issue) DESC) AS rnk
FROM           dbo.FinConsumerComplaints
GROUP BY       Issue,
               Consumer_consent_provided
HAVING         Consumer_consent_provided LIKE 'Consent not provided';
```
- 9 Is there a correlation between the company's response type and the presence of monetary relief in closed cases?
```

SELECT        Company_public_response,
              Company_response_to_consumer
FROM          dbo.FinConsumerComplaints
GROUP BY      Company_public_response,
              Company_response_to_consumer
HAVING        Company_response_to_consumer LIKE 'Closed with monetary relief'
AND           Company_public_response IS NOT NULL;
```
- 10 For closed cases with monetary relief, what types of issues were commonly associated with such resolutions?
```
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

			   
  


  


