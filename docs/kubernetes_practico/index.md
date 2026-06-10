# Módulo 2: Namespace y pod básico:

## Práctica 1: Acceder y configurar el Namespace de tu usuario para trabajar de forma aislada.

### Guía: Usa kubectl para seleccionar el contexto actual y el Namespace en el que vas a trabajar.

### Solución:

- El siguiente comando cambia el conexto actual para trabajar dentro del namespace jfuriool:

```shell
kubectl config set-context --current --namespace=jfuriool
Context "gke_mdona-cloud-labpri-common_europe-west1_mdona-cloud-labpri-common-gke2" modified.
```

## Práctica 2: Desplegar un pod básico y ver que está levantado.

### Fichero yaml: nginx-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Guía: Despliega un pod de nginx con el fichero yaml y valida que ha levantado. Debe de aparecer en estado Running.

### Solución:

- Con el comando "kubectl apply" puedes aplicar YAML sobre kubernetes:

```shell
kubectl apply -f nginx-pod.yaml
Warning: BCID failed open: BCID suppressed until 2026-06-09 13:33:50.704439716 +0000 UTC m=+713596.480065453 pod/nginx created
```

- Con el comando "kubectl get pod" puedes mostrar el listado de pods:

```shell
kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          9s
```

# Módulo 3: Servicio y acceso al pod:

## Práctica 1: Exponer el pod mediante un servicio.

### Fichero yaml: nginx-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  type: ClusterIP
```

### Guía: Crea un servicio interno con el fichero yaml y valida que está creado y asociado al pod.

### Solución:

- Con el comando "kubectl apply" puedes aplicar YAML sobre kubernetes:

```shell
kubectl apply -f nginx-service.yaml
service/nginx-service created
```

- Con el comando "kubectl get svc" puedes mostrar el listado de servicios:

```shell
kubectl get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   172.22.96.158   <none>        80/TCP    33s
```

- Con el comando "kubectl describe service" puedes ver los recursos del servicio:

```shell
kubectl describe service nginx-service
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   172.22.96.158   <none>        80/TCP    33s
[ LABPRI ] jfuriool_mercadona_com ~$ kubectl describe service nginx-service
Name:                     nginx-service
Namespace:                jfuriool
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       172.22.96.158
IPs:                      172.22.96.158
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                192.168.20.180:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:
  Type    Reason                          Age                From                   Message
  ----    ------                          ----               ----                   -------
  Normal  ADD                             56s                sc-gateway-controller  jfuriool/nginx-service
  Normal  DNSRecordProvisioningSucceeded  54s (x4 over 56s)  clouddns-controller    DNS records updated
```

## Práctica 2: Hacer un port forwarding

### Guía: Configura el acceso a un pod a través de un puerto (puerto 80) y consulta el contenido de bienvenida con el comando curl.

### Solución:

- Con el comando "kubectl port-forward" creará un tunel desde un puerto de tu máquina local al puerto del servicio. En este ejemplo, redigiremos el puerto local 8080 al puerto 80 del servicio:

```shell
kubectl port-forward service/nginx-service 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

- En otra consola y desde el bastión probaremos a acceder mediante curl:

```shell
curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

# Módulo 4: Deployment y escalado:

## Práctica 1: Desplegar 2 réplicas de un pod

### Fichero: nginx-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Guía: Crea un deployment con 2 réplicas y valida que han levantado 2 pods.

### Solución:

```shell
kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```

```shell
kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           71s
```

```shell
kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx                               1/1     Running   0          25m
nginx-deployment-59f6c6988b-czl66   1/1     Running   0          97s
nginx-deployment-59f6c6988b-lrl9b   1/1     Running   0          97s
```

## Práctica 2: Escalar el número de pods de un deployment.

### Guía: Escala las réplicas del deployment (con kubectl scale) ya creado a 4 réplicas del mismo pod.

### Solución:

```shell
kubectl scale deployment nginx-deployment --replicas 4
deployment.apps/nginx-deployment scaled
```

```shell
kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx                               1/1     Running   0          32m
nginx-deployment-59f6c6988b-2hgff   1/1     Running   0          12s
nginx-deployment-59f6c6988b-6jf4c   1/1     Running   0          12s
nginx-deployment-59f6c6988b-czl66   1/1     Running   0          8m34s
nginx-deployment-59f6c6988b-lrl9b   1/1     Running   0          8m34s
```

# Módulo 5: Configmaps y variables de entorno:

## Práctica 1: Uso de configmaps para pasar variables a los pods.

### Ficheros yaml: 
- configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mi-config
data:
  MENSAJE: "Hola desde Kubernetes"
```

