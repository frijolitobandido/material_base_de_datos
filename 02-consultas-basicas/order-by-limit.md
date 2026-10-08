# Cláusulas `ORDER BY` y `LIMIT`

`ORDER BY` y `LIMIT` permiten organizar y controlar la cantidad de filas que devuelve una consulta.

- `ORDER BY` ordena los resultados.
- `ASC` ordena de menor a mayor o de forma ascendente.
- `DESC` ordena de mayor a menor o de forma descendente.
- `LIMIT` restringe cuántas filas se devuelven.
- `OFFSET` permite saltar una cantidad de filas antes de comenzar a devolver resultados.

```sql
SELECT columnas
FROM tabla
WHERE condicion
ORDER BY columna ASC
LIMIT cantidad;
```

> **Idea central.** `ORDER BY` decide el orden visible de los resultados y `LIMIT` decide cuántos resultados se muestran.

---

## 1. Tabla de ejemplo

Para practicar usaremos la tabla `productos`.

```sql
CREATE TABLE productos (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    categoria VARCHAR(50),
    precio DECIMAL(10,2),
    stock INT,
    activo BOOLEAN
);
```

```sql
INSERT INTO productos (id, nombre, categoria, precio, stock, activo)
VALUES
    (1, 'Teclado mecánico', 'periféricos', 149.90, 12, TRUE),
    (2, 'Mouse inalámbrico', 'periféricos', 79.90, 25, TRUE),
    (3, 'Monitor 24 pulgadas', 'monitores', 899.00, 8, TRUE),
    (4, 'Webcam HD', 'cámaras', 219.50, 0, FALSE),
    (5, 'Alfombrilla grande', 'periféricos', 45.00, 40, TRUE);
```

| id | nombre | categoria | precio | stock | activo |
| ---: | :--- | :--- | ---: | ---: | :---: |
| 1 | Teclado mecánico | periféricos | 149.90 | 12 | `TRUE` |
| 2 | Mouse inalámbrico | periféricos | 79.90 | 25 | `TRUE` |
| 3 | Monitor 24 pulgadas | monitores | 899.00 | 8 | `TRUE` |
| 4 | Webcam HD | cámaras | 219.50 | 0 | `FALSE` |
| 5 | Alfombrilla grande | periféricos | 45.00 | 40 | `TRUE` |

---

## 2. Ordenar con `ORDER BY`

La sintaxis básica es:

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio;
```

Cuando no se escribe `ASC` ni `DESC`, muchos motores utilizan `ASC` por defecto. Sin embargo, escribirlo explícitamente hace más clara la intención.

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio ASC;
```

### Resultado ascendente

| nombre | precio |
| :--- | ---: |
| Alfombrilla grande | 45.00 |
| Mouse inalámbrico | 79.90 |
| Teclado mecánico | 149.90 |
| Webcam HD | 219.50 |
| Monitor 24 pulgadas | 899.00 |

El `ORDER BY` no modifica el orden físico de la tabla. Solo organiza el resultado de esa consulta.

> **Qué debes recordar.** Una tabla no debe considerarse ordenada permanentemente. Si necesitas un orden, escribe `ORDER BY` en la consulta.

---

## 3. Orden descendente con `DESC`

