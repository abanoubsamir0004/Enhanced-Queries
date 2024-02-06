# Task3: Enhanced-Queries

Here you will find some use cases and SQL code. 
- Analyze the use cases and find more efficient code to enhance the logic.
- Include the improved SQL code for your choice, along with an explanation of why it is better.


# Example 1: Identify the salesperson with the highest sales for each quarter of the year for the past two years.

## Original Query
```
SELECT
    sp.salesperson_id,
    q.quarter_name,
    SUM(t.order_amount) AS total_sales
FROM sales_transactions AS t
JOIN salespeople AS sp ON t.salesperson_id = sp.salesperson_id
JOIN quarters AS q ON t.transaction_date BETWEEN q.start_date AND q.end_date
GROUP BY
    sp.salesperson_id,
    q.quarter_name
```

## Imporved Query
```
WITH HighestSales AS (

    SELECT
    
        t.salesperson_id,
        q.quarter_name,
        SUM(t.order_amount) AS total_sales,
        RANK() OVER (PARTITION BY q.quarter_name ORDER BY SUM(t.order_amount) DESC) AS sales_rank

    FROM sales_transactions AS t
    JOIN quarters AS q 
      ON t.transaction_date BETWEEN q.start_date AND q.end_date

    WHERE EXTRACT(YEAR FROM t.transaction_date) >= EXTRACT(YEAR FROM CURRENT_DATE) - 2

    GROUP BY
        t.salesperson_id,
        q.quarter_name
)

SELECT

    salesperson_id,
    quarter_name,
    total_sales

FROM HighestSales
WHERE sales_rank = 1
```
## Explanation

1. **Common Table Expression (CTE):**
    - Introduced a CTE named HighestSales for better organization.

2. **Window Function (RANK()):**
    - Utilized RANK() to simplify ranking logic within the CTE.

3. **Filtering for the Past Two Years:**
    - Applied a WHERE clause to focus on the relevant timeframe, potentially improving time efficiency.

4. **Elimination of Unnecessary JOIN:**
    - Removed an unnecessary JOIN with the salespeople table, as the sales_transactions table already contains the salesperson_id, potentially enhancing performance.

5. **Concise and Readable Query:**
    - Achieved a more concise and readable structure, contributing to better understanding and maintenance.
      
## $\color{blue}{ Updated \space Query \space Regarding \space Feedback}$
```
WITH HighestSales AS (

    SELECT
        extract (year from t.transaction_date) as date_year,
        t.salesperson_id,
        q.quarter_name,
        SUM(t.order_amount) AS total_sales,
        DENSE_RANK() OVER (PARTITION BY date_year,q.quarter_name ORDER BY SUM(t.order_amount) DESC) AS sales_rank

    FROM sales_transactions AS t
    JOIN quarters AS q 
      ON t.transaction_date BETWEEN q.start_date AND q.end_date

    WHERE EXTRACT(YEAR FROM t.transaction_date) >= EXTRACT(YEAR FROM CURRENT_DATE) - 2

    GROUP BY
        date_year,
        q.quarter_name,
        t.salesperson_id
)

SELECT
    date_year,
    quarter_name,
    salesperson_id,
    total_sales

FROM HighestSales
WHERE sales_rank = 1
ORDER BY date_year,quarter_name, total_sales desc
```
## $\color{blue}{ Updated \space Explanation \space Regarding \space Feedback}$

1. **Missing Date_Year Field in Rank and Grouping:**
    - The code needs to include the `Date_Year` field in the rank and grouping to ensure data retrieval for all quarters across both 2022 and 2023. This adjustment will result in processing 8 quarters in total.‎

2. **Handling Sales ID with the Highest Sales in Different Years:**
    - The original code fails to consider scenarios where the same sales ID achieves the highest sales in both 2022 and 2023. To address this, the correction involves calculating the sum for each specific quarter within each year, rather than aggregating across years.

3. **Ineffectiveness of Rank or Dense Rank for First Rank:**
    - The use of `rank` or `dense rank` window functions in the code does not produce the intended effect, as only the first rank is required. Modifications are necessary to ensure that only the first rank is considered in the analysis.

# Example 2: Identifying the top 5 salespeople based on total sales for the year:
```
SELECT
    sp.salesperson_id,
    SUM(t.order_amount) AS total_sales
FROM sales_transactions AS t
JOIN salespeople AS sp ON t.salesperson_id = sp.salesperson_id
GROUP BY
    sp.salesperson_id
ORDER BY total_sales DESC
LIMIT 5
```

