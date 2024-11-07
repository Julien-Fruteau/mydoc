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

## update sequence

📖 case you need to update sequence when the following occurs: first create db schema, second import data

🤔 more details ? Because the sequence created stayed at the initial values whereas the PK from data imported increased

💡 create a migration script to run after data import

```sql
setval 'sequence_name', (select max(id) + 1 from table_name);
```

🤔 why `+ 1` ? In case max returns 0, the query would fail. Hence add + 1 as safeguard

rollback migration

```sql
alter sequence_name restart with 1;
```

## Dump & Restore

> locally from remote host with different pg version

📖 key points :

- avoid running pg_dump on remote host and exposing filesystem usage at risk, specifically the running database
- avoid installing on your local host a specific pg cli version

- dump:

```bash
DATA=/data
docker run --rm -it \
    --network="host" \
    -v $(pwd):$DATA \
    postgres:$PG_REMOTE_VERSION \
    pg_dump --verbose --host=$PG_HOST_FQDN --port=$PG_PORT --username=$PG_USER \
      --format=c --encoding=UTF-8 --no-privileges --no-owner --clean --create \
      --file $DATA/pg_dump.backup -n "public" $PG_DB
```

- restore:

```bash
DATA=/data
docker run --rm -it \
    --network="host" \
    -v $(pwd):$DATA \
    postgres:$PG_TARGET_VERSION \
    pg_restore --verbose --host=$PG_TARGET_HOST_FQDN --port=$PG_PORT --username=$PG_USER \
      --format=c  --no-privileges --no-owner --clean --dbname $PG_DB $DATA/pg_dump.backup
```

ℹ️ --create and --clean options are not compatible, either one of them for pg_restore
