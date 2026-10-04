# SQL Notes — Repaso activo y asociativo

Repositorio personal para repasar **SQL** de forma constante: teoria corta, sintaxis, ejemplos aplicados y ejercicios.

El contenido se centra en SQL estandar (ANSI SQL), que es el que comparten la mayoria de motores (PostgreSQL, SQL Server, Oracle, SQLite, etc.). Los ejemplos se ejecutan y verifican sobre **MySQL**, asi que cuando algo es una extension o un comportamiento propio de MySQL (y no del estandar), se marca explicitamente con una nota. De esta forma lo aprendido sirve mas alla de un solo motor.

## Mapa general

| Carpeta | Contenido |
| :--- | :--- |
| `01-fundamentos/` | Tipos de datos, DDL/DML/DQL, constraints |
| `02-consultas/` | Joins, subqueries, agregaciones, window functions |
| `03-indices-performance/` | Indices, EXPLAIN, optimizacion de queries |
| `04-transacciones/` | ACID, isolation levels, locks |
| `05-diseno/` | Normalizacion, modelado ER |
| `ejercicios/` | Banco adicional de preguntas de repaso |
| `errores-que-cometi.md` | Bugs y confusiones reales (el mejor material de repaso) |

## Indice por tema

### 01 - Fundamentos
| Archivo | Estado |
| :--- | :--- |
| [Tipos de datos](01-fundamentos/tipos-de-datos.md) | Listo |
| [DDL, DML, DQL](01-fundamentos/ddl-dml-dql.md) | Listo |
| [Constraints](01-fundamentos/constraints.md) | Listo |

### 02 - Consultas
| Archivo | Estado |
| :--- | :--- |
| Joins | Pendiente |
| Subqueries | Pendiente |
| Agregaciones y GROUP BY | Pendiente |
| Window functions | Pendiente |

### 03 - Indices y performance
| Archivo | Estado |
| :--- | :--- |
| Indices basicos | Pendiente |
| EXPLAIN / ANALYZE | Pendiente |
| Optimizacion de queries | Pendiente |

### 04 - Transacciones
| Archivo | Estado |
| :--- | :--- |
| ACID | Pendiente |
| Isolation levels | Pendiente |
| Locks | Pendiente |

### 05 - Diseno
| Archivo | Estado |
| :--- | :--- |
| Normalizacion | Pendiente |
| Modelado ER | Pendiente |

## Como se conectan los temas (vista rapida)

| Tema origen | Se conecta con | Por que |
| :--- | :--- | :--- |
| Tipos de datos | Indices | El tipo de columna afecta tamano y velocidad del indice |
| Constraints (FK) | Joins, Transacciones | Determinan integridad referencial y comportamiento en relaciones |
| Normalizacion | Joins | Mas normalizacion suele significar mas joins al consultar |
| Indices | EXPLAIN | El plan de ejecucion cambia segun los indices disponibles |

Cada archivo tiene, al final, una seccion "Relacionado con" que apunta a estos vinculos, para que al repasar uno termines revisando dos o tres mas sin darte cuenta.

## Plantilla usada en cada archivo de teoria

Cada archivo sigue el mismo orden, pensado para leerse de corrido: primero se entiende el concepto, luego se ve como se escribe, despues se aplica a un caso real, se revisan los errores tipicos, y se cierra con un ejercicio para comprobar si quedo claro.

```markdown
# Titulo del tema

## Teoria

## Sintaxis / codigo base

## Ejemplo aplicado

## Errores comunes

## Ejercicio

## Relacionado con
```

## Como repasar (sugerencia)

1. Abre el README y elige un tema al azar (no en orden).
2. Lee la Teoria y la Sintaxis, y antes de ver el Ejemplo intenta escribir tu propia consulta.
3. Revisa el Ejemplo aplicado y los Errores comunes.
4. Resuelve el Ejercicio del final sin ver la respuesta primero.
5. Sigue al menos un link de "Relacionado con".
6. Si te equivocaste en algo real (proyecto, practica), anotalo en `errores-que-cometi.md`.
