# Pizza Sales Data Analysis Project

This project integrates **PostgreSQL** for querying and **Excel** for visualization to derive business insights.

## **Key Components**
- **SQL Queries:** Extract key metrics—revenue, sales trends, and performance breakdowns.
- **Excel Dashboards:** Pivot tables, dynamic charts, and Power Query streamline data analysis.
- **Data Cleaning:** Resolves formatting issues for seamless database imports.

Using **SQL for data extraction** and **Excel for reporting**, this analysis supports data-driven decisions with **clear, actionable insights**.

<img width="1242" alt="Screenshot 2025-06-16 at 12 03 36 PM" src="https://github.com/user-attachments/assets/35afae93-11db-4b85-bf03-aec3aa2f2324" />


## Project Directory Structure

```
Pizza_Dashboard/
├── Images/
│   ├── Excel_worksheets/  # Screenshots of worksheets & analysis
│   ├── Misc/              # Background images & presentation slides
├── MARK_MAP.html          # Interactive mind map visualization
├── pizza_date_change.ipynb # Jupyter Notebook for date formatting
├── Pizza_sql_excel.xlsx   # Excel file with SQL-based reporting
├── Pizza_dashboard_excel.md # Excel Worksheets info
├── README.md              # Project documentation
├── Resource/
│   ├── pizza_sales_cleaned.csv  # Preprocessed dataset
│   ├── pizza_sales.csv          # Raw dataset
└── SQL_Postgres/
    ├── SQL_Pizza.docx, SQL_Pizza.pdf  # SQL documentation
    └──SQL_screenshots/  # Screenshots of SQL queries & database analysis
```

## Project Overview

