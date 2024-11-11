This script  pulls, merges, and injects the **core data** used for within the customer profitability project. 

The goal of the script is to do the following:

![[customer_profitability_refresh_extraction.png]]



Writeup of the 

This code extracts custom SQL data from a Pervasive SQL environment, processes it into dataframes, and then stores the results in a PostgreSQL database. It connects to two different data sources, cleans the data, merges it, and uploads it to a specified table. The process includes deleting existing records for the specified month to ensure data consistency. Additionally, the code includes a function to build the SQL query dynamically based on date ranges, and it uses the `wakepy` library to keep the system awake during execution.

Next the ETL moves to the [[2 Build Dataframe in BI DB]].

This process is repeated for the discounts data. Here are the Queries to pull the data from Production.

# Query for the EXTRACT of profitability data:

```
SELECT

    CombinedData.CUSTOMER,

    CombinedData.PART,

    CombinedData.PRODUCT_LINE,

    CombinedData.LOCATION,

    CombinedData.DATE_RECORD,

    RevenueCost.Revenue,

    RevenueCost.STD_COST,

    MfgVariance.MFG_VAR,

    P18Adjustment.P18ADJ

FROM

    (

--DATAFRAME OF ALL POSSIBLE COMBINATIONS OF THE FOLLOWING FIELDS TO LATER JOIN ONTO--

SELECT

  df.CUSTOMER,

  df.PART,

  df.PRODUCT_LINE,

  df.LOCATION,

  df.DATE_RECORD

FROM

  (

      -- Query 1: Orders from V_ORDER_HIST_LINE

    SELECT

          ORDHIST.CUSTOMER,

          ORDHIST.PART,

          ORDHIST.PRODUCT_LINE,

          ORDHIST.LOCATION,

          ORDHIST.DATE_INVOICE AS "DATE_RECORD"

      FROM

          V_ORDER_HIST_LINE AS ORDHIST

      WHERE

          ORDHIST.DATE_INVOICE BETWEEN '{startD}' AND '{endD}'

          AND ORDHIST.PRODUCT_LINE <> 'WS'

      GROUP BY

          CUSTOMER,

          PART,

          PRODUCT_LINE,

          LOCATION,

          DATE_INVOICE

  

      UNION ALL

  

      -- Query 2: Inventory transactions from v_inventory_hist

    SELECT

            INVX.CUSTOMER,

            INVX.PART,

            INVH.PRODUCT_LINE,

            INVH.LOCATION,

            INVH.DATE_HISTORY AS "DATE_RECORD"

        FROM

            v_inventory_hist AS INVH

        LEFT JOIN

            V_INV_CROSS_REF AS INVX ON INVH.PART = INVX.PART AND INVH.LOCATION = INVX.LOCATION

        WHERE

            code_transaction = 'P18'

            AND DATE_HISTORY BETWEEN '{startD}' AND '{endD}'

            AND CUSTOMER <> ''

            AND COST <> '0'

            AND PRODUCT_LINE IN ('CU', 'DP', 'LS', 'VI', 'TP', 'UL', 'WH', 'RF')

        GROUP BY

            INVX.CUSTOMER,

            INVX.PART,

            INVH.PRODUCT_LINE,

            INVH.LOCATION,

            INVH.DATE_HISTORY

      UNION ALL

  

      -- Query 3: Jobs from V_JOB_DETAIL

    SELECT

        B.CUSTOMER,

        B.PART,

        B.PRODUCT_LINE,

        B.LOCATION,

        B.DATE_CLOSED AS "DATE_RECORD"

    FROM

        V_JOB_DETAIL A

    JOIN

        V_JOB_HEADER B ON A.JOB = B.JOB AND A.SUFFIX = B.SUFFIX

    WHERE

        B.DATE_CLOSED >= '{startD}' AND B.DATE_CLOSED <= '{endD}'

        AND A.SEQ = '999999'

        AND A.DESCRIPTION IN ('Std Cost Variance', 'LATE COSTS TO JOBS')

    GROUP BY

        B.CUSTOMER,

        B.PART,

        B.PRODUCT_LINE,

        B.LOCATION,

        B.DATE_CLOSED

  ) AS df

GROUP BY

  df.CUSTOMER,

  df.PART,

  df.PRODUCT_LINE,

  df.LOCATION,

  df.DATE_RECORD

    ) AS CombinedData

  

-- Left join revenue and standard cost query

LEFT JOIN (

    SELECT

        V.CUSTOMER,

        V.PART,

        V.PRODUCT_LINE,

        V.LOCATION,

        V.DATE_INVOICE,

        SUM(V.EXTENSION) AS REVENUE,

        SUM(V.COST * V.QTY_SHIPPED) AS STD_COST

    FROM

        V_ORDER_HIST_LINE V

    WHERE

        DATE_INVOICE >= '{startD}' AND DATE_INVOICE <= '{endD}'

    GROUP BY

        V.CUSTOMER,

        V.PART,

        V.PRODUCT_LINE,

        V.LOCATION,

        V.DATE_INVOICE

) AS RevenueCost ON

CombinedData.CUSTOMER = RevenueCost.CUSTOMER

AND CombinedData.PART = RevenueCost.PART

AND CombinedData.PRODUCT_LINE = RevenueCost.PRODUCT_LINE

AND CombinedData.LOCATION = RevenueCost.LOCATION

AND CombinedData.DATE_RECORD = RevenueCost.DATE_INVOICE

  
  

-- Left join manufacturing variance query

LEFT JOIN (

    SELECT

        C.CUSTOMER,

        C.PART,

        C.PRODUCT_LINE,

        C.LOCATION,

        C.DATE_CLOSED,

        SUM(C.EXPR_1) AS MFG_VAR

    FROM (

        SELECT

            A.JOB,

            A.SUFFIX,

            SUM(A.AMOUNT_LABOR * -1) AS EXPR_1,

            B.CUSTOMER,

            B.PART,

            B.PRODUCT_LINE,

            B.LOCATION,

            B.DATE_CLOSED

        FROM

            V_JOB_DETAIL A

        JOIN

            V_JOB_HEADER B ON A.JOB = B.JOB AND A.SUFFIX = B.SUFFIX

        WHERE

            B.DATE_CLOSED >= '{startD}' AND B.DATE_CLOSED <= '{endD}'

            AND A.SEQ = '999999'

            AND A.DESCRIPTION IN ('Std Cost Variance', 'LATE COSTS TO JOBS')

        GROUP BY

            A.JOB, A.SUFFIX, B.CUSTOMER, B.PART, B.PRODUCT_LINE, B.LOCATION, B.DATE_CLOSED

    ) C

    GROUP BY

        C.CUSTOMER,

        C.PART,

        C.PRODUCT_LINE,

        C.LOCATION,

        C.DATE_CLOSED

) AS MfgVariance ON

CombinedData.CUSTOMER = MfgVariance.CUSTOMER

AND CombinedData.PART = MfgVariance.PART

AND CombinedData.PRODUCT_LINE = MfgVariance.PRODUCT_LINE

AND CombinedData.LOCATION = MfgVariance.LOCATION

AND CombinedData.DATE_RECORD = MfgVariance.DATE_CLOSED

  

-- Left join P18 adjustment query

LEFT JOIN (

    SELECT

        CUSTOMER,

        A.PART,

        PRODUCT_LINE,

        A.LOCATION,

        DATE_HISTORY,

        SUM(COST * -1) AS P18ADJ

    FROM

        v_inventory_hist A

    LEFT JOIN

        V_INV_CROSS_REF B ON A.PART = B.PART AND A.LOCATION = B.LOCATION

    WHERE

        code_transaction = 'P18'

        AND DATE_HISTORY BETWEEN '{startD}' AND '{endD}'

        AND CUSTOMER <> ''

        AND COST <> '0'

        AND PRODUCT_LINE IN ('CU', 'DP', 'LS', 'VI', 'TP', 'UL', 'WH', 'RF')

    GROUP BY

        CUSTOMER,

        A.PART,

        PRODUCT_LINE,

        A.LOCATION,

        DATE_HISTORY

) AS P18Adjustment ON

CombinedData.CUSTOMER = P18Adjustment.CUSTOMER

AND CombinedData.PART = P18Adjustment.PART

AND CombinedData.PRODUCT_LINE = P18Adjustment.PRODUCT_LINE

AND CombinedData.LOCATION = P18Adjustment.LOCATION

AND CombinedData.DATE_RECORD = P18Adjustment.DATE_HISTORY;
```

# Query for the EXTRACT Discounts from DLS

```
SELECT

    cust as customer,

    'DISCOUNT',

    left(post_date,6) post_month,

    sum(amount_oe)*-1

FROM

    v_gl_ar_detail

WHERE

    GL_NUMBER in ('47200-099')

    AND post_date_sql >= '{startD}' and post_date_sql <= '{endD}'

GROUP BY

    post_month,

    cust
```

# Query for the EXTRACT Discounts from PPL

```
SELECT

    cust as customer,

    'DISCOUNT',

    left(post_date,6) post_month,

    sum(amount_oe)*-1

FROM

    v_gl_ar_detail

WHERE

    GL_NUMBER in ('47200-100','47200-200','47200-500','47200-700')

    AND post_date_sql >= '{startD}' and post_date_sql <= '{endD}'

GROUP BY

    post_month,

    cust
```