`DESC` organiza de mayor a menor, de más reciente a más antiguo o de la Z a la A, según el tipo de dato.

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio DESC;
```

### Resultado descendente

| nombre | precio |
| :--- | ---: |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |
| Teclado mecánico | 149.90 |
| Mouse inalámbrico | 79.90 |
| Alfombrilla grande | 45.00 |

### Comparación rápida

| Orden | Primer resultado | Último resultado |
| :--- | :--- | :--- |
| `ASC` | Alfombrilla grande — 45.00 | Monitor — 899.00 |
| `DESC` | Monitor — 899.00 | Alfombrilla grande — 45.00 |

---

## 4. Ordenar texto

Los textos se ordenan alfabéticamente según la configuración de comparación del motor.

```sql
SELECT nombre, categoria
FROM productos
ORDER BY nombre ASC;
```

### Resultado

| nombre | categoria |
| :--- | :--- |
| Alfombrilla grande | periféricos |
| Monitor 24 pulgadas | monitores |
| Mouse inalámbrico | periféricos |
| Teclado mecánico | periféricos |
| Webcam HD | cámaras |

### Orden descendente de texto

```sql
SELECT nombre, categoria
FROM productos
ORDER BY nombre DESC;
```

| nombre | categoria |
| :--- | :--- |
| Webcam HD | cámaras |
| Teclado mecánico | periféricos |
| Mouse inalámbrico | periféricos |
| Monitor 24 pulgadas | monitores |
| Alfombrilla grande | periféricos |

Mayúsculas, minúsculas, acentos y caracteres especiales pueden ordenarse de manera diferente según la collation o configuración del motor.

---

## 5. Ordenar fechas

En una tabla de pedidos, `ORDER BY` permite consultar desde el pedido más antiguo o desde el más reciente.

```sql
CREATE TABLE pedidos (
    id INT PRIMARY KEY,
    cliente VARCHAR(100),
    fecha_pedido DATE,
    total DECIMAL(10,2)
);
```

```sql
INSERT INTO pedidos (id, cliente, fecha_pedido, total)
VALUES
    (101, 'Ana', '2026-10-01', 45.90),
    (102, 'Luis', '2026-10-04', 80.00),
    (103, 'Marta', '2026-10-08', 25.50);
```

### Más recientes primero

```sql
SELECT id, cliente, fecha_pedido
FROM pedidos
ORDER BY fecha_pedido DESC;
```

| id | cliente | fecha_pedido |
| ---: | :--- | :--- |
| 103 | Marta | 2026-10-08 |
| 102 | Luis | 2026-10-04 |
| 101 | Ana | 2026-10-01 |

### Más antiguos primero

```sql
SELECT id, cliente, fecha_pedido
FROM pedidos
ORDER BY fecha_pedido ASC;
```

| id | cliente | fecha_pedido |
| ---: | :--- | :--- |
| 101 | Ana | 2026-10-01 |
| 102 | Luis | 2026-10-04 |
| 103 | Marta | 2026-10-08 |

> **Regla práctica.** Para mostrar “lo más reciente”, normalmente se utiliza `ORDER BY fecha DESC`.

---

## 6. Ordenar por varias columnas

Se pueden indicar varios criterios separados por comas.

```sql
SELECT categoria, nombre, precio
FROM productos
ORDER BY categoria ASC, precio DESC;
```

El motor ordena primero por `categoria`. Si dos filas tienen la misma categoría, utiliza `precio DESC` como segundo criterio.

### Resultado simulado

| categoria | nombre | precio |
| :--- | :--- | ---: |
| cámaras | Webcam HD | 219.50 |
| monitores | Monitor 24 pulgadas | 899.00 |
| periféricos | Teclado mecánico | 149.90 |
| periféricos | Mouse inalámbrico | 79.90 |
| periféricos | Alfombrilla grande | 45.00 |

### Ejemplo con empate

```sql
SELECT categoria, nombre, precio
FROM productos
ORDER BY categoria ASC, precio ASC;
```

| categoria | nombre | precio |
| :--- | :--- | ---: |
| cámaras | Webcam HD | 219.50 |
| monitores | Monitor 24 pulgadas | 899.00 |
| periféricos | Alfombrilla grande | 45.00 |
| periféricos | Mouse inalámbrico | 79.90 |
| periféricos | Teclado mecánico | 149.90 |

### Prioridad de criterios

| Prioridad | Columna | Orden |
| ---: | :--- | :--- |
| 1 | `categoria` | Ascendente |
| 2 | `precio` | Ascendente |

> **Importante.** El segundo criterio solo se usa para ordenar las filas que empatan en el primero.

---

## 7. Usar `ORDER BY` con `WHERE`

El orden habitual coloca `WHERE` antes de `ORDER BY`.

```sql
SELECT nombre, precio, stock
FROM productos
WHERE activo = TRUE
ORDER BY precio DESC;
```

### Resultado

| nombre | precio | stock |
| :--- | ---: | ---: |
| Monitor 24 pulgadas | 899.00 | 8 |
| Teclado mecánico | 149.90 | 12 |
| Mouse inalámbrico | 79.90 | 25 |
| Alfombrilla grande | 45.00 | 40 |

Primero se excluye la webcam porque está inactiva y después se ordenan los productos restantes por precio.

### Orden de escritura

```sql
SELECT columnas
FROM tabla
WHERE condicion
ORDER BY columna ASC;
```

| Paso lógico | Operación |
| ---: | :--- |
| 1 | Obtener filas de `productos` |
| 2 | Filtrar las activas |
| 3 | Ordenar por precio descendente |
| 4 | Mostrar las columnas solicitadas |

---

## 8. `LIMIT`: limitar cantidad de filas

`LIMIT` indica cuántas filas como máximo se devuelven.

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio DESC
LIMIT 3;
```

