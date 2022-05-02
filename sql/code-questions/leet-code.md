# LetsCode Questions

## EASY QUESTIONS

### [595. Big Countries](https://leetcode.com/problems/big-countries/)

OBS: se atente se é `>=` ou `>`

```sql
SELECT 
    name,
    population,
    area
FROM 
    World
WHERE
    area >= 3000000 OR population  >= 25000000;
```

### [584. Find Customer Referee](https://leetcode.com/problems/find-customer-referee/)

obs: nessa questaao o que pega é que alguns tem reffere_id = null, e esse nul nâo é pego por igual ou difernet, entao tem que colcoar uma segunda condiçâo no `where`

```sql
-- EASY
select
    name
from
    customer
where
    referee_id <> 2 or referee_id is null;
    
```

### [183. Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/)

Envolve saber que customr nao está referenciado em Order. Então fazemos um `LEF JOIN` e verificamos onde esse join não entrou no customers, pois, nesses casos,todos os dados de `orders` serão `NULL`

```sql
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

### [175. Combine Two Tables](https://leetcode.com/problems/combine-two-tables)

```sql
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



### [1873. Calculate Special Bonus](https://leetcode.com/problems/calculate-special-bonus/)

```sql
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

### [627. Swap Salary](https://leetcode.com/problems/swap-salary/)

Nessa questão é proposto fazer um UPDATE que altere dois valores. Isso só é possível com `CASE` OU `IF` 

```sql
UPDATE salary
SET
    sex = CASE sex
        WHEN 'm' THEN 'f'
        ELSE 'm'
    END;
```

### [196. Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/)

`DELETE`

```sql
DELETE p1 FROM Person p1 INNER JOIN Person p2
WHERE p1.email = p2.email AND p1.id > p2.id;
```

### [1484. Group Sold Products By The Date](https://leetcode.com/problems/group-sold-products-by-the-date/)

```sql
SELECT 
    sell_date, 
    count(distinct(product)) as num_sold, 
    group_concat(
        distinct(product)
        ) as products 
from activities 
group by sell_date
```

### [1527. Patients With a Condition](https://leetcode.com/problems/patients-with-a-condition/)

```sql
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

### [1965. Employees With Missing Information](https://leetcode.com/problems/employees-with-missing-information/)

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

### [1795. Rearrange Products Table](https://leetcode.com/problems/rearrange-products-table/)

Consegui fazer 100% sozinho

```sql
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

### [608. Tree Node](https://leetcode.com/problems/tree-node/)

Arvore binária. Faz um SELECT para cada consulta pra saber se ele já está presente (o que significa que é filho de outro, nao sendo nem root e nem leaf)

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

### [176. Second Highest Salary](https://leetcode.com/problems/second-highest-salary/)

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

### [1581. Customer Who Visited but Did Not Make](https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions/)

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

### [1148. Article Views I](https://leetcode.com/problems/article-views-i/)

Fiz 100%, fácil

```sql
# Write an SQL query to find all the authors that viewed at least one of their own articles. Return the result table sorted by id in ascending order.

SELECT
    distinct author_id as id
FROM views
WHERE author_id = viewer_id
ORDER BY author_id;
```

Outro soluçâo R´padia

```sql
select author_id as id 
from Views 
where author_id = viewer_id
group by 1
order by 1;
```

### [197. Rising Temperature](https://leetcode.com/problems/rising-temperature/)

Você quer comparar uma data com a anterior. Ao invez de fazer winbdows function ou algo do tipo, você faz o join com a row que tenha uma date_diff de 1

```sql
select 
    w.id as Id
from 
    Weather w inner join Weather t
    on date_sub(w.recordDate, interval 1 day) = t.recordDate
where 
    w.temperature > t.temperature
```



### [607. Sales Person](https://leetcode.com/problems/sales-person/)

```sql
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

### [1741 - Find Total Time Spent by Each Employee](https://leetcode.com/problems/find-total-time-spent-by-each-employee)

Consegui fazer sozinho

```sql
select
    event_day as day,
    emp_id,
    sum(out_time - in_time) as total_time
from Employees
group by event_day, emp_id
```



### [1693 Daily Leads and Partners](https://leetcode.com/problems/daily-leads-and-partners)

Foiu feito um count em distinct, ou seja, o len(unique); Fiz sozinho

```sql
select
    date_id,
    make_name,
    count(distinct lead_id) as unique_leads,
    count(distinct partner_id) as unique_partners 
from DailySales 
group by date_id, make_name
```

### [1587 Bank Account Summary II](https://leetcode.com/problems/bank-account-summary-ii) 

sozinho

```sql
select
    name as NAME,
    sum(amount) as BALANCE
from Users inner join Transactions USING(account)
group by account
having sum(amount) > 10000 
```

### [ 1890 The Latest Login in 2020](https://leetcode.com/problems/the-latest-login-in-2020)       

Quando usa MAX/MIN retorna o menor valor etambem todos os outros dados da linha, volta a maior data e tambem o uiser_id dessa maior data, por isos só dá pra ter um unico max/min                                                   

