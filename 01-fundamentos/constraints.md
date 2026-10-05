# Guía de Teoría y Repaso: Constraints o Restricciones en SQL

Esta guía reúne la teoría, los ejemplos y las prácticas necesarias para repasar cómo utilizar **constraints** o **restricciones** para proteger la integridad de los datos en una base de datos.

Los constraints son reglas que el motor de base de datos aplica automáticamente. Su objetivo es impedir que se guarden datos inválidos, incompletos o inconsistentes.

<blockquote style="border-left: 5px solid #2563eb; padding-left: 12px;">
<strong>Concepto clave.</strong> Es mejor que la base de datos rechace un dato incorrecto a que la aplicación dependa únicamente de que alguien lo valide correctamente.
</blockquote>

Para los ejemplos utilizaremos dos tablas relacionadas:

- `clientes`: almacena la información de los clientes.
- `pedidos`: almacena los pedidos realizados por los clientes.

---

## 1. ¿Qué es un constraint?

Un **constraint** es una regla que se agrega a una columna o tabla para controlar qué datos pueden almacenarse.

Por ejemplo:

- Un cliente debe tener un identificador único.
- Un correo electrónico no debería repetirse.
- Un pedido debe pertenecer a un cliente existente.
- La edad de un cliente debe cumplir una condición válida.
- Si no se indica un país, se puede asignar uno automáticamente.

Los constraints forman parte del estándar SQL. Sin embargo, algunos detalles de sintaxis o comportamiento pueden variar entre MySQL, PostgreSQL, SQL Server y Oracle.

---

## 2. Principales tipos de constraints

| Constraint | ¿Qué hace? | ¿Permite `NULL`? | ¿Permite duplicados? |
| :--- | :--- | :--- | :--- |
| `PRIMARY KEY` | Identifica de forma única cada fila | No | No |
| `FOREIGN KEY` | Obliga a que el valor exista en otra tabla | Sí, si la columna lo permite | Sí |
| `NOT NULL` | Impide que una columna quede vacía | No | Sí |
| `UNIQUE` | Impide repetir valores | Sí, dependiendo del motor | No |
| `DEFAULT` | Coloca un valor automático si no se especifica otro | Depende de la columna | Depende de la columna |
| `CHECK` | Valida que se cumpla una condición | Depende de la columna | Depende de la condición |

### Diferencia rápida entre `PRIMARY KEY` y `UNIQUE`

- `PRIMARY KEY`: identifica cada fila y no permite `NULL` ni duplicados.
- `UNIQUE`: evita valores repetidos, pero normalmente puede permitir uno o varios valores `NULL`, según el motor de base de datos.

<blockquote style="border-left: 5px solid #6b7280; padding-left: 12px;">
<strong>Compatibilidad entre motores.</strong> Desde MySQL 8.0.16, las restricciones `CHECK` se aplican realmente. En versiones anteriores, MySQL podía aceptar la sintaxis, pero ignorar la condición.
</blockquote>

---

## 3. Creación de las tablas

Primero crearemos la tabla `clientes`:

```sql
CREATE TABLE clientes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) NOT NULL UNIQUE,
    edad TINYINT CHECK (edad >= 18),
    pais VARCHAR(50) DEFAULT 'Perú'
);
```

### ¿Qué restricción tiene cada columna?

| Columna | Restricción | Propósito |
| :--- | :--- | :--- |
| `id` | `PRIMARY KEY` | Identificar a cada cliente sin repetirlo |
| `email` | `NOT NULL` | Obligar a registrar un correo |
| `email` | `UNIQUE` | Evitar correos duplicados |
| `edad` | `CHECK (edad >= 18)` | Permitir únicamente edades de 18 o más |
| `pais` | `DEFAULT 'Perú'` | Usar Perú si no se especifica un país |

### Ejemplo de un registro válido

| id | email | edad | pais | ¿Por qué es válido? |
| :--- | :--- | ---: | :--- | :--- |
| 1 | ana@mail.com | 25 | Perú | Cumple todas las restricciones y `pais` usa su valor predeterminado |
| 2 | luis@mail.com | 30 | Chile | Cumple todas las restricciones y especifica el país |

Ahora crearemos la tabla `pedidos`:

```sql
CREATE TABLE pedidos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

En este caso, `cliente_id` es una **llave foránea**. Eso significa que cada pedido debe relacionarse con un cliente que exista en la tabla `clientes`.

---

## 4. `PRIMARY KEY`: identificar cada fila

La llave primaria identifica de manera única cada registro.

```sql
CREATE TABLE productos (
    id INT PRIMARY KEY,
    nombre VARCHAR(100)
);
```

### ¿Qué ocurriría si intentamos repetir un `id`?

```sql
INSERT INTO productos (id, nombre)
VALUES (1, 'Teclado');

-- Esto falla porque el id 1 ya existe
INSERT INTO productos (id, nombre)
VALUES (1, 'Mouse');
```

La base de datos rechaza el segundo registro porque dos filas no pueden tener la misma llave primaria.

### Comparación de resultados

| Operación | Estado | Motivo |
| :--- | :--- | :--- |
| Insertar `(1, 'Teclado')` | Aceptada | El identificador 1 todavía no existe |
| Insertar `(1, 'Mouse')` | Rechazada | El identificador 1 ya está ocupado |

> **Resultado esperado:** Error de duplicación de la llave primaria.

---

## 5. `NOT NULL`: obligar a registrar un valor

`NOT NULL` impide que una columna quede vacía.

```sql
INSERT INTO clientes (edad, pais)
VALUES (25, 'Perú');
```

Este `INSERT` falla porque no se proporcionó el `email`, y la columna fue definida de la siguiente forma:

```sql
email VARCHAR(100) NOT NULL UNIQUE
```

| email recibido | edad | Resultado | Explicación |
| :--- | ---: | :--- | :--- |
| `NULL` | 25 | Rechazado | `email` es obligatorio por `NOT NULL` |
| `ana@mail.com` | 25 | Aceptado | Se proporciona un correo válido |

### Ejemplo correcto

```sql
INSERT INTO clientes (email, edad)
VALUES ('ana@mail.com', 25);
```

Como no se indicó el país, se aplica el valor predeterminado:

| id | email | edad | pais |
| :--- | :--- | :--- | :--- |
| 1 | ana@mail.com | 25 | Perú |

---

## 6. `UNIQUE`: evitar valores repetidos

La restricción `UNIQUE` evita que dos clientes utilicen el mismo correo electrónico.

```sql
INSERT INTO clientes (email, edad, pais)
VALUES ('luis@mail.com', 30, 'Perú');

-- Esto falla porque el correo ya está registrado
INSERT INTO clientes (email, edad, pais)
VALUES ('luis@mail.com', 22, 'Chile');
```

### Resultado esperado

El primer `INSERT` funciona. El segundo es rechazado porque `email` tiene una restricción `UNIQUE`.

### Estado de la tabla después de los intentos

| id | email | edad | Resultado |
| :--- | :--- | ---: | :--- |
| 1 | `luis@mail.com` | 30 | Guardado |
| — | `luis@mail.com` | 22 | Rechazado por duplicado |

<blockquote style="border-left: 5px solid #7c3aed; padding-left: 12px;">
<strong>No los confundas.</strong> `UNIQUE` no es exactamente igual que `PRIMARY KEY`. Una tabla puede tener varias columnas `UNIQUE`, pero normalmente solo tiene una llave primaria.
</blockquote>

---

## 7. `CHECK`: validar una condición

`CHECK` permite exigir que un valor cumpla una condición determinada.

En nuestra tabla, la edad debe ser mayor o igual a 18:

```sql
edad TINYINT CHECK (edad >= 18)
```

### Ejemplo que falla

```sql
INSERT INTO clientes (email, edad)
VALUES ('menor@mail.com', 15);
```

El registro es rechazado porque `15 >= 18` es falso.

### Ejemplo correcto

```sql
INSERT INTO clientes (email, edad)
VALUES ('sofia@mail.com', 21);
```

Este registro sí cumple la condición.

### Valores evaluados por `CHECK`

| Edad enviada | Evaluación de `edad >= 18` | Resultado |
| ---: | :---: | :--- |
| 15 | Falsa | Rechazado |
| 18 | Verdadera | Aceptado |
| 21 | Verdadera | Aceptado |

<blockquote style="border-left: 5px solid #16a34a; padding-left: 12px;">
<strong>Pista práctica.</strong> `CHECK` sirve para reglas simples como edades, precios positivos, porcentajes entre 0 y 100 o estados permitidos.
</blockquote>

---

## 8. `DEFAULT`: colocar valores automáticamente

`DEFAULT` asigna un valor cuando el usuario no especifica uno.

En nuestra tabla:

```sql
pais VARCHAR(50) DEFAULT 'Perú'
```

### Ejemplo

```sql
INSERT INTO clientes (email, edad)
VALUES ('maria@mail.com', 28);
```

Aunque no se indicó `pais`, la base de datos completa el valor automáticamente:

| id | email | edad | pais |
| :--- | :--- | :--- | :--- |
| 3 | maria@mail.com | 28 | Perú |

Si queremos registrar otro país, podemos indicar el valor explícitamente:

```sql
INSERT INTO clientes (email, edad, pais)
VALUES ('carlos@mail.com', 35, 'Argentina');
```

---

## 9. `FOREIGN KEY`: relacionar tablas

Una llave foránea conecta una tabla hija con una tabla padre.

En este ejemplo:

- `clientes` es la tabla padre.
- `pedidos` es la tabla hija.
- `pedidos.cliente_id` apunta a `clientes.id`.

```sql
FOREIGN KEY (cliente_id) REFERENCES clientes(id)
```

Esto significa que no puede existir un pedido para un cliente inexistente.

### Relación entre las tablas

**Tabla padre: `clientes`**

| id | email |
| :---: | :--- |
| 1 | ana@mail.com |
| 2 | luis@mail.com |

**Tabla hija: `pedidos`**

| id | cliente_id | ¿Es válido? |
| :---: | :---: | :--- |
| 1 | 1 | Sí, el cliente 1 existe |
| 2 | 99 | No, el cliente 99 no existe |

### Ejemplo que falla

```sql
-- El cliente 99 no existe
INSERT INTO pedidos (cliente_id)
VALUES (99);
```

La base de datos rechaza la operación porque se estaría creando un pedido sin un cliente válido.

### Ejemplo correcto

```sql
-- Primero creamos el cliente
INSERT INTO clientes (email, edad)
VALUES ('ana@mail.com', 25);

