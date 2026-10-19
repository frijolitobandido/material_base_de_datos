# Álgebra relacional

El álgebra relacional es un lenguaje formal para consultar y transformar relaciones, que en la práctica representamos mediante tablas.

Sirve como base teórica para comprender cómo funcionan muchas consultas SQL, especialmente los filtros, las proyecciones, las combinaciones entre tablas y las operaciones de conjuntos.

> **Idea central.** Una operación de álgebra relacional recibe una o más relaciones y produce otra relación como resultado.

---

## 1. ¿Qué es una relación?

En el modelo relacional, una relación puede representarse como una tabla formada por:

- **Atributos:** las columnas de la relación.
- **Tuplas:** las filas de la relación.
- **Dominio:** los valores permitidos para un atributo.
- **Esquema:** el nombre de la relación y sus atributos.

Por ejemplo, la relación `PRODUCTOS` puede tener el esquema:

```text
PRODUCTOS(id, nombre, categoria, precio, stock)
```

Y estas tuplas:

| id | nombre | categoria | precio | stock |
| ---: | :--- | :--- | ---: | ---: |
| 1 | Teclado mecánico | periféricos | 149.90 | 12 |
| 2 | Mouse inalámbrico | periféricos | 79.90 | 25 |
| 3 | Monitor 24 pulgadas | monitores | 899.00 | 8 |
| 4 | Webcam HD | cámaras | 219.50 | 0 |
| 5 | Alfombrilla grande | periféricos | 45.00 | 40 |

En SQL, esta relación se parece al resultado de:

```sql
SELECT id, nombre, categoria, precio, stock
FROM productos;
```

---

## 2. Características del álgebra relacional

| Característica | Explicación |
| :--- | :--- |
| Formal | Utiliza símbolos y reglas precisas |
| Procedimental | Describe qué operaciones aplicar y en qué orden |
| Cerrada | El resultado de una operación también es una relación |
| Basada en conjuntos | Trabaja conceptualmente con conjuntos de tuplas |
| Base de SQL | Ayuda a comprender filtros, proyecciones y joins |
| Independiente del motor | No depende directamente de MySQL, PostgreSQL u otro SGBD |

### Álgebra relacional y SQL

| Álgebra relacional | SQL aproximado |
| :--- | :--- |
| Selección `σ` | `WHERE` |
| Proyección `π` | `SELECT columnas` |
| Renombramiento `ρ` | `AS` |
| Producto cartesiano `×` | `CROSS JOIN` |
| Join `⋈` | `JOIN ... ON` |
| Unión `∪` | `UNION` |
| Intersección `∩` | `INTERSECT` |
| Diferencia `−` | `EXCEPT` o `MINUS` |
| Agregación extendida | `GROUP BY`, `COUNT`, `SUM` |

> **Importante.** SQL no es idéntico al álgebra relacional pura: SQL permite duplicados, valores `NULL`, ordenamiento y otras características que no forman parte del modelo clásico de conjuntos.

---

## 3. Relaciones de ejemplo

Usaremos las siguientes relaciones durante la guía.

### `PRODUCTOS`

```text
PRODUCTOS(id_producto, nombre, categoria, precio, stock)
```

| id_producto | nombre | categoria | precio | stock |
| ---: | :--- | :--- | ---: | ---: |
| 1 | Teclado mecánico | periféricos | 149.90 | 12 |
| 2 | Mouse inalámbrico | periféricos | 79.90 | 25 |
| 3 | Monitor 24 pulgadas | monitores | 899.00 | 8 |
| 4 | Webcam HD | cámaras | 219.50 | 0 |
| 5 | Alfombrilla grande | periféricos | 45.00 | 40 |

### `CLIENTES`

```text
CLIENTES(id_cliente, nombre, ciudad)
```

| id_cliente | nombre | ciudad |
| ---: | :--- | :--- |
| 101 | Ana | Lima |
| 102 | Luis | Cusco |
| 103 | Marta | Lima |
| 104 | Carlos | Arequipa |

### `PEDIDOS`

```text
PEDIDOS(id_pedido, id_cliente, fecha, total)
```

| id_pedido | id_cliente | fecha | total |
| ---: | ---: | :--- | ---: |
| 1001 | 101 | 2026-10-01 | 149.90 |
| 1002 | 102 | 2026-10-02 | 79.90 |
| 1003 | 101 | 2026-10-05 | 899.00 |
| 1004 | 103 | 2026-10-08 | 45.00 |

### `DETALLE_PEDIDO`

```text
DETALLE_PEDIDO(id_pedido, id_producto, cantidad)
```

| id_pedido | id_producto | cantidad |
| ---: | ---: | ---: |
| 1001 | 1 | 1 |
| 1002 | 2 | 1 |
| 1003 | 3 | 1 |
| 1004 | 5 | 1 |

