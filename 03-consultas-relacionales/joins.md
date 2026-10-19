# `JOIN`: combinar datos de varias tablas

`JOIN` permite combinar filas de dos o más tablas utilizando una condición de relación.

En una base de datos relacional, la información suele distribuirse en varias tablas. Por ejemplo, los datos de un cliente pueden estar en `clientes` y sus pedidos en `pedidos`. Un `JOIN` permite consultar ambas partes como un solo resultado.

```sql
SELECT columnas
FROM tabla_a AS a
JOIN tabla_b AS b
    ON a.clave = b.clave;
```

> **Idea central.** Un `JOIN` no copia ni fusiona permanentemente las tablas: construye un resultado temporal combinando las filas relacionadas.

---

## 1. Tablas de ejemplo

Usaremos estas tablas para practicar.

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
    (1004, 103, '2026-10-08', 45.00),
    (1005, 999, '2026-10-09', 30.00);
```

| id_pedido | id_cliente | fecha_pedido | total |
| ---: | ---: | :--- | ---: |
| 1001 | 101 | 2026-10-01 | 149.90 |
| 1002 | 102 | 2026-10-02 | 79.90 |
| 1003 | 101 | 2026-10-05 | 899.00 |
| 1004 | 103 | 2026-10-08 | 45.00 |
| 1005 | 999 | 2026-10-09 | 30.00 |

El pedido `1005` utiliza el cliente `999`, que no aparece en `clientes`. Esto permite observar qué ocurre cuando no existe una coincidencia.

### `productos` y `detalle_pedido`

```sql
CREATE TABLE productos (
    id_producto INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio DECIMAL(10,2)
);

CREATE TABLE detalle_pedido (
    id_pedido INT,
    id_producto INT,
    cantidad INT,
    PRIMARY KEY (id_pedido, id_producto)
);
```

```sql
INSERT INTO productos (id_producto, nombre, precio)
VALUES
    (1, 'Teclado mecánico', 149.90),
    (2, 'Mouse inalámbrico', 79.90),
    (3, 'Monitor 24 pulgadas', 899.00),
    (4, 'Webcam HD', 219.50);

INSERT INTO detalle_pedido (id_pedido, id_producto, cantidad)
VALUES
    (1001, 1, 1),
    (1002, 2, 1),
    (1003, 3, 1),
    (1004, 4, 2);
```

---

## 2. ¿Por qué necesitamos `JOIN`?

La información está separada para evitar repetir datos.

| Tabla | Información principal |
| :--- | :--- |
| `clientes` | Identidad y ciudad del cliente |
| `pedidos` | Fecha y total de cada pedido |
| `productos` | Datos de cada producto |
| `detalle_pedido` | Productos incluidos en cada pedido |

Para mostrar el nombre del cliente junto con su pedido necesitamos relacionar:

```text
clientes.id_cliente = pedidos.id_cliente
```

Sin `JOIN`, tendríamos que consultar cada tabla por separado y combinar los resultados manualmente.

---

## 3. Sintaxis básica con `INNER JOIN`

```sql
SELECT
    c.nombre,
    p.id_pedido,
    p.total
FROM clientes AS c
INNER JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

### Resultado

| nombre | id_pedido | total |
| :--- | ---: | ---: |
| Ana | 1001 | 149.90 |
| Luis | 1002 | 79.90 |
| Ana | 1003 | 899.00 |
| Marta | 1004 | 45.00 |

El pedido `1005` no aparece porque no tiene un cliente coincidente. Carlos tampoco aparece porque no tiene pedidos.

### Partes de la consulta

| Parte | Función |
| :--- | :--- |
| `FROM clientes AS c` | Define la tabla principal y su alias |
| `INNER JOIN pedidos AS p` | Indica la tabla que se combinará |
| `ON c.id_cliente = p.id_cliente` | Define la relación |
| `SELECT c.nombre, p.id_pedido` | Elige las columnas del resultado |

> **Regla práctica.** Escribe la condición de relación en `ON`, no en `WHERE`, especialmente cuando trabajes con `LEFT JOIN`.

---

## 4. Aliases y nombres calificados

Los alias hacen que las consultas sean más cortas y evitan ambigüedades.

