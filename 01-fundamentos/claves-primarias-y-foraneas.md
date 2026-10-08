# Claves Primarias y Foráneas en SQL

Las **claves primarias** y **claves foráneas** permiten identificar registros y establecer relaciones entre tablas.

- Una **clave primaria** identifica de forma única cada fila de su propia tabla.

- Una **clave foránea** conecta una tabla con otra y mantiene la integridad de esa relación.

Estas claves son constraints, pero se estudian en un archivo separado porque son fundamentales para comprender el diseño relacional y los `JOIN`.

> **Idea central.** La clave primaria responde a “¿qué fila es esta?”; la clave foránea responde a “¿con qué registro de otra tabla se relaciona?”.

---

## 1. ¿Qué es una clave primaria?

Una **clave primaria** o `PRIMARY KEY` es una columna, o combinación de columnas, que identifica de forma única cada fila de una tabla.

Una clave primaria debe cumplir estas condiciones:

| Característica | ¿Qué significa? |
| --- | --- |
| Unicidad | No puede haber dos filas con la misma clave primaria |
| No admite `NULL` | Cada fila debe tener un identificador |
| Identifica una fila | Permite localizar un registro específico |
| Una por tabla | Una tabla tiene una sola definición de clave primaria, aunque puede estar formada por varias columnas |
| Puede ser simple o compuesta | Puede usar una columna o varias columnas combinadas |

### Ejemplo básico

```sql
CREATE TABLE estudiantes (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    correo VARCHAR(120)
);
```

En este ejemplo, `id` identifica de forma única a cada estudiante.

### Registros de ejemplo

| id | nombre | correo |
| --- | --- | --- |
| 1 | Lucía | [lucia@mail.com](mailto:lucia@mail.com) |
| 2 | Pedro | [pedro@mail.com](mailto:pedro@mail.com) |
| 3 | Ana | [ana@mail.com](mailto:ana@mail.com) |

Cada fila tiene un valor diferente en `id`.

---

## 2. ¿Qué ocurre si se repite una clave primaria?

```sql
INSERT INTO estudiantes (id, nombre, correo)
VALUES (1, 'Carlos', 'carlos@mail.com');
```

Esta operación falla si ya existe un estudiante con `id = 1`.

### Comparación de intentos

| Operación | Resultado | Motivo |
| --- | --- | --- |
| Insertar `(1, 'Lucía')` | Aceptada | El identificador 1 no existía |
| Insertar `(2, 'Pedro')` | Aceptada | El identificador 2 no existía |
| Insertar `(1, 'Carlos')` | Rechazada | El identificador 1 ya existe |
| Insertar `(NULL, 'Ana')` | Rechazada | Una clave primaria no puede ser `NULL` |

La restricción evita que dos personas distintas sean identificadas con el mismo `id`.

---

## 3. Clave primaria con `AUTO_INCREMENT`

Cuando el motor genera los identificadores automáticamente, se puede utilizar `AUTO_INCREMENT` en MySQL.

```sql
CREATE TABLE productos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio DECIMAL(10,2) NOT NULL
);
```

### Insertar sin indicar el `id`

```sql
INSERT INTO productos (nombre, precio)
VALUES ('Teclado', 149.90);

INSERT INTO productos (nombre, precio)
VALUES ('Mouse', 79.90);
```

### Resultado simulado

| id generado | nombre | precio |
| --- | --- | --- |
| 1 | Teclado | 149.90 |
| 2 | Mouse | 79.90 |

> **Compatibilidad entre motores.** `AUTO_INCREMENT` es una forma frecuente de MySQL. PostgreSQL utiliza alternativas como `GENERATED ALWAYS AS IDENTITY`, y otros motores tienen su propia sintaxis.

### Alternativa estándar

```sql
CREATE TABLE productos_estandar (
    id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);
```

La sintaxis exacta debe comprobarse según el SGBD utilizado.

---

## 4. Clave primaria compuesta

Una **clave primaria compuesta** utiliza dos o más columnas juntas para identificar una fila.