- pod-con-configmap.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-config
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh", "-c", "echo $MENSAJE && sleep 3600"]
    env:
    - name: MENSAJE
      valueFrom:
        configMapKeyRef:
          name: mi-config
          key: MENSAJE
```

### Guía: Configura un pod que cuando arranque muestre por pantalla el mensaje "Hola desde Kubernetes", pasándole la cadena del mensaje mediante un configmap. Comprueba el output mediante kubectl logs.

```shell
kubectl apply -f configmap.yaml
configmap/mi-config created
```

```shell
kubectl get configmap
NAME               DATA   AGE
ca-bundles         2      70m
kube-root-ca.crt   1      70m
mi-config          1      3m13s
```

```shell
kubectl apply -f pod-con-configmap.yaml
Warning: BCID failed open: BCID call failed: 403 Forbidden; BCID is unavailable
pod/pod-config created
```

```shell
kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx                               1/1     Running   0          59m
nginx-deployment-59f6c6988b-2hgff   1/1     Running   0          26m
nginx-deployment-59f6c6988b-6jf4c   1/1     Running   0          26m
nginx-deployment-59f6c6988b-czl66   1/1     Running   0          35m
nginx-deployment-59f6c6988b-lrl9b   1/1     Running   0          35m
pod-config                          1/1     Running   0          39s
```

```shell
kubectl logs pod-config
Hola desde Kubernetes
```

# Módulo 6: Secretos

## Práctica 1: Crear un secreto con usuario y contraseña

### Ficheros yaml: secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mi-secret
type: Opaque
data:
  usuario: <usuario>
  password: <password>
```

### Guía: Codifica el usuario y password a introducir en base64 y añádelas en el YAML.
Usuario: "<nombreusuario"
password: "k8s123"

### Solución:

Kubernetes requiere que los valores dentro de un secreto YAML estén codificados en Base64.
- Codificamos primero los datos suministrados:

```shell
echo -n '<nombreusuario>' | base64
PG5vbWJyZXVzdWFyaW8+
```

```shell
echo -n '<contraseña>' | base64    
PGNvbnRyYXNlw7FhPg==
```

```shell
sed -i 's/<usuario>/b21vcmNpbA==/' secret.yaml
```

```shell
sed -i 's/<password>/azhzMTIz/' secret.yaml    
```

```shell
cat secret.yaml 
apiVersion: v1
kind: Secret
metadata:
  name: mi-secret
type: Opaque
data:
  usuario: b21vcmNpbA==
  password: azhzMTIz
```

```shell
kubectl apply -f secret.yaml 
secret/mi-secret created
```

```shell
kubectl get secret
NAME        TYPE                             DATA   AGE
mi-secret   Opaque                           2      6s
regcred     kubernetes.io/dockerconfigjson   1      89m  
```

```shell
kubectl get secret mi-secret -o jsonpath='{.data.usuario}' | base64 --decode
omorcil
```

```shell
kubectl get secret mi-secret -o jsonpath='{.data.password}' | base64 --decode
k8s123
```

# Módulo 7: Cronjobs

## Práctica 1: Configurar un cronjob

### Ficheros yaml: cronjob.yaml

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hola-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hola
            image: busybox
            command: ["/bin/sh", "-c", "echo Hola desde CronJob"]
          restartPolicy: OnFailure
