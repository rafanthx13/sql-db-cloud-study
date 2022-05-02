# Aula 06 -  Windows Function



**OVER** and **PARTITION BY**. These are key to window functions. Not every window function uses **PARTITION BY**; we can also use **ORDER BY** or no statement at all depending on the query we want to run. You will practice using these clauses in the upcoming quizzes. If you want more details right now, [this resource](https://blog.sqlauthority.com/2015/11/04/sql-server-what-is-the-over-clause-notes-from-the-field-101/) from Pinal Dave is helpful.

*Note: You can’t use window functions and standard aggregations in the same query. More specifically, you can’t include window functions in a **GROUP BY** clause.*

### Windows Function 1

```
SELECT standard_amt_usd,
       SUM(standard_amt_usd) OVER (ORDER BY occurred_at) AS running_total
FROM orders
```

Saida 6912 results

| standard_amt_usd | running_total |
| :--------------- | :------------ |
| 0.00             | 0.00          |
| 2445.10          | 2445.10       |
| 2634.72          | 5079.82       |
| 0.00             | 5079.82       |
| 2455.08          | 7534.90       |
| 2504.98          | 10039.88      |
| 264.47           | 10304.35      |
| 1536.92          | 11841.27      |
| 374.25           | 12215.52      |
| 1402.19          | 13617.71      |
| 59.88            | 13677.59      |
| 2300.39          | 15977.98      |

### Windows Function 2

```
SELECT standard_amt_usd,
       DATE_TRUNC('year', occurred_at) as year,
       SUM(standard_amt_usd) OVER (PARTITION BY DATE_TRUNC('year', occurred_at) ORDER BY occurred_at) AS running_total
FROM orders
```



| standard_amt_usd | year                     | running_total |
| :--------------- | :----------------------- | :------------ |
| 0.00             | 2013-01-01T00:00:00.000Z | 0.00          |
| 2445.10          | 2013-01-01T00:00:00.000Z | 2445.10       |
| 2634.72          | 2013-01-01T00:00:00.000Z | 5079.82       |
| 0.00             | 2013-01-01T00:00:00.000Z | 5079.82       |

## Row_NUMBER

```
SELECT id,
       account_id,
       total,
       RANK() OVER (PARTITION BY account_id ORDER BY total DESC) AS total_rank
FROM orders
```

| id   | account_id | total | total_rank |
| :--- | :--------- | :---- | :--------- |
| 4308 | 1001       | 1410  | 1          |
| 4309 | 1001       | 1405  | 2          |
| 4316 | 1001       | 1384  | 3          |
| 4317 | 1001       | 1347  | 4          |

## ALIAS para Windows Function

Funciton no PostGre, naosei se funciona no MYSQL

```
SELECT id,
       account_id,
       DATE_TRUNC('year',occurred_at) AS year,
       DENSE_RANK() OVER account_year_window AS dense_rank,
       total_amt_usd,
       SUM(total_amt_usd) OVER account_year_window AS sum_total_amt_usd,
       COUNT(total_amt_usd) OVER account_year_window AS count_total_amt_usd,
       AVG(total_amt_usd) OVER account_year_window AS avg_total_amt_usd,
       MIN(total_amt_usd) OVER account_year_window AS min_total_amt_usd,
       MAX(total_amt_usd) OVER account_year_window AS max_total_amt_usd
FROM orders 
WINDOW account_year_window AS (PARTITION BY account_id ORDER BY DATE_TRUNC('year',occurred_at))
```

Serve para voce fazer diversas chamadas para a mesma windows fucntion

## LEAD e LAG

LAG cria uma coluna cujo valor é referente a um attr da ROW ANTERIOR

LEAD faz da ROW seguintes

EXEMPLO

```
SELECT account_id,
       standard_sum,
       LAG(standard_sum) OVER (ORDER BY standard_sum) AS lag,
       LEAD(standard_sum) OVER (ORDER BY standard_sum) AS lead,
       standard_sum - LAG(standard_sum) OVER (ORDER BY standard_sum) AS lag_difference,
       LEAD(standard_sum) OVER (ORDER BY standard_sum) - standard_sum AS lead_difference
FROM (
SELECT account_id,
       SUM(standard_qty) AS standard_sum
  FROM orders 
 GROUP BY 1
 ) sub
```

SAIDA

| account_id | standard_sum | lag  | lead | lag_difference | lead_difference |
| :--------- | :----------- | :--- | :--- | :------------- | :-------------- |
| 1901       | 0            |      | 79   |                | 79              |
| 3371       | 79           | 0    | 102  | 79             | 23              |
| 1961       | 102          | 79   | 116  | 23             | 14              |
| 3401       | 116          | 102  | 117  | 14             | 1               |
| 3741       | 117          | 116  | 123  | 1              | 6               |
| 4321       | 123          | 117  | 148  | 6              | 25              |
| 3941       | 148          | 123  | 149  | 25             | 1               |
| 1671       | 149          | 148  | 183  | 1              | 34              |
| 3521       | 183          | 149  | 290  | 34             |                 |



```
# Comparing a Row to a Previous Row
SELECT occurred_at,
       total_amt_usd,
       LEAD(total_amt_usd) OVER (ORDER BY occurred_at) AS lead,
       LEAD(total_amt_usd) OVER (ORDER BY occurred_at) - total_amt_usd AS lead_difference
FROM (
SELECT occurred_at,
       SUM(total_amt_usd) AS total_amt_usd
  FROM orders 
 GROUP BY 1
) sub
```



| occurred_at              | total_amt_usd | lead    | lead_difference |
| :----------------------- | :------------ | :------ | :-------------- |
| 2013-12-04T04:22:44.000Z | 627.48        | 2646.77 | 2019.29         |
| 2013-12-04T04:45:54.000Z | 2646.77       | 2709.62 | 62.85           |
| 2013-12-04T04:53:25.000Z | 2709.62       | 277.13  | -2432.49        |
| 2013-12-05T20:29:16.000Z | 277.13        | 3001.85 | 2724.72         |
| 2013-12-05T20:33:56.000Z | 3001.85       | 2802.90 | -198.95         |

# Windows Function pela documentaçao do postgre

https://www.postgresql.org/docs/9.1/tutorial-window.html

O QUE É EM ESSêNCIA

A *window function* performs a calculation across a set of table rows that are somehow related to the current row. This is comparable to the type of calculation that can be done with an aggregate function. But unlike regular aggregate functions, use of a window function does not cause rows to become grouped into a single output row — the rows retain their separate identities.

### Exemplos

**OBS: Você não usa WIINDOWS FUNCTION COM GROUP BY, NUCA OS 2 JUNTOS**

```
SELECT 
	depname, 
	empno, 
	salary, 
	avg(salary) OVER (PARTITION BY depname)
FROM empsalary;

#### O QUE FAZ
# vai fazer o avg de salario para cada depname e não vai agrgar, vai por em cada caso. É como o group by sem reduzir linhas
```



```
SELECT 
    depname, 
    empno, 
    salary,
    rank() OVER (PARTITION BY depname ORDER BY salary DESC) 
FROM empsalary;

#### O QUE FAZ
# Vai criar coluna de rank e vai ordenar os dados de acordo com esse rank, vai rankear de acordo com salary
```



```
SELECT 
	salary, 
	sum(salary) OVER () 
FROM empsalary;
#### O QUE FAZ
# Vai criar uma coluna constate chamada 'sum' cujo valor sera constatne :  a soma de salary
```

**Windows function com order**

```
SELECT 
	salary, 
	sum(salary) OVER (ORDER BY salary) 
FROM empsalary;
#### O QUE FAZ
# Vai ser uma soma acumulativa, e vai estar ordenado do menor (1 row) para o maior (ultima row)
```

### Tabela de Windows function

| Function                                                     | Return Type          | Description                                                  |
| ------------------------------------------------------------ | -------------------- | ------------------------------------------------------------ |
| `row_number()`                                               | `bigint`             | number of the current row within its partition, counting from 1 |
| `rank()`                                                     | `bigint`             | rank of the current row with gaps; same as `row_number` of its first peer |
| `dense_rank()`                                               | `bigint`             | rank of the current row without gaps; this function counts peer groups |
| `percent_rank()`                                             | `double precision`   | relative rank of the current row: (`rank` - 1) / (total rows - 1) |
| `cume_dist()`                                                | `double precision`   | relative rank of the current row: (number of rows preceding or peer with current row) / (total rows) |
| `ntile(num_buckets integer)`                                 | `integer`            | integer ranging from 1 to the argument value, dividing the partition as equally as possible |
| `lag(value anyelement [, offset integer [, default anyelement ]])` | `same type as value` | returns `value` evaluated at the row that is `offset` rows before the current row within the partition; if there is no such row, instead return `default` (which must be of the same type as `value`). Both `offset` and `default` are evaluated with respect to the current row. If omitted, `offset` defaults to 1 and `default` to null |
| `lead(value anyelement [, offset integer [, default anyelement ]])` | `same type as value` | returns `value` evaluated at the row that is `offset` rows after the current row within the partition; if there is no such row, instead return `default` (which must be of the same type as `value`). Both `offset` and `default` are evaluated with respect to the current row. If omitted, `offset` defaults to 1 and `default` to null |
| `first_value(value any)`                                     | `same type as value` | returns `value` evaluated at the row that is the first row of the window frame |
| `last_value(value any)`                                      | `same type as value` | returns `value` evaluated at the row that is the last row of the window frame |
| `nth_value(value any, nth integer)`                          | `same type as value` | returns `value` evaluated at the row that is the `nth` row of the window frame (counting from 1); null if no such row |