## Imporved Query

```
SELECT
    salesperson_id,
    SUM(order_amount) AS total_sales

FROM sales_transactions
WHERE EXTRACT(YEAR FROM transaction_date) = EXTRACT(YEAR FROM CURRENT_DATE) - 1
GROUP BY
  salesperson_id
ORDER BY
  total_sales DESC
LIMIT 5
```

## Explanation

1. **Removed Unnecessary JOIN:**
    - Eliminated the JOIN with salespeople since only salesperson_id is needed, ‎simplifying the query.‎

2. **Direct Filtering for Previous Year:**
    - Introduced a WHERE clause to filter transactions for the previous year. I assume we are still in the first month of the year 2024, thereby providing focused data.

3. **Simplified FROM Clause:**
    - Directly referenced sales_transactions without aliases for enhanced readability.‎

## $\color{blue}{ Updated \space Query \space Regarding \space Feedback}$
```
SELECT
    salesperson_id,
    SUM(order_amount) AS total_sales

FROM sales_transactions
WHERE EXTRACT(YEAR FROM transaction_date) = EXTRACT(YEAR FROM CURRENT_DATE) 
GROUP BY
  salesperson_id
ORDER BY
  total_sales DESC
LIMIT 5
```

## $\color{blue}{ Updated \space Explanation \space Regarding \space Feedback}$

1. **I have adjusted the Direct Filtering for the Previous Year, where I assumed that we are now in the first month of 2024. Thus, I am altering this assumption, considering that the data already contains records for the year 2024, in order to obtain the top salespeople for the current year.**



# Example 3: Return a list of all the employees in those departments.

```
SELECT 
  first_name,
  last_name
FROM employee e1
WHERE department_id IN (
   SELECT department_id
   FROM department
   WHERE manager_name=‘John Smith’)
```

## Imporved Query

```
SELECT 
  e.first_name,
  e.last_name

FROM employee e
INNER JOIN department d
  ON e.department_id = d.department_id

WHERE d.manager_name = 'John Smith'
```

## Explanation

1. **Use of EXISTS:**
   - The suggested improvement employs the EXISTS keyword to verify the existence of rows in the department table, eliminating reliance on the IN clause.

2. **Avoiding Subquery for Columns:**
   - The modification incorporates an INNER JOIN to establish a direct connection between the employee table (e) and the department table (d), thereby eliminating the necessity for a subquery.

3. **Explicit JOIN Syntax:**
   - Explicit JOIN syntax is adopted for enhanced clarity, facilitating a better understanding of the relationship between tables.

4. **Descriptive Alias Names:**
   - Descriptive aliases (e for employee, d for department) are utilized to enhance the query's readability and maintainability.

# Example 4: Find Duplicate Rows
```
SELECT 
  employee_id,
  last_name,
  first_name,
  dept_id,
  manager_id,
  salary
FROM employee
GROUP BY   
  employee_id,
  last_name,
  first_name,
  dept_id,
  manager_id,
  salary
HAVING COUNT(*) > 1
```

## Imporved Query 

No improved query is provided, as the original query is more effective in identifying duplicates.

# Example 5: Compute a Year-Over-Year Difference
-  The main query should return the year, year_amount, revenue_previous_year, Year-Over-Year_diff_value, and Year-Over-Year_diff_perc(%) from the year_metrics
```
SELECT
    extract(year from day) as year,
    SUM(daily_amount) as year_amount
  FROM sales
  GROUP BY year
```

## Imporved Query

```
WITH
year_metrics AS (
  SELECT

    EXTRACT(YEAR FROM day) AS year,
    SUM(daily_amount) AS year_amount

  FROM sales
  GROUP BY year
),
Final as (
  SELECT

    ym.year,
    ym.year_amount,
    COALESCE(LAG(ym.year_amount) OVER (ORDER BY ym.year), 0) AS revenue_previous_year,
    ym.year_amount - COALESCE(LAG(ym.year_amount) OVER (ORDER BY ym.year), 0) AS year_over_year_diff_value,

    CASE
      WHEN COALESCE(LAG(ym.year_amount) OVER (ORDER BY ym.year), 0) = 0
      THEN NULL
      ELSE (ym.year_amount - COALESCE(LAG(ym.year_amount) OVER (ORDER BY ym.year), 0))
            / COALESCE(LAG(ym.year_amount) OVER (ORDER BY ym.year), 0) * 100
    END AS year_over_year_diff_perc
  
  FROM year_metrics ym
)
SELECT *
FROM FINAL
```
## Explanation

