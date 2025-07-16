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

## Problem Statements and DAX Queries
### 1. Running Total of Credit Card Transactions
### DAX Query:
```
Total_of_credit_Transactions =
CALCULATE(
   SUM(credit_card[Total_trans_Amt]),
   FILTER(
      ALL(credit_card),
      credit_card[Week_Start_Date]<= MAX(credit_card[Week_Start_Date])
   )
) 
```
### 2. 4-Week Moving Average of the Credit Limit for Each Client
```
MOVING_AVG =
VAR TimePeriod = DATEINPERIOD(
  credit_card[Week_Start_date]},
  Max(Credit_card[Week_Start_Date]),
  -28,
   DAY
)
VAR TotalCreditLimit = CALCULATE(
   SUM(credit_card[Credit_Limit]),
   TimePeriod
)
VAR DistinctWeeks = CALCULATE(
    DISTINCTCOUNT(Calendar[Week_num]),
    TimePeriod
)
RETURN DIVIDE(TotalCreditLimit, DistinctWeeks)
```
### 3. Month-on-Month (MoM%) and Week-on-Week (WoW%) Growth on Transaction Amount
```
MOM_GROWTH =
VAR CurrentMonth = SUM(Credit_card[Total_Trans_Amt])
VAR PreviousMonth = CALCULATE(
    SUM(credit_card[Total_Trans_Amt]),
    DATEADD(Calendar[Date], -1 , MONTHS)
)
RETURN 
DIVIDE(CurrentMonth - PreviousMonth, PreviousMonth)


WOW_GROWTH =
VAR CurrentWeek = SUM(Credit_card[Total_Trans_Amt])
VAR PreviousWeek = CALCULATE(
    SUM(credit_card[Total_Trans_Amt]),
    DATEADD(Calendar[Date], -7 , DAY)
)
RETURN 
DIVIDE(CurrentWeek - PreviousWeek, PreviousWeek, 0)
```
### 4. Customer Acquisition Cost (CAC) as a Ratio of Transaction Amount
```
CAC =
DIVIDE(
    SUM(credit_card[Customer_Acq_Cost]),
    SUM(credit_card[Total_Trans_Amt])
    0
)
```
### 5. Yearly Avergae of Avg_Utlization_Ratio for All Clients
```
AVG_utilization_rate =
AVERAGE(credit_card[Avg_Utilization_Ratio])
```
### 6.Percentage of Interest Earned Compared to Total Revolving Balance for Each Client
```
Interest_by_revbal =
DIVIDE(
     SUM(credit_card[Interest_Earned]),
     SUM(credit_card[Total_Revolving_Bal]),
     0
)
```
### 7.Top 5 Clients by Total Transaction Amount
```
TOP5_CLIENT =
TOPN(
    5,
    SUMMARIZE(
        credit_card,
        credit_card[(Client_Num],
        "TOTAL_AMOUNT", SUM(credit_card[Total_Trans_Amt])
),
[TOTAL_AMOUNT],
DESC
```
### 8.Clients Whose Avg_Utilization_Ratio Exceeds 80%
```
High_Utilixation_Client
credit_card[Avg_Utilization_Ratio] > 0.8
```
#### 8.Income vs Credit Limit Correlation(Created using Quick Measure)
```
Income and Credit_Limit correlation for Client_Num = 
  VAR __CORRELATION_TABLE = VALUES('customer'[Client_Num])
  VAR __COUNT =
  	COUNTX(
  		KEEPFILTERS(__CORRELATION_TABLE),
  		CALCULATE(SUM('customer'[Income]) * SUM('credit_card'[Credit_Limit]))
  	)
  VAR __SUM_X =
  	SUMX(
  		KEEPFILTERS(__CORRELATION_TABLE),
  		CALCULATE(SUM('customer'[Income]))
  	)
  VAR __SUM_Y =
  	SUMX(
  		KEEPFILTERS(__CORRELATION_TABLE),
  		CALCULATE(SUM('credit_card'[Credit_Limit]))
  	)
  VAR __SUM_XY =
  	SUMX(
  		KEEPFILTERS(__CORRELATION_TABLE),
  		CALCULATE(SUM('customer'[Income]) * SUM('credit_card'[Credit_Limit]) * 1.)
  	)
  VAR __SUM_X2 =
  	SUMX(
  		KEEPFILTERS(__CORRELATION_TABLE),
  		CALCULATE(SUM('customer'[Income]) ^ 2)
  	)
  VAR __SUM_Y2 =
  	SUMX(
  		KEEPFILTERS(__CORRELATION_TABLE),
  		CALCULATE(SUM('credit_card'[Credit_Limit]) ^ 2)
  	)
  RETURN
  	DIVIDE(
  		__COUNT * __SUM_XY - __SUM_X * __SUM_Y * 1.,
  		SQRT(
  			(__COUNT * __SUM_X2 - __SUM_X ^ 2)
  				* (__COUNT * __SUM_Y2 - __SUM_Y ^ 2)
  		)
  	)
```
### 9. Average Customer Satisfaction Score by Credit Card Category
```
Avg_score_by_card_category =
SUMMARIZE(
      credit_card,
      credit_card[Card_Category],
      "AVG_SCORE", ROUND(AVERAGE(customer[Cust_Satisfaction_Score]),2))
```
### 10.Loan Approval vs Credit Limit (Analyzing How Credit Limit Affects Loan Approval)
```
Loan_yes =
CALCULATE(
      AVERAGE(credit_card[Credit_Limit]),
      customer[Personal_loan] = "yes"
)

Loan_no =
CALCULATE(
      AVERAGE(credit_card[Credit_Limit]),
      customer[Personal_loan] = "no")
```
### 11.Customer Churn Indicator (Clients with No Transactions in Last 6 Months)
```
no_trans_in_last_6_mon = 
VAR Last6Months =
     CALCULATE(
        SUM(credit_card[Total_Trans_Amt]), 
        DATESINPERIOD('Calendar'[Date],MAX('Calendar'[Date]), -6, MONTH)
)
RETURN
     IF(ISBLANK(Last6Months), || Last6Months = 0, TRUE, FALSE))
```
### 12.Delinquency Rate (Percentage of Clients with Delinquent Accounts)
```
Delinquency_rate =
VAR DelinuentAcc = 
         CALCULATE(
              COUNTROWS(credit_card),
              credit_card[Delinquent_Acc] > 0
)

VAR TotalAccounts =
         COUNTROWS(credit_card)
RETURN
         DIVIDE(DeliquentAcc, TotalAccounts,0)
```
#### 12.Credit Risk Score (Based on Avg_Utilization_Ratio, Delinquent Accounts, and Total Revolving Balance)
* To create the credit risk score, we normalize the revolving balance and then weight the average utilization ratio , delinquent accounts, and normalized revolving balance

-- Normalized_revolving__balance
```
normalized_revolving_balance
```


