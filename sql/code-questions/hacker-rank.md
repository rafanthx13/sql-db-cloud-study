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

Outra forma

````SQL
employee_id % 2 = 0
````

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

#### Hard: `CASE .. WHEN .. THEN .. ELSE` Tipo de tringulo

Tem que usar case => when e then para cada caso, e caso nao for nenhum deles, else

https://www.hackerrank.com/challenges/what-type-of-triangle/problem?isFullScreen=true

```
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

**Outro caso: leetCode**

É possível usar CASE em Update

````sql
UPDATE salary
SET
    sex = CASE sex
        WHEN 'm' THEN 'f'
        ELSE 'm'
    END;
````

#### hard: `CONCAT `

[Questão](https://www.hackerrank.com/challenges/the-pads/)

 [GitSolution - 998 The PADS](https://github.com/mremreozan/SQL-BigQuery/blob/master/Hackerrank/998 The PADS)

Generate the following two result sets:

1. Query an *alphabetically ordered* list of all names in **OCCUPATIONS**, immediately followed by the first letter of each profession as a parenthetical (i.e.: enclosed in parentheses). For example: `AnActorName(A)`, `ADoctorName(D)`, `AProfessorName(P)`, and `ASingerName(S)`.

2. Query the number of ocurrences of each occupation in **OCCUPATIONS**. Sort the occurrences in *ascending order*, and output them in the following format:

   ```
   There are a total of [occupation_count] [occupation]s.
   ```

```
SELECT
    -- Format (nome(primeira_letra_de_occupation))
    CONCAT(NAME,
           CONCAT(
               "(",   
               CONCAT(substr(OCCUPATION,1,1),
               ")" ) 
           )
    ) 
FROM OCCUPATIONS
ORDER BY NAME ASC;

-- 2ª Query (nesse editor, pode ter 2 queries diferentes, mas no mysql normal não pode)
SELECT
    "There are a total of ",
    COUNT(OCCUPATION),
    CONCAT(LOWER(occupation),"s.") 
FROM OCCUPATIONS 
GROUP BY OCCUPATION 
ORDER BY COUNT(OCCUPATION) ASC, OCCUPATION;

```

#### Hard: Variáveis + PivotTable

https://www.hackerrank.com/challenges/occupations/

Essa questâo é muito difícil. Só consegui vendo a solução dela

**Qual é o objetivo**

O objetivo é gerar a seguinte tabela mas: **Deixar todos os valores validos agrupados no topo em ordem alfabetica**.

Por isso é necessário saber: Qual é a coluna com mais valores válidos

![](https://i.imgur.com/u6DEcNQ.jpeg)

**O que torna difícil:** é onde por `NULL`, perceba que `NULL` está em todas as colunas exceto em uma única que tem algum valor

**Exemplo de saida**

```
Aamina Ashley   Christeen Eve
Julia  Belvet   Jane      Jennifer
Priya  Britney  Jenny     Ketty
NULL   Maria    Kristeen  Samantha
NULL   Meera    NULL      NULL
NULL   Naomi    NULL      NULL
NULL   Priyanka NULL      NULL
```

**Como foi feito**

A ieia é fazer um pivot table sobre uma coluna

1. Os valores da coluna occupation vão virar atributos da nova tabela, no caso: `Doctor, Professor, Singer e Actor`

   ```
   SELECT
       case when Occupation='Doctor' 
       	then Name end as Doctor,
       case when Occupation='Professor' 
       	then Name end as Professor,
       case when Occupation='Singer' 
       	then Name end as Singer,
       case when Occupation='Actor' 
       	then Name end as Actor
   FROM OCCUPATIONS
   ```

   

2. Vamos precisar de um contador para salvar cada um:

   ![](https://i.imgur.com/QzVCWFn.jpeg)

​		Fazemos isso com as variáveis

```
set @r1=0, @r2=0, @r3=0, @r4=0;

SELECT case 
		when Occupation='Doctor' then (@r1:=@r1+1)
        when Occupation='Professor' then (@r2:=@r2+1)
        when Occupation='Singer' then (@r3:=@r3+1)
        when Occupation='Actor' then (@r4:=@r4+1) 
end as RowNumber
-- rownumber é apenas um nome provisorio para a coluna
FROM OCCUPATIONS

