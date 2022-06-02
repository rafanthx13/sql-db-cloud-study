# Random Artigos - Interresasnts: Oraganizar depois



## 5-most-common-sql-mistakes-you-should-avoid

https://towardsdatascience.com/5-most-common-sql-mistakes-you-should-avoid-dd4eb4088f0c

1. **Don’t use Select \***: Select * outputs all the columns of a data table, it is an expensive operation and increases the execution time of the query. The ideal way is to select only the relevant columns in your subquery or the output table.

For example, imagine if we want to get the order id from the Orders table then we should select only the OrderID column instead of selecting all the columns using select *.

```
# Wrong way  
SELECT * FROM Orders

# Optimal way 
SELECT OrderID FROM Orders
```



<iframe src="https://towardsdatascience.com/media/28ae57a83256619622b06357dace2127" allowfullscreen="" frameborder="0" height="149" width="692" title="Optimize_sql.sql" class="fq aq as ag cf" scrolling="auto" style="box-sizing: inherit; height: 149px; top: 0px; left: 0px; width: 692px; position: absolute;"></iframe>

\2. **Don’t use ‘having’ instead of ‘where’**: Having clause is used to apply a filter on the aggregated columns (sum, min, max, etc.) created using the group by operation. But sometimes, programmers use the ‘having’ clause (instead of the ‘where’ clause) to filter the non aggregated columns too.

For example, in order to find the total orders executed by the employee with employee id 3, filtering using the ‘having’ clause will give unexpected results.

```
# Wrong way - having should not be used to apply filter on non-aggregated column (EmployeeID in this example)
SELECT OrderDate, count(OrderID) FROM Orders
group by OrderDate
having EmployeeID = 3


# Right Way - Where should be used to apply filter on non-aggregated column. 
SELECT OrderDate, count(OrderID) FROM Orders
where EmployeeID = 3
group by OrderDate
```



<iframe src="https://towardsdatascience.com/media/213c2fdba7dc1fb25064e9af9d09b801" allowfullscreen="" frameborder="0" height="276" width="692" title="Mistakes_having.sql" class="fq aq as ag cf" scrolling="auto" style="box-sizing: inherit; height: 276px; top: 0px; left: 0px; width: 692px; position: absolute;"></iframe>

\3. **Don’t Use ‘where’ for joining**: Some people perform the inner join using the ‘where’ clause, instead of using the “inner join’ clause. Although the performance is similar for both the syntax (the result of query optimization), it is not recommended to join using the ‘where’ clause due to the following reasons:

i. Using the ‘where’ clause for both filtering and joining **affects readability and understanding**.

ii. The **usage** of the ‘where’ clause for joining is **very limited** as it cannot perform any other join apart from the inner join.

```
# Wrong way - using where clause 
SELECT  Orders.OrderID, Orders.CustomerID, Customers.ContactName
FROM Orders, Customers
where Orders.CustomerID = Customers.CustomerID


# Right way - using inner join clause 
SELECT  Orders.OrderID, Orders.CustomerID, Customers.ContactName
FROM Orders
inner join 
Customers
on Orders.CustomerID = Customers.CustomerID
```



<iframe src="https://towardsdatascience.com/media/8b3c8c7cfc1de35e2bb7aa860700ace7" allowfullscreen="" frameborder="0" height="303" width="692" title="Optimize_join.sql" class="fq aq as ag cf" scrolling="auto" style="box-sizing: inherit; height: 303px; top: 0px; left: 0px; width: 692px; position: absolute;"></iframe>

\4. **Don’t Use Distinct**: The Distinct clause is used to find the distinct rows corresponding to the selected columns by dropping the duplicate rows. The ‘Distinct’ clause is a time-consuming operation in SQL and the alternative is to use ‘group by’. For example, the below queries find the count of orders from the order details table:

```
# Expensive operation - count distinct order ids - using distinct 
SELECT count(distinct OrderID)
FROM OrderDetails

# Right way - count distinct order ids - using groupby  
select count(*) 
from
(SELECT OrderID
FROM OrderDetails
group by OrderID)
```



<iframe src="https://towardsdatascience.com/media/766812dd3c5adbfb4c1504606de14483" allowfullscreen="" frameborder="0" height="259" width="692" title="Optimize_distinct.sql" class="fq aq as ag cf" scrolling="auto" style="box-sizing: inherit; height: 259px; top: 0px; left: 0px; width: 692px; position: absolute;"></iframe>

\5. **Avoid Predicates in filtering**: Predicate is an expression that equates to a boolean value i.e. True or False. Using predicates to perform filtering operation slows down the execution time as the predicates do not use the indexes (SQL index is a lookup table for quick retrieval). So, other alternatives should be used for filtering. For example, if we want to find the suppliers who have a phone number starting with (171) code.

```
# expensive way 
SELECT SupplierID,SupplierName FROM Suppliers
where SUBSTR(Phone, 1, 5) = '(171)'

# right way 
SELECT SupplierID,SupplierName FROM Suppliers
where Phone like  '(171)%'
```

## concepts-and-questions-to-prep-for-your-data-analyst-job-interview

https://towardsdatascience.com/concepts-and-questions-to-prep-for-your-data-analyst-job-interview-a075d571dae8

https://www.stratascratch.com/blog/data-analyst-interview-questions-and-answers/?utm_source=blog&utm_medium=click&utm_campaign=medium

https://www.stratascratch.com/blog/sql-interview-questions-for-the-data-analyst-position/?utm_source=blog&utm_medium=click&utm_campaign=medium