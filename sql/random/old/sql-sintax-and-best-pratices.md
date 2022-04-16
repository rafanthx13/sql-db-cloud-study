# SQL Sintaxe e Melhores Práticas

## Melhores práticas

+ **Comentários**: `--` ou `#`
+ Terminar os código com `;`
+ Evitar * ao invés de um por um

## Exemplos de códigos SQL

```sql
# Selecioanr Tudo
SELECT * FROM customer;

# FROM e WHERE
SELECT firstName, lastName, shippingAddress FROM customer WHERE customerID = 1001;

# JOIN
SELECT customer.customerID, order.order_id, order_line.order_item
FROM customer
    INNER JOIN order
        ON customer.customerID = order.customerID
    INNER JOIN order_line
        ON order.orderID = order_line.orderID;
```

## Contruir tabelas

1. Decida se será camelCsse ou senak_case
2. Toda tabela temque ter a chave primária como 'id', nunca 'tabela_id', isso guarda apaenas para FK
3. Jamais duplique o nome da tabela como atributo, pois ficará terrível no FrontEnd (city.city, product. product). Mude para: (city.name, product.desc)