```
          
### Guía: Configura un cronjob que se ejecute cada minuto e imprima por pantalla el mensaje "hola desde cronjob".

```shell
kubectl apply -f cronjob.yaml 
cronjob.batch/hola-cronjob created
```

```shell
kubectl get cronjobs
NAME           SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hola-cronjob   */1 * * * *   <none>     False     0        39s             58s
```

## Práctica 2: Revisar los logs de los Jobs creados.

### Guía: Consulta los logs de la ejecución del job más reciente con kubectl logs.

```shell
kubectl get jobs
NAME                    STATUS     COMPLETIONS   DURATION   AGE
hola-cronjob-29683574   Complete   1/1           5s         2m30s
hola-cronjob-29683575   Complete   1/1           3s         90s
hola-cronjob-29683576   Complete   1/1           4s         30s
```

```shell
kubectl get pods
NAME                                READY   STATUS      RESTARTS   AGE
hola-cronjob-29683574-fmf84         0/1     Completed   0          2m40s
hola-cronjob-29683575-fvpt5         0/1     Completed   0          100s
hola-cronjob-29683576-pt7sh         0/1     Completed   0          40s
nginx                               1/1     Running     0          87m
nginx-deployment-59f6c6988b-2hgff   1/1     Running     0          55m
nginx-deployment-59f6c6988b-6jf4c   1/1     Running     0          55m
nginx-deployment-59f6c6988b-czl66   1/1     Running     0          64m
nginx-deployment-59f6c6988b-lrl9b   1/1     Running     0          64m
pod-config                          1/1     Running     0          29m
```

```shell
kubectl logs hola-cronjob-29683576-pt7sh
Hola desde CronJob
```

# Módulo 8: Horizontal Pod Autoscaler (HPA)

## Práctica 1: Crea un deployment con una réplica y un servicio asociado el cual recibirá las peticiones.

### Requisitos: El deployment debe tener recursos (resources.requests.cpu) definidos.

### Ficheros yaml: 

- hpa-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m
```

- php-apache-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: php-apache
spec:
  selector:
    app: php-apache
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

### Guía: Crea un servicio interno para el deployment el cual después usuaremos para escalar.

### Solución:

Crearemos un deploy con recursos establecidos y un servicio el cual repartirá las peticiones a los pods del deploy.

```shell
kubectl apply -f hpa-deployment.yaml
deployment.apps/php-apache created
```

```shell
kubectl apply -f php-apache-service.yaml 
service/php-apache created
```

```shell
kubectl get pods
NAME                                READY   STATUS      RESTARTS   AGE
hola-cronjob-29683584-dlff9         0/1     Completed   0          2m5s
hola-cronjob-29683585-wmbnx         0/1     Completed   0          65s
hola-cronjob-29683586-wgrbt         0/1     Completed   0          5s
nginx                               1/1     Running     0          97m
nginx-deployment-59f6c6988b-2hgff   1/1     Running     0          65m
nginx-deployment-59f6c6988b-6jf4c   1/1     Running     0          65m
nginx-deployment-59f6c6988b-czl66   1/1     Running     0          73m
nginx-deployment-59f6c6988b-lrl9b   1/1     Running     0          73m
php-apache-758f598-5bmm2            1/1     Running     0          42s
pod-config                          1/1     Running     0          38m
```

```shell
kubectl get scv
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   172.22.96.158   <none>        80/TCP    88m
php-apache      ClusterIP   172.22.96.178   <none>        80/TCP    53s
```

```shell
kubectl describe deploy php-apache
Name:                   php-apache
Namespace:              jfuriool
CreationTimestamp:      Tue, 09 Jun 2026 16:25:20 +0200
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=php-apache
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=php-apache
  Containers:
   php-apache:
    Image:      k8s.gcr.io/hpa-example
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:  500m
    Requests:
      cpu:         200m
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   php-apache-758f598 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  82s   deployment-controller  Scaled up replica set php-apache-758f598 from 0 to 1
```

## Práctica 2: Configurar el HPA

### Guía: Crea un autoscaler con "kubectl autoscale" que mantenga el uso de CPU por debajo del 50%, escalando entre 1 y 5 réplicas según el uso de CPU.

- Creamos un HPA por línea de comandos que escale de 1 a 5 réplicas al llegar al 50% del uso de CPU.

```shell
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=50
Flag --cpu-percent has been deprecated, Use --cpu with percentage or resource quantity format (e.g., '70%' for utilization or '500m' for milliCPU).
horizontalpodautoscaler.autoscaling/php-apache autoscaled
```

```shell
kubectl autoscale deployment php-apache --cpu=50% --min=1 --max=50
```

```shell
kubectl get hpa
NAME         REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 0%/50%   1         50        1          109s
```