---

## 4. Notación básica

| Símbolo | Nombre | Función general |
| :---: | :--- | :--- |
| `σ` | Selección | Filtra tuplas |
| `π` | Proyección | Elige atributos |
| `ρ` | Renombramiento | Cambia nombres |
| `∪` | Unión | Combina tuplas de dos relaciones |
| `∩` | Intersección | Conserva tuplas comunes |
| `−` | Diferencia | Conserva tuplas de la primera que no están en la segunda |
| `×` | Producto cartesiano | Combina cada tupla con todas las de otra relación |
| `⋈` | Join | Combina tuplas relacionadas |
| `÷` | División | Resuelve consultas del tipo “para todos” |

### Forma general

```text
operador_condicion(relacion)
```

Ejemplo:

```text
σ precio > 100 (PRODUCTOS)
```

Se lee: “seleccionar de `PRODUCTOS` las tuplas cuyo precio sea mayor que 100”.

---

## 5. Operador de selección `σ`

La selección filtra filas de una relación según una condición.

```text
σ condición (RELACION)
```

La selección no elimina columnas; únicamente decide qué tuplas permanecen.

## 5.1 Selección por precio

```text
σ precio > 100 (PRODUCTOS)
```

### Resultado

| id_producto | nombre | categoria | precio | stock |
| ---: | :--- | :--- | ---: | ---: |
| 1 | Teclado mecánico | periféricos | 149.90 | 12 |
| 3 | Monitor 24 pulgadas | monitores | 899.00 | 8 |
| 4 | Webcam HD | cámaras | 219.50 | 0 |

### Equivalencia aproximada en SQL

```sql
SELECT *
FROM productos
WHERE precio > 100;
```

## 5.2 Selección por categoría

```text
σ categoria = 'periféricos' (PRODUCTOS)
```

| id_producto | nombre | categoria | precio | stock |
| ---: | :--- | :--- | ---: | ---: |
| 1 | Teclado mecánico | periféricos | 149.90 | 12 |
| 2 | Mouse inalámbrico | periféricos | 79.90 | 25 |
| 5 | Alfombrilla grande | periféricos | 45.00 | 40 |

```sql
SELECT *
FROM productos
WHERE categoria = 'periféricos';
```

## 5.3 Selección con varias condiciones

```text
σ precio > 100 AND stock > 0 (PRODUCTOS)
```

| id_producto | nombre | precio | stock |
| ---: | :--- | ---: | ---: |
| 1 | Teclado mecánico | 149.90 | 12 |
| 3 | Monitor 24 pulgadas | 899.00 | 8 |

La webcam no aparece porque su stock es cero.

```sql
SELECT *
FROM productos
WHERE precio > 100
  AND stock > 0;
```

## 5.4 Selección con `OR`

```text
σ categoria = 'monitores' OR stock = 0 (PRODUCTOS)
```

| id_producto | nombre | categoria | stock |
| ---: | :--- | :--- | ---: |
| 3 | Monitor 24 pulgadas | monitores | 8 |
| 4 | Webcam HD | cámaras | 0 |

## 5.5 Selecciones encadenadas

Una condición compuesta puede expresarse como una sola selección:

```text
σ precio > 100 AND stock > 0 (PRODUCTOS)
```

O como dos selecciones encadenadas:

```text
σ stock > 0 (σ precio > 100 (PRODUCTOS))
```

Ambas producen el mismo resultado en este caso.

| Forma | Lectura |
| :--- | :--- |
| Una selección | Aplica todas las condiciones juntas |
| Selecciones encadenadas | Filtra primero por una condición y luego por otra |

> **Regla práctica.** Las selecciones suelen poder aplicarse antes de un join para reducir la cantidad de filas que deben combinarse.

---

## 6. Operador de proyección `π`

La proyección selecciona columnas de una relación.

```text
π atributo1, atributo2 (RELACION)
```

A diferencia de `σ`, la proyección trabaja principalmente con atributos, no con filas.

## 6.1 Proyectar nombre y precio

```text
π nombre, precio (PRODUCTOS)
```

| nombre | precio |
| :--- | ---: |
| Teclado mecánico | 149.90 |
| Mouse inalámbrico | 79.90 |
| Monitor 24 pulgadas | 899.00 |
| Webcam HD | 219.50 |
| Alfombrilla grande | 45.00 |

```sql
SELECT nombre, precio
FROM productos;
```

## 6.2 Proyección después de una selección

```text
π nombre, precio (σ stock > 0 (PRODUCTOS))
```

| nombre | precio |
| :--- | ---: |
| Teclado mecánico | 149.90 |
| Mouse inalámbrico | 79.90 |
| Monitor 24 pulgadas | 899.00 |
| Alfombrilla grande | 45.00 |

