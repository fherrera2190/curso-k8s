# Curso Kubernetes - Comandos y Conceptos

## Conceptos Fundamentales

### Namespaces de Linux

- **IPC (Inter-Process Communication)**: Sirve para compartir información entre procesos. Los procesos de un contenedor no pueden ver los procesos de otro contenedor.
- **CGROUPS (Control Groups)**: Sirve para limitar los recursos de los contenedores (RAM, CPU, I/O).
- **Network**: Cada contenedor tiene su propia network namespace con una IP única.
- **Mount**: Cada contenedor tiene su propio filesystem. El mount permite que los contenedores compartan archivos entre sí, a menos que se compartan volumes.
- **PID**: Cada proceso tiene su propio ID de proceso.
- **USER**: Namespace de usuario.
- **UTS (Unix Time Sharing)**: Cada contenedor tiene su propio hostname.

### Kubernetes: El Pod

En Kubernetes la unidad mínima de ejecución es el **Pod**.

**¿Por qué?**
Porque un pod puede tener múltiples contenedores y estos comparten recursos (IPC, Network y UTS).

### LXC (Linux Containers)

Antes de Docker existía LXC. Cada contenedor es un LXC y cada uno tiene:
- Su propia red
- Su propio filesystem
- Su propio proceso
- Su propio usuario
- Su propio hostname
- Su propio mount
- Su propio PID
- Su propio IPC
- Su propio cgroup
- Su propio network namespace
- Su propio user namespace
- Su propio UTS namespace
- Su propio cgroup namespace
- Su propio time namespace

A raíz de esto nació Docker e incluyó bridge network.

### Pod en Kubernetes

El Pod es compartir namespace entre contenedores. Kubernetes comparte IPC, Network y UTS.


## Comandos Básicos

### Minikube
```bash
minikube status
minikube start
```

### Kubectl - Pods
```bash
# Listar pods
kubectl get pods
kubectl get po  # shortname

# Crear pod por CLI
kubectl run name --image=nginx:alpine  # se puede agregar puerto, sino por defecto 8080

# Describir pod
kubectl describe pod name

# Ver todos los recursos disponibles
kubectl api-resources

# Borrar pods
kubectl delete pod name name2 name3

# Ver pod específico
kubectl get pod name

# Obtener manifest
kubectl get pod name -o yaml
kubectl get pod name -o json

# Port forwarding
kubectl port-forward <pod-name> 7000:<pod-port>
# nginx:alpine por defecto usa el puerto 80
kubectl port-forward <pod-name> 7000:80
```

### Docker
```bash
docker ps -l
docker ps -f=name=gcr.io/k8s-minikube/kicbase:v0.0.48
```

## Video 32 - Acceder a Terminal del Contenedor

```bash
kubectl exec -ti podtest -- sh
cd /usr/share/nginx/html/
cat index.html
echo ":)" > index.html
cat index.html
```

## Video 33 - Logs

```bash
kubectl logs podtest

# Ver logs en vivo
kubectl logs -f podtest
```

## Video 34 - YAML y API Versions

```bash
kubectl api-versions

# En la carpeta kubernetes/pods
kubectl apply -f pod.yaml
kubectl delete -f pod.yaml
```

## Video 35 - Múltiples Contenedores

### Docker
```bash
# Linux
docker run --net host -ti python:3.6-alpine sh

# Windows
docker run -p 8081:8081 -ti python:3.6-alpine sh

# Dentro del contenedor
python -m http.server 8081
```

### Kubernetes
```bash
kubectl apply -f doscontenedores.yaml
kubectl events pod doscontenedores
kubectl logs doscontenedores -c cont1
kubectl logs doscontenedores -c cont2
```

## Video 36 - Comunicación entre Contenedores

```bash
kubectl logs doscontenedores -c cont2
kubectl exec -ti doscontenedores -- sh

# Instalar curl
apk add -U curl
curl localhost:8082

# Recrear pod
kubectl delete -f doscontenedores.yaml
kubectl apply -f doscontenedores.yaml
```