```

A medida que vai aparecendo cada um das 4 categorias agente altera o caontador

... ((continua mas desiti de entender))



**Solução**

+ A coluna `RowNumber` no final vai ter de 1 até N, onde N será a contagem da coluna com mais valores.
+ Temos 4 case para as 4 colunas. Se o `CASE` NÃO DA O `math` ENTÃO FICA `null`



```sql
-- SOLUCAO FINAL
set @r1=0, @r2=0, @r3=0, @r4=0;

select
	min(Doctor), min(Professor), min(Singer), min(Actor)
from
(
  select case 
    when Occupation='Doctor' then (@r1:=@r1+1)
    when Occupation='Professor' then (@r2:=@r2+1)
    when Occupation='Singer' then (@r3:=@r3+1)
    when Occupation='Actor' then (@r4:=@r4+1) 
  end as RowNumber,
    case when Occupation='Doctor' 
    	then Name end as Doctor,
    case when Occupation='Professor' 
    	then Name end as Professor,
    case when Occupation='Singer' 
    	then Name end as Singer,
    case when Occupation='Actor' 
    	then Name end as Actor
  from OCCUPATIONS
  order by Name
) Temp
group by RowNumber
```



**Saida da tabela temporaria**

```sql
-- Executando o seguinte codigo
select case 
    when Occupation='Doctor' then (@r1:=@r1+1)
    when Occupation='Professor' then (@r2:=@r2+1)
    when Occupation='Singer' then (@r3:=@r3+1)
    when Occupation='Actor' then (@r4:=@r4+1) 
  end as RowNumber,
    case when Occupation='Doctor' 
        then Name end as Doctor,
    case when Occupation='Professor' 
        then Name end as Professor,
    case when Occupation='Singer' 
        then Name end as Singer,
    case when Occupation='Actor' 
        then Name end as Actor
  from OCCUPATIONS
  order by Name
```

Saida

```
NULL Aamina NULL NULL NULL
NULL NULL Ashley NULL NULL
NULL NULL Belvet NULL NULL
NULL NULL Britney NULL NULL
NULL NULL NULL Christeen NULL
NULL NULL NULL NULL Eve
NULL NULL NULL Jane NULL
NULL NULL NULL NULL Jennifer
NULL NULL NULL Jenny NULL
NULL Julia NULL NULL NULL
NULL NULL NULL NULL Ketty
NULL NULL NULL Kristeen NULL
NULL NULL Maria NULL NULL
NULL NULL Meera NULL NULL
NULL NULL Naomi NULL NULL
NULL Priya NULL NULL NULL
NULL NULL Priyanka NULL NULL
NULL NULL NULL NULL Samantha
```

**Funcionamento do RowNumber fora da tabela interna**

Ele vai mostra o valor de @r que tava lá

O cosigo SQL sâo os 2 com:

+ nome mudado || serm orderby, msontrado tudo

```
set @r1=0, @r2=0, @r3=0, @r4=0;

select
    alfa, Doctor, Professor, Singer, Actor
from
(
  select case 
    when Occupation='Doctor' then (@r1:=@r1+1)
    when Occupation='Professor' then (@r2:=@r2+1)
    when Occupation='Singer' then (@r3:=@r3+1)
    when Occupation='Actor' then (@r4:=@r4+1) 
  end as alfa,
    case when Occupation='Doctor' 
        then Name end as Doctor,
    case when Occupation='Professor' 
        then Name end as Professor,
    case when Occupation='Singer' 
        then Name end as Singer,
    case when Occupation='Actor' 
        then Name end as Actor
  from OCCUPATIONS
  order by Name
) Temp
```

Saida

```
1 Aamina NULL NULL NULL
1 NULL Ashley NULL NULL
2 NULL Belvet NULL NULL
3 NULL Britney NULL NULL
1 NULL NULL Christeen NULL
1 NULL NULL NULL Eve
2 NULL NULL Jane NULL
2 NULL NULL NULL Jennifer
3 NULL NULL Jenny NULL
2 Julia NULL NULL NULL
3 NULL NULL NULL Ketty
4 NULL NULL Kristeen NULL
4 NULL Maria NULL NULL
5 NULL Meera NULL NULL
6 NULL Naomi NULL NULL
3 Priya NULL NULL NULL
7 NULL Priyanka NULL NULL
4 NULL NULL NULL Samantha
```

#### Easy

https://www.hackerrank.com/challenges/average-population-of-each-continent/

```
SELECT
    COUNTRY.Continent,
    FLOOR(AVG(CITY.POPULATION))
FROM CITY
INNER JOIN COUNTRY
ON CITY.COUNTRYCODE = COUNTRY.CODE
GROUP BY COUNTRY.Continent;