```sql
SELECT nombre, precio
FROM productos
WHERE stock > 0;
```

## 6.3 Proyección y eliminación de duplicados

En el álgebra relacional clásica, una relación es un conjunto, por lo que las tuplas duplicadas se eliminan.

Supongamos:

```text
π ciudad (CLIENTES)
```

### Resultado relacional

| ciudad |
| :--- |
| Lima |
| Cusco |
| Arequipa |

`Lima` aparece una sola vez aunque dos clientes vivan allí.

En SQL, la equivalencia debe escribirse con `DISTINCT`:

```sql
SELECT DISTINCT ciudad
FROM clientes;
```

> **Diferencia importante.** `SELECT ciudad FROM clientes` puede devolver duplicados en SQL, mientras que la proyección clásica elimina repeticiones.

---

## 7. Combinar selección y proyección

Una consulta suele necesitar primero filtrar filas y luego elegir columnas.

```text
π nombre, precio (σ categoria = 'periféricos' AND stock > 0 (PRODUCTOS))
```

### Resultado

| nombre | precio |
| :--- | ---: |
| Teclado mecánico | 149.90 |
| Mouse inalámbrico | 79.90 |
| Alfombrilla grande | 45.00 |

### Equivalencia SQL

```sql
SELECT nombre, precio
FROM productos
WHERE categoria = 'periféricos'
  AND stock > 0;
```

### Lectura paso a paso

| Paso | Operación | Resultado |
| ---: | :--- | :--- |
| 1 | `σ categoria = 'periféricos'` | Conserva tres productos |
| 2 | `σ stock > 0` | Los tres tienen stock |
| 3 | `π nombre, precio` | Conserva solo dos columnas |

---

## 8. Operador de renombramiento `ρ`

El renombramiento permite cambiar el nombre de una relación o de sus atributos.

```text
ρ NUEVO_NOMBRE (RELACION)
```

## 8.1 Renombrar una relación

```text
ρ P (PRODUCTOS)
```

Ahora podemos referirnos a `PRODUCTOS` como `P` dentro de una expresión más grande.

En SQL:

```sql
SELECT p.nombre, p.precio
FROM productos AS p;
```

## 8.2 Renombrar atributos

```text
ρ P(id, descripcion, tipo, precio, unidades) (PRODUCTOS)
```

La relación conserva los datos, pero cambia la forma de referirse a sus atributos.

| Nombre original | Nombre nuevo |
| :--- | :--- |
| `id_producto` | `id` |
| `nombre` | `descripcion` |
| `categoria` | `tipo` |
| `precio` | `precio` |
| `stock` | `unidades` |

En SQL se puede utilizar `AS` para cambiar nombres en el resultado:

```sql
SELECT
    id_producto AS id,
    nombre AS descripcion,
    categoria AS tipo,
    stock AS unidades
FROM productos;
```

> **Nota.** El renombramiento relacional es una operación formal; un alias en SQL normalmente cambia la referencia o el nombre visible del resultado, no el esquema permanente de la tabla.

---

## 9. Operador de unión `∪`

La unión combina las tuplas de dos relaciones compatibles.

```text
R ∪ S
```

Para que dos relaciones sean compatibles deben tener:

1. La misma cantidad de atributos.
2. Atributos correspondientes con dominios compatibles.
3. El mismo significado en las posiciones equivalentes.

## 9.1 Ejemplo de unión

Supongamos estas relaciones:

```text
CLIENTES_LIMA(id_cliente, nombre)
```

| id_cliente | nombre |
| ---: | :--- |
| 101 | Ana |
| 103 | Marta |

```text
CLIENTES_CUSCO(id_cliente, nombre)
```

| id_cliente | nombre |
| ---: | :--- |
| 102 | Luis |

La unión es:

```text
CLIENTES_LIMA ∪ CLIENTES_CUSCO
```

### Resultado

| id_cliente | nombre |
| ---: | :--- |
| 101 | Ana |
| 103 | Marta |
| 102 | Luis |

### SQL

```sql
SELECT id_cliente, nombre
FROM clientes_lima
UNION
SELECT id_cliente, nombre
FROM clientes_cusco;
```

`UNION` elimina duplicados. `UNION ALL` conserva las repeticiones.

| SQL | Comportamiento |
| :--- | :--- |
| `UNION` | Combina y elimina duplicados |
| `UNION ALL` | Combina y conserva duplicados |

---

## 10. Operador de intersección `∩`

La intersección conserva las tuplas que aparecen en ambas relaciones.

```text
R ∩ S
```

## 10.1 Ejemplo

```text
CLIENTES_LIMA ∩ CLIENTES_ACTIVOS
```

