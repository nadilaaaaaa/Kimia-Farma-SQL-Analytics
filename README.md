# 📊 Kimia Farma Business Analytics  

Kimia Farma is one of the largest pharmaceutical companies in Indonesia, with branches spread across multiple provinces. To support data-driven decision-making, this project applies SQL for ***data cleaning, transformation, and business analytics***. The processed data is used to build analytical tables and answer key business questions, such as revenue trends, branch performance, discount effectiveness, and regional profitability.

---

## 🚀 Features  

- **Data Cleaning** → Handle invalid values, trim strings, standardize formats  
- **Transformation** → Build unified analytical table `tabel_analisa`  
- **Branch & Province Analysis** → Identify top/bottom branches by profit and rating  
- **Product Analysis** → Find products contributing the most profit  
- **Performance Indicators** → Net Sales, Net Profit, Transaction Count, Ratings

---

## 📊 Business Questions  

1. How do Kimia Farma’s **net sales** grow year by year (**2020–2023**)?  
2. Which **provinces** record the most **transactions**?  
3. Which **provinces** achieve the highest **net sales**?  
4. Are there **branches** with high **branch ratings** but low **transaction ratings**?
5. How is the distribution of ***total profit*** across ***provinces*** in Indonesia? 
6. What is Kimia Farma’s overall **performance** in terms of **sales**, **profit**, **customers**, and **transactions**?  
7. How does the **average discount** differ across **provinces**?  
8. What is the relationship between **profit** and **transaction count** per **branch**?  
9. Which **products** have the highest **purchase frequency**? 

---

## 🛠️ Tech Stack  

- 🗄**BigQuery SQL** (main scripts)  
- 📊**Looker Studio** (data visualization)  
- 🐙**GitHub** (code repository and sharing)  

---
# 📖 Data Sources
There are four datasets used for analysis and stored in BigQuery:
- `kf_final_transaction` → records of all transactions
- `kf_inventory` → branch inventory data
- `kf_kantor_cabang` → branch details and ratings
- `kf_product` → product details and categories

Due to confidentiality, the datasets utilized in this project cannot be disclosed or shared publicly. Only the data structure and transformation process are presented for demonstration purposes

---


## 📝 SQL Scripts
###  1️⃣ Data Cleaning  
```sql
-- =========================================
-- CLEANING TRANSACTION TABLE
-- =========================================
CREATE OR REPLACE TABLE `kimia_farma.kf_final_transaction` AS SELECT
     TRIM (transaction_id) AS transaction_id,       
     date,
     branch_id,
     TRIM(customer_name) AS customer_name,
     product_id,

     CASE WHEN price < 0 THEN 0 ELSE price
     END AS price,

     CASE WHEN discount_percentage < 0 OR discount_percentage > 1 THEN 0
     ELSE discount_percentage
     END AS discount_percentage,

     CASE WHEN rating < 1 OR rating > 5 THEN NULL ELSE rating
     END AS rating
FROM `kimia_farma.kf_final_transaction`
WHERE transaction_id IS NOT NULL
AND branch_id IS NOT NULL
AND customer_name IS NOT NULL
AND product_id IS NOT NULL
AND price IS NOT NULL;

-- =========================================
-- CLEANING INVENTORY TABLE
-- =========================================
CREATE OR REPLACE TABLE `kimia_farma.kf_inventory` AS SELECT
Inventory_ID,
branch_id,
TRIM(product_id) AS product_id,
TRIM(product_name) AS product_name,
CASE WHEN opname_stock < 0 THEN 0 ELSE opname_stock
END AS opname_stock
FROM `kimia_farma.kf_inventory`
WHERE Inventory_ID IS NOT NULL
AND branch_id IS NOT NULL
AND product_id IS NOT NULL
AND product_name IS NOT NULL;

-- =========================================
-- CLEANING BRANCH TABLE
-- =========================================
CREATE OR REPLACE TABLE `kimia_farma.kf_kantor_cabang` AS SELECT
branch_id,
TRIM(branch_category) AS branch_category,
TRIM(branch_name) AS branch_name,
INITCAP(TRIM(kota)) AS kota,
INITCAP(TRIM(provinsi)) AS provinsi,
CASE WHEN rating < 0 OR rating > 5 THEN NULL
ELSE rating END AS rating
FROM `kimia_farma.kf_kantor_cabang`
WHERE branch_id IS NOT NULL;

-- =========================================
-- CLEANING PRODUCT TABLE
-- =========================================
CREATE OR REPLACE TABLE `kimia_farma.kf_product` AS SELECT
product_id,
TRIM(product_name) AS product_name,
TRIM(product_category) AS produc_category,
CASE WHEN price < 0 THEN 0
ELSE price
END AS price
FROM `kimia_farma.kf_product`;
```
### 2️⃣ Data Transformation
```sql
CREATE OR REPLACE TABLE kimia_farma.tabel_analisa AS
SELECT 
t.transaction_id,       
t.date,                
b.branch_id,           
b.branch_name,         
b.kota,                
b.provinsi,            
b.rating AS rating_cabang,        
t.customer_name,        
p.product_name,         
t.price AS actual_price,         
t.discount_percentage,  

CASE WHEN t.price <= 50000 THEN 0.10
     WHEN t.price <= 100000 THEN 0.15
     WHEN t.price <= 300000 THEN 0.20
     WHEN t.price <= 500000 THEN 0.25
     WHEN t.price >= 500000 THEN 0.30
END AS persentase_gross_laba,
 
t.price*(1-t.discount_percentage/100) AS nett_sales,  

(t.price * (1 - t.discount_percentage/100.0)) *
CASE WHEN t.price <=50000 THEN 0.10
     WHEN t.price <= 100000 THEN 0.15
     WHEN t.price <= 300000 THEN 0.20
     WHEN t.price <= 500000 THEN 0.25
     WHEN t.price >= 500000 THEN 0.30
END AS nett_profit,    
t.rating AS rating_transaksi           

FROM `kimia_farma.kf_final_transaction` t
JOIN `kimia_farma.kf_kantor_cabang` b
ON t.branch_id = b.branch_id
JOIN `kimia_farma.kf_product` p
ON t.product_id = p.product_id;
```
From these sources, a central analytical table called tabel_analisa was created.
It integrates transactions, branches, and products to support BI and analytics.