Es útil cuando ninguna columna por sí sola identifica el registro, pero la combinación sí lo hace.

### Ejemplo: inscripciones

```sql
CREATE TABLE inscripciones (
    alumno_id INT,
    curso_id INT,
    fecha_inscripcion DATE NOT NULL,
    PRIMARY KEY (alumno_id, curso_id)
);
```

En este caso:

- Un alumno puede inscribirse en varios cursos.

- Un curso puede tener varios alumnos.

- El mismo alumno no puede inscribirse dos veces en el mismo curso.

### Registros de ejemplo

| alumno_id | curso_id | fecha_inscripcion | ¿Se acepta? | Motivo |
| --- | --- | --- | --- | --- |
| 1 | 10 | 2026-10-01 | Sí | Primera combinación |
| 1 | 11 | 2026-10-02 | Sí | El curso es diferente |
| 2 | 10 | 2026-10-02 | Sí | El alumno es diferente |
| 1 | 10 | 2026-10-05 | No | La combinación ya existe |

> **No los confundas.** Una clave compuesta no significa que cada columna sea única por separado. La regla se aplica a la combinación completa.

---

## 5. ¿Qué es una clave foránea?

Una **clave foránea** o `FOREIGN KEY` es una columna, o conjunto de columnas, que apunta a una clave primaria o candidata de otra tabla.

Su función principal es mantener la **integridad referencial**.

### Conceptos relacionados

| Concepto | Significado |
| --- | --- |
| Tabla padre | Tabla que contiene la clave referenciada |
| Tabla hija | Tabla que contiene la clave foránea |
| Clave referenciada | Columna que identifica al registro padre |
| Clave foránea | Columna que almacena la referencia al padre |
| Integridad referencial | Regla que evita referencias inexistentes |

### Ejemplo conceptual

| Tabla `clientes` — padre | Tabla `pedidos` — hija |
| --- | --- |
| `clientes.id` | `pedidos.cliente_id` |
| 1 — Ana | 1 — pedido para cliente 1 |
| 2 — Luis | 2 — pedido para cliente 2 |

La columna `pedidos.cliente_id` apunta a `clientes.id`.

---

## 6. Crear tablas relacionadas

Primero se crea la tabla padre:

```sql
CREATE TABLE clientes (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    correo VARCHAR(120) UNIQUE
);
```

Después se crea la tabla hija:

```sql
CREATE TABLE pedidos (
    id INT PRIMARY KEY,
    cliente_id INT NOT NULL,
    fecha_pedido DATE NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);
```

### ¿Qué garantiza la relación?

| Tabla | Columna | Función |
| --- | --- | --- |
| `clientes` | `id` | Identifica al cliente |
| `pedidos` | `cliente_id` | Guarda el cliente asociado al pedido |
| Relación | `pedidos.cliente_id → clientes.id` | Impide usar un cliente inexistente |

---

## 7. Insertar datos en tablas relacionadas

La tabla padre debe tener el registro antes de insertar una fila que lo referencie.

### Insertar clientes

```sql
INSERT INTO clientes (id, nombre, correo)
VALUES
    (1, 'Ana', 'ana@mail.com'),
    (2, 'Luis', 'luis@mail.com');
```

### Insertar pedidos válidos

```sql
INSERT INTO pedidos (id, cliente_id, fecha_pedido, total)
VALUES
    (101, 1, '2026-10-04', 45.90),
    (102, 2, '2026-10-04', 80.00);
```

### Estado de las tablas

**Tabla padre: ****`clientes`**

| id | nombre | correo |
| --- | --- | --- |
| 1 | Ana | [ana@mail.com](mailto:ana@mail.com) |
| 2 | Luis | [luis@mail.com](mailto:luis@mail.com) |

**Tabla hija: ****`pedidos`**

| id | cliente_id | fecha_pedido | total |
| --- | --- | --- | --- |
| 101 | 1 | 2026-10-04 | 45.90 |
| 102 | 2 | 2026-10-04 | 80.00 |

### Insertar un pedido inválido

