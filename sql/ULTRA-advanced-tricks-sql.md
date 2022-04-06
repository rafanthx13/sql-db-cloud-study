# links

https://towardsdatascience.com/3-sql-swaps-to-write-better-code-f8d304699cde

## https://towardsdatascience.com/3-sql-swaps-to-write-better-code-f8d304699cde

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
### ≤ and ≥ → BETWEEN

-----------------

https://www.reddit.com/r/dataengineering/comments/rva6nc/how_did_you_become_a_sql_pro/

https://medium.com/analytics-and-data/the-path-to-learning-sql-and-mastering-it-to-become-a-data-engineer-256ea0fef4e7

https://levelup.gitconnected.com/advanced-sql-for-data-professionals-e0ef0740e8e1

https://www.datacamp.com/community/tutorials/sql-tutorial-query?tap_a=5644-dce66f&tap_s=638387-3d2eb6
