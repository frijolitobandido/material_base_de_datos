# Funciones SQL

Las funciones SQL permiten transformar, calcular o analizar valores dentro de una consulta.

Una función recibe uno o varios valores y devuelve un resultado:

```sql
FUNCION(argumento1, argumento2)
```

Por ejemplo:

```sql
SELECT UPPER(nombre)
FROM clientes;
```

Esta consulta transforma los nombres a mayúsculas únicamente en el resultado mostrado.

En esta guía estudiaremos principalmente funciones escalares, es decir, funciones que trabajan con cada fila de forma individual:

- Funciones de texto.
- Funciones numéricas.
- Funciones de fecha y hora.
- Funciones para trabajar con `NULL`.
- Funciones condicionales básicas.

> **Idea central.** Una función puede transformar un valor para mostrarlo o compararlo, pero no modifica la tabla a menos que se utilice dentro de una operación como `UPDATE`.

---

## 1. Tablas de ejemplo

Usaremos una tabla de clientes:

```sql
CREATE TABLE clientes (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    apellido VARCHAR(100),
    correo VARCHAR(150),
    telefono VARCHAR(30),
    ciudad VARCHAR(80),
    fecha_registro DATE
);
```

```sql
INSERT INTO clientes
    (id, nombre, apellido, correo, telefono, ciudad, fecha_registro)
VALUES
    (1, 'Ana', 'García', 'ana@mail.com', '555-1000', 'Lima', '2026-09-01'),
    (2, 'Luis', 'Pérez', 'luis@mail.com', NULL, 'Cusco', '2026-09-15'),
    (3, 'Marta', 'Rojas', 'marta@mail.com', '555-3000', 'Lima', '2026-10-01'),
    (4, 'Carlos', 'Torres', NULL, '555-4000', 'Arequipa', '2026-10-05');
```

| id | nombre | apellido | correo | telefono | ciudad | fecha_registro |
| ---: | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Ana | García | ana@mail.com | 555-1000 | Lima | 2026-09-01 |
| 2 | Luis | Pérez | luis@mail.com | `NULL` | Cusco | 2026-09-15 |
| 3 | Marta | Rojas | marta@mail.com | 555-3000 | Lima | 2026-10-01 |
| 4 | Carlos | Torres | `NULL` | 555-4000 | Arequipa | 2026-10-05 |

También usaremos una tabla de productos:

```sql
CREATE TABLE productos (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    precio DECIMAL(10,2),
    stock INT,
    descuento DECIMAL(5,2),
    fecha_ingreso DATE
);
```

```sql
INSERT INTO productos
    (id, nombre, precio, stock, descuento, fecha_ingreso)
VALUES
    (1, 'Teclado mecánico', 149.90, 12, 10.00, '2026-09-10'),
    (2, 'Mouse inalámbrico', 79.90, 25, NULL, '2026-09-20'),
    (3, 'Monitor 24 pulgadas', 899.00, 8, 15.00, '2026-10-02'),
    (4, 'Webcam HD', 219.50, 0, 5.00, '2026-10-05');
```

---

## 2. Sintaxis general de una función

Una función puede aparecer en `SELECT`, `WHERE`, `ORDER BY` o `UPDATE`.

```sql
SELECT funcion(columna) AS resultado
FROM tabla;
```

| Parte | Función |
| :--- | :--- |
| `funcion` | Operación que se aplicará |
| `columna` | Valor de cada fila que se procesará |
| `AS resultado` | Nombre visible del resultado |
| `FROM tabla` | Origen de los datos |

### Ejemplo

```sql
SELECT
    nombre,
    UPPER(nombre) AS nombre_mayusculas
FROM clientes;
```

| nombre | nombre_mayusculas |
| :--- | :--- |
| Ana | ANA |
| Luis | LUIS |
| Marta | MARTA |
| Carlos | CARLOS |

La función no cambia la columna `nombre`; solo transforma lo que se muestra.

---

## 3. Funciones de texto

Las funciones de texto sirven para cambiar mayúsculas, unir valores, buscar partes de una cadena o eliminar espacios innecesarios.

