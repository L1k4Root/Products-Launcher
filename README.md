# Products Launcher (NestJS + Microservicios)

> Launcher para levantar un conjunto de microservicios construidos con NestJS mediante Docker Compose.

## Requisitos

- Git
- Docker + Docker Compose
- Node.js (usa la versión definida por el proyecto)
  - Recomendado: `nvm` (macOS/Linux)

## Variables de entorno

1. Crea un archivo `.env` en la raíz del proyecto.
2. Usa como base `.env.template`.

> Nota: no subas `.env` al repositorio.

## Instalación

Clona el repositorio y sus submódulos:

```bash
git clone <URL_DEL_REPO>
cd 03-Products-Launcher

git submodule update --init --recursive
```

Si usas `nvm`, selecciona la versión de Node indicada por el proyecto:

```bash
nvm use
```

## Levantar el proyecto

Construye y levanta los servicios:

```bash
docker-compose up --build
```

Para correr en segundo plano:

```bash
docker-compose up -d --build
```

## Acceso

- API / App: `http://localhost:3000` (o el puerto definido en tu `.env`)

## Comandos útiles

Detener servicios:

```bash
docker-compose down
```

Ver logs:

```bash
docker-compose logs -f
```

Rebuild limpio (cuando cambian dependencias o Dockerfile):
p
```bash
docker-compose down
# opcional: limpia imágenes/volúmenes según tu caso
# docker system prune

docker-compose up --build
```

## Estructura

- Este repositorio actúa como **launcher**.
- Los microservicios viven como **submódulos**.

## Troubleshooting

### No carga la versión de Node

- Verifica que `nvm` esté instalado y que exista `.nvmrc` (si aplica).
- Ejecuta:

```bash
nvm use
```

### Cambios no se reflejan en Docker

- Rebuild:

```bash
docker-compose up --build
```

- Si el problema persiste, prueba bajar y levantar de nuevo:

```bash
docker-compose down

docker-compose up --build
```


