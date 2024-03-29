### Tamaño actual de la base de datos, todas las tablas
```sql
--tamaño actual de la DB. sizeAllTables
SELECT d.datname AS Name,  pg_catalog.pg_get_userbyid(d.datdba) AS Owner,
    CASE WHEN pg_catalog.has_database_privilege(d.datname, 'CONNECT')
        THEN pg_catalog.pg_database_size(d.datname)
        ELSE 0
    END AS size_bytes,
  CASE WHEN pg_catalog.has_database_privilege(d.datname, 'CONNECT')
        THEN pg_catalog.pg_size_pretty(pg_catalog.pg_database_size(d.datname))
        ELSE 'No Access'
    END AS SIZE
FROM pg_catalog.pg_database d
    ORDER BY
    CASE WHEN pg_catalog.has_database_privilege(d.datname, 'CONNECT')
        THEN pg_catalog.pg_database_size(d.datname)
        ELSE NULL
    END DESC -- nulls first
    LIMIT 50
```

### Findig the total size of your biggest tables, in human format
```sql
--Finding the total size of your biggest tables
SELECT nspname || '.' || relname AS "relation",
  pg_total_relation_size(C.oid) as "size_bytes",
    pg_size_pretty(pg_total_relation_size(C.oid)) AS "total_size"
  FROM pg_class C
  LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
  WHERE nspname NOT IN ('pg_catalog', 'information_schema')
    AND C.relkind <> 'i'
    AND nspname !~ '^pg_toast'
  ORDER BY pg_total_relation_size(C.oid) DESC
  LIMIT 200;
  ```
  
### pg_size_pretty(), Human format of size
Basic usage example for pg_size_pretty()
```console
postgres=# SELECT pg_size_pretty(16384::bigint);
 pg_size_pretty 
----------------
 16 kB
(1 row)
```
### Find the size of all rows for a specific
```sql
select sum(pg_column_size(t) + 24) 
from the_table t
where customer_id = 42;
```
You can se more [here](https://dba.stackexchange.com/q/167106)

### Conceder privilegios de insert, select o update a un usuario en postgre sql
Sintaxe básica:
```sql
GRANT INSERT ON TABLE nome_da_tabela TO nome_do_usuário;
```
Exemplo:
```sql
GRANT INSERT ON TABLE orders TO user1;
```
### Conceder privielgios a una secuencia para un usuario en específico en posgresql
Sintaxe básica:
```sql
GRANT USAGE, SELECT, UPDATE ON SEQUENCE nome_da_sequencia TO nome_do_usuário;
```
Exemplo:
```sql
GRANT USAGE, SELECT, UPDATE ON SEQUENCE orders_seq TO user1;
```

### Lateral
See more [here](https://medium.com/kkempin/postgresqls-lateral-join-bfd6bd0199df)
```sql
SELECT user_id, first_order_time, next_order_time, id FROM
  (SELECT user_id, min(created_at) AS first_order_time FROM orders GROUP BY user_id) o1
  LEFT JOIN LATERAL
  (SELECT id, created_at AS next_order_time
   FROM orders
   WHERE user_id = o1.user_id AND created_at > o1.first_order_time
   ORDER BY created_at ASC LIMIT 1)
   o2 ON true;
```

Lateral con where.
```sql
SELECT user_id, first_order_time, next_order_time, id FROM
  o1
  LEFT OUTER JOIN LATERAL
  (SELECT id, created_at AS next_order_time
   FROM orders
   WHERE user_id = o1.user_id AND created_at > o1.first_order_time
   ORDER BY created_at ASC LIMIT 1)
   as o2 ON true
   where ...
```

### count(*)>1
```sql
select nu_persona_id, count(1) 
from persona 
group by 1 
having count(1)>1
```
