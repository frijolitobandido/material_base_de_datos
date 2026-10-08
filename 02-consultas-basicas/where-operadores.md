# Cláusula `WHERE` y operadores de comparación

La cláusula `WHERE` permite filtrar las filas que devuelve una consulta.

Mientras `SELECT` indica qué columnas mostrar y `FROM` indica de qué tabla obtenerlas, `WHERE` establece qué filas cumplen una condición.

```sql
SELECT columnas
FROM tabla
WHERE condicion;
```

> **Idea central.** `WHERE` no cambia los datos: únicamente decide qué filas participan en el resultado de la consulta.

---

## 1. Tabla de ejemplo

Para practicar utilizaremos la tabla `productos`.

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

| id | nombre | categoria | precio | stock | activo |
| ---: | :--- | :--- | ---: | ---: | :---: |
| 1 | Teclado mecánico | periféricos | 149.90 | 12 | `TRUE` |
| 2 | Mouse inalámbrico | periféricos | 79.90 | 25 | `TRUE` |
| 3 | Monitor 24 pulgadas | monitores | 899.00 | 8 | `TRUE` |
| 4 | Webcam HD | cámaras | 219.50 | 0 | `FALSE` |
| 5 | Alfombrilla grande | periféricos | 45.00 | 40 | `TRUE` |

---

## 2. Estructura básica de `WHERE`

```sql
SELECT nombre, precio
FROM productos
WHERE precio > 100;
```

| Parte | Función |
| :--- | :--- |
| `SELECT nombre, precio` | Indica las columnas que se mostrarán |
| `FROM productos` | Indica la tabla de origen |
| `WHERE precio > 100` | Conserva las filas cuyo precio supera 100 |
| `;` | Finaliza la consulta |

### Resultado

| nombre | precio |
| :--- | ---: |
| Teclado mecánico | 149.90 |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |

Los productos con precio menor o igual a 100 quedan fuera del resultado, pero no se eliminan de la tabla.

> **No lo confundas.** `WHERE` filtra el resultado de un `SELECT`; no reemplaza a `DELETE` ni a `UPDATE`.

---

## 3. Operadores de comparación

Los operadores de comparación construyen condiciones.

| Operador | Significado | Ejemplo |
| :---: | :--- | :--- |
| `=` | Igual a | `precio = 79.90` |
| `<>` | Diferente de | `categoria <> 'cámaras'` |
| `!=` | Diferente de, común en varios motores | `stock != 0` |
| `>` | Mayor que | `precio > 100` |
| `<` | Menor que | `precio < 100` |
| `>=` | Mayor o igual que | `stock >= 25` |
| `<=` | Menor o igual que | `precio <= 100` |

`<>` es la forma estándar de SQL para “diferente de”. `!=` también es aceptado por motores populares, pero conviene conocer la diferencia de portabilidad.

---

## 4. Igualdad con `=`

El operador `=` encuentra valores exactamente iguales.

```sql
SELECT id, nombre, precio
FROM productos
WHERE precio = 79.90;
```

### Resultado

| id | nombre | precio |
| ---: | :--- | ---: |
| 2 | Mouse inalámbrico | 79.90 |

Para comparar texto se utilizan comillas simples:

```sql
SELECT id, nombre, categoria
FROM productos
WHERE categoria = 'periféricos';
```

### Resultado

| id | nombre | categoria |
| ---: | :--- | :--- |
| 1 | Teclado mecánico | periféricos |
| 2 | Mouse inalámbrico | periféricos |
| 5 | Alfombrilla grande | periféricos |

### Comparar valores booleanos

```sql
SELECT nombre, activo
FROM productos
WHERE activo = TRUE;
```

| nombre | activo |
| :--- | :---: |
| Teclado mecánico | `TRUE` |
| Mouse inalámbrico | `TRUE` |
| Monitor 24 pulgadas | `TRUE` |
| Alfombrilla grande | `TRUE` |

> **Consejo.** Los textos se escriben entre comillas simples; los números normalmente no necesitan comillas.

---

## 5. Diferente de: `<>` y `!=`