```sql
SELECT
    c.id_cliente,
    c.nombre,
    p.id_pedido,
    p.fecha_pedido
FROM clientes AS c
INNER JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

La notación `c.nombre` significa “la columna `nombre` de la tabla cuyo alias es `c`”.

### ¿Por qué calificar columnas?

| Situación | Ventaja |
| :--- | :--- |
| Dos tablas tienen una columna `id` | Indica exactamente cuál se necesita |
| Consulta con varias tablas | Evita ambigüedades |
| Mantenimiento del código | Hace visible el origen de cada dato |
| Columnas con el mismo nombre | Previene errores del motor |

---

## 5. Tipos principales de `JOIN`

## 5.1 `INNER JOIN`

Devuelve únicamente las filas que tienen coincidencia en ambas tablas.

```sql
SELECT
    c.nombre,
    p.id_pedido,
    p.total
FROM clientes AS c
INNER JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

| Cliente | Tiene pedido | Aparece en `INNER JOIN` |
| :--- | :---: | :---: |
| Ana | Sí | Sí |
| Luis | Sí | Sí |
| Marta | Sí | Sí |
| Carlos | No | No |
| Pedido 1005, cliente 999 | Sin cliente válido | No |

La palabra `INNER` puede omitirse:

```sql
SELECT c.nombre, p.id_pedido
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

`JOIN` sin una palabra adicional significa normalmente `INNER JOIN`.

---

## 5.2 `LEFT JOIN`

Devuelve todas las filas de la tabla izquierda y las coincidencias de la tabla derecha. Cuando no existe coincidencia, las columnas de la derecha aparecen como `NULL`.

```sql
SELECT
    c.id_cliente,
    c.nombre,
    p.id_pedido,
    p.total
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

### Resultado

| id_cliente | nombre | id_pedido | total |
| ---: | :--- | ---: | ---: |
| 101 | Ana | 1001 | 149.90 |
| 101 | Ana | 1003 | 899.00 |
| 102 | Luis | 1002 | 79.90 |
| 103 | Marta | 1004 | 45.00 |
| 104 | Carlos | `NULL` | `NULL` |

Carlos aparece aunque no tenga pedidos porque `clientes` es la tabla izquierda.

### Buscar clientes sin pedidos

```sql
SELECT
    c.id_cliente,
    c.nombre
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
WHERE p.id_pedido IS NULL;
```

| id_cliente | nombre |
| ---: | :--- |
| 104 | Carlos |

Esta técnica se conoce como antijoin mediante `LEFT JOIN`.

> **Importante.** La columna usada en `IS NULL` debe ser una columna que no pueda ser `NULL` en un pedido real, normalmente su clave primaria.

---

## 5.3 `RIGHT JOIN`

Devuelve todas las filas de la tabla derecha y las coincidencias de la tabla izquierda.

```sql
SELECT
    c.nombre,
    p.id_pedido,
    p.id_cliente
FROM clientes AS c
RIGHT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

### Resultado

| nombre | id_pedido | id_cliente |
| :--- | ---: | ---: |
| Ana | 1001 | 101 |
| Luis | 1002 | 102 |
| Ana | 1003 | 101 |
| Marta | 1004 | 103 |
| `NULL` | 1005 | 999 |

El pedido `1005` aparece aunque no tenga un cliente coincidente porque `pedidos` es la tabla derecha.

### Reescribir un `RIGHT JOIN` como `LEFT JOIN`

La misma consulta suele ser más clara si se invierten las tablas:

```sql
SELECT
    c.nombre,
    p.id_pedido,
    p.id_cliente
FROM pedidos AS p
LEFT JOIN clientes AS c
    ON c.id_cliente = p.id_cliente;
```

| Forma | Recomendación |
| :--- | :--- |
| `RIGHT JOIN` | Válido, pero puede ser menos intuitivo |
| Invertir tablas y usar `LEFT JOIN` | Suele facilitar la lectura |

---

## 5.4 `FULL OUTER JOIN`

Devuelve todas las filas de ambas tablas: las coincidentes y las que no tienen correspondencia.

```sql
SELECT
    c.id_cliente,
    c.nombre,
    p.id_pedido,
    p.id_cliente AS cliente_del_pedido
FROM clientes AS c
FULL OUTER JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

### Resultado conceptual

| id_cliente | nombre | id_pedido | cliente_del_pedido |
| ---: | :--- | ---: | ---: |
| 101 | Ana | 1001 | 101 |
| 101 | Ana | 1003 | 101 |
| 102 | Luis | 1002 | 102 |
| 103 | Marta | 1004 | 103 |
| 104 | Carlos | `NULL` | `NULL` |
| `NULL` | `NULL` | 1005 | 999 |

