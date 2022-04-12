# LetsCode

Star: 12/04/2022



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

