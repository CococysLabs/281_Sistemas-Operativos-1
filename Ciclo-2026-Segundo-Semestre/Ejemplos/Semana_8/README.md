# Semana 8: API Rust en GKE

Este ejemplo despliega una API HTTP escrita en Rust sobre Google Kubernetes
Engine (GKE). El tráfico llega mediante un Gateway API y un `HTTPRoute`. El
HorizontalPodAutoscaler aumenta el número de pods cuando el uso de CPU supera
el 30 por ciento del valor solicitado.

## Requisitos

Instala y configura:

- Google Cloud CLI (`gcloud`), `kubectl` y `gke-gcloud-auth-plugin`.
- Docker con una cuenta en un registro de imágenes.
- Python 3 y Locust para la prueba de carga.

La API escucha en el puerto `8080`. Sus rutas son:

- `GET /`: responde `OK` y se usa para comprobar la salud del servicio.
- `POST /grpc-2026`: recibe un reporte JSON y ejecuta una operación de CPU.

## Crear el clúster

Define el proyecto y la zona que vas a utilizar:

```bash
gcloud auth login
gcloud config set project TU_PROJECT_ID
gcloud config set compute/zone us-central1-a
gcloud services enable container.googleapis.com compute.googleapis.com logging.googleapis.com
gcloud components install gke-gcloud-auth-plugin
```

Crea el clúster con Gateway API habilitado:

```bash
gcloud container clusters create mumnk8s-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type e2-standard-2 \
  --gateway-api standard \
  --disk-size=50

gcloud container clusters get-credentials mumnk8s-cluster \
  --zone us-central1-a
kubectl get nodes
```

## Construir y publicar la imagen

Desde `Semana8`, reemplaza `TU_USUARIO_DOCKER` por tu usuario del registro:

```bash
docker build -t TU_USUARIO_DOCKER/api-rust:v1 api-rust
docker push TU_USUARIO_DOCKER/api-rust:v1
```

Edita `k8s/02-deployment.yaml` y coloca el mismo nombre de imagen en el campo
`spec.template.spec.containers[0].image`.

## Aplicar los manifiestos

```bash
kubectl apply -f k8s/
kubectl get pods -n mumnk8s
kubectl get hpa -n mumnk8s
```

El Gateway puede tardar varios minutos en recibir una dirección pública:

```bash
kubectl get gateway api-gateway -n mumnk8s -w
```

Antes de usar el balanceador, prueba el servicio con un reenvío local:

```bash
kubectl port-forward -n mumnk8s svc/api-rust-service 8080:80
```

En otra terminal:

```bash
curl http://localhost:8080/
curl -X POST http://localhost:8080/grpc-2026 \
  -H 'Content-Type: application/json' \
  -d '{"country":"USA","warplanes_in_air":10,"warships_in_water":5,"timestamp":"2026-03-12T20:15:30Z"}'
```

Cuando `kubectl get gateway` muestre una dirección en `ADDRESS`, usa esa
dirección para probar la API:

```bash
curl http://DIRECCION_DEL_GATEWAY/
```

## Ejecutar la prueba de carga

Instala Locust en un entorno virtual y ejecuta el archivo de prueba:

```bash
python3 -m venv test/venv
test/venv/bin/pip install locust
test/venv/bin/locust -f test/locustfile.py \
  --host http://DIRECCION_DEL_GATEWAY
```

Abre `http://localhost:8089` para iniciar la prueba. En otra terminal observa
el escalamiento:

```bash
kubectl get hpa -n mumnk8s -w
kubectl get pods -n mumnk8s -w
```

## Eliminar los recursos

Cuando termines, elimina los manifiestos y el clúster para evitar cargos:

```bash
kubectl delete -f k8s/
gcloud container clusters delete mumnk8s-cluster \
  --zone us-central1-a --quiet
```
