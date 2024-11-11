The profitability project is an initiative to be able for the DLS business to log, visualize, and make actionable data upon the profitability of its products across various dimensions.  

To achieve this the core data set for the profitability of products is pulled, transformed and stored into the [[_Business Intelligence Database Summary]] at the following level of granularity:

* Customer
* Part
* Product Line
* Location
* Day of Record 

We track the following metrics according by the aforementioned levels aggregated to the day. For example we would sum all revenue values for each unique intersection to the day.
(E.G. "AAI929"	"DT-400600-5-01"	"CC"	"05"	"2024-02-29" has 5 sales, the metric for revenue would be the aggregate of those 5 sales )
This is done to GREATLY reduce the amount of data stored.

The following metrics are tracked in the profitability project:

* Revenue
* Standard Cost
* Manufacturing Variance
* P18 Adjustments
* Standard Margin
* Customer Margin
* Standard Margin % (Calculated)
* Customer Margin % (Calculated)

Currently the data for all projects is aggregated via the {ETL_Profitability_Project}.

The dashboards, reports, and other ad hoc requests relating to this project are listed below:
* [[Cust by PL-Profitability(Monthly)]]



# New PROCESS
Automated process kicked off by the [[Main_Profitability ETL]]

# OLD PROCESS

**Profitability by Customer and Product Line**

1)     Open the last customer profitability report sent by Accounting. This will serve as the foundation for the report this month.

2)     Enable editing, unhide all columns, and then select all data and remove all subtotals.

![Graphical user interface, application, table, Excel
Description automatically generated](file:///C:/Users/lroe/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

3)     Remove the columns for the last month’s 2021 data. For example, if completing the report for the month of May, you would remove the columns containing data for April 2021.

4)     Add and format new columns to display 2021 sales for the month you are currently working on. In this example, you would add them for May 2021.

5)     In DLS, go to **Sales Analysis > File > Profitability by Customer and Product Line** and select the date range for the month you are pulling from.

6)     Export to Excel.

7)     In PPL, go to **Sales Analysis > Reports > Sales by Customer and Product Line**. Use the first and last day of the month as your parameters.

8)     Export the Crystal report as a Microsoft Excel Workbook Data-Only file.

9)     Copy and paste values only into the customer profitability file that was exported earlier.

  

10) Format your discounts as follows:

|   |   |   |   |   |
|---|---|---|---|---|
|**Customer**|**Customer Name**|**PL**|**Revenue**|**STD Cost**|
|AMEBUS|American Solutions For Busines|DISCOUNT|-1879.94|0|
|MPSS|Moore Wallace North America|DISCOUNT|-1827.28|0|
|PIEDNA|Piedmont National Corporation|DISCOUNT|-1646.61|0|
|WERNER|Werner Co|DISCOUNT|-1203.94|0|
|WORNAT|NATIONAL WORKFLOW|DISCOUNT|-1015.67|0|
|KENTLA|ORORA Business Service Cntr|DISCOUNT|-517.47|0|
|WEBBMA|Webb Mason Enterprise Print Mn|DISCOUNT|-404.04|0|
|C.P.S.|Consolidated Printing Supplies|DISCOUNT|-371.96|0|
|INTERM|Honeywell|DISCOUNT|-237.56|0|
|NUMERI|Numeridex, Inc.|DISCOUNT|-120.45|0|
|INNNAT|NATIONAL INNERWORKINGS|DISCOUNT|-61.19|0|
|TAYFRD|Taylor Print Impressions-FRD|DISCOUNT|-58.11|0|
|CROWN|Crown Packaging Corporation|DISCOUNT|-15.92|0|
|CANTEX|Cantex, Inc.|DISCOUNT|-22.53|0|
|RJRTCO|R. J. Reynolds Tobacco Company|DISCOUNT|-58645.00|0|

Use the **DLS and PPL Customer List** file in order to pull in the customer name.

11) Copy and paste the discount information into the customer profitability file that was exported earlier.

12) Add a new column to the left of the customer field and concatenate using the customer and PL fields.

13) Cut and paste the ICPPL records in a new sheet in the event they are needed later. Be sure to delete the blank lines from the original sheet.

14) Add a new column to the right of the cust margin field and perform a vlookup to determine customers that are new to the YTD customer profitability file. An example can be seen below:

=VLOOKUP(A2,'[September 2020 Customer Profitability with notes.xlsx]September 2020'!$A:$A,1,FALSE)

15) Filter the new column to show records with #N/A.

16) Copy your data in column A through column D (use alt + semicolon to copy only the information that is filtered) and insert at bottom of customer profitability report, starting in column A.

17) For your new records, add in zeros for the previous months in the revenue, standard cost, manufacturing variance, and P18 adjustment columns.

18) Now, sort records (starting in row 4) by column B (A to Z).

19) Perform a vlookup to pull the revenue, standard cost, manufacturing variance, and P18 adjustments in for the new month.