```

### Medium

https://www.hackerrank.com/challenges/the-report/

Obs:

​	+  nao tem como fazer um join de chave, entao filtra no where, assim o registro de 'grades' vai combinar com a sua respctiva nota

```sql
SELECT 
    IF(GRADE < 8, NULL, NAME),
    GRADE, 
    MARKS
FROM STUDENTS 
    JOIN GRADES
WHERE -- filter table GRADES
    MARKS BETWEEN MIN_MARK AND MAX_MARK
ORDER BY 
    GRADE DESC, NAME
```

O que tem em students

```
19 Samantha 87
21 Julia 96
11 Britney 95
32 Kristeen 100
12 Dyana 55
13 Jenny 66
14 Christene 88
15 Meera 24
16 Priya 76
17 Priyanka 77
18 Paige 74
19 Jane 64
21 Belvet 78
31 Scarlet 80
41 Salma 81
51 Amanda 34
61 Heraldo 94
71 Stuart 99
81 Aamina 77
76 Amina 89
91 Vivek 84
```

o que tem em grades

```
1 0 9
2 10 19
3 20 29
4 30 39
5 40 49
6 50 59
7 60 69
8 70 79
9 80 89
10 90 100
```

**Resultado da repostas correta**

```
Britney 10 95
Heraldo 10 94
Julia 10 96
Kristeen 10 100
Stuart 10 99
Amina 9 89
Christene 9 88
Salma 9 81
Samantha 9 87
Scarlet 9 80
Vivek 9 84
Aamina 8 77
Belvet 8 78
Paige 8 74
Priya 8 76
Priyanka 8 77
NULL 7 64
NULL 7 66
NULL 6 55
NULL 4 34
NULL 3 24
```

#### MEDIUM: Complexa

QUESTION: Julia just finished conducting a coding contest, and she needs your help assembling the leaderboard! Write a query to print the respective *hacker_id* and *name* of hackers who achieved full scores for *more than one* challenge. Order your output in descending order by the total number of challenges in which the hacker earned a full score. If more than one hacker received full scores in same number of challenges, then sort them by ascending *hacker_id*.

```
SELECT
    h.hacker_id,
    h.name
-- junta tudo
FROM
    submissions s
    inner join challenges c
        on s.challenge_id = c.challenge_id
    inner join difficulty d
        on c.difficulty_level = d.difficulty_level 
    inner join hackers h
        on s.hacker_id = h.hacker_id
where 
    -- acertou full score
    s.score = d.score and 
    -- acertou da full-score da sua respectiva dificuldade, 
    c.difficulty_level = d.difficulty_level 
group by 
    h.hacker_id, h.name
-- quem acertou ao menos 1 (quem encaixou no min 1 no WHERE)
having 
    count(s.hacker_id) > 1
-- Ordena pela contagem de full score
-- se tiverem iguais, ordena pelo IS
order by 
    count(s.hacker_id) desc, s.hacker_id asc;


```

#### MEDIUM: Subquerie para cada row de um select global

https://www.hackerrank.com/challenges/harry-potter-and-wands/

Primeira tentativa

```
select
    wands.id,
    Wands_Property.age,
    wands.coins_needed,
    wands.power
from wands
    inner join Wands_Property
        on wands.code = Wands_Property.code
where
    wands_property.is_evil = 0
order by 
    wands.power asc, Wands_Property.age desc
```

Acontece que falta uma coisa:

=> **QUAL O PREÇO MINIMO PARA A TUPLA: IDADE E POWER**

Para resolver isso temos que:

**FAZER UMA SUBQUERIES USANDO PARAMETROS GLOBAIS**

```
-- subquerie usando parametro globais
select min(coins_needed) 
from 
	Wands as w1 
join Wands_Property as p1 on (w1.code = p1.code) 				where 
-- Aqui esta os parametros globais 
-- w1 (sub) matn com w (global)
w1.power = w.power and p1.age = p.age
```

Ou seja, para cada varinha, temos que fazer uma query select para descobrir se ela é aquela com menor preço para seu respectivo power, age

**Soluçâo final**

```
select 
    w.id, p.age, w.coins_needed, w.power 
from Wands as w 
    join Wands_Property as p on (w.code = p.code)
where 
    p.is_evil = 0 and w.coins_needed = (
    select min(coins_needed) 
    from 
    	Wands as w1 
    	join Wands_Property as p1 on (w1.code = p1.code) 		where 
    	w1.power = w.power and p1.age = p.age) order by w.power desc, p.age desc
