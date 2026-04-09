# Flujos críticos

## 1. Registro, login y verify mock

Este flujo existe para poder probar el paso de datos entre `api-gateway` y `auth-ms`, no para seguridad real.

```mermaid
sequenceDiagram
  participant C as Cliente
  participant G as api-gateway
  participant A as auth-ms

  C->>G: POST /api/auth/register
  G->>A: auth.register.user
  A-->>G: usuario mock
  G-->>C: respuesta

  C->>G: POST /api/auth/login
  G->>A: auth.login.user
  A-->>G: accessToken mock
  G-->>C: respuesta

  C->>G: GET /api/auth/verify
  G->>A: auth.verify.user
  A-->>G: token válido mock
  G-->>C: respuesta
```

## 2. Gestión de productos

```mermaid
sequenceDiagram
  participant C as Cliente
  participant G as api-gateway
  participant P as products-ms
  participant DB as SQLite

  C->>G: POST/GET/PATCH/DELETE /api/products
  G->>P: cmd create/get/update/remove
  P->>DB: consulta o escritura Prisma
  DB-->>P: resultado
  P-->>G: respuesta
  G-->>C: respuesta HTTP
```

Notas importantes:

- el listado solo devuelve productos con `available = true`;
- el borrado actual es soft delete;
- la validación de productos para órdenes usa el mismo servicio.

## 3. Creación de orden

```mermaid
sequenceDiagram
  participant C as Cliente
  participant G as api-gateway
  participant O as order-ms
  participant P as products-ms
  participant PG as PostgreSQL
  participant Pay as payments-ms

  C->>G: POST /api/order
  G->>O: createOrder
  O->>P: validateProductsByIds
  P-->>O: productos válidos y precios
  O->>PG: create order + order items
  O->>Pay: create.payment.session
  Pay-->>O: checkout session
  O-->>G: orden + session
  G-->>C: respuesta HTTP
```

Qué hace `order-ms` aquí:

- valida que todos los productos existan y estén disponibles;
- calcula `totalPrice` y `totalItems`;
- persiste orden e ítems;
- llama a pagos para obtener el checkout.

## 4. Confirmación de pago con Stripe

```mermaid
sequenceDiagram
  participant S as Stripe
  participant Pay as payments-ms
  participant N as NATS
  participant O as order-ms
  participant PG as PostgreSQL

  S->>Pay: POST /payments/webhook
  Pay->>Pay: valida Stripe-Signature + raw body
  Pay->>N: emit payment.suceeded
  N->>O: event payment.suceeded
  O->>PG: update order paid=true status=PAID
```

Puntos clave:

- la validación del webhook depende del `rawBody`;
- `payments-ms` no actualiza órdenes directamente;
- el acoplamiento se reduce emitiendo un evento.

## 5. Consulta de orden

Cuando se consulta una orden:

1. `api-gateway` envía `findOneOrder`.
2. `order-ms` recupera la orden y sus ítems desde PostgreSQL.
3. `order-ms` vuelve a consultar `products-ms` para completar nombres de producto.
4. responde una vista enriquecida.

Esto hace que la orden no duplique todo el catálogo, aunque también introduce dependencia de lectura hacia `products-ms`.