```sql
SELECT nombre, categoria
FROM productos
WHERE categoria <> 'periféricos';
```

### Resultado

| nombre | categoria |
| :--- | :--- |
| Monitor 24 pulgadas | monitores |
| Webcam HD | cámaras |

La forma estándar equivalente es:

```sql
SELECT nombre, categoria
FROM productos
WHERE categoria != 'periféricos';
```

| Forma | Portabilidad | Recomendación |
| :--- | :--- | :--- |
| `<>` | Estándar SQL | Preferible si se busca mayor portabilidad |
| `!=` | Aceptada por muchos motores | Útil, pero revisar el motor objetivo |

> **Atención con `NULL`.** Una condición de “diferente de” tampoco incluye automáticamente los valores `NULL`. Para `NULL` se usa `IS NULL` o `IS NOT NULL`.

---

## 6. Mayor y menor que

### Mayor que: `>`

```sql
SELECT nombre, precio
FROM productos
WHERE precio > 200;
```

| nombre | precio |
| :--- | ---: |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |

El valor `200` no se incluye porque la condición exige que sea estrictamente mayor.

### Menor que: `<`

```sql
SELECT nombre, stock
FROM productos
WHERE stock < 10;
```

| nombre | stock |
| :--- | ---: |
| Monitor 24 pulgadas | 8 |
| Webcam HD | 0 |

El stock `10` tampoco se incluiría.

---

## 7. Mayor o igual y menor o igual

### Mayor o igual: `>=`

```sql
SELECT nombre, stock
FROM productos
WHERE stock >= 25;
```

| nombre | stock |
| :--- | ---: |
| Mouse inalámbrico | 25 |
| Alfombrilla grande | 40 |

El valor `25` sí se incluye porque la condición permite igualdad.

### Menor o igual: `<=`

```sql
SELECT nombre, precio
FROM productos
WHERE precio <= 79.90;
```

| nombre | precio |
| :--- | ---: |
| Mouse inalámbrico | 79.90 |
| Alfombrilla grande | 45.00 |

### Comparar los límites

| Condición | ¿Incluye el límite? | Valores de ejemplo que coinciden |
| :--- | :---: | :--- |
| `precio > 100` | No | 149.90, 219.50, 899.00 |
| `precio >= 100` | Sí | También incluiría exactamente 100.00 |
| `precio < 100` | No | 45.00, 79.90 |
| `precio <= 100` | Sí | También incluiría exactamente 100.00 |

---

## 8. Comparar fechas

Los operadores de comparación también funcionan con fechas cuando se almacenan en tipos adecuados.

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

### Pedidos posteriores a una fecha

```sql
SELECT id, cliente, fecha_pedido
FROM pedidos
WHERE fecha_pedido > '2026-10-01';
```

| id | cliente | fecha_pedido |
| ---: | :--- | :--- |
| 102 | Luis | 2026-10-04 |
| 103 | Marta | 2026-10-08 |

La fecha `2026-10-01` no aparece porque se utilizó `>` y no `>=`.

> **Formato recomendado.** Usa fechas en formato `AAAA-MM-DD` para evitar ambigüedades.

---

## 9. Combinar condiciones con `AND`

`AND` exige que todas las condiciones sean verdaderas.

```sql
SELECT nombre, precio, stock
FROM productos
WHERE precio > 100
  AND stock > 0;
```

### Resultado

La condición `stock > 0` debe cumplirse además de `precio > 100`, por lo que la webcam queda fuera del resultado.

| nombre | precio | stock |
| :--- | ---: | ---: |
| Teclado mecánico | 149.90 | 12 |
| Monitor 24 pulgadas | 899.00 | 8 |

### Evaluación de `AND`

| `precio > 100` | `stock > 0` | ¿Aparece? |
| :---: | :---: | :---: |
| Verdadero | Verdadero | Sí |
| Verdadero | Falso | No |
| Falso | Verdadero | No |
| Falso | Falso | No |

> **Regla práctica.** Con `AND`, una fila queda fuera si falla aunque sea una de las condiciones.