```sql
INSERT INTO pedidos (id, cliente_id, fecha_pedido, total)
VALUES (103, 99, '2026-10-04', 20.00);
```

La operación es rechazada porque no existe un cliente con `id = 99`.

| cliente_id enviado | ¿Existe en `clientes`? | Resultado |
| --- | --- | --- |
| 1 | Sí | Pedido aceptado |
| 2 | Sí | Pedido aceptado |
| 99 | No | Pedido rechazado |

> **Secuencia recomendada.** Inserta primero los registros de la tabla padre y después los registros relacionados de la tabla hija.

---

## 8. ¿Puede una clave foránea ser `NULL`?

Sí, una clave foránea puede permitir `NULL` si no se define como `NOT NULL`.

```sql
CREATE TABLE tareas_proyecto (
    id INT PRIMARY KEY,
    descripcion VARCHAR(150) NOT NULL,
    responsable_id INT,
    FOREIGN KEY (responsable_id) REFERENCES clientes(id)
);
```

En este caso, una tarea puede existir sin responsable asignado.

| responsable_id | Significado | ¿Se acepta? |
| --- | --- | --- |
| 1 | Responsable: Ana | Sí |
| 2 | Responsable: Luis | Sí |
| `NULL` | Todavía no hay responsable | Sí |
| 99 | Responsable inexistente | No |

Si la columna fuera obligatoria:

```sql
responsable_id INT NOT NULL
```

Entonces no se permitirían tareas sin responsable.

---

## 9. Acciones al eliminar o actualizar el padre

Una clave foránea puede definir qué sucede con las filas hijas cuando se elimina o actualiza el registro padre.

```sql
FOREIGN KEY (cliente_id) REFERENCES clientes(id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
```

### Acciones frecuentes de `ON DELETE`

| Acción | Qué ocurre en la tabla hija | Cuándo usarla |
| --- | --- | --- |
| `CASCADE` | Elimina las filas relacionadas | Cuando los hijos no tienen sentido sin el padre |
| `SET NULL` | Coloca la FK en `NULL` | Cuando se conserva el hijo sin relación |
| `RESTRICT` | Impide eliminar el padre | Cuando deben conservarse los hijos |
| `NO ACTION` | Similar a `RESTRICT` en muchos motores | Según las reglas del SGBD |

### Acciones frecuentes de `ON UPDATE`

| Acción | Qué ocurre |
| --- | --- |
| `CASCADE` | Actualiza la referencia en las filas hijas |
| `RESTRICT` | Impide cambiar la clave si hay referencias |
| `SET NULL` | Coloca `NULL` en las filas hijas, si la columna lo permite |

---

## 10. Ejemplo con `ON DELETE CASCADE`

```sql
CREATE TABLE clientes_cascade (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE pedidos_cascade (
    id INT PRIMARY KEY,
    cliente_id INT NOT NULL,
    total DECIMAL(10,2),
    FOREIGN KEY (cliente_id) REFERENCES clientes_cascade(id)
        ON DELETE CASCADE
);
```

### Estado antes de eliminar

**Clientes**

| id | nombre |
| --- | --- |
| 1 | Ana |
| 2 | Luis |

**Pedidos**

| id | cliente_id | total |
| --- | --- | --- |
| 101 | 1 | 45.90 |
| 102 | 1 | 20.00 |
| 103 | 2 | 80.00 |

Ahora se ejecuta:

```sql
DELETE FROM clientes_cascade
WHERE id = 1;
```

### Estado después de eliminar

**Clientes**

| id | nombre |
| --- | --- |
| 2 | Luis |

**Pedidos**

| id | cliente_id | total |
| --- | --- | --- |
| 103 | 2 | 80.00 |

Los pedidos `101` y `102` desaparecieron automáticamente porque pertenecían al cliente eliminado.

> **Revisa antes de usar ****`CASCADE`****.** Una eliminación puede propagarse a muchas filas relacionadas y borrar más información de la esperada.

---

## 11. Ejemplo con `ON DELETE SET NULL`

`SET NULL` permite eliminar el registro padre y conservar los registros hijos.

