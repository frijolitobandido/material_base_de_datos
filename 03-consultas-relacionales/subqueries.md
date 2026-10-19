# Subconsultas (`subqueries`) en SQL

Una subconsulta es una consulta SQL ubicada dentro de otra consulta.

La consulta interna produce un resultado que la consulta externa utiliza para filtrar, calcular o construir un nuevo conjunto de datos.

```sql
SELECT columnas
FROM tabla
WHERE columna operador (
    SELECT columna
    FROM otra_tabla
    WHERE condicion
);
```

También se conocen como **consultas anidadas**.

> **Idea central.** Una subconsulta permite dividir un problema en pasos: primero se obtiene un resultado intermedio y luego se utiliza ese resultado en la consulta principal.

---

## 1. Tablas de ejemplo

Usaremos las tablas `clientes`, `pedidos` y `productos`.

### `clientes`

```sql
CREATE TABLE clientes (
    id_cliente INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    ciudad VARCHAR(80)
);
```

```sql
INSERT INTO clientes (id_cliente, nombre, ciudad)
VALUES
    (101, 'Ana', 'Lima'),
    (102, 'Luis', 'Cusco'),
    (103, 'Marta', 'Lima'),
    (104, 'Carlos', 'Arequipa');
```

| id_cliente | nombre | ciudad |
| ---: | :--- | :--- |
| 101 | Ana | Lima |
| 102 | Luis | Cusco |
| 103 | Marta | Lima |
| 104 | Carlos | Arequipa |

### `pedidos`

```sql
CREATE TABLE pedidos (
    id_pedido INT PRIMARY KEY,
    id_cliente INT,
    fecha_pedido DATE,
    total DECIMAL(10,2)
);
```

```sql
INSERT INTO pedidos (id_pedido, id_cliente, fecha_pedido, total)
VALUES
    (1001, 101, '2026-10-01', 149.90),
    (1002, 102, '2026-10-02', 79.90),
    (1003, 101, '2026-10-05', 899.00),
    (1004, 103, '2026-10-08', 45.00);
```

| id_pedido | id_cliente | fecha_pedido | total |
| ---: | ---: | :--- | ---: |
| 1001 | 101 | 2026-10-01 | 149.90 |
| 1002 | 102 | 2026-10-02 | 79.90 |
| 1003 | 101 | 2026-10-05 | 899.00 |
| 1004 | 103 | 2026-10-08 | 45.00 |

### `productos`

```sql
CREATE TABLE productos (
    id_producto INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    categoria VARCHAR(50),
    precio DECIMAL(10,2),
    stock INT
);
```

```sql
INSERT INTO productos (id_producto, nombre, categoria, precio, stock)
VALUES
    (1, 'Teclado mecánico', 'periféricos', 149.90, 12),
    (2, 'Mouse inalámbrico', 'periféricos', 79.90, 25),
    (3, 'Monitor 24 pulgadas', 'monitores', 899.00, 8),
    (4, 'Webcam HD', 'cámaras', 219.50, 0),
    (5, 'Alfombrilla grande', 'periféricos', 45.00, 40);
```

| id_producto | nombre | categoria | precio | stock |
| ---: | :--- | :--- | ---: | ---: |
| 1 | Teclado mecánico | periféricos | 149.90 | 12 |
| 2 | Mouse inalámbrico | periféricos | 79.90 | 25 |
| 3 | Monitor 24 pulgadas | monitores | 899.00 | 8 |
| 4 | Webcam HD | cámaras | 219.50 | 0 |
| 5 | Alfombrilla grande | periféricos | 45.00 | 40 |

---

## 2. Anatomía de una subconsulta

```sql
SELECT nombre
FROM productos
WHERE precio > (
    SELECT AVG(precio)
    FROM productos
);
```

| Parte | Función |
| :--- | :--- |
| Consulta externa | Busca los productos que se mostrarán |
| Subconsulta | Calcula el precio promedio |
| `AVG(precio)` | Produce un valor numérico |
| `precio > (...)` | Compara cada producto con ese valor |

En este ejemplo, primero se calcula el promedio y luego se comparan los precios individuales.

### Promedio de referencia