-- Después creamos un pedido para ese cliente
INSERT INTO pedidos (cliente_id)
VALUES (1);
```

<blockquote style="border-left: 5px solid #0891b2; padding-left: 12px;">
<strong>Secuencia recomendada.</strong> Primero debe existir el registro de la tabla padre y después se puede insertar el registro relacionado en la tabla hija.
</blockquote>

---

## 10. Acciones de una `FOREIGN KEY`

Una llave foránea puede definir qué sucede cuando se actualiza o elimina el registro relacionado.

```sql
FOREIGN KEY (cliente_id) REFERENCES clientes(id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
```

### Opciones comunes de `ON DELETE`

| Acción | Efecto en la tabla hija |
| :--- | :--- |
| `CASCADE` | Borra también las filas relacionadas |
| `SET NULL` | Deja la llave foránea en `NULL` |
| `RESTRICT` | Impide borrar el registro padre si tiene hijos |
| `NO ACTION` | Similar a `RESTRICT` en la mayoría de motores |

---

## 11. Ejemplo completo con `ON DELETE CASCADE`

Supongamos que las tablas contienen estos datos:

### Tabla `clientes` antes del `DELETE`

| id | email | edad | pais |
| :--- | :--- | :--- | :--- |
| 1 | ana@mail.com | 25 | Perú |
| 2 | luis@mail.com | 30 | Perú |

### Tabla `pedidos` antes del `DELETE`

| id | cliente_id |
| :--- | :--- |
| 1 | 1 |
| 2 | 1 |
| 3 | 2 |

El cliente con `id = 1` tiene dos pedidos relacionados.

Ahora ejecutamos:

```sql
DELETE FROM clientes
WHERE id = 1;
```

Como la llave foránea utiliza `ON DELETE CASCADE`, también se eliminan automáticamente los pedidos del cliente 1.

### Tabla `clientes` después del `DELETE`

| id | email | edad | pais |
| :--- | :--- | :--- | :--- |
| 2 | luis@mail.com | 30 | Perú |

### Tabla `pedidos` después del `DELETE`

| id | cliente_id |
| :--- | :--- |
| 3 | 2 |

<blockquote style="border-left: 5px solid #dc2626; padding-left: 12px;">
<strong>Revisa antes de borrar.</strong> `CASCADE` puede borrar más datos de los esperados. Debe utilizarse únicamente cuando tenga sentido que los registros hijos desaparezcan junto con el registro padre.
</blockquote>

---

## 12. Ejemplo con `ON DELETE SET NULL`

Ahora imaginemos que queremos borrar un autor, pero conservar sus libros. En ese caso, los libros quedarían sin autor asignado.

```sql
CREATE TABLE autores (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE libros (
    id INT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    autor_id INT NULL,
    FOREIGN KEY (autor_id) REFERENCES autores(id)
        ON DELETE SET NULL
);
```

La columna `autor_id` debe permitir `NULL`. Por eso se escribe:

```sql
autor_id INT NULL
```

Si se elimina el autor con `id = 1`:

```sql
DELETE FROM autores
WHERE id = 1;
```

El libro no se elimina. Su relación cambia automáticamente:

| id | titulo | autor_id |
| :--- | :--- | :--- |
| 1 | Introducción a SQL | `NULL` |

<blockquote style="border-left: 5px solid #0f766e; padding-left: 12px;">
<strong>Qué debes recordar.</strong> Usa `SET NULL` cuando quieras conservar el registro hijo, pero permitir que deje de estar relacionado con el registro padre.
</blockquote>

---

## 13. Agregar un constraint a una tabla existente

Los constraints también pueden agregarse después de crear una tabla.

### Sintaxis general

```sql
ALTER TABLE nombre_tabla
ADD CONSTRAINT nombre_constraint
FOREIGN KEY (columna)
REFERENCES otra_tabla(columna);
```

### Ejemplo

```sql
ALTER TABLE pedidos
ADD CONSTRAINT fk_cliente
FOREIGN KEY (cliente_id)
REFERENCES clientes(id);
```

En este caso, `fk_cliente` es el nombre asignado a la restricción.

Nombrar los constraints ayuda a identificarlos cuando se necesita modificarlos o eliminarlos.

### Antes y después de agregar una restricción

| Situación | Estructura de `pedidos` | Consecuencia |
| :--- | :--- | :--- |
| Antes de agregar `fk_cliente` | `cliente_id` sin relación formal | Podrían entrar identificadores inexistentes |
| Después de agregar `fk_cliente` | `cliente_id` referencia `clientes(id)` | La base de datos valida cada pedido |

---

## 14. Errores comunes

### Error 1: confundir `UNIQUE` con `PRIMARY KEY`

`UNIQUE` evita duplicados, pero no necesariamente cumple todas las reglas de una llave primaria.

### Error 2: usar `ON DELETE CASCADE` sin analizar las consecuencias

Una eliminación puede propagarse a varias tablas y borrar una gran cantidad de información.

### Error 3: usar `SET NULL` junto con `NOT NULL`

La siguiente combinación es incompatible:

```sql
autor_id INT NOT NULL

FOREIGN KEY (autor_id) REFERENCES autores(id)
    ON DELETE SET NULL
```

`SET NULL` necesita poder colocar `NULL`, pero `NOT NULL` lo impide.

### Error 4: intentar insertar una llave foránea inexistente

```sql
INSERT INTO pedidos (cliente_id)
VALUES (999);
```

Si el cliente 999 no existe, la base de datos rechaza el pedido.

### Error 5: olvidar el motor de almacenamiento en MySQL

En MySQL, las llaves foráneas deben utilizar un motor que las soporte, como `InnoDB`. El motor antiguo `MyISAM` puede ignorar las `FOREIGN KEY`.

### Error 6: no hacer coincidir los tipos de datos

La columna de la llave foránea y la columna referenciada deben tener tipos compatibles.

Por ejemplo, si `clientes.id` es `INT`, `pedidos.cliente_id` también debería ser compatible con `INT`.

### Resumen visual de errores y soluciones

| Problema | Ejemplo | Solución |
| :--- | :--- | :--- |
| Correo repetido | Dos usuarios con el mismo `email` | Usar `UNIQUE` |
| Campo vacío | Cliente sin correo | Usar `NOT NULL` |
| Edad inválida | `edad = 15` | Usar `CHECK (edad >= 18)` |
| Pedido sin cliente | `cliente_id = 999` | Usar `FOREIGN KEY` |
| Borrado excesivo | `ON DELETE CASCADE` sin analizar | Revisar la acción antes de aplicarla |

---

## 15. Práctica guiada

Observa las siguientes tablas:

```sql
CREATE TABLE autores (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE libros (
    id INT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    autor_id INT NOT NULL,
    FOREIGN KEY (autor_id) REFERENCES autores(id)
);
```

### Pregunta 1
Quieres borrar un autor sin perder sus libros. Los libros deben quedar almacenados, pero sin autor asignado.

¿Qué acción usarías?

<details>
<summary>Ver respuesta</summary>

```sql
ON DELETE SET NULL
```

Además, `autor_id` debe permitir valores `NULL`, por lo que hay que cambiarlo:

```sql
autor_id INT NULL
```

La definición completa sería:

```sql
CREATE TABLE libros (
    id INT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    autor_id INT NULL,
    FOREIGN KEY (autor_id) REFERENCES autores(id)
        ON DELETE SET NULL
);
```

</details>

### Pregunta 2
Quieres impedir que se registren productos con un precio menor o igual a cero.

¿Qué constraint usarías?

<details>
<summary>Ver respuesta</summary>

```sql
precio DECIMAL(10, 2) CHECK (precio > 0)
```

</details>

### Pregunta 3
Quieres que el correo de cada usuario sea obligatorio y no se repita.

¿Qué constraints usarías?

<details>
<summary>Ver respuesta</summary>

```sql
email VARCHAR(100) NOT NULL UNIQUE
```

</details>

### Pregunta 4
¿Qué ocurriría al ejecutar este código?

```sql
INSERT INTO clientes (email, edad)
VALUES ('ana@mail.com', 16);
```

<details>
<summary>Ver respuesta</summary>

La operación fallaría porque la restricción `CHECK (edad >= 18)` no permite registrar una edad de 16.

</details>

---

## 16. Mini desafío final

Crea una tabla `empleados` que cumpla estas condiciones:

1. `id` debe identificar de forma única a cada empleado.
2. `nombre` no puede quedar vacío.
3. `email` no puede repetirse.
4. `edad` debe ser mayor o igual a 18.
5. `pais` debe tener `'Perú'` como valor predeterminado.

### Una solución posible

```sql
CREATE TABLE empleados (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    edad TINYINT CHECK (edad >= 18),
    pais VARCHAR(50) DEFAULT 'Perú'
);
```

### Verificación rápida

| Registro | `id` | `nombre` | `email` | `edad` | `pais` | Resultado |
| :--- | ---: | :--- | :--- | ---: | :--- | :--- |
| 1 | automático | Lucía | lucia@mail.com | 24 | Perú | Aceptado |
| 2 | automático | Pedro | pedro@mail.com | 15 | Perú | Rechazado por `CHECK` |
| 3 | automático | Otra Lucía | lucia@mail.com | 30 | Perú | Rechazado por `UNIQUE` |

```sql
-- Funciona
INSERT INTO empleados (nombre, email, edad)
VALUES ('Lucía', 'lucia@mail.com', 24);

-- Falla por edad inválida
INSERT INTO empleados (nombre, email, edad)
VALUES ('Pedro', 'pedro@mail.com', 15);

-- Falla por correo repetido
INSERT INTO empleados (nombre, email, edad)
VALUES ('Otra Lucía', 'lucia@mail.com', 30);
```

---

## 17. Resumen final

- `PRIMARY KEY` identifica cada fila de forma única.
- `FOREIGN KEY` mantiene relaciones válidas entre tablas.
- `NOT NULL` obliga a registrar un valor.
- `UNIQUE` evita valores duplicados.
- `DEFAULT` asigna un valor automático.
- `CHECK` valida una condición.
- `ON DELETE CASCADE` elimina automáticamente los registros relacionados.
- `ON DELETE SET NULL` conserva el registro hijo, pero elimina su relación.
- `RESTRICT` impide borrar un registro padre que todavía tiene registros hijos.

<blockquote style="border-left: 5px solid #0f766e; padding-left: 12px;">
<strong>Qué debes recordar.</strong> Los constraints permiten que la base de datos se proteja a sí misma y mantenga la información coherente, incluso cuando una aplicación comete errores al enviar datos.
</blockquote>

---

## Relacionado con otros temas

- [Tipos de datos](tipos-de-datos.md): los tipos de la llave foránea y la llave primaria deben ser compatibles.
- [DDL, DML y DQL](ddl-dml-dql.md): los constraints se definen principalmente mediante comandos DDL.
- Normalización: las llaves foráneas ayudan a organizar las relaciones entre tablas.
- Transacciones y locks: las operaciones en cascada pueden afectar varias filas y tablas dentro de una misma operación.
