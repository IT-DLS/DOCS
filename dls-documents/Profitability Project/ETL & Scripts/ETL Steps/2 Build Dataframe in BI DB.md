This Step works on building a data frame for reporting on and creating data within without contaminating the base data. 

The SQL script first deletes all records from the `df_sales_profitability` table to ensure it is empty. Then, it inserts new data into the table using two combined `INSERT INTO ... SELECT` statements with a `UNION ALL`. This Union All is used to combine both the customer discounts with the customer profitability metrics and coalesce all nulls to 0 for math, ytd purposes.

The first part of the insertion pulls data from the `bi_cust_discount` table, setting specific column values:

- The `part` column is set to `'discpart'` since no parts are tracked via the original data.
- The `location` column is set to `'01'` since location is tracked on these discounts.
- The `revenue` column uses `COALESCE` to replace any `NULL` values with `0`.
- The `std_cost`, `mfg_var`, and `p18adj` columns are set to `0` so they union nicer and play nice with the format of the data.
- The query filters records to include only those where `date_record` is from the previous year onwards and where the `customer` field is not empty.

The second part of the insertion pulls data from the `bi_sales_profitability` table, including:

- All columns (`customer`, `part`, `product_line`, `location`, `date_record`, `revenue`, `std_cost`, `mfg_var`, and `p18adj`).
- Using `COALESCE` to replace any `NULL` values in `revenue`, `std_cost`, `mfg_var`, and `p18adj` with `0`.
- The query similarly filters records to include only those where `date_record` is from the previous year onwards and where the `customer` field is not empty.

This Query is executed via python script within the main etl using the pyscopg2 library.

Followed by the execution of the [[3 Build Dummy Data]] process.

The SQL Code used:
# DF Query

```
DELETE FROM df_sales_profitability;

  

    INSERT INTO df_sales_profitability (customer, part, product_line, location, date_record, revenue, std_cost, mfg_var, p18adj)

    SELECT

        b.customer,

        'discpart' as part,

        b.product_line,

        '01' as location,

        b.date_record,

        COALESCE(b.revenue, 0) AS revenue,

        0 AS std_cost,

        0 AS mfg_var,

        0 AS p18adj

    FROM

        public.bi_cust_discount as b

    WHERE

        date_record >= (DATE(CONCAT((EXTRACT(YEAR FROM NOW()) - 1), '-01-01')))

        AND customer <> ''

  

    UNION ALL

  

    SELECT

        customer,

        part,

        product_line,

        location,

        date_record,

        COALESCE(revenue, 0) AS revenue,

        COALESCE(std_cost, 0) AS std_cost,

        COALESCE(mfg_var, 0) AS mfg_var,

        COALESCE(p18adj, 0) AS p18adj

    FROM

        public.bi_sales_profitability

    WHERE

        date_record >= (DATE(CONCAT((EXTRACT(YEAR FROM NOW()) - 1), '-01-01')))

        AND customer <> '';

    """
```