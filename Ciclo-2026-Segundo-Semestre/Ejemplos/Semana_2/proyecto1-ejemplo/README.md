# Proyecto Ping-Pong

Este directorio contiene dos implementaciones de una comunicación HTTP entre
un servicio `ping` y un servicio `pong`:

- `fiber`: usa el framework Fiber.
- `http-net`: usa la librería estándar `net/http`.

En ambas implementaciones, `pong` escucha en el puerto `8082` y `ping` en el
puerto `8081`. El servicio `ping` recibe una solicitud y luego consulta a
`pong`.

## Ejecución local

Inicia primero `pong`:

```bash
cd fiber/pong
go run pong.go
```

En otra terminal, inicia `ping`:

```bash
cd fiber/ping
go run ping.go
```

Prueba los endpoints:

```bash
curl http://localhost:8081/ping
curl http://localhost:8081/health
curl http://localhost:8082/health
```

Para usar la versión basada en la librería estándar, repite el procedimiento
en `http-net/pong` y `http-net/ping`. Los nombres de los archivos son
`pong.go` y `ping.go`, respectivamente.

Si los servicios están en máquinas distintas, cambia `TargetIP` en
`fiber/ping/ping.go` y la dirección equivalente en
`http-net/ping/main.go` por la dirección de la máquina donde corre `pong`.

## Construcción de imágenes

Construye cada imagen desde su propia carpeta:

```bash
docker build -t usuario/pong-app:latest fiber/pong
docker build -t usuario/ping-app:latest fiber/ping
```

Después de publicar las imágenes en un registro, pueden descargarse desde la
máquina que ejecutará cada servicio. La guía de la carpeta `Semana2` contiene
un ejemplo de ejecución con `containerd`.