---

## 10. Combinar condiciones con `OR`

`OR` acepta la fila cuando al menos una condición es verdadera.

```sql
SELECT nombre, categoria, precio
FROM productos
WHERE categoria = 'monitores'
   OR precio < 50;
```

### Resultado

| nombre | categoria | precio |
| :--- | :--- | ---: |
| Monitor 24 pulgadas | monitores | 899.00 |
| Alfombrilla grande | periféricos | 45.00 |

### Evaluación de `OR`

| Primera condición | Segunda condición | ¿Aparece? |
| :---: | :---: | :---: |
| Verdadera | Verdadera | Sí |
| Verdadera | Falsa | Sí |
| Falsa | Verdadera | Sí |
| Falsa | Falsa | No |

---

## 11. Paréntesis y prioridad de condiciones

Los paréntesis hacen explícita la lógica de una consulta.

```sql
SELECT nombre, categoria, precio, activo
FROM productos
WHERE activo = TRUE
  AND (categoria = 'periféricos' OR precio > 500);
```

### Resultado

| nombre | categoria | precio | activo |
| :--- | :--- | ---: | :---: |
| Teclado mecánico | periféricos | 149.90 | `TRUE` |
| Mouse inalámbrico | periféricos | 79.90 | `TRUE` |
| Monitor 24 pulgadas | monitores | 899.00 | `TRUE` |
| Alfombrilla grande | periféricos | 45.00 | `TRUE` |

La webcam queda fuera porque no está activa.

### Sin paréntesis

En términos generales, `NOT` tiene prioridad sobre `AND`, y `AND` tiene prioridad sobre `OR`.

```sql
-- Puede resultar difícil de leer
WHERE activo = TRUE
  AND categoria = 'periféricos'
  OR precio > 500;
```

Se recomienda escribirlo así:

```sql
WHERE activo = TRUE
  AND (categoria = 'periféricos' OR precio > 500);
```

| Recomendación | Motivo |
| :--- | :--- |
| Usar paréntesis | Hace visible la intención lógica |
| Dividir condiciones en líneas | Facilita la revisión |
| Probar con un `SELECT` | Permite comprobar las filas afectadas |

---

## 12. Negar una condición con `NOT`

`NOT` invierte una condición.

```sql
SELECT nombre, activo
FROM productos
WHERE NOT activo = TRUE;
```

### Resultado

| nombre | activo |
| :--- | :---: |
| Webcam HD | `FALSE` |

Una forma más directa sería:

```sql
SELECT nombre, activo
FROM productos
WHERE activo = FALSE;
```

### `NOT` con paréntesis

```sql
SELECT nombre, categoria
FROM productos
WHERE NOT (categoria = 'periféricos');
```

Esto devuelve las categorías diferentes de `periféricos`, con la misma advertencia sobre los valores `NULL`.

---

## 13. `BETWEEN`: comparar un rango

`BETWEEN` comprueba si un valor está dentro de un rango **incluyendo ambos límites**.

```sql
SELECT nombre, precio
FROM productos
WHERE precio BETWEEN 50 AND 250;
```

### Resultado

| nombre | precio |
| :--- | ---: |
| Teclado mecánico | 149.90 |
| Webcam HD | 219.50 |
| Mouse inalámbrico | 79.90 |

La condición equivale a:

```sql
WHERE precio >= 50
  AND precio <= 250
```

### Verificar los límites

| Precio | ¿Está entre 50 y 250? | Motivo |
| ---: | :---: | :--- |
| 45.00 | No | Menor que 50 |
| 50.00 | Sí | Límite inferior incluido |
| 149.90 | Sí | Está dentro del rango |
| 250.00 | Sí | Límite superior incluido |
| 899.00 | No | Mayor que 250 |

> **Cuidado.** `BETWEEN 50 AND 250` incluye 50 y 250.

---

## 14. `IN`: comparar con una lista

`IN` comprueba si un valor coincide con alguno de los valores de una lista.

```sql
SELECT nombre, categoria
FROM productos
WHERE categoria IN ('periféricos', 'cámaras');
```

