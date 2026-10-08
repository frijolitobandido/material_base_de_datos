# Expresiones `CASE` y `WHEN` en SQL

`CASE` permite devolver resultados diferentes según se cumplan determinadas condiciones.

Es la forma más utilizada de expresar lógica condicional dentro de una consulta SQL.

```sql
CASE
    WHEN condicion THEN resultado
    ELSE resultado_alternativo
END
```

- `CASE` inicia la expresión condicional.
- `WHEN` define una condición.
- `THEN` indica qué devolver si la condición se cumple.
- `ELSE` define el resultado cuando ninguna condición se cumple.
- `END` cierra la expresión.

> **Idea central.** `CASE` no modifica automáticamente los datos: crea un valor calculado para mostrar, ordenar, filtrar o utilizar dentro de otra operación.

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

## 2. Estructura básica de `CASE`

Una expresión `CASE` puede aparecer dentro de `SELECT`.

```sql
SELECT
    nombre,
    stock,
    CASE
        WHEN stock > 0 THEN 'Disponible'
        ELSE 'Agotado'
    END AS estado_stock
FROM productos;
```

### Resultado

| nombre | stock | estado_stock |
| :--- | ---: | :--- |
| Teclado mecánico | 12 | Disponible |
| Mouse inalámbrico | 25 | Disponible |
| Monitor 24 pulgadas | 8 | Disponible |
| Webcam HD | 0 | Agotado |
| Alfombrilla grande | 40 | Disponible |

`CASE` crea la columna calculada `estado_stock`; no modifica la columna `stock`.

### Partes de la expresión

| Parte | Función |
| :--- | :--- |
| `CASE` | Inicia la lógica |
| `WHEN stock > 0` | Comprueba la condición |
| `THEN 'Disponible'` | Resultado si es verdadera |
| `ELSE 'Agotado'` | Resultado alternativo |
| `END` | Cierra la expresión |
| `AS estado_stock` | Asigna el nombre de salida |

---

## 3. `CASE` con varias condiciones

Se pueden escribir varios bloques `WHEN` para representar diferentes rangos.

```sql
SELECT
    nombre,
    stock,
    CASE
        WHEN stock = 0 THEN 'Agotado'
        WHEN stock < 10 THEN 'Poco stock'
        ELSE 'Stock suficiente'
    END AS nivel_stock
FROM productos;
```

### Resultado

| nombre | stock | nivel_stock |
| :--- | ---: | :--- |
| Teclado mecánico | 12 | Stock suficiente |
| Mouse inalámbrico | 25 | Stock suficiente |
| Monitor 24 pulgadas | 8 | Poco stock |
| Webcam HD | 0 | Agotado |
| Alfombrilla grande | 40 | Stock suficiente |

### Orden de evaluación

SQL revisa las condiciones de arriba hacia abajo y se queda con el primer `WHEN` verdadero.

| Producto | Primera condición verdadera | Resultado |
| :--- | :--- | :--- |
| Webcam HD | `stock = 0` | Agotado |
| Monitor | `stock < 10` | Poco stock |
| Teclado | Ninguna anterior | Stock suficiente |

> **Regla importante.** El orden de los `WHEN` importa. Coloca primero las condiciones más específicas y después las generales.

---

## 4. El orden de los `WHEN` puede cambiar el resultado

Observa esta expresión:

```sql
CASE
    WHEN stock >= 0 THEN 'Tiene un valor válido'
    WHEN stock = 0 THEN 'Agotado'
    ELSE 'Otro'
END
```

La condición `stock >= 0` también es verdadera cuando `stock = 0`, por lo que el segundo `WHEN` nunca se alcanza para ese caso.

### Forma incorrecta

| stock | Primer `WHEN` | Resultado |
| ---: | :--- | :--- |
| 0 | `stock >= 0` | Tiene un valor válido |
| 8 | `stock >= 0` | Tiene un valor válido |

### Forma recomendada

```sql
CASE
    WHEN stock = 0 THEN 'Agotado'
    WHEN stock > 0 THEN 'Disponible'
    ELSE 'Valor no válido'
END
```