```sql
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

### [1407 - Top Travellers](https://leetcode.com/problems/top-travellers) 


```sql
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

### [1179 - Reformat Department Table](https://leetcode.com/problems/reformat-department-table) 

Caso de Pivot TABLE

**PIVOT TABLE de forma manual no my-sql**

```sql
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

### [586 - Customer Placing the Largest Number of Orders](https://leetcode.com/problems/customer-placing-the-largest-number-of-orders) 

O ORDER BY PODE SER USADO sobre uma funçâo: vai executar o count no group by e apartir do resultado ordenar quanto ao valor

```sql
SELECT customer_number
FROM orders
GROUP BY customer_number
ORDER BY COUNT(order_number) DESC 
LIMIT 1
```

### [620 - Not Boring Movies](https://leetcode.com/problems/not-boring-movies)



```sql
SELECT *
FROM cinema
WHERE 
    id % 2 <> 0 # want id impar/odd
    AND 
    description NOT LIKE '%boring%' # nao deve ter boring
ORDER BY 
    rating DESC;
```

### [1050 - Actors and Directors Who Cooperated At Least Three Times](https://leetcode.com/problems/actors-and-directors-who-cooperated-at-least-three-times)

```sql
select
    actor_id,
    director_id
from ActorDirector
group by actor_id, director_id
having count(*) > 2
```

### [1729 - Find Followers Count](https://leetcode.com/problems/find-followers-count)

```sql
select
    user_id,
    count(*) as followers_count
from Followers 
group by
    user_id 
order by user_id
```

### [181 - Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers) 

```sql
select
    e1.name as Employee
from Employee as e1 inner join Employee as e2 
    on e1.managerId = e2.id
where
    e1.salary > e2.salary
```

### [1141 - User ctivity for the Past 30 Days I](https://leetcode.com/problems/user-activity-for-the-past-30-days-i) 

Uso de dadta diff

```sql
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

### [ 1084 - Sales Analysis III](https://leetcode.com/problems/sales-analysis-iii) 

```sql
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

### [596 - Classes More Than 5 Students](https://leetcode.com/problems/classes-more-than-5-students)

```sql
SELECT
    class
FROM Courses
GROUP BY class
HAVING COUNT(*) >= 5
```

## MEDIUM and HARD QUESTIONS

### [262. Trips and Users](https://leetcode.com/problems/trips-and-users)

source: https://github.com/NIteshx2/AdvancedSQL_Interview/tree/master/LeetCode/262_Trips_and_Users

```sql
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

### [1393 - Capital Gain/Loss](https://leetcode.com/problems/capital-gainloss) 

sozinho; Estrat´gia: Uso uma SubQ para inverter o sinal de quem é Buy, e asim somar positivos e negativos no select de fora

```sql
select
    op.stock_name,
    sum(op.price) as capital_gain_loss
from ( select 
     stock_name,
    case 
        when operation = 'Buy'
        then price * (-1)
        else price
    end price
from Stocks
) as op
group by op.stock_name
```

### [626 - Exchange Seats](https://leetcode.com/problems/exchange-seats)

```sql
# eh possivel fazer com +1 e -1 porque começa com id numero de 1 a n
SELECT
	CASE
        # quando for o ultimo caso
		WHEN seat.id % 2 <> 0 AND seat.id = (SELECT COUNT(*) FROM seat) 
            THEN seat.id
        # quando tiver numero par -1
		WHEN seat.id % 2 = 0 
            THEN seat.id - 1
        # quando id impar + 1
		ELSE
			seat.id + 1
	END as id,
	student 
FROM seat
ORDER BY id
;
```

### [1158 - Market Analysis I](https://leetcode.com/problems/market-analysis-i) 

**``IFNULL``**

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

### [178 - Rank Scores](https://leetcode.com/problems/rank-scores)

+ Usa-se ` pois Rank é uma keyword do mysql
+ Strategy: Voce faz uma 2 coluna que é a contagem de valores >= aquele valor sem repetir o valor nessa condicional. Assim a 1 linha é o maior valor e vai conferir quantos valores sao iguais ou maior que ele. Como é o maior, é apenas ele mesmo, entoa é 1. O 2 valor será ele mais o anteriro sendo assim 2, e assim por diante
+ Percebe que a 2 coluna tem como ref o select de fora por `s.Score`

```sql
SELECT
  Score,
  (SELECT count(distinct Score) FROM Scores WHERE Score >= s.Score) `Rank`
FROM Scores s
ORDER BY Score desc

```

### [184 - Department Highest Salary](https://leetcode.com/problems/department-highest-salary)

```sql
# O grande problema dessa questao eh que
# ele quer que mostre o max(salary) repetido
# se o max de um departamenteo eh 900, deve mostrar todo 9000
# SOLUCAO: Check se o valor eh o max daquele departamnteo
#        Usando uma subquery. Tem que ser 2 select, pois
#       na subqueri precisamos do departamento que sera dinamico
# em suma: faz uma sub


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
;
```

MInha soluçâo:

Ao invez de fazer um select para cada um, criar uma tabela temporaria com sub-query com o max de cada departamtento, asism, mostra os funconarios cja coluna de slario seja igual ao max-salary.

FIZ SOZINHO

```sql
SELECT
    d.name as Department,
    e.name as Employee,
    e.salary as Salary