| Cálculo | Resultado aproximado |
| :--- | ---: |
| `(149.90 + 79.90 + 899.00 + 219.50 + 45.00) / 5` | 278.66 |

### Resultado de la consulta

| nombre | precio |
| :--- | ---: |
| Monitor 24 pulgadas | 899.00 |

---

## 3. Reglas generales

| Regla | Explicación |
| :--- | :--- |
| Usar paréntesis | La subconsulta normalmente va entre `()` |
| Colocar alias a subconsultas de `FROM` | El alias suele ser obligatorio |
| Revisar cuántas filas devuelve | No es igual comparar contra un valor que contra una lista |
| Revisar cuántas columnas devuelve | Un operador simple espera normalmente una sola columna |
| Indentar el código | Facilita distinguir consulta externa e interna |
| Probar la subconsulta por separado | Ayuda a verificar el resultado intermedio |
| Evitar anidación innecesaria | Una consulta demasiado profunda puede ser difícil de mantener |

### Probar por separado

```sql
-- Paso 1: verificar la subconsulta
SELECT AVG(precio) AS precio_promedio
FROM productos;

-- Paso 2: utilizarla en la consulta externa
SELECT nombre, precio
FROM productos
WHERE precio > (
    SELECT AVG(precio)
    FROM productos
);
```

> **Regla práctica.** Si no puedes explicar qué devuelve la subconsulta por separado, todavía no conviene integrarla en una consulta más grande.

---

## 4. Subconsulta escalar

Una subconsulta escalar devuelve un único valor: una fila y una columna.

```sql
SELECT nombre, precio
FROM productos
WHERE precio > (
    SELECT AVG(precio)
    FROM productos
);
```

`AVG(precio)` devuelve un solo número, por eso puede utilizarse con `>`.

## 4.1 Comparar con el máximo

```sql
SELECT nombre, precio
FROM productos
WHERE precio = (
    SELECT MAX(precio)
    FROM productos
);
```

### Resultado

| nombre | precio |
| :--- | ---: |
| Monitor 24 pulgadas | 899.00 |

## 4.2 Comparar con el mínimo

```sql
SELECT nombre, precio
FROM productos
WHERE precio = (
    SELECT MIN(precio)
    FROM productos
);
```

| nombre | precio |
| :--- | ---: |
| Alfombrilla grande | 45.00 |

## 4.3 Mostrar el valor de referencia

Una subconsulta escalar también puede aparecer en `SELECT`.

```sql
SELECT
    nombre,
    precio,
    (SELECT AVG(precio) FROM productos) AS precio_promedio
FROM productos;
```

| nombre | precio | precio_promedio |
| :--- | ---: | ---: |
| Teclado mecánico | 149.90 | 278.66 |
| Mouse inalámbrico | 79.90 | 278.66 |
| Monitor 24 pulgadas | 899.00 | 278.66 |
| Webcam HD | 219.50 | 278.66 |
| Alfombrilla grande | 45.00 | 278.66 |

> **Cuidado.** Una subconsulta escalar no debe devolver varias filas. Si lo hace, el motor generará un error en muchos SGBD.

---

## 5. Subconsulta con `IN`

`IN` se utiliza cuando la subconsulta devuelve una lista de valores.

### Clientes que tienen pedidos

```sql
SELECT nombre
FROM clientes
WHERE id_cliente IN (
    SELECT id_cliente
    FROM pedidos
);
```

### Resultado

| nombre |
| :--- |
| Ana |
| Luis |
| Marta |

Carlos no aparece porque su identificador no está en `pedidos`.

### Subconsulta externa e interna

| Consulta | Función |
| :--- | :--- |
| Externa | Busca nombres en `clientes` |
| Interna | Obtiene identificadores desde `pedidos` |
| `IN` | Comprueba si cada identificador pertenece a la lista |

## 5.1 Productos de categorías con cierto precio

```sql
SELECT nombre, categoria
FROM productos
WHERE categoria IN (
    SELECT categoria
    FROM productos
    WHERE precio > 200
);
```

Las categorías cuyos productos superan 200 son `monitores` y `cámaras`.

### Resultado

| nombre | categoria |
| :--- | :--- |
| Monitor 24 pulgadas | monitores |
| Webcam HD | cámaras |

## 5.2 `NOT IN`

