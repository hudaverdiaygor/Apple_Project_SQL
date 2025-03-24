

## Entity Relationship Diagram (ERD)

![ERD](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/erd.png)



## **Database Setup & Design**

### **Schema Structure**

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).


```sql
DROP TABLE IF EXISTS warranty;
DROP TABLE IF EXISTS sales;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS category;
DROP TABLE IF EXISTS stores;

CREATE TABLE stores(
store_id VARCHAR(5) PRIMARY KEY,
store_name	VARCHAR(30),
city	VARCHAR(25),
country VARCHAR(25)
);

DROP TABLE IF EXISTS category;
CREATE TABLE category
(category_id VARCHAR(10) PRIMARY KEY,
category_name VARCHAR(20)
);

CREATE TABLE products
(
product_id	VARCHAR(10) PRIMARY KEY,
product_name	VARCHAR(35),
category_id	VARCHAR(10),
launch_date	date,
price FLOAT,
CONSTRAINT fk_category FOREIGN KEY (category_id) REFERENCES category(category_id)
);

CREATE TABLE sales
(
sale_id	VARCHAR(15) PRIMARY KEY,
sale_date	DATE,
store_id	VARCHAR(10), -- this fk
product_id	VARCHAR(10), -- this fk
quantity INT,
CONSTRAINT fk_store FOREIGN KEY (store_id) REFERENCES stores(store_id),
CONSTRAINT fk_product FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE warranty
(
claim_id VARCHAR(10) PRIMARY KEY,	
claim_date	date,
sale_id	VARCHAR(15),
repair_status VARCHAR(15),
CONSTRAINT fk_orders FOREIGN KEY (sale_id) REFERENCES sales(sale_id)
);

```
## Objectives

The project is split into three tiers of questions to test SQL skills of increasing complexity:

### Easy to Medium (10 Questions)

1. Find the number of stores in each country.
2. Calculate the total number of units sold by each store.
3. Identify how many sales occurred in December 2023.
4. Determine how many stores have never had a warranty claim filed.
5. Calculate the percentage of warranty claims marked as "Warranty Void".
6. Identify which store had the highest total units sold in the last year.
7. Count the number of unique products sold in the last year.
8. Find the average price of products in each category.
9. How many warranty claims were filed in 2020?
10. For each store, identify the best-selling day based on highest quantity sold.

### Medium to Hard (5 Questions)

11. Identify the least selling product in each country for each year based on total units sold.
12. Calculate how many warranty claims were filed within 180 days of a product sale.
13. Determine how many warranty claims were filed for products launched in the last two years.
14. List the months in the last three years where sales exceeded 5,000 units in the USA.
15. Identify the product category with the most warranty claims filed in the last two years.

### Complex (5 Questions)

16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
17. Analyze the year-by-year growth ratio for each store.
18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.

### Bonus Question

- Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.

## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.


## **Solving Business Problems**

### Solutions Implemented:
1. Find the number of stores in each country

```sql
SELECT COUNT(*) AS num_of_stores,country 
FROM stores
GROUP BY country
ORDER BY 1 DESC;
```
2. Calculate the total number of units sold by each store.

```sql
SELECT SUM(sa.quantity),store_name
FROM stores AS st
JOIN sales AS sa
ON st.store_id=sa.store_id
GROUP BY 2
ORDER BY 1 DESC;
```
3. Identify how many sales occured in December 2023. 

```sql
SELECT COUNT(*) AS num_of_sales 
FROM sales
WHERE sale_date BETWEEN '2023-12-01' AND '2023-12-31';
```

4.Determine how many stores have never had a warrant claim filed.

```sql
SELECT COUNT(DISTINCT store_name) - (
SELECT COUNT(DISTINCT store_name)
FROM stores AS st
JOIN sales AS sa
ON st.store_id=sa.store_id
JOIN warranty AS w
ON w.sale_id=sa.sale_id) AS num_of_stores_no_warranty
FROM stores
;

```
5.Calculate the percentage of warranty claims marked as 'Warranty Void'.

```sql
SELECT ROUND(100*(SELECT COUNT(*) FROM warranty
WHERE repair_status = 'Warranty Void') / (SELECT COUNT(*) FROM warranty)::numeric,2) AS percent_warranty_void;
;

```
6.Identify which store had the highest total units sold in the last year.

