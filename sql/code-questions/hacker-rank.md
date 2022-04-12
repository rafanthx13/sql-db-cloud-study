# HackerRank - Questions

## Links

+ https://github.com/marinskiy/HackerrankPractice 

## Questões HakcerRank

https://www.hackerrank.com/challenges/revising-the-select-query-2/

```sql
SELECT NAME
FROM CITY
WHERE POPULATION >= 120000 AND COUNTRYCODE = 'USA';
```


https://www.hackerrank.com/challenges/binary-search-tree-1/problem

```sql
-- questao de arvore binaria:
SELECT N,
    CASE
        -- P = o pai
        WHEN P IS NULL THEN 'Root'
        -- N = Valor do node
        WHEN N IN (SELECT P FROM BST) THEN 'Inner'
        ELSE 'Leaf'
    END
FROM BST
ORDER BY N;
```

================
https://www.hackerrank.com/challenges/the-company/problem

```sql
select 
   C.company_code, 
   C.founder,
   count(distinct L.lead_manager_code),
   count(distinct S.senior_manager_code),
   count(distinct M.manager_code),
   count(distinct E.employee_code)
from Company as C,
     Lead_Manager as L,
     Senior_Manager as S,
     Manager as M,
     Employee as E 
WHERE E.manager_code = M.manager_code 
      AND M.senior_manager_code = S.senior_manager_code
      AND L.lead_manager_code = S.lead_manager_code
      AND C.company_code = L.company_code
group by 
    C.company_code, C.founder
order by 
    C.company_code
```

​                            





## Análise de Algumas respostas

saber se é par (even) ou impar (odd)

### Detectar par ou impar

```SQL
SELECT *
FROM dept_manager
WHERE MOD(emp_id, 2) = 0; -- par
```

=======================================

### Diferença de valores único e duplicados

```sql
SELECT COUNT(CITY) - COUNT(DISTINCT(CITY)) FROM STATION;
```

Será > 0 se houver houver city iguais (ou seja, duplicadas)

=======================================

### Usar `UNION`

Ao usar UNION, coloque entre paranteses, sem parentese nao funfa

```sql
(select city, length(city) from station
order by length(city), city asc
limit 1)
UNION
(select city, length(city) from station
order by length(city) desc
limit 1);
```

### Regex em SQL

https://github.com/marinskiy/HackerrankPractice [soluçao de problemas sql e outros]

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



### ordena pelos 3 ultimos caracters do nome**

```sql
SELECT Name
FROM Students
WHERE Marks > 75
ORDER BY SUBSTR(Name, - 3), Id ASC;
-- ordena pelos 3 ultimos caracters do nome
-- perceba que SUBSTR é um split, que, se negativo, epga de tras pra fenrete como em python
```



### O DEFAULT de `ORDER` é ser `ASC`

```sql
SELECT Name
FROM Employee
ORDER BY Name; -- o mesmo que ORDER BY NAME ASC
```

### Uso de funções de agregação *SEM* `groupby`

```sql
SELECT FLOOR(AVG(Population))
FROM City;

-- FLOR É O INTEIRO MAIS BAIOX; avg voce pode usar sem group by, apenas com os registro que vinhere

SELECT MAX(Population) - MIN(Population)
FROM City;
```

max, min, avg, count podem ser usadaos diretos na coluna, de forma direta

```sql
SELECT ROUND(SUM(Lat_N), 2), ROUND(SUM(Long_W), 2)
FROM Station;
```

mais um caso. nao sei prque mas par mimn essa funçoes de'redeceu' so se encaixam com group by.

mas na verdade no group by elas sao feitas várias vezes, quando qgente que uma unica row, nao precisa

### Análise complexa

```sql
-- O que foi feito
-- quer o maior produtorio, e quanto tiveram esse maior produtorio
-- para pegar quantos tiveram, entao usamos group by no produtorio, ordenamos ao invez para nao precisar do max (pode ser que funcione tabem)
-- por fim lmiitamos em 1, como o 1 vai ser o maior, pega entao uma linha resolvendo a questâo
SELECT *
FROM (
    SELECT Months * Salary, COUNT(*)
    FROM Employee
    GROUP BY (Months * Salary)
    ORDER BY Months * Salary DESC
    ) AS X
LIMIT 1;
```

Além de mostarr esse produtório, vai mostrar quantos chegaram com esse valor

### Mediana em SQL

**Oracle**

```sql
-- isso funfa no orcale mas nao no mysql
SELECT ROUND(MEDIAN(Lat_N), 4)
FROM Station;
```

É possivel fazer median no sql; isso é o mesmo no mysql: criando mediana

**MySql**

```sql
Select round(S.LAT_N,4) mediam 
from station S 
where (
    select count(Lat_N) from station where Lat_N < S.LAT_N 
) = (
    select count(Lat_N) from station where Lat_N > S.LAT_N
)
```