### Resultado

| nombre | categoria |
| :--- | :--- |
| Teclado mecánico | periféricos |
| Mouse inalámbrico | periféricos |
| Webcam HD | cámaras |
| Alfombrilla grande | periféricos |

La consulta equivale a:

```sql
WHERE categoria = 'periféricos'
   OR categoria = 'cámaras'
```

### `NOT IN`

```sql
SELECT nombre, categoria
FROM productos
WHERE categoria NOT IN ('periféricos', 'cámaras');
```

| nombre | categoria |
| :--- | :--- |
| Monitor 24 pulgadas | monitores |

> **Advertencia sobre `NULL`.** `NOT IN` puede producir resultados inesperados si la lista contiene `NULL`. Para valores ausentes se debe usar `IS NULL`.

---

## 15. `LIKE`: comparar patrones de texto

`LIKE` permite buscar texto mediante patrones.

| Símbolo | Significado |
| :---: | :--- |
| `%` | Cero o más caracteres |
| `_` | Exactamente un carácter |

### Comienza con un texto

```sql
SELECT nombre
FROM productos
WHERE nombre LIKE 'Mouse%';
```

| nombre |
| :--- |
| Mouse inalámbrico |

### Contiene un texto

```sql
SELECT nombre
FROM productos
WHERE nombre LIKE '%inalámbrico%';
```

| nombre |
| :--- |
| Mouse inalámbrico |

### Termina con un texto

```sql
SELECT nombre
FROM productos
WHERE nombre LIKE '%HD';
```

| nombre |
| :--- |
| Webcam HD |

### Un carácter desconocido

```sql
SELECT nombre
FROM productos
WHERE nombre LIKE 'Mouse _________';
```

El comportamiento exacto de mayúsculas, minúsculas y acentos depende de la configuración de comparación del motor.

> **Diferencia importante.** `=` busca coincidencia exacta; `LIKE` busca coincidencia con un patrón.

---

## 16. `IS NULL` e `IS NOT NULL`

No se debe comparar `NULL` usando `=` o `<>`.

### Consulta incorrecta

```sql
-- No es la forma correcta de buscar valores NULL
SELECT nombre, telefono
FROM empleados
WHERE telefono = NULL;
```

### Consulta correcta

```sql
SELECT nombre, telefono
FROM empleados
WHERE telefono IS NULL;
```

| nombre | telefono |
| :--- | :--- |
| Luis | `NULL` |

### Buscar valores que sí existen

```sql
SELECT nombre, telefono
FROM empleados
WHERE telefono IS NOT NULL;
```

| nombre | telefono |
| :--- | :--- |
| Ana | 555-1000 |

### Comparación de sintaxis

| Lo que buscas | Sintaxis correcta |
| :--- | :--- |
| Valor igual a 0 | `columna = 0` |
| Texto vacío | `columna = ''` |
| Ausencia de valor | `columna IS NULL` |
| Cualquier valor presente | `columna IS NOT NULL` |

> **Qué debes recordar.** `NULL` representa un valor desconocido o ausente. Por eso se utiliza `IS NULL`, no `= NULL`.

---

## 17. La lógica de tres valores

En SQL una comparación puede producir:

- `TRUE` — la condición se cumple.
- `FALSE` — la condición no se cumple.
- `UNKNOWN` — no se puede determinar, normalmente por la presencia de `NULL`.

```sql
SELECT nombre, telefono
FROM empleados
WHERE telefono <> '555-1000';
```

El teléfono `NULL` de Luis no se incluye como “diferente”, porque la comparación con `NULL` produce `UNKNOWN`, no `TRUE`.

| teléfono | Condición `telefono <> '555-1000'` | ¿Pasa el `WHERE`? |
| :--- | :---: | :---: |
| `555-1000` | `FALSE` | No |
| `NULL` | `UNKNOWN` | No |
| `555-2000` | `TRUE` | Sí |

`WHERE` conserva únicamente las filas cuya condición es `TRUE`.

---

## 18. Combinar `WHERE` con alias y expresiones