### Resultado

| nombre | precio |
| :--- | ---: |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |
| Teclado mecánico | 149.90 |

Aunque la tabla tiene cinco productos, solo se devuelven los tres primeros después de ordenar.

> **Orden recomendado.** Si usas `LIMIT`, utiliza también `ORDER BY` para definir qué filas deben ocupar los primeros lugares.

### ¿Qué ocurre sin `ORDER BY`?

```sql
SELECT nombre, precio
FROM productos
LIMIT 3;
```

El motor puede devolver tres filas, pero no debes asumir cuáles serán ni en qué orden. Sin `ORDER BY`, el orden no está garantizado.

---

## 9. `LIMIT` con `WHERE`

`WHERE` reduce las filas posibles y `LIMIT` limita el resultado final.

```sql
SELECT nombre, precio
FROM productos
WHERE activo = TRUE
ORDER BY precio DESC
LIMIT 2;
```

### Resultado

| nombre | precio |
| :--- | ---: |
| Monitor 24 pulgadas | 899.00 |
| Teclado mecánico | 149.90 |

El resultado corresponde a los dos productos activos más caros.

### Interpretación

| Parte | Función |
| :--- | :--- |
| `WHERE activo = TRUE` | Elimina productos inactivos |
| `ORDER BY precio DESC` | Coloca primero los más caros |
| `LIMIT 2` | Conserva solo los dos primeros |

---

## 10. `OFFSET`: saltar filas

`OFFSET` indica cuántas filas se deben saltar antes de comenzar a devolver resultados.

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio ASC
LIMIT 2 OFFSET 2;
```

Orden completo por precio ascendente:

| Posición | nombre | precio |
| ---: | :--- | ---: |
| 1 | Alfombrilla grande | 45.00 |
| 2 | Mouse inalámbrico | 79.90 |
| 3 | Teclado mecánico | 149.90 |
| 4 | Webcam HD | 219.50 |
| 5 | Monitor 24 pulgadas | 899.00 |

`OFFSET 2` salta las posiciones 1 y 2; `LIMIT 2` devuelve las posiciones 3 y 4.

### Resultado

| nombre | precio |
| :--- | ---: |
| Teclado mecánico | 149.90 |
| Webcam HD | 219.50 |

### Sintaxis alternativa frecuente

En MySQL y PostgreSQL también es común escribir:

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio ASC
LIMIT 2, 2;
```

En esta forma, el primer número es el desplazamiento y el segundo es la cantidad.

| Forma | Desplazamiento | Cantidad |
| :--- | ---: | ---: |
| `LIMIT 2 OFFSET 2` | 2 | 2 |
| `LIMIT 2, 2` | 2 | 2 |

> **Portabilidad.** `LIMIT ... OFFSET ...` es una forma clara y ampliamente utilizada, pero algunos motores emplean otras sintaxis, como `TOP` o `FETCH FIRST`.

---

## 11. Paginación

La paginación divide un conjunto grande de resultados en páginas.

Supongamos que cada página muestra dos productos:

```sql
-- Página 1
SELECT id, nombre, precio
FROM productos
ORDER BY id ASC
LIMIT 2 OFFSET 0;

-- Página 2
SELECT id, nombre, precio
FROM productos
ORDER BY id ASC
LIMIT 2 OFFSET 2;

-- Página 3
SELECT id, nombre, precio
FROM productos
ORDER BY id ASC
LIMIT 2 OFFSET 4;
```

### Resultados por página

