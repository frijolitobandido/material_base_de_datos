# Guía de Teoría y Repaso: `SELECT` y `FROM`

`SELECT` y `FROM` forman la estructura mínima para consultar información en SQL.

- `SELECT` indica qué columnas o expresiones queremos obtener.
- `FROM` indica de qué tabla o tablas se obtendrán los datos.

La consulta más sencilla tiene esta forma:

```sql
SELECT columna1, columna2
FROM nombre_tabla;
```

> **Idea central.** `FROM` define el origen de los datos y `SELECT` define qué parte de ese origen queremos visualizar.

---

## 1. La tabla de ejemplo

Para practicar usaremos una tabla llamada `productos`.

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

### Datos de ejemplo

```sql
INSERT INTO productos (id, nombre, categoria, precio, stock, activo)
VALUES
    (1, 'Teclado mecánico', 'periféricos', 149.90, 12, TRUE),
    (2, 'Mouse inalámbrico', 'periféricos', 79.90, 25, TRUE),
    (3, 'Monitor 24 pulgadas', 'monitores', 899.00, 8, TRUE),
    (4, 'Webcam HD', 'cámaras', 219.50, 0, FALSE),
    (5, 'Alfombrilla grande', 'periféricos', 45.00, 40, TRUE);
```

### Estado inicial de la tabla

| id | nombre | categoria | precio | stock | activo |
| ---: | :--- | :--- | ---: | ---: | :---: |
| 1 | Teclado mecánico | periféricos | 149.90 | 12 | `TRUE` |
| 2 | Mouse inalámbrico | periféricos | 79.90 | 25 | `TRUE` |
| 3 | Monitor 24 pulgadas | monitores | 899.00 | 8 | `TRUE` |
| 4 | Webcam HD | cámaras | 219.50 | 0 | `FALSE` |
| 5 | Alfombrilla grande | periféricos | 45.00 | 40 | `TRUE` |

---

## 2. Consultar todas las columnas con `SELECT *`

El asterisco significa “todas las columnas”.

```sql
SELECT *
FROM productos;
```

### Resultado simulado

| id | nombre | categoria | precio | stock | activo |
| ---: | :--- | :--- | ---: | ---: | :---: |
| 1 | Teclado mecánico | periféricos | 149.90 | 12 | `TRUE` |
| 2 | Mouse inalámbrico | periféricos | 79.90 | 25 | `TRUE` |
| 3 | Monitor 24 pulgadas | monitores | 899.00 | 8 | `TRUE` |
| 4 | Webcam HD | cámaras | 219.50 | 0 | `FALSE` |
| 5 | Alfombrilla grande | periféricos | 45.00 | 40 | `TRUE` |

### ¿Cuándo conviene usar `*`?

| Situación | ¿Conviene `SELECT *`? | Motivo |
| :--- | :---: | :--- |
| Explorar una tabla durante el aprendizaje | Sí | Permite observar todas las columnas |
| Revisar rápidamente algunos registros | Puede servir | Es una consulta corta |
| Crear una consulta de producción | Generalmente no | Puede traer columnas innecesarias |
| Crear una vista estable | No suele ser recomendable | La vista puede cambiar si se agregan columnas |
| Consultar muchas columnas o datos pesados | No | Puede aumentar el trabajo y el tráfico |

> **Regla práctica.** Usa `SELECT *` para explorar; en consultas definitivas, indica las columnas que realmente necesitas.

---

## 3. Seleccionar columnas específicas

En lugar de solicitar todas las columnas, podemos elegir únicamente las que queremos ver.

```sql
SELECT nombre, precio
FROM productos;
```

### Resultado

| nombre | precio |
| :--- | ---: |
| Teclado mecánico | 149.90 |
| Mouse inalámbrico | 79.90 |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |
| Alfombrilla grande | 45.00 |

La tabla original tiene seis columnas, pero la consulta devuelve únicamente dos.

### Orden de las columnas

El orden escrito en `SELECT` determina el orden del resultado.

```sql
SELECT precio, nombre, stock
FROM productos;
```

| precio | nombre | stock |
| ---: | :--- | ---: |
| 149.90 | Teclado mecánico | 12 |
| 79.90 | Mouse inalámbrico | 25 |
| 899.00 | Monitor 24 pulgadas | 8 |
| 219.50 | Webcam HD | 0 |
| 45.00 | Alfombrilla grande | 40 |

El orden de las columnas en el resultado no modifica la tabla original.

---

## 4. Consultar una sola columna

También es posible devolver una única columna.

```sql
SELECT categoria
FROM productos;
```

### Resultado

| categoria |
| :--- |
| periféricos |
| periféricos |
| monitores |
| cámaras |
| periféricos |

