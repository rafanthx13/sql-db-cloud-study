# Pivot Table

## Links



## O que eu entendi de pivot table

É sumarizar a tabela tendo uma coluna como base fazendo uma *tabela trasnposta*.

Tendo como base uma coluna, faz dos valores unicos de todas as outras colunas, e entao, faz a sua contagem.

**pívot table nâo é um mapemaneto, mas um conceito**: Por conta disso pode aplicar de diversas forma

no mysql, a unica  forma de fazer é:

```
Aplicar CASE WHEN THEN END para cada HARD-CODE
```

### Problema: HardCode

Só há esse problema em SGBD que nâo tem PIVOT TABLE natvio.

O mysql nao tem.

**Como pode haver muitos valores para uma coluna, é necessáiro então criar via python/code-languge para gerar s consulta SQL pois `VAI TER MUITO CASE WHEN`**



## Random Content

A **pivot table** is a [table](https://en.wikipedia.org/wiki/Table_(information)) of grouped values that aggregates the individual items of a more extensive table (such as from a [database](https://en.wikipedia.org/wiki/Database), [spreadsheet](https://en.wikipedia.org/wiki/Spreadsheet), or [business intelligence program](https://en.wikipedia.org/wiki/Business_intelligence_software)) within one or more discrete categories. This summary might include sums, averages, or other statistics, which the pivot table groups together using a chosen aggregation function applied to the grouped valu

Pivot tables are a piece of summarized information that is generated from a large underlying dataset. It is generally used to report on specific dimensions from the vast datasets. Essentially, the user can convert rows into columns. This gives the users the ability to transpose columns from a SQL Server table easily and create reports as per the requirements.



![](https://modern-sql.com/static/og-pivot.en.mOZoXXf5.svg)

![](https://www.sqlshack.com/wp-content/uploads/2020/04/pivot-table-example.png)

## Exemplo

https://codingsight.com/pivot-tables-in-mysql/

```
CREATE TABLE PurchaseOrderHeader (
  OrderID INT(11) NOT NULL,
  EmployeeID INT(11) NOT NULL,
  VendorID INT(11) NOT NULL,
  PRIMARY KEY (OrderID)
);

INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (1, 258, 1580);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (2, 254, 1496);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (3, 257, 1494);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (4, 261, 1650);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (5, 251, 1654);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (6, 253, 1664);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (7, 255, 1678);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (8, 256, 1616);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (9, 259, 1492);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (10, 250, 1602);
INSERT PurchaseOrderHeader(OrderID, EmployeeID, VendorID) VALUES (11, 258, 1540);



```

```
SELECT * FROM PurchaseOrderHeader;
```





```
OrderID	EmployeeID	VendorID
1		258			1580
2		254			1496
3		257			1494
4		261			1650
5		251			1654
6		253			1664
7		255			1678
8		256			1616
9		259			1492
10		250			1602
11		258			1540
```

```
VendorID	Emp250	Emp251	Emp252	Emp253	Emp254
1496		0		0		0		0		1
1654		0		1		0		0		0
1664		0		0		0		1		0
1602		1		0		0		0		0
```

## Gerando CASE QHEN

Gerado do código PHP

```
for $i = 0 to NUM_DEVIATION_RANGES
	$Min = $Low + ($StandardDeviation * $i)
	$Max = ($Min + $StandardDeviation) – 1;
	$Query .= "SUM( IF( CAST(gamedata.Data AS UNSIGNED) >= $Min _
              AND CAST(gamedata.Data AS UNSIGNED) <= $Max, 1, 0) ) AS ‘$Min-$Max’,";
end for 
```



```
SELECT
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 60 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 87,1,0) ) 
        AS '60-87',
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 88 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 115,1,0) ) 
        AS '88-115',
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 116 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 143,1,0) ) 
        AS '116-143',
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 144 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 171,1,0) ) 
        AS '144-171',
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 172 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 199,1,0) ) 
        AS '172-199',
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 200 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 227,1,0) ) 
        AS '200-227',
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 228 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 255,1,0) ) 
        AS '228-255',
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 256 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 283,1,0) ) 
        AS '256-283',
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 284 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 311,1,0) ) 
        AS '284-311',
    SUM( IF( 
        CAST(gamedata.Data AS UNSIGNED) >= 312 AND 
        CAST(gamedata.Data AS UNSIGNED) <= 339,1,0) ) 
        AS '312-339'
FROM gamedata, gamefields, field_types
WHERE gamedata.fieldid = gamefields.FieldID
    AND gamefields.FieldTypeID = field_types.typeid
    AND field_types.typename = 'Numeric'
    AND gamefields.FriendlyName = 'Total Eggs' 

```

 ## Exemplo de código de 	PVITO TABLE



```SQL
SELECT  P.`company_name`,
    COUNT(
        CASE 
            WHEN P.`action`='EMAIL' 
            THEN 1 
            ELSE NULL 
        END
    ) AS 'EMAIL',
    COUNT(
        CASE 
            WHEN P.`action`='PRINT' AND P.`pagecount` = '1' 
            THEN P.`pagecount` 
            ELSE NULL 
        END
    ) AS 'PRINT 1 pages',
    COUNT(
        CASE 
            WHEN P.`action`='PRINT' AND P.`pagecount` = '2' 
            THEN P.`pagecount` 
            ELSE NULL 
        END
    ) AS 'PRINT 2 pages',
    COUNT(
        CASE 
            WHEN P.`action`='PRINT' AND P.`pagecount` = '3' 
            THEN P.`pagecount` 
            ELSE NULL 
        END
    ) AS 'PRINT 3 pages'
FROM    test_pivot P
GROUP BY P.`company_name`;
```



```
select table_record_id,
group_concat(if(value_name='note', value_text, NULL)) as note
,group_concat(if(value_name='hire_date', value_text, NULL)) as hire_date
,group_concat(if(value_name='termination_date', value_text, NULL)) as termination_date
,group_concat(if(value_name='department', value_text, NULL)) as department
,group_concat(if(value_name='reporting_to', value_text, NULL)) as reporting_to
,group_concat(if(value_name='shift_start_time', value_text, NULL)) as shift_start_time
,group_concat(if(value_name='shift_end_time', value_text, NULL)) as shift_end_time
from other_value
where table_name = 'employee'
and is_active = 'y'
and is_deleted = 'n'
GROUP BY table_record_id
```



```
SELECT
    company_name,  
    SUM(action = 'EMAIL')AS Email,
    SUM(action = 'PRINT' AND pagecount = 1)AS Print1Pages,
    SUM(action = 'PRINT' AND pagecount = 2)AS Print2Pages,
    SUM(action = 'PRINT' AND pagecount = 3)AS Print3Pages
FROM t
GROUP BY company_name
```




