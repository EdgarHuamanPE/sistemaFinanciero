 📄 TRABAJO: CUSTOMER_PRODUCT 

---

## 🎯 OBJETIVO

Desarrollar un microservicio de **bff (bbf-Service)** que se integre con los microservicios existentes (**Customer Service** y **Product Service**) implementando el patrón **bff** para garantizar la adaptacion los microservicios a las necesidades específicas de cada frontend.

---

## 📋 DESCRIPCIÓN

En una arquitectura de microservicios para un sistema de banca, se requiere implementar el servicio de gestión de ventas de productos financieros. Este servicio debe:



## 🏗️ ARQUITECTURA DEL SISTEMA

### Arquitectura Completa
```
┌─────────────────────────────────────────────────────────────┐
│               ARQUITECTURA COMPLETA (CON ORDER)             │
└─────────────────────────────────────────────────────────────┘

                  ┌──────────────────┐
                  │   API BFF        │
                  │   Puerto: 9090   │
                  └────────┬─────────┘
                           │
              ┌────────────|
              │            │      
              ▼            ▼      
     ┌────────────┐ ┌────────────┐ 
     │  Customer  │ | Product    │ 
     │  Service   │ │  Service   │ 
     │   :9080    │ │   :9081    │ 
     └──────┬─────┘ └──────┬─────┘ 
            │              │       
            │              │              
            ▼              ▼         
       ┌───────────┐     ┌─────────┐    
       │customerdb │     │productdb│    
       │ :5435     │     │ :5436   │    
       └───────────┘     └─────────┘    
```


## 📊 MODELO DE DATOS

### Diagrama Entidad-Relación
 ```
  
┌─────────────────────────────┐
│          CUSTOMERS          │
├─────────────────────────────┤
│ PK  id                      │
│     document_type           │
│     document_number         │
│     first_name              │
│     last_name               | 
|     email                   | 
│     phone                   | 
|     status                  |
│     created_at              │
│     updated_at              │
└─────────────┬───────────────┘
              │       
              │  Custumer-Service       
              │    (customer_id)                     
              │ 
              ▼                  
Product-Service              
┌─────────────────────────────┐
│     CUSTOMER_PRODUCTS       │
├─────────────────────────────┤
│ PK  id                      │
│     customer_id             │
│ FK  product_id              │
│     account_number          │
│     start_date              |
|     end_date                |
|     status                  │
│     balance                 | 
|     contract_number         | 
|     channel_origin          |
|     created_at              |
|     updated_at              │
└──────────────────|──────────┘
                   |  N          
     product_id    ────────────────────┐
                                       │
                                       │
                                       │
                                       ▼ 1
                                 
                          ┌─────────────────────────────┐
                          │         PRODUCTS            │
                          ├─────────────────────────────┤
                          │ PK  id                      │
                          │     code                    │
                          │     name                    │
                          │     type                    |
                          |     category                |
                          |     currency                |
                          |     interest_rate           | 
                          |     description             | 
                          |     status                  |      
                          │     created_at              |
                          |     updated_at              │
                          └─────────────────────────────┘

```

## Tabla: `products`

Tabla que almacena los productos financieros disponibles en el sistema.

### Estructura de la tabla

| Columna        | Tipo              | Restricciones / Valor por defecto                             | Descripción |
|----------------|-----------------|---------------------------------------------------------------|-------------|
| `id`           | BIGSERIAL        | PRIMARY KEY                                                   | Identificador único del producto |
| `code`         | VARCHAR(20)      | NOT NULL, UNIQUE                                              | Código único del producto |
| `name`         | VARCHAR(100)     | NOT NULL                                                      | Nombre del producto |
| `type`         | VARCHAR(50)      | NOT NULL                                                      | Tipo de producto (ej. ahorro, crédito) |
| `category`     | VARCHAR(50)      |                                                               | Categoría del producto |
| `currency`     | VARCHAR(3)       |                                                               | Moneda del producto (ej. PEN, USD) |
| `interest_rate`| DECIMAL(5,2)     |                                                               | Tasa de interés asociada al producto |
| `description`  | VARCHAR(255)     |                                                               | Descripción del producto |
| `status`       | VARCHAR(20)      | NOT NULL, DEFAULT `'ACTIVO'`, CHECK (`'ACTIVO', 'INACTIVO', 'SUSPENDIDO', 'CERRADO'`) | Estado del producto |
| `created_at`   | TIMESTAMP        | NOT NULL, DEFAULT CURRENT_TIMESTAMP                           | Fecha de creación del registro |
| `updated_at`   | TIMESTAMP        | NOT NULL, DEFAULT CURRENT_TIMESTAMP                           | Fecha de última actualización |

### Índices

| Índice                        | Columnas                  | Propósito |
|--------------------------------|---------------------------|-----------|
| `idx_products_created_at`      | `created_at DESC`         | Optimiza consultas por fecha de creación más reciente |
| `idx_products_type`            | `type`                    | Optimiza consultas por tipo de producto |
| `idx_products_category`        | `category`                | Optimiza consultas por categoría |
| `idx_products_type_category`   | `type, category`          | Optimiza consultas combinadas por tipo y categoría |
| `idx_products_status`          | `status`                  | Optimiza consultas por estado del producto |

