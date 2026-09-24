# Examen #1: Diseño de Base de Datos - Tienda de Libros

Este repositorio contiene la solución completa para el examen de modelado y diseño de bases de datos de Campuslands, abarcando el proceso de normalización paso a paso, el Diagrama Conceptual Entidad-Relación y el Esquema Lógico UML.

---

## Parte 1: Proceso de Normalización (0FN a 3FN)

### Tabla Inicial (No Normalizada - 0FN)
La siguiente tabla representa los datos transaccionales brutos proporcionados para el inventario, ventas y clientes:

| ISBN | Título | Autor | Fecha Publicación | Editorial | Categoría | Precio | Stock | Cliente | Correo Cliente | Dirección Cliente | Teléfono Cliente | Método Pago | Monto |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 978-0-14-118280-3 | El Principito | Antoine de Saint-Exupéry | 1943-04-06 | Gallimard | Infantil | 10.00 | 50 | Juan Pérez | juan.perez@email.com | Calle Falsa 123 | 3001234567 | Tarjeta Crédito | 10.00 |
| 978-0-14-044913-6 | Orgullo y Prejuicio | Jane Austen | 1813-01-28 | Penguin | Romance | 15.00 | 30 | María García | maria.garcia@email.com | Avenida Siempre Viva 456 | 3109876543 | Nequi | 15.00 |
| 978-0-06-085052-4 | 1984 | George Orwell | 1949-06-08 | Signet Classics | Ciencia Ficción | 20.00 | 20 | Juan Pérez | juan.perez@email.com | Calle Falsa 123 | 3001234567 | Tarjeta Crédito | 20.00 |

---

### Primera Forma Normal (1FN)
**Regla:** Eliminar grupos repetitivos, garantizar atomicidad en cada campo y definir una clave primaria.

* **Acción:** Se separan las líneas de detalle de las transacciones y se asignan identificadores únicos a los clientes y transacciones.
* **Resultado 1FN:**
  * **CLIENTE_TRANSACCION:** (`id_transaccion`, `id_cliente`, `nombre_cliente`, `correo`, `direccion`, `telefono`, `metodo_pago`, `fecha_transaccion`, `monto_total`)
  * **DETALLE_LIBRO:** (`id_transaccion`, `isbn`, `titulo`, `autor`, `fecha_publicacion`, `editorial`, `categoria`, `precio`, `stock`, `cantidad`, `precio_unitario`)

---

### Segunda Forma Normal (2FN)
**Regla:** Estar en 1FN y eliminar dependencias parciales (cada atributo no clave debe depender totalmente de la clave primaria completa).

* **Acción:** Se independizan las entidades relativas a Libros y Detalles de Transacción para evitar la redundancia de datos del catálogo en cada venta.
* **Resultado 2FN:**
  * **CLIENTE:** (`id_cliente`, `nombre_cliente`, `correo`, `direccion`, `telefono`)
  * **LIBRO:** (`isbn`, `titulo`, `autor`, `fecha_publicacion`, `editorial`, `categoria`, `precio`, `stock`)
  * **TRANSACCION:** (`id_transaccion`, `id_cliente`, `metodo_pago`, `fecha_transaccion`, `monto_total`)
  * **DETALLE_TRANSACCION:** (`id_detalle`, `id_transaccion`, `isbn`, `cantidad`, `precio_unitario`)

---

### Tercera Forma Normal (3FN)
**Regla:** Estar en 2FN y eliminar dependencias transitivas (ningún atributo no clave debe depender de otro atributo no clave).