```

#### HARD: Uso pesado de subqueries em `Having`



https://www.hackerrank.com/challenges/challenges/

```SQL
/* these are the columns we want to output */
select c.hacker_id, h.name ,count(c.hacker_id) as c_count

/* this is the join we want to output them from */
from Hackers as h
    inner join Challenges as c on c.hacker_id = h.hacker_id

/* after they have been grouped by hacker */
group by c.hacker_id

/* but we want to be selective about which hackers we output */
/* having is required (instead of where) for filtering on groups */
having 

    /* output anyone with a count that is equal to... */
    c_count = 
        /* the max count that anyone has */
        (SELECT MAX(temp1.cnt)
        from (SELECT COUNT(hacker_id) as cnt
             from Challenges
             group by hacker_id
             order by hacker_id) temp1)

    /* or anyone who's count is in... */
    or c_count in 
        /* the set of counts... */
        (select t.cnt
         from (select count(*) as cnt 
               from challenges
               group by hacker_id) t
         /* who's group of counts... */
         group by t.cnt
         /* has only one element */
         having count(t.cnt) = 1)

/* finally, the order the rows should be output */
order by c_count DESC, c.hacker_id

/* ;) */
;
```

### REPEAT, VARIABEL Printar asterisco desenho SQL

```
SELECT 
    REPEAT('* ', @NUMBER := @NUMBER - 1) 
FROM 
    information_schema.tables, (SELECT @NUMBER:=21) t
-- Limit faz parar, senao é infinito
LIMIT 20
```

SAIDA

```
* * * * * * * * * * * * * * * * * * * * 
* * * * * * * * * * * * * * * * * * * 
* * * * * * * * * * * * * * * * * * 
* * * * * * * * * * * * * * * * * 
* * * * * * * * * * * * * * * * 
* * * * * * * * * * * * * * * 
* * * * * * * * * * * * * * 
* * * * * * * * * * * * * 
* * * * * * * * * * * * 
* * * * * * * * * * * 
* * * * * * * * * * 
* * * * * * * * * 
* * * * * * * * 
* * * * * * * 
* * * * * * 
* * * * * 
* * * * 
* * * 
* * 
* 
```

o INVERSO

```
SELECT 
    REPEAT('* ', @NUMBER := @NUMBER + 1) 
FROM 
    information_schema.tables, (SELECT @NUMBER:=0) t 
LIMIT 20
```

#### Printar numero primo de 1 a 1000

```
SELECT GROUP_CONCAT(NUMB SEPARATOR '&')
        FROM (SELECT @num:=@num+1 as NUMB 
                FROM information_schema.tables t1,
                     information_schema.tables t2,
                     (SELECT @num:=1) tmp
             ) tempNum
        WHERE NUMB<=1000 AND NOT EXISTS 
            (SELECT * FROM 
                (SELECT @nu:=@nu+1 as NUMA FROM
                    information_schema.tables t1,
                    information_schema.tables t2,
                    (SELECT @nu:=1) tmp1
                    LIMIT 1000
                ) tatata
                WHERE FLOOR(NUMB/NUMA)=(NUMB/NUMA) AND NUMA<NUMB AND NUMA>1
             )
             
```

saida

```
2&3&5&7&11&13&17&19&23&29&31&37&41&43&47&53&59&61&67&71&73&79&83&89&97&101&103&107&109&113&127&131&137&139&149&151&157&163&167&173&179&181&191&193&197&199&211&223&227&229&233&239&241&251&257&263&269&271&277&281&283&293&307&311&313&317&331&337&347&349&353&359&367&373&379&383&389&397&401&409&419&421&431&433&439&443&449&457&461&463&467&479&487&491&499&503&509&521&523&541&547&557&563&569&571&577&587&593&599&601&607&613&617&619&631&641&643&647&653&659&661&673&677&683&691&701&709&719&727&733&739&743&751&757&761&769&773&787&797&809&811&821&823&827&829&839&853&857&859&863&877&881&883&887&907&911&919&929&937&941&947&953&967&971&977&983&991&997
```



### MEDIUM: innerjoin em uma subquerie

https://www.hackerrank.com/challenges/contest-leaderboard/

Porque é dificil?

Olhe a tabela submissions: a somatoria que você quer é a soma das maiores notas para cada challenge. Acontece que para um chalenge pode ter mais de uma submissâo. Entâo você tem que filtrar **a maior nota por challenge ID, e é isso que é feito na subquerie**

```
select 
    h.hacker_id, name, sum(score) as total_score