Los valores repetidos aparecen porque cada fila de la tabla se mantiene en el resultado.

> **Importante.** `SELECT` no elimina duplicados automáticamente. Para eso existe `DISTINCT`.

---

## 5. `DISTINCT`: eliminar repeticiones del resultado

`DISTINCT` devuelve únicamente combinaciones diferentes de las columnas seleccionadas.

```sql
SELECT DISTINCT categoria
FROM productos;
```

### Resultado

| categoria |
| :--- |
| periféricos |
| monitores |
| cámaras |

### `DISTINCT` con varias columnas

```sql
SELECT DISTINCT categoria, activo
FROM productos;
```

### Resultado simulado

| categoria | activo |
| :--- | :---: |
| periféricos | `TRUE` |
| monitores | `TRUE` |
| cámaras | `FALSE` |

`DISTINCT` compara la combinación completa de las columnas, no cada columna por separado.

| categoria | activo | ¿Es otra combinación? |
| :--- | :---: | :---: |
| periféricos | `TRUE` | Sí, primera aparición |
| periféricos | `TRUE` | No, se repite la combinación |
| cámaras | `FALSE` | Sí, cambia la categoría |
| cámaras | `TRUE` | Sí, cambia el activo |

---

## 6. Alias para columnas con `AS`

Un alias cambia el nombre que aparece en el resultado sin cambiar el nombre real de la columna.

```sql
SELECT
    nombre AS producto,
    precio AS precio_unitario
FROM productos;
```

### Resultado

| producto | precio_unitario |
| :--- | ---: |
| Teclado mecánico | 149.90 |
| Mouse inalámbrico | 79.90 |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |
| Alfombrilla grande | 45.00 |

### Alias de la tabla

También puede asignarse un alias a la tabla:

```sql
SELECT p.nombre, p.precio
FROM productos AS p;
```

Después de asignar `p`, se puede escribir `p.nombre` y `p.precio`.

### ¿Por qué usar alias?

| Situación | Ventaja |
| :--- | :--- |
| Nombre largo de tabla | Reduce la cantidad de texto |
| Varias tablas | Evita ambigüedades entre columnas |
| Consultas con expresiones | Hace más claro el resultado |
| Reportes | Permite mostrar nombres comprensibles |

La palabra `AS` suele ser opcional para alias de tablas:

```sql
SELECT p.nombre
FROM productos p;
```

Sin embargo, escribir `AS` puede hacer el código más claro para quien está aprendiendo.

---

## 7. Expresiones dentro de `SELECT`

`SELECT` no solo devuelve columnas. También puede calcular expresiones.

### Aumentar un precio en el resultado

```sql
SELECT
    nombre,
    precio,
    precio * 1.18 AS precio_con_impuesto
FROM productos;
```

Esta consulta calcula un nuevo valor, pero no modifica la columna `precio`.

### Resultado simulado

| nombre | precio | precio_con_impuesto |
| :--- | ---: | ---: |
| Teclado mecánico | 149.90 | 176.88 |
| Mouse inalámbrico | 79.90 | 94.28 |
| Monitor 24 pulgadas | 899.00 | 1060.82 |
| Webcam HD | 219.50 | 259.01 |
| Alfombrilla grande | 45.00 | 53.10 |

> **Diferencia importante.** Un cálculo dentro de `SELECT` solo cambia el resultado mostrado. Para cambiar la tabla se necesita `UPDATE`.

### Calcular el valor del inventario

```sql
SELECT
    nombre,
    precio,
    stock,
    precio * stock AS valor_inventario
FROM productos;
```

### Resultado simulado

| nombre | precio | stock | valor_inventario |
| :--- | ---: | ---: | ---: |
| Teclado mecánico | 149.90 | 12 | 1798.80 |
| Mouse inalámbrico | 79.90 | 25 | 1997.50 |
| Monitor 24 pulgadas | 899.00 | 8 | 7192.00 |
| Webcam HD | 219.50 | 0 | 0.00 |
| Alfombrilla grande | 45.00 | 40 | 1800.00 |

---

## 8. Valores literales en `SELECT`

Es posible mostrar texto o valores fijos junto con las columnas.

```sql
SELECT
    nombre,
    'Producto' AS tipo_registro,
    precio
FROM productos;
```

### Resultado

| nombre | tipo_registro | precio |
| :--- | :--- | ---: |
| Teclado mecánico | Producto | 149.90 |
| Mouse inalámbrico | Producto | 79.90 |
| Monitor 24 pulgadas | Producto | 899.00 |
| Webcam HD | Producto | 219.50 |
| Alfombrilla grande | Producto | 45.00 |

El texto `'Producto'` no proviene de una columna; es un valor literal repetido en cada fila del resultado.