## 3.1 `UPPER`: convertir a mayúsculas

```sql
SELECT
    nombre,
    UPPER(nombre) AS nombre_mayusculas
FROM clientes;
```

| nombre | nombre_mayusculas |
| :--- | :--- |
| Ana | ANA |
| Luis | LUIS |
| Marta | MARTA |
| Carlos | CARLOS |

## 3.2 `LOWER`: convertir a minúsculas

```sql
SELECT
    correo,
    LOWER(correo) AS correo_normalizado
FROM clientes
WHERE correo IS NOT NULL;
```

| correo | correo_normalizado |
| :--- | :--- |
| ana@mail.com | ana@mail.com |
| luis@mail.com | luis@mail.com |
| marta@mail.com | marta@mail.com |

`LOWER` puede ser útil para normalizar valores antes de compararlos, aunque el comportamiento de mayúsculas y minúsculas también depende de la collation.

## 3.3 `CONCAT`: unir textos

`CONCAT` combina dos o más valores.

```sql
SELECT
    nombre,
    apellido,
    CONCAT(nombre, ' ', apellido) AS nombre_completo
FROM clientes;
```

| nombre | apellido | nombre_completo |
| :--- | :--- | :--- |
| Ana | García | Ana García |
| Luis | Pérez | Luis Pérez |
| Marta | Rojas | Marta Rojas |
| Carlos | Torres | Carlos Torres |

> **Cuidado con `NULL`.** El comportamiento de `CONCAT` cuando uno de sus argumentos es `NULL` puede variar entre motores. `CONCAT_WS` suele ser útil cuando se necesita ignorar separadores para valores nulos.

## 3.4 `CONCAT_WS`: unir usando un separador

```sql
SELECT
    CONCAT_WS(' - ', ciudad, apellido) AS etiqueta
FROM clientes;
```

| etiqueta |
| :--- |
| Lima - García |
| Cusco - Pérez |
| Lima - Rojas |
| Arequipa - Torres |

El primer argumento es el separador. Los siguientes argumentos son los valores que se unen.

## 3.5 `LENGTH` y `CHAR_LENGTH`

Estas funciones calculan la longitud de un texto, pero pueden medir cosas diferentes:

| Función | Qué mide normalmente |
| :--- | :--- |
| `LENGTH` | Bytes |
| `CHAR_LENGTH` | Caracteres |

```sql
SELECT
    nombre,
    LENGTH(nombre) AS cantidad_bytes,
    CHAR_LENGTH(nombre) AS cantidad_caracteres
FROM clientes;
```

| nombre | cantidad_bytes | cantidad_caracteres |
| :--- | ---: | ---: |
| Ana | 3 | 3 |
| Luis | 4 | 4 |
| Marta | 5 | 5 |
| Carlos | 6 | 6 |

Con caracteres acentuados, el número de bytes puede ser mayor que el número de caracteres, según la codificación.

> **Compatibilidad.** Algunos motores utilizan `LEN` para longitud de caracteres. Revisa la función equivalente si cambias de SGBD.

## 3.6 `TRIM`: quitar espacios

```sql
SELECT
    TRIM('   SQL   ') AS texto_limpio;
```

| texto_limpio |
| :--- |
| SQL |

También puede utilizarse con una columna:

```sql
SELECT
    nombre,
    TRIM(nombre) AS nombre_sin_espacios
FROM clientes;
```

`TRIM` es útil para limpiar valores importados desde archivos o formularios.

## 3.7 `LTRIM` y `RTRIM`

| Función | Acción |
| :--- | :--- |
| `LTRIM` | Elimina espacios del inicio |
| `RTRIM` | Elimina espacios del final |
| `TRIM` | Elimina espacios del inicio y del final |

```sql
SELECT
    LTRIM('   SQL') AS izquierda,
    RTRIM('SQL   ') AS derecha,
    TRIM('   SQL   ') AS ambos;
```

| izquierda | derecha | ambos |
| :--- | :--- | :--- |
| SQL | SQL | SQL |

## 3.8 `SUBSTRING`: extraer una parte

La sintaxis puede variar según el motor. En MySQL es común:

```sql
SELECT
    nombre,
    SUBSTRING(nombre, 1, 3) AS primeros_tres
FROM clientes;
```

| nombre | primeros_tres |
| :--- | :--- |
| Ana | Ana |
| Luis | Lui |
| Marta | Mar |
| Carlos | Car |

La posición inicial y la longitud exacta deben revisarse según el SGBD.

## 3.9 `LEFT` y `RIGHT`

```sql
SELECT
    nombre,
    LEFT(nombre, 2) AS inicio,
    RIGHT(nombre, 2) AS final
FROM clientes;
```

| nombre | inicio | final |
| :--- | :--- | :--- |
| Ana | An | na |
| Luis | Lu | is |
| Marta | Ma | ta |
| Carlos | Ca | os |

## 3.10 `REPLACE`: reemplazar texto

```sql
SELECT
    correo,
    REPLACE(correo, '@mail.com', '@empresa.com') AS correo_empresa
FROM clientes
WHERE correo IS NOT NULL;
```

| correo | correo_empresa |
| :--- | :--- |
| ana@mail.com | ana@empresa.com |
| luis@mail.com | luis@empresa.com |
| marta@mail.com | marta@empresa.com |

Esta operación solo cambia el resultado de la consulta.

---

## 4. Funciones numéricas

Las funciones numéricas permiten redondear, calcular valores absolutos, obtener restos y realizar operaciones matemáticas.

## 4.1 `ROUND`: redondear

```sql
SELECT
    nombre,
    precio,
    ROUND(precio * 1.18, 2) AS precio_con_impuesto
FROM productos;
```

| nombre | precio | precio_con_impuesto |
| :--- | ---: | ---: |
| Teclado mecánico | 149.90 | 176.88 |
| Mouse inalámbrico | 79.90 | 94.28 |
| Monitor 24 pulgadas | 899.00 | 1060.82 |
| Webcam HD | 219.50 | 259.01 |

El segundo argumento indica cuántos decimales conservar.

```sql
SELECT
    ROUND(15.678, 2) AS dos_decimales,
    ROUND(15.678, 0) AS entero_redondeado;
```

| dos_decimales | entero_redondeado |
| ---: | ---: |
| 15.68 | 16 |

## 4.2 `CEIL` y `CEILING`

Redondean hacia arriba:

```sql
SELECT
    CEIL(15.1) AS resultado_ceil,
    CEILING(15.1) AS resultado_ceiling;
```

| resultado_ceil | resultado_ceiling |
| ---: | ---: |
| 16 | 16 |

## 4.3 `FLOOR`

Redondea hacia abajo:

```sql
SELECT FLOOR(15.9) AS resultado;
```

| resultado |
| ---: |
| 15 |

### Comparación

| Valor | `ROUND` | `CEIL` | `FLOOR` |
| ---: | ---: | ---: | ---: |
| 15.1 | 15 | 16 | 15 |
| 15.5 | 16 | 16 | 15 |
| 15.9 | 16 | 16 | 15 |
| -15.1 | -15 | -15 | -16 |

## 4.4 `ABS`: valor absoluto

```sql
SELECT
    ABS(-25) AS valor_positivo,
    ABS(25) AS valor_original;
```

| valor_positivo | valor_original |
| ---: | ---: |
| 25 | 25 |

Puede utilizarse para calcular diferencias sin importar el signo:

```sql
SELECT
    nombre,
    ABS(precio - 100) AS diferencia_con_100
FROM productos;
```

## 4.5 `MOD`: resto de una división

```sql
SELECT
    MOD(10, 3) AS resto;
```

| resto |
| ---: |
| 1 |

También puede aparecer el operador `%` en algunos motores:

```sql
SELECT 10 % 3 AS resto;
```

> **Compatibilidad.** `MOD` suele ser más explícita y legible. Verifica la sintaxis disponible en el motor que uses.

## 4.6 Calcular descuentos

```sql
SELECT
    nombre,
    precio,
    descuento,
    ROUND(precio * (1 - descuento / 100), 2) AS precio_final
FROM productos
WHERE descuento IS NOT NULL;
```

