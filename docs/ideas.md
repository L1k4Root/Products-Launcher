# Ideas de mejora

## Prioridad alta

- Reemplazar `auth-ms` mock por autenticación real con persistencia, hash de contraseñas y JWT.
- Corregir y normalizar contratos NATS compartidos para evitar strings duplicados y typos como `payment.suceeded`.
- Estandarizar respuestas de error entre HTTP y RPC.
- Agregar pruebas end-to-end del flujo crítico completo.

## Prioridad media

- Crear una librería compartida de DTOs, enums y subjects.
- Implementar idempotencia del webhook de Stripe.
- Guardar más contexto de pagos en la orden o en un módulo propio de billing.
- Añadir health checks y readiness probes por servicio.
- Incorporar logs estructurados con correlación por request/evento.

## Prioridad baja

- Cambiar `products-ms` desde SQLite a PostgreSQL si el objetivo deja de ser solo local/aprendizaje.
- Exponer métricas y tracing distribuido.
- Añadir reintentos y timeout más explícitos en llamadas entre servicios.
- Incorporar versionado de contratos públicos.

## Mejoras de experiencia para onboarding

- Añadir colección de Postman o archivo `.http`.
- Incorporar seed inicial de productos.
- Crear script de bootstrap local con verificación de dependencias.
- Añadir ejemplos de payload por endpoint y por pattern NATS.