| stock | Resultado |
| ---: | :--- |
| 0 | Agotado |
| 8 | Disponible |
| `NULL` | Valor no válido |

---

## 5. `ELSE` y valores no contemplados

`ELSE` es opcional. Si se omite y ninguna condición se cumple, el resultado será `NULL`.

### Sin `ELSE`

```sql
SELECT
    nombre,
    CASE
        WHEN precio > 500 THEN 'Premium'
    END AS segmento
FROM productos;
```

### Resultado

| nombre | segmento |
| :--- | :--- |
| Teclado mecánico | `NULL` |
| Mouse inalámbrico | `NULL` |
| Monitor 24 pulgadas | Premium |
| Webcam HD | `NULL` |
| Alfombrilla grande | `NULL` |

### Con `ELSE`

```sql
SELECT
    nombre,
    CASE
        WHEN precio > 500 THEN 'Premium'
        ELSE 'Estándar'
    END AS segmento
FROM productos;
```

| nombre | segmento |
| :--- | :--- |
| Teclado mecánico | Estándar |
| Mouse inalámbrico | Estándar |
| Monitor 24 pulgadas | Premium |
| Webcam HD | Estándar |
| Alfombrilla grande | Estándar |

> **Recomendación.** Usa `ELSE` cuando quieras controlar explícitamente los valores que no cumplen ningún `WHEN`.

---

## 6. `CASE` simple

Existen dos formas principales de escribir `CASE`.

El `CASE` simple compara una expresión con diferentes valores:

```sql
SELECT
    nombre,
    categoria,
    CASE categoria
        WHEN 'periféricos' THEN 'Accesorios'
        WHEN 'monitores' THEN 'Pantallas'
        WHEN 'cámaras' THEN 'Video'
        ELSE 'Otra categoría'
    END AS grupo
FROM productos;
```

### Resultado

| nombre | categoria | grupo |
| :--- | :--- | :--- |
| Teclado mecánico | periféricos | Accesorios |
| Mouse inalámbrico | periféricos | Accesorios |
| Monitor 24 pulgadas | monitores | Pantallas |
| Webcam HD | cámaras | Video |
| Alfombrilla grande | periféricos | Accesorios |

### Comparación de sintaxis

| Tipo | Forma | Cuándo usarlo |
| :--- | :--- | :--- |
| `CASE` simple | `CASE columna WHEN valor THEN ...` | Comparar una misma columna con valores exactos |
| `CASE` buscado | `CASE WHEN condicion THEN ...` | Rangos, operadores y condiciones combinadas |

El `CASE` buscado es más flexible:

```sql
CASE
    WHEN precio < 100 THEN 'Económico'
    WHEN precio < 500 THEN 'Intermedio'
    ELSE 'Premium'
END
```

---

## 7. Clasificar rangos numéricos

`CASE` es útil para crear categorías a partir de rangos.

```sql
SELECT
    nombre,
    precio,
    CASE
        WHEN precio < 100 THEN 'Económico'
        WHEN precio <= 300 THEN 'Intermedio'
        ELSE 'Premium'
    END AS segmento_precio
FROM productos;
```

### Resultado

| nombre | precio | segmento_precio |
| :--- | ---: | :--- |
| Teclado mecánico | 149.90 | Intermedio |
| Mouse inalámbrico | 79.90 | Económico |
| Monitor 24 pulgadas | 899.00 | Premium |
| Webcam HD | 219.50 | Intermedio |
| Alfombrilla grande | 45.00 | Económico |

### Tabla de reglas

| Condición | Etiqueta |
| :--- | :--- |
| `precio < 100` | Económico |
| `precio >= 100 AND precio <= 300` | Intermedio |
| `precio > 300` | Premium |

---

## 8. Clasificar fechas

También pueden clasificarse fechas.

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
    (101, 'Ana', '2026-09-20', 45.90),
    (102, 'Luis', '2026-10-03', 80.00),
    (103, 'Marta', '2026-10-08', 125.50);