| nombre | precio | descuento | precio_final |
| :--- | ---: | ---: | ---: |
| Teclado mecánico | 149.90 | 10.00 | 134.91 |
| Monitor 24 pulgadas | 899.00 | 15.00 | 764.15 |
| Webcam HD | 219.50 | 5.00 | 208.53 |

Este cálculo no cambia `precio`; solo produce `precio_final` en el resultado.

---

## 5. Funciones de fecha y hora

Las funciones de fecha permiten obtener la fecha actual, extraer partes de una fecha y calcular diferencias.

## 5.1 Fecha y hora actuales

| Función frecuente | Resultado |
| :--- | :--- |
| `CURRENT_DATE` | Fecha actual |
| `CURRENT_TIME` | Hora actual |
| `CURRENT_TIMESTAMP` | Fecha y hora actuales |

```sql
SELECT
    CURRENT_DATE AS fecha_actual,
    CURRENT_TIME AS hora_actual,
    CURRENT_TIMESTAMP AS momento_actual;
```

El resultado depende del momento y de la zona horaria de la sesión.

> **Nota.** `NOW()` es frecuente en MySQL, pero `CURRENT_TIMESTAMP` suele ser más portable.

## 5.2 `YEAR`, `MONTH` y `DAY`

Estas funciones extraen partes de una fecha.

```sql
SELECT
    fecha_ingreso,
    YEAR(fecha_ingreso) AS anio,
    MONTH(fecha_ingreso) AS mes,
    DAY(fecha_ingreso) AS dia
FROM productos;
```

| fecha_ingreso | anio | mes | dia |
| :--- | ---: | ---: | ---: |
| 2026-09-10 | 2026 | 9 | 10 |
| 2026-09-20 | 2026 | 9 | 20 |
| 2026-10-02 | 2026 | 10 | 2 |
| 2026-10-05 | 2026 | 10 | 5 |

## 5.3 `DAYNAME` y `MONTHNAME`

Algunos motores ofrecen funciones para obtener el nombre del día o del mes.

```sql
SELECT
    fecha_ingreso,
    MONTHNAME(fecha_ingreso) AS nombre_mes
FROM productos;
```

El idioma del resultado depende de la configuración del motor.

| Función | Propósito |
| :--- | :--- |
| `DAYNAME` | Nombre del día |
| `MONTHNAME` | Nombre del mes |

## 5.4 `DATE_FORMAT` en MySQL

MySQL permite dar formato a las fechas con `DATE_FORMAT`.

```sql
SELECT
    fecha_ingreso,
    DATE_FORMAT(fecha_ingreso, '%d/%m/%Y') AS fecha_formateada
FROM productos;
```

| fecha_ingreso | fecha_formateada |
| :--- | :--- |
| 2026-09-10 | 10/09/2026 |
| 2026-09-20 | 20/09/2026 |
| 2026-10-02 | 02/10/2026 |
| 2026-10-05 | 05/10/2026 |

> **Compatibilidad.** `DATE_FORMAT` es propio de MySQL. PostgreSQL utiliza funciones como `TO_CHAR`, y SQL Server utiliza otras funciones de formato.

## 5.5 Diferencia entre fechas

La sintaxis cambia según el motor. En MySQL se puede utilizar `DATEDIFF`:

```sql
SELECT
    nombre,
    DATEDIFF(CURRENT_DATE, fecha_ingreso) AS dias_desde_ingreso
FROM productos;
```

| nombre | fecha_ingreso | dias_desde_ingreso |
| :--- | :--- | ---: |
| Teclado mecánico | 2026-09-10 | Depende de la fecha actual |
| Mouse inalámbrico | 2026-09-20 | Depende de la fecha actual |
| Monitor 24 pulgadas | 2026-10-02 | Depende de la fecha actual |
| Webcam HD | 2026-10-05 | Depende de la fecha actual |

## 5.6 Sumar o restar fechas

En MySQL:

```sql
SELECT
    nombre,
    fecha_ingreso,
    DATE_ADD(fecha_ingreso, INTERVAL 30 DAY) AS fecha_revision
FROM productos;
```

