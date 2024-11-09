WITH Join_Customers as
(select 
	CustomerUniqueId,
	(select c.CustomerID from Customers c where a.CustomerUniqueId=c.CustomerUniqueId) as CustomerID,
	Max(CASE WHEN DATEDIFF(day,GETDATE(),a.DateOfCaputre) =-2 THEN NGN_Cash ELSE 0 END) AS Cashbal_prev_day,
	Max(CASE WHEN DATEDIFF(day,GETDATE(),a.DateOfCaputre) =-2 THEN a.DateOfCaputre ELSE NULL END) AS prev_day,
	Max(CASE WHEN DATEDIFF(day,GETDATE(),a.DateOfCaputre) =-1 THEN a.DateOfCaputre ELSE NULL END) AS Last_Capture,
	Max(CASE WHEN DATEDIFF(day,GETDATE(),a.DateOfCaputre) =-1 THEN NGN_Cash ELSE 0 END) AS NGN_Cash
from [IInvestDB_LIVE_Extension].[dbo].[indvDailySnaps] a
group by CustomerUniqueId
)

,brr as 
(SELECT a.CustomerUniqueId, 
--prev_day,
--Yesterday, 
max(cashbal_prev_day) as cashbal_prev_day, max(NGN_Cash) as NGN_Cash,
sum(case when c.TransactionTypeId =1 and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1 
then Amount else 0 end) as Amount_purchased,
sum(case when c.TransactionTypeId =2 and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1 
then Amount else 0 end) as Amount_Deposited,
sum(case when c.TransactionTypeId =3 and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1 
then Amount else 0 end) as Amount_Liquidated,
sum(case when c.TransactionTypeId =5 and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1
then Amount else 0 end) as Amount_Withdrawn,
sum(case when c.TransactionTypeId =6 and c.DRCR ='DR' and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1 
then Amount else 0 end) as Debit,
sum(case when c.TransactionTypeId =6 and c.DRCR ='CR' and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1 
then Amount else 0 end) as Credit,
sum(case when c.TransactionTypeId =7 and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1
then Amount else 0 end) as Amount_Reversed,
sum(case when c.TransactionTypeId =8 and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1
then Amount else 0 end) as IntratransferCR,
sum(case when c.TransactionTypeId =9 and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1
then Amount else 0 end) as IntratransferDR,
sum(case when c.TransactionTypeId =10 and DATEDIFF(day,GETDATE(),c.TransactionDate) =-1
then Amount else 0 end) as ReversalDR
FROM Join_Customers a
JOIN wallets b on a.CustomerID=b.CustomerId
join WalletTransactions c on b.WalletId=c.WalletId 
group by a.CustomerUniqueId, prev_day) 


,brr2 as
(select CustomerUniqueId,  cashbal_prev_day, 
ROUND((cashbal_prev_day + Amount_Deposited + Amount_Liquidated + Credit + Amount_Reversed + IntratransferCR) - 
(Amount_Withdrawn+Amount_purchased+Debit+IntratransferDR+ReversalDR),2) as Calc_Balance,
ROUND(NGN_Cash,2) AS WalletBalance
from brr)


select * from brr2
where 
Calc_Balance <> WalletBalance


/*select * 
from Join_Customers
where CustomerUniqueId = 122790*/
