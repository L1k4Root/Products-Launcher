# Glosario

## API Gateway

Servicio que recibe las peticiones HTTP externas y las redirige al microservicio correcto.

## Microservicio

Servicio pequeño con una responsabilidad acotada y despliegue independiente.

## NATS

Broker de mensajería usado para la comunicación interna entre servicios.

## Message Pattern

Contrato request/response en NestJS microservices. Se usa cuando un servicio espera una respuesta.

## Event Pattern

Contrato basado en eventos. Se usa cuando un servicio notifica que algo ocurrió y otro reacciona.

## DTO

Objeto que define y valida la estructura esperada de entrada.

## Prisma

ORM usado para mapear modelos y hacer queries hacia la base de datos.

## Webhook

Endpoint HTTP que recibe eventos enviados por un sistema externo, en este caso Stripe.

## Stripe Checkout Session

Sesión creada en Stripe que entrega una URL de pago y concentra la experiencia de checkout.

## Soft delete

Técnica donde un registro no se elimina físicamente; solo se marca como inactivo.