```sql
SELECT nombre
FROM clientes
WHERE id_cliente NOT IN (
    SELECT id_cliente
    FROM pedidos
);
```

| nombre |
| :--- |
| Carlos |

> **Advertencia.** `NOT IN` puede producir resultados inesperados si la subconsulta devuelve `NULL`. Cuando sea posible, considera utilizar `NOT EXISTS`.

---

## 6. Subconsulta con `EXISTS`

`EXISTS` comprueba si la subconsulta devuelve al menos una fila.

```sql
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

### Resultado

| nombre |
| :--- |
| Ana |
| Luis |
| Marta |

No importa qué valor devuelva `SELECT 1`; `EXISTS` solo revisa si existe alguna fila.

## 6.1 `EXISTS` frente a `IN`

Ambas consultas pueden resolver el mismo problema:

```sql
-- Con IN
SELECT nombre
FROM clientes
WHERE id_cliente IN (
    SELECT id_cliente
    FROM pedidos
);
```

```sql
-- Con EXISTS
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

| Criterio | `IN` | `EXISTS` |
| :--- | :--- | :--- |
| Pregunta principal | ¿Está en esta lista? | ¿Existe al menos una coincidencia? |
| Relación con la consulta externa | Puede ser independiente | Normalmente correlacionada |
| Manejo de muchos resultados | Trabaja con un conjunto | Puede detenerse al encontrar una coincidencia |
| Riesgo con `NULL` | Importante en `NOT IN` | Generalmente más seguro |
| Claridad | Bueno para listas | Bueno para existencia de relaciones |

> **Nota.** El optimizador puede transformar ambas formas internamente. El rendimiento real debe comprobarse con `EXPLAIN` y datos representativos.

## 6.2 `NOT EXISTS`

Busca filas para las que no existe una coincidencia.

```sql
SELECT c.nombre
FROM clientes AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

| nombre |
| :--- |
| Carlos |

Esto equivale conceptualmente a un antijoin.

---

## 7. Subconsultas con `ANY` y `SOME`

`ANY` compara un valor con cada valor de una lista y devuelve verdadero si la comparación se cumple con al menos uno.

```sql
SELECT nombre, precio
FROM productos
WHERE precio > ANY (
    SELECT precio
    FROM productos
    WHERE categoria = 'periféricos'
);
```

Los precios de periféricos son 149.90, 79.90 y 45.00. La consulta busca productos cuyo precio sea mayor que al menos uno de ellos.

### Resultado

| nombre | precio |
| :--- | ---: |
| Teclado mecánico | 149.90 |
| Mouse inalámbrico | 79.90 |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |

`SOME` es un sinónimo frecuente de `ANY`:

```sql
WHERE precio > SOME (
    SELECT precio
    FROM productos
    WHERE categoria = 'periféricos'
)
```

### Equivalencias intuitivas

Para una comparación `>`:

```text
valor > ANY(lista) ≈ valor > MIN(lista)
```

Para una comparación `<`:

```text
valor < ANY(lista) ≈ valor < MAX(lista)
```

> **Compatibilidad.** `ANY` y `SOME` existen en SQL estándar, pero su disponibilidad y sintaxis pueden variar según el motor.

---

## 8. Subconsultas con `ALL`

`ALL` exige que la comparación sea verdadera para todos los valores devueltos por la subconsulta.

```sql
SELECT nombre, precio
FROM productos
WHERE precio > ALL (
    SELECT precio
    FROM productos
    WHERE categoria = 'periféricos'
);
```

El precio debe ser mayor que 149.90, 79.90 y 45.00.

### Resultado

| nombre | precio |
| :--- | ---: |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |

### Equivalencias intuitivas

```text
valor > ALL(lista) ≈ valor > MAX(lista)
```

```text
valor < ALL(lista) ≈ valor < MIN(lista)
```

| Operador | Significado |
| :--- | :--- |
| `> ANY` | Mayor que al menos un valor |
| `> ALL` | Mayor que todos los valores |
| `< ANY` | Menor que al menos un valor |
| `< ALL` | Menor que todos los valores |
| `= ANY` | Similar a `IN` |
| `<> ALL` | Similar a “diferente de todos” |

---

## 9. Subconsulta en `FROM`

Una subconsulta colocada en `FROM` se convierte en una tabla derivada.

Debe tener un alias.

```sql
SELECT
    resumen.ciudad,
    resumen.total_clientes
