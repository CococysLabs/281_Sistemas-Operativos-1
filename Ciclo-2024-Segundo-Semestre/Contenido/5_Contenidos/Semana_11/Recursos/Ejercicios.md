# Ejercicios Prácticos Resueltos de Google Cloud Platform

## 1. Instancias y Discos en Compute Engine

### Ejercicio 1.1: Crear y Configurar una Instancia Web

```bash
# 1. Crear una instancia con disco de arranque personalizado
gcloud compute instances create web-server-1 \
    --zone=us-central1-a \
    --machine-type=e2-medium \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --boot-disk-size=20GB \
    --boot-disk-type=pd-balanced \
    --tags=http-server,https-server

# 2. Configurar servidor web
gcloud compute ssh web-server-1 --zone=us-central1-a --command '
sudo apt-get update && \
sudo apt-get install -y apache2 && \
echo "<!DOCTYPE html>
<html><body><h1>¡Bienvenido a mi servidor web!</h1></body></html>" | \
sudo tee /var/www/html/index.html'

# 3. Crear regla de firewall para HTTP
gcloud compute firewall-rules create allow-http \
    --direction=INGRESS \
    --priority=1000 \
    --network=default \
    --action=ALLOW \
    --rules=tcp:80 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=http-server
```

### Ejercicio 1.2: Gestión de Discos Persistentes

```bash
# 1. Crear disco persistente adicional
gcloud compute disks create data-disk-1 \
    --zone=us-central1-a \
    --size=50GB \
    --type=pd-balanced

# 2. Adjuntar disco a la instancia
gcloud compute instances attach-disk web-server-1 \
    --disk=data-disk-1 \
    --zone=us-central1-a \
    --device-name=data-disk-1

# 3. Formatear y montar el disco
gcloud compute ssh web-server-1 --zone=us-central1-a --command '
sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/disk/by-id/google-data-disk-1 && \
sudo mkdir -p /mnt/disks/data && \
sudo mount -o discard,defaults /dev/disk/by-id/google-data-disk-1 /mnt/disks/data && \
sudo chmod a+w /mnt/disks/data'

# 4. Configurar montaje automático
gcloud compute ssh web-server-1 --zone=us-central1-a --command '
echo UUID=$(sudo blkid -s UUID -o value /dev/disk/by-id/google-data-disk-1) /mnt/disks/data ext4 discard,defaults,nofail 0 2 | \
sudo tee -a /etc/fstab'
```

### Ejercicio 1.3: Snapshots y Respaldos

```bash
# 1. Crear snapshot del disco de datos
gcloud compute disks snapshot data-disk-1 \
    --snapshot-names=data-backup-1 \
    --zone=us-central1-a \
    --storage-location=us-central1

# 2. Crear nuevo disco desde snapshot
gcloud compute disks create data-disk-2 \
    --source-snapshot=data-backup-1 \
    --zone=us-central1-a

# 3. Programar snapshots automáticos
gcloud compute resource-policies create snapshot-schedule daily-backup \
    --region=us-central1 \
    --snapshot-schedule="0 */24 * * *" \
    --max-retention-days=14

# 4. Adjuntar política al disco
gcloud compute disks add-resource-policies data-disk-1 \
    --resource-policies=daily-backup \
    --zone=us-central1-a
```

## 2. Redes y Firewalls en Google Cloud

### Ejercicio 2.1: Crear VPC Personalizada

```bash
# 1. Crear red VPC personalizada
gcloud compute networks create my-custom-network \
    --subnet-mode=custom \
    --bgp-routing-mode=regional

# 2. Crear subredes
gcloud compute networks subnets create subnet-us-central1 \
    --network=my-custom-network \
    --region=us-central1 \
    --range=10.0.1.0/24 \
    --secondary-range=services=10.0.2.0/24,pods=10.0.3.0/24

gcloud compute networks subnets create subnet-europe-west1 \
    --network=my-custom-network \
    --region=europe-west1 \
    --range=10.1.1.0/24

# 3. Crear instancias en diferentes subredes
gcloud compute instances create vm-us \
    --zone=us-central1-a \
    --machine-type=e2-medium \
    --network=my-custom-network \
    --subnet=subnet-us-central1

gcloud compute instances create vm-eu \
    --zone=europe-west1-b \
    --machine-type=e2-medium \
    --network=my-custom-network \
    --subnet=subnet-europe-west1
```

