# Data Dictionary — E-Commerce Sales & Profit Analysis

This file explains every column, measure and analysis sheet in `e_commerce_analysis.xlsx`, in plain words.

> **Think of it like a label on a jar:** the data is the food, and this page tells you exactly what is inside each jar, where it came from, and how it was prepared.

---

## 1. Dataset at a glance

| Item | Value |
|---|---|
| Workbook | `e_commerce_analysis.xlsx` |
| Main data sheet | `E-commerce` (Excel table name: `E_commerce`, also loaded into the Data Model) |
| Rows | 3,203 (each row is one **product line inside an order**) |
| Columns | 20 |
| Unique orders | 1,611 |
| Unique customers | 686 (identified by email) |
| Unique products | 1,494 |
| Product categories | 17 |
| States | 11 (all in the United States, western region) |
| Cities | 169 |
| Order date range | 7 Jan 2011 – 31 Dec 2014 |
| Total sales | $725,457.82 |
| Total profit | $108,418.45 |
| Overall profit margin | 14.94% |
| Missing values | None in any column |

**Grain (what one row means):** one row = one product on one order. An order with 3 different products takes 3 rows. That is why there are 3,203 rows but only 1,611 orders, so always count **distinct** Order IDs when you want the number of orders.

---

## 2. Column dictionary (sheet: `E-commerce`)

| # | Column | Data type | Origin | What it means (plain words) | Example |
|---|---|---|---|---|---|
| 1 | `Order ID` | Text | Source | Unique code of the order. Repeats on several rows when an order has several products. Format: `CA-<year>-<number>`. | `CA-2013-138688` |
| 2 | `Order Date` | Date | Source | The day the customer placed the order. | 2013-06-13 |
| 3 | `Order Year` | Whole number | Derived | Year taken from `Order Date`. Values: 2011–2014. | 2013 |
| 4 | `Month` | Text | Derived | Full month name taken from `Order Date`. | June |
| 5 | `Month No.` | Number (1–12)* | Derived | Month as a number, used to sort months in the right order (January first, not alphabetical). | 6 |
| 6 | `Year-Month` | Text | Derived | Year and month joined as `yyyy-MM`. Handy for monthly trend charts. 48 values. | 2013-06 |
| 7 | `Ship Date` | Date | Source | The day the order was shipped. | 2013-06-17 |
| 8 | `EmailID` | Text | Source (cleaned) | The customer's email, converted to lowercase. It is the **customer key**: the dataset has no customer name column. | darrinvanhuff@gmail.com |
| 9 | `Geography` | Text | Source | Country, city and state joined in one text, separated by commas. | United States,Los Angeles,California |
| 10 | `Geography_1` | Text | Source | The country part of `Geography`. Always `United States`, so it carries no extra information for analysis. | United States |
| 11 | `city` | Text | Source (fixed) | City of the delivery address. 169 cities. | Los Angeles |
| 12 | `state` | Text | Source (fixed) | State of the delivery address. 11 states. | California |
| 13 | `Category` | Text | Source | The product group. 17 values (for example Chairs, Phones, Binders). These are really product *sub-categories*, so there is no higher-level group such as "Furniture" in this file. | Labels |
| 14 | `Product Name` | Text | Source | Full product name. 1,494 different products. | Wilson Jones Active Use Binders |
| 15 | `Unit Selling Price` | Currency | Derived | Price for one unit = `Sales ÷ Quantity`. | 7.31 |
| 16 | `Sales` | Currency | Source | Money earned from the row (selling price × quantity). Range: 0.99 to 13,999.96. | 14.62 |
| 17 | `Quantity` | Whole number | Source | Number of units bought in that row. Range: 1 to 14. | 2 |
| 18 | `Profit` | Currency | Source | Money left after costs. **Negative means a loss.** Range: −3,399.98 to 6,719.98. | 6.8714 |
| 19 | `Profit Margin` | Decimal (ratio)* | Derived | Profit as a share of sales = `Profit ÷ Sales`. 0.47 means 47%. Range: −2.10 to 0.50. | 0.47 |
| 20 | `Shipping Days` | Whole number | Derived | Days between ordering and shipping = `Ship Date − Order Date`. Range: 0 to 7. Average: 3.93. | 4 |

\* **Heads-up:** in the worksheet table, `Month No.` and `Profit Margin` are stored as **text** (they sit left-aligned in the cells). If you use them in normal Excel formulas outside the Data Model, convert them to numbers first (for example with `VALUE()`).

---

## 3. How the columns were prepared (Power Query)

The query is named `E-commerce`. It reads the sheet `ecommerce` from a separate workbook called `ecommerce.xlsx`. Think of Power Query as a kitchen prep station: raw ingredients go in, and clean, ready-to-use ingredients come out, every time you press Refresh.

