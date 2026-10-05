# DDL, DML y DQL en SQL

SQL puede dividirse en varios sublenguajes según el tipo de operación que se realiza. Algunos comandos crean o modifican la estructura de la base de datos, otros cambian los registros y otros únicamente consultan información.

Esta guía explica tres grupos fundamentales:

- **DDL:** define y modifica estructuras.
- **DML:** inserta, actualiza y elimina datos.
- **DQL:** consulta datos sin modificarlos.

> **Idea central.** DDL trabaja sobre el molde de la base de datos; DML trabaja sobre el contenido; DQL permite leer y analizar ese contenido.

---

## 1. Diferencia general entre DDL, DML y DQL

| Sublenguaje | Nombre completo | ¿Qué modifica? | Comandos principales | Ejemplo de propósito |
| :--- | :--- | :--- | :--- | :--- |
| **DDL** | Data Definition Language | La estructura | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Crear una tabla o agregar una columna |
| **DML** | Data Manipulation Language | Los registros | `INSERT`, `UPDATE`, `DELETE` | Registrar o cambiar pedidos |
| **DQL** | Data Query Language | No modifica; consulta | `SELECT` | Buscar pedidos pendientes |

### Pregunta rápida

| Situación | Grupo correcto | Comando probable |
| :--- | :--- | :--- |
| Crear una tabla nueva | DDL | `CREATE TABLE` |
| Agregar una columna | DDL | `ALTER TABLE` |
| Registrar un cliente | DML | `INSERT` |
| Cambiar el estado de un pedido | DML | `UPDATE` |
| Borrar un registro específico | DML | `DELETE` |
| Consultar todos los clientes | DQL | `SELECT` |

> **Para recordar.** Si cambia la definición de la tabla, es DDL. Si cambia una fila, es DML. Si solo se leen datos, es DQL.

---

## 2. DDL: definir la estructura

El **Data Definition Language** se utiliza para crear, modificar o eliminar objetos de la base de datos, como tablas, columnas, vistas e índices.

Los comandos DDL más frecuentes son:

| Comando | Acción | ¿Qué afecta? |
| :--- | :--- | :--- |
| `CREATE` | Crea un objeto | Una tabla, base de datos, vista o índice |
| `ALTER` | Modifica un objeto existente | Columnas, tipos o restricciones |
| `DROP` | Elimina un objeto completo | La estructura y sus datos |
| `TRUNCATE` | Vacía una tabla completa | Todos sus registros, pero conserva la tabla |

---

## 3. `CREATE TABLE`: crear una tabla

La sintaxis general es:

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_dato,
    columna2 tipo_dato,
    columna3 tipo_dato
);
```

### Ejemplo

```sql
CREATE TABLE marcas (
    id INT PRIMARY KEY,
    nombre VARCHAR(45) NOT NULL
);
```

Esta instrucción crea una tabla llamada `marcas`.

| Columna | Tipo o restricción | Propósito |
| :--- | :--- | :--- |
| `id` | `INT PRIMARY KEY` | Identifica cada marca sin repetirla |
| `nombre` | `VARCHAR(45) NOT NULL` | Guarda el nombre y lo vuelve obligatorio |

### Estructura resultante

| Tabla creada | Columnas disponibles | Registros iniciales |
| :--- | :--- | :---: |
| `marcas` | `id`, `nombre` | 0 |

Crear la tabla no inserta datos automáticamente. Solo prepara la estructura donde después se almacenarán registros.

---

## 4. Crear tablas relacionadas

DDL también permite definir relaciones entre tablas mediante llaves foráneas.

```sql
CREATE TABLE marcas (
    id INT PRIMARY KEY,
    nombre VARCHAR(45) NOT NULL
);

