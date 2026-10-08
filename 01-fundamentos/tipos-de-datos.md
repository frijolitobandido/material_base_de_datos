# Tipos de Datos en SQL

Esta guía explica cómo elegir y utilizar los **tipos de datos** en SQL. Los tipos de datos determinan qué clase de información puede almacenarse en una columna y cómo el motor de base de datos la interpreta.

Elegir un tipo adecuado ayuda a:

- Guardar información de manera correcta.
- Evitar valores inválidos.
- Usar el espacio de almacenamiento de forma eficiente.
- Realizar operaciones y búsquedas con mayor seguridad.
- Mantener coherencia entre las tablas.

> **Concepto clave.** El tipo de dato describe qué puede guardar una columna; no es lo mismo almacenar un número, una fecha, un texto o un valor verdadero/falso.

---

## 1. ¿Qué es un tipo de dato?

Un **tipo de dato** es una definición que indica qué clase de valores puede recibir una columna.

Por ejemplo:

| Información | Tipo apropiado | Ejemplo |
| :--- | :--- | :--- |
| Edad de una persona | `INT` | `25` |
| Precio de un producto | `DECIMAL(10,2)` | `149.90` |
| Nombre | `VARCHAR(100)` | `'Lucía'` |
| Fecha de nacimiento | `DATE` | `'2001-08-15'` |
| Estado de una reserva | `BOOLEAN` | `TRUE` |

Cuando se crea una tabla, cada columna debe tener un tipo de dato:

```sql
CREATE TABLE estudiantes (
    id INT,
    nombre VARCHAR(100),
    fecha_nacimiento DATE,
    promedio DECIMAL(4,2)
);
```

En este ejemplo:

| Columna | Tipo | Qué puede almacenar |
| :--- | :--- | :--- |
| `id` | `INT` | Un número entero |
| `nombre` | `VARCHAR(100)` | Texto de hasta 100 caracteres |
| `fecha_nacimiento` | `DATE` | Una fecha |
| `promedio` | `DECIMAL(4,2)` | Un número con dos decimales |

---

## 2. Familias principales de tipos de datos

Los tipos de datos más utilizados se pueden organizar en varias familias:

| Familia | Tipos frecuentes | ¿Es exacta? | Característica principal | Se utiliza para |
| :--- | :--- | :---: | :--- | :--- |
| Numéricos enteros | `TINYINT`, `SMALLINT`, `INT`, `BIGINT` | Sí | Números sin decimales | Edades, IDs y cantidades |
| Numéricos exactos | `DECIMAL`, `NUMERIC` | Sí | Controlan precisión y escala | Precios, dinero y montos |
| Numéricos aproximados | `FLOAT`, `DOUBLE` | No | Pueden tener pequeñas diferencias de precisión | Cálculos científicos |
| Texto | `CHAR`, `VARCHAR`, `TEXT` | — | Almacenan caracteres | Nombres, correos y descripciones |
| Fecha y hora | `DATE`, `TIME`, `DATETIME`, `TIMESTAMP` | — | Representan momentos o fechas | Reservas, eventos y vencimientos |
| Lógicos | `BOOLEAN` | — | Representan verdadero o falso | Estados y banderas |
| Binarios | `BINARY`, `VARBINARY`, `BLOB` | — | Almacenan bytes | Archivos o datos codificados |
| Estructurados | `JSON` | — | Guardan objetos y listas flexibles | Preferencias y configuraciones |

### Criterios para elegir un tipo

| Pregunta | Ejemplo de decisión |
| :--- | :--- |
| ¿El valor puede tener decimales? | Usa `DECIMAL` o `DOUBLE`, no `INT` |
| ¿La precisión debe ser exacta? | Usa `DECIMAL` para dinero |
| ¿El texto tiene longitud fija? | Usa `CHAR`; si varía, usa `VARCHAR` |
| ¿Se necesita calcular con fechas? | Usa `DATE`, `TIME` o `DATETIME`, no texto |
| ¿El valor tiene opciones limitadas? | Usa `CHECK`, `ENUM` o una tabla relacionada |
| ¿El dato debe funcionar en varios motores? | Prefiere tipos estándar y revisa las diferencias del SGBD |