```sql
SELECT store_id,SUM(quantity) AS total_units_sold
FROM sales
WHERE sale_date >= CURRENT_DATE - INTERVAL '1 year' 
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;

```

7.Count the number of unique products sold in the last year.

```sql
SELECT COUNT(DISTINCT product_id) 
FROM sales
WHERE sale_date >= CURRENT_DATE - INTERVAL '1 year' ;

```

8.Find the average price of products in each category;

```sql
SELECT c.category_name,AVG(p.price) 
FROM category AS c
JOIN products AS p
ON c.category_id = p.category_id
GROUP BY 1
ORDER BY 2 DESC;
```

9.How many warranty claims were filed in 2020?

```sql
SELECT COUNT(*) AS num_of_claims_2020
FROM warranty
WHERE EXTRACT(YEAR FROM claim_date) = 2020 ;
```
10.For each store, identify the best selling day based on highest quantity sold. 

```sql
SELECT store_id,dow,num_of_quantity_sold
FROM(
SELECT store_id,
TO_CHAR(sale_date,'Day') AS dow,
SUM(quantity) AS num_of_quantity_sold,
RANK() OVER(PARTITION BY store_id ORDER BY SUM(quantity) DESC ) AS rank
FROM sales
GROUP BY 1,2
ORDER BY 1,3 DESC)
WHERE rank=1;
```
11.Identify the least selling product in each country in each year based on total units sold. 

```sql
SELECT product_id,country,extract AS year,sum AS total_sale FROM(
SELECT product_id,country,EXTRACT(YEAR FROM sale_date),SUM(quantity),RANK()OVER(PARTITION BY country,EXTRACT(YEAR FROM sale_date) ORDER BY SUM(quantity) ASC)
FROM sales AS sa
JOIN stores AS st
ON sa.store_id=st.store_id
GROUP BY 1,2,3
ORDER BY 2,3,4)
WHERE rank = 1;
```
12.Calculate how many warranty claims were filed within 180 days of a product sale.

```sql
SELECT COUNT(*) AS num_of_claims
FROM warranty AS w
JOIN sales AS sa
ON w.sale_id=sa.sale_id
WHERE claim_date <= sale_date + INTERVAL '180 days';
```
13.Determine how many warranty claims were filed for each product launched in the last two years. 

```sql
SELECT p.product_name,COUNT(w.claim_id) AS num_of_sales
FROM warranty AS w
JOIN sales AS sa
ON w.sale_id=sa.sale_id
JOIN products AS p
ON p.product_id=sa.product_id
WHERE launch_date>= CURRENT_DATE - INTERVAL '2 years'
GROUP BY 1
ORDER BY 2 DESC;
```
14.List the months in the last three years where the sales exceeded 5000 units in the USA. 


```sql
SELECT TO_CHAR(sale_date,'MM-YYYY') AS month,SUM(sa.quantity) AS num_of_sales
FROM stores AS st
JOIN sales AS sa
ON st.store_id=sa.store_id
WHERE st.country ='USA' AND sa.sale_date >= CURRENT_DATE- INTERVAL '3 years'
GROUP BY 1
HAVING SUM(sa.quantity) > 5000
ORDER BY 2 DESC;
```
15.Identify the product category with the most warranty claims filed in the last two years. 


```sql
SELECT c.category_id,c.category_name,COUNT(w.claim_id) AS num_of_claims
FROM warranty AS w
JOIN sales AS sa
ON w.sale_id=sa.sale_id
JOIN products AS p
ON p.product_id=sa.product_id
JOIN category AS c
ON p.category_id=c.category_id
WHERE w.claim_date >= CURRENT_DATE - INTERVAL '2 years'
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 1;

```
16.Determine the percentage chance of receiving warranty claims after each purchase for each country. 