| Página | `LIMIT` | `OFFSET` | Registros |
| ---: | ---: | ---: | :--- |
| 1 | 2 | 0 | IDs 1 y 2 |
| 2 | 2 | 2 | IDs 3 y 4 |
| 3 | 2 | 4 | ID 5 |

### Fórmula del desplazamiento

```text
OFFSET = (número_de_página - 1) * tamaño_de_página
```

| Página | Tamaño | Cálculo | `OFFSET` |
| ---: | ---: | :--- | ---: |
| 1 | 2 | `(1 - 1) * 2` | 0 |
| 2 | 2 | `(2 - 1) * 2` | 2 |
| 3 | 2 | `(3 - 1) * 2` | 4 |

> **Problema de paginación.** Si los datos cambian entre una página y otra, pueden aparecer duplicados o faltar filas. Para resultados estables, ordena con una columna consistente, normalmente una clave única.

---

## 12. Orden estable con una columna secundaria

Si varias filas tienen el mismo valor de orden, agrega un segundo criterio.

```sql
SELECT nombre, categoria, precio, id
FROM productos
ORDER BY categoria ASC, precio DESC, id ASC;
```

El `id` funciona como criterio final para resolver empates.

| Prioridad | Criterio | Propósito |
| ---: | :--- | :--- |
| 1 | `categoria ASC` | Agrupar categorías |
| 2 | `precio DESC` | Ordenar el precio dentro de cada categoría |
| 3 | `id ASC` | Resolver precios idénticos |

### Por qué importa

| Situación | Riesgo sin criterio secundario | Solución |
| :--- | :--- | :--- |
| Dos productos con igual precio | El orden puede variar | Agregar `id` |
| Paginación | Una fila puede cambiar de página | Usar orden determinista |
| Reportes repetibles | El resultado puede no ser idéntico | Definir todos los empates |

---

## 13. Ordenar por alias o expresión

Un alias de `SELECT` puede utilizarse en `ORDER BY` en muchos motores.

```sql
SELECT
    nombre,
    precio * 1.18 AS precio_con_impuesto
FROM productos
ORDER BY precio_con_impuesto DESC;
```

### Resultado

| nombre | precio_con_impuesto |
| :--- | ---: |
| Monitor 24 pulgadas | 1060.82 |
| Webcam HD | 259.01 |
| Teclado mecánico | 176.88 |
| Mouse inalámbrico | 94.28 |
| Alfombrilla grande | 53.10 |

También se puede ordenar directamente por la expresión:

```sql
SELECT
    nombre,
    precio * 1.18 AS precio_con_impuesto
FROM productos
ORDER BY precio * 1.18 DESC;
```

> **Recomendación.** Usar el alias suele hacer la consulta más legible, pero revisa la compatibilidad si escribes SQL para varios motores.

---

## 14. Ordenar por posición de columna

Algunos motores permiten ordenar usando la posición de la columna en `SELECT`.

```sql
SELECT nombre, precio, stock
FROM productos
ORDER BY 2 DESC;
```

El número `2` representa `precio`.

| Posición | Columna |
| ---: | :--- |
| 1 | `nombre` |
| 2 | `precio` |
| 3 | `stock` |

Aunque funciona en muchos motores, es más claro escribir el nombre:

```sql
ORDER BY precio DESC;
```

Si se cambia el orden de las columnas en `SELECT`, `ORDER BY 2` podría empezar a ordenar otra columna.

> **Regla práctica.** Prefiere nombres de columnas o alias antes que posiciones numéricas en consultas que deban mantenerse.

---

## 15. `NULL` en `ORDER BY`

El orden de los valores `NULL` puede variar entre motores y según se use `ASC` o `DESC`.

```sql
SELECT nombre, fecha_vencimiento
FROM productos_con_vencimiento
ORDER BY fecha_vencimiento ASC;
```

| Motor o configuración | Comportamiento de `NULL` |
| :--- | :--- |
| Algunos motores en `ASC` | Colocan `NULL` primero |
| Otros comportamientos | Colocan `NULL` al final |
| `DESC` | Puede invertir la posición según el motor |

Para controlar el comportamiento, se puede ordenar primero por una expresión:

```sql
SELECT nombre, fecha_vencimiento
FROM productos_con_vencimiento
ORDER BY
    CASE WHEN fecha_vencimiento IS NULL THEN 1 ELSE 0 END,
    fecha_vencimiento ASC;
```