> **Pista práctica.** Antes de elegir un tipo, pregunta qué representa el dato, qué operaciones se harán con él y cuál es su rango, tamaño y nivel de precisión esperado.

---

## 3. Tipos numéricos enteros

Los tipos enteros almacenan números sin parte decimal. Son útiles para identificadores, edades, cantidades y contadores.

### Tipos enteros comunes

| Tipo | Tamaño aproximado | Rango con signo | Rango sin signo | Uso habitual |
| :--- | :---: | :--- | :--- | :--- |
| `TINYINT` | 1 byte | -128 a 127 | 0 a 255 | Estados, edades o valores reducidos |
| `SMALLINT` | 2 bytes | -32.768 a 32.767 | 0 a 65.535 | Stock y contadores medianos |
| `INT` / `INTEGER` | 4 bytes | Aprox. -2,1 mil millones a 2,1 mil millones | Hasta aprox. 4,2 mil millones | IDs y cantidades generales |
| `BIGINT` | 8 bytes | Aprox. -9,2 × 10¹⁸ a 9,2 × 10¹⁸ | Rango positivo aún mayor | Sistemas masivos |

Los rangos pueden variar entre motores. `UNSIGNED` es una extensión frecuente de MySQL que elimina los valores negativos y amplía el rango positivo.

### Ejemplo

```sql
CREATE TABLE productos (
    id INT,
    stock INT,
    unidades_vendidas BIGINT,
    edad_garantia TINYINT
);
```

### Ejemplo de registros

| id | stock | unidades_vendidas | edad_garantia |
| ---: | ---: | ---: | ---: |
| 1 | 25 | 1250 | 2 |
| 2 | 8 | 580 | 1 |

### ¿Qué ocurre con los decimales?

```sql
INSERT INTO productos (id, stock)
VALUES (1, 12.5);
```

Una columna entera no es la opción adecuada para guardar `12.5`. Dependiendo del motor y de su configuración, el valor puede ser rechazado, redondeado o convertido.

> **Revisa antes de guardar.** No uses un tipo entero para precios, porcentajes o medidas que necesiten conservar decimales.

---

## 4. `DECIMAL` y `NUMERIC`: números exactos

`DECIMAL` y `NUMERIC` sirven para almacenar números con una cantidad exacta de decimales.

La sintaxis general es:

```sql
DECIMAL(precision, scale)
```

- **Precisión:** cantidad total de dígitos.
- **Escala:** cantidad de dígitos después del punto decimal.

### Ejemplo

```sql
precio DECIMAL(10,2)
```

Esto permite hasta 10 dígitos en total, incluyendo 2 decimales.

| Valor | ¿Es adecuado para `DECIMAL(10,2)`? | Qué ocurre |
| :--- | :---: | :--- |
| `25.50` | Sí | Conserva exactamente dos decimales |
| `1499.99` | Sí | Está dentro de la precisión definida |
| `10` | Sí | Puede mostrarse como `10.00` |
| `12.345` | Requiere revisión | Tiene más decimales que la escala definida; puede redondearse o rechazarse |
| `999999999.99` | Sí | Usa los 10 dígitos permitidos |
| `1000000000.00` | No | Supera la precisión total |

### Ejemplo aplicado

```sql
CREATE TABLE productos (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    precio DECIMAL(10,2)
);

INSERT INTO productos (id, nombre, precio)
VALUES (1, 'Teclado', 149.90);
```

> **No los confundas.** Para dinero normalmente conviene usar `DECIMAL`, no `FLOAT`, porque `DECIMAL` conserva una representación exacta.

---