### Seleccionar Contenedor Específico
```bash
# Seleccionar default
kubectl exec -it doscontenedores -- sh

# Seleccionar contenedor específico
kubectl exec -it doscontenedores -c cont1 -- sh

apk add -U curl
curl localhost:8082
curl localhost:8083
```

## Video 37 - Labels

```bash
kubectl apply -f labels.yaml
kubectl get pods -l app=front
kubectl get pods -l app=back
kubectl get pods -l env=dev
```

## Video 38 - Problemas de los Pods

**Problemas:**
- Los pods no hacen self healing
- Los pods no pueden actualizarse a sí mismos

```bash
kubectl apply -f test.yaml
```

**Solución:** Crear objeto ReplicaSet

## Video 39 - ReplicaSet: Cómo Funciona

Es un objeto separado al pod que está a un nivel más alto, el cual crea pods y los gestiona.

- Se define un template y la cantidad de réplicas que quiero
- Si un pod muere, el replicaset lo reemplaza
- Los labels son importantes para que el replicaset pueda identificar los pods
- Cuando el replicaset crea los pods, va a colocar en la metadata del pod **owner reference** que es el replicaset que lo creó
- El owner lo coloca el objeto de más alto nivel

## Video 40 - ReplicaSet: Estructura

```yaml
apiVersion: apps/v1
```

**¿Por qué API Group es apps?**
- El `kind` es el tipo de objeto

```bash
kubectl api-resources
kubectl apply -f rs.yaml
```

## Video 41 - ReplicaSet: Eliminar Pods

```bash
kubectl delete pod XXXXX
kubectl apply -f rs.yaml
```

## Video 42 - ReplicaSet: Inspección

```bash
kubectl describe rs rs -o yaml
kubectl get rs -o yaml
kubectl get pod rs-test-7mxsr -o yaml
```

## Video 43 - ReplicaSet: Adopción de Pods

```bash
kubectl delete -f rs.yaml

kubectl run podtest1 --image=nginx:alpine
kubectl run podtest2 --image=nginx:alpine

kubectl label pods podtest1 app=pod-label
kubectl label pods podtest2 app=pod-label

kubectl apply -f rs.yaml
```

El replicaset adopta los pods existentes.

```bash
kubectl delete pod podtest2
```

## Video 44 - Problemas del ReplicaSet

**Problema:** ReplicaSet debe mantener la cantidad de pods especificada. ¿Qué pasa si quiero cambiar la imagen o hacer cambios en el pod?

**Limitación:**
- El replicaset solo mira el número de pods y el label
- Si se requieren cambios, no se puede editar el replicaset, se debe eliminar y crear uno nuevo

```bash
kubectl get pods
kubectl get rs rs-test -o yaml
kubectl describe pod rs-test-f85zp
```

**Conclusión:** ReplicaSet no sirve para actualizar los pods, pero sí podría eliminarlos y volver a crear.

**Solución:** El objeto de mayor nivel es el **Deployment**

```
deployment -> replicaset -> pods
```


## Video 46 - Deployment

```bash
# Muestra todos los recursos disponibles
kubectl api-resources

kubectl apply -f dep.yaml

kubectl get deployment

# Muestra los deployments con sus labels
kubectl get deployment --show-labels

# Sirve para ver el estado del rollout
kubectl rollout status deployment deployment-test

# Muestra el historial de versiones del deployment
kubectl rollout history deployment deployment-test
```

## Video 47 - Inspección de Deployment

### Ver ReplicaSets
```bash
kubectl get rs
```
**Output:**
```
NAME                         DESIRED   CURRENT   READY   AGE
deployment-test-65b49fdf48   3         3         3       6m35s
```