* **Acción:** Se independizan los catálogos de `AUTOR`, `EDITORIAL`, `CATEGORIA` y `METODO_PAGO` para evitar inconsistencias al actualizar información externa al libro o transacción. Para soportar autores múltiples por libro de forma escalable, se implementa la entidad intermedia `LIBRO_AUTOR`.
* **Resultado Final (3FN):**
  1. **CLIENTE:** (`id_cliente` [PK], `nombre`, `correo`, `direccion`, `telefono`)
  2. **METODO_PAGO:** (`id_metodo_pago` [PK], `nombre_metodo`)
  3. **TRANSACCION:** (`id_transaccion` [PK], `fecha_transaccion`, `monto_total`, `id_cliente` [FK], `id_metodo_pago` [FK])
  4. **DETALLE_TRANSACCION:** (`id_detalle` [PK], `cantidad`, `precio_unitario`, `id_transaccion` [FK], `isbn` [FK])
  5. **LIBRO:** (`isbn` [PK], `titulo`, `fecha_publicacion`, `precio`, `stock`, `id_editorial` [FK], `id_categoria` [FK])
  6. **EDITORIAL:** (`id_editorial` [PK], `nombre_editorial`)
  7. **CATEGORIA:** (`id_categoria` [PK], `nombre_categoria`)
  8. **AUTOR:** (`id_autor` [PK], `nombre_autor`)
  9. **LIBRO_AUTOR:** (`isbn` [FK], `id_autor` [FK])

---

## Parte 2: Diagrama Conceptual de Entidad-Relación (E-R)

El modelo conceptual define el dominio del negocio mediante entidades (rectángulos), relaciones (rombos) y sus respectivas cardinalidades:

* **CLIENTE** `(1)` --- `<realiza>` --- `(N)` **TRANSACCION**
* **TRANSACCION** `(N)` --- `<paga_con>` --- `(1)` **METODO_PAGO**
* **TRANSACCION** `(1)` --- `<contiene>` --- `(N)` **DETALLE_TRANSACCION**
* **DETALLE_TRANSACCION** `(N)` --- `<incluye>` --- `(1)` **LIBRO**
* **LIBRO** `(N)` --- `<pertenece_a>` --- `(1)` **EDITORIAL**
* **LIBRO** `(N)` --- `<se_clasifica_en>` --- `(1)` **CATEGORIA**
* **LIBRO** `(N)` --- `<escrito_por>` --- `(M)` **AUTOR** *(resuelto mediante entidad asociativa en modelo lógico)*

---

## Parte 3: Esquema UML E-R (Modelo Lógico Final)

A continuación se muestra la estructura técnica final de las tablas con sus tipos de datos, claves primarias (PK) y claves foráneas (FK):

```mermaid
erDiagram
    CLIENTE {
        int id_cliente PK
        string nombre
        string correo
        string direccion
        string telefono
    }

    TRANSACCION {
        int id_transaccion PK
        datetime fecha_transaccion
        decimal monto_total
        int id_cliente FK
        int id_metodo_pago FK
    }

    METODO_PAGO {
        int id_metodo_pago PK
        string nombre_metodo
    }

    DETALLE_TRANSACCION {
        int id_detalle PK
        int id_transaccion FK
        string isbn FK
        int cantidad
        decimal precio_unitario
    }

    LIBRO {
        string isbn PK
        string titulo
        date fecha_publicacion
        decimal precio
        int stock
        int id_editorial FK
        int id_categoria FK
    }

    EDITORIAL {
        int id_editorial PK
        string nombre_editorial
    }

    CATEGORIA {
        int id_categoria PK
        string nombre_categoria
    }

    AUTOR {
        int id_autor PK
        string nombre_autor
    }

    LIBRO_AUTOR {
        string isbn FK
        int id_autor FK
    }

    CLIENTE ||--o{ TRANSACCION : realiza
    METODO_PAGO ||--o{ TRANSACCION : utilizado_en
    TRANSACCION ||--|{ DETALLE_TRANSACCION : contiene
    LIBRO ||--o{ DETALLE_TRANSACCION : incluido_en
    EDITORIAL ||--o{ LIBRO : publica
    CATEGORIA ||--o{ LIBRO : clasifica
    LIBRO ||--|{ LIBRO_AUTOR : asignado
    AUTOR ||--|{ LIBRO_AUTOR : escribe
