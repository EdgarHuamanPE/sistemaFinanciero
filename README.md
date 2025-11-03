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

### Tabla: orders

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| `id` | BIGSERIAL | PRIMARY KEY | ID único de la orden |
| `order_number` | VARCHAR(50) | UNIQUE, NOT NULL | Número de orden (ej: ORD-2025-001) |
| `user_id` | BIGINT | NOT NULL | ID del usuario (ref. externa) |
| `status` | VARCHAR(20) | NOT NULL, DEFAULT 'PENDING' | Estado de la orden |
| `total_amount` | NUMERIC(10,2) | NOT NULL, >= 0 | Monto total |
| `created_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Fecha de creación |
| `updated_at` | TIMESTAMP | NOT NULL, DEFAULT NOW() | Fecha de actualización |

**Estados válidos:** `PENDING`, `CONFIRMED`, `SHIPPED`, `DELIVERED`, `CANCELLED`

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

### RF-01: Crear Orden de Compra

**Endpoint:** `POST /api/orders`

**Request:**
```json
{
  "customerId": 1,
  "Products": [
    {
      "productId": 1,
      "account_number": "001-12345678",
      "start_date": "2022-05-10",
      "end_date": NULL,
      "status":"ACTIVO",
      "balance": 0,
      "contract_number": "CTR-20220510-01",
      "channel_origin": "Banca Móvil"
    },
     {
      "productId": 4,
      "account_number": "4111-1234-5678-9010",
      "start_date": "2023-03-15",
      "end_date": NULL,
      "status":"ACTIVO",
      "balance": 0,
      "contract_number": "CTR-20230315-02",
      "channel_origin": "Oficina"
    }
  ]
}
```

**Response (201 Created):**
```json
{
  "customer":  {
      "id": 1,
      "firstName": "Juan",
      "lastName": "Pérez",
      "documentType": "DNI",
      "documentNumber": "70123456" },
  "products": [
    {
      "typeProduct": "AHORRO",
      "name": "Cuenta de Ahorros Clásica",
      "balance": 3500.75
    },
     {
      "typeProduct": "CRÉDITO",
      "name": "Préstamo Personal",
      "balance": -1200.00
    },

  ],
}
```

**Proceso:**
1. Validar usuario llamando a User Service
2. Para cada item:
   - Validar producto en Product Service
   - Obtener precio actual
   - Calcular subtotal
3. Calcular total de la orden
4. Generar número de orden único
5. Guardar en BD
6. Retornar orden completa

### RF-02: Obtener Orden Completa

**Endpoint:** `GET /api/orders/{id}`

**Response (200 OK):**
```json
{
  "id": 1,
  "orderNumber": "ORD-2025-001",
  "user": { ... },
  "items": [ ... ],
  "totalAmount": 2599.98,
  "status": "CONFIRMED",
  "createdAt": "2025-01-20T10:30:00",
  "updatedAt": "2025-01-20T11:00:00"
}
```

---


## 📦 ESTRUCTURA DEL PROYECTO
```
order-service/
├── src/main/java/com/tecsup/orderservice/
│   ├── OrderServiceApplication.java
│   ├── controller/
│   │   └── OrderController.java
│   ├── service/
│   │   ├── OrderService.java
│   │   └── OrderItemService.java
│   ├── client/
│   │   ├── User.java          
│   │   ├── UserClient.java          ← Circuit Breaker
│   │   ├── Product.java 
│   │   └── ProductClient.java       ← Circuit Breaker
│   ├── entity/
│   │   ├── OrderEntity.java
│   │   └── OrderItemEntity.java
│   ├── dto/
│   │   ├── Order.java
│   │   ├── OrderItem.java
│   │   └── CreateOrderRequest.java
│   ├── repository/
│   │   ├── OrderRepository.java
│   │   └── OrderItemRepository.java
│   ├── mapper/
│   │   ├── OrderMapper.java
│   │   └── OrderItemMapper.java
│   └── config/
│       └── AppConfig.java
└── src/main/resources/
    ├── application.yml
    ├── bootstrap.yml
    └── db/migration/
        └── V1__INIT_SCHEMA.sql
```

---

## 📋 ENTREGABLES

### 1. Código Fuente
- [ ] Proyecto completo de Order Service
- [ ] Código limpio y comentado
- [ ] Estructura organizada

### 2. Base de Datos
- [ ] Script SQL (`V1__INIT_SCHEMA.sql`)
- [ ] Datos de prueba (mínimo 3 órdenes)

### 3. Configuración
- [ ] `application.yml` completo
- [ ] `bootstrap.yml` completo
- [ ] `config-repo/order-service.yml` con Circuit Breaker
- [ ] `docker-compose.yml` actualizado

---

## 🎓 CRITERIOS DE EVALUACIÓN

| Criterio | Puntos |
|----------|--------|
| Funcionalidad completa | 4 |
| Circuit Breaker User Service | 3 |
| Circuit Breaker Product Service | 3 |
| Fallback Methods correctos | 2 |
| Base de Datos (esquema + datos) | 2 |
| Pruebas (5 casos ejecutados) | 3 |
| Código limpio y organizado | 1 |
| **TOTAL** | **20** |


---