La clave foránea debe permitir `NULL`.

```sql
CREATE TABLE autores (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE libros (
    id INT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    autor_id INT,
    FOREIGN KEY (autor_id) REFERENCES autores(id)
        ON DELETE SET NULL
);
```

### Estado antes de eliminar

| libro_id | titulo | autor_id |
| --- | --- | --- |
| 1 | SQL desde cero | 10 |
| 2 | Consultas avanzadas | 10 |
| 3 | Diseño relacional | 11 |

Se elimina el autor `10`:

```sql
DELETE FROM autores
WHERE id = 10;
```

### Estado después de eliminar

| libro_id | titulo | autor_id | Resultado |
| --- | --- | --- | --- |
| 1 | SQL desde cero | `NULL` | Libro conservado, sin autor |
| 2 | Consultas avanzadas | `NULL` | Libro conservado, sin autor |
| 3 | Diseño relacional | 11 | No se modifica |

> **Qué debes recordar.** `SET NULL` conserva la fila hija, pero necesita que la columna foránea permita valores `NULL`.

---

## 12. Ejemplo con `ON DELETE RESTRICT`

`RESTRICT` impide eliminar un registro padre mientras existan filas hijas relacionadas.

```sql
CREATE TABLE departamentos (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE empleados_departamento (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    departamento_id INT NOT NULL,
    FOREIGN KEY (departamento_id) REFERENCES departamentos(id)
        ON DELETE RESTRICT
);
```

Si existen empleados en el departamento `5`, esta operación será rechazada:

```sql
DELETE FROM departamentos
WHERE id = 5;
```

| Situación | Resultado |
| --- | --- |
| Departamento sin empleados | Puede eliminarse |
| Departamento con empleados | Eliminación rechazada |
| Se trasladan primero los empleados | Después puede evaluarse la eliminación |

---

## 13. Claves foráneas compuestas

Una relación también puede utilizar más de una columna.

```sql
CREATE TABLE detalle_inscripciones (
    alumno_id INT,
    curso_id INT,
    periodo VARCHAR(20),
    nota DECIMAL(4,2),
    PRIMARY KEY (alumno_id, curso_id, periodo)
);
```

Cuando una clave foránea referencia una clave compuesta, debe incluir las mismas columnas y en el mismo orden lógico:

```sql
CREATE TABLE evaluaciones (
    alumno_id INT,
    curso_id INT,
    periodo VARCHAR(20),
    calificacion DECIMAL(4,2),
    FOREIGN KEY (alumno_id, curso_id, periodo)
        REFERENCES detalle_inscripciones(alumno_id, curso_id, periodo)
);
```

| alumno_id | curso_id | periodo | ¿Referencia válida? |
| --- | --- | --- | --- |
| 1 | 10 | 2026-1 | Sí, existe la combinación |
| 1 | 10 | 2026-2 | Depende de si existe ese periodo |
| 1 | 11 | 2026-1 | Depende de si existe esa combinación |

---

## 14. Agregar una clave foránea a una tabla existente

La relación también puede agregarse después de crear las tablas.

```sql
ALTER TABLE pedidos
ADD CONSTRAINT fk_pedidos_cliente
FOREIGN KEY (cliente_id)
REFERENCES clientes(id);
```

### Antes de agregarla

```sql
SELECT p.cliente_id
FROM pedidos p
LEFT JOIN clientes c ON c.id = p.cliente_id
WHERE c.id IS NULL
  AND p.cliente_id IS NOT NULL;
```

Si la consulta devuelve filas, existen pedidos con clientes inexistentes y primero deben corregirse.

| Situación encontrada | ¿Puede agregarse la FK? | Acción |
| --- | --- | --- |
| Todas las referencias existen | Sí | Crear la restricción |
| Hay referencias inexistentes | No, normalmente | Corregir, eliminar o asignar las filas |
| Hay `NULL` y la columna los permite | Sí | `NULL` significa sin relación |

---

## 15. Consultar tablas relacionadas con `JOIN`

Las claves primarias y foráneas permiten combinar información de varias tablas.