CREATE TABLE productos (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio DECIMAL(10,2) NOT NULL,
    marca_id INT,
    FOREIGN KEY (marca_id) REFERENCES marcas(id)
);
```

### Relación creada

| Tabla | Columna | Relación |
| :--- | :--- | :--- |
| `marcas` | `id` | Llave primaria |
| `productos` | `marca_id` | Llave foránea que apunta a `marcas.id` |

Por lo tanto, un producto puede relacionarse con una marca existente, pero no debería usar un `marca_id` que no exista en la tabla `marcas`.

> **Secuencia recomendada.** Cuando existen llaves foráneas, normalmente se crea primero la tabla padre y después la tabla hija.

---

## 5. `ALTER TABLE`: modificar una estructura

`ALTER TABLE` se utiliza cuando la tabla ya existe y se necesita cambiar su estructura.

### Agregar una columna

```sql
ALTER TABLE marcas
ADD comentarios VARCHAR(100);
```

### Antes y después

| Momento | Columnas de `marcas` |
| :--- | :--- |
| Antes | `id`, `nombre` |
| Después | `id`, `nombre`, `comentarios` |

La operación no agrega comentarios a las filas existentes. Solo crea una nueva columna para que pueda recibir datos.

### Cambiar el tipo de una columna

La sintaxis depende del motor.

**MySQL:**

```sql
ALTER TABLE marcas
MODIFY COLUMN nombre VARCHAR(100) NOT NULL;
```

**PostgreSQL:**

```sql
ALTER TABLE marcas
ALTER COLUMN nombre TYPE VARCHAR(100);
```

Antes de cambiar el tipo, hay que verificar que los valores actuales sean compatibles.

### Agregar una restricción

```sql
ALTER TABLE productos
ADD CONSTRAINT fk_producto_marca
FOREIGN KEY (marca_id) REFERENCES marcas(id);
```

| Situación | Resultado |
| :--- | :--- |
| La columna `marca_id` contiene valores válidos | La restricción puede agregarse |
| Existe un `marca_id` inexistente | El motor puede rechazar la operación |

---

## 6. `DROP TABLE`: eliminar una tabla

`DROP TABLE` elimina la tabla completa, incluyendo su estructura y sus datos.

```sql
DROP TABLE tabla_temporal;
```

### Comparación antes y después

| Elemento | Antes de `DROP TABLE` | Después de `DROP TABLE` |
| :--- | :--- | :--- |
| Tabla | Existe | Ya no existe |
| Columnas | Disponibles | Eliminadas |
| Registros | Se pueden consultar | Eliminados |
| Consultas sobre la tabla | Funcionan | Generan error |

> **Cuidado.** `DROP TABLE` es una operación destructiva. Antes de ejecutarlo, verifica el nombre de la tabla y confirma que no sea necesaria.

---

## 7. `TRUNCATE`: vaciar una tabla

`TRUNCATE TABLE` elimina todos los registros, pero conserva la estructura de la tabla.

```sql
TRUNCATE TABLE productos;
```

### `TRUNCATE` frente a `DELETE`

| Característica | `TRUNCATE` | `DELETE` |
| :--- | :--- | :--- |
| Grupo | DDL | DML |
| Elimina registros | Todos | Algunos o todos, según el `WHERE` |
| Permite `WHERE` | No | Sí |
| Conserva la tabla | Sí | Sí |
| Puede reiniciar autoincremento | Frecuentemente sí, según el motor | Generalmente no |
| Comportamiento transaccional | Depende del motor; suele tener restricciones | Normalmente participa en transacciones |

```sql
-- Vacía todos los productos
TRUNCATE TABLE productos;