`FULL OUTER JOIN` está disponible en PostgreSQL y otros motores, pero no en MySQL de forma directa.

### Simulación con `UNION`

```sql
SELECT
    c.id_cliente,
    c.nombre,
    p.id_pedido,
    p.id_cliente AS cliente_del_pedido
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente

UNION

SELECT
    c.id_cliente,
    c.nombre,
    p.id_pedido,
    p.id_cliente AS cliente_del_pedido
FROM pedidos AS p
LEFT JOIN clientes AS c
    ON c.id_cliente = p.id_cliente;
```

> **Compatibilidad.** Revisa si tu motor soporta `FULL OUTER JOIN`; si no, una combinación de `LEFT JOIN` y `UNION` puede simularlo.

---

## 5.5 `CROSS JOIN`

`CROSS JOIN` genera el producto cartesiano: combina cada fila de una tabla con cada fila de la otra.

```sql
SELECT
    c.nombre,
    p.id_pedido
FROM clientes AS c
CROSS JOIN pedidos AS p;
```

Como hay 4 clientes y 5 pedidos:

| Filas de `clientes` | Filas de `pedidos` | Resultado |
| ---: | ---: | ---: |
| 4 | 5 | 20 combinaciones |

Normalmente no se utiliza para relacionar clientes con sus pedidos. Para eso se necesita una condición `ON`.

> **Advertencia.** Un `CROSS JOIN` puede generar una cantidad enorme de filas. Úsalo solo cuando realmente necesites todas las combinaciones.

---

## 5.6 `SELF JOIN`

Un self join combina una tabla consigo misma. Se necesitan dos alias para representar cada papel.

Supongamos una tabla de empleados:

```sql
CREATE TABLE empleados (
    id_empleado INT PRIMARY KEY,
    nombre VARCHAR(100),
    id_supervisor INT
);
```

```sql
INSERT INTO empleados (id_empleado, nombre, id_supervisor)
VALUES
    (1, 'Ana', NULL),
    (2, 'Luis', 1),
    (3, 'Marta', 1),
    (4, 'Carlos', 2);
```

```sql
SELECT
    e.nombre AS empleado,
    s.nombre AS supervisor
FROM empleados AS e
LEFT JOIN empleados AS s
    ON e.id_supervisor = s.id_empleado;
```

### Resultado

| empleado | supervisor |
| :--- | :--- |
| Ana | `NULL` |
| Luis | Ana |
| Marta | Ana |
| Carlos | Luis |

El alias `e` representa al empleado y `s` representa al supervisor.

---

## 6. Condiciones de combinación

## 6.1 Join por igualdad

Es el caso más común:

```sql
SELECT c.nombre, p.id_pedido
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

Esto se denomina equijoin.

## 6.2 Join con varias condiciones

```sql
SELECT c.nombre, p.id_pedido, p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
   AND p.total >= 100;
```

### Resultado

| nombre | id_pedido | total |
| :--- | ---: | ---: |
| Ana | 1001 | 149.90 |
| Ana | 1003 | 899.00 |

La relación exige que el cliente coincida y que el pedido tenga un total mínimo.

## 6.3 Join con operadores de rango

Un theta join puede usar `>`, `<`, `>=`, `<=` u otros operadores.

```sql
CREATE TABLE rangos_descuento (
    nivel VARCHAR(30),
    minimo DECIMAL(10,2),
    maximo DECIMAL(10,2),
    porcentaje DECIMAL(5,2)
);
```

```sql
SELECT
    p.nombre,
    p.total,
    r.nivel,
    r.porcentaje
FROM pedidos AS p
JOIN rangos_descuento AS r
    ON p.total >= r.minimo
   AND p.total < r.maximo;
```

Este tipo de combinación relaciona una fila con el rango que contiene su valor.

## 6.4 `USING`

Cuando ambas tablas tienen una columna con el mismo nombre, algunos motores permiten:

```sql
SELECT
    nombre,
    id_pedido,
    total