### Ver ReplicaSets con Labels
```bash
kubectl get rs --show-labels
```
**Output:**
```
NAME                         DESIRED   CURRENT   READY   AGE     LABELS
deployment-test-65b49fdf48   3         3         3       6m53s   app=front,pod-template-hash=65b49fdf48
```

### Ver Pods con Labels
```bash
kubectl get pods --show-labels
```
**Output:**
```
NAME                               READY   STATUS    RESTARTS   AGE     LABELS
deployment-test-65b49fdf48-2d5wh   1/1     Running   0          7m32s   app=front,pod-template-hash=65b49fdf48
deployment-test-65b49fdf48-d9lnv   1/1     Running   0          7m32s   app=front,pod-template-hash=65b49fdf48
deployment-test-65b49fdf48-d9zzq   1/1     Running   0          7m32s   app=front,pod-template-hash=65b49fdf48
```

### Ver Deployment YAML
```bash
kubectl get deployment deployment-test -o yaml
```

**Output:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"front"},"name":"deployment-test","namespace":"default"},"spec":{"replicas":3,"selector":{"matchLabels":{"app":"front"}},"template":{"metadata":{"labels":{"app":"front"}},"spec":{"containers":[{"image":"nginx:alpine","name":"nginx","ports":[{"containerPort":80}]}]}}}}
  creationTimestamp: "2026-07-26T03:05:53Z"
  generation: 1
  labels:
    app: front
  name: deployment-test
  namespace: default
  resourceVersion: "6660"
  uid: f2d2953d-322a-4108-a724-7f0a22c58f71
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: front
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: front
    spec:
      containers:
      - image: nginx:alpine
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 3
  conditions:
  - lastTransitionTime: "2026-07-26T03:06:38Z"
    lastUpdateTime: "2026-07-26T03:06:38Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2026-07-26T03:05:53Z"
    lastUpdateTime: "2026-07-26T03:06:38Z"
    message: ReplicaSet "deployment-test-65b49fdf48" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 3
  replicas: 3
  terminatingReplicas: 0
  updatedReplicas: 3
```

### Ver ReplicaSet YAML
```bash
kubectl get rs deployment-test-65b49fdf48 -o yaml
```

**Output:**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  annotations:
    deployment.kubernetes.io/desired-replicas: "3"
    deployment.kubernetes.io/max-replicas: "4"
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2026-07-26T03:05:53Z"
  generation: 1
  labels:
    app: front
    pod-template-hash: 65b49fdf48
  name: deployment-test-65b49fdf48
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: Deployment
    name: deployment-test
    uid: f2d2953d-322a-4108-a724-7f0a22c58f71
  resourceVersion: "6659"
  uid: 7fb05a12-29f1-49a0-9d92-78e0dffaabbc
spec:
  replicas: 3
  selector:
    matchLabels:
      app: front
      pod-template-hash: 65b49fdf48
  template:
    metadata:
      labels:
        app: front
        pod-template-hash: 65b49fdf48
    spec:
      containers:
      - image: nginx:alpine
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 3
  fullyLabeledReplicas: 3
  observedGeneration: 1
  readyReplicas: 3
  replicas: 3
  terminatingReplicas: 0
```

### Ver Pod YAML
```bash
kubectl get pod deployment-test-65b49fdf48-2d5wh -o yaml
```

**Output:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-07-26T03:05:53Z"
  generateName: deployment-test-65b49fdf48-
  generation: 1
  labels:
    app: front
    pod-template-hash: 65b49fdf48
  name: deployment-test-65b49fdf48-2d5wh
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: deployment-test-65b49fdf48
    uid: 7fb05a12-29f1-49a0-9d92-78e0dffaabbc
  resourceVersion: "6647"
  uid: e39b1444-5aa4-43b8-82d5-094431b692f1