FROM (
    SELECT
        ciudad,
        COUNT(*) AS total_clientes
    FROM clientes
    GROUP BY ciudad
) AS resumen
WHERE resumen.total_clientes > 1;
```

### Resultado

| ciudad | total_clientes |
| :--- | ---: |
| Lima | 2 |

### Estructura

| Nivel | Función |
| :--- | :--- |
| Subconsulta | Cuenta clientes por ciudad |
| Tabla derivada `resumen` | Representa ese resultado intermedio |
| Consulta externa | Conserva ciudades con más de un cliente |

> **Regla práctica.** Una subconsulta en `FROM` funciona como una tabla temporal para esa consulta, pero no crea una tabla permanente.

## 9.1 Calcular sobre un resultado agrupado

```sql
SELECT
    resumen.id_cliente,
    resumen.total_comprado
FROM (
    SELECT
        id_cliente,
        SUM(total) AS total_comprado
    FROM pedidos
    GROUP BY id_cliente
) AS resumen
WHERE resumen.total_comprado > 100;
```

| id_cliente | total_comprado |
| ---: | ---: |
| 101 | 1048.90 |

---

## 10. Subconsulta en `SELECT`

Una subconsulta escalar puede producir una columna calculada.

```sql
SELECT
    c.nombre,
    (
        SELECT COUNT(*)
        FROM pedidos AS p
        WHERE p.id_cliente = c.id_cliente
    ) AS cantidad_pedidos
FROM clientes AS c;
```

### Resultado

| nombre | cantidad_pedidos |
| :--- | ---: |
| Ana | 2 |
| Luis | 1 |
| Marta | 1 |
| Carlos | 0 |

La subconsulta utiliza el `id_cliente` de cada fila externa. Por eso es una subconsulta correlacionada.

### Comparación con `LEFT JOIN`

La misma información puede obtenerse con:

```sql
SELECT
    c.nombre,
    COUNT(p.id_pedido) AS cantidad_pedidos
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON p.id_cliente = c.id_cliente
GROUP BY c.id_cliente, c.nombre;
```

| Forma | Ventaja principal |
| :--- | :--- |
| Subconsulta en `SELECT` | Puede ser intuitiva para un cálculo puntual |
| `LEFT JOIN` y `GROUP BY` | Suele ser más flexible para varios cálculos |

---

## 11. Subconsulta en `WHERE`

Es el uso más conocido.

## 11.1 Productos más caros que el promedio

```sql
SELECT nombre, precio
FROM productos
WHERE precio > (
    SELECT AVG(precio)
    FROM productos
);
```

## 11.2 Clientes con pedidos superiores a 500

```sql
SELECT nombre
FROM clientes
WHERE id_cliente IN (
    SELECT id_cliente
    FROM pedidos
    WHERE total > 500
);
```

| nombre |
| :--- |
| Ana |

## 11.3 Clientes con al menos un pedido superior a 500

```sql
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
      AND p.total > 500
);
```

`EXISTS` expresa de forma directa la relación con la fila externa.

---

## 12. Subconsultas correlacionadas

Una subconsulta correlacionada hace referencia a una columna de la consulta externa.

```sql
SELECT
    c.nombre,
    (
        SELECT COUNT(*)
        FROM pedidos AS p
        WHERE p.id_cliente = c.id_cliente
    ) AS cantidad_pedidos