-- Elimina solo productos de una marca
DELETE FROM productos
WHERE marca_id = 2;
```

> **No los confundas.** Ambos pueden dejar una tabla sin registros, pero `TRUNCATE` no permite elegir filas con `WHERE`, mientras que `DELETE` sí.

---

## 8. DML: manipular los registros

El **Data Manipulation Language** trabaja con los datos que ya existen dentro de las tablas.

| Comando | Acción | Ejemplo |
| :--- | :--- | :--- |
| `INSERT` | Agrega filas | Registrar un nuevo producto |
| `UPDATE` | Modifica filas | Cambiar un precio |
| `DELETE` | Elimina filas | Quitar un producto específico |

---

## 9. `INSERT`: insertar datos

La sintaxis recomendada es:

```sql
INSERT INTO nombre_tabla (columna1, columna2, columna3)
VALUES (valor1, valor2, valor3);
```

### Insertar una fila

```sql
INSERT INTO marcas (id, nombre)
VALUES (1, 'De la Rosa');
```

### Insertar varias filas

```sql
INSERT INTO marcas (id, nombre)
VALUES
    (2, 'Ricolino'),
    (3, 'Adams'),
    (4, 'Sonrics');
```

### Resultado simulado

| id | nombre |
| ---: | :--- |
| 1 | De la Rosa |
| 2 | Ricolino |
| 3 | Adams |
| 4 | Sonrics |

Es recomendable indicar siempre los nombres de las columnas. Así el código es más claro y no depende del orden interno de la tabla.

### Insertar un producto relacionado

```sql
INSERT INTO productos (id, nombre, precio, marca_id)
VALUES (1001, 'Paleta de caramelo', 2.50, 4);
```

| id | nombre | precio | marca_id | Resultado |
| ---: | :--- | ---: | ---: | :--- |
| 1001 | Paleta de caramelo | 2.50 | 4 | Aceptado si existe la marca 4 |
| 1002 | Galleta de chocolate | 8.00 | 99 | Rechazado si la marca 99 no existe |

---

## 10. `UPDATE`: actualizar datos

`UPDATE` modifica una o varias filas existentes.

### Sintaxis

```sql
UPDATE nombre_tabla
SET columna = nuevo_valor
WHERE condicion;
```

### Ejemplo

```sql
UPDATE productos
SET precio = 3.00
WHERE id = 1001;
```

### Antes y después

| id | nombre | Precio antes | Precio después |
| ---: | :--- | ---: | ---: |
| 1001 | Paleta de caramelo | 2.50 | 3.00 |
| 1002 | Galleta de chocolate | 8.00 | 8.00 |

Solo se modifica el producto cuyo `id` es `1001`.

### Actualizar con una operación

```sql
UPDATE productos
SET precio = precio * 1.10
WHERE marca_id = 4;
```

Esta operación aumenta en 10 % el precio de los productos de la marca 4.

| Producto | Precio antes | Operación | Precio después |
| :--- | ---: | :---: | ---: |
| Paleta de caramelo | 3.00 | `3.00 * 1.10` | 3.30 |
| Otro producto de marca 4 | 10.00 | `10.00 * 1.10` | 11.00 |
| Producto de marca 2 | 8.00 | No coincide | 8.00 |

> **Revisa antes de ejecutar.** Antes de hacer un `UPDATE`, puedes ejecutar un `SELECT` con la misma condición para comprobar qué filas serán afectadas.

```sql
SELECT *
FROM productos
WHERE marca_id = 4;
```

---

## 11. `DELETE`: eliminar datos

`DELETE` borra filas de una tabla.

### Sintaxis

```sql
DELETE FROM nombre_tabla
WHERE condicion;
```

### Ejemplo

```sql
DELETE FROM marcas
WHERE id = 2;
```

### Resultado simulado

| id | nombre | Estado después del `DELETE` |
| ---: | :--- | :--- |
| 1 | De la Rosa | Permanece |
| 2 | Ricolino | Eliminado |
| 3 | Adams | Permanece |
| 4 | Sonrics | Permanece |

### Eliminar según una condición

```sql
DELETE FROM productos
WHERE precio < 5.00;
```

Esta instrucción elimina todos los productos cuyo precio sea menor a 5.00.

> **Advertencia.** Un `DELETE` sin `WHERE` puede eliminar todos los registros de la tabla.

```sql
-- Muy peligroso: elimina todas las filas
DELETE FROM productos;
```

### Verificar antes de borrar

```sql
SELECT *
FROM productos
WHERE precio < 5.00;

