# Ultra Review de SQL

Por todas as minhas anotações



## Conceitos Avançados

### CTE

**O que é**:

*As stated above, a CTE (sometimes referred to as a “WITH statement”) is a temporary named result set that is not saved anywhere, it only exists in memory while the query is being run (unlike a temp table which exists in a temp DB file). The results of a CTE only exist within the execution scope of a specific statement; in other words, the results are only available to the CRUD statement immediately following the WITH clause in which the CTE is defined. CTEs are a great way to simplify and manage complex queries, making your code easier to read by breaking it into sections.*

**Resumo:** CTE é o mesmo que subquery, mas, é uma forma otimizada para certas situações. É uma consulta salva em tempo de execuçâo

**Usamos CTE a SubQuery quando ..**

+ Queremos que fique mais legível (a CTE vem antes da Query)
+ Quando chamamos uma mesma subquery mais de uma vez em lugares diferente.
  + Otimizaçâo: Se você cham 2 vezes uma cte, se trocar de CTE para subquery,, teria que executar a meama query 2 vezes
+ Pode-se usar CTE para chamadas recursivas



~~~sql
# Exemplod e 2 CTE
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

~~~

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
# Pivot Table
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

## Window Function

![](G:\Personal Projects\TECH-DEV-STUDIES\sql-db-cloud-study\sql\my-notes\imgs\img-01.png)

**Window functions** perform calculations on a set of rows that are related together. But, unlike the aggregate functions, windowing functions do not collapse the result of the rows into a single value. Instead, all the rows maintain their original identity and the calculated result is returned for every row.