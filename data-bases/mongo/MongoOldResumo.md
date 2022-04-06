# 

**NoSQL**: Nâo relacional
Há 4 tipos de bd nosql
+ 1. Chave/valor
+ 2. DOcumentos
  - MongoDB
+ 3. Grafos:
  
+ 4. COluna

## Mongo

+ Mongo é um banco de dados baseado em documentos em JSON. ELe armazena em JSON.
+ Mongo é **SEM ESQUEMA**
  - Você não tem a formalidade de um banco de dados comun
+ Ele nâo usa SQL. ELe usa uma API muito simples parecida com REST

**Vantagens**
+ Escalonamento Horizontal
  - É mais fácil trabalhar o mongoem CLsuteres do que o SQL comun.
  - Ele foi feito pra funcionar bem em sistemas distribuidos

**Particularidades**
+ Diferene do SQL, ele considera letra maisuculas e minustulas> WHERE é totalmente difernete de where

## Como instalar

//Procure no React a parte que eu instaleis

## Como subir o mongo

```
$ mongod
```

E deve ficar executando (nâo sair do comando) e ter como última linha algo como 

````
waiting for connections on port 27017

## Console do mongo

Execute o comando a seguir em outro terminal
````
mongo
````

Deve então estar o cursor em:

````
> _
````

Sair

````
exit
````

### Comadnos

#### Mostrar tipos de documentos (algo como tabelas)

````
show dbs
````

#### Saber em qual banco/schema está e como trocar de bd/schema

De acordo com a lsitagem do comando anteriro `show dbs` eu posso trocar entre bancos.

Esse db é como se fosse um schema, onde se pode ter várias collections (as collections sâo como tabelas)

Para saber em qual banco estou execute

````
db
````

Para trocar

````
use db_finance
````

Vai mostrar
````
switched to db db_finance
````


Onde user_fincnae é um novo banco, que pode ou nâo exisitr 

### Criar uma collection (como se fosse uma tabela)

````
db.createCollection('billingCycles')
````

Se der certo, ele retorna:

````
{ "ok" : 1 }

````

Um banco que tem uma collection aparece quando você executa `show dbs`. ENquanto nâo tiver nenhuma collection é como se estivesse em memória. Se você deletar todas as collections de um db, entâo o db é removedio

### Mostrar collections de um banco

````
show collections
````

### Deletar uma collection

