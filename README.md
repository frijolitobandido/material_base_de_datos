# Guía de Bases de Datos — Teoría, Repaso y Práctica

Repositorio personal para estudiar **bases de datos y SQL** mediante teoría, tablas comparativas, ejemplos aplicados, resultados simulados y ejercicios guiados.

La guía está organizada de forma progresiva: primero se estudian los fundamentos, luego las consultas, el diseño, el rendimiento, las transacciones y finalmente la seguridad y la práctica.

Los ejemplos se basan principalmente en SQL estándar. Cuando una característica depende de un motor específico, como MySQL, se indicará dentro de la explicación.

**Fecha de actualización:** 4 de octubre de 2026

---

## Mapa general de la guía

| Carpeta | Contenido |
| :--- | :--- |
| `01-fundamentos/` | Pendiente |
| `02-consultas-basicas/` | Pendiente |
| `03-consultas-relacionales/` | Pendiente |
| `04-diseno-modelado/` | Pendiente |
| `05-vistas-programabilidad/` | Pendiente |
| `06-indices-performance/` | Pendiente |
| `07-transacciones-concurrencia/` | Pendiente |
| `08-seguridad-administracion/` | Pendiente |
| `09-practica/` | Pendiente |
| `recursos/` | Pendiente |

---

## Índice por tema

### 01 — Fundamentos

| Archivo | Contenido |
| :--- | :--- |
| [Tipos de datos](01-fundamentos/tipos-de-datos.md) | Tipos numéricos, texto, fechas, booleanos, JSON y binarios |
| [DDL, DML y DQL](01-fundamentos/ddl-dml-dql.md) | Definir estructuras, manipular registros y consultar datos |
| `01-fundamentos/constraints.md` | Pendiente |
| `01-fundamentos/claves-primarias-y-foraneas.md` | Pendiente |
| `01-fundamentos/null-y-valores-por-defecto.md` | Pendiente |

### 02 — Consultas básicas

| Archivo | Contenido |
| :--- | :--- |
| `02-consultas-basicas/select-from.md` | Pendiente |
| `02-consultas-basicas/where-operadores.md` | Pendiente |
| `02-consultas-basicas/order-by-limit.md` | Pendiente |
| `02-consultas-basicas/funciones-sql.md` | Pendiente |
| `02-consultas-basicas/case-when.md` | Pendiente |

### 03 — Consultas relacionales y avanzadas

| Archivo | Contenido |
| :--- | :--- |
| `03-consultas-relacionales/joins.md` | Pendiente |
| `03-consultas-relacionales/subqueries.md` | Pendiente |
| `03-consultas-relacionales/cte-with.md` | Pendiente |
| `03-consultas-relacionales/agregaciones-group-by.md` | Pendiente |
| `03-consultas-relacionales/having.md` | Pendiente |
| `03-consultas-relacionales/window-functions.md` | Pendiente |
| `03-consultas-relacionales/union-intersect-except.md` | Pendiente |

### 04 — Diseño y modelado

| Archivo | Contenido |
| :--- | :--- |
| `04-diseno-modelado/normalizacion.md` | Pendiente |
| `04-diseno-modelado/desnormalizacion.md` | Pendiente |
| `04-diseno-modelado/modelado-er.md` | Pendiente |
| `04-diseno-modelado/relaciones-entre-tablas.md` | Pendiente |
| `04-diseno-modelado/reglas-de-integridad.md` | Pendiente |

### 05 — Vistas y programabilidad

| Archivo | Contenido |
| :--- | :--- |
| `05-vistas-programabilidad/views.md` | Pendiente |
| `05-vistas-programabilidad/stored-procedures.md` | Pendiente |
| `05-vistas-programabilidad/funciones-sql.md` | Pendiente |
| `05-vistas-programabilidad/triggers.md` | Pendiente |

### 06 — Índices y rendimiento

