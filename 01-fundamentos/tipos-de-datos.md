# Tipos de datos en SQL

## Teoria

El estandar SQL agrupa los tipos de datos en tres grandes familias: numericos, cadenas (texto) y fecha/hora. Casi todos los motores (MySQL, PostgreSQL, SQL Server, Oracle) implementan una version de estos mismos tipos, aunque con nombres o limites ligeramente distintos. Elegir el tipo correcto no es solo "que funcione": afecta el espacio en disco, la velocidad de los indices y hasta la precision de los calculos.

Puntos clave que se suelen olvidar:
- `CHAR` es de longitud fija (rellena con espacios), `VARCHAR` es de longitud variable. Esto aplica igual en casi todos los motores.
- `FLOAT`/`DOUBLE` son aproximados (no usar para dinero). `DECIMAL`/`NUMERIC` es exacto. `NUMERIC` es el nombre mas usado en el estandar; MySQL trata `DECIMAL` y `NUMERIC` como sinonimos.
- `TIMESTAMP` suele guardarse en UTC y depender de la zona horaria de la sesion; `DATETIME` no.

> **Nota:** `ENUM` y `SET` que se ven mas abajo **no son parte del estandar SQL**, son una extension propia de MySQL. Otros motores logran lo mismo con un `CHECK` o una tabla de referencia con `FOREIGN KEY`.

### Tabla: tipos numericos

| Tipo | Tamano aprox. | Rango (con signo) | Uso tipico |
| :--- | :--- | :--- | :--- |
| `TINYINT` | 1 byte | -128 a 127 | Flags, edades, estados pequenos |
| `SMALLINT` | 2 bytes | -32,768 a 32,767 | Stock, contadores medianos |
| `INT` / `INTEGER` | 4 bytes | -2,147,483,648 a 2,147,483,647 | IDs, contadores generales |
| `BIGINT` | 8 bytes | Rango muy grande (~9.2 x 10^18) | IDs de sistemas masivos |
| `DECIMAL(m,d)` / `NUMERIC(m,d)` | Variable | Exacto, definido por m y d | Dinero, precios, montos |
| `FLOAT` | 4 bytes | Aproximado | Calculos cientificos, no dinero |
| `DOUBLE` / `DOUBLE PRECISION` | 8 bytes | Aproximado, mayor precision que FLOAT | Calculos cientificos de mayor precision |

### Tabla: tipos de cadena (texto)

| Tipo | Longitud maxima | Caracteristica | Uso tipico |
| :--- | :--- | :--- | :--- |
| `CHAR(n)` | 255 caracteres | Longitud fija, rellena con espacios | Codigos fijos (ej. "PE", "US") |
| `VARCHAR(n)` | 65,535 caracteres (segun fila) | Longitud variable | Nombres, correos, textos cortos |
| `TEXT` | 65,535 caracteres | Para textos largos, no siempre indexable directo | Descripciones, comentarios largos |
| `ENUM(...)` *(extension MySQL)* | Lista fija de valores | Solo permite un valor de la lista definida | Categorias cerradas (ej. estado de un pedido) |
| `SET(...)` *(extension MySQL)* | Lista fija de valores | Permite multiples valores de la lista | Etiquetas o combinaciones (ej. idiomas hablados) |

### Tabla: tipos de fecha y hora

| Tipo | Formato | Rango aproximado | Uso tipico |
| :--- | :--- | :--- | :--- |
| `DATE` | YYYY-MM-DD | 1000-01-01 a 9999-12-31 | Fechas sin hora (nacimiento, vencimiento) |
| `DATETIME` | YYYY-MM-DD HH:MM:SS | 1000-01-01 a 9999-12-31 | Fecha y hora, sin depender de zona horaria |
| `TIMESTAMP` | YYYY-MM-DD HH:MM:SS | 1970-01-01 a 2038-01-19 (en MySQL) | Fecha y hora ligada a zona horaria (creado_en, actualizado_en) |
| `TIME` | HH:MM:SS | -838:59:59 a 838:59:59 | Duraciones u horarios |
| `YEAR` *(extension MySQL)* | YYYY | 1901 a 2155 | Solo el anio (ej. anio de fabricacion) |