FROM clientes AS c;
```

La referencia `c.id_cliente` proviene de la consulta externa.

### Funcionamiento conceptual

| Fila externa | Valor usado por la subconsulta | Resultado |
| :--- | ---: | ---: |
| Ana | 101 | 2 pedidos |
| Luis | 102 | 1 pedido |
| Marta | 103 | 1 pedido |
| Carlos | 104 | 0 pedidos |

### Subconsulta correlacionada con `EXISTS`

```sql
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
      AND p.total >= 100;
);
```

> **Corrección sintáctica:** dentro de la subconsulta no debe colocarse un punto y coma antes de cerrar la consulta externa.

```sql
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
      AND p.total >= 100
);
```

### Ventajas y precauciones

| Ventaja | Precaución |
| :--- | :--- |
| Expresa claramente una condición por fila | Puede ejecutarse muchas veces conceptualmente |
| Es útil para `EXISTS` | Puede ser menos eficiente en grandes volúmenes |
| Evita algunos duplicados de `JOIN` | Conviene comparar el plan con una alternativa |

---

## 13. Subconsultas de varias columnas

Algunas comparaciones pueden utilizar más de una columna.

```sql
SELECT *
FROM pedidos
WHERE (id_cliente, total) IN (
    SELECT id_cliente, MAX(total)
    FROM pedidos
    GROUP BY id_cliente
);
```

Esta forma busca los pedidos cuyo par `(id_cliente, total)` coincide con el máximo de cada cliente.

> **Compatibilidad.** Las comparaciones de filas pueden variar entre motores. Cuando se necesite mayor portabilidad, se puede utilizar un `JOIN` contra la subconsulta agrupada.

### Alternativa con tabla derivada

```sql
SELECT p.id_pedido, p.id_cliente, p.total
FROM pedidos AS p
JOIN (
    SELECT id_cliente, MAX(total) AS max_total
    FROM pedidos
    GROUP BY id_cliente
) AS maximos
    ON maximos.id_cliente = p.id_cliente
   AND maximos.max_total = p.total;
```

### Resultado

| id_pedido | id_cliente | total |
| ---: | ---: | ---: |
| 1001 | 101 | 149.90 |
| 1003 | 101 | 899.00 |
| 1002 | 102 | 79.90 |
| 1004 | 103 | 45.00 |

---

## 14. Subconsultas con `INSERT`

Una subconsulta puede proporcionar los registros que se insertarán.

```sql
CREATE TABLE clientes_lima (
    id_cliente INT,
    nombre VARCHAR(100)
);
```

```sql
INSERT INTO clientes_lima (id_cliente, nombre)
SELECT id_cliente, nombre
FROM clientes
WHERE ciudad = 'Lima';
```

### Resultado insertado

| id_cliente | nombre |
| ---: | :--- |
| 101 | Ana |
| 103 | Marta |

La subconsulta aquí es el `SELECT` que alimenta al `INSERT`.

---

## 15. Subconsultas con `UPDATE`

Una subconsulta puede calcular el valor que se asignará.

```sql
UPDATE productos
SET precio = precio * 1.10
WHERE categoria = (
    SELECT categoria
    FROM productos
    WHERE id_producto = 2
);
```

Esta consulta modificaría todos los productos de la categoría del producto 2. Antes de ejecutarla, conviene revisar qué filas serán afectadas.

```sql
SELECT id_producto, nombre, categoria, precio
FROM productos
WHERE categoria = (
    SELECT categoria
    FROM productos
    WHERE id_producto = 2
);
```

> **Advertencia.** Una subconsulta en `UPDATE` puede afectar muchas filas. Prueba primero con `SELECT` y usa una transacción cuando sea posible.

### Actualizar con un valor agregado

```sql
UPDATE productos
SET precio = (
    SELECT AVG(precio)
    FROM productos
)
WHERE id_producto = 4;
```

En algunos motores no se permite modificar una tabla y leerla directamente en una subconsulta del mismo nivel. La solución puede requerir una tabla derivada o una estrategia específica del SGBD.

---

## 16. Subconsultas con `DELETE`

También pueden utilizarse para identificar las filas que se eliminarán.

```sql
DELETE FROM clientes
WHERE id_cliente NOT IN (
    SELECT id_cliente
    FROM pedidos
);
```

Esta consulta eliminaría clientes sin pedidos. Es más seguro revisar antes:

```sql
SELECT *
FROM clientes
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = clientes.id_cliente
);
```

> **Advertencia.** Nunca pruebes directamente un `DELETE` complejo sin ejecutar antes una consulta `SELECT` con la misma condición.

---

## 17. Subconsulta frente a `JOIN`

Muchas subconsultas pueden reescribirse como joins, pero no siempre tienen la misma claridad.

## 17.1 Productos con categoría de precio alto

### Con subconsulta

```sql
SELECT nombre, categoria
FROM productos
WHERE categoria IN (
    SELECT categoria
    FROM productos
    WHERE precio > 500
);
```

### Con `JOIN`

```sql
SELECT DISTINCT p.nombre, p.categoria
FROM productos AS p
JOIN productos AS caros
    ON p.categoria = caros.categoria
