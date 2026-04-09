# Arquitectura del proyecto

## Resumen

Este proyecto usa una arquitectura de microservicios con NestJS donde cada servicio tiene una responsabilidad acotada:

- `api-gateway` es la puerta de entrada HTTP.
- `products-ms` administra catálogo.
- `order-ms` centraliza la lógica de órdenes.
- `payments-ms` encapsula la integración con Stripe.
- `auth-ms` entrega un flujo mock de autenticación.

La comunicación interna ocurre por NATS usando request/response y eventos.

## Por qué hay un gateway

El frontend o consumidor externo no habla directamente con todos los microservicios. En vez de eso, entra por `api-gateway`, que:

- valida la entrada HTTP;
- expone Swagger;
- traduce HTTP a mensajes NATS;
- mantiene una superficie pública más simple.

Esto evita que cada microservicio tenga que exponerse directamente a internet.

## Qué persiste cada servicio

| Servicio | Persistencia | Motivo |
| --- | --- | --- |
| `products-ms` | SQLite con Prisma | Catálogo simple para desarrollo local |
| `order-ms` | PostgreSQL con Prisma | Órdenes, ítems y recibos |
| `payments-ms` | No persiste localmente | Stripe es la fuente externa del checkout |
| `auth-ms` | No persiste | Servicio mock |
| `api-gateway` | No persiste | Solo enruta y adapta |

## Comunicación interna

### Request/response

Se usa cuando un servicio necesita una respuesta inmediata:

- `api-gateway -> products-ms`
- `api-gateway -> order-ms`
- `api-gateway -> auth-ms`
- `order-ms -> products-ms`
- `order-ms -> payments-ms`

### Eventos

Se usa cuando un hecho ya ocurrió y otro servicio debe reaccionar:

- `payments-ms -> order-ms` con `payment.suceeded`

## Decisiones de diseño visibles en el código

### `products-ms` es pequeño a propósito

Es el mejor lugar para aprender el patrón base del repo:

- DTOs de entrada;
- `MessagePattern` en controlador;
- Prisma en servicio;
- paginación básica;
- soft delete por `available`.

### `order-ms` concentra la orquestación

`order-ms` no solo guarda datos. También:

- valida productos;
- calcula totales;
- crea la orden;
- pide la sesión de pago;
- reacciona al evento de pago.

Por eso es el servicio más cercano a una “orquestación” de negocio.

### `payments-ms` mezcla HTTP y NATS

Eso es intencional:

- por NATS recibe solicitudes para crear sesiones de pago;
- por HTTP recibe webhooks de Stripe.

Stripe necesita un endpoint HTTP real y además exige validar la firma con el `raw body`. Esa es la razón de la configuración especial en `main.ts`.

## Lectura mental del sistema

Una forma útil de pensar el proyecto es esta:

- HTTP entra al gateway.
- NATS mueve comandos entre servicios.
- Prisma persiste el estado de negocio.
- Stripe confirma el pago.
- Un evento NATS cierra el circuito de la orden.

## Riesgos y puntos de mejora

- `auth-ms` hoy es demostrativo, no productivo.
- El nombre `payment.suceeded` tiene un typo histórico.
- Los contratos NATS están duplicados entre servicios y podrían compartirse.
- El manejo de errores RPC aún puede estandarizarse mejor.