## Sintaxis / codigo base

```sql
CREATE TABLE productos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio DECIMAL(10,2) NOT NULL,
    stock SMALLINT UNSIGNED DEFAULT 0,
    categoria ENUM('electronica', 'ropa', 'hogar') NOT NULL,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_vencimiento DATE
);
```

> **Nota:** `AUTO_INCREMENT` tambien es sintaxis propia de MySQL. El estandar SQL usa `GENERATED ALWAYS AS IDENTITY` (y PostgreSQL/SQL Server tienen su propia variante).

## Ejemplo aplicado

Caso: tabla de productos de una tienda online.

```sql
INSERT INTO productos (nombre, precio, stock, categoria, fecha_vencimiento)
VALUES ('Yogurt natural', 3.50, 120, 'hogar', '2026-10-01');
```

| Columna | Tipo elegido | Por que |
| :--- | :--- | :--- |
| `precio` | `DECIMAL(10,2)` | Nunca `FLOAT` en dinero: un error de redondeo ahi es un bug real |
| `stock` | `SMALLINT UNSIGNED` | No necesitas `INT` completo ni negativos, ahorras espacio |
| `categoria` | `ENUM(...)` | Restringe valores validos directamente en el esquema |

**Simulacion de `SELECT * FROM productos;`**

| id | nombre | precio | stock | categoria | creado_en | fecha_vencimiento |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Yogurt natural | 3.50 | 120 | hogar | 2026-09-18 10:02:31 | 2026-10-01 |
| 2 | Camisa manga larga | 45.00 | 30 | ropa | 2026-09-18 10:05:12 | NULL |
| 3 | Audifonos bluetooth | 89.90 | 15 | electronica | 2026-09-18 10:07:45 | NULL |

> **Nota:** `fecha_vencimiento` queda en `NULL` en los productos que no vencen: la columna no tiene `NOT NULL`, asi que SQL lo permite sin problema.

## Errores comunes 

- Usar `FLOAT` para precios y encontrarte con `9.999999999` en vez de `10.00`.
- Usar `VARCHAR(255)` para todo "por si acaso" sin pensar el limite real.
- Confundir `TIMESTAMP` y `DATETIME` en sistemas que manejan multiples zonas horarias.
- Olvidar `UNSIGNED` en columnas que nunca seran negativas (ids, contadores, stock) — ojo, `UNSIGNED` tambien es propio de MySQL.
- Usar `ENUM`/`SET` pensando que son estandar y encontrarte con que no existen al migrar a otro motor.

## Ejercicio

Tienes que guardar el precio de un producto con hasta 2 decimales exactos, y una columna de "talla" que solo pueda valer `S`, `M` o `L`.

¿Que tipo usarias para el precio, y que dos formas distintas tienes para restringir la columna de talla (una propia de MySQL y otra que funcione en cualquier motor)?

<details>
<summary>Ver respuesta</summary>

Para el precio: `DECIMAL(10,2)` (nunca `FLOAT`, por la precision exacta que exige el dinero).

Para la talla:
- Forma especifica de MySQL: `talla ENUM('S', 'M', 'L')`.
- Forma estandar (funciona en cualquier motor): `talla VARCHAR(1) CHECK (talla IN ('S', 'M', 'L'))`.

</details>

## Relacionado con
- [DDL, DML, DQL](ddl-dml-dql.md) — el `CREATE TABLE` de arriba es DDL puro.
- [Constraints](constraints.md) — `NOT NULL`, `DEFAULT`, `CHECK` y `PRIMARY KEY` son constraints, no tipos.
- Indices *(pendiente)* — el tipo de dato afecta el tamano y velocidad del indice.