Si tenemos:

| `CLIENTES_LIMA` |  | `CLIENTES_ACTIVOS` |  |
| --- | --- | --- | --- |
| id_cliente | nombre | id_cliente | nombre |
| 101 | Ana | 101 | Ana |
| 103 | Marta | 102 | Luis |
|  |  | 103 | Marta |

### Resultado

| id_cliente | nombre |
| ---: | :--- |
| 101 | Ana |
| 103 | Marta |

### SQL estándar

```sql
SELECT id_cliente, nombre
FROM clientes_lima
INTERSECT
SELECT id_cliente, nombre
FROM clientes_activos;
```

> **Compatibilidad.** `INTERSECT` está disponible en varios motores modernos, pero la compatibilidad depende de la versión. También puede simularse con `INNER JOIN` o `EXISTS`.

---

## 11. Operador de diferencia `−`

La diferencia devuelve las tuplas que están en la primera relación, pero no en la segunda.

```text
R − S
```

La operación no es conmutativa:

```text
R − S ≠ S − R
```

## 11.1 Clientes que no están en otra relación

```text
CLIENTES_LIMA − CLIENTES_ACTIVOS
```

Si `CLIENTES_LIMA` contiene a Ana y Marta, y `CLIENTES_ACTIVOS` contiene a Ana y Luis, el resultado es:

| id_cliente | nombre |
| ---: | :--- |
| 103 | Marta |

### SQL

```sql
SELECT id_cliente, nombre
FROM clientes_lima
EXCEPT
SELECT id_cliente, nombre
FROM clientes_activos;
```

En Oracle suele utilizarse `MINUS` en lugar de `EXCEPT`.

| Motor o estándar | Operador frecuente |
| :--- | :--- |
| SQL estándar y PostgreSQL | `EXCEPT` |
| Oracle | `MINUS` |
| MySQL | Puede requerir `NOT EXISTS`, `LEFT JOIN` o una subconsulta |

### Diferencia con `UNION`

| Operación | Conserva |
| :--- | :--- |
| `R ∪ S` | Lo que está en R o en S |
| `R ∩ S` | Lo que está en R y en S |
| `R − S` | Lo que está en R, pero no en S |

---

## 12. Producto cartesiano `×`

El producto cartesiano combina cada tupla de una relación con cada tupla de otra.

```text
R × S
```

Si `R` tiene 4 filas y `S` tiene 5 filas, el resultado tendrá:

```text
4 × 5 = 20 filas
```

## 12.1 Ejemplo

Supongamos:

```text
COLORES(color)
```

| color |
| :--- |
| Rojo |
| Azul |

```text
TALLAS(talla)
```

| talla |
| :--- |
| S |
| M |
| L |

Entonces:

```text
COLORES × TALLAS
```

### Resultado

| color | talla |
| :--- | :--- |
| Rojo | S |
| Rojo | M |
| Rojo | L |
| Azul | S |
| Azul | M |
| Azul | L |

### SQL

```sql
SELECT c.color, t.talla
FROM colores AS c
CROSS JOIN tallas AS t;
```

> **Advertencia.** Un producto cartesiano puede producir muchas filas. En consultas reales, un `JOIN` sin condición puede generar accidentalmente este resultado.

---

## 13. Join theta `⋈θ`

Un join theta combina relaciones usando una condición de comparación.

```text
R ⋈ condición S
```

También puede expresarse como un producto cartesiano seguido de una selección:

```text
R ⋈ condición S = σ condición (R × S)
```

## 13.1 Join entre clientes y pedidos

```text
CLIENTES ⋈ CLIENTES.id_cliente = PEDIDOS.id_cliente PEDIDOS
```

### Resultado

| id_cliente | nombre | ciudad | id_pedido | fecha | total |
| ---: | :--- | :--- | ---: | :--- | ---: |
| 101 | Ana | Lima | 1001 | 2026-10-01 | 149.90 |
| 102 | Luis | Cusco | 1002 | 2026-10-02 | 79.90 |
| 101 | Ana | Lima | 1003 | 2026-10-05 | 899.00 |
| 103 | Marta | Lima | 1004 | 2026-10-08 | 45.00 |

### SQL

```sql
SELECT
    c.nombre,
    c.ciudad,
    p.id_pedido,
    p.fecha,
    p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

---

## 14. Equijoin y join natural

## 14.1 Equijoin

Un equijoin utiliza igualdad en la condición:

```text
R ⋈ R.clave = S.clave S
```

En SQL:

```sql
SELECT *
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

## 14.2 Join natural

Un join natural combina automáticamente atributos con el mismo nombre.

```text
CLIENTES ⋈ PEDIDOS
```

