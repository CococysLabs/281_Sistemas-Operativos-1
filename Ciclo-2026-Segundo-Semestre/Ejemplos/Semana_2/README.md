# Semana 2: servicios web y Docker Compose

En esta semana hay dos ejercicios:

- `proyecto1-ejemplo`: dos servicios en Go que se comunican por HTTP. Hay una versión con Fiber y otra con `net/http`.
- `ejemplo-extra`: una aplicación Flask que guarda un contador de visitas en Redis y muestra el nombre del contenedor.

## Requisitos

Para el primer ejercicio se necesita Go. Para el segundo se necesitan Docker y Docker Compose.

## Ejercicio 1: Ping y Pong en Go

### Ejecutar con la librería estándar

Abre dos terminales. En la primera inicia el servidor:

```bash
cd proyecto1-ejemplo/http-net/pong
go run main.go
```

En la segunda inicia el cliente:

```bash
cd proyecto1-ejemplo/http-net/ping
go run main.go
```

El servicio `pong` escucha en el puerto `8082` y el servicio `ping` en el puerto `8081`.
Prueba la comunicación con:

```bash
curl http://localhost:8081/ping
```

La versión con Fiber se ejecuta de la misma manera:

```bash
cd proyecto1-ejemplo/fiber/pong
go run pong.go

# En otra terminal
cd proyecto1-ejemplo/fiber/ping
go run ping.go
```

Antes de ejecutar `fiber/ping`, revisa `TargetIP` en
`proyecto1-ejemplo/fiber/ping/ping.go`. El valor debe ser la dirección donde
está ejecutándose `pong`. En la versión `http-net`, revisa la dirección
configurada en el código de `ping`.

La versión con Fiber también tiene el endpoint de salud:

```bash
curl http://localhost:8081/health
curl http://localhost:8082/health
```

### Construir las imágenes

Cada servicio tiene su propio `Dockerfile`. Desde la carpeta del servicio,
construye la imagen con:

```bash
docker build -t usuario/pong-app:latest proyecto1-ejemplo/fiber/pong
docker build -t usuario/ping-app:latest proyecto1-ejemplo/fiber/ping
```

Sustituye `usuario` por el nombre de tu cuenta de Docker Hub si vas a publicar
las imágenes. La ejecución con `containerd` está explicada en
`proyecto1-ejemplo/README.md`.

## Ejercicio 2: Flask, Redis y Compose

Entra en la carpeta del ejercicio y levanta los servicios:

```bash
cd ejemplo-extra
docker compose up --build
```

Abre `http://localhost:5000` o consulta la aplicación con:

```bash
curl http://localhost:5000
```

Cada visita incrementa la clave `hits` en Redis. El volumen `redis-data`
conserva los datos aunque se reinicien los contenedores.

Para detener los servicios:

```bash
docker compose down
```

Para eliminar también los datos guardados en el volumen:

```bash
docker compose down -v
```

El color de la página se configura con la variable `COLOR_FONDO` en
`ejemplo-extra/docker-compose.yml`.
