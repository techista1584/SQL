# SQL  
## Data Analysis using SQL  

---

## 1. Report on the Issue  

### Project Title  
**The Case of Yellevate’s Client Disputes**

### Problem  
Identify the causes of client disputes and come up with actionable strategies to either:  
- Reduce lost revenue  
- Reduce the number of client disputes  

---

## Data Analysis Goals  

### SQL  
- Processing time for invoices to be settled (average number of days, rounded to whole number)  
- Processing time for the company to settle disputes (average number of days, rounded to whole number)  
- Percentage of revenue lost from disputes (within two decimal places)  
- Country with the highest losses from lost disputes (in USD)  

### Excel  
- Check if delayed settlements are caused by disputes  
- Check if invoice amount is related to disputes  
- Check if number of days late is related to lost disputes  
- Analyze trend in number of disputes per quarter  
- Identify country with greatest number of disputes  
- Identify top customer IDs with the most disputes  

---

## Recommendations  
- Provide strategies to reduce lost revenue or number of client disputes  
- Recommend improvements in data structure for better analysis  

---

## Methodology  

### A. Creation of Table: `Yellevate_invoices`

#### 1. Create Table  
Methodology

A.	Creation of Table “Yellevate_invoices”
1.	'Create table Yellevate_invoices'  
CREATE TABLE IF NOT EXISTS Yellevate_invoices    
(country VARCHAR,customer_ID VARCHAR,invoice_number NUMERIC,invoice_date DATE,due_date DATE,invoice_amount_usd NUMERIC, disputed NUMERIC,dispute_lost NUMERIC,settled_date DATE,days_to_settle INTEGER,days_late INTEGER);  
SELECT * FROM yellevate_invoices;
<img width="951" height="547" alt="image" src="https://github.com/user-attachments/assets/99b1cc37-550c-4532-a1a7-96e426c97f10" />
<br><br>

B.	Data Cleansing

--Check if when dispute_lost column has value of 1, then value of disputed column should also be 1  
SELECT * FROM yellevate_invoices  
WHERE disputed = 0 AND dispute_lost = 1;  
<img width="1015" height="296" alt="image" src="https://github.com/user-attachments/assets/a292f495-8428-41c6-b1ca-928116436010" />
<br><br>

--Replace 0/1 value of disputed to words
SELECT country,customer_id,invoice_number,invoice_date,due_date,invoice_amount_usd,
CASE
	WHEN disputed = 1 THEN 'yes'
	ELSE 'no'
END AS disputed2,
CASE
	WHEN dispute_lost = 1 THEN 'lost'
	WHEN disputed = 1 AND dispute_lost = 0 THEN 'win'
	WHEN disputed = 0 AND dispute_lost = 0 THEN 'no dispute'
	ELSE NULL
	END AS dispute_lost2,
settled_date,days_to_settle,days_late
FROM yellevate_invoices;
<img width="1015" height="765" alt="image" src="https://github.com/user-attachments/assets/03ae2cca-e657-417c-9ac9-012de603bbf1" />
<br><br>

--Check for misspelled country'
SELECT DISTINCT country FROM yellevate_invoices;
Result: no misspelled countries<br><br>
<img width="199" height="196" alt="image" src="https://github.com/user-attachments/assets/1a41e433-bb69-4318-b2fa-d38b6fbf9d2c" />
<br><br>

--Check if days to settle and days late value is accurate thru DATEDIF’
SELECT days_to_settle,(settled_date - invoice_date)AS days_to_settle2
FROM yellevate_invoices
WHERE days_to_settle !=(settled_date - invoice_date);
<img width="779" height="252" alt="image" src="https://github.com/user-attachments/assets/dcaafd95-a381-46a3-b698-967c51534da4" />
<br><br>

--Check if total distinct invoice number per transaction is equal to the total number of rows which is 2466
<img width="965" height="320" alt="image" src="https://github.com/user-attachments/assets/eeed47d4-15a5-4b36-b20e-1070cf1f3aef" />
<br><br>

--Add column to identify if due date was settled on time or not

SELECT country,customer_id,invoice_number,invoice_date,due_date,invoice_amount_usd,
CASE
	WHEN disputed = 1 THEN 'yes'
	ELSE 'no'
END AS disputed2,
CASE
	WHEN dispute_lost = 1 THEN 'lost'
<img width="1169" height="711" alt="image" src="https://github.com/user-attachments/assets/d209437d-ebf4-41cc-938c-47c838a24a05" />
<br><br>

C.	Data Analysis in SQL

1.Get the processing time (ave no. of days rounded to whole no.)in which invoices are settled'

SELECT ROUND(AVG(days_to_settle),0) AS avg_days_to_settle_invoice
FROM yellevate_invoices;
<img width="968" height="601" alt="image" src="https://github.com/user-attachments/assets/12065160-8e85-4f9b-8644-5dedca7ffd0c" />
<br><br>

2. Get the processing time (ave no. of days rounded to whole no.)in which disputes are settled'
SELECT disputed,ROUND(AVG(days_to_settle),0)AS avg_days_to_settle_disputes FROM yellevate_invoices
WHERE disputed = 1
GROUP BY disputed;
<img width="1015" height="284" alt="image" src="https://github.com/user-attachments/assets/d15d41a2-3847-4dd9-9bd2-6a3644f54998" />
<br><br>

3. Get the percentage disputes received by the company that were lost (within 2 decimal places)'
SELECT
	SUM(dispute_lost)AS lost_dispute_total,
	SUM(disputed)as total_disputes,
	round((SUM(dispute_lost)/SUM(disputed))*100,2) percent
FROM yellevate_invoices;
<img width="919" height="291" alt="image" src="https://github.com/user-attachments/assets/71582cc4-fc82-491e-8575-858a64cd5abb" />
<br><br>

4. Get percentage of revenue lost from disputes (within two decimal places).
WITH total_invoice_amount AS (
	SELECT SUM(invoice_amount_usd)AS total_invoice_amount
	FROM yellevate_invoices),
invoice_amount_lost AS (
	SELECT SUM(invoice_amount_usd)AS invoice_amount_lost
	FROM yellevate_invoices
	WHERE dispute_lost = 1)
SELECT
	total_invoice_amount,invoice_amount_lost,
	ROUND(invoice_amount_lost / total_invoice_amount,4)*100 percent
FROM total_invoice_amount,invoice_amount_lost;
<img width="664" height="425" alt="image" src="https://github.com/user-attachments/assets/e68c7c6e-8210-4d27-8002-aa8e27a09749" />
<br><br>

5. Get the country where the company reached the highest losses from lost disputes(inUSD)
SELECT country,SUM(invoice_amount_usd)
FROM yellevate_invoices
WHERE dispute_lost = 1
GROUP BY country
ORDER BY sum(invoice_amount_usd) DESC;
<img width="1015" height="501" alt="image" src="https://github.com/user-attachments/assets/5165733d-b822-48d8-bce3-c1565c3d7e9c" />