## 5. `FLOAT` y `DOUBLE`: números aproximados

`FLOAT` y `DOUBLE` almacenan números de punto flotante. Pueden representar valores muy grandes o muy pequeños, pero algunas operaciones pueden producir pequeñas diferencias de precisión.

Son útiles para:

- Mediciones científicas.
- Cálculos estadísticos.
- Coordenadas.
- Valores aproximados.

```sql
CREATE TABLE mediciones (
    id INT PRIMARY KEY,
    temperatura FLOAT,
    distancia DOUBLE
);
```

### Comparación rápida

| Necesidad | Tipo recomendado | Razón |
| :--- | :--- | :--- |
| Precio de un producto | `DECIMAL(10,2)` | Se necesita exactitud |
| Temperatura de un sensor | `FLOAT` | Se acepta aproximación |
| Cálculo científico de alta precisión | `DOUBLE` | Tiene mayor capacidad que `FLOAT` |

---

## 6. Tipos de texto

Los tipos de texto permiten almacenar caracteres, palabras, frases y documentos.

### `CHAR(n)`

Guarda texto de longitud fija.

```sql
codigo CHAR(6)
```

Es apropiado cuando todos los valores tienen una longitud similar, como códigos de país o identificadores con formato fijo.

### `VARCHAR(n)`

Guarda texto de longitud variable hasta el límite indicado.

```sql
nombre VARCHAR(100)
```

Es una opción habitual para nombres, correos y direcciones.

### `TEXT`

Se utiliza para textos largos, como comentarios, descripciones extensas o artículos.

```sql
descripcion TEXT
```

### Comparación

| Tipo | Longitud | Característica | Ejemplo de uso |
| :--- | :--- | :--- | :--- |
| `CHAR(2)` | Fija | Puede rellenarse con espacios | Códigos como `'PE'` o `'MX'` |
| `VARCHAR(100)` | Variable, hasta 100 | Usa solo el espacio necesario dentro del límite | Nombres y correos |
| `TEXT` | Variable y extensa | Adecuado para textos largos; su indexación depende del motor | Descripciones y comentarios |

### Ejemplo

```sql
CREATE TABLE usuarios (
    id INT PRIMARY KEY,
    codigo_pais CHAR(2),
    nombre VARCHAR(100),
    biografia TEXT
);
```

> **Regla de elección.** Usa `VARCHAR` cuando conoces un límite razonable y `TEXT` cuando el contenido puede ser considerablemente más largo.

---

## 7. Fechas y horas

Los tipos de fecha y hora permiten guardar momentos, fechas de vencimiento y horarios.

| Tipo | Formato | Característica | Ejemplo |
| :--- | :--- | :--- | :--- |
| `DATE` | `AAAA-MM-DD` | Solo fecha; no guarda hora | `'2026-10-04'` |
| `TIME` | `HH:MM:SS` | Hora o duración, según el motor | `'17:45:00'` |
| `DATETIME` | `AAAA-MM-DD HH:MM:SS` | Fecha y hora sin conversión automática de zona horaria | `'2026-10-04 17:45:00'` |
| `TIMESTAMP` | `AAAA-MM-DD HH:MM:SS` | Puede depender de la zona horaria de la sesión | `'2026-10-04 17:45:00'` |
| `YEAR` | `AAAA` | Solo año; disponibilidad variable | `2026` |

### Ejemplo aplicado

```sql
CREATE TABLE reservas (
    id INT PRIMARY KEY,
    fecha_reserva DATE,
    hora_entrada TIME,
    creado_en DATETIME,
    actualizado_en TIMESTAMP
);
```

### Insertar fechas

```sql
INSERT INTO reservas (
    id,
    fecha_reserva,
    hora_entrada,
    creado_en
)
VALUES (
    1,
    '2026-10-04',
    '14:30:00',
    '2026-10-04 14:25:00'
);
```

### Formato recomendado

