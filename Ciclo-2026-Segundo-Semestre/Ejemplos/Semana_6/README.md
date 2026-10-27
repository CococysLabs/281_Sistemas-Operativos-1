# Semana 6: daemon, Valkey y Grafana

El ejemplo ejecuta un daemon escrito en Go. El daemon guarda métricas
simuladas en Valkey, programa una tarea de `cron` que crea contenedores Docker
y deja Grafana disponible para consultar los datos.

## Requisitos

Se necesita Go, Docker con el complemento Compose y `cron`. En Ubuntu o
Debian:

```bash
sudo apt update
sudo apt install -y golang docker.io docker-compose-v2 cron
sudo systemctl enable --now cron
```

El daemon ejecuta comandos de Docker directamente. Si Docker no funciona sin
`sudo`, agrega tu usuario al grupo correspondiente y vuelve a iniciar sesión:

```bash
sudo usermod -aG docker "$USER"
newgrp docker
```

Comprueba la instalación:

```bash
go version
docker compose version
```

## Iniciar el ejemplo

Desde esta carpeta, da permisos a los scripts y ejecuta el de inicio:

```bash
chmod +x start.sh stop.sh
./start.sh
```

El script compila `daemon/main.go`, inicia Valkey y Grafana mediante Compose y
ejecuta el daemon en segundo plano. Grafana queda disponible en
`http://localhost:3001`.

El daemon genera métricas cada 20 segundos. También registra una tarea de
cron para ejecutar el binario con el argumento `crear-contenedor` una vez por
minuto.

Puedes revisar la tarea y los contenedores con:

```bash
crontab -l
docker ps
```

## Detener el ejemplo

Usa el script de cierre desde la misma carpeta:

```bash
./stop.sh
```

Este script detiene el daemon, elimina el `cronjob` del usuario y baja los
servicios de Docker Compose.

El script usa `crontab -r`, por lo que elimina todas las tareas del crontab del
usuario actual. Guarda cualquier tarea personal antes de ejecutar `stop.sh`.
