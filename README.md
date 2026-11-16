# **COZURA EMART: CUSTOMER VALUE & RFM SEGMENTATION ANALYSIS**

*Which customers are worth the most, which are slipping away, and where should marketing act first?*

**Prepared by:** Daniel C. Diala, Data & BI Analyst  
**Tools:** Power BI, Power Query, DAX

---

## Table of Contents

- [1. Project Background](#1-project-background)
  - [Company overview](#company-overview)
  - [Problem statement](#problem-statement)
  - [Objectives](#objectives)
  - [Analysis focus areas](#analysis-focus-areas)
- [2. Data Structure & Initial Checks](#2-data-structure--initial-checks)
- [3. Methodology: From Raw Orders to Customer Segments](#3-methodology-from-raw-orders-to-customer-segments)
- [4. Executive Summary](#4-executive-summary)
- [5. Insights Deep Dive](#5-insights-deep-dive)
- [6. Takeaway](#6-takeaway)
- [7. Recommendations](#7-recommendations)
- [8. Assumptions & Caveats](#8-assumptions--caveats)
- [9. Skills Demonstrated](#9-skills-demonstrated)

---

# 1. Project Background

## Company overview

Cozura EMart is an e-commerce store whose slogan is "Shop Smart, Live Better". It sells cycling bikes, components, clothing and accessories online to customers in **North America (United States, Canada), Europe (United Kingdom, Germany, France) and Australia**. The data covers internet sales from **1 July 2005 to 31 July 2008**: **27,659 orders from 18,484 customers worth \$29.36M**, which gives the business a solid 41% profit margin.

Because every sale is online and every order is tied to a customer record, the business holds a detailed purchase history for each customer. That history is the raw material for customer segmentation.

## Problem statement

Sales have grown quickly, but growth hides a retention problem. Customers have so far been treated as one large group, so the business cannot tell who its best customers are, who has quietly stopped buying, or which markets are losing customers fastest. Marketing money is therefore spread evenly rather than aimed at the customers and countries where it would have the most effect.

**Central question:** Which customers are the most valuable, which are at risk of being lost, and where should Cozura EMart focus its retention and marketing effort?

## Objectives

1.  Measure overall business performance: sales, cost, profit and margin, the best-selling products and the sales trend by year and by country.
2.  Profile the customer base by gender, marital status and age, and show how much each group contributes to sales.
3.  Segment every customer using RFM analysis (Recency, Frequency, Monetary value) into eight meaningful groups.
4.  Identify which segments and which countries are healthy and which are at risk.
5.  Give the marketing team a customer-level list and a recommended action for each segment.
6.  Turn the findings into clear, prioritised recommendations.

## Analysis focus areas

| **Focus area**            | **Question answered**                                 | **Dashboard page**    |
|---------------------------|-------------------------------------------------------|-----------------------|
| **Business performance**  | How much do we sell, how profitable are we and where? | Executive Overview    |
| **Customer profile**      | Who are our customers and who contributes the most?   | Customer Overview     |
| **Customer segmentation** | How healthy is the customer base according to RFM?    | Customer Segmentation |
| **Customer-level detail** | Which individual customers sit in each segment?       | Customer Details      |
| **Marketing activation**  | What should we do with each segment?                  | Marketing Actions     |


---

# 2. Data Structure & Initial Checks

The raw data is a star-schema extract with one fact table and five dimension tables. Only the fields needed for the analysis were kept, and the rest were removed in Power Query before modelling.

| **Table**             | **Raw size**             | **Kept**            | **What it holds**                                                            |
|-----------------------|--------------------------|---------------------|------------------------------------------------------------------------------|
| **FactInternetSales** | 60,398 rows x 26 columns | 14 columns          | One row per order line: keys, quantity, price, cost, sales amount and dates  |
| **DimCustomer**       | 18,484 rows x 29 columns | 8 columns           | One row per customer: name, birth date, gender, marital status, email, phone |
| **DimProduct**        | 606 rows x 36 columns    | 2 columns           | Product key and product name                                                 |
| **DimGeography**      | 655 rows x 11 columns    | 4 columns           | City, country and sales territory for each customer location                 |
| **DimDate**           | 2,191 rows x 19 columns  | 9 columns           | Calendar attributes (year, quarter, month, day names)                        |
| **DimSalesTerritory** | 11 rows x 6 columns      | 2 columns (10 rows) | Sales territory key and country                                              |

**Initial checks:** no duplicate order lines (60,398 unique) or duplicate customers (18,484 unique keys), every order links to a valid customer and product, and the data runs from 1 July 2005 to 31 July 2008.

### Data model (star schema)

<img width="1800" height="930" alt="data_model_star_schema" src="https://github.com/user-attachments/assets/10274c6b-0639-4ce0-bdb1-ade4cb9f0165" />


*Star schema used for the analysis. RFM Table and Segment Strategy were created in Power BI.*

*Data cleaning and transformation was done in Power Query to get the dataset ready and suitable for analysis. Details of data quality issues and mitigations are discussed in Section 8.*


---

# 3. Methodology: From Raw Orders to Customer Segments

## 3.1 What is RFM analysis?

RFM is a simple, widely used way of judging how valuable a customer is from their buying behaviour. It looks at three questions:

| **Letter**       | **Question**                              | **Why it matters**                                                                                       |
|------------------|-------------------------------------------|----------------------------------------------------------------------------------------------------------|
| **R: Recency**   | How recently did the customer last buy?   | Customers who bought recently are more likely to buy again than those who have not been seen for a year. |
| **F: Frequency** | How often did the customer buy?           | Customers who order repeatedly are more loyal than one-time buyers.                                      |
| **M: Monetary**  | How much did the customer spend in total? | Customers who spend more are worth more and deserve more attention.                                      |

Think of a local coffee shop. The person who came in yesterday, comes every week and orders the most expensive drinks is the owner's best customer. Someone who used to visit daily but has not been seen for six months is a customer to win back. Someone who bought one coffee two years ago is probably gone. RFM puts numbers on that instinct for every customer, so a business with thousands of customers can act on it.

## 3.2 Approach at a glance

| **Step** | **What was done**                                                    | **Output**                                                             |
|----------|----------------------------------------------------------------------|------------------------------------------------------------------------|
| 1        | Cleaned the data and built the star schema                           | Analysis-ready model (Section 2)                                       |
| 2        | Calculated Recency, Frequency and Monetary values for every customer | RFM Table with one row per customer                                    |
| 3        | Converted the raw values into scores from 1 (low) to 5 (high)        | Recency, Frequency and Monetary scores, plus the three-digit RFM Score |
| 4        | Combined the scores using fixed rules                                | Eight customer segments                                                |
| 5        | Grouped the segments into Doing Well and Needs Attention / At Risk   | Customer health KPIs                                                   |
| 6        | Attached a description and marketing action to each segment          | Segment Strategy table                                                 |
| 7        | Built the report pages                                               | Dashboard, customer list and marketing actions                         |

## 3.3 Step 1: Finding the RFM values

A **snapshot date** is needed to measure recency. The data ends on 31 July 2008, so the snapshot is set to one day after the last order in the dataset (**1 August 2008**). This is the "today" the analysis is measured from, and it is shown on the Customer Segmentation page as "Analysis As of: Aug. 1, 2008".

Three DAX measures calculate the raw values, and a calculated table called **RFM Table** applies them to each customer:

```dax
Snapshot Date   = CALCULATE(MAX(FactInternetSales[OrderDate]), ALL(FactInternetSales)) + 1
Last Order Date = MAX(FactInternetSales[OrderDate])

Recency Value   = DATEDIFF([Last Order Date], [Snapshot Date], DAY)
Frequency Value = CALCULATE(DISTINCTCOUNT(FactInternetSales[SalesOrderNumber]))
Monetary Value  = CALCULATE(SUM(FactInternetSales[SalesAmount]))

RFM Table =
ADDCOLUMNS(
    SUMMARIZE(FactInternetSales, FactInternetSales[CustomerKey]),
    "Recency",   [Recency Value],
    "Frequency", [Frequency Value],
    "Monetary",  [Monetary Value]
)
```

The result is one row for each of the 18,484 customers showing days since last order, number of orders and total spend. For example, a customer who last ordered on 5 June 2008, placed 3 orders and spent \$2,400 has a Recency of 57 days, a Frequency of 3 and a Monetary value of \$2,400.

## 3.4 Step 2: Turning values into RFM scores

Raw values are hard to compare, so each one is converted into a score from **1 to 5**, where 5 is best. Recency and Monetary use **quintiles** (the customer base is split into five equal groups using PERCENTILE.INC), so the thresholds adjust to the data:

| **Score** | **Recency (days since last order)**   | **Monetary (total spend)**   | **Frequency (orders)** |
|-----------|---------------------------------------|------------------------------|------------------------|
| **5**     | 73 days or fewer (most recent 20%)    | \$3,271.93 or more (top 20%) | 5 or more              |
| **4**     | 74 to 133 days                        | \$1,155.48 to \$3,271.92     | 4                      |
| **3**     | 134 to 206 days                       | \$91.97 to \$1,155.47        | 3                      |
| **2**     | 207 to 288 days                       | \$39.98 to \$91.96           | 2                      |
| **1**     | More than 288 days (least recent 20%) | Below \$39.98                | 1                      |

**Why Frequency uses fixed bands:** 62.9% of customers (11,619 of 18,484) have ordered only once, so splitting frequency into quintiles would give most customers the same value and the groups could not be separated. Fixed bands (1, 2, 3, 4 and 5 or more orders) were used instead. Only 95 customers (0.5%) have five or more orders.

```dax
Recency Score   = SWITCH(TRUE(),
    R <= PERCENTILE.INC(Recency, 0.2), 5,  R <= PERCENTILE.INC(Recency, 0.4), 4,
    R <= PERCENTILE.INC(Recency, 0.6), 3,  R <= PERCENTILE.INC(Recency, 0.8), 2,  1)

Monetary Score  = SWITCH(TRUE(),
    M >= PERCENTILE.INC(Monetary, 0.8), 5, M >= PERCENTILE.INC(Monetary, 0.6), 4,
    M >= PERCENTILE.INC(Monetary, 0.4), 3, M >= PERCENTILE.INC(Monetary, 0.2), 2, 1)

Frequency Score = SWITCH(TRUE(), F >= 5, 5, F = 4, 4, F = 3, 3, F = 2, 2, 1)

RFM Score   = Recency Score & Frequency Score & Monetary Score      -- e.g. "535"
FM AV.Score = ROUND((Frequency Score + Monetary Score) / 2, 0)
```

The three scores are joined into a three-digit **RFM Score** (for example 535 means Recency 5, Frequency 3, Monetary 5). Frequency and Monetary are then averaged into a single **FM AV.Score**, because both describe how much value a customer brings, while Recency describes whether they are still active.

## 3.5 Step 3: Segmenting customers

Each customer is placed in one of eight segments by testing the rules below **from top to bottom**. The first rule that matches decides the segment.

| **Segment**             | **Rule (R = Recency score, FM = FM AV.Score)** | **Customers** | **% of customers** | **In plain words**                         |
|-------------------------|------------------------------------------------|---------------|--------------------|--------------------------------------------|
| **Champions**           | R is 4 or 5 and FM is 4 or 5                   | 1,564         | 8.46%              | Bought recently, buy often, spend the most |
| **Loyal Customers**     | R is 3 and FM is 4 or 5                        | 773           | 4.18%              | Buy regularly and spend well               |
| **Potential Loyalists** | R is 4 or 5 and FM is 3                        | 2,153         | 11.65%             | Recent, with moderate spend and frequency  |
| **New Customers**       | R is 4 or 5 and FM is 1 or 2                   | 3,685         | 19.94%             | Bought very recently but little so far     |
| **Need Attention**      | R is 3 and FM is 1 to 3                        | 2,917         | 15.78%             | Middling on everything, may drift away     |
| **Can't Lose Them**     | R is 1 or 2 and FM is 4 or 5                   | 1,273         | 6.89%              | Big spenders who have gone quiet           |
| **At Risk**             | R is 1 or 2 and FM is 3                        | 2,016         | 10.91%             | Used to buy fairly often, now lapsing      |
| **Hibernating**         | Everything else (R is 1 or 2 and FM is 1 or 2) | 4,103         | 22.20%             | Long inactive, low frequency, low spend    |

The same rules can be read as a grid. Rows are Recency scores, columns are the FM average score:

|           | **FM = 1**         | **FM = 2**         | **FM = 3**              | **FM = 4**          | **FM = 5**          |
|-----------|--------------------|--------------------|-------------------------|---------------------|---------------------|
| **R = 5** | **New Customers**  | **New Customers**  | **Potential Loyalists** | **Champions**       | **Champions**       |
| **R = 4** | **New Customers**  | **New Customers**  | **Potential Loyalists** | **Champions**       | **Champions**       |
| **R = 3** | **Need Attention** | **Need Attention** | **Need Attention**      | **Loyal Customers** | **Loyal Customers** |
| **R = 2** | **Hibernating**    | **Hibernating**    | **At Risk**             | **Can't Lose Them** | **Can't Lose Them** |
| **R = 1** | **Hibernating**    | **Hibernating**    | **At Risk**             | **Can't Lose Them** | **Can't Lose Them** |

```dax
Segment = SWITCH(TRUE(),
    R >= 4 && FM >= 4, "Champions",
    R >= 3 && FM >= 4, "Loyal Customers",
    R >= 4 && FM = 3,  "Potential Loyalists",
    R >= 4 && FM <= 2, "New Customers",
    R = 3  && FM <= 3, "Need Attention",
    R <= 2 && FM >= 4, "Can't Lose Them",
    R <= 2 && FM = 3,  "At Risk",
    "Hibernating")
```

## 3.6 Step 4: Customer health groups and marketing actions

To give leadership a single health indicator, the eight segments were grouped in a **Segment Strategy** table (created in DAX, with a description and marketing action for each segment):

| **Segment group**         | **Segments included**                                                                    |
|---------------------------|------------------------------------------------------------------------------------------|
| Doing Well                | Champions, Loyal Customers, Potential Loyalists, New Customers (8,175 customers, 44.23%) |
| Needs Attention / At Risk | Need Attention, At Risk, Can't Lose Them, Hibernating (10,309 customers, 55.77%)         |

The "Doing Well %" and "Needs Attention %" KPI cards on the Customer Segmentation page are DAX measures that divide the customers in each group by the customers currently selected, so they update when the Country or Product slicers are used.

[View Report Sheet](./RFM_Analysis_Power_BI_Project.pbix)


---

# 4. Executive Summary

Cozura EMart is **profitable and growing**, with \$29.36M in sales at a 41.15% profit margin. Beneath that growth, **more than half of its customers (55.77%) are at risk or need attention**, about 22% are hibernating, and the **United States, the largest market by sales, is the weakest on customer health**. Customers order only 1.5 times on average, so the biggest opportunity is turning one-time buyers into repeat buyers and winning back valuable customers who have gone quiet.

<img width="1424" height="140" alt="kpi_headline_metrics" src="https://github.com/user-attachments/assets/f530c18f-176d-4ac9-a51b-fcf64b1f81a1" />


*Headline performance metrics (Executive Overview page)*

| **Metric**                                 | **Result** | **Note**                                                |
|--------------------------------------------|------------|---------------------------------------------------------|
| **Total sales**                            | \$29.36M   | 1 Jul 2005 to 31 Jul 2008                               |
| **Total cost**                             | \$17.28M   | 58.85% of sales                                         |
| **Total profit**                           | \$12.08M   | Sales minus product cost                                |
| **Profit margin**                          | 41.15%     | Profit divided by sales                                 |
| **Customers**                              | 18,484     | Customers who placed at least one order                 |
| **Orders**                                 | 27,659     | 1.50 orders per customer                                |
| **Customers doing well**                   | 44.23%     | Champions, Loyal, Potential Loyalists and New Customers |
| **Customers at risk or needing attention** | 55.77%     | About 22% are hibernating                               |



---

# 5. Insights Deep Dive

The report has seven pages. The Home page and the Insights & Recommendations page are not repeated here because their content appears in the Executive Summary, Takeaway and Recommendations. Each page below is shown first, then every chart with the insights it supports. The figures were recalculated from the underlying data and match the dashboard.

[View Interactive Dashboard](https://drive.google.com/file/d/1i28lpUCG-D5fGwV6rPsti2Wij9koDWF1/view?usp=sharing)



## 5.1: Executive Overview

**Objective:** give leadership a quick read on sales, cost and profit, the best-selling products, the sales trend and where revenue comes from.

<img width="991" height="556" alt="page_executive_overview" src="https://github.com/user-attachments/assets/dfcb78d3-1a25-4a20-9cf9-f1e400143a6e" />


*Page: Executive Overview*



### Sales, Profit, Margin and Cost

- Sales of **\$29.36M** less product cost of **\$17.28M** leave a profit of **\$12.08M**, a **41.15% margin**. Costs take 58.85 cents of every sales dollar.
- The margin has held up as the business grew: 40.2% in 2005, 40.5% in 2006, 41.6% in 2007 and 41.4% for January to July 2008 (calculated by order date). Growth was not bought with lower margins.
- The average order is worth about **\$1,061** and the average customer about **\$1,588**, but with only 1.5 orders per customer the second figure is the one with room to grow.



###  Top Selling Products (top 5 by number of orders)

<img width="772" height="402" alt="exec_top_selling_products" src="https://github.com/user-attachments/assets/b11d6d31-964e-4e0a-8cb2-4f88b3009b50" />


- The **Water Bottle (30 oz) leads with 4,244 orders**, followed by the Patch Kit (3,191), Mountain Tire Tube (3,095), Road Tire Tube (2,376) and Sport-100 Helmet, Red (2,230).
- All five are low-priced accessories and consumables. Together they appear in about 45% of all orders but bring in only \$131K (0.4% of sales), whereas the Mountain-200 bike range alone generates 27% of sales.
- The best sellers therefore bring customers through the door and are natural **entry products** to bundle with higher-value bikes, rather than being the main source of revenue.



### Sales Trend 

<img width="700" height="400" alt="exec_sales_trend" src="https://github.com/user-attachments/assets/3819606e-69cc-47be-bfa9-c02dc92572a8" />


- Sales rose from **\$2.99M in 2005 to \$10.42M in 2008**, more than three times higher, with each year above the one before it.
- Both end years are partial (2005 covers July to December, 2008 covers January to July), which makes the growth stronger than it looks: by order date, January to July 2008 sales of \$9.77M had already matched all of 2007 (\$9.79M).
- Growth came mainly from **more customers**: customers placing orders rose from 1,013 in 2005 to 2,677, 9,309 and 11,377 in the following years. Acquisition has been strong, which makes the low repeat rate more important.



### Total Sales by Country

<img width="764" height="782" alt="exec_sales_by_country" src="https://github.com/user-attachments/assets/50298ffb-9df4-428c-bcd1-ec670092337b" />


- The **United States (\$9.39M, 32.0%)** and **Australia (\$9.06M, 30.9%)** together produce 62.8% of sales. The UK (\$3.39M), Germany (\$2.89M), France (\$2.64M) and Canada (\$1.98M) follow.
- Taken as regions, North America (US and Canada) delivers 38.7% of sales and Europe (UK, Germany, France) 30.4%, which is about the same as Australia alone.
- Australia is the most productive market per customer. It has only 19.4% of customers (3,591) but 30.9% of sales, which means **\$2,523 per customer against \$1,201 in the US** and 1.87 orders per customer against 1.22. The US earns its place through volume (7,819 customers, 42.3% of the base), not depth.



## 5.2: Customer Overview

**Objective:** understand who the customers are by gender, marital status and age, and how much each group contributes to sales, so that marketing can be aimed at the right people.

<img width="1920" height="1080" alt="page_customer_overview" src="https://github.com/user-attachments/assets/7810c452-d7c4-402f-bd24-e4d57676a559" />


*Page: Customer Overview*



###  Customers and Orders

- **18,484 customers** placed **27,659 orders**, an average of 1.50 orders each. About 63% of customers have ordered only once, which frames the retention challenge on the next page.



###  Sales by Gender 

<img width="712" height="402" alt="cust_sales_by_gender" src="https://github.com/user-attachments/assets/1edc245e-5b25-4577-8fe3-a0c020ba7d77" />


- Sales are almost evenly split: **Female 50.46% (\$14.81M)** and **Male 49.54% (\$14.55M)**.
- Women make up 49.4% of customers but contribute slightly more, spending about \$1,622 each against \$1,555 for men. The gap is small, so gender is not a useful basis for targeting.



### Sales by Marital Status 

<img width="732" height="400" alt="cust_sales_by_marital_status" src="https://github.com/user-attachments/assets/cb25b9fb-3b8a-4762-9e5c-708c13d33cba" />


- **Married customers account for 51.73% (\$15.19M)** and **Single customers for 48.27% (\$14.17M)**.
- Single customers are 45.8% of the base but spend more each (about \$1,673 against \$1,517 for married customers, roughly 10% more), which makes them a valuable group to keep engaged.



### Sales by Age Band 

<img width="772" height="470" alt="cust_sales_by_age_band" src="https://github.com/user-attachments/assets/1f84255c-e501-4193-b3a2-2e9d04e3713c" />


- **Customers aged 35 to 44 lead with \$11.37M (38.7%)**, followed by 45 to 54 with \$9.04M (30.8%). Together these two bands produce **69.4% of sales**.
- Sales fall away sharply after 55: \$4.10M for 55 to 64, \$1.35M for 65 to 74 and almost nothing beyond 75. The youngest band (28 to 34) contributes \$3.40M (11.6%).



###  Customer by Age Band 

<img width="772" height="470" alt="cust_customers_by_age_band" src="https://github.com/user-attachments/assets/4af62d57-6327-422b-a410-80856e70a578" />


- The 35 to 54 group makes up **61.6% of customers (11,386)** but 69.4% of sales, so it over-delivers.
- Spend per customer confirms it: about \$1,825 in the 35 to 44 band and \$1,753 in the 45 to 54 band, compared with \$1,223 for ages 28 to 34 (about a third lower) and \$1,102 for ages 65 to 74.
- The younger band is 15.0% of customers (2,779) but only 11.6% of sales, so there is room to grow spending among younger customers.



## 5.3: Customer Segmentation (RFM Analysis)

**Objective:** show how customers are spread across the eight RFM segments, what each segment looks like on Recency, Frequency and Monetary value, and how healthy the customer base is overall. This is the core page of the project. The Country and Product Name slicers re-cut every chart.

<img width="1920" height="1080" alt="page_customer_segmentation" src="https://github.com/user-attachments/assets/a292adc3-0baf-419a-8f39-1ebae8418a0a" />


*Page: Customer Segmentation*



###  Needs Attention % and Doing Well %

<img width="622" height="140" alt="seg_kpi_cards" src="https://github.com/user-attachments/assets/efaf24c5-eb86-46fc-b356-7a5aafe85389" />



- **55.77% of customers (10,309) are at risk or need attention** and **44.23% (8,175) are doing well**.
- "Doing well" customers account for 54.0% of historic revenue and the other group for 46.0% (\$13.51M), so the group needing attention represents a lot of money, not just a lot of people.



### Customer Segment Count 

<img width="774" height="474" alt="seg_segment_count" src="https://github.com/user-attachments/assets/1e60376f-11f8-4567-b981-b31d297a802f" />


- **Hibernating (4,103, 22.2%)** is the largest segment, followed by New Customers (3,685, 19.9%) and Need Attention (2,917, 15.8%). Only 8.5% are Champions and 4.2% Loyal.
- The customer base is wide at the bottom and narrow at the top. Hibernating and New customers together are 42.1% of the base but contribute just 4.1% of revenue.



### Customer Segment by Average Recency (days since last order)

<img width="774" height="402" alt="seg_avg_recency" src="https://github.com/user-attachments/assets/00b0f3aa-69fd-413b-81cc-72a0e3566787" />


- **At Risk customers have been away for 401 days on average**, more than 13 months, followed by Hibernating (290) and Can't Lose Them (284). Their window for an easy win-back is closing.
- Champions (76 days), Potential Loyalists (72) and New Customers (70) all bought within the last quarter, which is why they score well on recency.



###  Customer Segment by Average Frequency (orders per customer)

<img width="774" height="418" alt="seg_avg_frequency" src="https://github.com/user-attachments/assets/0388a2cf-d53a-41d5-b1bd-05cdac342b87" />


- **Champions average 2.97 orders** and Loyal Customers 2.39, while Hibernating customers average 1.03, in effect a single purchase.
- Even the best customers order only about three times, so loyalty is shallow across the whole base. That explains the low company average of 1.5 orders per customer.



###  Customer Segment by Average Monetary Value (spend per customer)

<img width="774" height="474" alt="seg_avg_monetary" src="https://github.com/user-attachments/assets/5ba6d3a9-0fd4-4ccb-95c7-541b57e93a0d" />


- Loyal Customers (\$5,524), **Can't Lose Them (\$5,225)** and Champions (\$4,956) are the highest spenders. Hibernating (\$159) and New Customers (\$150) spend very little.
- **Can't Lose Them are lapsed Champions.** They spent as much as Champions but last bought about 9 months ago. These 1,273 customers (6.9% of the base) hold **\$6.65M (22.7%) of historic revenue**.
- Adding the At Risk segment, 3,289 customers (17.8%) hold **\$10.93M, 37.2% of all historic revenue**. This is the most valuable pool to win back.
- Champions are 8.5% of customers but 26.4% of revenue, and Champions plus Loyal customers (12.6%) generate 40.9%, so protecting this top group matters just as much.



###  Reading the page by country (Country slicer)

Selecting each country in the slicer shows how customer health differs by market:

| **Country**        | **Customers** | **Doing Well %** | **Needs attention %** | **Champions %** | **Hibernating %** |
|--------------------|---------------|------------------|-----------------------|-----------------|-------------------|
| **United States**  | 7,819         | 39.35%           | 60.65%                | 3.5%            | 26.0%             |
| **France**         | 1,810         | 41.66%           | 58.34%                | 6.6%            | 27.3%             |
| **Germany**        | 1,780         | 45.56%           | 54.44%                | 9.4%            | 25.8%             |
| **United Kingdom** | 1,913         | 46.42%           | 53.58%                | 9.4%            | 19.9%             |
| **Canada**         | 1,571         | 49.33%           | 50.67%                | 14.5%           | 18.5%             |
| **Australia**      | 3,591         | 52.07%           | 47.93%                | 16.5%           | 12.5%             |

- The **United States is the weakest market** (60.65% needing attention, only 3.5% Champions), followed by **France (58.34%)**. **Australia is the strongest** (52.07% doing well), followed by Canada (49.33%).
- Australia has 4.7 times the share of Champions found in the US (16.5% against 3.5%). Its customers also order more often (1.87 orders each against 1.22 in the US).
- Australia is not risk-free: 14.6% of its customers (about 520) are Can't Lose Them, the highest share of any market, so a win-back effort there would also pay off.
- Customer health is a **geography story, not a demographic one**. The share needing attention varies by about 5 points across age bands (53% to 58%) and by less than 1 point between genders and between marital statuses, but by nearly 13 points between countries.



##  5.4: Customer Details

**Objective:** give the marketing team a customer-level list so that every segment can be acted on, by filtering the page by segment, country or product.

<img width="993" height="555" alt="page_customer_details" src="https://github.com/user-attachments/assets/2b52a80d-0baa-403d-8d85-5a6212cf26a5" />


*Page: Customer Details (first 17 of 18,484 customers shown)*

- Each row shows the customer's demographics, contact details, location, **total revenue, RFM Score, Segment and Segment Group**, so the list can be filtered and exported straight into an email or CRM campaign.
- The score is easy to trace. For example, customer 11001 scores **535** (Recency 5, Frequency 3, Monetary 5) and is a Champion, while customer 11000 scores **235** (Recency 2, Frequency 3, Monetary 5), which makes them Can't Lose Them: a big spender who has not bought for a while.
- A win-back list is a single filter away. The five highest-spending Can't Lose Them customers (keys 12655, 13263, 12323, 12333 and 12650) are all in France. Each spent between \$10.8K and \$10.9K over four orders and last ordered 207 to 259 days before the snapshot, so they are the first people to contact.
- The page contains names, emails and phone numbers. These are fictional sample records, but in a live setting this page should be restricted to authorised marketing users.



##  5.5: Marketing Actions

**Objective:** translate each segment into a clear description and a recommended action, so the analysis leads directly to something the marketing team can do.

<img width="993" height="554" alt="page_marketing_actions" src="https://github.com/user-attachments/assets/7d3451d2-6b03-4a5f-97fe-b0c39aa0340e" />


*Page: Marketing Actions*

- Every segment has a plain-language description and an action, from **VIP perks and referral requests for Champions** to **personal outreach for Can't Lose Them** and **low-cost reactivation for Hibernating customers**.
- The actions follow the value and recency of each segment: spend more where value is high (Champions, Loyal, Can't Lose Them), nurture where potential is high (Potential Loyalists, New Customers), and limit spend where value is low and the customer is inactive (Hibernating).

The table below combines the segment sizes with the actions to show where effort is best placed. The priority column is a suggestion derived from this analysis.

| **Segment**             | **Customers** | **% of customers** | **Historic revenue** | **% of revenue** | **Suggested focus**                            |
|-------------------------|---------------|--------------------|----------------------|------------------|------------------------------------------------|
| **Can't Lose Them**     | 1,273         | 6.89%              | \$6.65M              | 22.65%           | Highest priority: personal win-back            |
| **At Risk**             | 2,016         | 10.91%             | \$4.28M              | 14.57%           | Highest priority: win-back offers and survey   |
| **Champions**           | 1,564         | 8.46%              | \$7.75M              | 26.40%           | Protect: VIP perks, referrals                  |
| **Loyal Customers**     | 773           | 4.18%              | \$4.27M              | 14.55%           | Protect: loyalty programme, upsell             |
| **Potential Loyalists** | 2,153         | 11.65%             | \$3.27M              | 11.15%           | Grow: recommendations, membership              |
| **Need Attention**      | 2,917         | 15.78%             | \$1.93M              | 6.57%            | Re-engage: limited-time offers                 |
| **New Customers**       | 3,685         | 19.94%             | \$0.55M              | 1.88%            | Convert: welcome series, second-purchase offer |
| **Hibernating**         | 4,103         | 22.20%             | \$0.65M              | 2.22%            | Low cost: single reactivation email            |


---

# 6. Takeaway

The project set out to find which customers Cozura EMart should protect, which it is losing, and where to act first. The analysis gives a specific answer:

- **The business is healthy, and growth has been driven by acquisition.** Sales rose from \$3M to \$10.4M a year, the margin held at about 41%, and the customer base grew from about 1,000 customers a year to more than 11,000.
- **The customer base is less healthy than the sales figures suggest.** 55.77% of customers are at risk or need attention, about 22% are hibernating, and 63% have bought only once.
- **A small group carries the business.** Champions and Loyal customers are 12.6% of customers but 40.9% of revenue, and the 3,289 customers in At Risk and Can't Lose Them hold \$10.93M (37.2%) of historic revenue. The most valuable customers to win back are the ones who used to spend the most.
- **The problem is a market issue.** The United States is the largest market (32% of sales, 7,819 customers) but has the weakest customer health (60.65% needing attention and only 3.5% Champions). Australia leads on both sales per customer and health, with customers who order almost twice as often as in the US.
- **Demographics do not explain risk, behaviour does.** Gender and marital status are balanced and the 35 to 54 age group drives about 69% of sales, but the share of at-risk customers barely varies by age or gender.
- **The cheapest route to growth is repeat purchasing.** Best sellers are low-priced accessories, customers order 1.5 times on average, and even Champions order only about three times.

In short, Cozura EMart does not need to chase more customers first. It needs to keep the ones it has by winning back valuable lapsed customers, fixing retention in the US and France, replicating what works in Australia, and turning new customers into repeat buyers.


---

# 7. Recommendations

| **Priority** | **Observation**                                                                                                                                     | **Recommendation**                                                                                                                                                                                         |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| High         | 3,289 At Risk and Can't Lose Them customers (17.8%) hold \$10.93M (37.2%) of historic revenue and have been inactive for 284 to 401 days on average | **Launch a win-back programme.** Target At Risk and Can't Lose Them with personalised offers and re-engagement emails, and start with those who spent the most (the Customer Details page gives the list). |
| High         | The US has 60.65% of customers needing attention (France 58.34%), yet the US is the largest market by sales                                         | **Fix retention in the US and France.** Run market-specific loyalty and re-engagement campaigns and investigate causes such as pricing, competition, delivery and service.                                 |
| High         | Australia has 52.07% of customers doing well, 16.5% Champions and \$2,523 sales per customer                                                        | **Replicate Australia's success.** Study the offers, channels and product mix that work there and test them in the US and Europe. Also run a win-back for its 520 Can't Lose Them customers.               |
| Medium       | Champions (8.5%) and Loyal (4.2%) customers produce 40.9% of revenue                                                                                | **Protect and grow top customers.** Reward them with VIP perks, early access and referral incentives, and use cross-selling to move Potential Loyalists up.                                                |
| Medium       | Customers aged 35 to 54 are 61.6% of customers and 69.4% of sales; ages 28 to 34 spend about a third less per customer                              | **Focus marketing on the 35 to 54 age group** with campaigns, bundles and loyalty rewards built around their needs, while testing ways to grow spend among younger customers.                              |
| Medium       | The top five products by orders are low-priced accessories (\$131K in sales) while the Mountain-200 range produces 27% of sales                     | **Use best sellers as entry products.** Bundle the Water Bottle and Patch Kit with higher-value items to raise average order value and encourage repeat purchases.                                         |
| Medium       | Customers order 1.5 times on average, 63% only once, and New Customers (19.9% of the base) have spent only \$150 each                               | **Convert new customers into repeat buyers** with onboarding emails, follow-up offers and a second-purchase incentive.                                                                                     |
| Ongoing      | Costs are 58.85% of sales; the margin has held at about 41% to 42%                                                                                  | **Keep costs under review.** Track margin by year and by product so rising sales keep turning into profit.                                                                                                 |


---

# 8. Assumptions & Caveats

The dataset was not free of problems. The tables below show what was found, how it was handled so the analysis stays reliable, and what would prevent it from happening again.

### Data quality issues

| **Issue found**                                                                                                                                                             | **Mitigation**                                                                                                                              | **Recommendation**                                                                    |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| Source tables were very wide and full of unused fields (for example DimProduct had 36 columns and DimCustomer 29, including Spanish and French duplicates of the same text) | Kept only the fields needed (2 to 14 per table) and renamed the English columns, which made the model smaller and easier to follow          | Extract only the required fields at source and keep a short data dictionary           |
| Key columns (CustomerKey, ProductKey, date keys and others) were stored as numbers and could be summed or aggregated by mistake                                             | Converted every key column to text                                                                                                          | Define identifier fields as text or key types in the source                           |
| Columns that were empty or almost empty (CarrierTrackingNumber and CustomerPONumber are blank in all 60,398 rows; Suffix, Title and AddressLine2 are 98% to 100% blank)     | Removed them from the model because they add no analytical value                                                                            | Profile null percentages on import and exclude fields that are nearly empty           |
| Age was not stored, and 30 customers appear aged 85 or over, up to 98 (born 1910)                                                                                           | Calculated age at the last order date and grouped it in bands, keeping the oldest customers in an 85+ band (30 customers, \$4,952 of sales) | Validate birth-date ranges when data is captured and flag implausible ages            |
| Duplicate records                                                                                                                                                           | Checked for duplicate order lines and duplicate customers; none were found                                                                  | Keep a duplicate check as a standard step in the data preparation                     |

### Analytical assumptions

| **Issue found**                                                                                                                                                                                                                                   | **Mitigation**                                                                                                              | **Recommendation**                                                                           |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
| The data ends in July 2008, so today's date would make every customer look lapsed                                                                                                                                                                 | Set the snapshot date to one day after the last order (1 August 2008) and show it on the page                               | Refresh the snapshot date automatically in a live version                                    |
| 62.9% of customers have ordered only once, so Frequency cannot be split into quintiles                                                                                                                                                            | Used fixed frequency bands (1, 2, 3, 4, 5 or more orders) and quintiles for Recency and Monetary                            | Review the bands as repeat purchasing improves                                               |
| "Doing Well" includes New Customers, who are recent but have spent little (\$150 on average)                                                                                                                                                      | Grouped segments by recency and value together and reported revenue next to customer counts so low-value groups are visible | Revisit the grouping with the business owner and track new-customer conversion               |
| Monetary value is lifetime spend, so it shows what a customer has spent, not what can be recovered                                                                                                                                                | Described figures as "historic revenue" and avoided promising recovery amounts                                              | Pair RFM with a win-back test to measure how much revenue returns                            |
                                             |
| The Sales Trend chart uses the date table related through the due date, not the order date, so a few orders near year ends are counted in the following year. By order date the totals are \$3.27M, \$6.53M, \$9.79M and \$9.77M for 2005 to 2008 | Kept the dashboard figures and stated the order-date totals where the difference matters                                    | Relate the date table through the order date and use the due date only for delivery analysis |
| Sales amounts come from several currency keys in the source                                                                                                                                                                                       | Treated all amounts as a single reporting currency, as provided in the source                                               | Confirm the reporting currency in the data dictionary                                        |
| The youngest customers in the data are aged 28, so younger buyers cannot be assessed                                                                                                                                                              | Used age bands that start at 28 to 34                                                                                       | Capture the full age range or confirm the target customer age                                |
| The data shows associations, not causes (for example why US customers lapse)                                                                                                                                                                      | Kept wording to what the data shows and listed causes (pricing, delivery, service) as things to investigate                 | Combine RFM with survey and service data to confirm causes                                   |



---

# 9. Skills Demonstrated

| **Skill area**                             | **How it was applied in this project**                                                                                                                                                          |
|--------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Data cleaning with Power Query**         | Promoted headers, set types, removed unused and empty columns, deleted a placeholder row, built a customer name column and converted keys to text across six tables                             |
| **Data modelling**                         | Built a star schema with one fact table and five dimensions, plus a calculated RFM table and a segment lookup table                                                                             |
| **DAX**                                    | Wrote measures, calculated columns and calculated tables (SUMMARIZE, ADDCOLUMNS, DATATABLE, SWITCH, PERCENTILE.INC, DATEDIFF, DIVIDE, ALL, ALLSELECTED) to turn orders into scores and segments |
| **RFM analysis and customer segmentation** | Calculated Recency, Frequency and Monetary values, scored them from 1 to 5 using quintiles and fixed bands, and mapped them into eight segments with documented rules                           |
| **KPI design**                             | Defined sales, cost, profit, margin, orders per customer, Doing Well % and Needs Attention % so each answers a business question                                                                |
| **Dashboard and visual design**            | Built a seven-page interactive report with custom backgrounds, KPI cards, bar, donut, ribbon and map visuals, slicers, a page navigator and a drill-through customer list                       |
| **Business storytelling**                  | Moved from problem to method, findings, takeaway and recommendations, explaining RFM in plain language for non-technical readers                                                                |
| **Insight generation**                     | Went beyond the charts to find lapsed Champions, revenue concentration by segment, the US and Australia contrast and the difference between best-selling and revenue-driving products           |
| **Data quality awareness**                 | Documented what was wrong with the data, how each issue was handled and how to avoid it in future                                                                                               |
| **Documentation**                          | Produced a clear data model diagram, step-by-step methodology, DAX references and a structured report for both technical and non-technical audiences                                            |

[Back to top](#table-of-contents)