WHERE caros.precio > 500;
```

| Opción | Puede ser mejor cuando |
| :--- | :--- |
| Subconsulta | Se piensa en términos de una lista o condición independiente |
| `JOIN` | Se necesitan columnas de ambas partes |
| `EXISTS` | Solo interesa saber si existe una coincidencia |
| Tabla derivada | Se necesita reutilizar un resultado calculado |
| CTE | La consulta intermedia debe nombrarse y reutilizarse |

## 17.2 Riesgo de duplicados

Un `JOIN` puede repetir filas si hay varias coincidencias. `EXISTS` normalmente evita ese problema porque solo verifica existencia.

```sql
-- Puede repetir clientes si tienen varios pedidos
SELECT c.nombre
FROM clientes AS c
JOIN pedidos AS p
    ON p.id_cliente = c.id_cliente;
```

```sql
-- Cada cliente aparece como máximo una vez
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

---

## 18. `NULL` y subconsultas

Los valores `NULL` pueden afectar `IN`, `NOT IN`, comparaciones y agregaciones.

## 18.1 Problema con `NOT IN`

Si la subconsulta devuelve incluso un `NULL`, la lógica de `NOT IN` puede producir `UNKNOWN` y no devolver las filas esperadas.

```sql
SELECT nombre
FROM clientes
WHERE id_cliente NOT IN (
    SELECT id_cliente
    FROM pedidos
);
```

Si `pedidos.id_cliente` puede contener `NULL`, considera:

```sql
SELECT c.nombre
FROM clientes AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

## 18.2 Agregaciones y `NULL`

Las funciones como `AVG`, `SUM` y `COUNT(columna)` normalmente ignoran valores `NULL`.

| Función | Tratamiento habitual de `NULL` |
| :--- | :--- |
| `COUNT(*)` | Cuenta filas |
| `COUNT(columna)` | No cuenta valores `NULL` |
| `AVG(columna)` | Ignora `NULL` |
| `SUM(columna)` | Ignora `NULL` |
| `MAX` / `MIN` | Ignoran `NULL` |

---

## 19. Subconsultas anidadas

Una subconsulta puede contener otra subconsulta.

```sql
SELECT nombre, precio
FROM productos
WHERE precio > (
    SELECT AVG(precio)
    FROM productos
    WHERE categoria IN (
        SELECT categoria
        FROM productos
        WHERE stock = 0
    )
);
```

### Lectura de dentro hacia fuera

| Nivel | Pregunta |
| ---: | :--- |
| 1 | ¿Qué categorías tienen productos agotados? |
| 2 | ¿Cuál es el promedio de precio de esas categorías? |
| 3 | ¿Qué productos superan ese promedio? |

### Recomendación

Si una consulta tiene demasiados niveles, considera utilizar un CTE:

```sql
WITH categorias_agotadas AS (
    SELECT DISTINCT categoria
    FROM productos
    WHERE stock = 0
), promedio AS (
    SELECT AVG(precio) AS valor
    FROM productos
    WHERE categoria IN (SELECT categoria FROM categorias_agotadas)
)
SELECT p.nombre, p.precio
FROM productos AS p
CROSS JOIN promedio AS a
WHERE p.precio > a.valor;
```

Los CTE se estudiarán en la guía `cte-with.md`.

---

## 20. Rendimiento de subconsultas

Una subconsulta no es automáticamente lenta ni rápida. El resultado depende del motor, los índices, el volumen y el plan de ejecución.

## 20.1 Recomendaciones

| Recomendación | Motivo |
| :--- | :--- |
| Indexar columnas de relación | Acelera búsquedas en `IN`, `EXISTS` y joins |
| Seleccionar solo columnas necesarias | Reduce datos intermedios |
| Usar `EXISTS` para existencia | Evita traer una lista completa cuando no hace falta |
| Evitar funciones sobre claves filtradas | Puede impedir el uso de índices |
| Revisar `EXPLAIN` | Muestra el plan de ejecución |
| Comparar alternativas | Un `JOIN`, `EXISTS` o subconsulta pueden comportarse distinto |
| Evitar anidación excesiva | Mejora legibilidad y mantenimiento |

### Ejemplo con índice

```sql
CREATE INDEX idx_pedidos_cliente
ON pedidos (id_cliente);
```

Este índice puede ayudar a consultas como:

```sql
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