Si ambas relaciones tienen `id_cliente`, ese atributo se utiliza para relacionarlas.

En SQL existe una forma equivalente:

```sql
SELECT *
FROM clientes
NATURAL JOIN pedidos;
```

Sin embargo, `NATURAL JOIN` puede ser riesgoso si en el futuro aparecen otras columnas con el mismo nombre.

| Tipo | Ventaja | Riesgo |
| :--- | :--- | :--- |
| `JOIN ... ON` | La relación es explícita | Requiere escribir la condición |
| `NATURAL JOIN` | Es más corto | Puede cambiar si cambian las columnas |

> **Recomendación.** En SQL profesional, suele ser más seguro utilizar `JOIN ... ON` de forma explícita.

---

## 15. Semijoin y antijoin

## 15.1 Semijoin

Un semijoin devuelve las tuplas de la primera relación que tienen al menos una coincidencia en la segunda, pero no incorpora necesariamente los atributos de la segunda relación.

```text
R ⋉ S
```

### Clientes que tienen pedidos

```text
CLIENTES ⋉ PEDIDOS
```

### Resultado

| id_cliente | nombre | ciudad |
| ---: | :--- | :--- |
| 101 | Ana | Lima |
| 102 | Luis | Cusco |
| 103 | Marta | Lima |

Carlos no aparece porque no tiene pedidos.

### SQL con `EXISTS`

```sql
SELECT c.id_cliente, c.nombre, c.ciudad
FROM clientes AS c
WHERE EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

## 15.2 Antijoin

Un antijoin devuelve las tuplas de la primera relación que no tienen coincidencia en la segunda.

```text
R ▷ S
```

### Clientes sin pedidos

```sql
SELECT c.id_cliente, c.nombre, c.ciudad
FROM clientes AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

### Resultado

| id_cliente | nombre | ciudad |
| ---: | :--- | :--- |
| 104 | Carlos | Arequipa |

---

## 16. División relacional `÷`

La división se utiliza para consultas del tipo:

- Clientes que compraron **todos** los productos de una lista.
- Estudiantes que aprobaron **todos** los cursos requeridos.
- Proveedores que suministran **todas** las piezas necesarias.

```text
R ÷ S
```

## 16.1 Ejemplo conceptual

Supongamos:

```text
COMPRAS(id_cliente, id_producto)
```

| id_cliente | id_producto |
| ---: | ---: |
| 101 | 1 |
| 101 | 2 |
| 101 | 3 |
| 102 | 1 |
| 102 | 2 |
| 103 | 1 |

Y una relación de productos requeridos:

```text
REQUERIDOS(id_producto)
```

| id_producto |
| ---: |
| 1 |
| 2 |

La división es:

```text
COMPRAS ÷ REQUERIDOS
```

### Resultado

| id_cliente |
| ---: |
| 101 |
| 102 |

Ana y Luis compraron todos los productos requeridos. Marta solo compró el producto 1.

## 16.2 Simulación con SQL

SQL no suele tener un operador `DIVISION` directo. Una forma de expresar “para todos” utiliza `NOT EXISTS` doble:

```sql
SELECT c.id_cliente
FROM clientes AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM requeridos AS r
    WHERE NOT EXISTS (
        SELECT 1
        FROM compras AS x
        WHERE x.id_cliente = c.id_cliente
          AND x.id_producto = r.id_producto
    )
);
```

### Lectura de la consulta

| Nivel | Pregunta |
| ---: | :--- |
| 1 | ¿Existe algún producto requerido? |
| 2 | ¿Para ese producto falta una compra del cliente? |
| Resultado | Si no falta ninguno, el cliente cumple “para todos” |

> **Qué debes recordar.** La división es la operación que resuelve consultas universales; en SQL suele expresarse mediante `NOT EXISTS` anidados.

---

## 17. Operadores extendidos

El álgebra relacional extendida incorpora operaciones útiles para expresar consultas más cercanas a SQL.

## 17.1 Agregación y agrupación

La agrupación se puede representar con una notación como:

```text
γ atributo; función(atributo) → alias (RELACION)
```

Ejemplo: contar pedidos por cliente.

```text
γ id_cliente; COUNT(id_pedido) → cantidad_pedidos (PEDIDOS)
```

### Resultado

| id_cliente | cantidad_pedidos |
| ---: | ---: |
| 101 | 2 |
| 102 | 1 |
| 103 | 1 |

### SQL

```sql
SELECT
    id_cliente,
    COUNT(*) AS cantidad_pedidos
FROM pedidos
GROUP BY id_cliente;
```

## 17.2 Suma de totales

```text
γ id_cliente; SUM(total) → total_comprado (PEDIDOS)
```

| id_cliente | total_comprado |
| ---: | ---: |
| 101 | 1048.90 |
| 102 | 79.90 |
| 103 | 45.00 |

