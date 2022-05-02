## Formatting Your Queries

### Using Upper and Lower Case in SQL

SQL queries can be run successfully whether characters are written in upper- or lower-case. In other words, SQL queries are not case-sensitive. The following query:

 **is common and best practice to capitalize all SQL commands, like SELECT and FROM, and keep everything else in your query lower case.**

Capitalizing command words makes queries easier to read, which will matter more as you write more complex queries. For now, it is just a good habit to start getting into, to make your SQL queries more readable.

One other note: The text data stored in SQL tables can be either upper or lower case, and SQL *is* case-sensitive in regard to this text data.



### Commands

You have already learned a lot about writing code in SQL! Let's take a moment to recap all that we have covered before moving on:

| **Statement** | **How to Use It**               | **Other Details**                                     |
| :------------ | :------------------------------ | :---------------------------------------------------- |
| SELECT        | SELECT **Col1**, **Col2**, ...  | Provide the columns you want                          |
| FROM          | FROM **Table**                  | Provide the table where the columns exist             |
| LIMIT         | LIMIT **10**                    | Limits based number of rows returned                  |
| ORDER BY      | ORDER BY **Col**                | Orders table based on the column. Used with **DESC**. |
| WHERE         | WHERE **Col > 5**               | A conditional statement to filter your results        |
| LIKE          | WHERE **Col LIKE '%me%'**       | Only pulls rows where column has 'me' within the text |
| IN            | WHERE **Col IN ('Y', 'N')**     | A filter for only rows with column of 'Y' or 'N'      |
| NOT           | WHERE **Col NOT IN ('Y', 'N')** | **NOT** is frequently used with **LIKE** and **IN**   |
| AND           | WHERE **Col1 > 5 AND Col2 < 3** | Filter rows where two or more conditions must be true |
| OR            | WHERE **Col1 > 5 OR Col2 < 3**  | Filter rows where at least one condition must be true |
| BETWEEN       | WHERE **Col BETWEEN 3 AND 5**   | Often easier syntax than using an **AND**             |

### Other Tips

Though SQL is **not case sensitive** (it doesn't care if you write your statements as all uppercase or lowercase), we discussed some best practices. **The order of the key words does matter!** Using what you know so far, you will want to write your statements as:

```
SELECT col1, col2
FROM table1
WHERE col3  > 5 AND col4 LIKE '%os%'
ORDER BY col5
LIMIT 10;
```

Notice, you can retrieve different columns than those being used in the **ORDER BY** and **WHERE** statements. Assuming all of these column names existed in this way (`col1`, `col2`, `col3`, `col4`, `col5`) within a table called `table1`, this query would run just fine.





# Recap

### Primary and Foreign Keys

You learned a key element for **JOIN**ing tables in a database has to do with primary and foreign keys:

- **primary keys** - are unique for every row in a table. These are generally the first column in our database (like you saw with the **id** column for every table in the Parch & Posey database).
- **foreign keys** - are the **primary key** appearing in another table, which allows the rows to be non-unique.

Choosing the set up of data in our database is very important, but not usually the job of a data analyst. This process is known as **Database Normalization**.

### JOINs

In this lesson, you learned how to combine data from multiple tables using **JOIN**s. The three **JOIN** statements you are most likely to use are:

1. **JOIN** - an **INNER JOIN** that only pulls data that exists in both tables.
2. **LEFT JOIN** - pulls all the data that exists in both tables, as well as all of the rows from the table in the **FROM** even if they do not exist in the **JOIN** statement.

## Parte 3 



### Group By



The key takeaways here:

- **GROUP BY** can be used to aggregate data within subsets of the data. For example, grouping for different accounts, different regions, or different sales representatives.

  

- Any column in the **SELECT** statement that is not within an aggregator must be in the **GROUP BY** clause.

  

- The **GROUP BY** always goes between **WHERE** and **ORDER BY**.

  

- **ORDER BY** works like **SORT** in spreadsheet software.

### GROUP BY - Expert Tip

Before we dive deeper into aggregations using **GROUP BY** statements, it is worth noting that SQL evaluates the aggregations before the **LIMIT** clause. If you don’t group by any columns, you’ll get a 1-row result—no problem there. If you group by a column with enough unique values that it exceeds the **LIMIT** number, the aggregates will be calculated, and then some rows will simply be omitted from the results.

This is actually a nice way to do things because you know you’re going to get the correct aggregates. If SQL cuts the table down to 100 rows, then performed the aggregations, your results would be substantially different. The above query’s results exceed 100 rows, so it’s a perfect example. In the next concept, use the SQL environment to try removing the **LIMIT** and running it again to see what changes.



Key takeaways:

- You can **GROUP BY** multiple columns at once, as we showed here. This is often useful to aggregate across a number of different segments.

  

- The order of columns listed in the **ORDER BY** clause does make a difference. You are ordering the columns from left to right.

### GROUP BY - Expert Tips

- The order of column names in your **GROUP BY** clause doesn’t matter—the results will be the same regardless. If we run the same query and reverse the order in the **GROUP BY** clause, you can see we get the same results.

  

- As with **ORDER BY**, you can substitute numbers for column names in the **GROUP BY** clause. It’s generally recommended to do this only when you’re grouping many columns, or if something else is causing the text in the GROUP BY clause to be excessively long.

  

- A reminder here that any column that is not within an aggregation must show up in your GROUP BY statement. If you forget, you will likely get an error. However, in the off chance that your query does work, you might not like the results!

## DISTINCT CLAUSUE

**DISTINCT** is always used in **SELECT** statements, and it provides the unique rows for all columns written in the **SELECT** statement. Therefore, you only use **DISTINCT** once in any particular **SELECT** statement.

You could write:

```
SELECT DISTINCT column1, column2, column3
FROM table1;
```

which would return the unique (or **DISTINCT**) rows across all three columns.

You would **not** write:

```
SELECT DISTINCT column1, DISTINCT column2, DISTINCT column3
FROM table1;
```

You can think of **DISTINCT** the same way you might think of the statement "unique".

### DISTINCT - Expert Tip

It’s worth noting that using **DISTINCT**, particularly in aggregations, can slow your queries down quite a bit.



### CLAUSLUE HAVING

HAVING - Expert Tip**HAVING** is the “clean” way to filter a query that has been aggregated, but this is also commonly done using a [subquery](https://community.modeanalytics.com/sql/tutorial/sql-subqueries/). Essentially, any time you want to perform a **WHERE** on an element of your query that was created by an aggregate, you need to use **HAVING** instead.

STATEMENTE VEERDADEIROS

Often there is confusion about the difference between **WHERE** and **HAVING**. Select all the statements that are true regarding **HAVING** and **WHERE** statements.

- **WHERE** subsets the returned data based on a logical condition.
- **WHERE** appears after the **FROM**, **JOIN**, and **ON** clauses, but before **GROUP BY**.
- **HAVING** appears after the **GROUP BY** clause, but before the **ORDER BY** clause.
- **HAVING** is like **WHERE**, but it works on logical statements involving aggregations.

## CASE

### CASE - Expert Tip

- The CASE statement always goes in the SELECT clause.

  

- CASE must include the following components: WHEN, THEN, and END. ELSE is an optional component to catch cases that didn’t meet any of the other previous CASE conditions.

  

- You can make any conditional statement using any conditional operator (like [WHERE](https://mode.com/resources/sql-tutorial/sql-where)) between WHEN and THEN. This includes stringing together multiple conditional statements using AND and OR.

  

- You can include multiple WHEN statements, as well as an ELSE statement again, to deal with any unaddressed conditions.