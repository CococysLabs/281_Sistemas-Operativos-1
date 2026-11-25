# Ejercicios Prácticos de Kubernetes

## 1. ConfigMaps y Secrets

### Ejercicio 1.1: Configuración de una aplicación usando ConfigMaps
**Objetivo**: Crear un ConfigMap para una aplicación web y montarlo en un Pod.

**Pasos**:
1. Crear un ConfigMap con variables de configuración para una aplicación web:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
data:
  DB_HOST: "mysql-service"
  DB_PORT: "3306"
  APP_COLOR: "blue"
  APP_MODE: "production"
```

2. Crear un Pod que use el ConfigMap:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp
    image: nginx
    envFrom:
    - configMapRef:
        name: webapp-config
```

3. Verificar la configuración:
```bash
kubectl exec webapp-pod -- env | grep APP_
```

### Ejercicio 1.2: Manejo de información sensible con Secrets
**Objetivo**: Crear y utilizar Secrets para credenciales de base de datos.

**Pasos**:
1. Crear un Secret con credenciales:
```bash
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=secretpass123
```

2. Crear un Pod que use el Secret:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  containers:
  - name: secure-app
    image: nginx
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

## 2. Persistent Volumes y Persistent Volume Claims

### Ejercicio 2.1: Creación y uso de PV y PVC
**Objetivo**: Crear un PV, reclamarlo con un PVC y usarlo en un Pod.

**Pasos**:
1. Crear un Persistent Volume:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/pv
```

2. Crear un Persistent Volume Claim:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

3. Crear un Pod que use el PVC:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
  - name: task-pv-container
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: data-volume
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: data-pvc
```

## 3. Helm

### Ejercicio 3.1: Creación de un Chart personalizado
**Objetivo**: Crear un Chart de Helm básico para desplegar una aplicación web.

**Pasos**:
1. Crear un nuevo Chart:
```bash
helm create mychart
```

2. Modificar values.yaml para personalizar la configuración:
```yaml
# values.yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
service:
  type: NodePort
  port: 80
```

3. Modificar el template de deployment (templates/deployment.yaml):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 80
```

4. Instalar el Chart:
```bash
helm install myapp ./mychart
```

### Ejercicio 3.2: Trabajando con repositorios de Helm
**Objetivo**: Instalar y configurar una aplicación desde un repositorio oficial.

**Pasos**:
1. Agregar el repositorio de Bitnami:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

2. Crear un archivo de valores personalizados (wordpress-values.yaml):
```yaml
wordpressUsername: admin
wordpressPassword: secretpass
service:
  type: NodePort
persistence:
  enabled: true
  size: 1Gi
```

3. Instalar WordPress usando el Chart de Bitnami:
```bash
helm install my-wordpress bitnami/wordpress -f wordpress-values.yaml
```

**Verificación**:
```bash
helm list
kubectl get all -l "app.kubernetes.io/instance=my-wordpress"
```