| Step | What was done | Why it matters |
|---|---|---|
| 1 | Promoted the first row to column headers | So columns have real names |
| 2 | Set data types: text, dates, numbers, whole numbers | Prevents dates being read as text and sums failing |
| 3 | Swapped the `state` and `city` labels | In the source, the two names were the wrong way round |
| 4 | Set `Sales` and `Profit` to Currency type | Correct money formatting |
| 5 | Added `Unit Selling Price` = `Sales / Quantity` | Price per unit |
| 6 | Added `Profit Margin` = `Profit / Sales` | Profitability as a ratio |
| 7 | Read `Order Date` and `Ship Date` using the **English (India)** locale | Makes day/month order read correctly in dates |
| 8 | Added `Shipping Days` = `Duration.Days([Ship Date] - [Order Date])` | Delivery speed measure |
| 9 | Added `Order Year`, `Month`, `Month No.` and `Year-Month` from `Order Date` | Time-based analysis and sorting |
| 10 | Lowercased `EmailID` | Stops the same customer being counted twice because of capital letters |
| 11 | Reordered columns | Easier to read |

---

## 4. DAX measures (Power Pivot / Data Model)

The workbook's Data Model holds one table (`E-commerce`) and eight measures. A **measure** is a reusable formula that recalculates automatically for whatever filter you pick (a year, a state, a category).

The plain meaning and a matching DAX formula are given below. These formulas are written to match the results shown in the workbook (the grand totals tie out exactly). If your original DAX in Power Pivot is worded differently, trust the original, and update this table to match it.

| Measure | Plain meaning | Equivalent DAX | Grand total in workbook |
|---|---|---|---|
| `Total Sales` | All money earned | `SUM('E-commerce'[Sales])` | $725,457.82 |
| `Total Profit` | All profit (losses reduce it) | `SUM('E-commerce'[Profit])` | $108,418.45 |
| `Profit_Margin` | Profit per dollar of sales | `DIVIDE([Total Profit], [Total Sales])` | 14.94% |
| `Total Quantity` | Units sold | `SUM('E-commerce'[Quantity])` | 12,264 |
| `Distinct Orders` | Number of different orders | `DISTINCTCOUNT('E-commerce'[Order ID])` | 1,611 |
| `Distinct Customers` | Number of different customers | `DISTINCTCOUNT('E-commerce'[EmailID])` | 686 |
| `Avg Shipping Days` | Average days from order to shipping | `AVERAGE('E-commerce'[Shipping Days])` | 3.93 |
| `Average Order Value (AOV)` | Average sales per order | `DIVIDE([Total Sales], [Distinct Orders])` | about $450.31 |

**Why `Profit_Margin` is a measure and not an average of the column:** averaging row-level margins gives tiny orders the same weight as huge ones. Dividing total profit by total sales is the correct, fair way to get a margin for any group.

---

## 5. Analysis sheets

| Sheet | What it contains | Main fields used |
|---|---|---|
| `E-commerce` | The clean, row-level data (see section 2) | All 20 columns |
| `product Analysis` | Year summary, category-by-year sales grid, category summary, state summary, and a Colorado drill-down by category and product | `Order Year`, `Category`, `state`, `Product Name`, `Total Sales`, `Total Profit`, `Profit_Margin`, `Total Quantity` |
| `Customer Analysis` | One row per customer (686 rows plus a grand total) | `EmailID`, `Total Sales`, `Total Profit`, `Profit_Margin`, `Distinct Orders`, `Total Quantity` |
| `Shipping Analysis` | Average shipping days by category and by state | `Category`, `state`, `Avg Shipping Days`, `Total Sales`, `Distinct Orders`, `Distinct Customers`, `Total Quantity`, `Total Profit` |
| `Dashboard` | Final one-page view: 6 KPI cards, 6 charts and 6 insight call-outs | KPI cards: Total Sales, Total Profit, Profit Margin, Total Quantity, Total Orders, Customers. Charts: Total Sales & Profit by Year, Avg Shipping Days by State, Total Sales by State (map), Total Sales by Category, Total Profit by Category, Top 5 Customers |

The workbook contains **8 PivotTables**, all connected to the Data Model.

---

## 6. Data quality notes and things to know

1. **No missing values.** All 20 columns are fully filled in across 3,203 rows.
2. **Many rows per order.** Count orders with a distinct count of `Order ID`, never with a plain row count.
3. **Customers are emails.** There is no customer-name field. Names such as "Raymond Buch" in the insights come from reading the email address (`raymondbuch@gmail.com`).
4. **`Category` is really a sub-category.** There are 17 product types and no parent group.
5. **Losses are real values.** Negative `Profit` rows are kept on purpose, because they drive the loss-making findings.
6. **Extreme margins exist.** `Profit Margin` goes as low as −2.10 (a loss of 210% of sales). For example, a Lexmark laser printer sold in Colorado shows a loss of $3,399.98 on sales of $2,549.99.
7. **Some shipping dates fall in 2015.** Orders placed at the end of December 2014 ship in early January 2015 (latest ship date: 6 Jan 2015). This is normal, but a "ship year" filter would not match the "order year".
8. **Same-day shipping is possible.** `Shipping Days` can be 0.
9. **Small groups give shaky averages.** Wyoming has only 1 order, so its 5.0-day shipping average should not be compared with California's 1,021 orders.
10. **`Geography_1` is constant** (always United States), and `Geography` repeats information already in `city` and `state`. They can be hidden from reports.
11. **Currency symbol.** The dashboard shows dollar ($) values. The data itself stores plain currency numbers with no symbol.
12. **Refreshing needs the source file.** The Power Query connection points to a local file named `ecommerce.xlsx`. To refresh on another computer, update the file path in Power Query (Data > Queries & Connections > Edit > Source).