-- Solo después de revisar el resultado:
DELETE FROM productos
WHERE precio < 5.00;
```

---

## 12. DQL: consultar información

El **Data Query Language** se utiliza principalmente para leer datos. Su comando central es `SELECT`.

### Consulta básica

```sql
SELECT nombre, precio
FROM productos;
```

### Consultar todas las columnas

```sql
SELECT *
FROM productos;
```

El asterisco funciona para practicar, pero en consultas reales suele ser mejor indicar las columnas necesarias.

### Filtrar con `WHERE`

```sql
SELECT nombre, precio
FROM productos
WHERE precio > 5.00;
```

### Resultado simulado

| nombre | precio |
| :--- | ---: |
| Galleta de chocolate | 8.00 |
| Audífonos bluetooth | 89.90 |

El producto con precio menor o igual a 5 no aparece porque no cumple la condición.

---

## 13. Ordenar y limitar resultados

### `ORDER BY`

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio DESC;
```

| Posición | nombre | precio |
| :---: | :--- | ---: |
| 1 | Audífonos bluetooth | 89.90 |
| 2 | Galleta de chocolate | 8.00 |
| 3 | Paleta de caramelo | 3.00 |

`ASC` ordena de menor a mayor y `DESC` de mayor a menor.

### `LIMIT`

```sql
SELECT nombre, precio
FROM productos
ORDER BY precio DESC
LIMIT 2;
```

| nombre | precio |
| :--- | ---: |
| Audífonos bluetooth | 89.90 |
| Galleta de chocolate | 8.00 |

---

## 14. Flujo completo: DDL, DML y DQL

Un flujo frecuente consiste en crear una tabla, insertar datos y consultarlos.

```sql
-- 1. DDL: definir la estructura
CREATE TABLE pedidos (
    id INT PRIMARY KEY,
    cliente_id INT NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    estado VARCHAR(20) DEFAULT 'pendiente'
);

-- 2. DML: insertar registros
INSERT INTO pedidos (id, cliente_id, total)
VALUES
    (1, 1, 45.90),
    (2, 3, 12.00),
    (3, 2, 8.50);

-- 3. DML: actualizar un registro
UPDATE pedidos
SET estado = 'entregado'
WHERE id = 3;

-- 4. DQL: consultar pedidos pendientes
SELECT id, cliente_id, total, estado
FROM pedidos
WHERE estado = 'pendiente';
```

### Estado de la tabla después de las operaciones

| id | cliente_id | total | estado |
| ---: | ---: | ---: | :--- |
| 1 | 1 | 45.90 | pendiente |
| 2 | 3 | 12.00 | pendiente |
| 3 | 2 | 8.50 | entregado |

### Resultado de la consulta DQL

| id | cliente_id | total | estado |
| ---: | ---: | ---: | :--- |
| 1 | 1 | 45.90 | pendiente |
| 2 | 3 | 12.00 | pendiente |

El pedido 3 sigue existiendo, pero no aparece porque su estado es `entregado`.

---

## 15. Operaciones combinadas

### Actualizar varios registros con una condición

Queremos aumentar en 1.00 el total de todos los pedidos pendientes:

```sql
UPDATE pedidos
SET total = total + 1.00
WHERE estado = 'pendiente';
```

| id | Estado | Total antes | Total después |
| ---: | :--- | ---: | ---: |
| 1 | pendiente | 45.90 | 46.90 |
| 2 | pendiente | 12.00 | 13.00 |
| 3 | entregado | 8.50 | 8.50 |

### Borrar usando una subconsulta

Supongamos que queremos eliminar los pedidos del cliente cuyo nombre es `Ana`, pero no conocemos su identificador.

```sql
DELETE FROM pedidos
WHERE cliente_id = (
    SELECT id
    FROM clientes
    WHERE nombre = 'Ana'
);
```

El `SELECT` interno busca el `id` de Ana y el `DELETE` externo elimina los pedidos relacionados.