## Práctica 3: Generar carga de prueba.

### Guía: Levanta un pod de una image busybox y lanza un bucle while dentro del pod que genere carga mediante un wget -q -O a http://php-apache

- Con "kubectl run -i -tty" podemos generar un pod desde una imagen y lanzar comandos desde dentro de él. Lanzando el wget sobre el servicio del deploy de php-apache creamos tráfico entrante.

```shell
kubectl run -i --tty load-generator --image=busybox /bin/sh
Warning: BCID failed open: BCID suppressed until 2026-06-09 16:47:18.669581103 +0000 UTC m=+725179.495438256
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
/ # while true; do wget -q -O- http://php-apache; done
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!^C
/ # 
```

- Desde otra consola revisamos el escalado y el hpa.

```shell
watch "kubectl get po && kubectl get hpa"

Every 2.0s: kubectl get po && kubectl get hpa                                                                                                             bastion-infra-02wh.c.mdona-cloud-netops-labpri.internal: Tue Jun  9 16:40:52 2026

NAME                                READY   STATUS      RESTARTS   AGE
hola-cronjob-29683598-w8qnt         0/1     Completed   0          2m52s
hola-cronjob-29683599-2xjsx         0/1     Completed   0          112s
hola-cronjob-29683600-p9gkb         0/1     Completed   0          52s
load-generator                      1/1     Running     0          3m6s
nginx                               1/1     Running     0          112m
nginx-deployment-59f6c6988b-2hgff   1/1     Running     0          79m
nginx-deployment-59f6c6988b-6jf4c   1/1     Running     0          79m
nginx-deployment-59f6c6988b-czl66   1/1     Running     0          88m
nginx-deployment-59f6c6988b-lrl9b   1/1     Running     0          88m
php-apache-758f598-52xtb            1/1     Running     0          60s
php-apache-758f598-5bmm2            1/1     Running     0          15m
php-apache-758f598-nnmmw            1/1     Running     0          60s
php-apache-758f598-q7j6x            1/1     Running     0          60s
php-apache-758f598-vpnrf            1/1     Running     0          57s
pod-config                          1/1     Running     0          53m
NAME         REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 0%/50%   1         50        9          9m18s
```

# Módulo 9: Limpiar el entorno de trabajo

## Práctica 1: Eliminar los recursos creados
### Guía: Limpia el entorno y déjalo sin recursos

- Limpiamos el entorno para dejarlo listo para volver a empezar:

```shell
kubectl delete all --all

pod "hola-cronjob-29683601-5nkw8" deleted from jfuriool namespace
pod "hola-cronjob-29683602-jhbhr" deleted from jfuriool namespace
pod "hola-cronjob-29683603-qvg9z" deleted from jfuriool namespace
pod "load-generator" deleted from jfuriool namespace
pod "nginx" deleted from jfuriool namespace
pod "nginx-deployment-59f6c6988b-2hgff" deleted from jfuriool namespace
pod "nginx-deployment-59f6c6988b-6jf4c" deleted from jfuriool namespace
pod "nginx-deployment-59f6c6988b-czl66" deleted from jfuriool namespace
pod "nginx-deployment-59f6c6988b-lrl9b" deleted from jfuriool namespace
pod "php-apache-758f598-52xtb" deleted from jfuriool namespace
pod "php-apache-758f598-5bmm2" deleted from jfuriool namespace
pod "php-apache-758f598-nnmmw" deleted from jfuriool namespace
pod "php-apache-758f598-q7j6x" deleted from jfuriool namespace
pod "php-apache-758f598-vpnrf" deleted from jfuriool namespace
pod "pod-config" deleted from jfuriool namespace
service "nginx-service" deleted from jfuriool namespace
service "php-apache" deleted from jfuriool namespace
deployment.apps "nginx-deployment" deleted from jfuriool namespace
deployment.apps "php-apache" deleted from jfuriool namespace
horizontalpodautoscaler.autoscaling "php-apache" deleted from jfuriool namespace
cronjob.batch "hola-cronjob" deleted from jfuriool namespace
job.batch "hola-cronjob-29683601" deleted from jfuriool namespace
```

```shell
kubectl get all
No resources found in jfuriool namespace.
```