```

```sql
SELECT
    id,
    cliente,
    fecha_pedido,
    CASE
        WHEN fecha_pedido < '2026-10-01' THEN 'Anterior'
        WHEN fecha_pedido = '2026-10-01' THEN 'Inicio del mes'
        ELSE 'Posterior'
    END AS periodo
FROM pedidos;
```

### Resultado

| id | cliente | fecha_pedido | periodo |
| ---: | :--- | :--- | :--- |
| 101 | Ana | 2026-09-20 | Anterior |
| 102 | Luis | 2026-10-03 | Posterior |
| 103 | Marta | 2026-10-08 | Posterior |

---

## 9. Combinar condiciones con `AND` y `OR`

Dentro de `WHEN` se pueden combinar varias condiciones.

```sql
SELECT
    nombre,
    precio,
    stock,
    CASE
        WHEN stock = 0 THEN 'Agotado'
        WHEN activo = FALSE THEN 'Inactivo'
        WHEN precio > 500 AND stock < 10 THEN 'Premium con poco stock'
        ELSE 'Disponible'
    END AS estado_comercial
FROM productos;
```

### Resultado

| nombre | precio | stock | activo | estado_comercial |
| :--- | ---: | ---: | :---: | :--- |
| Teclado mecánico | 149.90 | 12 | `TRUE` | Disponible |
| Mouse inalámbrico | 79.90 | 25 | `TRUE` | Disponible |
| Monitor 24 pulgadas | 899.00 | 8 | `TRUE` | Premium con poco stock |
| Webcam HD | 219.50 | 0 | `FALSE` | Agotado |
| Alfombrilla grande | 45.00 | 40 | `TRUE` | Disponible |

La webcam queda como `Agotado` porque esa condición aparece antes que `activo = FALSE`.

> **Revisión.** Cuando varias condiciones podrían cumplirse, decide cuál tiene prioridad y coloca primero el `WHEN` correspondiente.

---

## 10. Trabajar con `NULL` en `CASE`

Las comparaciones con `NULL` no son verdaderas. Si una columna puede ser `NULL`, conviene contemplarlo.

```sql
CREATE TABLE empleados (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    telefono VARCHAR(30)
);
```

```sql
INSERT INTO empleados (id, nombre, telefono)
VALUES
    (1, 'Ana', '555-1000'),
    (2, 'Luis', NULL),
    (3, 'Marta', '555-3000');
```

### Forma recomendada: `IS NULL`

```sql
SELECT
    nombre,
    CASE
        WHEN telefono IS NULL THEN 'Sin teléfono'
        ELSE telefono
    END AS telefono_mostrado
FROM empleados;
```

| nombre | telefono_mostrado |
| :--- | :--- |
| Ana | 555-1000 |
| Luis | Sin teléfono |
| Marta | 555-3000 |

### Forma alternativa: `COALESCE`

```sql
SELECT
    nombre,
    COALESCE(telefono, 'Sin teléfono') AS telefono_mostrado
FROM empleados;
```

Para un reemplazo simple de `NULL`, `COALESCE` suele ser más corto.

> **No lo confundas.** `WHEN telefono = NULL` no es una forma correcta de detectar un valor nulo; utiliza `IS NULL`.

---

## 11. `CASE` dentro de `ORDER BY`

`CASE` permite crear un orden personalizado.

Queremos mostrar primero los productos activos y después los inactivos:

```sql
SELECT
    nombre,
    activo,
    precio
FROM productos
ORDER BY
    CASE WHEN activo = TRUE THEN 0 ELSE 1 END,
    nombre ASC;
```

### Resultado

| orden | nombre | activo | precio |
| ---: | :--- | :---: | ---: |
| 1 | Alfombrilla grande | `TRUE` | 45.00 |
| 2 | Monitor 24 pulgadas | `TRUE` | 899.00 |
| 3 | Mouse inalámbrico | `TRUE` | 79.90 |
| 4 | Teclado mecánico | `TRUE` | 149.90 |
| 5 | Webcam HD | `FALSE` | 219.50 |

El `CASE` devuelve 0 para los activos y 1 para los inactivos; por eso los activos aparecen primero.

### Orden personalizado de estados

```sql
SELECT
    id,
    nombre,
    stock