### Mostrar una constante

```sql
SELECT
    nombre,
    18 AS porcentaje_impuesto
FROM productos;
```

| nombre | porcentaje_impuesto |
| :--- | ---: |
| Teclado mecánico | 18 |
| Mouse inalámbrico | 18 |
| Monitor 24 pulgadas | 18 |
| Webcam HD | 18 |
| Alfombrilla grande | 18 |

---

## 9. Valores `NULL` en el resultado

`NULL` representa ausencia de valor, no necesariamente cero ni una cadena vacía.

```sql
CREATE TABLE empleados (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    telefono VARCHAR(30)
);

INSERT INTO empleados (id, nombre, telefono)
VALUES
    (1, 'Ana', '555-1000'),
    (2, 'Luis', NULL);
```

```sql
SELECT nombre, telefono
FROM empleados;
```

### Resultado

| nombre | telefono |
| :--- | :--- |
| Ana | 555-1000 |
| Luis | `NULL` |

En esta guía solo mostramos `NULL`. La forma correcta de filtrarlo con `IS NULL` se estudiará en `where-operadores.md`.

> **No lo confundas.** `NULL` no es igual a `0`, `''` ni `'NULL'` como texto.

---

## 10. Consultar más de una tabla con `FROM`

`FROM` también puede indicar más de una tabla. Sin una condición de relación, esto produce un producto cartesiano.

```sql
SELECT
    clientes.nombre AS cliente,
    pedidos.id AS pedido
FROM clientes, pedidos;
```

Si hay 2 clientes y 3 pedidos, el resultado puede tener 6 combinaciones.

| Cantidad de clientes | Cantidad de pedidos | Filas del producto cartesiano |
| ---: | ---: | ---: |
| 2 | 3 | 6 |

Para relacionar tablas de forma controlada se utiliza `JOIN`, que se estudiará en consultas relacionales.

La forma moderna y recomendada para una relación es:

```sql
SELECT
    c.nombre,
    p.id
FROM clientes AS c
JOIN pedidos AS p
    ON p.cliente_id = c.id;
```

> **Advertencia.** Evita listar varias tablas separadas por comas si todavía no conoces el producto cartesiano. Una relación incompleta puede generar resultados duplicados o incorrectos.

---

## 11. Orden lógico de una consulta básica

Aunque escribimos primero `SELECT`, el motor necesita identificar el origen mediante `FROM`.

En una consulta sencilla:

```sql
SELECT nombre, precio
FROM productos;
```

| Parte | Función |
| :--- | :--- |
| `SELECT` | Indica qué devolver |
| `nombre, precio` | Columnas seleccionadas |
| `FROM` | Indica el origen |
| `productos` | Tabla consultada |
| `;` | Final de la instrucción |

Cuando agreguemos más cláusulas, el orden de escritura habitual será:

```sql
SELECT columnas
FROM tabla
WHERE condicion
GROUP BY columnas
HAVING condicion_de_grupo
ORDER BY columnas
LIMIT cantidad;
```

En esta guía nos concentramos en `SELECT` y `FROM`; las demás cláusulas se explicarán en archivos posteriores.

---

## 12. Diferencia entre consultar y modificar

Una consulta `SELECT` muestra información, pero no cambia la tabla.

| Comando | Acción | ¿Modifica los datos? |
| :--- | :--- | :---: |
| `SELECT` | Consulta | No |
| `INSERT` | Inserta filas | Sí |
| `UPDATE` | Modifica filas | Sí |
| `DELETE` | Elimina filas | Sí |

### Ejemplo

```sql
SELECT precio
FROM productos;
```

Esta instrucción muestra los precios, pero no los cambia.

Para modificar un precio se necesitaría una operación distinta:

```sql
UPDATE productos
SET precio = precio * 1.10
WHERE id = 1;
```

> **Regla práctica.** Si solo necesitas observar datos, comienza con `SELECT`. No uses `UPDATE` o `DELETE` para comprobar información.

---

## 13. Errores comunes

| Error | Qué ocurre | Cómo evitarlo |
| :--- | :--- | :--- |
| Olvidar `FROM` | El motor no sabe de dónde obtener la columna | Indicar siempre la tabla de origen cuando se consultan columnas |
| Escribir mal el nombre de la tabla | La consulta falla | Revisar el nombre real de la tabla |
| Usar `SELECT *` en todo | Se devuelven columnas innecesarias | Seleccionar solo las columnas requeridas |
| Confundir alias con nombres reales | Se intenta consultar una columna inexistente | Recordar que el alias solo cambia el resultado |
| Esperar que `SELECT` modifique datos | La consulta solo muestra información | Usar `UPDATE` para modificar |
| Confundir `NULL` con `0` | Se interpretan mal los resultados | Tratar `NULL` como ausencia de valor |
| Repetir columnas sin necesidad | El resultado puede ser difícil de leer | Seleccionar cada columna una vez |
| Consultar varias tablas sin relación | Se genera un producto cartesiano | Usar `JOIN ... ON` |
| Olvidar el punto y coma | Algunas herramientas no ejecutan la sentencia | Terminar la instrucción con `;` |