### Revisar el plan

```sql
EXPLAIN
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

---

## 21. Errores comunes

| Error | Qué ocurre | Cómo evitarlo |
| :--- | :--- | :--- |
| Subconsulta escalar devuelve varias filas | El operador `=`, `>` o `<` no sabe cuál usar | Usar `IN`, `ANY`, `ALL` o agregar el resultado |
| Subconsulta devuelve varias columnas con un operador simple | La comparación no coincide | Devolver una sola columna o usar comparación de filas |
| Olvidar paréntesis | Error de sintaxis | Encerrar la subconsulta cuando corresponda |
| No poner alias a una tabla derivada | El motor puede rechazar la consulta | Escribir `AS alias` |
| Usar `NOT IN` con `NULL` | Puede no devolver resultados esperados | Preferir `NOT EXISTS` |
| Confundir `IN` con `EXISTS` | La consulta no expresa bien la intención | Usar `IN` para pertenencia y `EXISTS` para existencia |
| No correlacionar la subconsulta | Se compara contra un resultado global en vez de por fila | Referenciar explícitamente la tabla externa |
| Correlacionar con una columna equivocada | Resultados incorrectos | Revisar las claves de relación |
| Usar una subconsulta para todo | La consulta se vuelve difícil de leer | Considerar `JOIN`, CTE o una vista |
| Modificar sin probar | `UPDATE` o `DELETE` pueden afectar muchas filas | Ejecutar primero un `SELECT` |
| Ignorar duplicados de un `JOIN` equivalente | Las filas pueden repetirse | Analizar cardinalidad o usar `EXISTS` |
| No revisar el plan | Puede haber problemas de rendimiento | Utilizar `EXPLAIN` |

### Ejemplo incorrecto y corregido

```sql
-- Incorrecto si la subconsulta devuelve varios precios
SELECT nombre
FROM productos
WHERE precio = (
    SELECT precio
    FROM productos
    WHERE categoria = 'periféricos'
);
```

```sql
-- Correcto para comparar con varios valores
SELECT nombre
FROM productos
WHERE precio IN (
    SELECT precio
    FROM productos
    WHERE categoria = 'periféricos'
);
```

---

## 22. Práctica guiada

## Pregunta 1: productos sobre el promedio

Muestra los productos cuyo precio sea superior al precio promedio.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, precio
FROM productos
WHERE precio > (
    SELECT AVG(precio)
    FROM productos
);
```

</details>

## Pregunta 2: clientes con pedidos

Muestra los clientes que tienen al menos un pedido.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

</details>

## Pregunta 3: clientes sin pedidos

Muestra los clientes que no tienen pedidos.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT c.nombre
FROM clientes AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

</details>

## Pregunta 4: pedidos de clientes de Lima

Muestra los pedidos realizados por clientes de Lima.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT id_pedido, id_cliente, total
FROM pedidos
WHERE id_cliente IN (
    SELECT id_cliente
    FROM clientes
    WHERE ciudad = 'Lima'
);
```

</details>

## Pregunta 5: tabla derivada

Calcula el total comprado por cliente y muestra solo los totales superiores a 100.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT resumen.id_cliente, resumen.total_comprado
FROM (
    SELECT
        id_cliente,
        SUM(total) AS total_comprado
    FROM pedidos
    GROUP BY id_cliente
) AS resumen
WHERE resumen.total_comprado > 100;
```

</details>

---

## 23. Mini desafío final

Usa las tablas `clientes`, `pedidos` y `productos` para resolver:

1. Obtener los productos cuyo precio sea mayor que el promedio.
2. Obtener los clientes que realizaron un pedido superior a 500.
3. Obtener los clientes que nunca realizaron pedidos.
4. Obtener los pedidos realizados por clientes de Lima.
5. Obtener el pedido o los pedidos con el total máximo.
6. Calcular la cantidad de pedidos por cliente mediante una subconsulta en `SELECT`.
7. Explicar cuándo sería mejor usar `JOIN`, `IN` o `EXISTS`.