````
db.billingCycles.drop()
````
`
Deve retornar `true` se conseguir

### Limpar Console

```
cls
```

### Inserir dados

````
db.billingcycles.insert({name:"Janeiro/17", month: 1, year: 2017})
````

A colection nâo precisa exisitr, se nâo existir enoa é criada

Outra forma é

````
db.billingcycles.save({name:"Fevereiro/17", month: 2, year: 2017})
````

O `save` tem o difenrencia de update um dado se você passar o `_id`

OBS:
Numa mesma collection, você pode armazenar coisa complemtenamente diferente entre si.

OBS: no console do mongo, você pode clicar ENTER e escrever.

Em: ` db.billingcycles.save({` ao apertae `ENTER` ele vai criar uma margem pois percebe que vocÊ nâo terminou de escrever o JSON.

````
 db.billingcycles.save({
... name: "Marco/17",
... month: 5,
... year: 2017,
... credits: [
... {name: "salario", value: 5000}
... ],
... debts: [
... {name: "luz", value: 100, status: "pago"},
... {name: "tetelfone", value: 1000, status: "PENDENTE"}
... ]
... })
WriteResult({ "nInserted" : 1 })

## Buscar

#### Bs=uscar todos os docuemtnos de uma collection

````
db.billingcycles.find()
````

Retorna algo como

````
{ "_id" : ObjectId("5dd82e65d1e07a6f01b2e860"), "name" : "Janeiro/17", "month" : 1, "year" : 2017 }
{ "_id" : ObjectId("5dd82ebfd1e07a6f01b2e861"), "name" : "Fevereiro/17", "month" : 2, "year" : 2017 }
{ "_id" : ObjectId("5dd82fadd1e07a6f01b2e862"), "name" : "Marco/17", "month" : 5, "year" : 2017, "credits" : [ { "name" : "salario", "value" : 5000 } ], "debts" : [ { "name" : "luz", "value" : 100, "status" : "pago" }, { "name" : "tetelfone", "value" : 1000, "status" : "PENDENTE" } ] }

````

 Uma forma de aparecer elegante é
````
db.billingcycles.find().pretty()
````


````
{
	"_id" : ObjectId("5dd82e65d1e07a6f01b2e860"),
	"name" : "Janeiro/17",
	"month" : 1,
	"year" : 2017
}
{
	"_id" : ObjectId("5dd82fadd1e07a6f01b2e862"),
	"name" : "Marco/17",
	"month" : 5,
	"year" : 2017,
	"credits" : [
		{
			"name" : "salario",
			"value" : 5000
		}
	],
	"debts" : [
		{
			"name" : "luz",
			"value" : 100,
			"status" : "pago"
		},
		{
			"name" : "tetelfone",
			"value" : 1000,
			"status" : "PENDENTE"
		}
	]
}

`````

### Encontrar um documento especpifico

+ db.billingcycles.find({$or: [{month: 1}, {month: 2}]}).pretty()
  - Pegar documentos cujo months seja 1 ou month seja 2
+ db.billingcycles.find({credits:{$exists:true}})
  - Pegar documentos que tenham o atribuo credits

````
> db.billingcycles.findOne()
{
	"_id" : ObjectId("5dd82e65d1e07a6f01b2e860"),
	"name" : "Janeiro/17",
	"month" : 1,
	"year" : 2017
}
> db.billingcycles.findOne({month: 2})
{
	"_id" : ObjectId("5dd82ebfd1e07a6f01b2e861"),
	"name" : "Fevereiro/17",
	"month" : 2,
	"year" : 2017
}
db.billingcycles.find({$or: [{month: 1}, {month: 2}]}).pretty()
{
	"_id" : ObjectId("5dd82e65d1e07a6f01b2e860"),
	"name" : "Janeiro/17",
	"month" : 1,
	"year" : 2017
}
{
	"_id" : ObjectId("5dd82ebfd1e07a6f01b2e861"),
	"name" : "Fevereiro/17",
	"month" : 2,
	"year" : 2017
}
> db.billingcycles.find({credits:{$exists:true}})
{ "_id" : ObjectId("5dd82fadd1e07a6f01b2e862"), "name" : "Marco/17", "month" : 5, "year" : 2017, "credits" : [ { "name" : "salario", "value" : 5000 } ], "debts" : [ { "name" : "luz", "value" : 100, "status" : "pago" }, { "name" : "tetelfone", "value" : 1000, "status" : "PENDENTE" } ] }


````

Para paginaçâo:

+ db.billingcycles.find({year:2017}).skip(1)
  - Pula o primeiro
+ db.billingcycles.find({year:2017}).skip(1).limit(1)
  - Pula o primeiro e retorna somente 1


````
> db.billingcycles.find({year:2017})
{ "_id" : ObjectId("5dd82e65d1e07a6f01b2e860"), "name" : "Janeiro/17", "month" : 1, "year" : 2017 }
{ "_id" : ObjectId("5dd82ebfd1e07a6f01b2e861"), "name" : "Fevereiro/17", "month" : 2, "year" : 2017 }
{ "_id" : ObjectId("5dd82fadd1e07a6f01b2e862"), "name" : "Marco/17", "month" : 5, "year" : 2017, "credits" : [ { "name" : "salario", "value" : 5000 } ], "debts" : [ { "name" : "luz", "value" : 100, "status" : "pago" }, { "name" : "tetelfone", "value" : 1000, "status" : "PENDENTE" } ] }
> db.billingcycles.find({year:2017}).skip(1)
{ "_id" : ObjectId("5dd82ebfd1e07a6f01b2e861"), "name" : "Fevereiro/17", "month" : 2, "year" : 2017 }
{ "_id" : ObjectId("5dd82fadd1e07a6f01b2e862"), "name" : "Marco/17", "month" : 5, "year" : 2017, "credits" : [ { "name" : "salario", "value" : 5000 } ], "debts" : [ { "name" : "luz", "value" : 100, "status" : "pago" }, { "name" : "tetelfone", "value" : 1000, "status" : "PENDENTE" } ] }
> db.billingcycles.find({year:2017}).skip(1).limit(1)
{ "_id" : ObjectId("5dd82ebfd1e07a6f01b2e861"), "name" : "Fevereiro/17", "month" : 2, "year" : 2017 }

````

**Buscar somente alguns valores da consulta**

Eu passo um segund objeto, que tenha o id (pois é obrigatorio por default) e as colunas que eu quero como `1`

````
db.billingcycles.find({credits:{$exists:true}}, {_id:0, name:1}).pretty()
{ "name" : "Janeiro/17" }
{ "name" : "Marco/17" }

````

### Agregate : Fazneod umonte de coisa

O que o código abaixo faz:
1. O `$project` é como um SELECT: no caso, ele vai gerar um objeto {credit, debt} cujo valor serrao repectivamente o somatorio `$sum` dos valores de credit e debt respectivamente
2. $group: ???? (nao entendi direito


````
db.billingcycles.aggregate([{
...   $project:{credit:{$sum:"$credits.value"},debt:{$sum:"$debts.value"}},
... }, {
...   $group:{_id:null,credit:{$sum:"$credit"},debt:{$sum:"$debt"}}
... }
... ])
{ "_id" : null, "credit" : 5000, "debt" : 1100 }
````


#### Update

````
db.billingcycles.update({
  $and:[{ month: 1}, { year:2017 }]}, {
  $set: {
    credits:[{
      name:"Salário",value:5000
    }]
  }
})
````

O $and é o WHERE e o set os novos valores


Exemplo:`a seguir foi feito o UPDTE  e depois buscou-se o deocuemtno. PErcebe-se que erelamente foi moidificado

````
> db.billingcycles.update({
...   $and:[{ month: 1}, { year:2017 }]}, {
...   $set: {
...     credits:[{
...       name:"Salário",value:5000
...     }]
...   }
... })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.billingcycles.findOne()
{
	"_id" : ObjectId("5dd82e65d1e07a6f01b2e860"),
	"name" : "Janeiro/17",
	"month" : 1,
	"year" : 2017,
	"credits" : [
		{
			"name" : "Salário",
			"value" : 5000
		}
	]
}

````


### COntar quantidade de registors

db.billingcycles.count()

````
> db.billingcycles.count()
3
> db.billingcycles.remove({month:2})
WriteResult({ "nRemoved" : 1 })
> db.billingcycles.count()
2

### Apagar um registro

````
db.billingcycles.remove({month:2})
````

### Apagar um banco

````
db.dropDatabase()
````