| Dato | Formato recomendado |
| :--- | :--- |
| Fecha | `AAAA-MM-DD` |
| Hora | `HH:MM:SS` |
| Fecha y hora | `AAAA-MM-DD HH:MM:SS` |

> **Evita ambigüedades.** Guarda las fechas en un formato estándar como `2026-10-04`, no en formatos ambiguos como `04/10/26`.

### Ejemplo integrador: tabla de productos

Ahora combinaremos varios tipos en una tabla de una tienda:

```sql
CREATE TABLE productos_tienda (
    id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio DECIMAL(10,2) NOT NULL,
    stock SMALLINT UNSIGNED DEFAULT 0,
    categoria VARCHAR(20),
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    fecha_vencimiento DATE
);
```

### ¿Por qué se eligió cada tipo?

| Columna | Tipo elegido | Característica aprovechada | Razón de uso |
| :--- | :--- | :--- | :--- |
| `id` | `INT` | Entero e identificador | No necesita decimales |
| `nombre` | `VARCHAR(100)` | Texto de longitud variable | Los nombres no tienen siempre la misma longitud |
| `precio` | `DECIMAL(10,2)` | Precisión exacta | Evita errores de redondeo en dinero |
| `stock` | `SMALLINT UNSIGNED` | Entero sin valores negativos | El inventario no puede ser menor que cero |
| `categoria` | `VARCHAR(20)` | Texto limitado | Permite guardar una categoría corta |
| `creado_en` | `TIMESTAMP` | Fecha y hora automática | Registra cuándo se creó el producto |
| `fecha_vencimiento` | `DATE` | Solo fecha | Algunos productos vencen y otros no |

### Insertar productos

```sql
INSERT INTO productos_tienda
    (id, nombre, precio, stock, categoria, fecha_vencimiento)
VALUES
    (1, 'Yogurt natural', 3.50, 120, 'alimentos', '2026-10-01'),
    (2, 'Camisa manga larga', 45.00, 30, 'ropa', NULL),
    (3, 'Audífonos bluetooth', 89.90, 15, 'electronica', NULL);
```

### Resultado simulado de `SELECT *`

| id | nombre | precio | stock | categoria | creado_en | fecha_vencimiento |
| :---: | :--- | ---: | ---: | :--- | :--- | :--- |
| 1 | Yogurt natural | 3.50 | 120 | alimentos | 2026-09-18 10:02:31 | 2026-10-01 |
| 2 | Camisa manga larga | 45.00 | 30 | ropa | 2026-09-18 10:05:12 | `NULL` |
| 3 | Audífonos bluetooth | 89.90 | 15 | electronica | 2026-09-18 10:07:45 | `NULL` |

En este ejemplo, `creado_en` se completa automáticamente gracias a `DEFAULT CURRENT_TIMESTAMP`. En cambio, `fecha_vencimiento` puede quedar en `NULL` porque no todos los productos tienen fecha de vencimiento.

> **Qué debes recordar.** Un tipo de dato no se elige de forma aislada: debe corresponder con las operaciones, límites, precisión y reglas del dato que representa.

---

## 8. `BOOLEAN`: verdadero o falso

`BOOLEAN` representa una condición lógica. En algunos motores, especialmente MySQL, se almacena internamente como un tipo numérico equivalente a `TINYINT(1)`.

```sql
CREATE TABLE tareas (
    id INT PRIMARY KEY,
    titulo VARCHAR(100),
    completada BOOLEAN DEFAULT FALSE
);
```

### Ejemplo de datos

| id | titulo | completada |
| ---: | :--- | :---: |
| 1 | Repasar SQL | `FALSE` |
| 2 | Resolver ejercicios | `TRUE` |

### Consultar valores booleanos

```sql
SELECT *
FROM tareas
WHERE completada = TRUE;
```

La consulta devuelve únicamente las tareas terminadas.

---

## 9. `ENUM`: conjunto limitado de opciones

