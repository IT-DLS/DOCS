Nick Sarkesian's team needs help to audit and track all orders that are printed and having materials ordered/assigned to them. Previously there was an issue where the following ocurred:

>Job was printed and processed to be sent to Procurement team for them to order/assign raw Materials to the Job.

>Sarah Haakenson never received the work order due to it being accidentally refiled in the Job jacked room.

>weeks later when customer asked for the order the error was discovered and no one had known the order was missing/never started.

Currently the manual fix to detect these errors is the following.

* Daily report with all orders Created with a -1 Day Lag is generated
* Sarah/Dave Scans the work orders they have sent to their desk into the work order file they have on the G: drive
* Comparisons are made to compared against Orders that Printed from the -1 Day lag report for all opened orders in Itasca versus what exists in either respective excel file


My Thoughts:

The process appears to be a okay but has some potential pitfalls and data loss issues.

1. -1 Day lag will create an issue where a scanned order would have no reference point and potentially create a false positive on a missing order and also would not tie out to a missing order report from a prior day since the validation data will no longer hold the order.
2.  Potential Issue with initial problem of REFILING of the printed Job jacket not being addressed in this reporting process, A scan in section should be added as well to the Job printer to identify any process/lag issues ?
3. Potentially a very time consuming process with clerical/scanning errors to create more noise or false positives in the process.


Solutions

1.) An internal site made by IT to scan in all jobs at each station as the come in that communicates live with a view in GSS based on certain Criteria and creates a rolling automatic exception report.
	> could be using the BI database or compare to the live data source if Needed.  
	> excel vb option, last resort ripcord, should avoid AT ALL COSTS
	
2.) Identify key fields within GSS to identify NON Moving cut Job orders in Itasca and generate a smart rolling exception report.
	> Likely filtering ALL Open work orders at Itasca , with quantity attached and not flagged as closed
	> Find tables that show items are procured against a work order or assigned to a work order
	> create a sliding time criteria to identify non moving work orders to generate for the exception report.
	> if a work order has been "printed" is FLAG_WO_PRTD in the JOB_HEADER table. I put printed in quotes because a user may hit the print button to view the work order printout on their computer, but might not actually print it.