FROM clientes
JOIN pedidos
USING (id_cliente);
```

`USING (id_cliente)` equivale aproximadamente a:

```sql
ON clientes.id_cliente = pedidos.id_cliente
```

| Forma | Ventaja | Limitación |
| :--- | :--- | :--- |
| `ON` | Flexible y explícita | Escribe la comparación completa |
| `USING` | Más corta cuando el nombre coincide | Solo sirve para columnas con el mismo nombre |

> **Recomendación.** `ON` suele ser más clara cuando estás aprendiendo o cuando la relación no es una igualdad simple.

---

## 7. `ON` y `WHERE`: una diferencia importante

La ubicación de una condición puede cambiar el resultado, especialmente con `LEFT JOIN`.

## 7.1 Condición en `ON`

```sql
SELECT
    c.nombre,
    p.id_pedido,
    p.total
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
   AND p.total > 100;
```

### Resultado

| nombre | id_pedido | total |
| :--- | ---: | ---: |
| Ana | 1001 | 149.90 |
| Ana | 1003 | 899.00 |
| Luis | `NULL` | `NULL` |
| Marta | `NULL` | `NULL` |
| Carlos | `NULL` | `NULL` |

Se conservan todos los clientes, pero solo se combinan pedidos mayores de 100.

## 7.2 Condición en `WHERE`

```sql
SELECT
    c.nombre,
    p.id_pedido,
    p.total
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
WHERE p.total > 100;
```

### Resultado

| nombre | id_pedido | total |
| :--- | ---: | ---: |
| Ana | 1001 | 149.90 |
| Ana | 1003 | 899.00 |

En este caso, las filas con `p.total = NULL` se eliminan por el `WHERE`. En la práctica, el resultado se comporta como un filtro de coincidencias.

| Ubicación | Efecto |
| :--- | :--- |
| Condición en `ON` | Decide qué filas de la tabla derecha se combinan |
| Condición en `WHERE` | Filtra el resultado después del join |

> **Regla práctica.** Si quieres conservar las filas sin coincidencia de un `LEFT JOIN`, coloca la condición de la tabla derecha en `ON`.

---

## 8. Combinar más de dos tablas

Para obtener el detalle completo de un pedido se necesitan varias relaciones.

```sql
SELECT
    p.id_pedido,
    c.nombre AS cliente,
    pr.nombre AS producto,
    d.cantidad,
    pr.precio,
    d.cantidad * pr.precio AS subtotal
FROM pedidos AS p
JOIN clientes AS c
    ON c.id_cliente = p.id_cliente
JOIN detalle_pedido AS d
    ON d.id_pedido = p.id_pedido
JOIN productos AS pr
    ON pr.id_producto = d.id_producto;
```

### Resultado

| id_pedido | cliente | producto | cantidad | precio | subtotal |
| ---: | :--- | :--- | ---: | ---: | ---: |
| 1001 | Ana | Teclado mecánico | 1 | 149.90 | 149.90 |
| 1002 | Luis | Mouse inalámbrico | 1 | 79.90 | 79.90 |
| 1003 | Ana | Monitor 24 pulgadas | 1 | 899.00 | 899.00 |
| 1004 | Marta | Webcam HD | 2 | 219.50 | 439.00 |

### Secuencia de relaciones

```text
pedidos
   ↓ id_cliente
clientes

pedidos
   ↓ id_pedido
 detalle_pedido
   ↓ id_producto
productos
```

> **Consejo.** Antes de escribir una consulta con varios joins, dibuja las tablas y las claves que conectan cada una.

---

## 9. Duplicados y cardinalidad

Un cliente puede tener muchos pedidos. Por eso, al combinar `clientes` y `pedidos`, el nombre del cliente puede aparecer varias veces.

| Relación | Tipo de cardinalidad | Efecto en el resultado |
| :--- | :--- | :--- |
| Un cliente — muchos pedidos | 1:N | El cliente se repite una vez por pedido |
| Un pedido — muchos productos | 1:N | El pedido se repite una vez por detalle |
| Clientes — productos mediante pedidos | N:M | Se necesita una tabla intermedia |

### Ejemplo

Ana tiene dos pedidos:

| nombre | id_pedido |
| :--- | ---: |
| Ana | 1001 |
| Ana | 1003 |

Esto no es un error: cada fila representa una relación diferente.

### Evitar duplicados cuando solo se necesita una lista

```sql
SELECT DISTINCT c.nombre
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

| nombre |
| :--- |
| Ana |
| Luis |
| Marta |

> **Importante.** No uses `DISTINCT` para ocultar duplicados que todavía no comprendes. Primero revisa la cardinalidad y las condiciones del join.

