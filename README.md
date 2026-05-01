# README — Sistema de Gestión de Firma Jurídica

Base de datos relacional para gestionar clientes, asuntos legales y procuradores de una firma jurídica.

---

## Estructura del repositorio

```
/
├── diagramas/
│   └── DER_firma_juridica.png          ← Diagrama Entidad-Relación (modelo conceptual)
├── modelo_logico/
│   └── tablas.md                       ← Tablas del modelo relacional con PK y FK
│   └── tablas.png
└── README.md
```

---

## Interpretación del problema

Una firma jurídica necesita registrar:

- **Clientes** a quienes presta servicios legales.
- **Asuntos legales** que gestiona para esos clientes, con su historial y estado.
- **Procuradores** que trabajan en la firma y son asignados a los asuntos.

El sistema debe permitir saber qué asuntos tiene cada cliente, en qué estado se encuentran, y qué procuradores están trabajando en cada uno.

---

## Modelo conceptual — Entidades y atributos    

### CLIENTE
Representa a una persona física que contrata los servicios de la firma.

| Atributo          | Descripción                            |
|-------------------|----------------------------------------|
| DNI               | Identificador único del cliente (PK)   |
| nombre            | Nombre completo                        |
| direccion         | Domicilio (opcional)                   |
| fecha_nacimiento  | Fecha de nacimiento (opcional)         |

### ASUNTO
Representa un caso o expediente legal gestionado por la firma.

| Atributo         | Descripción                                                 |
|------------------|-------------------------------------------------------------|
| num_expediente   | Número único del expediente (PK)                            |
| fecha_inicio     | Fecha en que se abrió el caso                               |
| fecha_fin        | Fecha de cierre — puede estar vacía si el caso sigue abierto |
| estado           | Estado actual: `en proceso`, `cerrado` o `suspendido`       |
| DNI_cliente      | Referencia al cliente propietario del asunto (FK → CLIENTE) |

### PROCURADOR
Profesional colegiado que lleva los asuntos legales.

| Atributo       | Descripción                                       |
|----------------|---------------------------------------------------|
| DNI            | Identificador único del procurador (PK)           |
| nombre         | Nombre                                            |
| apellidos      | Apellidos                                         |
| num_colegiado  | Número de colegiatura (único por procurador)      |
| casos_ganados  | Contador de casos ganados                         |

---

## Relaciones y cardinalidades

### CLIENTE — ASUNTO (1:N)

Un cliente puede tener muchos asuntos a lo largo del tiempo, pero cada asunto pertenece a un único cliente. La cardinalidad es **1:N**.

- **Participación de ASUNTO**: total — todo asunto debe pertenecer a un cliente.
- **Participación de CLIENTE**: parcial — puede existir un cliente sin asuntos registrados todavía.

Se resuelve añadiendo la clave foránea `DNI_cliente` dentro de la tabla `ASUNTO`.

### ASUNTO — PROCURADOR (N:M)

Un asunto puede ser llevado por varios procuradores, y un procurador puede encargarse de varios asuntos. La cardinalidad es **N:M**.

- **Participación de ASUNTO**: parcial — un asunto podría no tener procurador asignado todavía.
- **Participación de PROCURADOR**: parcial — un procurador puede estar registrado sin asuntos activos.

Esta relación no puede resolverse con una simple clave foránea. Se crea una tabla intermedia llamada `ASUNTO_PROCURADOR`.

---

## Transformación al modelo relacional

1. **Cada entidad → una tabla.**  
   `CLIENTE`, `ASUNTO` y `PROCURADOR` se convierten directamente en tablas, usando sus identificadores únicos como claves primarias.

2. **Relación 1:N (CLIENTE–ASUNTO) → clave foránea.**  
   La PK de `CLIENTE` (`DNI`) se añade como atributo foráneo en `ASUNTO` (`DNI_cliente`), indicando a qué cliente pertenece cada asunto.

3. **Relación N:M (ASUNTO–PROCURADOR) → tabla intermedia.**  
   Se crea la tabla `ASUNTO_PROCURADOR` con clave primaria compuesta por `num_expediente` (FK → `ASUNTO`) y `DNI_procurador` (FK → `PROCURADOR`). Esto evita duplicados y permite en el futuro añadir atributos propios de la relación, como una fecha de asignación.

---

## Decisiones de diseño

| Decisión | Justificación |
|---|---|
| DNI como PK de CLIENTE y PROCURADOR | El enunciado lo indica explícitamente; es un identificador natural único |
| `fecha_fin` puede estar vacía | Un asunto en curso todavía no tiene fecha de cierre |
| `casos_ganados` inicia en 0 | Al registrar un procurador nuevo, aún no tiene casos ganados |
| `num_colegiado` es único | Cada número de colegiatura identifica a un único profesional |
| Tabla intermedia ASUNTO_PROCURADOR | Es la única forma correcta de representar una relación N:M en el modelo relacional |

---

## Diagramas

- `diagramas/DER_firma_juridica.png` — Diagrama Entidad-Relación con entidades, atributos y relaciones.
- `modelo_logico/tablas.md` — Representación de las tablas resultantes con sus claves primarias (PK) y foráneas (FK).