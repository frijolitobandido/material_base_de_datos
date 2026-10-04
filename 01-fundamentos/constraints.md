# Constraints (restricciones)

## Teoria

Los constraints son reglas que el motor de base de datos aplica automaticamente para proteger la integridad de los datos. Son parte del estandar SQL y se comportan igual (con pequenas variaciones de sintaxis) en MySQL, PostgreSQL, SQL Server u Oracle. La idea central: es mejor que la base de datos rechace datos invalidos, a que la aplicacion "confie" en validarlos siempre bien.

### Tabla: constraints principales

| Constraint | Que hace | Permite NULL | Permite duplicados |
| :--- | :--- | :--- | :--- |
| `PRIMARY KEY` | Identifica de forma unica cada fila | No | No |
| `FOREIGN KEY` | Obliga a que un valor exista en otra tabla | Si (si la columna lo permite) | Si |
| `NOT NULL` | La columna no puede quedar vacia | No | Si |
| `UNIQUE` | No permite valores repetidos | Si (varios NULL permitidos) | No |
| `DEFAULT` | Asigna un valor si no se especifica uno | — | — |
| `CHECK` | Valida una condicion sobre el valor | — | — |

> **Nota:** `CHECK` es parte del estandar desde hace decadas, pero MySQL recien lo empezo a **aplicar de verdad** desde la version 8.0.16 (antes lo aceptaba en la sintaxis, pero lo ignoraba silenciosamente).

## Sintaxis / codigo base

**Esqueleto general al crear tabla:**
```sql
CREATE TABLE nombre_tabla (
    columna tipo_dato CONSTRAINT_1 CONSTRAINT_2,
    ...
    FOREIGN KEY (columna) REFERENCES otra_tabla(columna)
);
```

```sql
CREATE TABLE clientes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) NOT NULL UNIQUE,
    edad TINYINT CHECK (edad >= 18),
    pais VARCHAR(50) DEFAULT 'Peru'
);

CREATE TABLE pedidos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

**Agregar un constraint a una tabla existente:**
```sql
ALTER TABLE nombre_tabla
ADD CONSTRAINT nombre_constraint FOREIGN KEY (columna) REFERENCES otra_tabla(columna);
```

```sql
ALTER TABLE pedidos
ADD CONSTRAINT fk_cliente FOREIGN KEY (cliente_id) REFERENCES clientes(id);
```

## Ejemplo aplicado

Caso: relacion clientes -> pedidos, donde no puede existir un pedido sin cliente.

```sql
-- Esto falla: viola FOREIGN KEY porque cliente_id=99 no existe
INSERT INTO pedidos (cliente_id) VALUES (99);

-- Esto funciona
INSERT INTO clientes (email, edad) VALUES ('ana@mail.com', 25);
INSERT INTO pedidos (cliente_id) VALUES (1);

-- ON DELETE CASCADE: si borras el cliente 1, sus pedidos se borran tambien
DELETE FROM clientes WHERE id = 1;
```

**Simulacion antes del `DELETE` — `SELECT * FROM clientes;` y `SELECT * FROM pedidos;`**

Tabla `clientes`:

| id | email | edad | pais |
| :--- | :--- | :--- | :--- |
| 1 | ana@mail.com | 25 | Peru |
| 2 | luis@mail.com | 30 | Peru |

Tabla `pedidos`:

| id | cliente_id |
| :--- | :--- |
| 1 | 1 |
| 2 | 1 |
| 3 | 2 |

**Simulacion despues de `DELETE FROM clientes WHERE id = 1;`**

Tabla `clientes`:

| id | email | edad | pais |
| :--- | :--- | :--- | :--- |
| 2 | luis@mail.com | 30 | Peru |

Tabla `pedidos`:

| id | cliente_id |
| :--- | :--- |
| 3 | 2 |

> **Nota:** Al borrar el cliente 1, `ON DELETE CASCADE` borro tambien sus pedidos (id 1 y 2) de forma automatica. Con `RESTRICT` en vez de `CASCADE`, ese `DELETE` habria fallado por integridad referencial.

| Accion sobre `ON DELETE` | Efecto en la tabla hija |
| :--- | :--- |
| `CASCADE` | Borra tambien las filas relacionadas |
| `SET NULL` | Deja la columna FK en NULL (debe permitir NULL) |
| `RESTRICT` (por defecto) | Impide borrar el padre si tiene hijos relacionados |
| `NO ACTION` | Similar a RESTRICT en la mayoria de motores |

## Errores comunes

- Olvidar que `UNIQUE` permite multiples `NULL` (no es lo mismo que `PRIMARY KEY`).
- Usar `ON DELETE CASCADE` sin pensarlo bien: puede borrar mas datos de los esperados en cadena.
- Definir `FOREIGN KEY` sin que la columna referenciada tenga un indice o PK; la mayoria de motores lo exige.
- En MySQL, el motor `MyISAM` ignora las `FOREIGN KEY` por completo (solo se validan con `InnoDB`); esto es una particularidad del motor, no del estandar.

## Ejercicio

Tienes dos tablas: `autores (id, nombre)` y `libros (id, titulo, autor_id)`, donde `autor_id` es FOREIGN KEY hacia `autores(id)`. Quieres poder borrar un autor sin que MySQL te bloquee, pero sin perder el historial de sus libros (solo quieres que el libro quede "sin autor").

¿Que opcion de `ON DELETE` usarias y que condicion debe cumplir la columna `autor_id` para que funcione?

<details>
<summary>Ver respuesta</summary>

```sql
FOREIGN KEY (autor_id) REFERENCES autores(id)
    ON DELETE SET NULL
```

La columna `autor_id` debe permitir `NULL` (no puede tener `NOT NULL`), porque `ON DELETE SET NULL` necesita poder dejar ese campo vacio cuando se borra el autor relacionado.

</details>

## Relacionado con
- [Tipos de datos](tipos-de-datos.md) — el tipo de la FK debe coincidir con el tipo de la PK referenciada.
- [DDL, DML, DQL](ddl-dml-dql.md) — los constraints se definen en el DDL.
- Normalizacion *(pendiente)* — las FK son la base de las relaciones entre tablas normalizadas.
- Transacciones / Locks *(pendiente)* — `ON DELETE CASCADE` interactua con locks en operaciones grandes.