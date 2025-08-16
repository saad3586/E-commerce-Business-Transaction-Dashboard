**Data Analyst Assignment Task Completed Solutions**

**Task 1: Assessment Form (MCQs)**
Since this is a link-based MCQ test (SQL basics, Excel functions, data visualization), your best prep strategy is:
•	SQL Basics: Focus on SELECT, WHERE, GROUP BY, JOIN, DISTINCT, ORDER BY, and aggregate functions (SUM, COUNT, AVG).
•	Excel Functions: Practice VLOOKUP, INDEX-MATCH, IF, SUMIF, COUNTIF, TEXT, LEFT/RIGHT/MID, CONCATENATE, PivotTables.
•	Data Visualization: Understand when to use bar charts vs. pie charts vs. line charts, basics of dashboards, KPIs, and storytelling with data.

**Task 2: Business Dashboard Creation**
Step 1: Get the Dataset
Dataset fields:
•	order_id
•	category
•	sales_amount
•	quantity
•	region (or city)
•	customer (optional)
•	date
________________________________________
1) Import & Clean
1.	Open Power BI Desktop → Home → Get data → Excel → pick your file → select the sheet → Load.
2.	Transform Data (Power Query):
o	Ensure data types:
	Order ID: Text
	Product Category: Text
	Sales Amount: Decimal Number
	Quantity Sold: Whole Number
	Region/City: Text → set Data category = City later in Report view
	Customer Name (optional): Text
	Order Date: Date
o	Remove blanks/duplicates: Home → Remove Rows → Remove Blank Rows; Remove Duplicates (on Order ID if each row is an order; skip if rows are order-lines).
o	Close & Apply.
Tip: If rows are order lines (same Order ID appears multiple times), use DISTINCTCOUNT for orders; if each row is a unique order, COUNT is fine.
________________________________________
2) Create a Date Table (for monthly trends)
Modeling → New table (DAX):
Date = CALENDAR(MIN(Sales[Order Date]), MAX(Sales[Order Date]))
Add helper columns (Modeling → New column):
Year = YEAR('Date'[Date])
Month Number = MONTH('Date'[Date])
Month Name = FORMAT('Date'[Date], "MMM")
Year-Month = FORMAT('Date'[Date], "YYYY-MM")
Modeling → Mark as date table → select 'Date'[Date].
Create relationship: Model view → drag 'Date'[Date] → Sales[Order Date] (Many-to-one, single direction from Date to Sales).
________________________________________
3) Measures (KPIs)
Modeling → New measure (put these on the Sales table):
Total Sales = SUM(Sales[Sales Amount])

-- Use ONE of the following for orders:
Total Orders = DISTINCTCOUNT(Sales[Order ID])  -- safer if order lines exist
-- or, if guaranteed 1 row per order:
-- Total Orders = COUNT(Sales[Order ID])

Avg Order Value = DIVIDE([Total Sales], [Total Orders])

Sales Last Month = CALCULATE([Total Sales], DATEADD('Date'[Date], -1, MONTH))
MoM % = DIVIDE([Total Sales] - [Sales Last Month], [Sales Last Month])

Cumulative Sales = 
CALCULATE(
    [Total Sales],
    FILTER(ALLSELECTED('Date'[Date]), 'Date'[Date] <= MAX('Date'[Date]))
)
________________________________________
4) Build the Report (Report view)
A) KPI Cards
1.	Insert → Card (three cards).
2.	Set Fields:
o	Card 1: [Total Sales]
o	Card 2: [Total Orders]
o	Card 3: [Avg Order Value]
3.	Format → Data label display units (Auto/Thousands), currency for Sales/AOV.
B) Time Trend (Line Chart: Sales Over Time – Monthly)
1.	Insert → Line chart.
2.	Axis: 'Date'[Year-Month] (sort by Month Number if you use Month Name).
3.	Values: [Total Sales].
4.	Optional: Add [Cumulative Sales] as a second line.
5.	Add slicers for 'Date'[Year] and 'Date'[Month Name] if desired.
If the X-axis shows out-of-order months, click the Month Name column → Column tools → Sort by column → Month Number.
C) Category-wise Sales (Bar or Pie)
•	Clustered bar chart (recommended over pie for readability).
o	Axis: Sales[Product Category]
o	Values: [Total Sales]
•	Optional tooltip: add [Total Orders], [Avg Order Value] to Tooltips well.
D) Region-wise Performance (Map or Bar)
•	For map:
1.	Insert → Map.
2.	Location: Sales[Region/City] (Report view → Column tools → Data category = City).
3.	Size: [Total Sales] (or [Total Orders]).
•	If geo ambiguity occurs, switch to a bar chart by Region/City.
E) Top 5 Products / Customers
Option 1: Visual-level “Top N” filter
1.	Insert → Bar chart.
2.	Axis: Sales[Customer Name] (or Sales[Product Category] if you want top products).
3.	Values: [Total Sales] (or SUM(Sales[Quantity Sold])).
4.	On the Visual’s Filters pane → for the field on Axis → Filter type = Top N → Show items = 5 → By value = [Total Sales] → Apply filter.
Option 2: DAX Rank (if you need more control)
Sales by Customer = [Total Sales]

Customer Rank by Sales = 
RANKX(
    ALL(Sales[Customer Name]),
    [Sales by Customer],
    ,
    DESC,
    Dense
)
Top 5 Customer Sales = IF([Customer Rank by Sales] <= 5, [Sales by Customer])
Use Top 5 Customer Sales as Values and filter where it’s not blank.
________________________________________
5) Slicers & Interactions
•	Add Slicers for: 'Date'[Year], Sales[Product Category], Sales[Region/City].
•	Format → Edit interactions if a slicer should not affect some visuals.
•	Turn on Visual header tooltips and Data labels where helpful.

