# Aula 4 - SQL Subqueries & Temporary Table



## Intro

### What this lesson is about...

Up to this point you have learned a lot about working with data using SQL. This lesson will focus on three topics:

1. Subqueries
2. Table Expressions
3. Persistent Derived Tables

------

Both **subqueries** and **table expressions** are methods for being able to write a query that creates a table, and then write a query that interacts with this newly created table. Sometimes the question you are trying to answer doesn't have an answer when working directly with existing tables in database.

However, if we were able to create new tables from the existing tables, we know we could query these new tables to answer our question. This is where the queries of this lesson come to the rescue.

If you can't yet think of a question that might require such a query, don't worry because you are about to see a whole bunch of them!

## Subquerie



Elas prescisam ter o **ALIAS **SEMPRE. Lembres, a keyword ALIAS é opcional para criar esse apelido

Exemplo de Subqueries

```
SELECT 
	channel, 
	AVG(events) AS average_events
FROM (
	SELECT DATE_TRUNC('day',occurred_at) AS day,
             channel, COUNT(*) as events
      FROM web_events 
      GROUP BY 1,2) sub
GROUP BY channel
ORDER BY 2 DESC;
```

### Usar Subquerie além do FROM

Em gerla, agente usa mais no FROM, substituindo uma tabela pelo SELECT-Subqueries.

**MAS PODEMOS SUBSTITUIR EM QUALQUER LUGAR QUE SE REFEIRA A UM VALOR OU A UMA COLUNA**

Se você que por exemplo usar no where, em geral, você pode retornar uma coluna ou epans um valor a depender do que se refere:

+ EXEMPLO: `WHERE nome = (SELECT MAX(a.nome )...)`
  + Se no wheer vocÊ quer usar a subquries como valor, ela deve entao sempre retotnar apenas um único valor
+  EXEMPLO: `WHERE nome IN (SELECT a.nome ...)`
  + SE VOCÊ CHAMA A SUBQUERIE PARA O `IN` entâo deve retornar apenas uma coluna

**Observaçâo**

+ Em casos que usa subquery sem ser no FROM **NÃO PODE COLCOAR UM ALIAS**
+ Casos de Subquery



Exemplos

```
SELECT AVG(standard_qty) avg_std, AVG(gloss_qty) avg_gls, AVG(poster_qty) avg_pst
FROM orders
WHERE DATE_TRUNC('month', occurred_at) = 
     (SELECT DATE_TRUNC('month', MIN(occurred_at)) FROM orders);

SELECT SUM(total_amt_usd)
FROM orders
WHERE DATE_TRUNC('month', occurred_at) = 
      (SELECT DATE_TRUNC('month', MIN(occurred_at)) FROM orders);
      
      
SELECT t3.rep_name, t3.region_name, t3.total_amt
FROM(SELECT region_name, MAX(total_amt) total_amt
     FROM(SELECT s.name rep_name, r.name region_name, SUM(o.total_amt_usd) total_amt
             FROM sales_reps s
             JOIN accounts a
             ON a.sales_rep_id = s.id
             JOIN orders o
             ON o.account_id = a.id
             JOIN region r
             ON r.id = s.region_id
             GROUP BY 1, 2) t1
     GROUP BY 1) t2
JOIN (SELECT s.name rep_name, r.name region_name, SUM(o.total_amt_usd) total_amt
     FROM sales_reps s
     JOIN accounts a
     ON a.sales_rep_id = s.id
     JOIN orders o
     ON o.account_id = a.id
     JOIN region r
     ON r.id = s.region_id
     GROUP BY 1,2
     ORDER BY 3 DESC) t3
ON t3.region_name = t2.region_name AND t3.total_amt = t2.total_amt;

SELECT COUNT(*)
FROM (SELECT a.name
       FROM orders o
       JOIN accounts a
       ON a.id = o.account_id
       GROUP BY 1
       HAVING SUM(o.total) > (SELECT total 
                   FROM (SELECT a.name act_name, SUM(o.standard_qty) tot_std, SUM(o.total) total
                         FROM accounts a
                         JOIN orders o
                         ON o.account_id = a.id
                         GROUP BY 1
                         ORDER BY 2 DESC
                         LIMIT 1) inner_tab)
             ) counter_tab;
             
SELECT AVG(avg_amt)
FROM (SELECT o.account_id, AVG(o.total_amt_usd) avg_amt
    FROM orders o
    GROUP BY 1
    HAVING AVG(o.total_amt_usd) > (SELECT AVG(o.total_amt_usd) avg_all
                                   FROM orders o)) temp_table;
```