spec:
  containers:
  - image: nginx:alpine
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-5gsnb
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-5gsnb
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-07-26T03:06:33Z"
    observedGeneration: 1
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-07-26T03:05:54Z"
    observedGeneration: 1
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-07-26T03:06:33Z"
    observedGeneration: 1
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-07-26T03:06:33Z"
    observedGeneration: 1
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-07-26T03:05:53Z"
    observedGeneration: 1
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://54f599fcd6acb78576382fd9ab4b838816399317114c9df34af13f30d386a8a6
    image: nginx:alpine
    imageID: docker-pullable://nginx@sha256:4a73073bd557c65b759505da037898b61f1be6cbcc3c2c3aeac22d2a470c1752
    lastState: {}
    name: nginx
    ready: true
    resources: {}
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2026-07-26T03:06:30Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-5gsnb
      readOnly: true
    recursiveReadOnly: Disabled
  hostIP: 192.168.49.2
  hostIPs:
  - ip: 192.168.49.2
  observedGeneration: 1
  phase: Running
  podIP: 10.244.0.3
  podIPs:
  - ip: 10.244.0.3
  qosClass: BestEffort
  startTime: "2026-07-26T03:05:54Z"
```

## Video 48

kubectl rollout status deployment deployment-test

## Video 49

$ kubectl get rs -l app=front
NAME                         DESIRED   CURRENT   READY   AGE
deployment-test-6c64f866cb   3         3         3       44m
deployment-test-7784486fc7   0         0         0       42m
deployment-test-78bbbbc89b   0         0         0       83m
deployment-test-c564f8455    0         0         0       86m

$ kubectl rollout history deployment deployment-test
deployment.apps/deployment-test 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
5         <none>
6         <none>

## Video 50

### 3 formas de dejar comentarios en un kubectl rollout status deployment deployment-test al realizar un rollout 

1- kubectl apply -f dep.yaml --record 

2- Agregar a archivo dep.yaml

```yaml
  annotations:
    kubernetes.io/change-cause: "Change port to 101"
```
3- kubectl annotate deployment deployment-test kubernetes.io/change-cause="Change port to 101"

kubectl rollout history deployment deployment-test --revision=1

## Video 51 

### Rollback

kubectl rollout undo deployment deployment-test --to-revision=3

### Verificar el rollback
kubectl describe deployments.apps deployment-test

## Video 54 Services

### Los servicios son una forma de exporar aplicaciones dentro del cluster, la ip de los servicios no cambian

### Los endpoints guardan la información de los pods que están ejecutando la aplicación IP

### Los servicios se crean con el comando kubectl expose

kubectl apply -f service.yml

kubectl get svc

kubectl port-forward svc/mi-servicio 8080:8080

# Ejecutar en otra terminal desde toolbox para alt-services
for i in $(seq 1 100000); do wget -qO- --header="Connection: close" http://nginx-balance-svc; done

# Video 55

kubectl get svc

kubectl get svc -l app=front

kubectl describe svc mi-servicio

# Video 56

kubectl get ep mi-servicio

kubectl get pods -l app=front

kubectl get pods -l app=front -o wide

kubectl delete pod deployment-test-7784486fc7-9ccsc

kubectl run podtest2 --image=nginx:alpine

kubectl get ep mi-servicio -o wide

kubectl describe ep mi-servicio

# Video 57

kubectl exec -it toolbox-deployment-59f55f59f-lvjsc --sh

### dns

curl mi-servicio:8080

# Video 58

ClusterIP es el tipo de servicio que expone la aplicación dentro del cluster
esta ip es virtual del cluster y no es accesible desde fuera del cluster

# Video 59

NodePort: expone la aplicación en un puerto específico del nodo

minikube service mi-servicio-nodeport --url

# Video 60

LoadBalancer: expone la aplicación en un puerto específico del nodo y crea un balanceador de carga


# Video 63

docker run --rm -dti -v $PWD/:/go --net host --name goland golang bash

CMD
docker run --rm -dti -v //c/Users/20352675688/Desktop/devops/curso-k8s/kubernetes/k8s-hands-on/backend/src:/go -p 8080:8080 --name goland golang bash