### Ejercicio 2.2: Configuración de Firewalls

```bash
# 1. Regla de firewall para SSH interno
gcloud compute firewall-rules create allow-internal-ssh \
    --network=my-custom-network \
    --direction=INGRESS \
    --priority=1000 \
    --action=ALLOW \
    --rules=tcp:22 \
    --source-ranges=10.0.0.0/8

# 2. Regla para ICMP interno
gcloud compute firewall-rules create allow-internal-icmp \
    --network=my-custom-network \
    --direction=INGRESS \
    --priority=1000 \
    --action=ALLOW \
    --rules=icmp \
    --source-ranges=10.0.0.0/8

# 3. Regla para aplicación web
gcloud compute firewall-rules create allow-web \
    --network=my-custom-network \
    --direction=INGRESS \
    --priority=1000 \
    --action=ALLOW \
    --rules=tcp:80,tcp:443 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=web-server
```

### Ejercicio 2.3: Cloud NAT y Cloud Router

```bash
# 1. Crear Cloud Router
gcloud compute routers create nat-router \
    --network=my-custom-network \
    --region=us-central1

# 2. Configurar Cloud NAT
gcloud compute routers nats create nat-config \
    --router=nat-router \
    --region=us-central1 \
    --nat-all-subnet-ip-ranges \
    --auto-allocate-nat-external-ips

# 3. Crear instancia privada sin IP externa
gcloud compute instances create private-instance \
    --zone=us-central1-a \
    --machine-type=e2-medium \
    --network=my-custom-network \
    --subnet=subnet-us-central1 \
    --no-address
```

### Ejercicio 2.4: Load Balancer HTTP(S)

```bash
# 1. Crear grupo de instancias
gcloud compute instance-templates create web-template \
    --machine-type=e2-medium \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --tags=web-server \
    --metadata=startup-script='#! /bin/bash
        apt-get update
        apt-get install -y apache2
        echo "<!DOCTYPE html><html><body><h1>Servidor $(hostname)</h1></body></html>" > /var/www/html/index.html'

# 2. Crear grupo de instancias administrado
gcloud compute instance-groups managed create web-group \
    --template=web-template \
    --size=2 \
    --zone=us-central1-a

# 3. Configurar reglas de salud
gcloud compute health-checks create http web-health-check \
    --port=80

# 4. Crear backend service
gcloud compute backend-services create web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=web-health-check \
    --global

# 5. Añadir grupo de instancias al backend
gcloud compute backend-services add-backend web-backend-service \
    --instance-group=web-group \
    --instance-group-zone=us-central1-a \
    --global

# 6. Crear URL map
gcloud compute url-maps create web-map \
    --default-service=web-backend-service

# 7. Crear frontend
gcloud compute forwarding-rules create web-frontend \
    --ports=80 \
    --global \
    --target-http-proxy=web-proxy

# 8. Crear proxy HTTP
gcloud compute target-http-proxies create web-proxy \
    --url-map=web-map
```

## Comandos Útiles para Verificación

```bash
# Listar instancias
gcloud compute instances list

# Ver detalles de una instancia
gcloud compute instances describe web-server-1 --zone=us-central1-a

# Listar discos
gcloud compute disks list

# Ver reglas de firewall
gcloud compute firewall-rules list

# Verificar redes VPC
gcloud compute networks list

# Ver subredes
gcloud compute networks subnets list

# Verificar NAT
gcloud compute routers nats describe nat-config \
    --router=nat-router \
    --region=us-central1
```

Estos ejercicios proporcionan una experiencia práctica completa con Google Cloud Platform, cubriendo:
- Gestión de instancias y discos
- Configuración de redes VPC
- Implementación de firewalls
- Configuración de NAT y Load Balancers