```sql
SELECT
    p.id AS pedido_id,
    c.nombre AS cliente,
    p.total
FROM pedidos p
INNER JOIN clientes c
    ON p.cliente_id = c.id;
```

### Resultado simulado

| pedido_id | cliente | total |
| --- | --- | --- |
| 101 | Ana | 45.90 |
| 102 | Luis | 80.00 |

La condición del `JOIN` conecta:

```sql
p.cliente_id = c.id
```

| Columna de la tabla hija | Columna de la tabla padre |
| --- | --- |
| `pedidos.cliente_id` | `clientes.id` |

> **Conexión importante.** La clave foránea mantiene la relación válida; el `JOIN` utiliza esa relación para consultar información combinada.

---

## 16. Diferencias entre `PRIMARY KEY`, `UNIQUE` y `FOREIGN KEY`

| Restricción | Propósito | ¿Puede repetirse? | ¿Puede ser `NULL`? | ¿Apunta a otra tabla? |
| --- | --- | --- | --- | --- |
| `PRIMARY KEY` | Identificar cada fila | No | No | No necesariamente |
| `UNIQUE` | Evitar duplicados | No, con particularidades de `NULL` | Depende del motor y definición | No |
| `FOREIGN KEY` | Mantener una relación | Sí | Sí, si se permite | Sí |

### Ejemplo visual

| Tabla `clientes` | Tabla `pedidos` |
| --- | --- |
| `id = 1` — clave primaria | `cliente_id = 1` — clave foránea |
| `id = 2` — clave primaria | `cliente_id = 2` — clave foránea |
| `correo` — puede ser `UNIQUE` | `total` — dato del pedido |

---

## 17. Errores comunes

| Error | Qué ocurre | Cómo evitarlo |
| --- | --- | --- |
| Crear la tabla hija antes que la padre | La referencia puede fallar | Crear primero la tabla padre |
| Insertar una FK inexistente | El registro es rechazado | Verificar que exista el padre |
| Usar `SET NULL` junto con `NOT NULL` | La acción es incompatible | Permitir `NULL` en la FK |
| Usar `CASCADE` sin analizarlo | Puede borrar muchos registros | Revisar las dependencias antes |
| Borrar el padre con `RESTRICT` | La operación es rechazada | Eliminar o reasignar primero los hijos |
| No hacer coincidir los tipos | La FK puede no crearse | Usar tipos compatibles |
| Referenciar columnas no únicas | La relación puede ser inválida | Referenciar una PK o clave candidata |
| Olvidar una columna de una FK compuesta | La relación queda incompleta | Usar todas las columnas correspondientes |
| Intentar agregar la FK con datos inválidos | `ALTER TABLE` puede fallar | Auditar referencias antes |

### Tipos compatibles

```sql
CREATE TABLE clientes_tipo (
    id INT PRIMARY KEY
);

CREATE TABLE pedidos_tipo (
    id INT PRIMARY KEY,
    cliente_id INT,
    FOREIGN KEY (cliente_id) REFERENCES clientes_tipo(id)
);
```

La columna `cliente_id` debe ser compatible con `clientes_tipo.id` en tipo, signo y características relevantes del motor.

---

## 18. Práctica guiada

### Pregunta 1: identificar las claves

Observa estas tablas:

```sql
CREATE TABLE autores_practica (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE libros_practica (
    id INT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    autor_id INT,
    FOREIGN KEY (autor_id) REFERENCES autores_practica(id)
);
```

¿Cuál es la clave primaria de cada tabla y cuál es la clave foránea?

<details>
<summary>Ver respuesta</summary>

| Tabla | Clave |
| --- | --- |
| `autores_practica` | `id` es la clave primaria |
| `libros_practica` | `id` es la clave primaria |
| `libros_practica` | `autor_id` es la clave foránea que apunta a `autores_practica.id` |

</details>

### Pregunta 2: insertar un libro

Registra un libro para el autor con `id = 3`.

<details>
<summary>Ver respuesta</summary>