=VLOOKUP(A4,'[CustomerProfitability_2020-10-01_2020-10-31.xlsx]Sheet'!$A:$H,5,FALSE)

=VLOOKUP(A4,'[CustomerProfitability_2020-10-01_2020-10-31.xlsx]Sheet'!$A:$H,6,FALSE)

=VLOOKUP(A4,'[CustomerProfitability_2020-10-01_2020-10-31.xlsx]Sheet'!$A:$H,7,FALSE)

=VLOOKUP(A4,'[CustomerProfitability_2020-10-01_2020-10-31.xlsx]Sheet'!$A:$H,8,FALSE)

20) Copy and paste values only to get rid of the formulas for the new month.

21) Replace #N/A with 0 for the new month.

22) Perform a vlookup to pull the revenue, standard cost, manufacturing variance, and P18 adjustments in for the previous YTD columns. In 2022, use the following file: **December 2021 Customer Profitability**

Revenue: column 113

Standard Cost: column 114

Manufacturing Variance:  column 115

P18 Adjustments: column 116

23) Copy and paste values only to get rid of the formulas.

24) Replace #N/A with 0.

25) Update and fill formulas in current YTD columns to account for new customer records.

26) Copy and paste values only to get rid of the formulas.

27) Replace #N/A with 0.

28) Perform a vlookup to pull the revenue, standard cost, manufacturing variance, and P18 adjustments in for the previous year’s month. In 2022, use the following file: **December 2021 Customer Profitability**.

**January**

Revenue: column E = 5

Standard Cost: column F = 6

Manufacturing Variance: column G = 7

P18 Adjustments: column H = 8

**February**

Revenue: column N = 14

Standard Cost: column O = 15

Manufacturing Variance: column P = 16

P18 Adjustments: column Q = 17

**March**

Revenue: column W = 23

Standard Cost: column X = 24

Manufacturing Variance: column Y = 25

P18 Adjustments: column Z = 26

**April**

Revenue: column AF = 32

Standard Cost: column AG = 33

Manufacturing Variance: column AH = 34

P18 Adjustments: column AI = 35

**May**

Revenue: column AO = 41

Standard Cost: column AP = 42

Manufacturing Variance: column AQ = 43

P18 Adjustments: column AR = 44

**June**

Revenue: column AX = 50

Standard Cost: column AY = 51

Manufacturing Variance: column AZ = 52

P18 Adjustments: column BA = 53

**July**

Revenue: column BG = 59

Standard Cost: column BH = 60

Manufacturing Variance: column BI = 61

P18 Adjustments: column BJ = 62

**August**

Revenue: column BP = 68

Standard Cost: column BQ = 69

Manufacturing Variance: column BR = 70

P18 Adjustments: column BS = 71

**September**

Revenue: column BY = 77

Standard Cost: column BZ = 78

Manufacturing Variance: column CA = 79

P18 Adjustments: column CB = 80

**October**

Revenue: column CH = 86

Standard Cost: column CI = 87

Manufacturing Variance: column CJ = 88

P18 Adjustments: column CK = 89

**November**

Revenue: column CQ = 95

Standard Cost: column CR = 96

Manufacturing Variance: column CS = 97

P18 Adjustments: column CT = 98

**December**

Revenue: column CZ = 104

Standard Cost: column DA = 105

Manufacturing Variance: column DB = 106

P18 Adjustments: column DC = 107

29) Add subtotals for revenue, standard cost, manufacturing variance, and P18 adjustments in each month. Make sure to calculate at each change in customer.

30) Fill in standard margin, standard margin %, custom margin, and custom margin % formulas for each month, YTD, and YTD 2021.

31) Insert a blank row above grand total row.

32) Highlight all customer total rows using Blue, Accent 5, Lighter 80%.

(note: use alt+semicolon to avoid highlighting the individual cells).

33) Highlight grand total row using Blue, Accent 1, Lighter 40%.

34) Sort totals by YTD revenue, high to low.

35) Group standard cost to standard margin % for each month and minimize.

36) Hide previous months and future months.

37) Rename worksheet and file as needed.

38) Send email with new file.

  

**OEM Customer Profitability Walkthrough**

1)     Once the normal customer profitability report has been finalized, open the file. This will serve as the foundation for the report this month.

2)     In DLS, go to **Accounts Receivable > Reports > OEM Customer List**.

3)     Export the Crystal report as a Microsoft Excel Workbook Data-Only file.

4)     Copy the list of customers and concatenate customer with _Total_. You should end up with two records for each OEM customer:

![Graphical user interface, application, table
Description automatically generated](file:///C:/Users/lroe/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

5)     In the customer profitability report, perform a vlookup to determine which customers do not appear in the OEM customer list.

![Application, table, Excel
Description automatically generated](file:///C:/Users/lroe/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

6)     Filter to show cells containing #N/A and then delete those sheet rows.

7)     Remove the WH line for customer BARINC as well

8)     Save and email file as needed.