---

## 10. `JOIN` y agregaciones

Los joins suelen combinarse con `COUNT`, `SUM` y `GROUP BY`.

## 10.1 Cantidad de pedidos por cliente

```sql
SELECT
    c.id_cliente,
    c.nombre,
    COUNT(p.id_pedido) AS cantidad_pedidos
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
GROUP BY c.id_cliente, c.nombre;
```

### Resultado

| id_cliente | nombre | cantidad_pedidos |
| ---: | :--- | ---: |
| 101 | Ana | 2 |
| 102 | Luis | 1 |
| 103 | Marta | 1 |
| 104 | Carlos | 0 |

Se utiliza `COUNT(p.id_pedido)` y no `COUNT(*)`, porque Carlos conserva una fila del `LEFT JOIN` aunque el pedido sea `NULL`.

## 10.2 Total comprado por cliente

```sql
SELECT
    c.nombre,
    COALESCE(SUM(p.total), 0) AS total_comprado
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
GROUP BY c.id_cliente, c.nombre;
```

| nombre | total_comprado |
| :--- | ---: |
| Ana | 1048.90 |
| Luis | 79.90 |
| Marta | 45.00 |
| Carlos | 0.00 |

---

## 11. `EXISTS` frente a `JOIN`

Si solo queremos saber si existe una coincidencia, `EXISTS` puede ser más claro que un `JOIN`.

### Con `JOIN`

```sql
SELECT DISTINCT c.nombre
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

### Con `EXISTS`

```sql
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

| Opción | Cuándo usarla |
| :--- | :--- |
| `JOIN` | Necesitas columnas de ambas tablas |
| `EXISTS` | Solo necesitas comprobar que existe una relación |
| `LEFT JOIN ... IS NULL` | Buscas filas sin coincidencia |
| `NOT EXISTS` | Buscas ausencia de coincidencia |

---

## 12. `NATURAL JOIN` y por qué tener cuidado

`NATURAL JOIN` utiliza automáticamente todas las columnas que tienen el mismo nombre.

```sql
SELECT *
FROM clientes
NATURAL JOIN pedidos;
```

Aunque es corto, puede ser peligroso. Si mañana se agrega otra columna con el mismo nombre a ambas tablas, la condición del join cambiará automáticamente.

| Forma | Claridad | Control |
| :--- | :---: | :---: |
| `JOIN ... ON` | Alta | Alto |
| `JOIN ... USING` | Media/alta | Medio |
| `NATURAL JOIN` | Puede ser ambigua | Bajo |

> **Recomendación.** Prefiere `JOIN ... ON` en consultas educativas, profesionales y de producción.

---

## 13. Claves y calidad de los joins

Un buen join normalmente relaciona:

- Una clave primaria con una clave foránea.
- Columnas con tipos compatibles.
- Valores que representan la misma entidad.

```text
clientes.id_cliente      ←→      pedidos.id_cliente
PRIMARY KEY                      FOREIGN KEY
```

### Problemas frecuentes

| Problema | Consecuencia |
| :--- | :--- |
| Unir columnas equivocadas | Resultados incorrectos |
| Omitir parte de una clave compuesta | Filas duplicadas o asociaciones erróneas |
| Tipos incompatibles | Errores o conversiones costosas |
| Datos sin integridad referencial | Filas sin correspondencia |
| Condición incompleta | Producto cartesiano parcial |

### Clave compuesta

Si una tabla tiene una clave compuesta, la relación debe utilizar todas sus columnas:

```sql
ON a.codigo = b.codigo
AND a.version = b.version
```

Usar solo `codigo` podría combinar versiones que no corresponden.

---

## 14. Orden lógico de una consulta con `JOIN`

La forma habitual de escribir una consulta es:

```sql
SELECT columnas
FROM tabla_principal
JOIN tabla_relacionada
    ON condicion_de_relacion
WHERE filtro
GROUP BY columnas
HAVING filtro_de_grupo
ORDER BY columnas
LIMIT cantidad;
```

### Ejemplo completo

```sql
SELECT
    c.nombre,
    COUNT(p.id_pedido) AS cantidad_pedidos
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
WHERE c.ciudad = 'Lima'
GROUP BY c.id_cliente, c.nombre
HAVING COUNT(p.id_pedido) > 0
ORDER BY cantidad_pedidos DESC;
```