| Archivo | Contenido |
| :--- | :--- |
| `06-indices-performance/indices-basicos.md` | Pendiente |
| `06-indices-performance/indices-compuestos.md` | Pendiente |
| `06-indices-performance/explain-analyze.md` | Pendiente |
| `06-indices-performance/query-optimization.md` | Pendiente |
| `06-indices-performance/problemas-n-plus-one.md` | Pendiente |

### 07 — Transacciones y concurrencia

| Archivo | Contenido |
| :--- | :--- |
| `07-transacciones-concurrencia/acid.md` | Pendiente |
| `07-transacciones-concurrencia/commit-rollback.md` | Pendiente |
| `07-transacciones-concurrencia/isolation-levels.md` | Pendiente |
| `07-transacciones-concurrencia/locks.md` | Pendiente |
| `07-transacciones-concurrencia/deadlocks.md` | Pendiente |

### 08 — Seguridad y administración

| Archivo | Contenido |
| :--- | :--- |
| `08-seguridad-administracion/usuarios-roles-permisos.md` | Pendiente |
| `08-seguridad-administracion/sql-injection.md` | Pendiente |
| `08-seguridad-administracion/copias-de-seguridad.md` | Pendiente |
| `08-seguridad-administracion/restauracion.md` | Pendiente |
| `08-seguridad-administracion/auditoria.md` | Pendiente |

### 09 — Práctica

| Archivo | Contenido |
| :--- | :--- |
| `09-practica/ejercicios-basicos.md` | Pendiente |
| `09-practica/ejercicios-intermedios.md` | Pendiente |
| `09-practica/ejercicios-avanzados.md` | Pendiente |
| `09-practica/casos-practicos.md` | Pendiente |
| `09-practica/soluciones.md` | Pendiente |

---

## Recursos

| Archivo | Contenido |
| :--- | :--- |
| `recursos/glosario.md` | Pendiente |
| `recursos/errores-comunes.md` | Pendiente |
| `recursos/comandos-consulta-rapida.md` | Pendiente |
| `recursos/diferencias-motores.md` | Pendiente |

---

## Cómo se conectan los temas

| Tema de partida | Se conecta con | Motivo |
| :--- | :--- | :--- |
| Tipos de datos | Constraints e índices | El tipo define el valor permitido y afecta el almacenamiento |
| DDL, DML y DQL | Todos los temas | DDL crea estructuras, DML modifica registros y DQL consulta |
| Constraints | `JOIN` y transacciones | Mantienen relaciones válidas entre tablas |
| Normalización | `JOIN` | Separar la información suele requerir combinar tablas |
| Índices | `EXPLAIN` | El plan de ejecución muestra si se aprovechan los índices |
| DML | Transacciones | `INSERT`, `UPDATE` y `DELETE` suelen formar parte de transacciones |
| Diseño | Performance | Una estructura correcta facilita consultas eficientes |

---

## Formato de cada guía

Cada archivo se elaborará como material de teoría y repaso, no como informe académico. La estructura prevista es:

```markdown
# Guía de Teoría y Repaso: Tema

## Explicación inicial

## Conceptos principales

## Tablas comparativas

## Sintaxis y código base

## Ejemplos aplicados

## Resultados simulados

## Errores comunes

## Práctica guiada

## Mini desafío final

## Resumen

## Relacionado con otros temas
```

Las tablas deben mostrar no solo los nombres de los conceptos, sino también sus características, ventajas, limitaciones, usos y resultados cuando sea necesario.

---

## Ruta sugerida de aprendizaje

1. [Tipos de datos](01-fundamentos/tipos-de-datos.md)
2. [DDL, DML y DQL](01-fundamentos/ddl-dml-dql.md)
3. Constraints
4. Consultas básicas
5. `JOIN` y subconsultas
6. Agregaciones y funciones de ventana
7. Diseño y normalización
8. Vistas y programabilidad
9. Índices y optimización
10. Transacciones y concurrencia
11. Seguridad y respaldos
12. Ejercicios y casos prácticos

> **Método de repaso.** Lee la teoría, observa las tablas, intenta escribir el código antes de mirar la solución y comprueba qué cambia en la estructura, en los registros o en el resultado de la consulta.