FROM productos
ORDER BY CASE
    WHEN stock = 0 THEN 1
    WHEN stock < 10 THEN 2
    ELSE 3
END;
```

| Prioridad | Condición | Resultado del `CASE` |
| ---: | :--- | ---: |
| 1 | Agotado | 1 |
| 2 | Poco stock | 2 |
| 3 | Stock suficiente | 3 |

---

## 12. `CASE` dentro de `WHERE`

Una expresión `CASE` puede utilizarse dentro de un filtro, aunque muchas veces una condición directa es más clara.

```sql
SELECT nombre, precio, stock
FROM productos
WHERE CASE
    WHEN activo = TRUE THEN stock
    ELSE 0
END > 0;
```

### Resultado

| nombre | precio | stock |
| :--- | ---: | ---: |
| Teclado mecánico | 149.90 | 12 |
| Mouse inalámbrico | 79.90 | 25 |
| Monitor 24 pulgadas | 899.00 | 8 |
| Alfombrilla grande | 45.00 | 40 |

Una alternativa más directa sería:

```sql
SELECT nombre, precio, stock
FROM productos
WHERE activo = TRUE
  AND stock > 0;
```

> **Regla práctica.** Usa `CASE` en `WHERE` cuando realmente necesites convertir una lógica en un valor. Si basta con combinar condiciones, `AND` y `OR` suelen ser más claros.

---

## 13. `CASE` dentro de `UPDATE`

`CASE` permite asignar valores diferentes según cada fila.

```sql
UPDATE productos
SET categoria = CASE
    WHEN precio >= 500 THEN 'premium'
    WHEN precio >= 100 THEN 'estándar'
    ELSE 'económico'
END;
```

### Simulación del resultado

| nombre | precio | categoria anterior | categoria nueva |
| :--- | ---: | :--- | :--- |
| Teclado mecánico | 149.90 | periféricos | estándar |
| Mouse inalámbrico | 79.90 | periféricos | económico |
| Monitor 24 pulgadas | 899.00 | monitores | premium |
| Webcam HD | 219.50 | cámaras | estándar |
| Alfombrilla grande | 45.00 | periféricos | económico |

Antes de ejecutar el `UPDATE`, conviene revisar la transformación:

```sql
SELECT
    id,
    nombre,
    categoria AS categoria_actual,
    CASE
        WHEN precio >= 500 THEN 'premium'
        WHEN precio >= 100 THEN 'estándar'
        ELSE 'económico'
    END AS categoria_nueva
FROM productos;
```

> **Advertencia.** `CASE` dentro de `UPDATE` sí puede modificar los datos porque forma parte de una operación DML.

---

## 14. `CASE` para calcular descuentos

```sql
SELECT
    nombre,
    precio,
    CASE
        WHEN precio >= 500 THEN ROUND(precio * 0.20, 2)
        WHEN precio >= 100 THEN ROUND(precio * 0.10, 2)
        ELSE 0.00
    END AS descuento_calculado
FROM productos;
```

### Resultado

| nombre | precio | descuento_calculado |
| :--- | ---: | ---: |
| Teclado mecánico | 149.90 | 14.99 |
| Mouse inalámbrico | 79.90 | 0.00 |
| Monitor 24 pulgadas | 899.00 | 179.80 |
| Webcam HD | 219.50 | 21.95 |
| Alfombrilla grande | 45.00 | 0.00 |

### Precio final

```sql
SELECT
    nombre,
    precio,
    precio - CASE
        WHEN precio >= 500 THEN ROUND(precio * 0.20, 2)
        WHEN precio >= 100 THEN ROUND(precio * 0.10, 2)
        ELSE 0.00
    END AS precio_final
