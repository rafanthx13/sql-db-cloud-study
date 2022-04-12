

##  Advanced SQL — CTEs
Common Table Expressions using the WITH clause.

https://levelup.gitconnected.com/advanced-sql-ctes-f36d00cbf051

What is a CTE?

As stated above, a CTE (sometimes referred to as a “WITH statement”) is a temporary named result set that is not saved anywhere, it only exists in memory while the query is being run (unlike a temp table which exists in a temp DB file). The results of a CTE only exist within the execution scope of a specific statement; in other words, the results are only available to the CRUD statement immediately following the WITH clause in which the CTE is defined. CTEs are a great way to simplify and manage complex queries, making your code easier to read by breaking it into sections.

CTE syntax

I have written a simple SQL query as an example to showcase how a CTE can be used:

````sql
WITH shipped AS (
    SELECT 
    CASE
        WHEN shipped_date > required_date THEN 'LATE'
        WHEN shipped_date <= required_date THEN 'ON-TIME'
        ELSE NULL
    END AS [shipped_on_time], sc.customer_id, first_name, last_name, order_date, 
     required_date, shipped_date, store_id, order_id 
    FROM sales.customers sc
    JOIN sales.orders so
    ON sc.customer_id = so.customer_id
    WHERE order_date >= '2018-01-01'
    AND order_status = 4
)

SELECT customer_id, first_name + ' ' + last_name as full_name, shipped_on_time, order_id
FROM shipped 
WHERE shipped_on_time = 'ON-TIME'
AND store_id = 3
````

````sql
WITH CTE-name AS (CTE-definition) 
````

You can also define multiple CTEs by separating the definitions using commas:

````sql
WITH name AS (definition),
second-name AS (second-definition)
````

In order to define a CTE, you simply write a SELECT statement as your parenthetical text. This statement should capture any data that you desire to use within another query. Once you have defined your CTE, it can now be referenced in your final query:

````sql
WITH name AS (definition)
SELECT * FROM name
````

Common use cases

A CTE is an incredibly useful and flexible tool to add to your SQL repertoire. Here are a couple of the most common use cases:
+ CTEs are often used to increase the readability and manageability of queries. By defining multiple CTEs, you can break up a long, complex query into chunks of logic that can be referenced in your final query. They are also a great way to follow the DRY principle if you are referencing the same data in multiple subqueries.
+ CTEs can be used to create recursive queries, which is helpful as a normal SELECT statement cannot reference itself. See Microsoft’s documentation on recursive queries using CTEs for more information on this.

**Considerações Finais**

+ A CTE exists only in the execution scope of the subsequent statement.
+ Unlike a temp table, CTEs are not saved anywhere.
+ A CTE is defined by writing a SELECT statement in parentheses following a WITH clause.
+ A CTE can be referenced like any other table following the FROM clause in a query.
+ CTE’s are a great way to DRY up code, increase readability and manageability, and write recursive queries.


----------------

https://towardsdatascience.com/3-sql-swaps-to-write-better-code-f8d304699cde

### Subqueries → CTEs

That last example shown above can also be used for this swap! If you look above, we start with a subquery within a query and then finish by using CTEs.
What exactly is a CTE? CTE stands for common table expression. It creates a temporary set of results that you can use in your proceeding queries. It requires that you start it with WITH and use table_name AS before each individual CTE. Your last CTE in the sequence should be a simple SELECT statement without a table_name nor AS.

````sql
WITH 
money_summed AS (
   SELECT 
      customer_id,
      date,
      SUM(balance) AS balance,
      SUM(debt) AS debt
   FROM bank_balance 
   GROUP BY customer_id, date
),

money_available_calculated AS (
   SELECT 
      customer_id,
      date,
      (balance - debt) AS money_available 
   FROM money_summed 
   GROUP BY customer_id, date
)

SELECT
   customer_id,
   money_available 
FROM money_available_calculated 
WHERE date = '2022-01-01' 

````
