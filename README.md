# **Sale Management Project**

<p> This case I receive a mail from sales mananager to some requests. He wants  to improve his internet sales reports and want to move from static reports to visual dashboards.
Essentially, His teams want to focus it on how much they have sold of what products, to which clients and how it has been over time. Seeing as each sales person works on different products and customers it would be beneficial to be able to filter them also. They also measure our numbers against budget so he added that in a spreadsheet so they can compare our values against performance. 
The budget is for 2021 and they usually look 2 years back in time when they do analysis of sales.
</p>

## **Table of Content:**

1. [Business Request & User Stories](#Business-Request-&-User-Stories)
2. [Data Cleansing & Transformation (SQL)](#Data-Cleansing-&-Transformation-(SQL))
3. [Data Model](#Data-Model)
4. [Sales Management Dashboard](#Sales_Management_Dashboard)

## 1. Business Request & User Stories


The business request for this data analyst project was an executive sales report for sales managers. Based on the request that was made from the business we following user stories were defined to fulfill delivery and ensure that acceptance criteria’s were maintained throughout the project.
|     No #    |     As a (role)             |     I want (request / demand)                                |     So that I (user value)                                                        |     Acceptance Criteria                                                          |
|-------------|-----------------------------|--------------------------------------------------------------|-----------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
|     1       |     Sales Manager           |     To get a dashboard overview of   internet sales          |     Can follow better which customers and products sells   the best               |     A Power BI dashboard which updates data once a day                           |
|     2       |     Sales Representative    |     A detailed overview of Internet   Sales per Customers    |     Can follow up my customers that   buys the most and who we can sell ore to    |     A Power BI dashboard which   allows me to filter data   for each customer    |
|     3       |     Sales Representative    |     A detailed overview of Internet   Sales per Products     |     Can follow up my Products that sells the most                                 |     A Power BI dashboard which allows me to filter data   for each Product       |
|     4       |     Sales Manager           |     A dashboard overview of internet   sales                 |     Follow sales over time against budget                                         |     A Power Bi dashboard with graphs and KPIs comparing   against budget.        |

## 2. Data Cleansing & Transformation (SQL)

To create the necessary data model for doing analysis and fulfilling the business needs defined in the user stories the following tables were extracted using SQL.
One data source (sales budgets) were provided in Excel format and were connected in the data model in a later step of the process.
Below are the SQL statements for cleansing and transforming necessary data.

### 2.1 Dim_Date

```sql
-- Cleansed DIM_Date Table --
SELECT 
  [DateKey], 
  [FullDateAlternateKey] AS Date, 
  --[DayNumberOfWeek], 
  [EnglishDayNameOfWeek] AS Day, 
  --[SpanishDayNameOfWeek], 
  --[FrenchDayNameOfWeek], 
  --[DayNumberOfMonth], 
  --[DayNumberOfYear], 
  --[WeekNumberOfYear],
  [EnglishMonthName] AS Month, 
  Left([EnglishMonthName], 3) AS MonthShort,   -- Useful for front end date navigation and front end graphs.
  --[SpanishMonthName], 
  --[FrenchMonthName], 
  [MonthNumberOfYear] AS MonthNo, 
  [CalendarQuarter] AS Quarter, 
  [CalendarYear] AS Year --[CalendarSemester], 
  --[FiscalQuarter], 
  --[FiscalYear], 
  --[FiscalSemester] 
FROM 
 [AdventureWorksDW2019].[dbo].[DimDate]
WHERE 
  CalendarYear >= 2019
```
### 2.2. DIM_Customers:
```sql
-- Cleansed DIM_Customers Table --
SELECT 
  c.customerkey AS CustomerKey, 
  --      ,[GeographyKey]
  --      ,[CustomerAlternateKey]
  --      ,[Title]
  c.firstname AS [First Name], 
  --      ,[MiddleName]
  c.lastname AS [Last Name], 
  c.firstname + ' ' + lastname AS [Full Name], 
  -- Combined First and Last Name
  --      ,[NameStyle]
  --      ,[BirthDate]
  --      ,[MaritalStatus]
  --      ,[Suffix]
  CASE c.gender WHEN 'M' THEN 'Male' WHEN 'F' THEN 'Female' END AS Gender,
  --      ,[EmailAddress]
  --      ,[YearlyIncome]
  --      ,[TotalChildren]
  --      ,[NumberChildrenAtHome]
  --      ,[EnglishEducation]
  --      ,[SpanishEducation]
  --      ,[FrenchEducation]
  --      ,[EnglishOccupation]
  --      ,[SpanishOccupation]
  --      ,[FrenchOccupation]
  --      ,[HouseOwnerFlag]
  --      ,[NumberCarsOwned]
  --      ,[AddressLine1]
  --      ,[AddressLine2]
  --      ,[Phone]
  c.datefirstpurchase AS DateFirstPurchase, 
  --      ,[CommuteDistance]
  g.city AS [Customer City] -- Joined in Customer City from Geography Table
FROM 
  [AdventureWorksDW2019].[dbo].[DimCustomer] as c
  LEFT JOIN dbo.dimgeography AS g ON g.geographykey = c.geographykey 
ORDER BY 
  CustomerKey ASC -- Ordered List by CustomerKey


```
### 2.3  DIM_Products:

```sql
-- Cleansed DIM_Products Table --
SELECT 
  p.[ProductKey], 
  p.[ProductAlternateKey] AS ProductItemCode, 
  --      ,[ProductSubcategoryKey], 
  --      ,[WeightUnitMeasureCode]
  --      ,[SizeUnitMeasureCode] 
  p.[EnglishProductName] AS [Product Name], 
  ps.EnglishProductSubcategoryName AS [Sub Category], -- Joined in from Sub Category Table
  pc.EnglishProductCategoryName AS [Product Category], -- Joined in from Category Table
  --      ,[SpanishProductName]
  --      ,[FrenchProductName]
  --      ,[StandardCost]
  --      ,[FinishedGoodsFlag] 
  p.[Color] AS [Product Color], 
  --      ,[SafetyStockLevel]
  --      ,[ReorderPoint]
  --      ,[ListPrice] 
  p.[Size] AS [Product Size], 
  --      ,[SizeRange]
  --      ,[Weight]
  --      ,[DaysToManufacture]
  p.[ProductLine] AS [Product Line], 
  --     ,[DealerPrice]
  --      ,[Class]
  --      ,[Style] 
  p.[ModelName] AS [Product Model Name], 
  --      ,[LargePhoto]
  p.[EnglishDescription] AS [Product Description], 
  --      ,[FrenchDescription]
  --      ,[ChineseDescription]
  --      ,[ArabicDescription]
  --      ,[HebrewDescription]
  --      ,[ThaiDescription]
  --      ,[GermanDescription]
  --      ,[JapaneseDescription]
  --      ,[TurkishDescription]
  --      ,[StartDate], 
  --      ,[EndDate], 
  ISNULL (p.Status, 'Outdated') AS [Product Status] 
FROM 
  [AdventureWorksDW2019].[dbo].[DimProduct] as p
  LEFT JOIN dbo.DimProductSubcategory AS ps ON ps.ProductSubcategoryKey = p.ProductSubcategoryKey 
  LEFT JOIN dbo.DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey 
order by 
  p.ProductKey asc
```

### 2.4: FACT_InternetSales:

```sql
-- Cleansed FACT_InternetSales Table --
SELECT 
  [ProductKey], 
  [OrderDateKey], 
  [DueDateKey], 
  [ShipDateKey], 
  [CustomerKey], 
  --  ,[PromotionKey]
  --  ,[CurrencyKey]
  --  ,[SalesTerritoryKey]
  [SalesOrderNumber], 
  --  [SalesOrderLineNumber], 
  --  ,[RevisionNumber]
  --  ,[OrderQuantity], 
  --  ,[UnitPrice], 
  --  ,[ExtendedAmount]
  --  ,[UnitPriceDiscountPct]
  --  ,[DiscountAmount] 
  --  ,[ProductStandardCost]
  --  ,[TotalProductCost] 
  [SalesAmount] --  ,[TaxAmt]
  --  ,[Freight]
  --  ,[CarrierTrackingNumber] 
  --  ,[CustomerPONumber] 
  --  ,[OrderDate] 
  --  ,[DueDate] 
  --  ,[ShipDate] 
FROM 
  [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE 
  LEFT (OrderDateKey, 4) >= YEAR(GETDATE()) -2 -- Ensures always only bring two years of date from extraction.
ORDER BY
  OrderDateKey ASC

```
## 3.  Data Model

Below is a screenshot of the data model after cleansed and prepared tables were read into Power BI.
This data model also shows how FACT_Budget has been connected to FACT_InternetSales and other necessary DIM 
tables.
![DATAMODEL](https://user-images.githubusercontent.com/88467188/131869397-41dc270e-bf65-4bea-9c4e-571aebf74a08.png)

## 4. Sales Management Dashboard

The finished sales management dashboard with one page with works as a dashboard and overview, with two other pages focused on combining tables for necessary details and visualizations to show sales over time, per customers and per products.
![POWER_BI](https://user-images.githubusercontent.com/88467188/131869407-49a2786e-d7b5-4d9a-9aaa-b17800c56509.png)


## 5.	Power BI Services.

I published dashboards on Power BI services so anyone can open and interact with [Sale Management dashboard](https://app.powerbi.com/view?r=eyJrIjoiYjE1Y2Q5NjMtMjg5OC00OWU1LWFjNjUtNGNjNTllN2E3OTc4IiwidCI6IjM1ZTE1M2EzLTViYzgtNGZjMC04YmZhLTVkNDFhZmQ0NDU0NSIsImMiOjN9)