```sql
SELECT
    id_cliente,
    SUM(total) AS total_comprado
FROM pedidos
GROUP BY id_cliente;
```

## 17.3 Ordenamiento

El ordenamiento no pertenece al álgebra relacional clásica porque las relaciones son conjuntos sin un orden garantizado. En álgebra extendida puede representarse como:

```text
τ precio DESC (PRODUCTOS)
```

Equivale aproximadamente a:

```sql
SELECT *
FROM productos
ORDER BY precio DESC;
```

## 17.4 División y funciones calculadas

SQL también incorpora expresiones que no están en el álgebra clásica, como:

- Cálculos aritméticos.
- `CASE`.
- Funciones de texto.
- Funciones de fecha.
- `LIMIT` y `OFFSET`.
- Valores `NULL`.

---

## 18. Árbol de consulta

Un árbol de consulta muestra el orden de las operaciones.

Consulta:

```sql
SELECT nombre, precio
FROM productos
WHERE categoria = 'periféricos'
  AND precio > 50;
```

Expresión relacional:

```text
π nombre, precio (
    σ categoria = 'periféricos' AND precio > 50 (
        PRODUCTOS
    )
)
```

Árbol conceptual:

```text
          π nombre, precio
                  |
 σ categoria = 'periféricos' AND precio > 50
                  |
              PRODUCTOS
```

### Lectura de abajo hacia arriba

| Nivel | Operación |
| ---: | :--- |
| 1 | Leer la relación `PRODUCTOS` |
| 2 | Filtrar categoría y precio |
| 3 | Proyectar nombre y precio |

---

## 19. Optimización de expresiones

Dos expresiones pueden producir el mismo resultado, pero una puede ser más eficiente.

## 19.1 Aplicar selecciones temprano

Expresión inicial:

```text
π nombre, total (
    σ ciudad = 'Lima' AND total > 100 (
        CLIENTES ⋈ PEDIDOS
    )
)
```

Una forma de razonarla es filtrar primero los clientes de Lima y los pedidos mayores de 100:

```text
π nombre, total (
    σ ciudad = 'Lima' (CLIENTES)
    ⋈
    σ total > 100 (PEDIDOS)
)
```

### Ventaja

| Sin filtrado temprano | Con filtrado temprano |
| :--- | :--- |
| Se combinan más filas | Se combinan menos filas |
| Mayor uso de memoria | Menor cantidad de datos intermedios |
| Más trabajo para el join | Join potencialmente más eficiente |

## 19.2 Proyectar solo lo necesario

Si una consulta solo necesita `nombre` y `total`, no es necesario conservar todas las columnas durante cada paso.

```text
π nombre, total (
    ...
)
```

> **Importante.** El optimizador del SGBD puede reordenar operaciones internamente. El árbol relacional ayuda a entender la lógica, pero el plan real se revisa con herramientas como `EXPLAIN`.

---

## 20. Propiedades de las operaciones

## 20.1 Conmutatividad

Una operación es conmutativa si el orden no cambia el resultado.

### Unión

```text
R ∪ S = S ∪ R
```

### Intersección

```text
R ∩ S = S ∩ R
```

### Producto cartesiano

```text
R × S = S × R
```

Conceptualmente puede cambiar el orden de las columnas, pero representa las mismas combinaciones.

### Diferencia

```text
R − S ≠ S − R
```

### Join

```text
R ⋈ S = S ⋈ R
```

Siempre que se ajuste la condición y el orden de atributos.

## 20.2 Asociatividad

La unión y la intersección son asociativas:

```text
(R ∪ S) ∪ T = R ∪ (S ∪ T)
```

```text
(R ∩ S) ∩ T = R ∩ (S ∩ T)
```

Los joins también pueden reagruparse en muchas situaciones:

```text
(R ⋈ S) ⋈ T = R ⋈ (S ⋈ T)
```

Estas propiedades ayudan a los optimizadores de consultas.

---

## 21. Diferencia entre álgebra relacional y cálculo relacional

| Aspecto | Álgebra relacional | Cálculo relacional |
| :--- | :--- | :--- |
| Enfoque | Procedimental | Declarativo |
| Indica | Cómo obtener el resultado | Qué resultado se desea |
| Usa | Operadores sobre relaciones | Predicados y variables |
| Relación con SQL | Ayuda a entender operaciones | Ayuda a entender la naturaleza declarativa |

SQL es principalmente declarativo: se indica qué datos se necesitan y el motor decide cómo obtenerlos.

El álgebra relacional, en cambio, permite describir una secuencia de operaciones sobre las relaciones.

---

## 22. Traducción de SQL a álgebra relacional

## Ejemplo 1: filtro simple