| nombre | fecha_ingreso | fecha_revision |
| :--- | :--- | :--- |
| Teclado mecánico | 2026-09-10 | 2026-10-10 |
| Mouse inalámbrico | 2026-09-20 | 2026-10-20 |
| Monitor 24 pulgadas | 2026-10-02 | 2026-11-01 |
| Webcam HD | 2026-10-05 | 2026-11-04 |

Estas funciones son útiles para vencimientos, revisiones, renovaciones y fechas de entrega.

---

## 6. Funciones para trabajar con `NULL`

Los valores `NULL` representan ausencia o desconocimiento de información.

## 6.1 `COALESCE`: elegir el primer valor no nulo

```sql
SELECT
    nombre,
    COALESCE(telefono, 'Sin teléfono') AS telefono_mostrado
FROM clientes;
```

| nombre | telefono_mostrado |
| :--- | :--- |
| Ana | 555-1000 |
| Luis | Sin teléfono |
| Marta | 555-3000 |
| Carlos | 555-4000 |

`COALESCE` revisa los valores de izquierda a derecha y devuelve el primero que no sea `NULL`.

```sql
SELECT COALESCE(NULL, NULL, 'valor disponible', 'otro valor') AS resultado;
```

| resultado |
| :--- |
| valor disponible |

## 6.2 `NULLIF`: convertir una igualdad en `NULL`

`NULLIF(a, b)` devuelve `NULL` si `a` y `b` son iguales; de lo contrario devuelve `a`.

```sql
SELECT
    NULLIF(0, 0) AS resultado_igual,
    NULLIF(10, 0) AS resultado_distinto;
```

| resultado_igual | resultado_distinto |
| :---: | ---: |
| `NULL` | 10 |

Una utilidad común es evitar divisiones entre cero:

```sql
SELECT
    nombre,
    precio / NULLIF(stock, 0) AS precio_por_unidad_stock
FROM productos;
```

Si `stock` es cero, el divisor se convierte en `NULL` en lugar de intentar dividir entre cero.

## 6.3 `IFNULL` en MySQL

MySQL ofrece `IFNULL(valor, reemplazo)`:

```sql
SELECT
    nombre,
    IFNULL(telefono, 'Sin teléfono') AS telefono_mostrado
FROM clientes;
```

| nombre | telefono_mostrado |
| :--- | :--- |
| Ana | 555-1000 |
| Luis | Sin teléfono |
| Marta | 555-3000 |
| Carlos | 555-4000 |

`COALESCE` suele ser preferible cuando se busca mayor compatibilidad, porque permite más de dos alternativas y forma parte del estándar SQL.

### Comparación

| Necesidad | Opción |
| :--- | :--- |
| Elegir entre dos valores en MySQL | `IFNULL` |
| Elegir entre varias alternativas | `COALESCE` |
| Usar una forma más portable | `COALESCE` |
| Convertir una igualdad en `NULL` | `NULLIF` |

---

## 7. Funciones condicionales básicas

Las funciones condicionales permiten crear resultados diferentes según una condición.

## 7.1 `CASE`

```sql
SELECT
    nombre,
    stock,
    CASE
        WHEN stock = 0 THEN 'Agotado'
        WHEN stock < 10 THEN 'Poco stock'
        ELSE 'Disponible'
    END AS estado_stock
FROM productos;
```

### Resultado

| nombre | stock | estado_stock |
| :--- | ---: | :--- |
| Teclado mecánico | 12 | Disponible |
| Mouse inalámbrico | 25 | Disponible |
| Monitor 24 pulgadas | 8 | Poco stock |
| Webcam HD | 0 | Agotado |

`CASE` se estudiará con mayor profundidad en la guía `case-when.md`, pero aquí se muestra su relación con las funciones y expresiones.

## 7.2 `CASE` para clasificar precios

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

| nombre | precio | segmento |
| :--- | ---: | :--- |
| Teclado mecánico | 149.90 | Intermedio |
| Mouse inalámbrico | 79.90 | Económico |
| Monitor 24 pulgadas | 899.00 | Premium |
| Webcam HD | 219.50 | Intermedio |

---

## 8. Combinar varias funciones