| Paso | Operación | Resultado |
| ---: | :--- | :--- |
| 1 | Buscar `id` de Ana | Se obtiene, por ejemplo, `5` |
| 2 | Buscar pedidos con `cliente_id = 5` | Se encuentran sus pedidos |
| 3 | Ejecutar `DELETE` | Se eliminan esos pedidos |

> **Secuencia segura.** Antes de ejecutar un `DELETE` con subconsulta, prueba primero el `SELECT` equivalente para revisar los registros afectados.

---

## 16. Transacciones y posibilidad de revertir

El comportamiento transaccional depende del motor y del tipo de comando, pero existe una diferencia práctica importante:

| Operación | Grupo | ¿Puede usar `WHERE`? | Comportamiento habitual ante `ROLLBACK` |
| :--- | :--- | :---: | :--- |
| `CREATE TABLE` | DDL | No | Puede hacer commit automático según el motor |
| `ALTER TABLE` | DDL | No | Puede hacer commit automático según el motor |
| `DROP TABLE` | DDL | No | Generalmente no se revierte como un DML común |
| `TRUNCATE` | DDL | No | Tiene restricciones transaccionales según el motor |
| `INSERT` | DML | No | Normalmente puede revertirse antes de `COMMIT` |
| `UPDATE` | DML | Sí, indirectamente | Normalmente puede revertirse antes de `COMMIT` |
| `DELETE` | DML | Sí | Normalmente puede revertirse antes de `COMMIT` |
| `SELECT` | DQL | — | No modifica datos |

### Ejemplo con DML y transacción

```sql
START TRANSACTION;

UPDATE pedidos
SET estado = 'cancelado'
WHERE id = 2;

-- Si el cambio fue incorrecto:
ROLLBACK;

-- Si el cambio fue revisado y es correcto:
-- COMMIT;
```

> **Compatibilidad entre motores.** No asumas que todos los SGBD tratan DDL exactamente igual. Revisa la documentación del motor que estés utilizando.

---

## 17. Errores comunes

| Error | Qué puede ocurrir | Cómo evitarlo |
| :--- | :--- | :--- |
| Omitir `WHERE` en `UPDATE` | Se modifican todas las filas | Probar primero la condición con `SELECT` |
| Omitir `WHERE` en `DELETE` | Se eliminan todos los registros | Revisar la consulta antes de ejecutarla |
| Confundir `DELETE` y `TRUNCATE` | Se usa una operación demasiado amplia | Recordar que `TRUNCATE` no permite `WHERE` |
| Usar `DROP TABLE` sin verificar | Se elimina estructura y datos | Confirmar el nombre y hacer respaldo |
| Insertar una llave foránea inexistente | La inserción es rechazada | Crear primero el registro padre |
| Actualizar sin una condición suficientemente específica | Se modifican más filas de las esperadas | Filtrar por una clave primaria |
| Esperar que DDL se revierta siempre | El `ROLLBACK` puede no restaurar la estructura | Revisar el comportamiento del motor |

### Forma insegura y forma recomendada

```sql
-- Riesgoso
UPDATE productos
SET precio = 0;

-- Recomendado: primero comprobar qué filas coinciden
SELECT id, nombre, precio
FROM productos
WHERE marca_id = 4;

UPDATE productos
SET precio = 0
WHERE marca_id = 4;
```

---

## 18. Práctica guiada

### Pregunta 1: clasificar comandos

Clasifica cada instrucción como DDL, DML o DQL:

```sql
CREATE TABLE alumnos (
    id INT PRIMARY KEY,
    nombre VARCHAR(100)
);

INSERT INTO alumnos (id, nombre)
VALUES (1, 'María');

SELECT * FROM alumnos;
```

<details>
<summary>Ver respuesta</summary>

| Instrucción | Grupo | Motivo |
| :--- | :--- | :--- |
| `CREATE TABLE` | DDL | Crea una estructura |
| `INSERT` | DML | Agrega un registro |
| `SELECT` | DQL | Consulta información |