1. **Common Table Expression (CTE):**
   - A Common Table Expression (CTE) named year_metrics is employed for the purpose of calculating yearly metrics, encompassing the determination of the total amount for each respective year.

2. **Window Functions (LAG):**
   - The LAG() window function is utilized to retrieve the revenue_previous_year, which signifies the year_amount from the preceding year.

3. **Calculations:**
   - The year_over_year_diff_value is computed as the disparity between the current year's amount and the revenue_previous_year.

4. **Percentage Calculation:**
   - The year_over_year_diff_perc is calculated to represent the percentage change. A NULL value is assigned in cases where the previous year's amount is zero, mitigating the risk of division by zero errors.
     
## $\color{blue}{ Updated \space Imporved \space Query \space Regarding \space Feedback}$
```
WITH year_metrics AS (
  SELECT

    EXTRACT(YEAR FROM day) AS year,
    SUM(daily_amount) AS year_amount

  FROM sales
  GROUP BY year
),
Final AS (
  SELECT

    year,
    year_amount,
    COALESCE(LAG(year_amount) OVER (ORDER BY year), 0) AS revenue_previous_year,
    year_amount - COALESCE(LAG(year_amount) OVER (ORDER BY year), 0) AS year_over_year_diff_value,

    CASE
      WHEN COALESCE(LAG(year_amount) OVER (ORDER BY year), 0) = 0
      THEN NULL
      ELSE (year_amount - COALESCE(LAG(year_amount) OVER (ORDER BY year), 0)) /
           COALESCE(LAG(year_amount) OVER (ORDER BY year), 0) * 100
    END AS year_over_year_diff_perc

  FROM year_metrics
)
SELECT *
FROM Final;

```
## $\color{blue}{ Updated \space Explanation \space Regarding \space Feedback}$

1. **Removed unnecessary alias `ym` from the SELECT clause in the `Final` CTE.**
2. **Replaced `ym.year` with `year` in the `Final` CTE to improve readability.**
3. **Consolidated references to `year_metrics` in the `Final` CTE for clarity.**

# Example 6: Return the MAX sales and 2nd max sales
```
SELECT
(SELECT MAX(sales) FROM Sales_orders) max_sales,
(SELECT MAX(sales) FROM Sales_orders
WHERE sales NOT IN (SELECT MAX(sales) FROM Sales_orders )) as 2ND_max_sales;
```
## Explanation

1. **Common Table Expression (CTE):**
   - The Common Table Expression (CTE) named RankedSales is effectively utilized to assign a row number to each sales value, employing a descending order as the basis for ranking.

2. **ROW_NUMBER() Function:**
   - The ROW_NUMBER() window function is employed to streamline the ranking process of sales values, eliminating the need for subqueries and enhancing overall query efficiency.

3. **Conditional Aggregation:**
   - Conditional aggregation is implemented to derive the second maximum sales value by applying filters based on the assigned sales rank within the RankedSales CTE.

4. **WHERE Clause:**
   - The WHERE clause is strategically employed to restrict the consideration to only the top two ranked sales values. This optimization enhances performance by reducing unnecessary calculations and focusing on the relevant data subset.


## Imporved Query

```
WITH RankedSales AS (
  SELECT
    sales,
    ROW_NUMBER() OVER (ORDER BY sales DESC) AS sales_rank
  FROM Sales_orders
)

SELECT
  MAX(sales) AS max_sales,
  COALESCE(MAX(CASE WHEN sales_rank = 2 THEN sales END), 0) AS second_max_sales
FROM RankedSales
WHERE sales_rank <= 2

```
## $\color{blue}{ Updated \space Imporved \space Query \space Regarding \space Feedback}$
```
WITH RankedSales AS (
  SELECT
    sales,
    DENSE_RANK() OVER (ORDER BY sales DESC) AS sales_rank
  FROM Sales_orders
)

SELECT
  MAX(sales) AS max_sales,
  COALESCE(MAX(CASE WHEN sales_rank = 2 THEN sales END), 0) AS second_max_sales
FROM RankedSales
WHERE sales_rank <= 2;
```
## $\color{blue}{ Updated \space Explanation \space Regarding \space Feedback}$

**DENSE_NUMBER() Function:**
  - Utilize the DENSE_RANK() window function instead of ROW_NUMBER() to enable the second_max_sales to account for multiple sales with the same amount when they share the rank of 2.