Las funciones pueden anidarse, es decir, una función puede recibir como argumento el resultado de otra.

```sql
SELECT
    LOWER(TRIM(correo)) AS correo_limpio
FROM clientes
WHERE correo IS NOT NULL;
```

### Orden de procesamiento

| Paso | Operación |
| ---: | :--- |
| 1 | `TRIM` elimina espacios externos |
| 2 | `LOWER` convierte el resultado a minúsculas |
| 3 | `AS correo_limpio` asigna el nombre visible |

### Combinar texto y valores nulos

```sql
SELECT
    CONCAT(
        UPPER(nombre),
        ' — ',
        COALESCE(telefono, 'sin teléfono')
    ) AS ficha_cliente
FROM clientes;
```

| ficha_cliente |
| :--- |
| ANA — 555-1000 |
| LUIS — sin teléfono |
| MARTA — 555-3000 |
| CARLOS — 555-4000 |

### Combinar funciones numéricas

```sql
SELECT
    nombre,
    ROUND(
        precio * (1 - COALESCE(descuento, 0) / 100),
        2
    ) AS precio_final
FROM productos;
```

| nombre | descuento | precio_final |
| :--- | ---: | ---: |
| Teclado mecánico | 10.00 | 134.91 |
| Mouse inalámbrico | `NULL` | 79.90 |
| Monitor 24 pulgadas | 15.00 | 764.15 |
| Webcam HD | 5.00 | 208.53 |

---

## 9. Funciones en `WHERE` y `ORDER BY`

Las funciones también pueden utilizarse para filtrar y ordenar.

### Función en `WHERE`

```sql
SELECT nombre, correo
FROM clientes
WHERE LOWER(ciudad) = 'lima';
```

| nombre | correo |
| :--- | :--- |
| Ana | ana@mail.com |
| Marta | marta@mail.com |

### Función en `ORDER BY`

```sql
SELECT nombre, precio
FROM productos
ORDER BY ROUND(precio, 0) DESC;
```

| nombre | precio |
| :--- | ---: |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |
| Teclado mecánico | 149.90 |
| Mouse inalámbrico | 79.90 |

> **Rendimiento.** Aplicar una función sobre una columna en `WHERE` puede impedir que el motor aproveche un índice de forma eficiente. Cuando sea posible, compara la columna directamente o utiliza un índice funcional si el motor lo admite.

---

## 10. Funciones en `UPDATE`

Una función puede utilizarse para modificar datos, pero en ese caso la operación deja de ser una simple consulta.

```sql
UPDATE clientes
SET correo = LOWER(TRIM(correo))
WHERE correo IS NOT NULL;
```

### Antes y después

| id | correo antes | correo después |
| ---: | :--- | :--- |
| 1 | ` Ana@Mail.com ` | `ana@mail.com` |
| 2 | `luis@mail.com` | `luis@mail.com` |
| 3 | `MARTA@MAIL.COM` | `marta@mail.com` |
| 4 | `NULL` | `NULL` |

> **Precaución.** Antes de usar funciones dentro de `UPDATE`, realiza un `SELECT` con la misma expresión para verificar el resultado.

```sql
SELECT
    id,
    correo,
    LOWER(TRIM(correo)) AS correo_nuevo
FROM clientes
WHERE correo IS NOT NULL;
```

---

## 11. Funciones escalares frente a funciones agregadas

Esta guía se concentra en funciones escalares, que producen un resultado por cada fila.

| Tipo | Resultado | Ejemplo |
| :--- | :--- | :--- |
| Escalar | Un valor por fila | `UPPER(nombre)` |
| Agregada | Un valor para varias filas | `COUNT(*)` |
| Ventana | Un valor por fila considerando un grupo | `ROW_NUMBER()` |

### Ejemplo escalar

```sql
SELECT nombre, UPPER(nombre)
FROM clientes;
```

| Entrada | Salida |
| :--- | :--- |
| Ana | ANA |
| Luis | LUIS |
| Marta | MARTA |
| Carlos | CARLOS |

Las funciones agregadas como `COUNT`, `SUM`, `AVG`, `MIN` y `MAX` se estudiarán junto con `GROUP BY`.