Un alias creado en `SELECT` normalmente no puede utilizarse en `WHERE` del mismo nivel, porque `WHERE` se evalúa antes de construir el resultado final.

```sql
-- Puede fallar según el motor
SELECT
    nombre,
    precio * 1.18 AS precio_con_impuesto
FROM productos
WHERE precio_con_impuesto > 200;
```

La forma segura es repetir la expresión:

```sql
SELECT
    nombre,
    precio * 1.18 AS precio_con_impuesto
FROM productos
WHERE precio * 1.18 > 200;
```

### Resultado

| nombre | precio_con_impuesto |
| :--- | ---: |
| Monitor 24 pulgadas | 1060.82 |
| Webcam HD | 259.01 |

> **Nota.** Algunos motores permiten alias en determinadas cláusulas, pero para mantener la consulta portable conviene no depender de ese comportamiento en `WHERE`.

---

## 19. `WHERE` en `SELECT`, `UPDATE` y `DELETE`

La cláusula `WHERE` no pertenece únicamente a `SELECT`.

| Comando | Uso de `WHERE` | Riesgo si se omite |
| :--- | :--- | :--- |
| `SELECT` | Filtra filas mostradas | Se muestran más filas de las esperadas |
| `UPDATE` | Filtra filas modificadas | Se modifican todas las filas |
| `DELETE` | Filtra filas eliminadas | Se eliminan todas las filas |

### Comprobar antes de modificar

```sql
-- Primero revisar
SELECT id, nombre, precio
FROM productos
WHERE categoria = 'periféricos';

-- Ejecutar solo después de revisar
UPDATE productos
SET precio = precio * 1.10
WHERE categoria = 'periféricos';
```

> **Advertencia.** Antes de ejecutar un `UPDATE` o `DELETE`, prueba la misma condición con `SELECT`.

---

## 20. Errores comunes

| Error | Qué ocurre | Cómo evitarlo |
| :--- | :--- | :--- |
| Escribir texto sin comillas | El motor interpreta el texto como columna | Usar comillas simples |
| Usar `= NULL` | La comparación no devuelve `TRUE` | Usar `IS NULL` |
| Confundir `>` con `>=` | Se excluye o incluye el límite incorrectamente | Revisar si el límite debe incluirse |
| Olvidar paréntesis con `AND` y `OR` | La consulta puede devolver filas inesperadas | Agrupar la lógica explícitamente |
| Usar `NOT IN` con `NULL` sin analizarlo | Puede no devolver lo esperado | Tratar los `NULL` por separado |
| Usar `LIKE` cuando se necesita igualdad exacta | Coinciden más valores de los deseados | Usar `=` |
| Omitir `WHERE` en `UPDATE` | Se actualiza toda la tabla | Probar primero con `SELECT` |
| Omitir `WHERE` en `DELETE` | Se eliminan todos los registros | Revisar la condición antes |
| Comparar fechas con formato ambiguo | El motor puede interpretar mal la fecha | Usar `AAAA-MM-DD` |
| Suponer que los acentos siempre se comparan igual | Depende de la collation | Revisar configuración del motor |

### Ejemplo incorrecto y corregido

```sql
-- Incorrecto: busca NULL con igualdad
SELECT *
FROM empleados
WHERE telefono = NULL;
```

```sql
-- Correcto
SELECT *
FROM empleados
WHERE telefono IS NULL;
```

---

## 21. Práctica guiada

### Pregunta 1: precio mínimo

Muestra los productos cuyo precio sea mayor que 100.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, precio
FROM productos
WHERE precio > 100;
```

</details>

### Pregunta 2: stock disponible

Muestra los productos que tienen stock mayor que cero.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, stock
FROM productos
WHERE stock > 0;
```

</details>

### Pregunta 3: dos condiciones

Muestra los productos activos cuyo stock sea mayor que cero.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, stock, activo
FROM productos
WHERE activo = TRUE
  AND stock > 0;
```

</details>

### Pregunta 4: rango de precios

Muestra los productos cuyo precio esté entre 50 y 250, incluyendo los límites.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, precio
FROM productos
WHERE precio BETWEEN 50 AND 250;
```