`ENUM` permite definir una lista cerrada de valores posibles. Su disponibilidad y comportamiento varían entre motores.

```sql
CREATE TABLE pedidos (
    id INT PRIMARY KEY,
    estado ENUM('pendiente', 'enviado', 'entregado', 'cancelado')
);
```

### Valores permitidos

| Valor enviado | Resultado |
| :--- | :--- |
| `'pendiente'` | Aceptado |
| `'enviado'` | Aceptado |
| `'entregado'` | Aceptado |
| `'cancelado'` | Aceptado |
| `'devuelto'` | Rechazado o convertido según la configuración del motor |

Una alternativa más portable es utilizar `VARCHAR` junto con un `CHECK`:

```sql
estado VARCHAR(20)
    CHECK (estado IN ('pendiente', 'enviado', 'entregado', 'cancelado'))
```

> **Compatibilidad entre motores.** Si buscas que el diseño funcione en distintos SGBD, `VARCHAR` con `CHECK` suele ser una alternativa más portable que `ENUM`.

---

## 10. `JSON`: información estructurada y flexible

`JSON` permite almacenar objetos y listas con una estructura flexible.

```sql
CREATE TABLE configuraciones (
    id INT PRIMARY KEY,
    datos JSON
);
```

### Ejemplo de inserción

```sql
INSERT INTO configuraciones (id, datos)
VALUES (
    1,
    '{"tema":"oscuro", "idioma":"es", "notificaciones":true}'
);
```

### ¿Cuándo utilizarlo?

| Situación | ¿JSON es buena opción? |
| :--- | :---: |
| Preferencias variables de un usuario | Sí |
| Datos que siempre tienen las mismas columnas | No, suele ser mejor una tabla normal |
| Información que debe filtrarse y relacionarse constantemente | Generalmente no |
| Estructura flexible que cambia con frecuencia | Puede ser útil |

> **Usa con criterio.** JSON aporta flexibilidad, pero no debería reemplazar automáticamente un diseño relacional bien estructurado.

---

## 11. `BINARY`, `VARBINARY` y `BLOB`

Estos tipos se utilizan para datos binarios, es decir, información que no se interpreta directamente como texto.

| Tipo | Uso orientativo |
| :--- | :--- |
| `BINARY(n)` | Datos binarios de longitud fija |
| `VARBINARY(n)` | Datos binarios de longitud variable |
| `BLOB` | Archivos o datos binarios grandes |

Aunque es posible guardar imágenes o documentos dentro de una base de datos, muchas aplicaciones almacenan el archivo fuera de la base y guardan solo su ruta o URL.

```sql
CREATE TABLE archivos (
    id INT PRIMARY KEY,
    nombre VARCHAR(150),
    ruta VARCHAR(255),
    contenido BLOB
);
```

> **Decisión de diseño.** Antes de guardar archivos en un `BLOB`, considera el tamaño, las copias de seguridad y la forma en que la aplicación los descargará.

---

## 12. Combinar tipos de datos con constraints

Los tipos de datos indican qué clase de valor se acepta. Los constraints agregan reglas adicionales.

```sql
CREATE TABLE empleados (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    salario DECIMAL(10,2) CHECK (salario > 0),
    fecha_ingreso DATE NOT NULL,
    activo BOOLEAN DEFAULT TRUE
);
```

### ¿Qué protege cada definición?

| Columna | Tipo o constraint | Protección |
| :--- | :--- | :--- |
| `id` | `INT PRIMARY KEY` | Identificador único |
| `nombre` | `VARCHAR(100) NOT NULL` | Texto obligatorio |
| `salario` | `DECIMAL(10,2)` | Dinero con precisión decimal |
| `salario` | `CHECK (salario > 0)` | Impide salarios cero o negativos |
| `fecha_ingreso` | `DATE NOT NULL` | Fecha obligatoria |
| `activo` | `BOOLEAN DEFAULT TRUE` | Estado automático inicial |