### Ejemplo incorrecto y corregido

```sql
-- Incorrecto: columna inexistente o mal escrita
SELECT product_name, cost
FROM productos;
```

```sql
-- Correcto: usa los nombres reales
SELECT nombre, precio
FROM productos;
```

---

## 14. Práctica guiada

### Pregunta 1: seleccionar columnas

Muestra el nombre y el stock de todos los productos.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, stock
FROM productos;
```

</details>

### Pregunta 2: consultar todas las columnas

Muestra toda la información de la tabla `productos`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT *
FROM productos;
```

</details>

### Pregunta 3: usar un alias

Muestra `nombre` como `producto` y `precio` como `precio_actual`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    nombre AS producto,
    precio AS precio_actual
FROM productos;
```

</details>

### Pregunta 4: calcular una columna

Muestra el nombre, el precio y el valor total del inventario de cada producto.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    nombre,
    precio,
    stock,
    precio * stock AS valor_inventario
FROM productos;
```

</details>

### Pregunta 5: categorías sin repetir

Muestra cada categoría una sola vez.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT DISTINCT categoria
FROM productos;
```

</details>

---

## 15. Mini desafío final

Usa la tabla `productos` y escribe consultas para obtener:

1. El nombre y la categoría de cada producto.
2. El nombre y el precio con un impuesto del 18 %.
3. Las categorías sin repetir.
4. El nombre, el stock y el valor del inventario.
5. El nombre de cada producto acompañado de la etiqueta fija `'Catálogo'`.

### Una solución posible

```sql
-- 1. Nombre y categoría
SELECT nombre, categoria
FROM productos;

-- 2. Precio con impuesto
SELECT
    nombre,
    precio,
    precio * 1.18 AS precio_con_impuesto
FROM productos;

-- 3. Categorías sin repetir
SELECT DISTINCT categoria
FROM productos;

-- 4. Valor del inventario
SELECT
    nombre,
    stock,
    precio * stock AS valor_inventario
FROM productos;

-- 5. Etiqueta fija
SELECT
    nombre,
    'Catálogo' AS origen
FROM productos;
```

### Resultado esperado de la consulta de inventario

| nombre | stock | valor_inventario |
| :--- | ---: | ---: |
| Teclado mecánico | 12 | 1798.80 |
| Mouse inalámbrico | 25 | 1997.50 |
| Monitor 24 pulgadas | 8 | 7192.00 |
| Webcam HD | 0 | 0.00 |
| Alfombrilla grande | 40 | 1800.00 |

---

## 16. Resumen final

- `SELECT` indica qué columnas o expresiones devolver.
- `FROM` indica de qué tabla o tablas se obtienen los datos.
- `SELECT *` devuelve todas las columnas, pero no siempre es la mejor opción.
- El orden de las columnas en `SELECT` determina el orden del resultado.
- `DISTINCT` elimina combinaciones repetidas del resultado.
- `AS` permite asignar alias a columnas y tablas.
- Una expresión en `SELECT` calcula un valor para mostrarlo, pero no modifica la tabla.
- `NULL` representa ausencia de valor y no equivale a cero ni a una cadena vacía.
- Consultar varias tablas sin una relación puede producir un producto cartesiano.
- `SELECT` pertenece a DQL y no modifica los registros.
- Las cláusulas `WHERE`, `ORDER BY`, `GROUP BY` y `JOIN` se estudiarán en las siguientes guías.

> **Qué debes recordar.** Una consulta básica responde a dos preguntas: ¿de dónde vienen los datos? (`FROM`) y ¿qué quiero ver? (`SELECT`).

---

## Relacionado con otros temas

- [Tipos de datos](../01-fundamentos/tipos-de-datos.md): determinan el tipo de valores que se muestran.
- [DDL, DML y DQL](../01-fundamentos/ddl-dml-dql.md): `SELECT` pertenece a DQL.
- [Constraints](../01-fundamentos/constraints.md): ayudan a garantizar que los datos consultados sean válidos.
- [WHERE y operadores](where-operadores.md): permiten filtrar las filas devueltas.
- [ORDER BY y LIMIT](order-by-limit.md): permiten ordenar y limitar resultados.
- [Joins](../03-consultas-relacionales/joins.md): permiten combinar información de varias tablas.
