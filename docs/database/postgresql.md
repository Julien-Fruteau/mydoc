---
title: Postgresql
tags:
  - database
  - postgresql
  - postgres
  - pg
  - sql
---

## administration

## union caveats

- multiple union with null as "fieldName" can lead to type mismatch. 
- For instance consider:

```sql
select a as "A", null as "D" from table1
union
select null as "A", d as "D" from table2
```

- with table1:

| a    | b       | c       |
| ---- | ------- | ------- |
| int4 | varchar | boolean |

- with table2:

| d       | e       | f      |
| ------- | ------- | ------ |
| numeric | varchar | bigint |

⚠️ Problem: `null as` can be infered to the wrong type and union fails

> for instance `null as "D"` can be set to _varchar_ whereas d from table2 is _numeric_

💡Solution: cast type using `null::<type>`

- Query updated:

```sql
select a as "A", null::numeric as "D" from table1
union
select null::int4 as "A", d as "D" from table2
```