```sql
INSERT INTO libros_practica (id, titulo, autor_id)
VALUES (20, 'SQL intermedio', 3);
```

Funciona únicamente si el autor `3` existe.

</details>

### Pregunta 3: elegir una acción de borrado

Quieres eliminar un autor, pero conservar sus libros sin autor asignado.

¿Qué acción usarías?

<details>
<summary>Ver respuesta</summary>

```sql
ON DELETE SET NULL
```

Además, `autor_id` no debe tener `NOT NULL`.

</details>

### Pregunta 4: clave compuesta

Una tabla registra la inscripción de alumnos a cursos. Un alumno puede estar en muchos cursos, pero no debe repetirse en el mismo curso.

¿Qué clave primaria podrías utilizar?

<details>
<summary>Ver respuesta</summary>

```sql
PRIMARY KEY (alumno_id, curso_id)
```

La combinación identifica una inscripción única.

</details>

---

## 19. Mini desafío final

Crea un modelo sencillo para una biblioteca con estas condiciones:

1. Cada autor debe tener un identificador único.

1. Cada libro debe tener un identificador único.

1. Cada libro puede pertenecer a un autor.

1. No debe existir un libro con un autor inexistente.

1. Si se elimina un autor, los libros deben conservarse sin autor asignado.

### Una solución posible

```sql
CREATE TABLE autores_biblioteca (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE libros_biblioteca (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    autor_id INT,
    FOREIGN KEY (autor_id)
        REFERENCES autores_biblioteca(id)
        ON DELETE SET NULL
);
```

### Resultado esperado

| Operación | Resultado |
| --- | --- |
| Insertar autor con `id = 1` | Aceptado |
| Insertar libro con `autor_id = 1` | Aceptado |
| Insertar libro con `autor_id = 99` | Rechazado |
| Eliminar autor `id = 1` | Aceptado |
| Consultar el libro después de eliminar el autor | El libro permanece con `autor_id = NULL` |

### Prueba de funcionamiento

```sql
INSERT INTO autores_biblioteca (nombre)
VALUES ('María Pérez');

INSERT INTO libros_biblioteca (titulo, autor_id)
VALUES ('Fundamentos de SQL', 1);

-- Falla si el autor 99 no existe
INSERT INTO libros_biblioteca (titulo, autor_id)
VALUES ('Libro inválido', 99);

-- Conserva el libro, pero elimina la relación con el autor
DELETE FROM autores_biblioteca
WHERE id = 1;
```

---

## 20. Resumen final

- `PRIMARY KEY` identifica de forma única cada fila.

- Una clave primaria no admite duplicados ni `NULL`.

- Una clave primaria puede ser simple o compuesta.

- `AUTO_INCREMENT` genera identificadores automáticamente en MySQL.

- `FOREIGN KEY` conecta una tabla hija con una tabla padre.

- La integridad referencial impide referencias a registros inexistentes.

- Una FK puede permitir `NULL` cuando la relación es opcional.

- `CASCADE`, `SET NULL` y `RESTRICT` determinan qué ocurre al eliminar o actualizar el padre.

- Las FK compuestas deben utilizar las columnas correspondientes de la clave referenciada.

- Las claves primarias y foráneas permiten realizar `JOIN` de forma coherente.

- Antes de agregar una FK a datos existentes, hay que revisar las referencias inválidas.

> **Qué debes recordar.** Las claves primarias identifican los registros; las claves foráneas conectan esos registros y protegen la coherencia de las relaciones entre tablas.

---

## Relacionado con otros temas

- [Tipos de datos](tipos-de-datos.md): las columnas de las claves deben utilizar tipos compatibles.

- [DDL, DML y DQL](ddl-dml-dql.md): las claves se definen con `CREATE TABLE` o `ALTER TABLE`.

- [Constraints](constraints.md): las claves primarias y foráneas son restricciones de integridad.

- [Joins](../03-consultas-relacionales/joins.md): utiliza las relaciones entre tablas para consultar datos combinados.

- [Normalización](../04-diseno-modelado/normalizacion.md): organiza la información y reduce redundancias.

