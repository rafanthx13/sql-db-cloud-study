# Ever SQL

**Site que testa consultas e especifica o que pode melhorar para ser mais eficiente**

## UNION x UNION ALL

Use UNION ALL instead of UNION

Always use UNION ALL unless you need to eliminate duplicate records. By using UNION ALL, you'll avoid the expensive distinct operation the database applies when using a UNION clause.

```
-- faça assim
SELECT
  id
FROM
  tbl1
WHERE
  user_id = 1
UNION ALL
SELECT
  id
FROM
  tbl2
WHERE
  account_id = 1;
```

Instead of:

```
-- nao faça assim
SELECT
  id
FROM
  tbl1
WHERE
  user_id = 1
UNION
SELECT
  id
FROM
  tbl2
WHERE
  account_id = 1;
```

