# Ejercicios Prácticos Resueltos de Docker

## 1. Dockerfile y Construcción de Imágenes

### Ejercicio 1.1: Creación de una Aplicación Web Python
```bash
# Crear estructura del proyecto
mkdir python-app
cd python-app
```

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "¡Hola desde Docker!"

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

```plaintext
# requirements.txt
flask==2.0.1
```

```dockerfile
# Dockerfile
FROM python:3.9-slim as builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.9-slim
WORKDIR /app

COPY --from=builder /root/.local /root/.local
COPY app.py .

ENV PATH=/root/.local/bin:$PATH

EXPOSE 5000
CMD ["python", "app.py"]
```

```bash
# Construir y ejecutar
docker build -t webapp:v1 .
docker run -d -p 5000:5000 webapp:v1

# Verificar el contenedor
docker ps
curl localhost:5000
```

### Ejercicio 1.2: Creación de una Aplicación Node.js
```bash
# Crear estructura
mkdir node-app
cd node-app
```

```javascript
// app.js
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('¡Hola desde Node.js!');
});

app.listen(port, () => {
  console.log(`Aplicación ejecutándose en puerto ${port}`);
});
```

```json
// package.json
{
  "name": "docker-node-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

```dockerfile
# Dockerfile
FROM node:16-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY app.js .

EXPOSE 3000
CMD ["node", "app.js"]
```

```bash
# Construir y ejecutar
docker build -t nodeapp:v1 .
docker run -d -p 3000:3000 nodeapp:v1
```

## 2. Redes de Contenedores

### Ejercicio 2.1: Aplicación Web + Base de Datos
```bash
# Crear red
docker network create app-net

# Ejecutar MySQL
docker run -d \
  --name mysql \
  --network app-net \
  -e MYSQL_ROOT_PASSWORD=secretpass \
  -e MYSQL_DATABASE=testdb \
  mysql:8.0

# Crear aplicación PHP
mkdir php-app
cd php-app
```

```php
<?php
// index.php
$host = 'mysql';
$user = 'root';
$pass = 'secretpass';
$db   = 'testdb';

$conn = new mysqli($host, $user, $pass, $db);

if ($conn->connect_error) {
    die("Conexión fallida: " . $conn->connect_error);
}

echo "¡Conexión exitosa a MySQL!";
?>
```

```dockerfile
# Dockerfile
FROM php:8.0-apache
RUN docker-php-ext-install mysqli
COPY index.php /var/www/html/
```

```bash
# Construir y ejecutar PHP
docker build -t php-mysql:v1 .
docker run -d \
  --name webapp \
  --network app-net \
  -p 80:80 \
  php-mysql:v1
```

## 3. Almacenamiento en Contenedores

### Ejercicio 3.1: Persistencia de Datos con MySQL
```bash
# Crear volumen
docker volume create mysql_data

# Ejecutar MySQL con volumen
docker run -d \
  --name mysql_persistent \
  -v mysql_data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secretpass \
  mysql:8.0

# Insertar datos de prueba
docker exec -it mysql_persistent mysql -psecretpass -e "
CREATE DATABASE IF NOT EXISTS testdb;
USE testdb;
CREATE TABLE users (id INT, name VARCHAR(50));
INSERT INTO users VALUES (1, 'Usuario1');
"

# Verificar persistencia
docker stop mysql_persistent
docker rm mysql_persistent

docker run -d \
  --name mysql_persistent_2 \
  -v mysql_data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secretpass \
  mysql:8.0

docker exec -it mysql_persistent_2 mysql -psecretpass -e "
USE testdb;
SELECT * FROM users;
"
```

### Ejercicio 3.2: Compartir Archivos con Nginx
```bash
# Crear directorio para contenido web
mkdir nginx-content
echo "<h1>¡Contenido Persistente!</h1>" > nginx-content/index.html

# Ejecutar Nginx con bind mount
docker run -d \
  --name nginx_persistent \
  -v "$(pwd)"/nginx-content:/usr/share/nginx/html \
  -p 8080:80 \
  nginx

# Modificar contenido en tiempo real
echo "<h1>¡Contenido Actualizado!</h1>" > nginx-content/index.html
```

## 4. Seguridad en Contenedores

### Ejercicio 4.1: Contenedor Seguro Node.js
```dockerfile
# Dockerfile.secure
FROM node:16-alpine

# Crear usuario no privilegiado
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

COPY package*.json ./
RUN npm install
COPY app.js .

# Cambiar propiedad de archivos
RUN chown -R appuser:appgroup /app

# Usar usuario no privilegiado
USER appuser

EXPOSE 3000
CMD ["node", "app.js"]
```

```bash
# Construir y ejecutar
docker build -t nodeapp:secure -f Dockerfile.secure .
docker run -d \
  --name secure_node \
  --read-only \
  --tmpfs /tmp \
  --cap-drop ALL \
  -p 3000:3000 \
  nodeapp:secure
```

### Ejercicio 4.2: MySQL Seguro con Secretos
```bash
# Crear archivo de secretos
echo "mysecretpassword" > mysql_password.txt

# Ejecutar MySQL con secretos
docker run -d \
  --name mysql_secure \
  --network app-net \
  -v mysql_data:/var/lib/mysql \
  -v "$(pwd)"/mysql_password.txt:/run/secrets/mysql_password:ro \
  -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/mysql_password \
  --security-opt=no-new-privileges \
  --cap-drop=ALL \
  --cap-add=CHOWN \
  --cap-add=NET_BIND_SERVICE \
  --cap-add=DAC_OVERRIDE \
  mysql:8.0
```

## Comandos Útiles para Verificación

```bash
# Verificar contenedores en ejecución
docker ps

# Ver logs
docker logs <container_name>

# Inspeccionar red
docker network inspect app-net

# Listar volúmenes
docker volume ls

# Ver estadísticas de uso
docker stats

# Verificar configuración de seguridad
docker inspect <container_name> | grep -A 20 "SecurityOpt"
```

Estos ejercicios proporcionan una experiencia práctica completa con Docker, cubriendo los aspectos más importantes de:
- Construcción de imágenes
- Redes
- Almacenamiento
- Seguridad
