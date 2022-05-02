# DELETE no SQL

Caos curiosos apra realisar tarefas com `DELETE`

## Deletar duplicatas

+ Você liga a tabela a si mesma
+ Com `p1.Id > p2.Id` você vai: **DELETAR as dupliataas e manter aquela com MENOR id**

````
DELETE p1 
FROM Person p1 INNER JOIN Person p2
WHERE 
	p1.email = p2.email AND p1.id > p2.id;
````