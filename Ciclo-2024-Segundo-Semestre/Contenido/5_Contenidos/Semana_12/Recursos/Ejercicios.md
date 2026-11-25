# Ejercicios Prácticos de Kubernetes y GKE

## 1. Objetos Fundamentales de Kubernetes

### Ejercicio 1.1: Trabajando con Pods

```yaml
# nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

```bash
# Crear y gestionar el pod
kubectl apply -f nginx-pod.yaml
kubectl get pod nginx-pod
kubectl describe pod nginx-pod
kubectl port-forward nginx-pod 8080:80
curl localhost:8080
```

### Ejercicio 1.2: Implementación de ReplicaSet

```yaml
# nginx-replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

```bash
# Gestionar ReplicaSet
kubectl apply -f nginx-replicaset.yaml
kubectl get rs nginx-rs
kubectl get pods -l app=nginx
kubectl scale rs/nginx-rs --replicas=5
```

### Ejercicio 1.3: Deployments y Actualizaciones

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

```bash
# Gestionar Deployment
kubectl apply -f nginx-deployment.yaml
kubectl get deployment nginx-deployment
kubectl rollout status deployment/nginx-deployment

# Actualizar imagen
kubectl set image deployment/nginx-deployment nginx=nginx:1.22
kubectl rollout history deployment/nginx-deployment

# Rollback si es necesario
kubectl rollout undo deployment/nginx-deployment
```

## 2. Services en Kubernetes

### Ejercicio 2.1: ClusterIP Service

```yaml
# nginx-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f nginx-clusterip.yaml
kubectl get service nginx-clusterip
kubectl describe service nginx-clusterip
```

### Ejercicio 2.2: NodePort Service

```yaml
# nginx-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

```bash
kubectl apply -f nginx-nodeport.yaml
kubectl get service nginx-nodeport
```

### Ejercicio 2.3: LoadBalancer Service

```yaml
# nginx-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f nginx-loadbalancer.yaml
kubectl get service nginx-lb
```

## 3. Google Kubernetes Engine (GKE)

### Ejercicio 3.1: Crear Cluster GKE

```bash
# Crear cluster estándar
gcloud container clusters create my-cluster \
    --zone us-central1-a \
    --num-nodes 3 \
    --machine-type e2-medium \
    --enable-autoscaling \
    --min-nodes 1 \
    --max-nodes 5

# Obtener credenciales
gcloud container clusters get-credentials my-cluster \
    --zone us-central1-a

# Verificar nodos
kubectl get nodes
```

### Ejercicio 3.2: Despliegue de Aplicación Web

```yaml
# web-application.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

```bash
kubectl apply -f web-application.yaml
kubectl get deployment web-app
kubectl get service web-service
```

### Ejercicio 3.3: Configuración de Autoscaling

```yaml
# autoscaling.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

```bash
kubectl apply -f autoscaling.yaml
kubectl get hpa web-app-hpa
```

### Ejercicio 3.4: Aplicación Multi-tier

```yaml
# multi-tier-app.yaml
# Frontend
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx:1.21
        ports:
        - containerPort: 80
---
# Backend
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
---
# Frontend Service
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
---
# Backend Service
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
```

```bash
# Desplegar aplicación multi-tier
kubectl apply -f multi-tier-app.yaml

# Verificar despliegues
kubectl get deployments
kubectl get services

# Escalar componentes
kubectl scale deployment frontend --replicas=5
kubectl scale deployment backend --replicas=3
```

## Comandos Útiles para Monitoreo

```bash
# Ver todos los recursos
kubectl get all

# Verificar logs
kubectl logs deployment/web-app

# Ver detalles del cluster
kubectl cluster-info

# Monitorear nodos
kubectl top nodes

# Monitorear pods
kubectl top pods

# Verificar eventos
kubectl get events --sort-by=.metadata.creationTimestamp

# Estado de los servicios
kubectl get services -o wide
```

## Limpieza de Recursos

```bash
# Eliminar servicios
kubectl delete service frontend-service backend-service

# Eliminar deployments
kubectl delete deployment frontend backend

# Eliminar cluster GKE
gcloud container clusters delete my-cluster --zone us-central1-a
```

Estos ejercicios proporcionan una experiencia práctica completa con Kubernetes y GKE, cubriendo:
- Objetos básicos de Kubernetes
- Servicios y exposición de aplicaciones
- Gestión de clusters GKE
- Despliegue de aplicaciones
- Configuración de autoscaling
- Aplicaciones multi-tier