# Example 7: 
- This query will select the Essns of all employees who work the same
(project, hours) combination on some project that employee ‘John
Smith’ (whose Ssn =‘123456789’) works on. I

```
SELECT DISTINCT Pnumber
FROM PROJECT
WHERE Pnumber IN  
( SELECT Pnumber
FROM PROJECT, DEPARTMENT, EMPLOYEE
WHERE Dnum=Dnumber AND
Mgr_ssn=Ssn AND Lname=‘Smith’ )
OR
Pnumber IN
( SELECT Pno
FROM WORKS_ON, EMPLOYEE
WHERE Essn=Ssn AND Lname=‘Smith’ )
```

## Imporved Query 1

```
WITH JohnSmithProjects AS (
  SELECT DISTINCT Pnumber
  FROM PROJECT P
  JOIN DEPARTMENT D
        ON P.Dnum = D.Dnumber
  JOIN EMPLOYEE E
        ON D.Mgr_ssn = E.Ssn
        AND E.Lname = 'Smith'
  UNION

  SELECT DISTINCT Pno
  FROM WORKS_ON WO
  JOIN EMPLOYEE E
        ON WO.Essn = E.Ssn AND E.Lname = 'Smith'
)

SELECT DISTINCT Pnumber
FROM PROJECT
WHERE Pnumber IN (SELECT Pnumber FROM JohnSmithProjects)


```

## Explanation 1

1. **Common Table Expression (CTE):**
   - A Common Table Expression (CTE) named JohnSmithProjects is established to retrieve project numbers where employees engage in the same (project, hours) combination as 'John Smith'.

2. **JOIN Clauses:**
   - JOIN clauses are employed to establish connections between pertinent tables (PROJECT, DEPARTMENT, EMPLOYEE, WORKS_ON). This approach enhances the query's structure and readability, promoting a more organized presentation of information.

3. **UNION Operator:**
   - The UNION operator is utilized to amalgamate project numbers obtained from the two subqueries. This operation guarantees distinct values, ensuring a comprehensive and unique list of project numbers.

4. **Simplified WHERE Clause:**
   - The WHERE clause in the final SELECT statement is deliberately simplified to filter project numbers based on the JohnSmithProjects CTE. This simplification enhances the clarity of the query and focuses on the essential filtering criterion.

## $\color{blue}{ Updated \space Imporved \space Query 1 \space Regarding \space Feedback}$
```
WITH JohnSmithProjects AS (
    SELECT DISTINCT P.Pnumber
    FROM PROJECT P
    JOIN DEPARTMENT D ON P.Dnum = D.Dnumber
    JOIN EMPLOYEE E ON D.Mgr_ssn = E.Ssn AND E.Lname = 'Smith'

    UNION

    SELECT DISTINCT WO.Pno AS Pnumber
    FROM WORKS_ON WO
    JOIN EMPLOYEE E ON WO.Essn = E.Ssn AND E.Lname = 'Smith'
)

SELECT DISTINCT P.Pnumber
FROM PROJECT P
JOIN JohnSmithProjects JSP ON P.Pnumber = JSP.Pnumber
```
## $\color{blue}{ Updated \space Explanation 1 \space Regarding \space Feedback}$
1. **Alias Consistency:**
   - Ensured clarity through consistent and meaningful aliases like "P" for PROJECT, "D" for DEPARTMENT, and "E" for EMPLOYEE.

2. **CTE Streamlining:**
   - Refactored CTE "JohnSmithProjects" for conciseness, directly selecting P.Pnumber from PROJECT in the first union.

3. **JOIN Optimization:**
   - Optimized performance by replacing the IN clause with a JOIN operation in the final SELECT, potentially leveraging database optimizations.


## Imporved Query 2

```
WITH JohnHours AS
(
  SELECT 
    Pno, 
    Hours
  FROM WORKS_ON
  WHERE Essn = '123456789'
)

SELECT Essn
FROM WORKS_ON
JOIN JohnHours
  ON WORKS_ON.Pno = JohnHours.Pno 
  AND WORKS_ON.Hours = JohnHours.Hours

```

## Explanation 2

1. **Common Table Expression (CTE):**
   - A Common Table Expression (CTE) named JohnHours is employed to filter the WORKS_ON table based on the specified employee's Social Security Number (SSN) ('123456789').

2. **JOIN Clause:**
   - The primary query incorporates a JOIN clause that connects the original WORKS_ON table with the CTE (JohnHours), facilitating the matching process based on both project number (Pno) and hours (Hours).

