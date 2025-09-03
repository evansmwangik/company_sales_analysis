# Personal Project
# company_sales_analysis

## Overview
In this project, I go through a company's sales data and analyse it's performance across different years (2011-2013). I do:
1. [Data Cleaning](#data-cleaning), SQL
2. [Data Analysis/Exploration](#data-analysisexploration), SQL & Power BI
3. [Data Visualization](#data-visualization), Power BI

## Data Cleaning
- I start by first creating a copy of the original file so that we have the raw data still intact throughout our analysis.
- I then proceed and start by looking for any duplicates.
  - To do this effectively and have consistency through the rest of the analysis, I create an "id" column in the copy table using data present in all rows in the dataset.
- Data Standardization
  - I convert "M"s and "F"s in the gender column to Male and Female.
  - I clean the company finacials columns that had the wrong data so that they read the right amounts.
- I later export the file to my local drive as a csv file for dashboard creation using Power BI.

## Data Analysis/Exploration
I go through the data and get insights such as:
1. No. of:
- Countries the data is representing
- Product Categories(3), Sub-Categories(17) and actual Product Count(130).
- Transactions Count (Over 112K)
- Items sold (Over 1M)
- Years the data is covering (6 Years)
2. Total Profit over the years (~42M)
3. The year with the highest sales is 2015. Years 2014 and 2016 have incomplete data. The data for the two years goes to July.
  - 2016 had a chance at topping the sales by profit if all the data for the year was provided.
4. December emerges as the top performing month, by profit generated over the years (except for the 2 years where the data was not provided).
  - Position 2 and 3 interms of performance cchanges between the pairs July & August and November & October over the years.
5. Biggest number of purchases originates from customers aged between 25-35 (34%) followed by 35-44 (28%).
6. Country Ranks:
  - USA leads in terms of transactions count and is followed by Australia
  - USA leads in terms of total items ordered and is followed by Australia
  - USA leads in terms of total profit generated and is followed by Australia
7.  Items Purchase Analysis:
    - Accessories are the most purchased by count or items ordered
    - Bikes leads in terms of profit generated then followed by accessories
   
## Data Visualization
- Using Power BI, a dashboard displaying [metrics mentioned in the Data Analysis/Exploration](#data-analysis) and more is created.
- The dashboard highlights other insights such as:
  - Unit Price vs Unit Cost
  - Product Performance over the years
  - Monthly Profit breakdown
  - Purchases by gender
  - Ages Bins showing the ages with the highest number of trasactions etc.
 
- I also provide filters to navigate through the data.

Tools Used: SQL, Power BI
  

