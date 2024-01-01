# DAX Note

## 1. Data Modeling
- CARDINALITY - the uniqueness of values in a column
- FILTER FLOW - from one side relationship to many side relationship (downstream)

## 2. DAX Review
- `CALCULATE` works similarly like `SUMIF` or `COUNTIF` in Excel
- `CALCULATE` creates new filter context
- Variable evalation order: can't modify how a variable defined later in the `RETURN` statement (i.e. through `CALCULATE`)
  
## 3. Scalar functions
- Difference between Tabular Functions and Scalar Functions in Power BI [-> Read this](https://radacad.com/power-bi-dax-back-to-basics-scalar-vs-tabular-functions)
- We use `CALCULATE` with `FILTER` as an argument when:
  + Aiming to show only values that are the same as the filter(s), other cells will remain `BLANK`
  + Comparing column value > column value; column value > measure; measure > measure
  + The filter has many `OR` conditions
  [-> Read this](https://community.powerbi.com/t5/Desktop/DAX-Calculate-function-with-and-without-FILTER/m-p/679222).
- Why we should always use `FILTER` in `CALCULATE` function? [-> Read (1)](https://blog.enterprisedna.co/how-to-use-simple-filters-in-power-bi) and [(2)](https://stackoverflow.com/questions/72000696/why-calculate-is-not-modifying-filter-context-when-used-with-filter)
- Use `KEEPFILTER` as an argument, instead of `FILTER` to preserve the existing filter [-> Read this](https://learn.microsoft.com/en-us/dax/best-practices/dax-avoid-avoid-filter-as-filter-argument)

## 4. Aggregation functions
- For large datasets (1M+ rows), using `COUNTROWS` & `VALUES` may put less strain on the DAX engines than `DISTINCTCOUNT`
- `SWITCH(True)` is the common pattern to replace nested `IF`
- `COALESCE` replaces `IF` & `ISBLANK`

## 5. Advanced `CALCULATE`
- By default, calculated columns understand row context but not filter context. To create filter context at the row-level, you can use `CALCULATE`
- As a measure (eg. `SUM`), DAX automatically evaluates a `CALCULATE` & `SUMX` measure, to create the filter context needed to produce the correct values
- Anytime you write a function that contains the logical statement (`IN`, >, <, =, etc.), you've create a table (internally processed with `FILTER` & `ALL`)
- `ALL` modifier is evaluated first, so it is overwritten by other filters in the logical statements of `CALCUALATE`
- Common `CALCULATE` modifiers:
  + Modify filters: `ALL`, `ALLSELECTED`, `ALLSELECTED`, `KEEPFILTERS`, `REMOVEFILTERS` 
  + Use relationships: `USERELATIONSHIP`
  + Change filter propagation: `CROSSFILTER`
- `CALCULATE` evaluates all kinds of filter arguments as a table
- `REMOVEFILTERS` is an alias for `ALL`, but can only be used as a `CALCULATE` modifier (not as a table function). 
- Differences between `REMOVEFILTERS` and `ALL` [->Read this](https://www.sqlbi.com/articles/managing-all-functions-in-dax-all-allselected-allnoblankrow-allexcept)
  + `REMOVEFILTERS` is just an alias of `ALL`, it works just the same. Basically, `ALL` returns a table including all rows, ignoring any filters that might have been applied. However, when `ALL` is used as a filter argument of `CALCULATE` or `CALCULATETABLE`, it behave totally differently: it removes filters from the table and does not return a table. To alleviate this confusing behavior of `ALL`, `REMOVEFILTERS` was introduced to replace `ALL` when it is used inside `CALCULATE`.
- `KEEPFILTERS` does not remove an existing column or table filter for an individual `CALCULATE` expression, but adds new filter context (like Inner Join)
  + `KEEPFILTERS`: only show the value where the initial filter context is the same
  + `CALCULATE` modifier: always show the value regardless of any selected filter

## 6. Table & Filter functions
- Return columns or tables rather than scalar values, and can be used to either generate new data or serve as table inputs within DAX measures.
   + Filter Data: `ALL`, `FILTER`, `DISTINCT`, `VALUES`, `ALLEXCEPT`, `ALLSELECTED`
   + Add Data: `SELECTEDCOLUMNS`, `ADDCOLUMNS`, `SUMMARIZE`
   + Create Data: `ROW`, `DATATABLE`, `GENERATESERIES`, { } Table Constructor
- DAX functions can accept either physical table or calculated / virtual table (with function like `FILTER`, `ALL`)
- `FILTER` is both a table function and an iterator. It is often used to reduce the number of rows to scan
- `ALL` is both a table filter and a `CALCULATE` modifier. It removes initial filter context and does not accept table expression (only physical table references)
- When the input parameter is a *column name*, `VALUES` returns a one-column table that contains the distinct values from the specified column. Duplicate values are removed and only unique values are returned. A `BLANK` value can be added. When the input parameter is a *table name*, `VALUES` returns the rows from the specified table. Duplicate rows are preserved. A BLANK row can be added.
- `VALUES` will always show the blank row but `DISTINCT` will not (use `VALUES` to check if your lookup tables have missing or duplicate values)
- `SELECTEDVALUE` returns a value when there's only one value in a specified column, otherwise returns an (optional) alternate result. It can be interpretated as the combination of `IF`, `HASONEVALUE`, and `VALUES`.
  + `SELECTEDVALUE` can be used to retrieve a column from the same table [-> Read this](https://www.sqlbi.com/articles/using-the-selectedvalue-function-in-dax) 
  + `SELECTEDVALUE` can be used to display details, not subtotals or grand totals of a table
- `ALLEXCEPT(TableName, ColumnName, ColumnName...)` is typically used as a `CALCULATE` modifier and not a stand-alone table function. TableName must be physical. ColumnName must be in the same reference table or on the one-side of the relationship (Expanded Table).
- `ALLSELECTED` works the same as `ALL` but only applies to selected filters [-> Read this](https://www.thedataschool.com.au/marina-ustinova/power-bi-dax-functions-allselected-and-all)
- The differences 
  - `ALLSELECTED`: removes only filters applied to the specified columns in the query
  - `ALLEXCEPT`: removes all filters except the filters applied to the specified columns in the query
  - `SELECTCOLUMNS`: returns a table with selected columns from the table plus any new columns specified by the DAX expressions. It can accept the virtual table (`FILTER`, `ALL`).
- `ADDCOLUMNS` & `SELECTCOLUMNS` are nearly identical and behave similarly with an exception, `SELECTCOLUMNS` starts with a blank table whereas `ADDCOLUMNS` starts with the entire original table and tacks on columns
- `SUMMARIZE` works similarly as `SELECT DISTINCT` in SQL

## 7. Calculated table joins
- `CROSSJOIN` returns a table that contains the cartesian product of the specified tables
  + Number of rows = product of rows in all tables
  + Number of columns = sum of columns in all tables
  + Column names must be different in all table arguments
  + Tables can be either physical table like 'Product Lookup' or table expression like `VALUES`, `DISTINCT`, `FILTER`, etc.
- `UNION` combines rows from 2 or more tables sharing the same column structure
  + Tables can be either physical tables or virtual tables generated by DAX like `DATATABLE`
- `EXCEPT(LeftTable, RightTable)` returns all rows from the left table which do not appear in the right table
  + LeftTable must be the physical table in your data model
  + Both tables must have the same number of columns
  + Column names are determined by the left table
  + The resulting table does not retain relationships to other tables (expanded table)
- `INTERSECT`returns all rows from the left table which also appear in the right table
  + LeftTable must be the physical table in your data model
  + Order matters (T1, T2) may have a different result set than (T2, T1)
  + Columns are compared based on positioning in their respective tables
  + Duplicate rows are retained
  + Column names are determined by the left table
- Difference between `CALCULATE` & `CALCULATETABLE`
  + `CALCULATE`: returns a scalar value (for measures)
  + `CALCULATETABLE`: returns a virtual table (for tables) [-> Read this](https://community.powerbi.com/t5/Community-Blog/CALCULATE-amp-CALCULATETABLE-What-s-The-Real-Difference/ba-p)

## 8. Relationship Functions
- `RELATED`: Currently in the "many" side, want to specify data from the "one" side of the relationship
- `RELATEDTABLE`: Currently in the "one" side, want to aggregate data from the "many" side of the relationship 
  + is commonly used with aggregators like `COUNTROWS`, `SUMX`, `AVERAGEX`, etc.
  + `COUNTROWS ( RELATEDTABLE ( 'Food Inventory' ) )`
  + `SUMX ( RELATEDTABBLE ( 'Food Inventory' ), [Quantity Sold] * [Retail Price] )`
  + `RELATEDTABLE` is shortcut for `CALCULATEDTABLE` (without logical expressions) and performs a context transition from row context to filter context, in order to return only the rows which satisfy the filter conditions.
- `USERELATIONSHIP (ColumnName1 = Foreign key, ColumnName2 = Primary key)` activates an inactive relationship between two tables. This function should only be used in functions that accept a filter parameter (`CALCULATE`, `TOTALYTD`).
- `CROSSFILTER(LeftColumn= "Many" side, RightColumn= "One" side, CrossFilterType)` works similarly as `USERELATIONSHIP`, but through another table (modify the filter flow). Instead of turning on bidirectional relationships, we should use `CROSSFILTER` for specific cases
- `TREATAS` allows us to create virtual, summarized versions of our tables to match the granularity that we need to form a valid relationship.
  + Notice that we should only use `TREATAS` when `USERELATIONSHIP` can't be used [-> Read this](https://www.mssqltips.com/sqlservertip/5482/how-to-use-the-treatas-function-in-dax)
  
## 9. Iterrator Functions
- Iterator Cardinality is the number of rows in the tables being iterated; the more unique rows, the higher the cardinality
  + Physical relationships: cardinality is defined as the max number of unique rows in the largest table
  + Virtual relationships: cardinality is defined as the number of unique rows in each table multipplied together
- `CONCATENATEX` works similarly as `GROUP_CONCAT` in MySQL or `STRING_AGG` in SQL Server
- `AVERAGE` & `AVERAGEX` do NOT count days with zero sales when computing an average (ignore blank cells). To evaluate an average over a date range that includes dates with no sales, use `DIVIDE` & `COUNTROWS` instead
- Moving Average using Variables, `FILTERS`, & `AVERAGEX` [-> Read this](https://medium.com/analytics-vidhya/moving-average-using-dax-power-bi-413a31099091)
- Moving Average using `DATESINPERIOD` & `AVERAGEX` [-> Read this](https://www.sqlbi.com/articles/rolling-12-months-average-in-dax)
- `DATESINPERIOD` works similarly like `DATEDIFF` [-> Read this](https://dax.guide/datesinperiod/)

## 10. Advanced Time Intelligence
- `CALENDAR` works similarly as `GENERATESERIES`, but for `DATETIME` datatype
- `CALENDARAUTO(Fiscal_Year_End_Month)`. `CALENDARAUTO` without parameters will automatically start at Jan 01
