# Ejercicio Práctico: Exploración de Virtualización y MicroVMs

## Objetivos del ejercicio
- Comprender las diferencias entre virtualización tradicional y MicroVMs
- Implementar diferentes tipos de virtualización
- Crear y gestionar MicroVMs
- Comparar rendimiento entre soluciones

## Parte 1: Virtualización Tradicional con VirtualBox

### 1.1 Configuración inicial
```bash
# Instalar VirtualBox
sudo apt update
sudo apt install virtualbox

# Crear una VM tradicional
VBoxManage createvm --name "VM-Traditional" --ostype "Ubuntu_64" --register
VBoxManage modifyvm "VM-Traditional" --memory 2048 --cpus 2
VBoxManage createhd --filename "VM-Traditional.vdi" --size 20000
```

### 1.2 Medición de recursos
```bash
# Medir tiempo de inicio
time VBoxManage startvm "VM-Traditional"

# Monitorear uso de recursos
top -p $(pgrep VBoxHeadless)
```

## Parte 2: Contenedores con Docker

### 2.1 Implementación básica
```bash
# Instalar Docker
sudo apt install docker.io

# Crear un contenedor simple
cat << EOF > Dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

docker build -t nginx-container .
docker run -d -p 8080:80 nginx-container
```

## Parte 3: MicroVMs con Firecracker

### 3.1 Configuración de Firecracker
```bash
# Instalar Firecracker
wget https://github.com/firecracker-microvm/firecracker/releases/download/v1.3.0/firecracker-v1.3.0-x86_64
chmod +x firecracker-v1.3.0-x86_64
sudo mv firecracker-v1.3.0-x86_64 /usr/local/bin/firecracker

# Preparar kernel y rootfs
wget https://s3.amazonaws.com/spec.ccfc.min/img/hello/kernel/hello-vmlinux.bin
wget https://s3.amazonaws.com/spec.ccfc.min/img/hello/fsfiles/hello-rootfs.ext4
```

### 3.2 Crear y ejecutar MicroVM
```bash
# Configuración de MicroVM
cat << EOF > config.json
{
  "boot-source": {
    "kernel_image_path": "hello-vmlinux.bin",
    "boot_args": "console=ttyS0 reboot=k panic=1"
  },
  "drives": [
    {
      "drive_id": "rootfs",
      "path_on_host": "hello-rootfs.ext4",
      "is_root_device": true,
      "is_read_only": false
    }
  ],
  "machine-config": {
    "vcpu_count": 2,
    "mem_size_mib": 1024
  }
}
EOF

# Iniciar MicroVM
firecracker --config-file config.json
```

## Parte 4: Análisis Comparativo

### 4.1 Comparar métricas
```bash
# Script para medir tiempos de inicio
#!/bin/bash
echo "Midiendo tiempo de inicio VM tradicional..."
time VBoxManage startvm "VM-Traditional"

echo "Midiendo tiempo de inicio contenedor Docker..."
time docker run nginx-container

echo "Midiendo tiempo de inicio MicroVM..."
time firecracker --config-file config.json
```

### 4.2 Tabla de comparación

| Métrica                | VM Tradicional | Contenedor | MicroVM |
|------------------------|----------------|------------|---------|
| Tiempo de inicio       | ~30s           | ~1s        | ~125ms  |
| Uso de memoria base   | ~512MB         | ~50MB      | ~5MB    |
| Aislamiento           | Alto           | Medio      | Alto    |
| Densidad de instancias| Baja           | Alta       | Alta    |

## Parte 5: Ejercicios Prácticos

1. Modifica la configuración de la MicroVM para usar diferentes cantidades de memoria y CPU
2. Implementa un servicio web en cada tipo de virtualización y compara el rendimiento
3. Realiza pruebas de carga en cada implementación
4. Documenta las diferencias en seguridad y aislamiento

## Notas Importantes
- Asegúrate de tener privilegios de administrador para todos los comandos
- Los recursos asignados pueden necesitar ajuste según tu hardware
- Mantén un registro de todas las métricas para comparación
- Considera aspectos de seguridad al implementar cada solución
