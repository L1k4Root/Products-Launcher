# Desarrollo y lectura del código

## Cómo seguir una petición de punta a punta

### Caso 1: crear producto

1. Entra por [`api-gateway/src/products/products.controller.ts`](/Users/cfs-andres/Workspace/Cursos/Nest+Microservicios/03-Products-Launcher/api-gateway/src/products/products.controller.ts).
2. El controller publica `{ cmd: 'createProduct' }`.
3. Lo consume [`products-ms/src/products/products.controller.ts`](/Users/cfs-andres/Workspace/Cursos/Nest+Microservicios/03-Products-Launcher/products-ms/src/products/products.controller.ts).
4. El servicio escribe con Prisma en SQLite.

### Caso 2: crear orden

1. Entra por [`api-gateway/src/order/order.controller.ts`](/Users/cfs-andres/Workspace/Cursos/Nest+Microservicios/03-Products-Launcher/api-gateway/src/order/order.controller.ts).
2. Se envía `createOrder` a `order-ms`.
3. `order-ms` valida productos con `products-ms`.
4. `order-ms` persiste la orden.
5. `order-ms` solicita la sesión de pago a `payments-ms`.

### Caso 3: webhook de Stripe

1. Stripe llama a [`payments-ms/src/payments/payments.controller.ts`](/Users/cfs-andres/Workspace/Cursos/Nest+Microservicios/03-Products-Launcher/payments-ms/src/payments/payments.controller.ts).
2. `payments-ms` verifica la firma.
3. Emite `payment.suceeded`.
4. `order-ms` consume el evento y actualiza la orden.

## Organización por servicio

### `api-gateway`

- `src/*/*.controller.ts`: endpoints HTTP
- `src/nats`: cliente NATS compartido
- `src/common`: DTOs comunes y filtro RPC

### `products-ms`

- `src/products`: mensajes NATS y lógica de catálogo
- `prisma/schema.prisma`: modelo `Product`
- `generated/prisma`: cliente generado

### `order-ms`

- `src/orders`: mensajes, DTOs y lógica de órdenes
- `src/prisma`: servicio de acceso a PostgreSQL
- `prisma/schema.prisma`: modelos `Order`, `OrderItem`, `OrderReceipt`

### `payments-ms`

- `src/payments`: creación de sesión y webhook Stripe
- `src/nats`: cliente para emitir eventos

### `auth-ms`

- `src/auth`: DTOs y respuestas mock

## Cómo agregar una feature nueva sin perder el patrón

Si agregas una capacidad nueva, sigue esta secuencia:

1. define el endpoint HTTP si será público;
2. define el pattern NATS que representará la operación;
3. implementa el controller NATS del microservicio dueño del dominio;
4. mueve la lógica real al service;
5. si persiste datos, actualiza Prisma en el servicio dueño;
6. documenta el flujo y el contrato.

## Recomendaciones para mantener el proyecto sano

- Evita poner lógica de negocio pesada en `api-gateway`.
- Deja que cada microservicio sea dueño de su persistencia.
- Usa eventos para notificar hechos ya ocurridos.
- Usa request/response solo cuando realmente necesitas respuesta inmediata.
- Si agregas nuevos subjects NATS, documenta quién los emite y quién los consume.

## Checks útiles durante desarrollo

Levantar todo:

```bash
docker-compose up --build
```

Ver logs:

```bash
docker-compose logs -f
```

Ver logs de un servicio:

```bash
docker-compose logs -f order-ms
```

Parar y limpiar:

```bash
docker-compose down -v
```