SQL:

```sql
SELECT *
FROM productos
WHERE stock > 0;
```

Álgebra:

```text
σ stock > 0 (PRODUCTOS)
```

## Ejemplo 2: selección de columnas

SQL:

```sql
SELECT nombre, categoria
FROM productos;
```

Álgebra:

```text
π nombre, categoria (PRODUCTOS)
```

## Ejemplo 3: filtro y columnas

SQL:

```sql
SELECT nombre, precio
FROM productos
WHERE precio BETWEEN 50 AND 250;
```

Álgebra:

```text
π nombre, precio (
    σ precio >= 50 AND precio <= 250 (PRODUCTOS)
)
```

## Ejemplo 4: join

SQL:

```sql
SELECT c.nombre, p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente
WHERE p.total > 100;
```

Álgebra:

```text
π nombre, total (
    σ total > 100 (
        CLIENTES ⋈ CLIENTES.id_cliente = PEDIDOS.id_cliente PEDIDOS
    )
)
```

## Ejemplo 5: agrupación

SQL:

```sql
SELECT id_cliente, SUM(total) AS total_comprado
FROM pedidos
GROUP BY id_cliente;
```

Álgebra extendida:

```text
γ id_cliente; SUM(total) → total_comprado (PEDIDOS)
```

---

## 23. Errores comunes

| Error | Qué ocurre | Cómo evitarlo |
| :--- | :--- | :--- |
| Confundir selección con proyección | Se filtran columnas cuando se querían filtrar filas | `σ` trabaja con filas y `π` con atributos |
| Olvidar la compatibilidad en `UNION` | Las relaciones no pueden combinarse correctamente | Revisar cantidad y dominios de columnas |
| Invertir una diferencia | `R − S` no equivale a `S − R` | Recordar que la operación depende del orden |
| Hacer un producto cartesiano accidental | Se generan muchas combinaciones incorrectas | Usar un `JOIN` con condición |
| Confundir join con producto cartesiano | Falta la condición de relación | Escribir la condición `ON` |
| Suponer que SQL elimina duplicados | SQL trabaja normalmente con bolsas de filas | Usar `DISTINCT` cuando corresponda |
| Usar `= NULL` | Las comparaciones con `NULL` no funcionan así | Usar `IS NULL` |
| Pensar que las relaciones tienen orden | El álgebra clásica no garantiza orden | Usar `ORDER BY` en SQL |
| Aplicar división a cualquier consulta | La división requiere una condición universal | Buscar expresiones como “para todos” |
| Mezclar atributos ambiguos | No se sabe de qué relación proviene una columna | Usar nombres calificados o alias |
| Proyectar antes de conservar claves necesarias | Luego no se puede realizar el join | Mantener los atributos requeridos |
| Suponer que la expresión escrita es el plan real | El optimizador puede reordenar operaciones | Revisar `EXPLAIN` |

### Ejemplo incorrecto y corregido

```sql
-- Riesgo: producto cartesiano
SELECT c.nombre, p.total
FROM clientes AS c, pedidos AS p;
```

```sql
-- Correcto: join con relación explícita
SELECT c.nombre, p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

---

## 24. Práctica guiada

## Pregunta 1: selección

Obtén los productos cuyo stock sea mayor que cero.

<details>
<summary>Ver respuesta</summary>

```text
σ stock > 0 (PRODUCTOS)
```

```sql
SELECT *
FROM productos
WHERE stock > 0;
```

</details>

## Pregunta 2: proyección

Obtén únicamente el nombre y la categoría de los productos.

<details>
<summary>Ver respuesta</summary>

```text
π nombre, categoria (PRODUCTOS)
```

```sql
SELECT nombre, categoria
FROM productos;
```

</details>

## Pregunta 3: selección y proyección

Obtén el nombre y el precio de los productos cuyo precio sea mayor que 100.

<details>
<summary>Ver respuesta</summary>

```text
π nombre, precio (
    σ precio > 100 (PRODUCTOS)
)
```

```sql
SELECT nombre, precio
FROM productos
WHERE precio > 100;
```

</details>

## Pregunta 4: join

Obtén el nombre del cliente y el total de cada pedido.

<details>
<summary>Ver respuesta</summary>

```text
π nombre, total (
    CLIENTES ⋈ CLIENTES.id_cliente = PEDIDOS.id_cliente PEDIDOS
)
```

```sql
SELECT c.nombre, p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

</details>

## Pregunta 5: diferencia

Obtén los clientes que no tienen pedidos.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT c.id_cliente, c.nombre
FROM clientes AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

</details>

---

## 25. Mini desafío final

Usa las relaciones `PRODUCTOS`, `CLIENTES` y `PEDIDOS` para resolver:

1. Obtener los productos de la categoría `periféricos` con stock disponible.
2. Mostrar solo el nombre y el precio de los productos que cuestan menos de 200.
3. Combinar clientes y pedidos mediante un join.
4. Obtener los clientes que no tienen pedidos.
5. Calcular la cantidad de pedidos por cliente.
6. Explicar qué operación representa una consulta de clientes que compraron todos los productos requeridos.

## Una solución posible

```text
-- 1. Selección por categoría y stock
σ categoria = 'periféricos' AND stock > 0 (PRODUCTOS)
```

```sql
SELECT *
FROM productos
WHERE categoria = 'periféricos'
  AND stock > 0;
```

```text
-- 2. Selección seguida de proyección
π nombre, precio (
    σ precio < 200 (PRODUCTOS)
)
```

```sql
SELECT nombre, precio
FROM productos
WHERE precio < 200;
```

```sql
-- 3. Join entre clientes y pedidos
SELECT c.nombre, p.id_pedido, p.total
FROM clientes AS c
JOIN pedidos AS p
    ON c.id_cliente = p.id_cliente;
```

```sql
-- 4. Antijoin: clientes sin pedidos
SELECT c.id_cliente, c.nombre
FROM clientes AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM pedidos AS p
    WHERE p.id_cliente = c.id_cliente
);
```

```sql
-- 5. Agrupación
SELECT
    id_cliente,
    COUNT(*) AS cantidad_pedidos
FROM pedidos
GROUP BY id_cliente;
```

```text
-- 6. División relacional
COMPRAS ÷ REQUERIDOS
```

La división representa clientes que cumplen una condición para todos los elementos de una relación requerida.

### Tabla de correspondencia final

| Problema | Operación relacional principal |
| :--- | :--- |
| Filtrar productos por precio | Selección `σ` |
| Elegir nombre y precio | Proyección `π` |
| Cambiar el nombre de una relación | Renombramiento `ρ` |
| Combinar clientes y pedidos | Join `⋈` |
| Clientes sin pedidos | Antijoin o diferencia |
| Cantidad de pedidos por cliente | Agrupación `γ` |
| Clientes que cumplen todos los requisitos | División `÷` |

---

## 26. Resumen final

- El álgebra relacional es un lenguaje formal para operar sobre relaciones.
- Una relación puede representarse como una tabla.
- La selección `σ` filtra filas según una condición.
- La proyección `π` elige columnas y, en el modelo clásico, elimina duplicados.
- El renombramiento `ρ` cambia nombres de relaciones o atributos.
- La unión `∪` combina relaciones compatibles.
- La intersección `∩` conserva las tuplas comunes.
- La diferencia `−` conserva las tuplas de la primera relación que no están en la segunda.
- El producto cartesiano `×` combina cada fila de una relación con cada fila de otra.
- El join `⋈` combina filas relacionadas mediante una condición.
- El semijoin devuelve filas con coincidencia sin incorporar necesariamente las columnas de la otra relación.
- El antijoin devuelve filas sin coincidencia.
- La división `÷` resuelve consultas del tipo “para todos”.
- El álgebra extendida incluye agrupación, agregación y ordenamiento.
- SQL se puede interpretar mediante operaciones relacionales, aunque no es idéntico al álgebra clásica.
- Aplicar filtros y proyecciones temprano puede reducir el costo de una consulta.
- Para analizar el plan real de ejecución se utiliza `EXPLAIN`.

> **Qué debes recordar.** Una consulta SQL compleja puede entenderse como una combinación de operaciones: seleccionar filas, proyectar columnas, combinar relaciones y agrupar resultados.

---

## Relacionado con otros temas

- [`SELECT` y `FROM`](../02-consultas-basicas/select-from.md): equivalen principalmente a proyección y origen de la relación.
- [WHERE y operadores](../02-consultas-basicas/where-operadores.md): se relacionan con la selección `σ`.
- [ORDER BY y LIMIT](../02-consultas-basicas/order-by-limit.md): corresponden a operaciones extendidas de orden y límite.
- [Funciones SQL](../02-consultas-basicas/funciones-sql.md): permiten crear expresiones sobre atributos.
- [Joins](joins.md): desarrolla las combinaciones entre tablas.
- [Subconsultas](subqueries.md): permiten expresar semijoins, antijoins y divisiones.
- [Agregaciones y `GROUP BY`](agregaciones-group-by.md): desarrolla la agrupación y las funciones agregadas.
- [Transformación ER a relacional](../04-diseno-modelado/transformacion-er-a-relacional.md): convierte modelos conceptuales en relaciones.
- [Índices y rendimiento](../06-indices-performance/indices-basicos.md): ayuda a optimizar las operaciones implementadas por el SGBD.
