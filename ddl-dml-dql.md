# DDL, DML y DQL

## Teoria 

SQL se divide en sublenguajes segun que hacen: unos definen la estructura, otros manipulan los datos y otros solo consultan.

| Sublenguaje | Nombre completo | Que hace | Comandos principales |
| :--- | :--- | :--- | :--- |
| DDL | Data Definition Language | Define o modifica la estructura de la base de datos | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| DML | Data Manipulation Language | Modifica los datos dentro de las tablas | `INSERT`, `UPDATE`, `DELETE` |
| DQL | Data Query Language | Consulta datos, no los modifica | `SELECT` |

Diferencia importante: en MySQL, la mayoria de comandos DDL hacen commit automatico (no se pueden revertir con `ROLLBACK`), mientras que DML si participa en transacciones. `TRUNCATE` parece DML pero en realidad es DDL: borra todos los registros y reinicia el `AUTO_INCREMENT`, y no se puede revertir como un `DELETE` dentro de una transaccion.

## Sintaxis / codigo base

### DDL: crear, modificar y eliminar estructuras

**Esqueleto — crear tabla:**
```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_dato,
    columna2 tipo_dato,
    ...
);
```

```sql
CREATE TABLE marcas (
    clave INT PRIMARY KEY,
    nombre VARCHAR(45)
);
```

**Esqueleto — agregar columna:**
```sql
ALTER TABLE nombre_tabla
ADD nombre_columna tipo_dato;
```

```sql
ALTER TABLE marcas
ADD comentarios VARCHAR(100);
```

**Esqueleto — eliminar tabla:**
```sql
DROP TABLE nombre_tabla;
```

```sql
DROP TABLE tabla_temporal;
```

### DML: insertar, actualizar y borrar datos

> Cuidado con `UPDATE` y `DELETE`: siempre usar `WHERE` para indicar que fila afectar. Si se omite, se modifica o borra **toda la tabla**.

**Esqueleto — insertar:**
```sql
INSERT INTO nombre_tabla (columna1, columna2, ...)
VALUES (valor1, valor2, ...);
```

```sql
INSERT INTO marcas (clave, nombre)
VALUES (1, 'De la rosa'), (2, 'Ricolino'), (3, 'Adams');
```

**Esqueleto — actualizar:**
```sql
UPDATE nombre_tabla
SET columna = nuevo_valor
WHERE condicion;
```

```sql
UPDATE marcas
SET nombre = 'Adams Mexico'
WHERE clave = 3;
```

**Esqueleto — borrar:**
```sql
DELETE FROM nombre_tabla
WHERE condicion;
```

```sql
DELETE FROM marcas
WHERE clave = 2;
```

### DQL: consultar

```sql
SELECT nombre, clave FROM marcas WHERE clave = 1;
```

## Ejemplo aplicado

Flujo tipico al crear una feature nueva: primero defines la tabla (DDL), luego insertas datos de prueba (DML) y finalmente verificas con consultas (DQL).

```sql
-- 1. Definir estructura (DDL)
CREATE TABLE pedidos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    estado VARCHAR(20) DEFAULT 'pendiente'
);

-- 2. Insertar datos de prueba (DML)
INSERT INTO pedidos (cliente_id, total) VALUES (1, 45.90);

-- 3. Verificar (DQL)
SELECT * FROM pedidos WHERE estado = 'pendiente';
```

**Simulacion de `SELECT * FROM pedidos WHERE estado = 'pendiente';`**

| id | cliente_id | total | estado |
| :--- | :--- | :--- | :--- |
| 1 | 1 | 45.90 | pendiente |
| 2 | 3 | 12.00 | pendiente |

> **Nota:** El pedido con `estado = 'entregado'` no aparece aqui porque no cumple el `WHERE`, aunque sigue existiendo en la tabla.

Caso combinado — actualizar con una operacion matematica: aumentar en 1.00 el total de todos los pedidos pendientes.

```sql
UPDATE pedidos
SET total = total + 1.00
WHERE estado = 'pendiente';
```

**Simulacion del resultado tras el `UPDATE` (`SELECT * FROM pedidos;`)**

| id | cliente_id | total (antes) | total (despues) | estado |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 45.90 | 46.90 | pendiente |
| 2 | 3 | 12.00 | 13.00 | pendiente |
| 3 | 2 | 8.50 | 8.50 | entregado |

> **Nota:** El pedido con id 3 tiene `estado = 'entregado'`, por eso su total no cambio aunque el `UPDATE` se ejecuto sobre toda la tabla.

Caso combinado — borrar usando una subconsulta: eliminar los pedidos del cliente cuyo nombre es "Ana" sin conocer su id de memoria.

```sql
DELETE FROM pedidos
WHERE cliente_id = (
    SELECT id FROM clientes WHERE nombre = 'Ana'
);
```

## Errores comunes 

| Error | Por que pasa | Como evitarlo |
| :--- | :--- | :--- |
| `ALTER TABLE` dentro de una transaccion esperando poder revertirlo | DDL hace commit automatico en MySQL | Hacer los cambios de estructura fuera de transacciones largas, con respaldo previo |
| Confundir `DELETE` con `TRUNCATE` | Ambos "vacian" la tabla, pero se comportan distinto | `DELETE` es DML (revertible, admite WHERE); `TRUNCATE` es DDL (todo o nada) |
| Olvidar el `WHERE` en `UPDATE`/`DELETE` | Falta de costumbre o prisa | Escribir primero el `SELECT` con la misma condicion para verificar antes de ejecutar |

## Relacionado con
- [Tipos de datos](tipos-de-datos.md) — se usan al definir columnas en el DDL.
- [Constraints](constraints.md) — se declaran junto con el DDL (`CREATE`/`ALTER`).
- Transacciones (ACID) *(pendiente)* — clave para entender por que DDL y DML se comportan distinto.
