Version oracle:

```sql
SELECT * FROM v$version
```

Consulta del número de versión simplificada:
```sql
SELECT version FROM v$instance
```

Paginación:
```sql
SELECT *
FROM (
  SELECT t.*, ROW_NUMBER() OVER (ORDER BY columna_orden) AS rn
  FROM tu_tabla t
)
WHERE rn BETWEEN :start_index AND :end_index
```
Ejemplo de paginación para la tabla estudiante:
```sql
select * from (
    SELECT ROW_NUMBER() OVER (ORDER BY t.FE_D_CREACION DESC) AS RNUM,
           t.*
    FROM estudiante t
    order by t.FE_D_CREACION desc
) where rnum between :PI_START_ROW AND :PI_END_ROW;
```