### Restricciones adicionales

- `code` debe ser único para cada producto.
- `status` solo puede contener los valores: `'ACTIVO'`, `'INACTIVO'`, `'SUSPENDIDO'`, `'CERRADO'`.

---

Este formato lo puedes copiar directamente a tu README para documentar la tabla de productos.  

Si quieres, también puedo hacer una **versión visual con esquema de tabla y relaciones** lista para incluir en README.  

¿Quieres que haga eso también?


### Tabla: order_items

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| `id` | BIGSERIAL | PRIMARY KEY | ID único del item |
| `order_id` | BIGINT | NOT NULL, FK → orders(id) CASCADE | ID de la orden |
| `product_id` | BIGINT | NOT NULL | ID del producto (ref. externa) |
| `quantity` | INTEGER | NOT NULL, > 0 | Cantidad |
| `unit_price` | NUMERIC(10,2) | NOT NULL, >= 0 | Precio unitario |
| `subtotal` | NUMERIC(10,2) | NOT NULL, >= 0 | Subtotal (qty × price) |

### Script SQL
```sql
-- tabla producto
CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    code VARCHAR(20) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50) NOT NULL,
    category VARCHAR(50),
    currency VARCHAR(3),
    interest_rate DECIMAL(5,2),
    description VARCHAR(255),
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVO',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT chk_product_status CHECK (status IN ('ACTIVO', 'INACTIVO', 'SUSPENDIDO', 'CERRADO'))
);

CREATE INDEX idx_products_created_at ON products(created_at DESC);
CREATE INDEX idx_products_type ON products(type);
CREATE INDEX idx_products_category ON products(category);
CREATE INDEX idx_products_type_category ON products(type, category);
CREATE INDEX idx_products_status ON products(status);

--tabla cliente
CREATE TABLE customers (
    id BIGSERIAL PRIMARY KEY,
    document_type VARCHAR(10) NOT NULL,
    document_number VARCHAR(20) UNIQUE NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    phone VARCHAR(20),
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVO',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT chk_customer_status CHECK (status IN ('ACTIVO', 'INACTIVO', 'SUSPENDIDO', 'CERRADO')),
    CONSTRAINT chk_customer_document_type CHECK (document_type IN ('DNI', 'RUC', 'CE', 'PAS'))
);

CREATE INDEX idx_customer_document_type ON customers(document_type);
CREATE INDEX idx_customer_status ON customers(status);
CREATE INDEX idx_customer_name ON customers(first_name, last_name);
CREATE INDEX idx_customer_email ON customers(email);



--tabla cliente_producto
CREATE TABLE customer_products (
    id BIGSERIAL PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    account_number VARCHAR(30) UNIQUE NOT NULL,
    start_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    end_date TIMESTAMP,
    status VARCHAR(20) DEFAULT 'ACTIVO',
    balance DECIMAL(18,2),
    contract_number VARCHAR(30),
    channel_origin VARCHAR(50),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT chk_customer_status CHECK (status IN ('ACTIVO', 'INACTIVO', 'SUSPENDIDO', 'CERRADO')),
    CONSTRAINT chk_channel_origin CHECK (channel_origin IN ('AGENCIA','APP_MOVIL','CALL_CENTER','WEB') OR channel_origin IS NULL),
    CONSTRAINT fk_customer_product_product FOREIGN KEY (product_id) REFERENCES products(id)
);


CREATE INDEX idx_cp_customer_id ON customer_products(customer_id);
CREATE INDEX idx_cp_product_id ON customer_products(product_id);
CREATE INDEX idx_cp_customer_status ON customer_products(customer_id, status);
CREATE INDEX idx_cp_status ON customer_products(status);
CREATE INDEX idx_cp_channel_origin ON customer_products(channel_origin);


--Data de prueba para productos

-- Cuentas de ahorro y corrientes
INSERT INTO products  (code, name, type, category, currency, interest_rate, description, status)
VALUES
('SAVINGS_BASIC', 'Cuenta de Ahorros Clásica', 'AHORRO', 'Cuenta básica', 'PEN', 0.50, 'Cuenta de ahorros sin costo de mantenimiento.', 'ACTIVO'),
('SAVINGS_PLUS', 'Cuenta de Ahorros Plus', 'AHORRO', 'Cuenta premium', 'PEN', 1.00, 'Cuenta con mayor tasa por saldo promedio superior a S/ 5,000.', 'ACTIVO'),
('CURRENT_ACCOUNT', 'Cuenta Corriente', 'AHORRO', 'Empresarial', 'PEN', 0.00, 'Cuenta para operaciones empresariales con chequera.', 'ACTIVO');

-- Tarjetas de crédito
INSERT INTO products (code, name, type, category, currency, interest_rate, description, status)
VALUES
('CREDIT_CLASSIC', 'Tarjeta de Crédito Clásica', 'CRÉDITO', 'Tarjeta consumo', 'PEN', 45.00, 'Tarjeta con línea de crédito personal y pagos mensuales.', 'ACTIVO'),
('CREDIT_GOLD', 'Tarjeta de Crédito Gold', 'CRÉDITO', 'Tarjeta consumo', 'USD', 39.00, 'Tarjeta con beneficios internacionales y acumulación de puntos.', 'ACTIVO'),
('CREDIT_BUSINESS', 'Tarjeta Empresarial', 'CRÉDITO', 'Tarjeta empresarial', 'PEN', 35.00, 'Tarjeta para compras de negocios con control de gastos.', 'ACTIVO');

-- Préstamos
INSERT INTO products (code, name, type, category, currency, interest_rate, description, status)
VALUES
('LOAN_PERSONAL', 'Préstamo Personal', 'CRÉDITO', 'Consumo', 'PEN', 12.50, 'Crédito para necesidades personales con cuotas mensuales fijas.', 'ACTIVO'),
('LOAN_VEHICLE', 'Préstamo Vehicular', 'CRÉDITO', 'Automotriz', 'PEN', 9.50, 'Financiamiento para compra de autos nuevos o usados.', 'ACTIVO'),
('LOAN_MORTGAGE', 'Crédito Hipotecario', 'CRÉDITO', 'Vivienda', 'PEN', 8.00, 'Financiamiento para compra o construcción de vivienda.', 'ACTIVO');

-- Depósitos e inversiones
INSERT INTO products (code, name, type, category, currency, interest_rate, description, status)
VALUES
('FIXED_TERM', 'Depósito a Plazo Fijo', 'INVERSIÓN', 'Plazo fijo', 'PEN', 6.00, 'Depósito con tasa fija según plazo contratado.', 'ACTIVO'),
('MUTUAL_FUND', 'Fondo Mutuo Conservador', 'INVERSIÓN', 'Fondo mutuo', 'PEN', 0.00, 'Inversión colectiva con perfil conservador.', 'ACTIVO');

-- Seguros
INSERT INTO products (code, name, type, category, currency, interest_rate, description, status)
VALUES
('INSURANCE_LIFE', 'Seguro de Vida', 'SEGURO', 'Vida individual', 'PEN', NULL, 'Protección económica ante fallecimiento del asegurado.', 'ACTIVO'),
('INSURANCE_CARD', 'Seguro contra Fraude en Tarjeta', 'SEGURO', 'Tarjeta', 'PEN', NULL, 'Cobertura ante robos o fraudes con tarjeta.', 'ACTIVO');


--Data de prueba para clientes

INSERT INTO customers (document_type, document_number, first_name, last_name, email, phone, status)
VALUES
-- Clientes naturales (DNI)
('DNI', '45896321', 'María', 'Gonzales', 'maria.gonzales@gmail.com', '987654321', 'ACTIVO'),
('DNI', '72648953', 'Juan', 'Huamán', 'juan.huaman@gmail.com', '998877665', 'ACTIVO'),
('DNI', '60421789', 'Carla', 'Ramírez', 'carla.ramirez@yahoo.com', '912345678', 'SUSPENDIDO'),

-- Cliente con carnet de extranjería (CE)
('CE', 'E1234567', 'John', 'Smith', 'john.smith@hotmail.com', '955667788', 'ACTIVO'),

-- Cliente con pasaporte (PAS)
('PAS', 'P9876543', 'Sofía', 'Martínez', 'sofia.martinez@gmail.com', '911223344', 'INACTIVO'),

-- Clientes con RUC (empresas)
('RUC', '20456789123', 'Inversiones Andinas', 'S.A.C.', 'contacto@andinas.com.pe', '014567890', 'ACTIVO'),
('RUC', '20678912345', 'Servicios del Sur', 'E.I.R.L.', 'ventas@servsur.com.pe', '016789123', 'CERRADO');

--Data de prueba para clintes-productos
INSERT INTO customer_products
(customer_id, product_id, account_number, start_date, end_date, status, balance , contract_number, channel_origin)
VALUES
-- Juan Pérez - Cuenta de Ahorros
(1, 1, '001-12345678', '2022-05-10', NULL, 'ACTIVO', 3500.75, 'CTR-20220510-01', 'WEB'),

-- Juan Pérez - Tarjeta de Crédito
(1, 4, '4111-1234-5678-9010', '2023-03-15', NULL, 'ACTIVO', -1200.00,'CTR-20230315-02', 'WEB'),

-- María López - Depósito a Plazo Fijo
(2, 5, 'DPF-202309-001', '2023-09-01', '2024-09-01', 'ACTIVO', 10000.00,'CTR-20230901-03', 'CALL_CENTER'),

-- Carlos Gómez - Crédito Personal
(3, 3, 'CR-2022-8899', '2022-02-20', NULL, 'ACTIVO', -5000.00,'CTR-20220220-04', 'APP_MOVIL'),

-- Carlos Gómez - Cuenta Corriente
(3, 2, '002-99887766', '2021-12-01', NULL, 'ACTIVO', 2500.00,'CTR-20211201-05', 'AGENCIA');





```

---

## 🎯 REQUERIMIENTOS FUNCIONALES

### RF-01: Crear customer