### Resultado

| nombre | cantidad_pedidos |
| :--- | ---: |
| Ana | 2 |
| Marta | 1 |

| Cláusula | Función |
| :--- | :--- |
| `FROM` | Define la tabla inicial |
| `JOIN ... ON` | Combina tablas relacionadas |
| `WHERE` | Filtra filas individuales |
| `GROUP BY` | Forma grupos |
| `HAVING` | Filtra grupos |
| `ORDER BY` | Ordena el resultado |

---

## 15. Errores comunes

| Error | Qué ocurre | Cómo evitarlo |
| :--- | :--- | :--- |
| Omitir la condición `ON` | Se produce un producto cartesiano o error | Escribir siempre la relación |
| Usar una columna ambigua | El motor no sabe qué tabla consultar | Utilizar alias, por ejemplo `c.id_cliente` |
| Poner un filtro de `LEFT JOIN` en `WHERE` sin querer | Se pierden filas sin coincidencia | Mover la condición a `ON` |
| Confundir `INNER JOIN` con `LEFT JOIN` | Se eliminan filas que se querían conservar | Elegir el tipo según el objetivo |
| Usar `DISTINCT` para esconder un problema | Se ocultan duplicados reales | Revisar la cardinalidad |
| Unir por una columna que no es clave | Aparecen combinaciones incorrectas | Relacionar claves compatibles |
| Omitir una columna de una clave compuesta | Se multiplican las filas | Incluir todas las columnas de la clave |
| Usar `COUNT(*)` con `LEFT JOIN` | Una fila sin coincidencia puede contar como 1 | Usar `COUNT(clave_de_la_tabla_derecha)` |
| Confundir `ON` con `WHERE` | Cambia la lógica del resultado | Comprender cuándo se aplica cada condición |
| Utilizar `NATURAL JOIN` sin control | La relación puede cambiar al modificar el esquema | Preferir `JOIN ... ON` |
| Hacer muchos joins sin revisar índices | La consulta puede ser lenta | Analizar claves e índices con `EXPLAIN` |

### Ejemplo incorrecto y corregido

```sql
-- Incorrecto: no existe condición de relación
SELECT c.nombre, p.total
FROM clientes AS c, pedidos AS p;
```

```sql
-- Correcto
SELECT c.nombre, p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

---

## 16. Práctica guiada

## Pregunta 1: pedidos con nombre de cliente

Muestra el nombre del cliente y el total de cada pedido.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    c.nombre,
    p.id_pedido,
    p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

</details>

## Pregunta 2: todos los clientes

Muestra todos los clientes, incluso quienes no tienen pedidos.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    c.nombre,
    p.id_pedido
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

</details>

## Pregunta 3: clientes sin pedidos

Obtén los clientes que no tienen ningún pedido.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT c.nombre
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
WHERE p.id_pedido IS NULL;
```

</details>

## Pregunta 4: detalle de pedidos

Muestra el cliente, el pedido y el producto comprado.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    c.nombre AS cliente,
    p.id_pedido,
    pr.nombre AS producto,
    d.cantidad
FROM clientes AS c
JOIN pedidos AS p
    ON p.id_cliente = c.id_cliente
JOIN detalle_pedido AS d
    ON d.id_pedido = p.id_pedido
JOIN productos AS pr
    ON pr.id_producto = d.id_producto;
```

</details>

## Pregunta 5: pedidos de alto valor

Muestra los pedidos cuyo total sea mayor que 100 junto con el nombre del cliente.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    c.nombre,
    p.id_pedido,
    p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
   AND p.total > 100;
```

</details>

---

## 17. Mini desafío final

Usa las tablas del ejemplo para resolver:

1. Mostrar cada cliente junto con sus pedidos.
2. Mostrar todos los clientes, incluyendo los que no tienen pedidos.
3. Encontrar los pedidos cuyo cliente no existe en `clientes`.
4. Mostrar el detalle completo de cada pedido.
5. Calcular el total comprado por cada cliente, incluyendo a quienes tienen cero.
6. Mostrar los clientes que han realizado al menos un pedido.
7. Explicar qué tipo de join se necesita para combinar todas las posibles parejas de clientes y pedidos.

## Una solución posible

