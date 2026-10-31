# config-server · LogiTrack

Servidor de **configuración centralizada** de LogiTrack (Spring Cloud Config Server) para la materia
Desarrollo y Arquitecturas Avanzadas de Software, FI UNJu 2026.

Al arrancar, clona un repositorio Git con los archivos de configuración (`<servicio>.yml`,
`<servicio>-dev.yml`, `<servicio>-prod.yml`) y se los entrega por HTTP a `shipment-service`,
`courier-service` y `security-service`.

| | |
|---|---|
| Lenguaje | Java 25 |
| Framework | Spring Boot 4.0.9 · Spring Cloud 2025.1.1 |
| Puerto | `8888` |

## De dónde lee la configuración

El repositorio de configuración es **independiente** de este: no hace falta tenerlo copiado acá, el servidor lo
clona por URL al iniciar (`clone-on-start: true`). Por defecto usa
`https://github.com/maxiacostag-cloud/logitrack-config.git`, rama `main`.

| Variable | Para qué | Por defecto |
|---|---|---|
| `CONFIG_REPO_URI` | URL del repositorio con los `.yml` | `https://github.com/maxiacostag-cloud/logitrack-config.git` |
| `CONFIG_REPO_BRANCH` | Rama a leer | `main` |
| `SERVER_PORT` | Puerto | `8888` |

Si el repositorio de configuración cambia de dueño o de nombre (por ejemplo, al pasarlo a la organización), solo
hay que cambiar `CONFIG_REPO_URI`, sin tocar el código. Si el repositorio es privado, hay que agregar
`spring.cloud.config.server.git.username` y `password` (token), que se pasan por variable de entorno y nunca se
escriben en el código.

## Ejecutar

```bash
./mvnw -B clean verify          # los tests no clonan el repositorio ni necesitan internet
./mvnw spring-boot:run          # necesita internet para clonar el repositorio de configuración
```

Comprobar que sirve la configuración:

- http://localhost:8888/shipment-service/dev
- http://localhost:8888/courier-service/prod
- Health: http://localhost:8888/actuator/health

## Docker

```bash
docker build -t macostag26/logitrack-config-server:dev .
```

## Seguridad

- El repositorio de configuración es público: no debe contener contraseñas ni claves. Los secretos
  (`DB_PASSWORD`, `JWT_SECRET`) se escriben como `${VARIABLE}` en los `.yml` y llegan por variables de entorno de
  cada servicio.
- Solo `health` e `info` de actuator están expuestos.
