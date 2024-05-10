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
Donde:
```
start_index = (pagina_i - 1) * perPage + 1

end_index = pagina_i * perPage
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
### Test de paginación usando subconsultas:
A continuación muestro el codigo que tendrás que añadir a tu procedimento para dotarlo de paginación
```sql
PROCEDURE PAGINATION_TEST(
  IN_PAGES IN NUMBER, --NULL, [0 o más]
  IN_PER_PAGE IN NUMBER, --NULL, [0 o más]
  OUT_V_ERR_COD OUT VARCHAR2,
  OUT_V_ERR_MSG OUT VARCHAR2,
  OUT_PAGES OUT NUMBER,
  OUT_PER_PAGE OUT NUMBER,
  OUT_START_INDEX OUT NUMBER,
  OUT_END_INDEX OUT NUMBER
) AS
  V_PAGES NUMBER;
  V_PER_PAGE NUMBER;
  V_START_INDEX NUMBER;
  V_END_INDEX NUMBER;

BEGIN
  -- Validación de IN_PAGES
  IF IN_PAGES IS NULL OR IN_PAGES <= 0 THEN
    V_PAGES := 1; -- Valor por defecto
  ELSE
    V_PAGES := IN_PAGES;
  END IF;

  -- Validación de IN_PER_PAGE
  IF IN_PER_PAGE IS NULL OR IN_PER_PAGE <= 0 THEN
    V_PER_PAGE := 10; -- Valor por defecto
  ELSIF IN_PER_PAGE > 100 THEN
    V_PER_PAGE := 100; -- Límite máximo
  ELSE
    V_PER_PAGE := IN_PER_PAGE;
  END IF;

  -- Cálculo de los índices de inicio y fin
  V_START_INDEX := (V_PAGES - 1) * V_PER_PAGE + 1;
  V_END_INDEX := V_PAGES * V_PER_PAGE;

  -- Asignar códigos de éxito
  OUT_V_ERR_COD := V_V_COD_OK; -- Asumiendo que V_V_COD_OK es una variable definida previamente
  OUT_V_ERR_MSG := V_V_MSG_OK; -- Asumiendo que V_V_MSG_OK es una variable definida previamente

  OUT_PAGES := V_PAGES;
  OUT_PER_PAGE := V_PER_PAGE;
  OUT_START_INDEX := V_START_INDEX;
  OUT_END_INDEX := V_END_INDEX;

EXCEPTION
  WHEN OTHERS THEN
    -- En caso de error, asignar códigos de error y mensaje de error
    OUT_V_ERR_COD := '-1';
    OUT_V_ERR_MSG := 'Error al buscar registros: ' || SQLERRM;

    -- Realizar rollback en caso de error
    ROLLBACK;

END PAGINATION_TEST;
```
Resultados para los valores:

| IN_PAGES | IN_PER_PAGE | OUT_PAGES | OUT_PER_PAGE | OUT_START_INDEX | OUT_END_INDEX |
|----------|--------------|------------|---------------|-----------------|----------------|
| EMPTY    | EMPTY        | 1          | 10            | 1               | 10             |
| NULL     | NULL         | 1          | 10            | 1               | 10             |
| 0        | 10           | 1          | 10            | 1               | 10             |
| 1        | 10           | 1          | 10            | 1               | 10             |
| 2        | 10           | 2          | 10            | 11              | 20             |
| 3        | 10           | 3          | 10            | 21              | 30             |
| 4        | 10           | 4          | 10            | 31              | 40             |

la paginación debe completarse en una query como la siguiente:

```sql
SELECT * FROM (
    SELECT ROW_NUMBER() OVER (ORDER BY E.fecha_creacion DESC) AS secuencia,
           COUNT(*) OVER ()                                    AS cantidad,
           E.*
    FROM Estudiante E
) AS subconsulta
WHERE secuencia BETWEEN :V_START_INDEX AND :V_END_INDEX;
```