## Una solución posible

```sql
-- 1. Productos sobre el promedio
SELECT nombre, precio
FROM productos
WHERE precio > (
    SELECT AVG(precio)
    FROM productos
);

-- 2. Clientes con un pedido superior a 500
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
      AND p.total > 500
);

-- 3. Clientes sin pedidos
SELECT c.nombre
FROM clientes AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);

-- 4. Pedidos de clientes de Lima
SELECT p.id_pedido, p.id_cliente, p.total
FROM pedidos AS p
WHERE p.id_cliente IN (
    SELECT c.id_cliente
    FROM clientes AS c
    WHERE c.ciudad = 'Lima'
);

-- 5. Pedido o pedidos con total máximo
SELECT id_pedido, id_cliente, total
FROM pedidos
WHERE total = (
    SELECT MAX(total)
    FROM pedidos
);

-- 6. Cantidad de pedidos por cliente
SELECT
    c.nombre,
    (
        SELECT COUNT(*)
        FROM pedidos AS p
        WHERE p.id_cliente = c.id_cliente
    ) AS cantidad_pedidos
FROM clientes AS c;
```

### Tabla de decisión

| Necesidad | Técnica recomendada |
| :--- | :--- |
| Comparar con un único valor calculado | Subconsulta escalar |
| Comparar con una lista | `IN` |
| Verificar si existe una relación | `EXISTS` |
| Verificar que no exista una relación | `NOT EXISTS` |
| Comparar con algún valor de una lista | `ANY` o `SOME` |
| Comparar con todos los valores | `ALL` |
| Usar un resultado como tabla | Subconsulta en `FROM` |
| Calcular un valor por cada fila externa | Subconsulta correlacionada |
| Reutilizar varias etapas con nombres | CTE |

---

## 24. Resumen final

- Una subconsulta es una consulta dentro de otra consulta.
- Una subconsulta escalar devuelve una fila y una columna.
- `IN` compara un valor con una lista producida por la subconsulta.
- `EXISTS` comprueba si existe al menos una fila coincidente.
- `NOT EXISTS` busca la ausencia de una coincidencia.
- `ANY` o `SOME` requieren que la comparación se cumpla para al menos un valor.
- `ALL` exige que la comparación se cumpla para todos los valores.
- Una subconsulta en `FROM` se conoce como tabla derivada y necesita un alias.
- Una subconsulta en `SELECT` puede calcular un valor para cada fila.
- Una subconsulta correlacionada utiliza columnas de la consulta externa.
- Las subconsultas también pueden usarse con `INSERT`, `UPDATE` y `DELETE`.
- `NOT IN` puede comportarse de forma inesperada cuando aparecen valores `NULL`.
- Muchas subconsultas pueden reescribirse con `JOIN`, `EXISTS` o CTE.
- El rendimiento debe comprobarse con índices, datos reales y `EXPLAIN`.
- Una subconsulta debe probarse por separado antes de integrarla en una consulta compleja.

> **Qué debes recordar.** Primero identifica qué resultado necesita la consulta externa: un valor, una lista, una existencia, una tabla intermedia o una condición para cada fila.

---

## Relacionado con otros temas

- [Álgebra relacional](algebra-relacional.md): explica semijoins, antijoins y división relacional.
- [JOIN](joins.md): ofrece otra forma de combinar tablas y resolver muchas subconsultas.
- [`SELECT` y `FROM`](../02-consultas-basicas/select-from.md): estructura de las consultas externas e internas.
- [WHERE y operadores](../02-consultas-basicas/where-operadores.md): recibe con frecuencia los resultados de una subconsulta.
- [Funciones SQL](../02-consultas-basicas/funciones-sql.md): se utilizan dentro de subconsultas escalares y agrupadas.
- [Agregaciones y `GROUP BY`](agregaciones-group-by.md): produce resultados que pueden utilizarse en subconsultas.
- [CTE y `WITH`](cte-with.md): alternativa legible para consultas con varias etapas.
- [Claves primarias y foráneas](../01-fundamentos/claves-primarias-y-foraneas.md): permiten construir condiciones de correlación confiables.