FROM productos;
```

| nombre | precio | precio_final |
| :--- | ---: | ---: |
| Teclado mecánico | 149.90 | 134.91 |
| Mouse inalámbrico | 79.90 | 79.90 |
| Monitor 24 pulgadas | 899.00 | 719.20 |
| Webcam HD | 219.50 | 197.55 |
| Alfombrilla grande | 45.00 | 45.00 |

---

## 15. `CASE` con `IN`, `BETWEEN` y `LIKE`

Las condiciones de `WHEN` pueden usar operadores que ya conocemos.

```sql
SELECT
    nombre,
    categoria,
    CASE
        WHEN categoria IN ('monitores', 'cámaras') THEN 'Video y pantalla'
        WHEN nombre LIKE '%inalámbrico%' THEN 'Conectividad'
        ELSE 'Otros accesorios'
    END AS grupo_producto
FROM productos;
```

### Resultado

| nombre | categoria | grupo_producto |
| :--- | :--- | :--- |
| Teclado mecánico | periféricos | Otros accesorios |
| Mouse inalámbrico | periféricos | Conectividad |
| Monitor 24 pulgadas | monitores | Video y pantalla |
| Webcam HD | cámaras | Video y pantalla |
| Alfombrilla grande | periféricos | Otros accesorios |

---

## 16. `CASE` anidado

Es posible colocar un `CASE` dentro de otro, aunque conviene no abusar porque la consulta puede volverse difícil de leer.

```sql
SELECT
    nombre,
    CASE
        WHEN activo = FALSE THEN 'No disponible'
        ELSE CASE
            WHEN stock = 0 THEN 'Agotado'
            WHEN stock < 10 THEN 'Poco stock'
            ELSE 'Disponible'
        END
    END AS estado_final
FROM productos;
```

### Resultado

| nombre | activo | stock | estado_final |
| :--- | :---: | ---: | :--- |
| Teclado mecánico | `TRUE` | 12 | Disponible |
| Mouse inalámbrico | `TRUE` | 25 | Disponible |
| Monitor 24 pulgadas | `TRUE` | 8 | Poco stock |
| Webcam HD | `FALSE` | 0 | No disponible |
| Alfombrilla grande | `TRUE` | 40 | Disponible |

En muchos casos se puede simplificar la misma lógica usando condiciones combinadas:

```sql
CASE
    WHEN activo = FALSE THEN 'No disponible'
    WHEN stock = 0 THEN 'Agotado'
    WHEN stock < 10 THEN 'Poco stock'
    ELSE 'Disponible'
END
```

> **Recomendación.** Antes de anidar `CASE`, intenta ordenar las condiciones en una sola expresión.

---

## 17. Diferencia entre `CASE` y `IF`

Algunos motores ofrecen funciones condicionales propias.

| Forma | Característica |
| :--- | :--- |
| `CASE` | Estándar SQL y permite muchas condiciones |
| `IF` | Función frecuente en MySQL |
| `IIF` | Disponible en algunos motores |
| `DECODE` | Función histórica de Oracle |

### `CASE` portable

```sql
SELECT
    nombre,
    CASE WHEN stock > 0 THEN 'Disponible' ELSE 'Agotado' END AS estado
FROM productos;
```

### `IF` en MySQL

```sql
SELECT
    nombre,
    IF(stock > 0, 'Disponible', 'Agotado') AS estado
FROM productos;
```

Ambas pueden producir el mismo resultado en MySQL, pero `CASE` suele ser preferible cuando se busca compatibilidad entre motores.

---

## 18. `CASE` y tipos de datos

Los resultados de las ramas de `CASE` deberían ser compatibles entre sí.

```sql
SELECT
    nombre,
    CASE
        WHEN stock > 0 THEN 'Disponible'
        ELSE 'Agotado'
    END AS estado
FROM productos;
```

Esta expresión devuelve texto en todas sus ramas.

Evita mezclar resultados sin una razón clara:

```sql
-- Puede provocar conversiones inesperadas
CASE
    WHEN stock > 0 THEN 'Disponible'
    ELSE 0