Así los valores con fecha aparecen antes y los `NULL` después.

> **Compatibilidad.** La forma exacta de usar `NULLS FIRST` o `NULLS LAST` depende del motor. PostgreSQL, por ejemplo, permite escribirlas explícitamente.

---

## 16. Diferencia entre `LIMIT` y filtrar

`LIMIT` no decide qué filas cumplen una condición; solo limita cuántas se muestran después del orden.

```sql
SELECT nombre, precio
FROM productos
WHERE precio > 100
ORDER BY precio DESC
LIMIT 2;
```

| Paso | Cantidad o resultado |
| ---: | :--- |
| Filas iniciales | 5 |
| Filas que cumplen `precio > 100` | 3 |
| Filas ordenadas por precio descendente | 3 |
| Filas devueltas por `LIMIT 2` | 2 |

Resultado:

| nombre | precio |
| :--- | ---: |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |

---

## 17. `ORDER BY` con `DISTINCT`

Cuando se usa `DISTINCT`, el orden debe ser compatible con las columnas seleccionadas según las reglas del motor.

```sql
SELECT DISTINCT categoria
FROM productos
ORDER BY categoria ASC;
```

### Resultado

| categoria |
| :--- |
| cámaras |
| monitores |
| periféricos |

Una consulta como esta es clara porque ordena por la misma columna que devuelve.

```sql
SELECT DISTINCT categoria, activo
FROM productos
ORDER BY categoria ASC, activo DESC;
```

| categoria | activo |
| :--- | :---: |
| cámaras | `FALSE` |
| monitores | `TRUE` |
| periféricos | `TRUE` |

---

## 18. Orden lógico de una consulta

Una consulta con filtro, orden y límite suele escribirse así:

```sql
SELECT columnas
FROM tabla
WHERE condicion
ORDER BY columna ASC
LIMIT cantidad OFFSET desplazamiento;
```

| Cláusula | Propósito |
| :--- | :--- |
| `SELECT` | Define las columnas o expresiones |
| `FROM` | Define el origen |
| `WHERE` | Filtra filas |
| `ORDER BY` | Ordena el resultado |
| `LIMIT` | Limita la cantidad |
| `OFFSET` | Salta filas antes de devolverlas |

### Ejemplo completo

```sql
SELECT id, nombre, precio
FROM productos
WHERE activo = TRUE
ORDER BY precio DESC, id ASC
LIMIT 2 OFFSET 0;
```

| id | nombre | precio |
| ---: | :--- | ---: |
| 3 | Monitor 24 pulgadas | 899.00 |
| 1 | Teclado mecánico | 149.90 |

---

## 19. Errores comunes

| Error | Qué ocurre | Cómo evitarlo |
| :--- | :--- | :--- |
| Usar `LIMIT` sin `ORDER BY` | No se sabe qué filas serán devueltas | Definir un orden explícito |
| Confundir `ASC` con `DESC` | Los resultados aparecen al revés | Revisar si se necesita menor a mayor o mayor a menor |
| Escribir `LIMIT` antes de `ORDER BY` | La sintaxis es incorrecta | Respetar el orden de las cláusulas |
| Olvidar el segundo criterio | Los empates pueden cambiar de posición | Agregar una columna única como `id` |
| Usar `OFFSET` sin un orden estable | Las páginas pueden cambiar | Ordenar por una clave consistente |
| Confundir `LIMIT 2 OFFSET 4` | Se saltan o devuelven filas incorrectas | Recordar: primero cantidad, luego desplazamiento |
| Ordenar por posición numérica | Cambia el significado al modificar `SELECT` | Preferir nombres o alias |
| Suponer el orden de `NULL` | Cada motor puede comportarse distinto | Controlarlo explícitamente |
| Creer que `ORDER BY` modifica la tabla | Solo cambia el resultado de la consulta | Usar `UPDATE` para modificar datos |

### Ejemplo incorrecto y corregido

```sql
-- Incompleto: devuelve dos filas no deterministas
SELECT *
FROM productos
LIMIT 2;
```