FROM Employee e
    INNER JOIN Department d
    ON e.departmentId = d.id
    INNER JOIN (
        SELECT 
            d1.name,
            max(salary) as max_salary
        FROM Employee e1
            INNER JOIN Department d1
            ON e1.departmentId = d1.id
        GROUP BY 
            d1.name
        ) temp
    ON d.name = temp.name
WHERE
    e.salary = temp.max_salary 
```

com windows funcion

```sql
SELECT
    name as Department,
    employee,
    salary
FROM
(
SELECT
    departmentID,
    name as employee,
    salary,
    dense_rank() OVER(Partition by departmentID order by salary desc) as rank_no
    FROM
    Employee
    )e
LEFT JOIN
    DEPARTMENT d
on d.id=e.departmentID
where rank_no=1;
```

### [180 - Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers)

```sql
SELECT DISTINCT
    l1.Num AS ConsecutiveNums
FROM
    Logs l1,
    Logs l2,
    Logs l3
WHERE
    l1.Id = l2.Id - 1
    AND l2.Id = l3.Id - 1
    AND l1.Num = l2.Num
    AND l2.Num = l3.Num
```

### [177 - Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary) 

CRIANDO UMA PROCEDURE

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
DECLARE M INT;
SET M=N-1;
  RETURN (
      # Write your MySQL query statement below.
      SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT M, 1
  );
END
```

### [601 - Human Traffic of Stadium](https://leetcode.com/problems/human-traffic-of-stadium)

**Idea**

I've seen pretty many solutions using `join` of three tables or creating temporary tables with `n^3` rows. With my 5-years' working experience on data analysis, I can guarantee you this method will cause you "out of spool space" issue when you deal with a large table in big data field.

I recommend you to learn and master window functions like `lead`, `lag` and use them as often as you can in your codes. These functions are very fast, and whenever you find yourself creating duplicate temp tables, you should ask yourself: can I solve this with window functions.

**MySQL**

```sql
SELECT ID
    , visit_date
    , people
FROM (
    SELECT ID
        , visit_date
        , people
        , LEAD(people, 1) OVER (ORDER BY id) nxt
        , LEAD(people, 2) OVER (ORDER BY id) nxt2
        , LAG(people, 1) OVER (ORDER BY id) pre
        , LAG(people, 2) OVER (ORDER BY id) pre2
    FROM Stadium
) cte 
WHERE (cte.people >= 100 AND cte.nxt >= 100 AND cte.nxt2 >= 100) 
    OR (cte.people >= 100 AND cte.nxt >= 100 AND cte.pre >= 100)  
    OR (cte.people >= 100 AND cte.pre >= 100 AND cte.pre2 >= 100) 
```

### [185 - Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries) 

```sql
# Write an SQL query to find the employees who are high earners 
#   (A high earner in a department is an employee who has a salary in the top three unique salaries for that department)
# in each of the departments.

# ou seja que o top 3 dae cada departamento

SELECT 
    D.Name as Department, 
    E.Name as Employee, 
    E.Salary 
FROM 
    Department D, Employee E, Employee E2  
WHERE 
    D.ID = E.DepartmentId 
    and E.DepartmentId = E2.DepartmentId 
    and E.Salary <= E2.Salary
group by 
    D.ID, E.Name 
having 
    # vai pegar quem te tem somente ate 3 com slary maior ou igua a ele
    count(distinct E2.Salary) <= 3
order by 
    D.Name, E.Salary desc
```

## Questions Analyseds

### `DISTINCT` e `GROUP_CONCAT` sobre `GROUP BY`

**1484. Group Sold Products By The Date **

```sql
SELECT 
    sell_date, 
    count(distinct(product)) as num_sold, 
    group_concat(
        distinct(product)
        ) as products 
from activities 
group by sell_date
```

No grouopby é possivel fazer:

+ distinct, assim, `count(distinct` é a caontagem de valore sunicos
+ group_concat: É possivel fazer um group by de tudo. A saida é separada por virgula mas sem espaç

### `FULL OUTER JOIN` implementado no Mysql

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

### DISTINC vs GROUP BY

[1148. Article Views ](https://leetcode.com/problems/article-views-i/)

**distinct ém uito custoso. uMA FORMA DE VOCÊ TIRAR DUPLICATAS DE FORMA OTIMIZADA É ATRAVEZ DO GROUP BY.**

**GROUP BY É MAIS RÁPIDO QUE DISTINCT**

```sql
SELECT
    distinct author_id as id
FROM views
WHERE author_id = viewer_id
ORDER BY author_id;
```

Outro soluçâo mais rápida

```sql
select author_id as id 
from Views 
where author_id = viewer_id
group by 1
order by 1;
```
