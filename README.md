# Sales-Performance-Analysis-SQL-Project
This project provides an end-to-end analysis of sales performance using SQL. It focuses on transforming raw transactional data into meaningful insights that help stakeholders understand revenue trends, product performance, customer behavior, and overall business growth.
---Retreive data--
select * from [sales_data_sample-1]

--Total count
select count(*) from [sales_data_sample-1];

-- Total sales in millions
SELECT cast(SUM(SALES)/1000000 as decimal(10,2)) AS Total_Sales_Millions
FROM [sales_data_sample-1];

-- Last 7 days sales
SELECT 
    SUM(SALES) AS Total_Sales_Last_7_Days
FROM 
    [sales_data_sample-1]
WHERE 
    ORDERDATE >= DATEADD(DAY, -7, (SELECT MAX(ORDERDATE) FROM [sales_data_sample-1]));

-- Yesterday Sales
SELECT 
    SUM(SALES) AS Yesterday_Sales
FROM 
    [sales_data_sample-1]
WHERE 
    CAST(ORDERDATE AS DATE) = 
        DATEADD(DAY, -1, (SELECT CAST(MAX(ORDERDATE) AS DATE) FROM [sales_data_sample-1]));

-- Running Total Sales

SELECT  
    CAST(ORDERDATE AS DATE) AS Order_Date,
    SUM(SUM(SALES)) OVER (ORDER BY CAST(ORDERDATE AS DATE)) AS Running_Total_Sales
FROM  
    [sales_data_sample-1]
GROUP BY  
    CAST(ORDERDATE AS DATE)
ORDER BY  
    Order_Date;
-- Total sales by Month and Territory
SELECT  
    FORMAT(ORDERDATE, 'yyyy-MMM') AS YearMonth,
    TERRITORY,
    SUM(SALES) AS Total_Sales
FROM  
    [sales_data_sample-1]
GROUP BY  
    FORMAT(ORDERDATE, 'yyyy-MMM'),
    TERRITORY
ORDER BY  
    MIN(ORDERDATE),
    TERRITORY;

--Order count filtered by year and order date
select COUNT(ORDERNUMBER) from [sales_data_sample-1] where YEAR(ORDERDATE) = 2005 and TERRITORY = 'EMEA';

-- Total shipped order
select COUNT(ORDERNUMBER) from [sales_data_sample-1] where YEAR(ORDERDATE) = 2005 and TERRITORY = 'EMEA' and STATUS = 'Shipped';

---Order Count by Status--
SELECT 
    STATUS,
    COUNT(*) AS Order_Count
FROM 
    [sales_data_sample-1]
WHERE 
    YEAR(ORDERDATE) = 2005
    AND TERRITORY = 'EMEA'
GROUP BY 
    STATUS
ORDER BY 
    Order_Count DESC;


--Total Orders by Territory & Status--

SELECT
    TERRITORY,
    STATUS,
    COUNT(ORDERNUMBER) AS Total_Orders
FROM
    [sales_data_sample-1]
WHERE 
    YEAR(ORDERDATE) = 2005
GROUP BY
    TERRITORY, STATUS
ORDER BY
    TERRITORY, STATUS;

--Monthly Order Count by Product--
SELECT
    PRODUCTLINE,
    SUM(CASE WHEN MONTH(ORDERDATE) = 1 THEN 1 ELSE 0 END) AS January,
    SUM(CASE WHEN MONTH(ORDERDATE) = 2 THEN 1 ELSE 0 END) AS February,
    SUM(CASE WHEN MONTH(ORDERDATE) = 3 THEN 1 ELSE 0 END) AS March,
    SUM(CASE WHEN MONTH(ORDERDATE) = 4 THEN 1 ELSE 0 END) AS April,
    SUM(CASE WHEN MONTH(ORDERDATE) = 5 THEN 1 ELSE 0 END) AS May,
    SUM(CASE WHEN MONTH(ORDERDATE) = 6 THEN 1 ELSE 0 END) AS June,
    SUM(CASE WHEN MONTH(ORDERDATE) = 7 THEN 1 ELSE 0 END) AS July,
    SUM(CASE WHEN MONTH(ORDERDATE) = 8 THEN 1 ELSE 0 END) AS August,
    SUM(CASE WHEN MONTH(ORDERDATE) = 9 THEN 1 ELSE 0 END) AS September,
    SUM(CASE WHEN MONTH(ORDERDATE) = 10 THEN 1 ELSE 0 END) AS October,
    SUM(CASE WHEN MONTH(ORDERDATE) = 11 THEN 1 ELSE 0 END) AS November,
    SUM(CASE WHEN MONTH(ORDERDATE) = 12 THEN 1 ELSE 0 END) AS December,
    COUNT(*) AS Total
FROM
    [sales_data_sample-1]
WHERE 
    YEAR(ORDERDATE) = 2005
GROUP BY PRODUCTLINE
ORDER BY PRODUCTLINE;


--Sales by Product Line--
SELECT
    PRODUCTLINE,
    SUM(SALES) AS Total_Sales
FROM 
    [sales_data_sample-1]
WHERE 
    YEAR(ORDERDATE) = 2003
GROUP BY 
    PRODUCTLINE
ORDER BY 
    Total_Sales DESC;

--Sales by Deal Size--

   SELECT DEALSIZE,
    SUM(SALES) AS Total_Sales
FROM 
    [sales_data_sample-1]
WHERE 
    YEAR(ORDERDATE) = 2003
GROUP BY 
    DEALSIZE
ORDER BY 
    Total_Sales DESC;

--Yearly Sales Summary--
SELECT 
    YEAR(ORDERDATE) AS Year,
    SUM(SALES) AS Total_Sales
FROM 
    [sales_data_sample-1]
GROUP BY 
    YEAR(ORDERDATE)
ORDER BY 
    Year;

--Top 10 Customers by Sales--
SELECT 
    CUSTOMERNAME,
    SUM(SALES) AS Total_Sales
FROM 
    [sales_data_sample-1]
GROUP BY 
    CUSTOMERNAME
ORDER BY 
    Total_Sales DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;

--Monthly Sales Growth (MoM Growth %)--
WITH MonthlySales AS (
    SELECT 
        FORMAT(ORDERDATE,'yyyy-MM') AS YearMonth,
        SUM(SALES) AS TotalSales
    FROM 
        [sales_data_sample-1]
    GROUP BY 
        FORMAT(ORDERDATE,'yyyy-MM')
)
SELECT
    YearMonth,
    TotalSales,
    LAG(TotalSales) OVER (ORDER BY YearMonth) AS PreviousMonthSales,
    ((TotalSales - LAG(TotalSales) OVER (ORDER BY YearMonth)) * 100.0 
         / NULLIF(LAG(TotalSales) OVER (ORDER BY YearMonth),0)) AS GrowthPercent
FROM 
    MonthlySales;

--Average Order Value--
SELECT 
    SUM(SALES) * 1.0 / COUNT(ORDERNUMBER) AS Average_Order_Value
FROM 
    [sales_data_sample-1];

--Customer Repeat Purchase Count--
SELECT
    CUSTOMERNAME,
    COUNT(DISTINCT ORDERNUMBER) AS Order_Count
FROM 
    [sales_data_sample-1]
GROUP BY 
    CUSTOMERNAME
HAVING 
    COUNT(DISTINCT ORDERNUMBER) > 1
ORDER BY 
    Order_Count DESC;