</details>

### Pregunta 2: actualizar un pedido

Marca como `cancelado` el pedido con `id = 2`.

<details>
<summary>Ver respuesta</summary>

```sql
UPDATE pedidos
SET estado = 'cancelado'
WHERE id = 2;
```

Es DML porque cambia el contenido de una fila, no la estructura de la tabla.

</details>

### Pregunta 3: diferencia entre `DELETE` y `TRUNCATE`

Quieres eliminar solo los productos de la categoría `ofertas`.

¿Qué comando usarías?

<details>
<summary>Ver respuesta</summary>

```sql
DELETE FROM productos
WHERE categoria = 'ofertas';
```

`TRUNCATE` no sirve para este caso porque elimina todos los registros y no permite usar `WHERE`.

</details>

### Pregunta 4: verificar antes de borrar

Quieres borrar los pedidos del cliente `10`. Escribe primero una consulta segura para revisar los registros.

<details>
<summary>Ver respuesta</summary>

```sql
SELECT *
FROM pedidos
WHERE cliente_id = 10;
```

Después de revisar el resultado, podrías ejecutar:

```sql
DELETE FROM pedidos
WHERE cliente_id = 10;
```

</details>

---

## 19. Mini desafío final

Crea y utiliza una tabla `tareas` con estas condiciones:

1. Debe tener un `id` entero como llave primaria.
2. Debe guardar un título obligatorio.
3. Debe tener un estado con valor predeterminado `pendiente`.
4. Debes insertar tres tareas.
5. Debes marcar una tarea como `completada`.
6. Debes consultar únicamente las tareas pendientes.

### Una solución posible

```sql
-- DDL
CREATE TABLE tareas (
    id INT PRIMARY KEY,
    titulo VARCHAR(100) NOT NULL,
    estado VARCHAR(20) DEFAULT 'pendiente'
);

-- DML: insertar
INSERT INTO tareas (id, titulo)
VALUES
    (1, 'Repasar tipos de datos'),
    (2, 'Practicar SELECT'),
    (3, 'Leer sobre constraints');

-- DML: actualizar
UPDATE tareas
SET estado = 'completada'
WHERE id = 1;

-- DQL: consultar
SELECT id, titulo, estado
FROM tareas
WHERE estado = 'pendiente';
```

### Resultado esperado

| id | titulo | estado |
| ---: | :--- | :--- |
| 2 | Practicar SELECT | pendiente |
| 3 | Leer sobre constraints | pendiente |

---

## 20. Resumen final

- DDL define y modifica la estructura de la base de datos.
- `CREATE` crea tablas y otros objetos.
- `ALTER` modifica objetos existentes.
- `DROP` elimina una estructura completa.
- `TRUNCATE` vacía una tabla, pero conserva su estructura.
- DML trabaja con los registros.
- `INSERT` agrega filas.
- `UPDATE` modifica filas.
- `DELETE` elimina filas según una condición.
- DQL consulta información mediante `SELECT`.
- `WHERE` ayuda a limitar las filas afectadas o consultadas.
- Antes de ejecutar un `UPDATE` o `DELETE`, conviene comprobar la condición con un `SELECT`.
- El comportamiento de DDL y transacciones puede variar entre motores.

> **Qué debes recordar.** Una base de datos se construye con DDL, se llena y modifica con DML, y se revisa mediante DQL.

---

## Relacionado con otros temas

- [Tipos de datos](tipos-de-datos.md): se utilizan al definir las columnas mediante DDL.
- [Constraints](constraints.md): se declaran principalmente con `CREATE TABLE` y `ALTER TABLE`.
- [Joins](../02-consultas/joins.md): amplían las consultas DQL al combinar tablas.
- [Transacciones ACID](../04-transacciones/acid.md): ayudan a comprender `COMMIT`, `ROLLBACK` y la diferencia entre DDL y DML.