---

### tabel_analisa

| Column              | Type    | Description |
|---------------------|---------|-------------|
| transaction_id      | STRING  | Unique transaction ID |
| date                | DATE    | Transaction date |
| branch_id           | STRING  | Kimia Farma branch code |
| branch_name         | STRING  | Branch name |
| kota                | STRING  | City where branch is located |
| provinsi            | STRING  | Province where branch is located |
| rating_cabang       | FLOAT   | Average branch rating (1–5) |
| customer_name       | STRING  | Customer making the purchase |
| product_name        | STRING  | Product/medicine name |
| actual_price        | NUMERIC | Listed product price |
| discount_percentage | FLOAT   | Discount applied (0–1) |
| persentase_gross_laba | FLOAT | Gross margin percentage based on price range |
| nett_sales          | NUMERIC | Sales after discount |
| nett_profit         | NUMERIC | Profit after applying gross margin |
| rating_transaksi    | FLOAT   | Customer rating for transaction (1–5) |



### 3️⃣ Data Exploration
#### 📊 Year-over-Year Revenue & Profit (2020–2023)
```sql
SELECT 
    EXTRACT(YEAR FROM date) AS tahun,
    SUM(nett_sales) AS total_nett_sales,
    SUM(nett_profit) AS total_nett_profit
FROM `kimia_farma.tabel_analisa`
GROUP BY tahun
ORDER BY tahun;
```

#### 🏢 Top 10 Provinces by Total Transactions
```sql
SELECT
    provinsi,
    COUNT(transaction_id) AS total_transaksi
FROM `kimia_farma.tabel_analisa`
GROUP BY provinsi
ORDER BY total_transaksi DESC
LIMIT 10;
```
#### 💰 Top 10 Provinces by Net Sales
```sql
SELECT
     provinsi,
     SUM(nett_sales) AS nett_sales
FROM `kimia_farma.tabel_analisa`
GROUP BY provinsi
ORDER BY nett_sales DESC
LIMIT 10;
```
#### ⭐ Top 5 Branches: Highest Rating, Lowest Transaction Rating
```sql
SELECT
    branch_name,
    provinsi,
    AVG(rating_cabang) AS avg_branch_rating,
    AVG(rating_transaksi) AS avg_transaction_rating
FROM `kimia_farma.tabel_analisa`
GROUP BY branch_name, provinsi
HAVING AVG(rating_cabang) >= 4.5
ORDER BY avg_transaction_rating ASC
LIMIT 5;
```
#### 📌 Overall Business Performance Summary
```sql
SELECT
     COUNT(transaction_id) AS banyak_transaksi,
     COUNT (DISTINCT customer_name) AS banyak_customer,
     SUM(nett_sales) AS total_pendapatan,
     SUM(nett_profit) AS total_laba,
FROM `kimia_farma.tabel_analisa`;
```
#### 🎟️ Average Discount by Province
```sql
SELECT 
     provinsi,
     AVG(discount_percentage) AS avg_diskon,
FROM `kimia_farma.tabel_analisa`
GROUP BY provinsi
ORDER BY avg_diskon DESC;
```
#### ⚖️ Profit vs Number of Transactions (by Branch)
```sql
SELECT 
     branch_name,
     SUM(nett_profit) AS total_laba,
     COUNT(transaction_id) AS banyak_transaksi,
FROM `kimia_farma.tabel_analisa`
GROUP BY branch_name
ORDER BY total_laba DESC;
```
#### 💊 Total Transactions by Product
```sql
SELECT
     product_name,
     COUNT(transaction_id) AS banyak_transaksi,
FROM `kimia_farma.tabel_analisa`
GROUP BY product_name
ORDER BY banyak_transaksi DESC;
```
## 📈 Dashboard Preview
🔗***Interactive Dashboard: https://lookerstudio.google.com/reporting/f5603bef-598f-4bfd-9b80-24ceb2d5513b***

<img width="888" height="1425" alt="image" src="https://github.com/user-attachments/assets/6756abf1-a4ef-491e-90e8-08941bee26a8" />







