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
