# Semana 10: alertas de Falco

Este ejemplo instala Falco en un clúster de Kubernetes y agrega una regla
personalizada. La regla genera una alerta de prioridad `CRITICAL` cuando un
proceso dentro de un contenedor lee un archivo ubicado en `/etc/tmp`.

## Requisitos

- Un clúster de Kubernetes con permisos para crear un DaemonSet privilegiado.
- `kubectl` configurado para apuntar al clúster.
- Helm 3.

Falco necesita observar llamadas al sistema en los nodos. Por esa razón, su
instalación requiere permisos elevados y debe hacerse únicamente en un
clúster de pruebas o en uno donde esta configuración esté autorizada.

## Archivos

- `falco-rules.yaml`: valores de Helm con las reglas personalizadas.
- `test-pod.yaml`: pod que crea un archivo en `/etc/tmp` y lo lee de forma
  periódica para generar la alerta.

## Instalar Falco

Agrega el repositorio y actualiza su índice:

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
```

Instala Falco en un namespace propio:

```bash
helm upgrade --install falco falcosecurity/falco \
  --namespace falco \
  --create-namespace \
  --values falco-rules.yaml
```

Espera a que el DaemonSet esté listo:

```bash
kubectl get pods -n falco -w
```

## Generar la alerta

Aplica el pod de prueba:

```bash
kubectl apply -f test-pod.yaml
kubectl get pod falco-read-test -w
```

El contenedor escribe primero en `/etc/tmp/archivo-prueba` y luego lo lee
cada diez segundos. Revisa los logs de Falco:

```bash
kubectl logs -n falco -l app.kubernetes.io/name=falco --tail=100 -f
```

La salida debe incluir el texto `CRITICAL ALERT: Read from /etc/tmp in
container`, el nombre del pod y el archivo que fue leído.

También se puede comprobar la actividad del pod:

```bash
kubectl logs falco-read-test
```

## Revisar y eliminar el ejemplo

Para inspeccionar la regla cargada:

```bash
kubectl exec -n falco ds/falco -- falco --list
```

Si el nombre del DaemonSet cambia según la versión del chart, obtén el nombre
con `kubectl get daemonsets -n falco` y sustituye `ds/falco` en el comando.

Al finalizar, elimina el pod de prueba y Falco:

```bash
kubectl delete -f test-pod.yaml
helm uninstall falco --namespace falco
kubectl delete namespace falco
```

## Consideraciones sobre la regla

La condición usa `open_read` y filtra `fd.name` con `startswith`. Por lo
tanto, detecta la lectura de archivos dentro de `/etc/tmp`, no solamente una
apertura del directorio. La regla se limita a eventos de contenedores para
evitar alertas de procesos del sistema que no pertenezcan a un pod.