```sql
-- Correcto: devuelve los dos productos más caros
SELECT id, nombre, precio
FROM productos
ORDER BY precio DESC, id ASC
LIMIT 2;
```

---

## 20. Práctica guiada

### Pregunta 1: ordenar precios

Muestra todos los productos ordenados del más barato al más caro.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio ASC;
```

</details>

### Pregunta 2: productos más caros

Muestra los tres productos de mayor precio.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio DESC
LIMIT 3;
```

</details>

### Pregunta 3: productos activos más caros

Muestra los dos productos activos con mayor precio.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, precio, activo
FROM productos
WHERE activo = TRUE
ORDER BY precio DESC
LIMIT 2;
```

</details>

### Pregunta 4: segunda página

Muestra la segunda página de productos, usando dos productos por página y ordenando por `id`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT id, nombre
FROM productos
ORDER BY id ASC
LIMIT 2 OFFSET 2;
```

</details>

### Pregunta 5: varios criterios

Ordena por categoría ascendente y, dentro de cada categoría, por stock descendente.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT categoria, nombre, stock
FROM productos
ORDER BY categoria ASC, stock DESC;
```

</details>

---

## 21. Mini desafío final

Usa la tabla `productos` y escribe consultas para obtener:

1. Los tres productos más baratos.
2. Los productos activos ordenados por stock de mayor a menor.
3. La segunda página de productos, con tres productos por página, ordenados por precio ascendente.
4. Los productos ordenados por categoría y luego por nombre.
5. Los dos productos con mayor valor de inventario (`precio * stock`).

### Una solución posible

```sql
-- 1. Tres productos más baratos
SELECT nombre, precio
FROM productos
ORDER BY precio ASC
LIMIT 3;

-- 2. Productos activos por stock descendente
SELECT nombre, stock, activo
FROM productos
WHERE activo = TRUE
ORDER BY stock DESC, id ASC;

-- 3. Segunda página, tres productos por página
SELECT id, nombre, precio
FROM productos
ORDER BY precio ASC, id ASC
LIMIT 3 OFFSET 3;

-- 4. Categoría y nombre
SELECT categoria, nombre
FROM productos
ORDER BY categoria ASC, nombre ASC;

-- 5. Mayor valor de inventario
SELECT
    nombre,
    precio,
    stock,
    precio * stock AS valor_inventario
FROM productos
ORDER BY valor_inventario DESC, id ASC
LIMIT 2;
```

### Resultado esperado de la consulta 5

| nombre | precio | stock | valor_inventario |
| :--- | ---: | ---: | ---: |
| Monitor 24 pulgadas | 899.00 | 8 | 7192.00 |
| Mouse inalámbrico | 79.90 | 25 | 1997.50 |

---

## 22. Resumen final

- `ORDER BY` ordena las filas del resultado.
- `ASC` ordena ascendentemente y suele ser el comportamiento predeterminado.
- `DESC` ordena descendentemente.
- Se pueden combinar varios criterios de orden.
- Un criterio secundario resuelve empates del criterio principal.
- `LIMIT` restringe la cantidad de filas devueltas.
- `OFFSET` salta filas antes de devolver resultados.
- La paginación utiliza normalmente `LIMIT` y `OFFSET`.
- Para paginar de manera estable se necesita un orden determinista.
- `LIMIT` sin `ORDER BY` no garantiza qué filas serán devueltas.
- El orden de `NULL` depende del motor y puede controlarse explícitamente.
- `ORDER BY` organiza el resultado, pero no modifica la tabla.

> **Qué debes recordar.** Primero filtra con `WHERE` si es necesario, después ordena con `ORDER BY` y finalmente limita con `LIMIT`.

---

## Relacionado con otros temas

- [`SELECT` y `FROM`](select-from.md): forman la base de estas consultas.
- [WHERE y operadores](where-operadores.md): filtran las filas antes de ordenarlas.
- [Funciones SQL](funciones-sql.md): permiten ordenar por valores calculados.
- [Agregaciones y `GROUP BY`](../03-consultas-relacionales/agregaciones-group-by.md): permiten ordenar resultados agrupados.
- [Índices básicos](../06-indices-performance/indices-basicos.md): pueden ayudar a ordenar y filtrar con mayor eficiencia.