</details>

### Pregunta 5: lista de categorías

Muestra los productos de las categorías `monitores` o `cámaras`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, categoria
FROM productos
WHERE categoria IN ('monitores', 'cámaras');
```

</details>

### Pregunta 6: valor ausente

Muestra los empleados que no tienen teléfono registrado.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, telefono
FROM empleados
WHERE telefono IS NULL;
```

</details>

---

## 22. Mini desafío final

Usa la tabla `productos` y escribe consultas para obtener:

1. Los productos activos con precio menor o igual a 200.
2. Los productos de categoría `periféricos` o `monitores` con stock disponible.
3. Los productos cuyo nombre contenga la palabra `inalámbrico`.
4. Los productos cuyo precio no esté entre 50 y 250.
5. Los productos con stock igual a cero o inactivos.

### Una solución posible

```sql
-- 1. Activos con precio menor o igual a 200
SELECT nombre, precio, activo
FROM productos
WHERE activo = TRUE
  AND precio <= 200;

-- 2. Categorías específicas con stock
SELECT nombre, categoria, stock
FROM productos
WHERE categoria IN ('periféricos', 'monitores')
  AND stock > 0;

-- 3. Nombre que contiene un texto
SELECT nombre
FROM productos
WHERE nombre LIKE '%inalámbrico%';

-- 4. Precio fuera del rango inclusivo
SELECT nombre, precio
FROM productos
WHERE NOT (precio BETWEEN 50 AND 250);

-- 5. Sin stock o inactivos
SELECT nombre, stock, activo
FROM productos
WHERE stock = 0
   OR activo = FALSE;
```

### Resultado esperado de la consulta 1

| nombre | precio | activo |
| :--- | ---: | :---: |
| Teclado mecánico | 149.90 | `TRUE` |
| Mouse inalámbrico | 79.90 | `TRUE` |
| Alfombrilla grande | 45.00 | `TRUE` |

### Resultado esperado de la consulta 2

| nombre | categoria | stock |
| :--- | :--- | ---: |
| Teclado mecánico | periféricos | 12 |
| Mouse inalámbrico | periféricos | 25 |
| Monitor 24 pulgadas | monitores | 8 |
| Alfombrilla grande | periféricos | 40 |

---

## 23. Resumen final

- `WHERE` filtra las filas de una consulta.
- `=` compara igualdad.
- `<>` es el operador estándar para “diferente de”.
- `!=` también es común, pero su uso depende de la compatibilidad del motor.
- `>`, `<`, `>=` y `<=` comparan valores numéricos, texto o fechas según el tipo.
- `AND` exige que todas las condiciones se cumplan.
- `OR` exige que al menos una condición se cumpla.
- `NOT` invierte una condición.
- `BETWEEN` incluye los dos límites del rango.
- `IN` compara con una lista de valores.
- `LIKE` permite buscar patrones usando `%` y `_`.
- `IS NULL` e `IS NOT NULL` son la forma correcta de trabajar con `NULL`.
- Los paréntesis ayudan a evitar errores de lógica.
- Siempre conviene probar con `SELECT` antes de usar `UPDATE` o `DELETE`.

> **Qué debes recordar.** `WHERE` no modifica la tabla: selecciona las filas que cumplen una condición y deja fuera las demás del resultado.

---

## Relacionado con otros temas

- [`SELECT` y `FROM`](select-from.md): forman la base de la consulta filtrada.
- [Tipos de datos](../01-fundamentos/tipos-de-datos.md): determinan cómo se comparan los valores.
- [DDL, DML y DQL](../01-fundamentos/ddl-dml-dql.md): `WHERE` se utiliza con consultas y operaciones DML.
- [ORDER BY y LIMIT](order-by-limit.md): permiten ordenar y limitar los resultados filtrados.
- [Funciones SQL](funciones-sql.md): permiten transformar valores antes de compararlos.
- [Joins](../03-consultas-relacionales/joins.md): permiten filtrar información combinada de varias tablas.