END
```

### Resultados coherentes e incoherentes

| Ejemplo | Tipo de resultado | Recomendación |
| :--- | :--- | :--- |
| `'Sí'` / `'No'` | Texto | Correcto |
| `1` / `0` | Numérico | Correcto |
| Fechas válidas / `NULL` | Fecha o nulo | Correcto |
| `'Disponible'` / `0` | Mixto | Evitar |

---

## 19. `CASE` y valores `NULL` en condiciones

Si ninguna condición reconoce un `NULL` y no existe `ELSE`, el resultado será `NULL`.

```sql
CREATE TABLE productos_opcionales (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    stock INT
);
```

```sql
SELECT
    nombre,
    CASE
        WHEN stock > 0 THEN 'Disponible'
        WHEN stock = 0 THEN 'Agotado'
        ELSE 'Stock desconocido'
    END AS estado_stock
FROM productos_opcionales;
```

| stock | Resultado |
| ---: | :--- |
| 10 | Disponible |
| 0 | Agotado |
| `NULL` | Stock desconocido |

El `ELSE` permite diferenciar un valor desconocido de un stock igual a cero.

---

## 20. Errores comunes

| Error | Qué ocurre | Cómo evitarlo |
| :--- | :--- | :--- |
| Olvidar `END` | La consulta genera un error de sintaxis | Cerrar siempre el `CASE` |
| Omitir `ELSE` sin intención | Los casos no contemplados devuelven `NULL` | Agregar un resultado alternativo |
| Colocar primero una condición muy general | Los `WHEN` posteriores nunca se evalúan | Ordenar de lo específico a lo general |
| Usar `= NULL` | La condición nunca es verdadera | Utilizar `IS NULL` |
| Mezclar tipos incompatibles | El motor puede convertir valores de forma inesperada | Mantener resultados compatibles |
| Confundir `CASE` en `SELECT` con `UPDATE` | Se espera que la tabla cambie | Recordar que `SELECT` solo muestra |
| Anidar demasiados `CASE` | La consulta se vuelve difícil de mantener | Simplificar condiciones o crear una vista |
| Usar `CASE` para todo un filtro | Puede ser menos claro y menos eficiente | Preferir `AND`, `OR` y `WHERE` cuando corresponda |
| Cambiar los datos sin revisar | `UPDATE` puede modificar muchas filas | Probar primero con `SELECT` |

### Ejemplo incorrecto y corregido

```sql
-- Incorrecto: la condición general captura todos los valores positivos
CASE
    WHEN stock >= 0 THEN 'Válido'
    WHEN stock = 0 THEN 'Agotado'
    ELSE 'Otro'
END
```

```sql
-- Correcto: primero se revisa el caso específico
CASE
    WHEN stock = 0 THEN 'Agotado'
    WHEN stock > 0 THEN 'Disponible'
    ELSE 'Desconocido'
END
```

---

## 21. Práctica guiada

### Pregunta 1: disponibilidad

Clasifica cada producto como `Disponible` o `Agotado` según su stock.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    nombre,
    CASE
        WHEN stock > 0 THEN 'Disponible'
        ELSE 'Agotado'
    END AS estado
FROM productos;
```

</details>

### Pregunta 2: niveles de stock

Clasifica los productos como `Agotado`, `Poco stock` o `Suficiente`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    nombre,
    CASE
        WHEN stock = 0 THEN 'Agotado'
        WHEN stock < 10 THEN 'Poco stock'
        ELSE 'Suficiente'
    END AS nivel_stock
FROM productos;
```

</details>

### Pregunta 3: segmento de precio

Clasifica los precios menores a 100 como `Económico`, los de hasta 300 como `Intermedio` y el resto como `Premium`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    nombre,
    precio,
    CASE
        WHEN precio < 100 THEN 'Económico'
        WHEN precio <= 300 THEN 'Intermedio'
        ELSE 'Premium'
    END AS segmento
FROM productos;
```

</details>

### Pregunta 4: ordenar activos primero

Ordena los productos activos antes que los inactivos usando `CASE` en `ORDER BY`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT nombre, activo
FROM productos
ORDER BY CASE
    WHEN activo = TRUE THEN 0
    ELSE 1