---

## 12. Compatibilidad entre motores

No todas las funciones tienen el mismo nombre o sintaxis en todos los SGBD.

| Necesidad | MySQL | PostgreSQL | Nota |
| :--- | :--- | :--- | :--- |
| Fecha actual | `CURRENT_DATE` / `CURDATE()` | `CURRENT_DATE` | La forma estándar es más portable |
| Fecha y hora | `CURRENT_TIMESTAMP` / `NOW()` | `CURRENT_TIMESTAMP` / `NOW()` | Revisar zona horaria |
| Longitud de caracteres | `CHAR_LENGTH()` | `CHAR_LENGTH()` | `LENGTH()` puede medir bytes |
| Formato de fecha | `DATE_FORMAT()` | `TO_CHAR()` | Sintaxis diferente |
| Reemplazo de `NULL` | `IFNULL()` | `COALESCE()` | `COALESCE` es más portable |
| Redondeo hacia arriba | `CEIL()` | `CEIL()` | Suele coincidir |
| Texto a mayúsculas | `UPPER()` | `UPPER()` | Generalmente compatible |

> **Regla práctica.** Aprende primero la idea de la operación y después revisa la sintaxis exacta del motor que utilizarás.

---

## 13. Errores comunes

| Error | Qué ocurre | Cómo evitarlo |
| :--- | :--- | :--- |
| Confundir transformación con modificación | `UPPER` en `SELECT` no guarda el texto en mayúsculas | Usar `UPDATE` si se necesita persistir el cambio |
| Aplicar funciones a `NULL` sin revisarlo | El resultado puede ser `NULL` | Usar `COALESCE` o filtrar con `IS NOT NULL` |
| Dividir entre cero | La consulta puede fallar o devolver error | Usar `NULLIF(divisor, 0)` |
| Confundir `LENGTH` y `CHAR_LENGTH` | Se cuentan bytes en vez de caracteres | Elegir la función según lo que se necesita medir |
| Usar una función de otro motor | La sintaxis no se reconoce | Revisar la documentación del SGBD |
| Dar formato a fechas como texto demasiado pronto | Se dificulta ordenar o comparar | Mantener el tipo fecha y formatear al mostrar |
| Anidar funciones sin alias | El resultado es difícil de entender | Asignar nombres descriptivos con `AS` |
| Aplicar funciones en `WHERE` sin revisar índices | La consulta puede volverse más lenta | Comparar directamente o usar índices adecuados |
| Usar `CONCAT` con `NULL` sin analizarlo | El resultado puede ser inesperado | Usar `COALESCE` o `CONCAT_WS` |

### Ejemplo inseguro y corregido

```sql
-- Puede provocar división por cero si stock vale 0
SELECT precio / stock AS precio_por_unidad
FROM productos;
```

```sql
-- Evita dividir entre cero
SELECT precio / NULLIF(stock, 0) AS precio_por_unidad
FROM productos;
```

---

## 14. Práctica guiada

### Pregunta 1: nombre completo

Muestra el nombre completo de cada cliente en una sola columna.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    CONCAT(nombre, ' ', apellido) AS nombre_completo
FROM clientes;
```

</details>

### Pregunta 2: correo normalizado

Muestra los correos sin espacios externos y en minúsculas.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    LOWER(TRIM(correo)) AS correo_normalizado
FROM clientes
WHERE correo IS NOT NULL;
```

</details>

### Pregunta 3: precio redondeado

Muestra el nombre y el precio con impuesto del 18 %, redondeado a dos decimales.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    nombre,
    ROUND(precio * 1.18, 2) AS precio_con_impuesto
FROM productos;
```

</details>

### Pregunta 4: teléfono alternativo

Muestra el teléfono y reemplaza los valores `NULL` por `Sin teléfono`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    nombre,
    COALESCE(telefono, 'Sin teléfono') AS telefono_mostrado
FROM clientes;
```

</details>

### Pregunta 5: estado del stock