### Registros de ejemplo

| nombre | salario | fecha_ingreso | activo | Resultado |
| :--- | ---: | :--- | :---: | :--- |
| Lucía | `2500.00` | `2026-01-15` | `TRUE` | Aceptado |
| Pedro | `0.00` | `2026-02-01` | `TRUE` | Rechazado por `CHECK` |
| Ana | `1800.50` | `NULL` | `TRUE` | Rechazado por `NOT NULL` |

---

## 13. Modificar el tipo de una columna

Si una tabla ya existe, es posible cambiar el tipo de una columna con `ALTER TABLE`. La sintaxis exacta puede variar según el motor.

### MySQL

```sql
ALTER TABLE productos
MODIFY COLUMN precio DECIMAL(10,2);
```

### PostgreSQL

```sql
ALTER TABLE productos
ALTER COLUMN precio TYPE NUMERIC(10,2);
```

Antes de modificar una columna, conviene revisar si los datos actuales pueden convertirse sin perder información.

| Dato actual | Nuevo tipo | ¿Qué revisar? |
| :--- | :--- | :--- |
| `VARCHAR` con números | `INT` | Que no existan textos no numéricos |
| `DECIMAL(10,2)` | `INT` | Se perderían los decimales |
| `VARCHAR` con fechas | `DATE` | Que todas tengan un formato válido |

> **Revisa antes de cambiar.** Modificar el tipo de una columna puede producir errores o pérdida de información si los datos existentes no son compatibles.

---

## 14. Errores comunes al elegir tipos

### Error 1: guardar números como texto

```sql
precio VARCHAR(20)
```

Esto dificulta ordenar y calcular precios correctamente.

```sql
-- Mejor alternativa
precio DECIMAL(10,2)
```

### Error 2: usar `FLOAT` para dinero

Las aproximaciones de `FLOAT` pueden generar diferencias inesperadas en operaciones financieras.

```sql
-- Preferible para dinero
saldo DECIMAL(12,2)
```

### Error 3: elegir un tamaño insuficiente

```sql
nombre VARCHAR(10)
```

Un nombre como `'Alejandra María'` puede superar ese límite. Conviene elegir un tamaño razonable de acuerdo con el dominio del dato.

### Error 4: almacenar fechas como texto

```sql
fecha VARCHAR(20)
```

Así se dificultan las comparaciones, ordenamientos y cálculos de fechas.

```sql
-- Mejor alternativa
fecha DATE
```

### Error 5: utilizar `TEXT` para cualquier texto

`TEXT` no siempre ofrece las mismas posibilidades de indexación o restricciones que `VARCHAR`. Si el contenido tiene un límite razonable, `VARCHAR` suele expresar mejor la intención.

### Tabla de diagnóstico rápido

| Problema | Tipo poco conveniente | Tipo recomendado |
| :--- | :--- | :--- |
| Precio guardado como texto | `VARCHAR` | `DECIMAL` |
| Fecha guardada como texto | `VARCHAR` | `DATE` o `DATETIME` |
| Nombre con tamaño fijo innecesario | `CHAR(100)` | `VARCHAR(100)` |
| Estado sin valores controlados | `VARCHAR` libre | `CHECK` o `ENUM` |
| Número con decimales en columna entera | `INT` | `DECIMAL` o `DOUBLE` |

---

## 15. Práctica guiada

### Pregunta 1: precio de un producto

Quieres almacenar precios como `19.99`, `250.00` y `1499.95`.

¿Qué tipo elegirías?

<details>
<summary>Ver respuesta</summary>

```sql
DECIMAL(10,2)
```

Se necesita conservar exactamente dos decimales.

</details>

### Pregunta 2: fecha de nacimiento

Quieres guardar únicamente la fecha de nacimiento, sin hora.

<details>
<summary>Ver respuesta</summary>

```sql
DATE
```