```sql
-- 1. Cliente y pedidos
SELECT c.nombre, p.id_pedido, p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;

-- 2. Todos los clientes
SELECT c.nombre, p.id_pedido, p.total
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;

-- 3. Pedidos sin cliente válido
SELECT p.id_pedido, p.id_cliente, p.total
FROM pedidos AS p
LEFT JOIN clientes AS c
    ON c.id_cliente = p.id_cliente
WHERE c.id_cliente IS NULL;

-- 4. Detalle completo
SELECT
    p.id_pedido,
    c.nombre AS cliente,
    pr.nombre AS producto,
    d.cantidad,
    d.cantidad * pr.precio AS subtotal
FROM pedidos AS p
JOIN clientes AS c
    ON c.id_cliente = p.id_cliente
JOIN detalle_pedido AS d
    ON d.id_pedido = p.id_pedido
JOIN productos AS pr
    ON pr.id_producto = d.id_producto;

-- 5. Total por cliente, incluidos los que tienen cero
SELECT
    c.nombre,
    COALESCE(SUM(p.total), 0) AS total_comprado
FROM clientes AS c
LEFT JOIN pedidos AS p
    ON p.id_cliente = c.id_cliente
GROUP BY c.id_cliente, c.nombre;

-- 6. Clientes con al menos un pedido
SELECT c.nombre
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);

-- 7. Todas las combinaciones
SELECT c.nombre, p.id_pedido
FROM clientes AS c
CROSS JOIN pedidos AS p;
```

### Tabla de decisión

| Necesidad | Operación recomendada |
| :--- | :--- |
| Solo coincidencias | `INNER JOIN` |
| Conservar toda la tabla izquierda | `LEFT JOIN` |
| Conservar toda la tabla derecha | `RIGHT JOIN` |
| Conservar ambas tablas | `FULL OUTER JOIN` |
| Todas las combinaciones | `CROSS JOIN` |
| Relacionar una tabla consigo misma | `SELF JOIN` |
| Verificar existencia | `EXISTS` o semijoin |
| Buscar ausencia | `NOT EXISTS` o antijoin |

---

## 18. Resumen final

- `JOIN` combina filas de varias tablas mediante una condición.
- `INNER JOIN` devuelve solo coincidencias.
- `LEFT JOIN` conserva todas las filas de la tabla izquierda.
- `RIGHT JOIN` conserva todas las filas de la tabla derecha.
- `FULL OUTER JOIN` conserva las filas de ambas tablas, coincidan o no.
- `CROSS JOIN` genera el producto cartesiano.
- Un `SELF JOIN` relaciona una tabla consigo misma usando alias.
- `ON` define la relación entre las tablas.
- `WHERE` filtra el resultado después de la combinación.
- En un `LEFT JOIN`, mover una condición de `ON` a `WHERE` puede eliminar las filas con `NULL`.
- Los duplicados pueden ser consecuencia normal de una relación 1:N o N:M.
- `DISTINCT` no debe utilizarse para ocultar una relación mal definida.
- Las tablas intermedias resuelven relaciones de muchos a muchos.
- `COUNT(clave)` suele ser más adecuado que `COUNT(*)` con `LEFT JOIN`.
- `EXISTS` es útil cuando solo interesa comprobar que existe una coincidencia.
- Las claves primarias y foráneas ayudan a construir joins correctos.
- `EXPLAIN` permite analizar el rendimiento de consultas con joins.

> **Qué debes recordar.** Antes de escribir un `JOIN`, identifica qué representa cada tabla, cuál es la clave que las conecta y qué filas deseas conservar aunque no exista coincidencia.

---

## Relacionado con otros temas

- [Álgebra relacional](algebra-relacional.md): explica joins, producto cartesiano, semijoins y antijoins.
- [`SELECT` y `FROM`](../02-consultas-basicas/select-from.md): estructura base de la consulta.
- [WHERE y operadores](../02-consultas-basicas/where-operadores.md): filtra filas combinadas.
- [Funciones SQL](../02-consultas-basicas/funciones-sql.md): permite calcular valores sobre datos unidos.
- [Agregaciones y `GROUP BY`](agregaciones-group-by.md): resume resultados de varios joins.
- [Subconsultas](subqueries.md): ofrece alternativas con `EXISTS` y consultas anidadas.
- [Claves primarias y foráneas](../01-fundamentos/claves-primarias-y-foraneas.md): define las relaciones entre tablas.
- [Índices básicos](../06-indices-performance/indices-basicos.md): puede mejorar el rendimiento de los joins.