This project demonstrates end-to-end data analysis workflow including:
- Data cleaning and preprocessing with Python/Pandas
- [ PostgreSQL database setup](#database-setup) - and data import 
- Business analytics through SQL queries
- [Key Performance Indicator (KPI) Calculations](#business-analytics-queries)  - Sales trend analysis and reporting

## Technologies Used

- **Database:** PostgreSQL  
- **Data Processing:** Python, Pandas  
- **Query Language:** SQL  
- **Data Format:** CSV  
- **Data Visualization & Reporting:** Excel (Pivot Tables, Dashboards)  


## Dataset

The project uses `Resource/pizza_sales.csv` containing pizza order data with the following columns:
- `pizza_id` - Unique identifier for each pizza
- `order_id` - Order reference number
- `pizza_name_id` - Pizza product identifier
- `quantity` - Number of pizzas ordered
- `order_date` - Date of order
- `order_time` - Time of order
- `unit_price` - Price per pizza
- `total_price` - Total price for the order line
- `pizza_size` - Size of pizza (S, M, L, XL, XXL)
- `pizza_category` - Category of pizza
- `pizza_ingredients` - List of ingredients
- `pizza_name` - Full name of pizza

## Challenges & Solutions

### 1. Date Format Mismatch
**Problem**: CSV contained dates in DD-MM-YYYY format while PostgreSQL expected YYYY-MM-DD format.

**Error Encountered**:
```
ERROR: date/time field value out of range: "13-01-2015"
```

**Solution**: Used Python/Pandas to reformat dates before import:

```python
import pandas as pd

# Load and clean the data
df = pd.read_csv("Resource/pizza_sales.csv")
df["order_date"] = pd.to_datetime(df["order_date"], dayfirst=True).dt.strftime("%Y-%m-%d")
df.to_csv("Resource/pizza_sales_cleaned.csv", index=False)
```

### 2. File Path Management
**Problem**: Ensuring the cleaned CSV file was saved in the correct location.

**Solution**: Verified working directory and used explicit file paths:

```python
import os
print(os.getcwd())
df.to_csv("Resource/pizza_sales_cleaned.csv", index=False)
```

## Database Setup

### Table Creation in PostgreSQL
```sql
CREATE TABLE pizza_sales (
    pizza_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    pizza_name_id VARCHAR(50) NOT NULL,
    quantity INT NOT NULL,
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    pizza_size VARCHAR(5) NOT NULL,
    pizza_category VARCHAR(20) NOT NULL,
    pizza_ingredients TEXT NOT NULL,
    pizza_name VARCHAR(100) NOT NULL
);
```

### Data Import
```sql
\copy pizza_sales FROM 'Resource/pizza_sales_cleaned.csv' DELIMITER ',' CSV HEADER;
```

## Business Analytics Queries

The **PostgreSQL** results can be matched with Dashboard insights.  [View SQL_Pizza.pdf](https://github.com/gbadrain/Pizza_Dashboard/blob/main/SQL_Postgres/SQL_Pizza.pdf)

**Excel Worksheets** [Information Excel Worksheets](https://github.com/gbadrain/Pizza_Dashboard/blob/main/Pizza_dashboard_excel.md)


### Key Performance Indicators (KPIs)

#### 1. Total Revenue
```sql
SELECT ROUND(SUM(total_price)::numeric, 2) AS total_revenue 
FROM pizza_sales;
```

#### 2. Average Order Value
```sql
SELECT ROUND(SUM(total_price) / COUNT(DISTINCT order_id), 2) AS avg_order_value 
FROM pizza_sales;
```

#### 3. Total Pizzas Sold
```sql
SELECT SUM(quantity) AS total_pizzas_sold 
FROM pizza_sales;
```

#### 4. Total Orders
```sql
SELECT COUNT(DISTINCT order_id) AS total_orders 
FROM pizza_sales;
```

#### 5. Average Pizzas Per Order
```sql
SELECT ROUND(SUM(quantity) / COUNT(DISTINCT order_id), 2) AS avg_pizzas_per_order 
FROM pizza_sales;
```

### Sales Trend Analysis

#### Daily Order Trends
```sql
SELECT TO_CHAR(order_date, 'Day') AS order_day, 
       COUNT(DISTINCT order_id) AS total_orders 
FROM pizza_sales
GROUP BY order_day
ORDER BY total_orders DESC;
```

#### Hourly Order Patterns
```sql
SELECT EXTRACT(HOUR FROM order_time) AS order_hour, 
       COUNT(DISTINCT order_id) AS total_orders
FROM pizza_sales
GROUP BY order_hour
ORDER BY order_hour;
```

### Category-wise Performance

#### Sales by Pizza Category
```sql
SELECT pizza_category, 
       ROUND(SUM(total_price)::numeric, 2) AS total_revenue,
       ROUND((SUM(total_price) * 100) / (SELECT SUM(total_price) FROM pizza_sales), 2) AS percentage_sales
FROM pizza_sales
GROUP BY pizza_category
ORDER BY percentage_sales DESC;
```

#### Sales by Pizza Size
```sql
SELECT pizza_size, 
       ROUND(SUM(total_price)::numeric, 2) AS total_revenue,
       ROUND((SUM(total_price) * 100) / (SELECT SUM(total_price) FROM pizza_sales), 2) AS percentage_sales
FROM pizza_sales
GROUP BY pizza_size
ORDER BY pizza_size;
```

### Product Performance Analysis

#### Top 5 Best Selling Pizzas
```sql
SELECT pizza_name, SUM(quantity) AS total_pizza_sold
FROM pizza_sales
GROUP BY pizza_name
ORDER BY total_pizza_sold DESC
LIMIT 5;
```

#### Bottom 5 Performing Pizzas
```sql
SELECT pizza_name, SUM(quantity) AS total_pizza_sold
FROM pizza_sales
GROUP BY pizza_name
ORDER BY total_pizza_sold ASC
LIMIT 5;
```

### Time-based Filtering

#### Monthly Analysis (January)
```sql
SELECT TO_CHAR(order_date, 'Day') AS order_day, 
       COUNT(DISTINCT order_id) AS total_orders 
FROM pizza_sales
WHERE EXTRACT(MONTH FROM order_date) = 1
GROUP BY order_day
ORDER BY total_orders DESC;
```

#### Quarterly Analysis (Q1)
```sql
SELECT TO_CHAR(order_date, 'Day') AS order_day, 
       COUNT(DISTINCT order_id) AS total_orders 
FROM pizza_sales
WHERE EXTRACT(QUARTER FROM order_date) = 1
GROUP BY order_day
ORDER BY total_orders DESC;
```

## Data Validation

After successful import, verify data integrity:

```sql
-- Check sample data
SELECT * FROM pizza_sales LIMIT 10;

-- Verify record count
SELECT COUNT(*) FROM pizza_sales;

-- Check for null values
SELECT COUNT(*) FROM pizza_sales WHERE order_date IS NULL;
```

## Interactive Visualization

**MARK_MAP.html** - The MARK_MAP.html file provides an interactive visualization of the pizza sales database. Using Markmap, this file converts structured Markdown into a mind map, helping users:
- Understand key pizza sales trends at a glance
- See hierarchical relationships in data insights
- Navigate complex SQL queries efficiently

Simply double-click to view the interactive [MARK_MAP.html file](https://github.com/gbadrain/Pizza_Dashboard/blob/main/MARK_MAP.html)

## Prerequisites

- PostgreSQL installed and running
- Python 3.x with Pandas library
- Access to Resource/pizza_sales.csv dataset
- Basic knowledge of SQL and Python

## Getting Started

1. **Clone or download the project files**
2. **Install required Python packages**:
   ```bash
   pip install pandas
   ```
3. **Set up PostgreSQL database**
4. **Run the Python script to clean the data**
5. **Create the database table using the provided SQL**
6. **Import the cleaned CSV file**
7. **Execute the analytical queries**

## Key Insights

This analysis enables you to understand:
- Revenue performance and trends
- Customer ordering patterns by day and hour
- Product category performance
- Best and worst performing pizzas
- Seasonal and monthly variations
- Average order characteristics

## Contact & Support

**Gurpreet Singh Badrain**  
*Market Research Analyst & Aspiring Data Analyst*

- **My portfolio**: [Data Guru](https://datascienceportfol.io/gbadrain)
- **GitHub**: [gbadrain](https://github.com/gbadrain)
- **LinkedIn**: [gurpreet-badrain](http://linkedin.com/in/gurpreet-badrain-b258a0219)
- **Email**: gbadrain@gmail.com
- **Streamlit**: [gbadrain-Machine Learnning](https://gbadrain-machine-learning.streamlit.app)


---
## **Support gratitude**  
**Copilot AI** provided valuable assistance.  
Subscriber : **datatutorials1** https://www.youtube.com/@datatutorials1



## Show Your Support

If you found this project helpful, please consider giving it a star on GitHub!

[![GitHub stars](https://img.shields.io/github/stars/gbadrain/Music_Recommendation.svg?style=social&label=Star)](https://github.com/gbadrain/Music_Recommendation)
