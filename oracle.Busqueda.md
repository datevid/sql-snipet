# Snipets para búsqueda

### 1. Mostrar secuencia a cada registro:
```sql
SELECT ROW_NUMBER() OVER (ORDER BY E.fecha_creacion DESC) AS secuencia,
       E.*
FROM Estudiante E;

```
### 2. Mostrar secuencia o número de fila o index en cada registro:
```sql
SELECT ROW_NUMBER() OVER (ORDER BY E.fecha_creacion DESC) AS secuencia,
       E.*
FROM Estudiante E;

```
### 3. Mostrar columna cantidad sin group by:
```sql
SELECT COUNT(*) OVER () AS "Cantidad",
       E.*
FROM Estudiante E;
```
### 4. Mostrar secuencia (índice) o cantidad sin group by:
```sql
SELECT ROW_NUMBER() OVER (ORDER BY E.fecha_creacion DESC) AS secuencia,
       COUNT(*) OVER ()                                    AS cantidad,
       E.*
FROM Estudiante E;

```
###  5. Paginación usando subconsultas:
```sql
SELECT * FROM (
    SELECT ROW_NUMBER() OVER (ORDER BY E.fecha_creacion DESC) AS secuencia,
           COUNT(*) OVER ()                                    AS cantidad,
           E.*
    FROM Estudiante E
) AS subconsulta
WHERE secuencia BETWEEN :start_index AND :end_index;
```
Ejemplo de paginación para la tabla estudiante:

```sql
select * from (
    SELECT ROW_NUMBER() OVER (ORDER BY t.fecha_creacion DESC) AS RNUM,
           t.*
    FROM estudiante t
) where rnum between :PI_START_ROW AND :PI_END_ROW;
```
ejemplo de paginacion usando subconsultas:

pagina 1| 10 por página
```
where secuencia between (1-1)*10+1 and 1*10;
```
pagina 2| 10 por página
```
where secuencia between (2-1)*10+1 and 2*10;
```
pagina 3| 10 por página
```
where secuencia between (3-1)*10+1 and 3*10;
```
Ecuación:

pagina pagina_i| perPage por página
```
where secuencia between (pagina_i - 1) * perPage + 1 and pagina_i * perPage;
```
start_index = (pagina_i - 1) * perPage + 1

end_index = pagina_i * perPage

### 6. Paginación sin usar subconsultas:

```sql
SELECT ROW_NUMBER() OVER (ORDER BY E.fecha_creacion DESC) AS secuencia,
       COUNT(*) OVER ()                                    AS cantidad,
       E.*
FROM Estudiante E
OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY;
```
ejemplo del offet:

pagina 1| 10 por página
```
offset (1-1)*10 rows fetch next 10 rows only;
```
pagina 2| 10 por página
```
offset (2-1)*10 rows fetch next 10 rows only;
```
pagina 3| 10 por página
```
offset (3-1)*10 rows fetch next 10 rows only;
```
Ecuación:

pagina page_i| perPage por página
```
offset (pagina_i-1)*perPage rows fetch next perPage rows only;
```