## **WITH** -  **Common Table Expression** or **CTE**

The **WITH** statement is often called a **Common Table Expression** or **CTE**. Though these expressions serve the exact same purpose as subqueries, they are more common in practice, as they tend to be cleaner for a future reader to follow the logic.

In the next concept, we will walk through this example a bit more slowly to make sure you have all the similarities between subqueries and these expressions down for you to use in practice! If you are already feeling comfortable skip ahead to practice the quiz section.

**Porque usar CTE ao invez de SubQuerie**

CTE é usado quando a consulta interna é bem demorada. A CTE cria uma tabela temporaria, entao, toda vez que fizer essa subconsutla ela já vai está noa memória.

Usamos CTE ao invez de subquerie por questoa de eficientea, se a querie interna for muito demorara ou se repetir demais nas nossas consutlas

**Exemplos simples**

```
WITH events AS (
          SELECT DATE_TRUNC('day',occurred_at) AS day, 
                        channel, COUNT(*) as events
          FROM web_events 
          GROUP BY 1,2)

SELECT channel, AVG(events) AS average_events
FROM events
GROUP BY channel
ORDER BY 2 DESC;

WITH table1 AS (
          SELECT *
          FROM web_events),

     table2 AS (
          SELECT *
          FROM accounts)


SELECT *
FROM table1
JOIN table2
ON table1.account_id = table2.id;
```

**obs**

+ As CTE são semrepe com alias

## Exemplos mais complxso de CTE

CTE dentro de outra dentro da consulta normal

```
WITH t1 AS (
   SELECT r.name region_name, SUM(o.total_amt_usd) total_amt
   FROM sales_reps s
   JOIN accounts a
   ON a.sales_rep_id = s.id
   JOIN orders o
   ON o.account_id = a.id
   JOIN region r
   ON r.id = s.region_id
   GROUP BY r.name), 
t2 AS (
   SELECT MAX(total_amt)
   FROM t1)
SELECT r.name, COUNT(o.total) total_orders
FROM sales_reps s
JOIN accounts a
ON a.sales_rep_id = s.id
JOIN orders o
ON o.account_id = a.id
JOIN region r
ON r.id = s.region_id
GROUP BY r.name
HAVING SUM(o.total_amt_usd) = (SELECT * FROM t2);
```

Outro exemplo

```
WITH t1 AS (
   SELECT a.id, a.name, SUM(o.total_amt_usd) tot_spent
   FROM orders o
   JOIN accounts a
   ON a.id = o.account_id
   GROUP BY a.id, a.name
   ORDER BY 3 DESC
   LIMIT 1)
SELECT a.name, w.channel, COUNT(*)
FROM accounts a
JOIN web_events w
ON a.id = w.account_id AND a.id =  (SELECT id FROM t1)
GROUP BY 1, 2
ORDER BY 3 DESC;
```

Outro caso

```
WITH t1 AS (
   SELECT AVG(o.total_amt_usd) avg_all
   FROM orders o
   JOIN accounts a
   ON a.id = o.account_id),
t2 AS (
   SELECT o.account_id, AVG(o.total_amt_usd) avg_amt
   FROM orders o
   GROUP BY 1
   HAVING AVG(o.total_amt_usd) > (SELECT * FROM t1))
SELECT AVG(avg_amt)
FROM t2;
```

## Porque usamos CTE

+ Para nao ter uma subquerie dentro de outra. Estando fora fica mais fácil gerenciar tudo
+ Quando voce chama uma subquerie mais de uma vez em lugares difernetes. Ao invez de escrever ela duas vezes, com uma CTE você só referencia ela e a consulta é feita apenas uma unica vez