from
    hackers as h 
    /* find max_score by each challenge_id*/
    inner join (
        select 
            hacker_id,  
            max(score) as score 
        from submissions 
        group by challenge_id, hacker_id
    ) max_score
    	-- faz o math para ter seomtne linhas com max_score, assim, sem  duplciata por challenge
        on h.hacker_id = max_score.hacker_id
        
group by h.hacker_id, name

/* don't accept hackers with total_score=0 */
having total_score > 0

/* finally order as required */
order by total_score desc, h.hacker_id
;
```

#### Easy

https://www.hackerrank.com/challenges/asian-population/

````sql
select
    sum(city.population)
from
    city inner join country
    on city.countrycode = country.code
where
    country.continent = 'Asia'
````

#### Medium: `Having` Complexo

https://www.hackerrank.com/challenges/challenges/

```
/*1st SELECTING THE VALUES requested to be printed*/
SELECT h.hacker_id, h.name, COUNT(c.challenge_id) AS challenge_counter
FROM hackers h
JOIN challenges c
    ON h.hacker_id = c.hacker_id
GROUP BY h.hacker_id, h.name

/*2nd applying the values found before*/
HAVING 
challenge_counter IN (
    SELECT aux_table.counter
    FROM(
        SELECT hacker_id, COUNT(challenge_id) AS counter 
        FROM challenges
        GROUP BY hacker_id
        ORDER BY counter DESC
    ) AS aux_table
    GROUP BY aux_table.counter 
    HAVING COUNT(aux_table.counter) = 1
)
OR
challenge_counter =(
    SELECT MAX(aux_table.counter)
    FROM(
        SELECT hacker_id, COUNT(challenge_id) AS counter
        FROM challenges
        GROUP BY hacker_id
        ORDER BY counter DESC
    ) AS aux_table)
/* Finally we order as requested (by counter and hacker_id)*/
ORDER BY challenge_counter DESC, h.hacker_id ASC;
```

## Medium: Envolvvendo datas e `DATA-DIFF`

https://www.hackerrank.com/challenges/sql-projects/problem

obs: * sql_mode='' based on [@jakab922](https://www.hackerrank.com/jakab922)'s comment. HackerRank changed the default sql_mode behavior to only_full_group_by at some point after I posted this.

Also, a full solution without the need to change modes was posted by [@dougal_michael](https://www.hackerrank.com/dougal_michael) below, which uses the MIN() function instead.*

```
SET sql_mode = '';
SELECT 
    Start_Date, End_Date
FROM 
    -- start-date que não sejam end-date
    (SELECT Start_Date FROM Projects 
        WHERE Start_Date NOT IN (SELECT End_Date FROM Projects)) a,
    -- end-date que não sejam start-date
    (SELECT End_Date FROM Projects 
        WHERE End_Date NOT IN (SELECT Start_Date FROM Projects)) b 
WHERE Start_Date < End_Date
GROUP BY Start_Date 
ORDER BY DATEDIFF(End_Date, Start_Date), Start_Date
```

#### MEDIUM : `USING`

USING é outra forma de escrever INNE JOIN

```
ON is the more general of the two. One can join tables ON a column, a set of columns and even a condition. For example:

SELECT * FROM world.City JOIN world.Country ON (City.CountryCode = Country.Code) WHERE ...

--------------

USING is useful when both tables share a column of the exact same name on which they join. In this case, one may say:

SELECT ... FROM film JOIN film_actor USING (film_id) WHERE ...
```



```
Select S.Name
From ( Students S join Friends F Using(ID)
       join Packages P1 on S.ID=P1.ID
       join Packages P2 on F.Friend_ID=P2.ID)
Where P2.Salary > P1.Salary
Order By P2.Salary;
```

#### Hard

https://www.hackerrank.com/challenges/15-days-of-learning-sql/

```
select 
submission_date ,

( SELECT COUNT(distinct hacker_id)  
 FROM Submissions s2  
 WHERE s2.submission_date = s1.submission_date AND    (SELECT COUNT(distinct s3.submission_date) FROM      Submissions s3 WHERE s3.hacker_id = s2.hacker_id AND s3.submission_date < s1.submission_date) = dateDIFF(s1.submission_date , '2016-03-01')) ,

(select hacker_id  from submissions s2 where s2.submission_date = s1.submission_date 
group by hacker_id order by count(submission_id) desc , hacker_id limit 1) as shit,
(select name from hackers where hacker_id = shit)
from 
(select distinct submission_date from submissions) s1
group by submission_date
```

