# Windows Function

## Links

https://portosql.wordpress.com/2018/10/14/funcoes-de-janela-window-functions/



## O que é

As funções de janela (*window functions*) auxiliam de forma intuitiva na resolução de uma variedade de tarefas. Neste contexto, a janela se refere a um conjunto de linhas cujo conteúdo é definido na cláusula OVER.

Boa defniçâo

**Window functions** perform calculations on a set of rows that are related together. But, unlike the aggregate functions, windowing functions do not collapse the result of the rows into a single value. Instead, all the rows maintain their original identity and the calculated result is returned for every row.

## Quando usamos

https://www.analyticsvidhya.com/blog/2020/12/window-function-a-must-know-sql-concept/

```
SQL aggregate functions only give us a single value for the group of rows aggregated together (think of the first query we wrote).

But the latter queries couldn’t simply be solved using such functions. Those queries want us to maintain the original identity of the individual rows, something that the aggregate functions fail to address. Therefore, to solve such queries we need different kinds of functions – the Window functions.
```

**Ou seja, usamos quando queremos algo parecido com o `GROUP BY` mas que seja aplciada a toda a row, sem reduzir por agregação **

**Observe a diferneça**

![](https://miro.medium.com/max/700/1*bWw3tHCsXRHyOKOkm81eUg.png)

**Why use Window Functions?**

One major advantage of window functions is that it allows you to work with both aggregate and non-aggregate values all at once because the rows are not collapsed together.

**Ordem da Windows Function**

![img](https://miro.medium.com/max/700/1*MCyFDF35OZ3n0dyZqX6_Tw.png)

This is important because based off of this logical order, window functions are allowed in `SELECT` and `ORDER BY,` but they are not allowed in `FROM`, `WHERE`, `GROUP BY`, or `HAVING` clauses

# Artigos



## Artigo 2

https://towardsdatascience.com/5-window-function-examples-to-take-your-sql-skills-to-the-next-level-2b3306650bb6

Exemplo para testar no site: https://www.sql-practice.com/

Usando `windows function`

```sql
select
   p.city,
   p.weight,
   max(weight) over (partition by city) as maxwt_by_city

from patients p
ORDER BY p.city
```

Não usando `windows function`

```sql
-- Maior pesoa para cada cidade
with maxwt_groupby_base as (
select
    city
  , max(weight) as maxwt_by_city
 
from patients
group by
  city
)

-- unindo a base anterior com inerjoin a base
, maxwt_groupby as (
select
   p.city,
   p.weight,
   m.maxwt_by_city
  
from patients p
INNER JOIN maxwt_groupby_base m
  on p.city = m.city
)

-- selecionando a sida 
-- (essa parte nem precisa, poderia por acima)
select * from maxwt_groupby
ORDER BY city
```

**O que diferencia**

+ Agente quer por no resultado da query o maior peso da cidade em relaçâo a cada paciente
+ Sem windos funcion você tem que usa CTE para pegar esses addos

OutPut:

 The output will give you the same number of rows and your final column will be a max weight by city. This means that if a city appears more than once, then the weight value will be duplicated. This duplication happens frequently with window functions since we are performing an aggregation without collapsing any rows in the table.



| city | weight | maxwt_groupby |
| ---- | ------ | ------------- |
| Ajax | 74     | 106           |
| Ajax | 69     | 106           |
| Ajax | 76     | 106           |
| Ajax | 100    | 106           |



**Funçôes de agregação**

- [avg(X)](https://www.sqlite.org/lang_aggfunc.html#avg)
- [count(*)](https://www.sqlite.org/lang_aggfunc.html#count)
- [count(X)](https://www.sqlite.org/lang_aggfunc.html#count)
- [group_concat(X)](https://www.sqlite.org/lang_aggfunc.html#group_concat)
- [group_concat(X,Y)](https://www.sqlite.org/lang_aggfunc.html#group_concat)
- [max(X)](https://www.sqlite.org/lang_aggfunc.html#max_agg)
- [min(X)](https://www.sqlite.org/lang_aggfunc.html#min_agg)
- [sum(X)](https://www.sqlite.org/lang_aggfunc.html#sumunc)
- [total(X)](https://www.sqlite.org/lang_aggfunc.html#sumunc)

## Artigo 3

https://www.analyticsvidhya.com/blog/2020/12/window-function-a-must-know-sql-concept/

#### What are Window Functions in SQL?

**Window functions** perform calculations on a set of rows that are related together. But, unlike the aggregate functions, windowing functions do not collapse the result of the rows into a single value. Instead, all the rows maintain their original identity and the calculated result is returned for every row.

The **OVER** clause signifies a window of rows over which a window function is applied. It can be used with aggregate functions, like we have used with the SUM function here, thereby turning it into a window function. Or, it can also be used with non-aggregate functions that are only used as window functions (we will learn more about them in the later sections).

#### Clauselue Over

Exemplo

![](https://cdn.analyticsvidhya.com/wp-content/uploads/2020/11/over-clause-sql.png)

Observe que esse somatório é fácil de obter, é um SUM comun. **A windows fuinction permite usar as funçôes de agregação e aplicar em todas as rows ao invés de faer um redux**

#### Clausule Partition

For example, to display the total salary per job category for all the rows we would have to modify our original SQL query as follows:

![](https://cdn.analyticsvidhya.com/wp-content/uploads/2020/11/partition-by-sql.png)

**O PARTITION aplica a agregçaâo num group by, e mais uma vez, nao faz redux**

Nesse caso, vai pegar a somatoria para cada categoria. É um group by da agregaçâo 

## Artigo 4

# **List of Window Functions**

https://towardsdatascience.com/a-guide-to-advanced-sql-window-functions-f63f2642cbf9

Now that you know the syntax, let’s take look at the different kinds of window functions that can be substituted in place of the red font below.

![img](https://miro.medium.com/max/700/1*NV9bCjSJTKXQ8_-X-aHjVg.png)

Image by Author

There are three main types of window functions available to use: aggregate, ranking, and value functions. In the image below, you can see some of the names of the functions that fall within each group.

![img](https://miro.medium.com/max/700/1*u8SPxze2b1OdCdPo1maVGw.png)

Image by Author

Here’s a quick overview of what each type of window function is useful for.

**Aggregate functions:** we can use these functions to calculate various aggregations such as average, total # of rows, maximum or minimum values, or total sum within each window or partition.

**Ranking functions:** these functions are useful for ranking rows within its partition.

**Value functions:** these functions allow you to compare values from previous or following rows within the partition or the first or last value within the partition.