```sql
SELECT
country,
	total_unit_sold,
	total_claim,
	COALESCE(total_claim::numeric/total_unit_sold::numeric * 100, 0)
	AS risk
FROM
(SELECT 
	st.country,
	SUM(s.quantity) AS total_unit_sold,
	COUNT(w.claim_id) AS total_claim
FROM sales AS s
JOIN stores AS st
ON s.store_id = st.store_id
LEFT JOIN 
warranty AS w
ON w.sale_id = s.sale_id
GROUP BY 1) t1
ORDER BY 4 DESC;

```
17.Analyze year by year growth for each store.  


```sql
WITH cte AS(
SELECT sa.store_id,st.store_name,EXTRACT(YEAR FROM sale_date) AS year,SUM(sa.quantity*p.price) AS total_sale
FROM sales AS sa
JOIN products AS p
ON sa.product_id=p.product_id
JOIN stores AS st
ON st.store_id=sa.store_id
GROUP BY 1,2,3
ORDER BY 1,2,3
)
SELECT store_name,year,total_sale AS current_year_sale ,100*(total_sale - LAG(total_sale ,1)OVER(PARTITION BY store_name ORDER BY year))/(LAG(total_sale ,1)OVER(PARTITION BY store_name)) AS perc_diff
FROM cte;

```

18.Calculate the correlation between product price and warranty claims for products sold in the last five years,segmented by price range.

```sql
SELECT 
	CASE
		WHEN p.price < 500 THEN 'Less Than $500'
		WHEN p.price BETWEEN 500 AND 1000 THEN '$500-$1000'
		ELSE 'More Than $1000'
	END AS price_segment,
	COUNT(w.claim_id) AS total_Claim
FROM warranty AS w
LEFT JOIN
sales AS s
ON w.sale_id = s.sale_id
JOIN 
products AS p
ON p.product_id = s.product_id
WHERE claim_date >= CURRENT_DATE - INTERVAL '5 year'
GROUP BY 1;

```

19. Identify the store with the highest percentage of 'Paid Repaired' claims relative to total claims filed. 

```sql
WITH paid_repair
AS
(SELECT 
	s.store_id,
	COUNT(w.claim_id) as paid_repaired
FROM sales as s
RIGHT JOIN warranty as w
ON w.sale_id = s.sale_id
WHERE w.repair_status = 'Paid Repaired'
GROUP BY 1
),

total_repaired
AS
(SELECT 
	s.store_id,
	COUNT(w.claim_id) as total_repaired
FROM sales as s
RIGHT JOIN warranty as w
ON w.sale_id = s.sale_id
GROUP BY 1)

SELECT 
	tr.store_id,
	st.store_name,
	pr.paid_repaired,
	tr.total_repaired,
	ROUND(pr.paid_repaired::numeric/
			tr.total_repaired::numeric * 100
		,2) as percentage_paid_repaired
FROM paid_repair as pr
JOIN 
total_repaired tr
ON pr.store_id = tr.store_id
JOIN stores as st
ON tr.store_id = st.store_id

```


20.Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.

```sql
WITH monthly_sales
AS
(SELECT 
	store_id,
	EXTRACT(YEAR FROM sale_date) as year,
	EXTRACT(MONTH FROM sale_date) as month,
	SUM(p.price * s.quantity) as total_revenue
FROM sales as s
JOIN 
products as p
ON s.product_id = p.product_id
GROUP BY 1, 2, 3
ORDER BY 1, 2,3
)
SELECT 
	store_id,
	month,
	year,
	total_revenue,
	SUM(total_revenue) OVER(PARTITION BY store_id ORDER BY year, month) as running_total
FROM monthly_sales
```
21.Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.

```sql
SELECT 
	p.product_name,
	CASE 
		WHEN s.sale_date BETWEEN p.launch_date AND p.launch_date + INTERVAL '6 month' THEN '0-6 month'
		WHEN s.sale_date BETWEEN  p.launch_date + INTERVAL '6 month'  AND p.launch_date + INTERVAL '12 month' THEN '6-12' 
		WHEN s.sale_date BETWEEN  p.launch_date + INTERVAL '12 month'  AND p.launch_date + INTERVAL '18 month' THEN '6-12'
		ELSE '18+'
	END as plc,
	SUM(s.quantity) as total_qty_sale
	
FROM sales as s
JOIN products as p
ON s.product_id = p.product_id
GROUP BY 1, 2
ORDER BY 1, 3 DESC 
```


