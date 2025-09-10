# Personal Project
# company_sales_analysis

## Overview
In this project, I go through a company's sales data and analyse it's performance across different years (2011-2013). I do:
1. [Data Cleaning](#data-cleaning), SQL
2. [Data Analysis/Exploration](#data-analysisexploration), SQL & Power BI
3. [Data Visualization](#data-visualization), Power BI

## Data Cleaning
- I start by first creating a copy of the original file so that we have the raw data still intact throughout our analysis.
```sql
CREATE TABLE company_sales
LIKE sales;
INSERT INTO company_sales
SELECT * FROM sales;

-- Data successfully copied 
SELECT * FROM company_sales;
```
<img src="Assets/company_sales table successful creation.png" alt="company sales copy table created">

- I then proceed and start by looking for any duplicates.
  - To do this effectively and have consistency through the rest of the analysis, I create an "id" column in the copy table using data present in all rows in the dataset. I delete 1000 duplicate rows from the dataset.
  ```sql
  ALTER TABLE company_sales
  ADD COLUMN id INT AUTO_INCREMENT PRIMARY KEY;

  WITH sales_dups AS
  (SELECT  *,
  ROW_NUMBER() OVER(PARTITION BY
  `company_sales`.`Date`,
  `company_sales`.`Day`,
  `company_sales`.`Month`,
  `company_sales`.`Year`,
  `company_sales`.`Customer_Age`,
  `company_sales`.`Age_Group`,
  `company_sales`.`Customer_Gender`,
  `company_sales`.`Country`,
  `company_sales`.`State`,
  `company_sales`.`Product_Category`,
  `company_sales`.`Sub_Category`,
  `company_sales`.`Product`,
  `company_sales`.`Order_Quantity`,
  `company_sales`.`Unit_Cost`,
  `company_sales`.`Unit_Price`,
  `company_sales`.`Profit`,
  `company_sales`.`Cost`,
  `company_sales`.`Revenue`) AS occurence
  FROM `bike_sales`.`company_sales`)

  DELETE FROM company_sales
  WHERE id IN(SELECT id FROM sales_dups WHERE occurence > 1)
  ;
  ```
- Data Standardization
  - I convert "M"s and "F"s in the gender column to Male and Female.
  ```sql
  UPDATE company_sales
  SET Customer_Gender = "Male"
  WHERE Customer_Gender = "M"; 

  UPDATE company_sales
  SET Customer_Gender = "Female"
  WHERE Customer_Gender = "F"; 
  ```
  <img src="Assets/customer_gender before standardization.png" alt="customer_gender before standardization"><img src="Assets/customer_gender after standardization.png" alt="customer_gender after standardization">
  - I clean the company finacials columns that had the wrong data so that they read the right amounts.
  ```sql
  -- Updating the company revenue metrics columns
  -- Notes:
    -- Profit = (Unit_Price * Order_Quantity) - (Unit_Cost * Order_Quantity)
    -- Cost = Unit_Cost * Order_Quantity
    -- Revenue = Unit_Price * Order_Quantity

  -- Checking if the above metrics match the above and updating where not
  WITH sales_metrics AS
  (SELECT id, Profit, (Unit_Price * Order_Quantity) - (Unit_Cost * Order_Quantity) AS profit_ 
  FROM company_sales)

  UPDATE company_sales tb1
  JOIN sales_metrics tb2
    ON tb1.id = tb2.id
  SET tb1.Profit = tb2.profit_
  WHERE tb1.id = tb2.id AND tb1.Profit != tb2.profit_
  ;

  WITH sales_metrics AS
  (SELECT id, Cost, (Unit_Cost * Order_Quantity) AS cost_ 
  FROM company_sales)

  UPDATE company_sales tb1
  JOIN sales_metrics tb2
    ON tb1.id = tb2.id
  SET tb1.Cost = tb2.cost_
  WHERE tb1.id = tb2.id AND tb1.Cost != tb2.cost_company_salescompany_sales
  ;

  WITH sales_metrics AS
  (SELECT id, Revenue, (Unit_Price * Order_Quantity) AS rev 
  FROM company_sales)

  UPDATE company_sales tb1
  JOIN sales_metrics tb2
    ON tb1.id = tb2.id
  SET tb1.Revenue = tb2.rev
  WHERE tb1.id = tb2.id AND tb1.Revenue != tb2.rev
  ;
  ```
- I later export the file to my local drive as a csv file for dashboard creation using Power BI.
```sql
SELECT *
FROM company_sales
INTO OUTFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/company_sales.csv'
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

## Data Analysis/Exploration
I go through the data and get insights such as:
1. No. of:
- Countries the data is representing
- Product Categories(3), Sub-Categories(17) and actual Product Count(130).
- Transactions Count (Over 112K)
- Items sold (Over 1M)
- Years the data is covering (6 Years)
2. Total Profit over the years (~42M)

`No. 1 & 2 Above Rep.`
<img src="Assets/Various Data Metrics.png" alt="Various Metrics in the Dataset">

3. The year with the highest sales is 2015. Years 2014 and 2016 have incomplete data. The data for the two years goes to July.
  - 2016 had a chance at topping the sales by profit if all the data for the year was provided.
  ```sql
  SELECT `Year`, SUM(Profit) AS Profit
  FROM company_sales
  GROUP BY `Year`
  ORDER BY Profit DESC;
  ```
  <img src="Assets/profit over the years.png" alt="profit over the years">
4. December emerges as the top performing month, by profit generated over the years (except for the 2 years where the data was not provided).
  - Position 2 and 3 interms of performance cchanges between the pairs July & August and November & October over the years.
  <img src="Assets/profit by month over the years.png" alt="profit by month over the years"> <img src="Assets/profit by month in all years.png" alt="top 3 best months every year by profit">
5. Biggest number of purchases originates from customers aged between 25-35 (34%) followed by 35-44 (28%).
<img src="Assets/customer count by age group.png" alt="customer count by age groups">
6. Country Ranks:
  - USA leads in terms of transactions count and is followed by Australia
  - USA leads in terms of total items ordered and is followed by Australia
  - USA leads in terms of total profit generated and is followed by Australia.

  ```sql
  SELECT Country,
  COUNT(*) count,
  COUNT(*) * 100/(SELECT COUNT(*) FROM company_sales) AS percentage
  FROM company_sales
  GROUP BY Country
  ORDER BY percentage DESC; -- United states is the highest by transactions count, followed by Australia
  -- By Products Count, ordered products
  SELECT Country,
  SUM(Order_Quantity),
  SUM(Order_Quantity) * 100/(SELECT SUM(Order_Quantity) FROM company_sales) AS percentage
  FROM company_sales
  GROUP BY Country
  ORDER BY percentage DESC; -- United States still leads and it's still followed by Australia
  -- By Profit
  SELECT Country,
  SUM(Profit) as Profit_,
  SUM(Profit) * 100/(SELECT SUM(Profit) FROM company_sales) AS percentage
  FROM company_sales
  GROUP BY Country
  ORDER BY percentage DESC; -- Same case as the previous analysis
  ```
  <img src="Assets/rank by transactions.png" alt="country rank by transaction count"> <img src="Assets/rank by quantity ordered in every transaction.png" alt="country rank by items ordered"> <img src="Assets/rank by profit generated.png" alt="country rank by profit">

7.  Items Purchase Analysis:
    - Accessories are the most purchased by count or items ordered
    - Bikes lead in terms of profit generated then followed by accessories
    
  ```sql
  SELECT Product_Category,
         COUNT(*) count,
         COUNT(*) * 100/(SELECT COUNT(*) FROM company_sales) AS percentage
  FROM company_sales
  GROUP BY Product_Category; -- Accessories are the mostly purchased items followed by Bikes, then Clothing
  
  -- Highest purchased by item count
  SELECT Product_Category,
         SUM(Order_Quantity),
         SUM(Order_Quantity) * 100/(SELECT SUM(Order_Quantity) FROM company_sales) AS percentage
  FROM company_sales
  GROUP BY Product_Category; -- Accessories are the ones that are majorly bough by quantity then followed by Clothing and then the bikes
  
  -- Highest by Profit
  SELECT Product_Category,
         SUM(Profit),
         SUM(Profit) * 100/(SELECT SUM(Profit) FROM company_sales) AS percentage
  FROM company_sales
  GROUP BY Product_Category
  ORDER BY percentage DESC;
  ```
  <img src="Assets/purchased items rank.png" alt="item categories ranks">
  <img src="Assets/Leading items by profit.png" alt="leading items by profit">

    
   
## Data Visualization
- Using Power BI, a dashboard displaying [metrics mentioned in the Data Analysis/Exploration](#data-analysisexploration) and more is created.
- The dashboard highlights other insights such as:
  - Unit Price vs Unit Cost
  - Product Performance over the years
  - Monthly Profit breakdown
  - Purchases by gender
  - Ages Bins showing the ages with the highest number of trasactions etc.
 
- I also provide filters to navigate through the data.
<img src="Assets/sales analysis db.png" alt="sales analysis dashboard">
<img src="Assets/customer demographics db.png" alt="customer demographics dashboard">

Tools Used: SQL, Power BI
  