</details>

### Pregunta 3: estado de una tarea

Una tarea solo puede estar terminada o pendiente.

¿Qué tipo y restricción podrías utilizar?

<details>
<summary>Ver respuesta</summary>

Una opción sería:

```sql
completada BOOLEAN DEFAULT FALSE
```

Otra opción, si se desea guardar texto, sería:

```sql
estado VARCHAR(10)
    CHECK (estado IN ('pendiente', 'terminada'))
```

</details>

### Pregunta 4: descripción extensa

Quieres almacenar comentarios que pueden tener varios miles de caracteres.

<details>
<summary>Ver respuesta</summary>

```sql
TEXT
```

</details>

---

## 16. Ejercicio

Crea una tabla `cursos` que cumpla estas condiciones:

1. `id` debe ser un identificador entero y único.
2. `nombre` debe almacenar texto obligatorio.
3. `precio` debe aceptar dos decimales y ser mayor que cero.
4. `fecha_inicio` debe almacenar una fecha obligatoria.
5. `activo` debe comenzar con `TRUE` automáticamente.
6. `descripcion` debe permitir texto largo.

### Una solución posible

```sql
CREATE TABLE cursos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(120) NOT NULL,
    precio DECIMAL(10,2) CHECK (precio > 0),
    fecha_inicio DATE NOT NULL,
    activo BOOLEAN DEFAULT TRUE,
    descripcion TEXT
);
```

### Verificación rápida

| Registro | nombre | precio | fecha_inicio | activo | Resultado |
| :---: | :--- | ---: | :--- | :---: | :--- |
| 1 | SQL básico | `99.90` | `2026-11-01` | `TRUE` | Aceptado |
| 2 | SQL avanzado | `0.00` | `2026-11-05` | `TRUE` | Rechazado por `CHECK` |
| 3 | Bases de datos | `150.00` | `NULL` | `TRUE` | Rechazado por `NOT NULL` |

```sql
-- Funciona
INSERT INTO cursos (nombre, precio, fecha_inicio, descripcion)
VALUES ('SQL básico', 99.90, '2026-11-01', 'Curso introductorio de SQL');

-- Falla por precio inválido
INSERT INTO cursos (nombre, precio, fecha_inicio)
VALUES ('SQL avanzado', 0.00, '2026-11-05');

-- Falla por fecha faltante
INSERT INTO cursos (nombre, precio)
VALUES ('Bases de datos', 150.00);
```

---

## 17. Resumen final

- El tipo de dato define qué clase de información puede guardar una columna.
- `INT` sirve para números enteros.
- `DECIMAL` es apropiado para valores exactos, especialmente dinero.
- `FLOAT` y `DOUBLE` se utilizan para valores aproximados.
- `CHAR`, `VARCHAR` y `TEXT` almacenan texto con diferentes características.
- `DATE`, `TIME`, `DATETIME` y `TIMESTAMP` representan fechas y horas.
- `BOOLEAN` representa estados verdadero/falso.
- `JSON` permite guardar estructuras flexibles, pero no debe reemplazar sin motivo a las tablas relacionales.
- Los tipos de datos y los constraints trabajan juntos para proteger la información.
- Antes de modificar un tipo, hay que revisar los datos que ya existen.

> **Qué debes recordar.** Elegir correctamente los tipos de datos desde el principio evita conversiones problemáticas, consultas confusas y errores en la información almacenada.

---

## Relacionado con otros temas

- [DDL, DML y DQL](ddl-dml-dql.md): los tipos de datos se definen al crear y modificar tablas.
- [Constraints](constraints.md): agregan reglas como `NOT NULL`, `UNIQUE` y `CHECK`.
- [Normalización](../05-diseno/normalizacion.md): ayuda a organizar correctamente la información.
- [Índices básicos](../06-indices-performance/indices-basicos.md): algunos tipos y tamaños afectan el almacenamiento y el rendimiento.
