# Ultra Review de SQL

Reunião de todas as minhas anotações

## TOC - Index

Gerado em: https://rafanthx13.github.io/md-toc-generator/

+ [Ultra Review de SQL](#ultra-review-de-sql)
  - [TOC - Index](#toc---index)
  - [Tasks](#tasks)
    * [Comentários](#comentários)
    * [Quantitade de valores duplicados](#quantitade-de-valores-duplicados)
    * [Mediana](#mediana)
  - [Keyword in SQL](#keyword-in-sql)
    * [Apelido/alias/`AS`](#apelido/alias/`as`)
    * [`MOD%` Par ou Impar](#`mod%`-par-ou-impar)
    * [Comparar valor](#comparar-valor)
    * [`CASE`](#`case`)
    * [Null Functions](#null-functions)
    * [`CAST`](#`cast`)
    * [Number Functions](#number-functions)
    * [Date Functions](#date-functions)
    * [`ORDER BY`](#`order-by`)
    * [Regex](#regex)
    * [`USING`](#`using`)
    * [String Functions](#string-functions)
    * [FULL PUTER JOIN NO MYSQL](#full-puter-join-no-mysql)
  - [Conceitos Avançados](#conceitos-avançados)
    * [Subquery](#subquery)
      + [Correlated Subqueries](#correlated-subqueries)
      + [SubQuery as Value](#subquery-as-value)
      + [Subquery as list_of_values](#subquery-as-list_of_values)
      + [Subquerie no FROM](#subquerie-no-from)
    * [CTE](#cte)
    * [Pivot Table](#pivot-table)
      + [Exemplo Simples](#exemplo-simples)
      + [Pivot Reverse](#pivot-reverse)
      + [Outros Exemplos de pivot table](#outros-exemplos-de-pivot-table)
  - [Window Function](#window-function)
    * [CheatSheet Resume](#cheatsheet-resume)
  - [NULL BEHAVIOR](#null-behavior)
    * [Excluindo um valor pode excluir inclusive NULL](#excluindo-um-valor-pode-excluir-inclusive-null)
    * [Aggregation Behavior](#aggregation-behavior)
    * [Ordering Behavior](#ordering-behavior)
    * [Em condicional](#em-condicional)
  - [Otimização](#otimização)
    * [Usar `GROUP BY` ao invés de `DISTINCT`](#usar-`group-by`-ao-invés-de-`distinct`)
    * [From Git - Query Optimization](#from-git---query-optimization)

## Tasks

Resolver certas demandas que pode aparecer

### Comentários

```
Curto: use --
Longo: use /* */ 
```



### Quantitade de valores duplicados

```sql
SELECT COUNT(CITY) - COUNT(DISTINCT(CITY)) FROM STATION;
-- Será > 0 se houver houver city iguais (ou seja, duplicadas)
```

### Mediana

```sql
Select round(S.LAT_N,4) mediam 
from station S 
where (
    select count(Lat_N) from station where Lat_N < S.LAT_N 
) = (
    select count(Lat_N) from station where Lat_N > S.LAT_N
)
```

## Keyword in SQL

### Apelido/alias/`AS`

É opcional colocar a KEYWORD `AS ` mas recomendável por favorecer leitura

### `MOD%` Par ou Impar

```sql
SELECT *
FROM cinema
WHERE 
    id % 2 <> 0 -- want id impar/odd
    AND 
    description NOT LIKE '%boring%' # nao deve ter boring
ORDER BY 
    rating DESC;
```

### Comparar valor

```
<> = DIFERENTE | > = MENOS        | AND
=  = IGUALDADE | >= = MENOR IGUAL | OR
<  = MAIS      | <= MAIOR IGUAL   | NOT
IN
NOT IN
IS NOT NULL
IS NULL
BETWEEN == betwen pode ser usada entre datas
  - BETWEEN "2013-10-01" AND "2013-10-03"
IF(GRADE < 8, NULL, NAME)
```



### `CASE`

`CASE` .... N * () `WHEN` bool `THEN` value) `END` `AS ` `alias`. Usa-se `ELSE` também

**Exemplos**:

EX1

```sql
-- detectar tipo de triangulo 
SELECT
CASE
    WHEN A + B <= C 
    THEN "Not A Triangle"
    
    WHEN A = B AND B = C
    THEN "Equilateral"
    
    WHEN A != B AND B != C AND A != C
    THEN "Scalene"  ELSE  "Isosceles"
       
END
from triangles;
```

EX2

```sql
SELECT 
    id,
    CASE 
        WHEN p_id IS NULL 
            THEN 'Root'
        WHEN id IN (SELECT p_id FROM Tree) 
            THEN 'Inner'
        ELSE 'Leaf'
    END AS type
FROM Tree
```

EX3

```SQL
SELECT
	CASE
        -- quando for o ultimo caso
		WHEN seat.id % 2 <> 0 AND seat.id = (SELECT COUNT(*) FROM seat) 
            THEN seat.id
        -- quando tiver numero par -1
		WHEN seat.id % 2 = 0 
            THEN seat.id - 1
        -- quando id impar + 1
		ELSE
			seat.id + 1
	END as id,
	student 
FROM seat
ORDER BY id
```

### Null Functions

IFFNULL: Se for nulo, faz um relplace

EX1

```sql
SELECT 
    u.user_id AS buyer_id, 
    join_date, 
    IFNULL(COUNT(order_date), 0) AS orders_in_2019 
FROM Users as u LEFT JOIN Orders as o
    ON u.user_id = o.buyer_id
        AND YEAR(order_date) = '2019'
GROUP BY
    u.user_id
```

### `CAST`

Converter tipos. Em geral fazemos entre string e Date

EX1

```sql
SELECT 
    s.product_id, product_name
FROM Sales s LEFT JOIN Product p
    ON s.product_id = p.product_id
GROUP BY 
    s.product_id
HAVING MIN(sale_date) >= CAST('2019-01-01' AS DATE) AND
       MAX(sale_date) <= CAST('2019-03-31' AS DATE)
```

### Number Functions

```
ROUND(value, [decima_cases]) = Areddonda
FLOOR(value) = Aredonda para o INT mais baixo
```

### Date Functions

```
YEAR(time_stamp) = 2020 

where 
    datediff('2019-07-27', activity_date) < 30
```

link: 

https://medium.com/@aakriti.sharma18/data-cleaning-with-sql-eaab6d29d007

Parsing Dates
If the date is written as a string we can surely convert it to date data type using CAST. But this only works when it is in an SQL identified format. But what if it is other formats like MM/DD/YYYY? We first need to convert it to a string in an acceptable format and then cast it. For example,

(SUBSTR(date, 7, 4) || ‘-’ || LEFT(date, 2) || ‘-’ || SUBSTR(date, 4, 2))::date AS cleaned_date
EXTRACT
A lot of times we need to take into account a specific part of the date, like sales in this month, admissions in this year. In such cases, we extract information from the date column as :

EXTRACT('year'   FROM cleaned_date) AS year,
EXTRACT('month'  FROM cleaned_date) AS month,        
EXTRACT('day'    FROM cleaned_date) AS day,        
EXTRACT('hour'   FROM cleaned_date) AS hour,        
EXTRACT('minute' FROM cleaned_date) AS minute,        EXTRACT('second' FROM cleaned_date) AS second,        EXTRACT('decade' FROM cleaned_date) AS decade,        
EXTRACT('dow'    FROM cleaned_date) AS day_of_week
NOW
SQL provides a wide variety of functions to retrieve the current date, time, and timestamp. Fun fact: They can be printed without the FROM clause.

SELECT CURRENT_DATE AS date,        
CURRENT_TIME AS time,        
CURRENT_TIMESTAMP AS timestamp,        
LOCALTIME AS localtime,        
LOCALTIMESTAMP AS localtimestamp,
CURRENT_TIME AT TIME ZONE 'PST' AS time_pst,        
NOW() AS now

### `ORDER BY`

**O default é ASC**

```sql
SELECT Name
FROM Employee
ORDER BY Name; -- é o mesmo que ORDER BY NAME ASC
```

**ORDER BY pode ter mais de um valor**

```sql
select 
    u.name, 
    ifnull(sum(r.distance), 0) as travelled_distance
-- lef join pois pode nao ter distance
from users u left join rides r
    on u.id = r.user_id
group by 
    r.user_id
order by 
    -- order by eh a ultima coisa a ser feita,
    -- por isso da pra fazer sobre o attr com alias
    travelled_distance desc, u.name asc
```

**Da pra ordenar por qualquer coisa**

```sql
SELECT Name
FROM Students
WHERE Marks > 75
ORDER BY SUBSTR(Name, - 3), Id ASC;
-- ordena pelos 3 ultimos caracters do nome
-- perceba que SUBSTR é um split, que, se negativo, epga de tras pra fenrete como em python
```

**`ORDER BY 1 e GROUP BY 1`**

Quando você ver isso, o número indica a coluna. 

Ao invez de escrever

```sql
select author_id as id 
from Views 
where author_id = viewer_id
group by author_id
order by author_id;
```

Voce pode escrever assim

```sql
select author_id as id 
from Views 
where author_id = viewer_id
group by 1
order by 1;
```

**COMEÇA DE 1**

### Regex

```
exemplos:
REGEXP '^DIAB1| +DIAB1'
```

Uso de REGEX: Pode ser tanto RLIKE quanto REGEXP, SÃO A MESMA COISA


**BUSCA OS NOMES DAS CIDADE QUE COMEÇAM COM VOGAL**

```sql
select city from station where city REGEXP '^[aeiou]';
```

**Termina com vogal**

```sql
select distinct(city) from station where city REGEXP '[aeiou]$';
```

**termina e começa com vogal**

```sql
select distinct(city) from station where city REGEXP '^[aeiou]+(.)+[aeiou]$';
```

**que comecenão comece com  vogal**

```sql
select distinct city from station where city NOT REGEXP '^[aeiou]';
```

**nao sei como mas esse caso é difernete do 'que comece e termine com vogal'**

```sql
SELECT DISTINCT City
FROM Station
WHERE REGEXP_LIKE(City, '^[^AEIOU].*[^aeiou]$');
```

### `USING`

`USING` o `ON a.id = b.id` quando

EX1

```sql
SELECT T.employee_id
FROM
  (SELECT * FROM Employees LEFT JOIN Salaries USING(employee_id)
   UNION 
   SELECT * FROM Employees RIGHT JOIN Salaries USING(employee_id))
AS T
WHERE T.salary IS NULL OR T.name IS NULL
ORDER BY employee_id;
```

EX2

```sql
select
    name as NAME,
    sum(amount) as BALANCE
from Users inner join Transactions USING(account)
group by account
having sum(amount) > 10000 
```



### String Functions 

```sql
-- REPLACE
REPLACE (str, find, repl)

-- REVERSE
REVERSE('Hello') --returns olleH


-- SLICE
SUBSTRING ( string_expression, start, length )
SELECT SUBSTRING('Hello', 1, 2) --returns 'He'
SELECT SUBSTRING('Hello', 3, 3) --returns 'llo'
SELECT LEFT('Hello',2) --return He
SELECT RIGHT('Hello',2) --return lo

-- MAISUCULO MINUSCUÇP
SELECT UPPER('HelloWorld') --returns 'HELLOWORLD'
SELECT LOWER('HelloWorld') --returns 'helloworld'

-- CONCAT
SELECT 'Hello' || 'World' || '!'; --returns HelloWorld!
SELECT CONCAT('Hello', 'World', '!'); --returns 'HelloWorld!'
GROUP_CONCAT(list_ov_values)
 -- funciona em Group by

-- REPLICATE (FOR ....)
SELECT REPLICATE ('Hello',4) --returns 'HelloHelloHelloHello'

```

### FULL PUTER JOIN NO MYSQL

O Mysql nao tem, entao agente implementa uisando `LEFT` e `RIGHT`

```sql
SELECT T.employee_id
FROM
  -- Left _ righ = full outer join
  -- Using é quando duas tabelas tem o mesmo nome apra pk=fk
  (SELECT * FROM Employees LEFT JOIN Salaries USING(employee_id)
   UNION 
   SELECT * FROM Employees RIGHT JOIN Salaries USING(employee_id))
AS T
WHERE T.salary IS NULL OR T.name IS NULL
ORDER BY employee_id;
```



## Conceitos Avançados

### Subquery

####  Correlated Subqueries

Correlated (also known as Synchronized or Coordinated) Subqueries are nested queries that make references to the current row of their outer query:

Lembra umpouco o for da progrmaaçâo. é você fazer **VÁRIOS SELECTS** vairando como parametro um valor que está de fora

Exemplo:

```sql
SELECT EmployeeId
 FROM Employee AS eOuter
 WHERE Salary > (
     SELECT AVG(Salary)
     FROM Employee eInner
     WHERE eInner.DepartmentId = eOuter.DepartmentId
 )
-- eOuter vem de fora
```

#### SubQuery as Value

```sql
Select 
    Department.Name, 
    emp1.Name, 
    emp1.Salary 
from 
    Employee emp1 join Department 
    on emp1.DepartmentId = Department.Id
where 
    emp1.Salary = (
        Select Max(Salary) 
        from Employee emp2 
        where emp2.DepartmentId = emp1.DepartmentId
    )
```

#### Subquery as list_of_values

EX1

```sql
SELECT
    -- seleciona o maior
    MAX(salary) as SecondHighestSalary
FROM Employee 
    -- mas nao vai mostrar o 1 maior
    -- Se hpuver só um elemento: nao mostra nada 'null'
WHERE salary 
    NOT IN ( SELECT MAX(salary) FROM Employee)
```

EX2

```SQL
SELECT name
FROM salesPerson
WHERE  sales_id NOT IN
    (
        SELECT o.sales_id
        FROM orders o 
        LEFT JOIN company c
        ON o.com_id = c.com_id    
        WHERE c.name = 'RED'
    )
```

#### Subquerie no FROM

Funciona como uma tabel atemporaraia

```sql
SELECT * FROM (SELECT city, temp_hi - temp_lo AS temp_var FROM weather) AS w
WHERE temp_var > 20;
```

### CTE

**O que é**:

*As stated above, a CTE (sometimes referred to as a “WITH statement”) is a temporary named result set that is not saved anywhere, it only exists in memory while the query is being run (unlike a temp table which exists in a temp DB file). The results of a CTE only exist within the execution scope of a specific statement; in other words, the results are only available to the CRUD statement immediately following the WITH clause in which the CTE is defined. CTEs are a great way to simplify and manage complex queries, making your code easier to read by breaking it into sections.*

**Resumo:** CTE é o mesmo que subquery, mas, é uma forma otimizada para certas situações. É uma consulta salva em tempo de execuçâo

**Usamos CTE a SubQuery quando ..**

+ Queremos que fique mais legível (a CTE vem antes da Query)
+ Quando chamamos uma mesma subquery mais de uma vez em lugares diferente.
  + Otimizaçâo: Se você cham 2 vezes uma cte, se trocar de CTE para subquery,, teria que executar a meama query 2 vezes
+ Pode-se usar CTE para chamadas recursivas

```sql
-- Exemplo de 2 CTE numa mesma query
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
       FROM money_summed # uso de CTE 1
       GROUP BY customer_id, date
    )

SELECT
   customer_id,
   money_available 
FROM money_available_calculated # uso de CTE 2
WHERE date = '2022-01-01' 
````

### Pivot Table

**O que é**: *Pivot tables are a piece of summarized information that is generated from a large underlying dataset. It is generally used to report on specific dimensions from the vast datasets. Essentially, the user can convert rows into columns. This gives the users the ability to transpose columns from a SQL Server table easily and create reports as per the requirements.*

![](https://www.sqlshack.com/wp-content/uploads/2020/04/pivot-table-example.png)

É como fazer a matrix tranposta, só que em tabelas.

No MySQL nõa há funçâo que faça isso, entao é mais trabalhoso, tendo que fazer de forma bem manual.

Exemplo: Temos um consulta que gera a seguinte tabela

**Como fazer**

+ Em geral, é a mesma consulta, mas você a torna uma subquery, e an query de fora você  seleciona com  `SUM` com `CASE ` sobre o `GROUP BY`

#### Exemplo Simples

```sql
SELECT teams.conference AS conference,
       players.year,
       COUNT(1) AS players
  FROM benn.college_football_players players
  JOIN benn.college_football_teams teams
    ON teams.school_name = players.school_name
 GROUP BY 1,2
 ORDER BY 1,2
```

|      |    conference     | year | players |
| :--: | :---------------: | :--: | ------- |
|  1   |        ACC        |  FR  | 607     |
|  2   |        ACC        |  JR  | 356     |
|  3   |        ACC        |  SO  | 341     |
|  4   |        ACC        |  SR  | 259     |
|  5   | American Athletic |  FR  | 418     |
|  6   | American Athletic |  JR  | 241     |
|  7   | American Athletic |  SO  | 247     |
|  8   | American Athletic |  SR  | 218     |
|  9   |      Big 12       |  FR  | 456     |
|  10  |      Big 12       |  JR  | 270     |
|  11  |      Big 12       |  SO  | 254     |
|  12  |      Big 12       |  SR  | 210     |

Queremos fazer uma Pivot Table nessa tabela para juntar os `players`, ter o total para cada `conference` e tudo isso está numa única linha

**CADA **

```sql
-- Pivot Table
SELECT conference,
       SUM(CASE WHEN year = 'FR' THEN players ELSE NULL END) AS fr,
       SUM(CASE WHEN year = 'SO' THEN players ELSE NULL END) AS so,
       SUM(CASE WHEN year = 'JR' THEN players ELSE NULL END) AS jr,
       SUM(CASE WHEN year = 'SR' THEN players ELSE NULL END) AS sr
  FROM (
        SELECT teams.conference AS conference,
               players.year,
               COUNT(1) AS players
          FROM benn.college_football_players players
          JOIN benn.college_football_teams teams
            ON teams.school_name = players.school_name
         GROUP BY 1,2
       ) sub
 GROUP BY 1
 ORDER BY 1
```

|      |   conference   | total_players |  fr  |  so  |  jr  | sr   |
| :--: | :------------: | :-----------: | :--: | :--: | :--: | ---- |
|  1   |      SEC       |     1650      | 659  | 362  | 368  | 261  |
|  2   |      ACC       |     1563      | 607  | 341  | 356  | 259  |
|  3   | Conference USA |     1495      | 519  | 324  | 351  | 301  |
|  4   |    Big Ten     |     1466      | 636  | 314  | 284  | 232  |

#### Pivot Reverse

Usa-se muito constante par afazelo

vamos converter a seguinte tabela

![https://mode.com/resources/images/common-problems/earthquake-table.png](https://mode.com/resources/images/common-problems/earthquake-table.png)

Para que cada uma das colunas que representam os anos se torne um registro. Para fazer, usa-se muito Hard Code.

```sql
 SELECT years.*,
           earthquakes.magnitude,
           CASE year
             WHEN 2000 THEN year_2000
             WHEN 2001 THEN year_2001
             WHEN 2002 THEN year_2002
             WHEN 2003 THEN year_2003
             WHEN 2004 THEN year_2004
             WHEN 2005 THEN year_2005
             WHEN 2006 THEN year_2006
             WHEN 2007 THEN year_2007
             WHEN 2008 THEN year_2008
             WHEN 2009 THEN year_2009
             WHEN 2010 THEN year_2010
             WHEN 2011 THEN year_2011
             WHEN 2012 THEN year_2012
             ELSE NULL END
             AS number_of_earthquakes
      FROM tutorial.worldwide_earthquakes earthquakes
     CROSS JOIN (
         SELECT year
         FROM (VALUES (2000),(2001),(2002),(2003),(2004),(2005),(2006),
               (2007),(2008),(2009),(2010),(2011),(2012)) v(year)
     ) years
```

|      | year | magnitude  | number_of_earthquakes |
| :--: | :--: | :--------: | --------------------- |
|  1   | 2000 | 8.0 to 9.9 | 1                     |
|  2   | 2001 | 8.0 to 9.9 | 1                     |
|  3   | 2002 | 8.0 to 9.9 | 0                     |
|  4   | 2003 | 8.0 to 9.9 | 1                     |
|  5   | 2004 | 8.0 to 9.9 | 2                     |

#### Outros Exemplos de pivot table

ex1

```sql
SELECT  P.`company_name`,
    COUNT(
        CASE 
            WHEN P.`action`='EMAIL' 
            THEN 1 
            ELSE NULL 
        END
    ) AS 'EMAIL',
    COUNT(
        CASE 
            WHEN P.`action`='PRINT' AND P.`pagecount` = '1' 
            THEN P.`pagecount` 
            ELSE NULL 
        END
    ) AS 'PRINT 1 pages',
    COUNT(
        CASE 
            WHEN P.`action`='PRINT' AND P.`pagecount` = '2' 
            THEN P.`pagecount` 
            ELSE NULL 
        END
    ) AS 'PRINT 2 pages',
    COUNT(
        CASE 
            WHEN P.`action`='PRINT' AND P.`pagecount` = '3' 
            THEN P.`pagecount` 
            ELSE NULL 
        END
    ) AS 'PRINT 3 pages'
FROM    test_pivot P
GROUP BY P.`company_name`;
```

ex2


```sql
select table_record_id,
    group_concat(if(value_name='note', value_text, NULL)) as note
    ,group_concat(if(value_name='hire_date', value_text, NULL)) as hire_date
    ,group_concat(if(value_name='termination_date', value_text, NULL)) as termination_date
    ,group_concat(if(value_name='department', value_text, NULL)) as department
    ,group_concat(if(value_name='reporting_to', value_text, NULL)) as reporting_to
    ,group_concat(if(value_name='shift_start_time', value_text, NULL)) as shift_start_time
    ,group_concat(if(value_name='shift_end_time', value_text, NULL)) as shift_end_time
from other_value
where table_name = 'employee'
and is_active = 'y'
and is_deleted = 'n'
GROUP BY table_record_id
```

ex3

```sql
SELECT
    company_name,  
    SUM(action = 'EMAIL')AS Email,
    SUM(action = 'PRINT' AND pagecount = 1)AS Print1Pages,
    SUM(action = 'PRINT' AND pagecount = 2)AS Print2Pages,
    SUM(action = 'PRINT' AND pagecount = 3)AS Print3Pages
FROM t
GROUP BY company_name
```

ex4

```
select id, 
	sum(case when month = 'jan' then revenue else null end) as Jan_Revenue,
	sum(case when month = 'feb' then revenue else null end) as Feb_Revenue,
	sum(case when month = 'mar' then revenue else null end) as Mar_Revenue,
	sum(case when month = 'apr' then revenue else null end) as Apr_Revenue,
	sum(case when month = 'may' then revenue else null end) as May_Revenue,
	sum(case when month = 'jun' then revenue else null end) as Jun_Revenue,
	sum(case when month = 'jul' then revenue else null end) as Jul_Revenue,
	sum(case when month = 'aug' then revenue else null end) as Aug_Revenue,
	sum(case when month = 'sep' then revenue else null end) as Sep_Revenue,
	sum(case when month = 'oct' then revenue else null end) as Oct_Revenue,
	sum(case when month = 'nov' then revenue else null end) as Nov_Revenue,
	sum(case when month = 'dec' then revenue else null end) as Dec_Revenue
from department
group by id
order by id
```



## Window Function

### CheatSheet Resume

![](G:\Personal Projects\TECH-DEV-STUDIES\sql-db-cloud-study\sql\my-notes\imgs\img-01.png)

**Window functions** perform calculations on a set of rows that are related together. But, unlike the aggregate functions, windowing functions do not collapse the result of the rows into a single value. Instead, all the rows maintain their original identity and the calculated result is returned for every row.

## NULL BEHAVIOR

https://github.com/shawlu95/Beyond-LeetCode-SQL/tree/master/Hacks/02_NULL_pathology

Se vocÊ vai fazer conta, ou alguma verificaçâo numa coluna que tem valor NULL, é necessário saber como vai se comportar

### Excluindo um valor pode excluir inclusive NULL

Exemplo

```sql
SELECT name FROM balance WHERE name != "Alice";
-- Não retorna valores null
```

Agora, se quisermos que alme de diferen de Alice retorne nULL, tem que colcoar

```sql
SELECT name FROM balance WHERE name != "Alice" OR name IS NULL;
```

### Aggregation Behavior

NULL é ignorado em n *SUM()*, *MAX()*, *MIN()*, *AVG()*.

**MAS** se tiver GORUP BY, ai nao sera

```
mysql> SELECT name, SUM(balance) FROM Balance GROUP BY name;
+-------+--------------+
| name  | SUM(balance) |
+-------+--------------+
| Alice |           10 |
| Bob   |           15 |
| NULL  |           20 |
| Cindy |          100 |
| Shaw  |         NULL |
+-------+--------------+
5 rows in set (0.00 sec)
```



### Ordering Behavior

Note that when in ASC order, NULL appears first. In DESC order, NULL appears last. Use *COALESCE()* if we want to place *NULL* at the end, while still have *ASC* order for the rest of the rows.

```
mysql> SELECT name, balance FROM Balance ORDER BY balance;
+-------+---------+
| name  | balance |
+-------+---------+
| Cindy |    NULL |
| Shaw  |    NULL |
| NULL  |    NULL |
| Bob   |       5 |
| Alice |      10 |
| Bob   |      10 |
| NULL  |      20 |
| Cindy |     100 |
+-------+---------+
8 rows in set (0.00 sec)

mysql> SELECT name, balance FROM Balance ORDER BY COALESCE(balance, 1E9);
+-------+---------+
| name  | balance |
+-------+---------+
| Bob   |       5 |
| Alice |      10 |
| Bob   |      10 |
| NULL  |      20 |
| Cindy |     100 |
| Cindy |    NULL |
| Shaw  |    NULL |
| NULL  |    NULL |
+-------+---------+
8 rows in set (0.00 sec)
```



In a GROUP BY clause, ordering is applied by default. See the following example: without using ORDER BY name, the results are already ordered by name.

```
mysql> SELECT name, SUM(balance) AS sum_balance FROM Balance GROUP BY name;
+-------+-------------+
| name  | sum_balance |
+-------+-------------+
| Alice |          10 |
| Bob   |          15 |
| NULL  |          20 |
| Cindy |         100 |
| Shaw  |        NULL |
+-------+-------------+
5 rows in set (0.00 sec)

mysql> SELECT name, SUM(balance) AS sum_balance FROM Balance GROUP BY name ORDER BY sum_balance;
+-------+-------------+
| name  | sum_balance |
+-------+-------------+
| Shaw  |        NULL |
| Alice |          10 |
| Bob   |          15 |
| NULL  |          20 |
| Cindy |         100 |
+-------+-------------+
5 rows in set (0.00 sec)
```

### Em condicional

```
True and NULL returns NULL
True or NULL returns True
False and NULL returns False
False or NULL returns NULL
NULL = 0 returns NULL
NULL != 12 returns NULL
NULL +3 returns NULL
NULL || 'str' returns NULL
NULL = NULL returns NULL
NULL != NULL returns NULL
```

Use `IS NULL` ou `IS NOT NULL`

## Otimização

### Usar `GROUP BY` ao invés de `DISTINCT`

**distinct ém uito custoso. uMA FORMA DE VOCÊ TIRAR DUPLICATAS DE FORMA OTIMIZADA É ATRAVEZ DO `GROUP BY`**

**GROUP BY É MAIS RÁPIDO QUE DISTINCT**

```sql
SELECT
    distinct author_id as id
FROM views
WHERE author_id = viewer_id
ORDER BY author_id;
```

Outro solução mais rápida

```sql
select author_id as id 
from Views 
where author_id = viewer_id
group by 1
order by 1;
```

### From Git - Query Optimization

https://github.com/shawlu95/Beyond-LeetCode-SQL

![alt-text](https://github.com/shawlu95/Beyond-LeetCode-SQL/raw/master/assets/sql_order.png)

* Place smaller table first when joining multiple tables
* Largest table is the base table
  - base table is placed on right hand side of equal sign (where clause)
* Place most restrictive condition **last**:
  - The condition in the WHERE clause of a statement that returns the fewest rows of data
  - the most restrictive condition was listed last in the WHERE clause,
* try to use indexed column

```SQL
FROM TABLE1,   -- Smallest table
     TABLE2,   -- to
     TABLE3    -- Largest table, also base table
WHERE TABLE1.COLUMN = TABLE3.COLUMN    -- Join condition
  AND TABLE2.COLUMN = TABLE3.COLUMN    -- Join condition
[ AND CONDITION1 ]                     -- Filter condition
[ AND CONDITION2 ]                     -- Filter condition
```

* Using the `like` operator and wildcards (flexible search)
* Avoiding the `or` operator, use `in` operator
  - data retrieval is measurably faster by replac- ing OR conditions with the IN predicate
* Avoiding the `HAVING` clause
  - try to frame the restriction earlier (`where` clause)
  - try to keep `HAVING` clause simple (use constant, not function)
* avoiding large sort operations
  - it is best to schedule queries with large sorts as periodic batch processes during off-peak database usage so that the performance of most user processes is not affected.
* Prefer stored procedure
  - compiled and permanently stored in the database in an executable format.
* Disabling indexes during batch loads
  - When the batch load is complete, you should rebuild the indexes.
  - reduction of fragmentation that is found in the index
* cost-based optimization: check database server manual
* Using view: keep the levels of code in your query as flat as possible and to test and tune the statements that make up your views