Clasifica los productos como `Agotado`, `Poco stock` o `Disponible`.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT
    nombre,
    CASE
        WHEN stock = 0 THEN 'Agotado'
        WHEN stock < 10 THEN 'Poco stock'
        ELSE 'Disponible'
    END AS estado_stock
FROM productos;
```

</details>

---

## 15. Mini desafío final

Usa las tablas `clientes` y `productos` para crear consultas que:

1. Muestren el nombre completo en mayúsculas.
2. Muestren un correo alternativo con `sin correo` cuando el valor sea `NULL`.
3. Calculen el precio final usando el descuento, tratando un descuento `NULL` como cero.
4. Calculen el valor del inventario redondeado a dos decimales.
5. Clasifiquen cada producto según su stock.
6. Muestren el mes de ingreso de cada producto.

### Una solución posible

```sql
-- 1. Nombre completo en mayúsculas
SELECT
    UPPER(CONCAT(nombre, ' ', apellido)) AS nombre_completo
FROM clientes;

-- 2. Correo alternativo
SELECT
    nombre,
    COALESCE(correo, 'sin correo') AS correo_mostrado
FROM clientes;

-- 3. Precio final con descuento
SELECT
    nombre,
    ROUND(
        precio * (1 - COALESCE(descuento, 0) / 100),
        2
    ) AS precio_final
FROM productos;

-- 4. Valor del inventario
SELECT
    nombre,
    ROUND(precio * stock, 2) AS valor_inventario
FROM productos;

-- 5. Clasificación del stock
SELECT
    nombre,
    CASE
        WHEN stock = 0 THEN 'Agotado'
        WHEN stock < 10 THEN 'Poco stock'
        ELSE 'Disponible'
    END AS estado_stock
FROM productos;

-- 6. Mes de ingreso
SELECT
    nombre,
    MONTH(fecha_ingreso) AS mes_ingreso
FROM productos;
```

### Resultado esperado de la consulta de precios

| nombre | descuento | precio_final |
| :--- | ---: | ---: |
| Teclado mecánico | 10.00 | 134.91 |
| Mouse inalámbrico | `NULL` | 79.90 |
| Monitor 24 pulgadas | 15.00 | 764.15 |
| Webcam HD | 5.00 | 208.53 |

---

## 16. Resumen final

- Las funciones transforman o calculan valores dentro de las consultas.
- Las funciones de texto incluyen `UPPER`, `LOWER`, `CONCAT`, `TRIM`, `SUBSTRING` y `REPLACE`.
- Las funciones numéricas incluyen `ROUND`, `CEIL`, `FLOOR`, `ABS` y `MOD`.
- Las funciones de fecha permiten obtener partes de una fecha y trabajar con fechas actuales.
- `COALESCE` devuelve el primer valor no nulo.
- `NULLIF` convierte una igualdad en `NULL` y ayuda a evitar divisiones entre cero.
- `IFNULL` es una alternativa frecuente en MySQL.
- `CASE` permite crear resultados condicionales.
- Una función dentro de `SELECT` transforma el resultado, pero no modifica la tabla.
- Las funciones también pueden utilizarse en `WHERE`, `ORDER BY` y `UPDATE`.
- Las funciones exactas y su sintaxis pueden variar entre motores.
- Las funciones agregadas se estudiarán junto con `GROUP BY`.

> **Qué debes recordar.** Una función SQL recibe valores, aplica una operación y devuelve un resultado que puede mostrarse, compararse o utilizarse dentro de otra expresión.

---

## Relacionado con otros temas

- [`SELECT` y `FROM`](select-from.md): las funciones suelen utilizarse dentro de `SELECT`.
- [WHERE y operadores](where-operadores.md): permite filtrar usando funciones.
- [ORDER BY y LIMIT](order-by-limit.md): permite ordenar por resultados calculados.
- [CASE y WHEN](case-when.md): desarrolla con más profundidad las expresiones condicionales.
- [Agregaciones y `GROUP BY`](../03-consultas-relacionales/agregaciones-group-by.md): explica `COUNT`, `SUM`, `AVG`, `MIN` y `MAX`.
- [Tipos de datos](../01-fundamentos/tipos-de-datos.md): determinan qué funciones pueden aplicarse a cada valor.
