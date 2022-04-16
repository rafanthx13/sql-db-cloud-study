# LetsCode

Star: 12/04/2022



## Principais

#### `DISTINCT` e `GROUP_CONCAT` sobre `GROUP BY`

**1484. Group Sold Products By The Date - **

```
SELECT 
    sell_date, 
    count(distinct(product)) as num_sold, 
    group_concat(
        distinct(product)
        ) as products 
from activities 
group by sell_date
```

No grouoby é possivel fazer:

+ distinct, assim, `count(distinct` é a caontagem de valore sunicos
+ group_concat: É possivel fazer um group by de tudo. A saida é separada por virgula mas sem espaç

#### `FULL OUTER JOIN` implementado no Mysql

**1965. Employees With Missing Information**

**MYSQL NÂO TEM FULL OUTER JOIN. PARA FAZE-LO FAZMOS USANDO UNION DE LEFT JOIN COM RIGHT JOIN**

We can implement OUTER JOIN in MySQL by taking a LEFT JOIN and RIGHT JOIN union.
If column names of two tables are identical, we can use the USING clause instead of the ON clause.

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

#### DISTINC vs GROUP BY

[1148. Article Views I](https://leetcode.com/problems/article-views-i/)

**distinct ém uito custoso. uMA FORMA DE VOCÊ TIRAR DUPLICATAS DE FORMA OTIMIZADA É ATRAVEZ DO GROUP BY.**

**GROUP BY É MAIS RÁPIDO QUE DISTINCT**

```
SELECT
    distinct author_id as id
FROM views
WHERE author_id = viewer_id
ORDER BY author_id;
```

Outro soluçâo R´padia

```
select author_id as id 
from Views 
where author_id = viewer_id
group by 1
order by 1;
```



## Rnadom



#### [595. Big Countries](https://leetcode.com/problems/big-countries/)

OBS: se atente se é `>=` ou `>`

```
SELECT 
    name,
    population,
    area
FROM 
    World
WHERE
    area >= 3000000 OR population  >= 25000000;
```

#### [584. Find Customer Referee](https://leetcode.com/problems/find-customer-referee/)

obs: nessa questaao o que pega é que alguns tem reffere_id = null, e esse nul nâo é pego por igual ou difernet, entao tem que colcoar uma segunda condiçâo no `where`

```
-- EASY
select
    name
from
    customer
where
    referee_id <> 2 or referee_id is null;
    
```

#### [183. Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/)

Envolve saber que customr nao está referenciado em Order. Então fazemos um `LEF JOIN` e verificamos onde esse join não entrou no customers, pois, nesses casos,todos os dados de `orders` serão `NULL`

```
-- EASY
select
    name as Customers
from
    customers as c
    left join
        orders as o
    on
        c.id = o.customerid
where
    o.customerid is null;
```



#### [175. Combine Two Tables](https://leetcode.com/problems/combine-two-tables)

```
select
    firstname,
    lastname,
    city,
    state
from
    Person
    	left join
    address
    	on Person.personid = address.personid;
```



#### [1873. Calculate Special Bonus](https://leetcode.com/problems/calculate-special-bonus/)

```
select employee_id,
    case 
        when employee_id % 2 <> 0 and name not like 'M%'
            then salary
        else 0
    end
    as bonus
from Employees 
order by employee_id;
```



#### [627. Swap Salary](https://leetcode.com/problems/swap-salary/)

Nessa questão é proposto fazer um UPDATE que altere dois valores. Isso só é possível com `CASE` OU `IF` 

```
UPDATE salary
SET
    sex = CASE sex
        WHEN 'm' THEN 'f'
        ELSE 'm'
    END;
```



#### [196. Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/)

`DELETE`

```
DELETE p1 FROM Person p1 INNER JOIN Person p2
WHERE p1.email = p2.email AND p1.id > p2.id;
```

#### [1484. Group Sold Products By The Date](https://leetcode.com/problems/group-sold-products-by-the-date/)

```
SELECT 
    sell_date, 
    count(distinct(product)) as num_sold, 
    group_concat(
        distinct(product)
        ) as products 
from activities 
group by sell_date
```



#### [1527. Patients With a Condition](https://leetcode.com/problems/patients-with-a-condition/)

```
select patient_id, patient_name, conditions
from patients
where conditions like '% DIAB1%'
or conditions like 'DIAB1%'

ou

# Using REGEXP
select patient_id, patient_name, conditions
from patients
where conditions REGEXP '^DIAB1| +DIAB1'
```



#### [1965. Employees With Missing Information](https://leetcode.com/problems/employees-with-missing-information/)

We can implement OUTER JOIN in MySQL by taking a LEFT JOIN and RIGHT JOIN union.
If column names of two tables are identical, we can use the USING clause instead of the ON clause.

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



#### [1795. Rearrange Products Table](https://leetcode.com/problems/rearrange-products-table/)

Consegui fazer 100% sozinho

```
# Write your MySQL query statement below
select 
    p.*
from (
    select product_id, 'store1' as store, store1 as price
    from Products where store1 is not null
    UNION ALL
    select product_id, 'store2' as store, store2 as price
    from Products where store2 is not null
    UNION ALL
    select product_id, 'store3' as store, store3 as price
    from Products where store3 is not null

) as p
order by p.product_id;
```



#### [608. Tree Node](https://leetcode.com/problems/tree-node/)

Arvore binária. Faz um SELECT para cada consulta pra saber se ele já está presente (o que significa que é filho de outro, nao sendo nem root e nem leaf)

```
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



#### [176. Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)

**where filtra o resultado de max (sem grouppby)**

lembrando que max retorna um unico valor: nnull, ou o mairo que encontrar. Mas, se o WHERE tirar o maior elemneto, entao msotraria o segundo;

Lembre-se da ordem do SQL. Sabendo disso, como isso ocorre:

+ FROM employee; WHERE => Retirar o maior valor; Mostra o maior valor resultante
+ O WHERE ocorre ates e por isso filtra a tabela, asim, quando pega max,v vai epgar o 2 max pois o primeiro já não está mais lá

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

#### [1581. Customer Who Visited but Did Not Make](https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions/)

Eu fiz 100%

```SQL
# Write an SQL query to find the IDs of the users who visited without making any transactions and the number of times they made these types of visits.
# Return the result table sorted in any order.

select
    customer_id,
    COUNT(*) as count_no_trans
from
    visits v left join transactions t 
    ON v.visit_id = t.visit_id
WHERE 
    t.transaction_id IS NULL
group by
    v.customer_id  

```

#### [1148. Article Views I](https://leetcode.com/problems/article-views-i/)

Fiz 100%, fácil

```
# Write an SQL query to find all the authors that viewed at least one of their own articles. Return the result table sorted by id in ascending order.

SELECT
    distinct author_id as id
FROM views
WHERE author_id = viewer_id
ORDER BY author_id;
```

Outro soluçâo R´padia

```
select author_id as id 
from Views 
where author_id = viewer_id
group by 1
order by 1;
```



## Hard Questions

#### [262. Trips and Users](https://leetcode.com/problems/trips-and-users)

source: https://github.com/NIteshx2/AdvancedSQL_Interview/tree/master/LeetCode/262_Trips_and_Users

```
SELECT
  t.Request_at AS Day
  ,ROUND(SUM(t.Status != "completed") / COUNT(*), 2) AS 'Cancellation Rate'
FROM Trips t
JOIN Users d 
  ON t.Driver_Id = d.Users_Id 
  AND d.Banned = 'No'
  AND t.Request_at BETWEEN "2013-10-01" AND "2013-10-03"
JOIN Users c 
  ON t.Client_Id = c.Users_Id 
  And c.Banned = 'No'
GROUP BY t.Request_at;
```

