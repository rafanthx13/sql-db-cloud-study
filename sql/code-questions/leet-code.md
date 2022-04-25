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

## Random 2 - Easy Questions

#### [ 1741- Find Total Time Spent by Each Employee](https://leetcode.com/problems/find-total-time-spent-by-each-employee)

Consegui fazer sozinho

```
select
    event_day as day,
    emp_id,
    sum(out_time - in_time) as total_time
from Employees
group by event_day, emp_id
```



#### [ 1693 Daily Leads and Partners](https://leetcode.com/problems/daily-leads-and-partners)|

Foiu feito um count em distinct, ou seja, o len(unique); Fiz sozinho

```
select
    date_id,
    make_name,
    count(distinct lead_id) as unique_leads,
    count(distinct partner_id) as unique_partners 
from DailySales 
group by date_id, make_name 


```



#### [1587 Bank Account Summary II](https://leetcode.com/problems/bank-account-summary-ii) 

sozinho

```
select
    name as NAME,
    sum(amount) as BALANCE
from Users inner join Transactions USING(account)
group by account
having sum(amount) > 10000 
```



#### [ 1890 The Latest Login in 2020](https://leetcode.com/problems/the-latest-login-in-2020) |        

Quando usa MAX/MIN retorna o menor valor etambem todos os outros dados da linha, volta a maior data e tambem o uiser_id dessa maior data, por isos só dá pra ter um unico max/min                                                   

```
SELECT
    user_id,
    MAX(time_stamp) AS last_stamp
FROM 
    Logins
WHERE 
    YEAR(time_stamp) = 2020 
GROUP BY 
    user_id;

```



#### [] 1407 | Top Travellers](https://leetcode.com/problems/top-travellers) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```
select 
    u.name, 
    ifnull(sum(r.distance), 0) as travelled_distance
# lef join pois pode nao ter distance
from users u left join rides r
    on u.id = r.user_id
group by 
    r.user_id
order by 
    # order by eh a ultima coisa a ser feita,
    # por isso da pra fazer sobre o attr com alias
    travelled_distance desc, u.name asc


```

| 1179 | [Reformat Department Table](https://leetcode.com/problems/reformat-department-table) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Caso de Pivot TABLE

**PIVOT TABLE de forma manual no my-sql**

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

#### 

| 586  | [Customer Placing the Largest Number of Orders](https://leetcode.com/problems/customer-placing-the-largest-number-of-orders) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

O ORDER BY PODE SER USADO sobre uma funçâo: vai executar o count no group by e apartir do resultado ordenar quanto ao valor

```
SELECT customer_number
FROM orders
GROUP BY customer_number
ORDER BY COUNT(order_number) DESC 
LIMIT 1

```



#### 

| 620  | [Not Boring Movies](https://leetcode.com/problems/not-boring-movies) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |



```
SELECT *
FROM cinema
WHERE 
    id % 2 <> 0 # want id impar/odd
    AND 
    description NOT LIKE '%boring%' # nao deve ter boring
ORDER BY 
    rating DESC;

```



#####

[1050 - Actors and Directors Who Cooperated At Least Three Times](https://leetcode.com/problems/actors-and-directors-who-cooperated-at-least-three-times)

```
select
    actor_id,
    director_id
from ActorDirector
group by actor_id, director_id
having count(*) > 2

```

##### [ 1729 - Find Followers Count](https://leetcode.com/problems/find-followers-count)



```
select
    user_id,
    count(*) as followers_count
from Followers 
group by
    user_id 
order by user_id
```





| 181  | [Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```
select
    e1.name as Employee
from Employee as e1 inner join Employee as e2 
    on e1.managerId = e2.id
where
    e1.salary > e2.salary


```



| 1141 | [User Activity for the Past 30 Days I](https://leetcode.com/problems/user-activity-for-the-past-30-days-i) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Uso de dadta diff

```
select 
    activity_date as day, 
    count(distinct user_id) as active_users 
from 
    Activity
where 
    datediff('2019-07-27', activity_date) < 30
group by 
    activity_date

```

| 1084 | [Sales Analysis III](https://leetcode.com/problems/sales-analysis-iii) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |





```
# Queremos somente produto que somente foram vendidos em um periodo
# usamos group by para pelo max/min saber se: se estiver algum alem do periodo, quer dizer que foi vendido fora do periodo e entao nao entra
# USO DE CAST para converter uma cnstatne em DATE
SELECT 
    s.product_id, product_name
FROM Sales s LEFT JOIN Product p
    ON s.product_id = p.product_id
GROUP BY 
    s.product_id
HAVING MIN(sale_date) >= CAST('2019-01-01' AS DATE) AND
       MAX(sale_date) <= CAST('2019-03-31' AS DATE)
```

| 596  | [Classes More Than 5 Students](https://leetcode.com/problems/classes-more-than-5-students) |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```
SELECT
    class
FROM Courses
GROUP BY class
HAVING COUNT(*) >= 5
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

#### [197. Rising Temperature](https://leetcode.com/problems/rising-temperature/)

Você quer comparar uma data com a anterior. Ao invez de fazer winbdows function ou algo do tipo, você faz o join com a row que tenha uma date_diff de 1

```
select 
    w.id as Id
from 
    Weather w inner join Weather t
    on date_sub(w.recordDate, interval 1 day) = t.recordDate
where 
    w.temperature > t.temperature
```



#### [607. Sales Person](https://leetcode.com/problems/sales-person/)

```
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

