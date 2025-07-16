# Financial Analysis using Power BI & DAX
This project focuses on analyzing credit card usage and financial metrics for a banking institution using Power BI and DAX queries. The goal is to provide key insights into customer behavior, credit utilization, and financial performance by applying advanced DAX functions.

## Project Overview
The analysis covers various aspects of financial data, including:

* Running totals of credit card transactions.
* 4-week moving averages of credit limits for each client.
* Month-over-month and week-over-week growth in transaction amounts.
* Customer Acquisition Cost (CAC) as a ratio of transaction amounts.
* Yearly average of client utilization ratios.
* Top 5 clients by total transaction amount.
* Credit risk scores based on multiple financial metrics.
* Churn indicators for inactive clients.

''' Total_of_credit_Transactions =
CALCULATE(
   SUM(credit_card[Total_trans_Amt]),
   FILTER(
      ALL(credit_card),
      credit_card[Week_Start_Date]<= MAX(credit_card[Week_Start_Date])
   )
)''' 