END;
```

</details>

### Pregunta 5: tratar `NULL`

Muestra un teléfono alternativo cuando el valor sea `NULL`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    nombre,
    CASE
        WHEN telefono IS NULL THEN 'Sin teléfono'
        ELSE telefono
    END AS telefono_mostrado
FROM empleados;
```

</details>

---

## 22. Mini desafío final

Usa la tabla `productos` y escribe consultas que:

1. Clasifiquen los productos según su stock.
2. Clasifiquen los productos según su precio.
3. Marquen como `Prioridad` los productos activos con stock menor a 10.
4. Ordenen primero los productos agotados y después los disponibles.
5. Calculen una etiqueta según la categoría.

### Una solución posible

```sql
-- 1. Clasificación por stock
SELECT
    nombre,
    CASE
        WHEN stock = 0 THEN 'Agotado'
        WHEN stock < 10 THEN 'Poco stock'
        ELSE 'Disponible'
    END AS estado_stock
FROM productos;

-- 2. Clasificación por precio
SELECT
    nombre,
    CASE
        WHEN precio < 100 THEN 'Económico'
        WHEN precio <= 300 THEN 'Intermedio'
        ELSE 'Premium'
    END AS segmento_precio
FROM productos;

-- 3. Productos prioritarios
SELECT
    nombre,
    CASE
        WHEN activo = TRUE AND stock < 10 THEN 'Prioridad'
        ELSE 'Normal'
    END AS prioridad
FROM productos;

-- 4. Agotados primero
SELECT nombre, stock
FROM productos
ORDER BY CASE
    WHEN stock = 0 THEN 0
    ELSE 1
END,
 nombre ASC;

-- 5. Etiqueta por categoría
SELECT
    nombre,
    CASE categoria
        WHEN 'periféricos' THEN 'Accesorios'
        WHEN 'monitores' THEN 'Pantallas'
        WHEN 'cámaras' THEN 'Video'
        ELSE 'Otros'
    END AS etiqueta
FROM productos;
```

### Resultado esperado de la consulta de prioridad

| nombre | activo | stock | prioridad |
| :--- | :---: | ---: | :--- |
| Teclado mecánico | `TRUE` | 12 | Normal |
| Mouse inalámbrico | `TRUE` | 25 | Normal |
| Monitor 24 pulgadas | `TRUE` | 8 | Prioridad |
| Webcam HD | `FALSE` | 0 | Normal |
| Alfombrilla grande | `TRUE` | 40 | Normal |

---

## 23. Resumen final

- `CASE` permite crear resultados condicionales.
- `WHEN` define las condiciones.
- `THEN` indica el resultado de una condición verdadera.
- `ELSE` cubre los casos restantes.
- `END` cierra la expresión.
- SQL evalúa los `WHEN` de arriba hacia abajo.
- El primer `WHEN` verdadero determina el resultado.
- El `CASE` simple compara una expresión con valores exactos.
- El `CASE` buscado permite rangos, operadores y condiciones combinadas.
- `CASE` puede utilizarse en `SELECT`, `WHERE`, `ORDER BY` y `UPDATE`.
- Para detectar `NULL`, se debe usar `IS NULL`.
- Las ramas de `CASE` deberían devolver tipos compatibles.
- `CASE` es más portable que funciones condicionales específicas de un motor.

> **Qué debes recordar.** `CASE` convierte reglas de negocio en valores que SQL puede mostrar, ordenar, filtrar o guardar.

---

## Relacionado con otros temas

- [`SELECT` y `FROM`](select-from.md): `CASE` suele utilizarse dentro de `SELECT`.
- [WHERE y operadores](where-operadores.md): permite combinar `CASE` con condiciones.
- [ORDER BY y LIMIT](order-by-limit.md): `CASE` puede crear órdenes personalizados.
- [Funciones SQL](funciones-sql.md): `CASE` se combina con funciones de texto, números y fechas.
- [Agregaciones y `GROUP BY`](../03-consultas-relacionales/agregaciones-group-by.md): `CASE` puede utilizarse en cálculos agrupados.
