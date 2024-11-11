This step builds the Customer by product Line profitability view, which aggregates to the Customer Product Line level metrics:

* Revenue
* Standard Cost
* Manufacturing Variance
* P18 Adjustments
* Standard Margin
* Customer Margin

The SQL Query executed will build the `bi_CustPL_profitability_dash` by doing the following.

1. **Clear Existing Table and View**:
    
    - Drop the view `bi_CustPL_profitability_dash` if it exists.
    - Drop the table `df_cust_pl_profitability` if it exists.
2. **Create New Table**:
    
    - Create the table `df_cust_pl_profitability` using a query that aggregates data from the `df_sales_profitability` table.
    - The query includes calculating year-to-date (YTD) sums for `revenue`, `std_cost`, `mfg_var`, `p18adj`, `std_margin`, and `cust_margin`.
    - Group by `df_prof_key`, customer information, product line, and month.
3. **Create View**:
    
    - Create the view `bi_CustPL_profitability_dash` to be used as a dashboard.
    - Select and join data from the `df_cust_pl_profitability` table, including:
        - Current month data.
        - Previous year month data.
        - Current year-to-date data.
        - Previous year-to-date data.
    - Order the results by `df_prof_key` and `month_start`.


# Query to build df_cust_pl_profitability

```
-- Clear the table AND ITS VIEW

    DROP VIEW IF EXISTS bi_CustPL_profitability_dash;

    DROP TABLE IF EXISTS df_cust_pl_profitability;

    -- Create the table with the provided query

    CREATE TABLE df_cust_pl_profitability AS

    SELECT *,

        --YTD

        SUM(revenue) OVER (PARTITION BY df_prof_key, EXTRACT(YEAR FROM month_start) ORDER BY month_start ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS ytd_revenue,

        SUM(std_cost) OVER (PARTITION BY df_prof_key, EXTRACT(YEAR FROM month_start) ORDER BY month_start ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS ytd_std_cost,

        SUM(mfg_var) OVER (PARTITION BY df_prof_key, EXTRACT(YEAR FROM month_start) ORDER BY month_start ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS ytd_mfg_var,

        SUM(p18adj) OVER (PARTITION BY df_prof_key, EXTRACT(YEAR FROM month_start) ORDER BY month_start ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS ytd_p18adj,

        SUM(std_margin) OVER (PARTITION BY df_prof_key, EXTRACT(YEAR FROM month_start) ORDER BY month_start ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS ytd_std_margin,

        SUM(cust_margin) OVER (PARTITION BY df_prof_key, EXTRACT(YEAR FROM month_start) ORDER BY month_start ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS ytd_cust_margin

    FROM (

        SELECT

            CONCAT(a.customer,'|',a.product_line) AS df_prof_key,

            a.customer,

            cm.name_customer,

            CASE

                WHEN a.product_line = '' THEN 'FREIGHT'

                ELSE a.product_line

            END AS product_line,

            cs.c_sort_1 AS "customer_abc",

            sp.name AS "salesperson",

            DATE_TRUNC('Month', CAST(a.date_record AS DATE))::DATE as month_start,

            SUM(COALESCE(a.revenue,0)) AS revenue,

            SUM(COALESCE(a.std_cost,0)) AS std_cost,

            SUM(COALESCE(a.mfg_var,0)) AS mfg_var,

            SUM(COALESCE(a.p18adj,0)) AS p18adj,

            CAST(SUM(COALESCE(a.revenue, 0) - COALESCE(a.std_cost, 0)) AS NUMERIC(10, 2)) AS std_margin,

            CAST(SUM(COALESCE(a.revenue, 0) - COALESCE(a.std_cost, 0) - COALESCE(a.mfg_var, 0) - COALESCE(a.p18adj, 0)) AS NUMERIC(10, 2)) AS cust_margin

        FROM

            df_sales_profitability AS a

        LEFT OUTER JOIN v_customer_master AS cm ON a.customer = cm.customer

        LEFT OUTER JOIN v_salespersons AS sp ON cm.salesperson = sp.id

        LEFT OUTER JOIN v_customer_sales AS cs ON a.customer = cs.customer

        WHERE a.customer <> ''

        GROUP BY

            a.customer,

            cm.name_customer,

            a.product_line,

            DATE_TRUNC('Month', CAST(a.date_record AS DATE))::DATE,

            cs.c_sort_1,

            sp.name

    ) AS a;

  

    -- CREATE THE VIEW TO BE USED AS THE DAH

    Create View bi_CustPL_profitability_dash

    AS

    SELECT

        a.df_prof_key AS df_prof_key_a,

        a.customer,

        a.name_customer,

        a.product_line,  

        a.customer_abc,

        a.salesperson,

        a.month_start,

        a.revenue,

        a.std_cost,

        a.mfg_var,

        a.p18adj,

        a.std_margin,

        a.cust_margin,

        --Last Year Month Data

        b.month_start AS PRYR_MTH_month_start,

        b.revenue AS PRYR_MTH_revenue,

        b.std_cost AS PRYR_MTH_std_cost,

        b.mfg_var AS PRYR_MTH_mfg_var,

        b.p18adj AS PRYR_MTH_p18adj,

        b.std_margin AS PRYR_MTH_std_margin,

        b.cust_margin AS PRYR_MTH_cust_margin,

        --Current Year To Date Data

        a.ytd_revenue,

        a.ytd_std_cost,  

        a.ytd_mfg_var,

        a.ytd_p18adj,

        a.ytd_std_margin,

        a.ytd_cust_margin,

        --LY_Year_To_Date

        b.ytd_revenue as PRYR_MTH_ytd_revenue,

        b.ytd_std_cost as PRYR_MTH_ytd_std_cost,  

        b.ytd_mfg_var as PRYR_MTH_ytd_mfg_var,

        b.ytd_p18adj as PRYR_MTH_ytd_p18adj,

        b.ytd_std_margin as PRYR_MTH_ytd_std_margin,

        b.ytd_cust_margin as PRYR_MTH_ytd_cust_margin

    FROM

        public.df_cust_pl_profitability AS a

    LEFT JOIN

        public.df_cust_pl_profitability AS b

        ON DATE_TRUNC('month', a.month_start - INTERVAL '1 year') = b.month_start

        AND a.df_prof_key = b.df_prof_key

  

    order by a.df_prof_key, a.month_start;
```