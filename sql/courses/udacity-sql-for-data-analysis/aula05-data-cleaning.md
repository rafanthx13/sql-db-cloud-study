# Aula 5 - SQL Data Cleaning

In this lesson, you will be learning a number of techniques to

1. Clean and re-structure messy data.
2. Convert columns to different data types.
3. Tricks for manipulating **NULL**s.

This will give you a robust toolkit to get from raw data to clean data that's useful for analysis.



## Funções para slice string

1. **LEFT**
   + Pega as N primeiros char da string
2. **RIGHT**
   + Pega os últimos N char da stirng (é o LEDT de traz pra frente)
3. **LENGTH**
   + Retorna o tamanho de chars da string

**Exemplo de uso**

```
SELECT SUM(vowels) vowels, SUM(other) other
FROM (SELECT name, CASE WHEN LEFT(UPPER(name), 1) IN ('A','E','I','O','U') 
                        THEN 1 ELSE 0 END AS vowels, 
          CASE WHEN LEFT(UPPER(name), 1) IN ('A','E','I','O','U') 
                       THEN 0 ELSE 1 END AS other
         FROM accounts) t1;
```



1. **POSITION **e **STRPOS**

   + Retorna o undex de onde esta um char
   + **POSITION** takes a character and a column, and provides the index where that character is for each row. The index of the first position is 1 in SQL. If you come from another programming language, many begin indexing at 0. Here, you saw that you can pull the index of a comma as **POSITION(',' IN city_state)**.

   + **STRPOS** provides the same result as **POSITION**, but the syntax for achieving those results is a bit different as shown here: **STRPOS(city_state, ',')**.
   + Note, both **POSITION** and **STRPOS** are case sensitive, so looking for **A** is different than looking for **a**.

2. **LOWER** e **UPPER**:

   + Maiusculo e minusculo o text

**Exemplo**

```
SELECT LEFT(name, STRPOS(name, ' ') -1 ) first_name, 
       RIGHT(name, LENGTH(name) - STRPOS(name, ' ')) last_name
FROM sales_reps;
```

### Concat

Junta strings. Outra forma de fazer é usando pip `||`

**CONCAT(first_name, ' ', last_name)** or with piping as **first_name || ' ' || last_name**.

Exemplo

```
WITH t1 AS (
 SELECT LEFT(primary_poc,     STRPOS(primary_poc, ' ') -1 ) first_name,  RIGHT(primary_poc, LENGTH(primary_poc) - STRPOS(primary_poc, ' ')) last_name, name
 FROM accounts)
SELECT first_name, last_name, CONCAT(first_name, '.', last_name, '@', REPLACE(name, ' ', ''), '.com')
FROM  t1;
```

### CAST

Converter tipos no SQL

```
SELECT CAST("2017-08-29" AS DATE);
SELECT CAST(150 AS CHAR);
SELECT CAST("14:06:10" AS TIME);
CAST ('10.2' AS DOUBLE);
CAST('true' AS BOOLEAN),
   CAST('false' as BOOLEAN),
```

### COASLECE

**COALESCE** returns the first non-NULL value passed for each row. Hence why the video used **no_poc** if the value in the row was NULL.

**Resuni**: Se for nulo substitui por outro valor, é um replace quando nnan em python

https://www.w3schools.com/sql/sql_isnull.asp

No mysql há outra funão que faz a mesma coisa `IFNULL()`