Tienda de Libros — Normalización y Diseño de Bases de Datos

Nelly Yulied Benítez Martínez  
Módulo de Desarrollo Backend / Bases de Datos  
24 de septiembre de 2026  

---

## Parte 1: Proceso de Normalización Paso a Paso

### 0. Tabla Inicial Desnormalizada (Non-Normal Form - NNF)
* **Atributos:** `ISBN`, `Titulo`, `Autor`, `Fecha_Publicacion`, `Editorial`, `Categoria`, `Precio`, `Stock`, `Cliente`, `Correo_Cliente`, `Direccion_Cliente`, `Telefono_Cliente`, `Metodo_Pago`, `Monto_Pago`.
* **Anomalías detectadas:** Grupos de datos repetidos, atributos no atómicos, redundancia de datos y riesgo de anomalías de actualización, inserción y borrado.

---

### 1. Primera Forma Normal (1FN) — Valores Atómicos
* **Regla aplicada:** Cada columna debe contener un solo valor atómico (indivisible) y no deben existir grupos de datos repetidos.
* **Diagnóstico de violaciones:** 
  * La información de la transacción, el cliente y el libro estaban agrupados en una sola celda/fila sin atomicidad.
* **Solución:** Se define la clave primaria compuesta (`id_transaccion`, `isbn`) y se separan los campos.

**Esquema en 1FN:**
`TRANSACCION_UNIFICADA` (<u>id_transaccion</u>, <u>isbn</u>, titulo, autor, fecha_publicacion, editorial, categoria, precio, stock, cliente_nombre, correo_cliente, direccion_cliente, telefono_cliente, metodo_pago, monto_pago)

---

### 2. Segunda Forma Normal (2FN) — Eliminación de Dependencias Parciales
* **Regla aplicada:** La tabla debe estar en 1FN y cada atributo no clave debe depender de la totalidad de la clave primaria compuesta, no de una parte.
* **Diagnóstico de dependencias parciales:**
  * `titulo`, `autor`, `fecha_publicacion`, `editorial`, `categoria`, `precio`, `stock` dependen **únicamente de `isbn`** (mitad de la clave).
  * `cliente_nombre`, `direccion_cliente`, `telefono_cliente` dependen **únicamente de `correo_cliente`**.
* **Solución:** Se divide la tabla moviendo los atributos a la entidad que los determina.

**Esquemas en 2FN:**
* `LIBRO_2FN` (<u>isbn</u>, titulo, autor, fecha_publicacion, editorial, categoria, precio, stock)
* `CLIENTE_2FN` (<u>correo_cliente</u>, cliente_nombre, direccion_cliente, telefono_cliente)
* `TRANSACCION_2FN` (<u>id_transaccion</u>, *correo_cliente*, metodo_pago, monto_pago, fecha_transaccion)
* `DETALLE_TRANSACCION_2FN` (<u>id_transaccion</u>, <u>isbn</u>, cantidad, precio_unitario)

---

### 3. Tercera Forma Normal (3FN) — Eliminación de Dependencias Transitivas
* **Regla aplicada:** La tabla debe estar en 2FN y ningún atributo no clave debe depender transitivamente de la clave primaria ($x \rightarrow y \rightarrow z$). Cada atributo debe depender *"de la clave, de toda la clave y de nada más que de la clave"*.
* **Diagnóstico de dependencias transitivas:**
  * En `LIBRO`: `editorial` y `categoria` son entidades independientes. Además, la relación entre `libro` y `autor` es multivaluada/muchos a muchos (**N:M**), requiriendo una tabla intermedia.
  * Cadena transitiva en Transacciones: `id_transaccion` $\rightarrow$ `id_cliente` $\rightarrow$ `nombre`, `direccion`, `telefono`.
* **Solución:** Se crean tablas independientes con claves primarias estables (`id` técnico).

#### Esquema Relacional Final en 3FN:
1. **cliente** (<u>id_cliente</u>, nombre, correo, direccion, telefono)
2. **editorial** (<u>id_editorial</u>, nombre_editorial)
3. **categoria** (<u>id_categoria</u>, nombre_categoria)
4. **autor** (<u>id_autor</u>, nombre_autor)
5. **libro** (<u>isbn</u>, titulo, fecha_publicacion, precio, stock, *id_editorial*, *id_categoria*)
6. **libro_autor** (<u>*isbn*</u>, <u>*id_autor*</u>) *(Tabla intermedia N:M)*
7. **metodo_pago** (<u>id_metodo_pago</u>, nombre_metodo)
8. **transaccion** (<u>id_transaccion</u>, fecha_transaccion, monto_total, *id_cliente*, *id_metodo_pago*)
9. **detalle_transaccion** (<u>id_detalle</u>, cantidad, precio_unitario, *id_transaccion*, *isbn*)

---

## Parte 2: Diagrama Conceptual Entidad-Relación (E-R)

### Entidades y Sus Atributos:
* **cliente:** `id_cliente` (PK), `nombre`, `correo`, `direccion`, `telefono`
* **transaccion:** `id_transaccion` (PK), `fecha_transaccion`, `monto_total`
* **metodo_pago:** `id_metodo_pago` (PK), `nombre_metodo`
* **libro:** `isbn` (PK), `titulo`, `fecha_publicacion`, `precio`, `stock`
* **editorial:** `id_editorial` (PK), `nombre_editorial`
* **categoria:** `id_categoria` (PK), `nombre_categoria`
* **autor:** `id_autor` (PK), `nombre_autor`

### Cardinalidades y Relaciones:
* **cliente** `realiza` **transaccion** $\rightarrow$ **1 : N**
* **transaccion** `utiliza` **metodo_pago** $\rightarrow$ **N : 1**
* **transaccion** `contiene` **libro** $\rightarrow$ **N : M** *(Resuelto con la entidad asociativa `detalle_transaccion`)*
* **libro** `pertenece_a` **editorial** $\rightarrow$ **N : 1**
* **libro** `se_clasifica_en` **categoria** $\rightarrow$ **N : 1**
* **libro** `es_escrito_por` **autor** $\rightarrow$ **N : M** *(Resuelto con la tabla intermedia `libro_autor`)*

---

## Parte 3: Esquema UML E-R (Modelo Lógico Final)

```mermaid
erDiagram
    CLIENTE ||--o{ TRANSACCION : realiza
    METODO_PAGO ||--o{ TRANSACCION : utilizado_en
    TRANSACCION ||--|{ DETALLE_TRANSACCION : contiene
    LIBRO ||--o{ DETALLE_TRANSACCION : incluido_en
    EDITORIAL ||--o{ LIBRO : publica
    CATEGORIA ||--o{ LIBRO : clasifica
    LIBRO }|--|{ LIBRO_AUTOR : asignado
    AUTOR }|--|{ LIBRO_AUTOR : escribe

    CLIENTE {
        int id_cliente PK
        string nombre
        string correo
        string direccion
        string telefono
    }

    METODO_PAGO {
        int id_metodo_pago PK
        string nombre_metodo
    }

    TRANSACCION {
        int id_transaccion PK
        datetime fecha_transaccion
        decimal monto_total
        int id_cliente FK
        int id_metodo_pago FK
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
        string isbn PK, FK
        int id_autor PK, FK
    }
