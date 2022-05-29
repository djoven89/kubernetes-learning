# Actualizaciones de una aplicación

En este ejercicio se pretende simular los 3 métodos de actualización más comunes en Kubernetes. Para ello, usaremos una imagen que contiene una web sencilla desarrollada en Flask, la cual se desplegará en 3 entornos distintos: dev, staging, prod.

Para este ejercicio se usarán los siguientes objetos de Kubernetes:

* Namespace.
* ResourceQuota.
* LimitRange.
* Service.
* Deployment.

Para la realización del ejercicio, se usará el entorno del curso, compuesto por un nodo master y dos nodos worker. Todos ellos usarán la versión: `1.23.6`. Además, se hará uso de las siguientes imágenes:

* djoven89/app-lab2:1.0.0
* djoven89/app-lab2:1.0.1

## Información de los entornos

A continuación se indican los requisitos que tendrán cada uno de los tres entornos a desplegar para la práctica.

### Dev

Para este entorno se usará la estrategía de actualización [Rolling updates].

[Rolling updates]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment

Además, queremos aislar el entorno y limitar el uso de recursos que puede usar. Para ello, usaremos los objetos: `namespace`, `quota` y `limitrange`.

Las características a implementar para cada objeto son:

**Namespace:**

* Se llamará `cliente2-dev-ns`.

* Tendrá las siguientes etiquetas:
    * customer  = cliente2
    * app       = flask
    * env       = dev

* Tendrá las siguientes anotaciones:
    * customerName  = Daniel
    * customerEmail = daniel@dev.app2.lan

**Quotas:**

* Se llamará `cliente2-dev-quota`.

* Tendrá las siguientes etiquetas:
    * customer  = cliente2
    * app       = flask
    * env       = dev

* El máximo de recursos que podrá usar son:
    * 3 pods.
    * 1 replicas.
    * 1 servicios.
    * 3 secretos.
    * 4 configmaps.
    * 0 persistent volume claims.
    * 1.5 core de CPU como limit.
    * 1 core de CPU como request.
    * 1.5 GB de RAM como limit.
    * 1 GB de RAM como request.

**LimitRange:**

* Se llamará `cliente2-dev-limitrange`.

* Tendrá las siguientes etiquetas:
    * customer  = cliente12
    * app       = flask
    * env       = dev

* Los valores por defecto para pods y pvcs:
    * Requests:
        * CPU: 0.1
        * RAM: 150 MB
    * Limits:
        * CPU: 0.2
        * RAM: 250 MB

* Los valores mínimos para pods y pvcs:
    * CPU: 0.1
    * RAM: 150 MB

* Los valores máximos para pods y pvcs:
    * CPU: 0.2
    * RAM: 250 MB

La aplicación constará de 2 pods en ejecucción.

Los requisitos de la estrategía de actualización serán:

* Durante una actualización, siempre tendrá que haber 1 pod como mínimo en activo.
* Se podrá exceder en 1 los pods a crear durante la actualización.


### Staging

Para este entorno se usará la estrategía de actualización [Canary deployment].

[Canary deployment]: https://kubernetes.io/es/docs/concepts/workloads/controllers/deployment/#despligue-canary

Además, queremos aislar el entorno y limitar el uso de recursos que puede usar. Para ello, usaremos los objetos: `namespace`, `quota` y `limitrange`.

Las características a implementar para cada objeto son:

**Namespace:**

* Se llamará `cliente2-staging-app`.

* Tendrá las siguientes etiquetas:
    * customer  = cliente2
    * app       = flask
    * env       = staging

* Tendrá las siguientes anotaciones:
    * customerName  = Daniel
    * customerEmail = daniel@dev.app2.lan

**Quotas:**

* Se llamará `cliente2-staging-quota`.

* Tendrá las siguientes etiquetas:
    * customer  = cliente12
    * app       = flask
    * env       = staging

* El máximo de recursos que podrá usar son:
    * 6 pods.
    * 2 replicas.
    * 2 servicios.
    * 6 secretos.
    * 6 configmaps.
    * 0 persistent volume claims.
    * 1.5 core de CPU como limit.
    * 1 core de CPU como request.
    * 2 GB de RAM como limit.
    * 1.5 GB de RAM como request.

**LimitRange:**

* Se llamará `cliente2-staging-limitrange`

* Tendrá las siguientes etiquetas:
    * customer  = cliente12
    * app       = flask
    * env       = staging

* Los valores por defecto para pods y pvcs:
    * Requests:
        * CPU: 0.1
        * RAM: 150 MB
        * PVC: 0 MB
    * Limits:
        * CPU: 0.5
        * RAM: 500 MB
        * PVC: 0 GB

* Los valores mínimos para pods y pvcs:
    * CPU: 0.1
    * RAM: 150 MB
    * PVC: 0 MB

* Los valores máximos para pods y pvcs:
    * CPU: 0.5
    * RAM: 500 MB
    * PVC: 0 GB

La aplicación constará de 6 pods en ejecucción.

Cuando se actualice la aplicación, se irán reduciendo e incrementando paulatinamente la antigua y nueva versión. A continuación los valores de pods que ejecutan la antigua versión y la nueva:

* 5:1
* 3:3
* 1:5
* 0:6

### Prod

Para este entorno se usará la estrategía de actualización Blue-Green deployment.

Además, queremos aislar el entorno y limitar el uso de recursos que puede usar. Para ello, usaremos los objetos: `namespace`, `quota` y `limitrange`.

Las características a implementar para cada objeto son:

**Namespace:**

* Se llamará `cliente2-prod-app`.

* Tendrá las siguientes etiquetas:
    * customer  = cliente2
    * app       = flask
    * env       = prod

* Tendrá las siguientes anotaciones:
    * customerName  = Daniel
    * customerEmail = daniel@dev.app2.lan

**Quotas:**

* Se llamará `cliente2-prod-quota`.

* Tendrá las siguientes etiquetas:
    * customer  = cliente12
    * app       = flask
    * env       = prod

* El máximo de recursos que podrá usar son:
    * 12 pods.
    * 2 replicas.
    * 2 servicios.
    * 10 secretos.
    * 10 configmaps.
    * 0 persistent volume claims.
    * 1.5 core de CPU como limit.
    * 1 core de CPU como request.
    * 2 GB de RAM como limit.
    * 1.5 GB de RAM como request.

**LimitRange:**

* Se llamará `cliente2-prod-limitrange`

* Tendrá las siguientes etiquetas:
    * customer  = cliente12
    * app       = flask
    * env       = prod

* Los valores por defecto para pods y pvcs:
    * Requests:
        * CPU: 0.1
        * RAM: 150 MB
        * PVC: 100 MB
    * Limits:
        * CPU: 1.0
        * RAM: 750 MB
        * PVC: 1 GB

* Los valores mínimos para pods y pvcs:
    * CPU: 0.1
    * RAM: 150 MB
    * PVC: 0 MB

* Los valores máximos para pods y pvcs:
    * CPU: 1.0
    * RAM: 750 MB
    * PVC: 0 GB

La aplicación constará de 6 pods en ejecucción.

## Despliegue en el entorno dev

A continuación se indican las acciones a realizar para crear el entorno, desplegar la aplicación, testearla y actualizarla usando el método `rolling updates`.

### 1/4 Preparación del entorno

Lo primero que haremos será crear el `namespace`, `quota` y `limitrange`.

`kubectl apply -f 01-dev-namespace.yaml`

Revisamos que se haya creado el `namespace` y sus requisitos:

```
kubectl get ns -l env=dev
  NAME              STATUS   AGE
  cliente2-dev-ns   Active   19s


kubectl describe ns cliente2-dev-ns
  Name:         cliente2-dev-ns
  Labels:       app=flask
                customer=client2
                env=dev
                kubernetes.io/metadata.name=cliente2-dev-ns
  Annotations:  customerEmail: daniel@dev.app2.lan
                customerName: Daniel
  Status:       Active

  Resource Quotas
    Name:                   cliente2-dev-quota
    Resource                Used  Hard
    --------                ---   ---
    configmaps              1     3
    limits.cpu              0     1500m
    limits.memory           0     1536Mi
    persistentvolumeclaims  0     0
    pods                    0     3
    replicationcontrollers  0     1
    requests.cpu            0     1
    requests.memory         0     1Gi
    secrets                 1     3
    services                0     1

  Resource Limits
  Type       Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
  ----       --------  ---    ---    ---------------  -------------  -----------------------
  Container  cpu       100m   200m   100m             200m           -
  Container  memory    150Mi  250Mi  150Mi            250Mi          -


kubectl -n cliente2-dev-ns describe limitranges cliente2-dev-limitrange
  Name:       cliente2-dev-limitrange
  Namespace:  cliente2-dev-ns
  Type        Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
  ----        --------  ---    ---    ---------------  -------------  -----------------------
  Container   memory    150Mi  250Mi  150Mi            250Mi          -
  Container   cpu       100m   200m   100m             200m           -
```

### 2/4 Despliegue

Una vez que tenemos el entorno listo, crearemos el servicio. El cual tendrá los siguientes requisitos:

* Será de tipo `NodePort`.
* Expondrá el puerto `32020`.
* El puerto del contenedor será `8080`.

`kubectl apply -f 02-dev-service.yaml`

Comprobamos que el servicio haya sido creado por éxito:

```
kubectl -n cliente2-dev-ns get svc -o wide
  NAME                   TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE   SELECTOR
  cliente2-dev-service   NodePort   10.111.9.66   <none>        8080:32020/TCP   6s    app=flask,customer=cliente2,env=dev


kubectl -n cliente2-dev-ns describe svc cliente2-dev-service
  Name:                     cliente2-dev-service
  Namespace:                cliente2-dev-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=dev
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=dev
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.104.91.220
  IPs:                      10.104.91.220
  Port:                     cliente2-dev-web-port  8080/TCP
  TargetPort:               8080/TCP
  NodePort:                 cliente2-dev-web-port  32020/TCP
  Endpoints:                10.244.104.60:8080,10.244.166.187:8080
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>
```

A continuación, crearemos el `deployment` con las siguientes características:

* Habrá 2 pods en ejecución.
* Siempre deberá de haber 1 pod como mínimo activo durante una actualización.
* Se podrá execeder en 1 los pods durante una actualización.

`kubectl apply -f 03-dev-deployment.yaml --record`

A continuación, revisamos el estado del despliegue:

```
kubectl -n cliente2-dev-ns get deploy,rs,pods -o wide
  NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS        IMAGES                    SELECTOR
  deployment.apps/cliente2-dev-deployment   2/2     2            2           2m23s   client2-dev-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=dev

  NAME                                                 DESIRED   CURRENT   READY   AGE     CONTAINERS        IMAGES                    SELECTOR
  replicaset.apps/cliente2-dev-deployment-798b79d588   2         2         2       2m23s   client2-dev-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=dev,pod-template-hash=798b79d588

  NAME                                           READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-dev-deployment-798b79d588-8tqbz   1/1     Running   0          2m23s   10.244.166.187   node1   <none>           <none>
  pod/cliente2-dev-deployment-798b79d588-gbdcd   1/1     Running   0          2m23s   10.244.104.60    node2   <none>           <none>


kubectl -n cliente2-dev-ns describe svc cliente2-dev-service
  Name:                     cliente2-dev-service
  Namespace:                cliente2-dev-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=dev
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=dev
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.104.91.220
  IPs:                      10.104.91.220
  Port:                     cliente2-dev-web-port  8080/TCP
  TargetPort:               8080/TCP
  NodePort:                 cliente2-dev-web-port  32020/TCP
  Endpoints:                10.244.104.60:8080,10.244.166.187:8080
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>


kubectl describe ns cliente2-dev-ns
  Name:         cliente2-dev-ns
  Labels:       app=flask
                customer=client2
                env=dev
                kubernetes.io/metadata.name=cliente2-dev-ns
  Annotations:  customerEmail: daniel@dev.app2.lan
                customerName: Daniel
  Status:       Active

  Resource Quotas
    Name:                   cliente2-dev-quota
    Resource                Used   Hard
    --------                ---    ---
    configmaps              1      3
    limits.cpu              400m   1500m
    limits.memory           500Mi  1536Mi
    persistentvolumeclaims  0      0
    pods                    2      3
    replicationcontrollers  0      1
    requests.cpu            200m   1
    requests.memory         300Mi  1Gi
    secrets                 1      3
    services                1      1

  Resource Limits
  Type       Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
  ----       --------  ---    ---    ---------------  -------------  -----------------------
  Container  cpu       100m   200m   100m             200m           -
  Container  memory    150Mi  250Mi  150Mi            250Mi          -
```

Finalmente, vamos al navegador web y accedemos al servidor `master` y con el puerto `32020`.

Para obtener la IP de los nodos ejecutamos el siguiente comando:

```
kubectl get nodes -o wide
  NAME     STATUS   ROLES                  AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
  master   Ready    control-plane,master   6d20h   v1.23.6   192.168.100.10   <none>        Ubuntu 20.04.1 LTS   5.4.0-52-generic   docker://19.3.9
  node1    Ready    <none>                 6d20h   v1.23.6   192.168.100.11   <none>        Ubuntu 20.04.1 LTS   5.4.0-52-generic   docker://19.3.9
  node2    Ready    <none>                 6d20h   v1.23.6   192.168.100.12   <none>        Ubuntu 20.04.1 LTS   5.4.0-52-generic   docker://19.3.9
```

En mi caso concreto, sería: <http://192.168.100.10:32020>


### 3/4 Actualizaciones

Con la aplicación correctamente ejecutándose, procederemos a actualizar la aplicación estableciendo la imagen: `djoven89/app-lab2:1.0.1`.

`kubectl -n cliente2-dev-ns set image deployment/cliente2-dev-deployment client2-dev-web=djoven89/app-lab2:1.0.1 --record`

Comprobamos el estado de los objetos en Kubernetes:

```
kubectl -n cliente2-dev-ns get deploy,rs,pod
  NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/cliente2-dev-deployment   2/2     2            2           3m34s

  NAME                                                 DESIRED   CURRENT   READY   AGE
  replicaset.apps/cliente2-dev-deployment-798b79d588   0         0         0       3m34s
  replicaset.apps/cliente2-dev-deployment-7fb775895    2         2         2       36s

  NAME                                          READY   STATUS    RESTARTS   AGE
  pod/cliente2-dev-deployment-7fb775895-62l92   1/1     Running   0          36s
  pod/cliente2-dev-deployment-7fb775895-gkd69   1/1     Running   0          34s


kubectl -n cliente2-dev-ns describe deployments cliente2-dev-deployment
  Name:                   cliente2-dev-deployment
  Namespace:              cliente2-dev-ns
  CreationTimestamp:      Sun, 29 May 2022 12:47:07 +0000
  Labels:                 app=flask
                          customer=cliente2
                          env=dev
  Annotations:            deployment.kubernetes.io/revision: 2
  Selector:               app=flask,customer=cliente2,env=dev
  Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
  StrategyType:           RollingUpdate
  MinReadySeconds:        0
  RollingUpdateStrategy:  1 max unavailable, 1 max surge
  Pod Template:
    Labels:  app=flask
            customer=cliente2
            env=dev
    Containers:
    client2-dev-web:
      Image:        djoven89/app-lab2:1.0.1
      Port:         80/TCP
      Host Port:    0/TCP
      Environment:  <none>
      Mounts:       <none>
    Volumes:        <none>
  Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
  OldReplicaSets:  <none>
  NewReplicaSet:   cliente2-dev-deployment-7fb775895 (2/2 replicas created)
  Events:
    Type    Reason             Age    From                   Message
    ----    ------             ----   ----                   -------
    Normal  ScalingReplicaSet  5m23s  deployment-controller  Scaled up replica set cliente2-dev-deployment-798b79d588 to 2
    Normal  ScalingReplicaSet  2m25s  deployment-controller  Scaled up replica set cliente2-dev-deployment-7fb775895 to 1
    Normal  ScalingReplicaSet  2m25s  deployment-controller  Scaled down replica set cliente2-dev-deployment-798b79d588 to 1
    Normal  ScalingReplicaSet  2m25s  deployment-controller  Scaled up replica set cliente2-dev-deployment-7fb775895 to 2
    Normal  ScalingReplicaSet  2m23s  deployment-controller  Scaled down replica set cliente2-dev-deployment-798b79d588 to 0


kubectl -n cliente2-dev-ns describe service cliente2-dev-service
  Name:                     cliente2-dev-service
  Namespace:                cliente2-dev-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=dev
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=dev
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.111.9.66
  IPs:                      10.111.9.66
  Port:                     cliente2-dev-web-port  8080/TCP
  TargetPort:               8080/TCP
  NodePort:                 cliente2-dev-web-port  32020/TCP
  Endpoints:                10.244.104.47:8080,10.244.166.173:8080
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>


kubectl -n cliente2-dev-ns describe pod cliente2-dev-deployment-7fb775895-62l92
  Name:         cliente2-dev-deployment-7fb775895-62l92
  Namespace:    cliente2-dev-ns
  Priority:     0
  Node:         node2/192.168.100.12
  Start Time:   Sun, 29 May 2022 12:50:05 +0000
  Labels:       app=flask
                customer=cliente2
                env=dev
                pod-template-hash=7fb775895
  Annotations:  cni.projectcalico.org/podIP: 10.244.104.47/42
                cni.projectcalico.org/podIPs: 10.244.104.47/42
                kubernetes.io/limit-ranger:
                  LimitRanger plugin set: cpu, memory request for container client2-dev-web; cpu, memory limit for container client2-dev-web
  Status:       Running
  IP:           10.244.104.47
  IPs:
    IP:           10.244.104.47
  Controlled By:  ReplicaSet/cliente2-dev-deployment-7fb775895
  Containers:
    client2-dev-web:
      Container ID:   docker://ac79c13d77cf0c78e250f1bb6c8942e70b430572d7e32d18a76b089825b5330f
      Image:          djoven89/app-lab2:1.0.1
      Image ID:       docker-pullable://djoven89/app-lab2@sha256:afb5f4deaae4ade91d7d5a7bfb62e0c964d28736a586aecc7a417bd3682a0c8e
      Port:           80/TCP
      Host Port:      0/TCP
      State:          Running
        Started:      Sun, 29 May 2022 12:50:06 +0000
      Ready:          True
      Restart Count:  0
      Limits:
        cpu:     200m
        memory:  250Mi
      Requests:
        cpu:        100m
        memory:     150Mi
      Environment:  <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-fm4zb (ro)
  Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
  Volumes:
    kube-api-access-fm4zb:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      ConfigMapOptional:       <nil>
      DownwardAPI:             true
  QoS Class:                   Burstable
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                              node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  109s  default-scheduler  Successfully assigned cliente2-dev-ns/cliente2-dev-deployment-7fb775895-62l92 to node2
    Normal  Pulled     108s  kubelet            Container image "djoven89/app-lab2:1.0.1" already present on machine
    Normal  Created    108s  kubelet            Created container client2-dev-web
    Normal  Started    108s  kubelet            Started container client2-dev-web
```

Finalmente, volvemos a acceder a la aplicación desde el navegador.

### 4/4 Rollback

En caso de que tengamos que realizar un rollback y volver a la versión anterior de la aplicación, podremos usar el comando `rollout`.

Lo primero buscaremos la versión en el historia:

```
kubectl -n cliente2-dev-ns rollout history deployment cliente2-dev-deployment
  deployment.apps/cliente2-dev-deployment
  REVISION  CHANGE-CAUSE
  1         <none>
  2         kubectl set image deployment/cliente2-dev-deployment client2-dev-web=djoven89/app-lab2:1.0.1 --namespace=cliente2-dev-ns --record=true
```

Después, restauramos usando la revisión `1` en este caso:

`kubectl -n cliente2-dev-ns rollout undo deployment cliente2-dev-deployment --to-revision=1`

Comprobamos el estado de los objectos tras la acción anterior:

```
kubectl -n cliente2-dev-ns get deploy,rs,pod
  NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/cliente2-dev-deployment   2/2     2            2           15m

  NAME                                                 DESIRED   CURRENT   READY   AGE
  replicaset.apps/cliente2-dev-deployment-798b79d588   2         2         2       15m
  replicaset.apps/cliente2-dev-deployment-7fb775895    0         0         0       9m24s

  NAME                                           READY   STATUS    RESTARTS   AGE
  pod/cliente2-dev-deployment-798b79d588-mwcd2   1/1     Running   0          22s
  pod/cliente2-dev-deployment-798b79d588-xn8zj   1/1     Running   0          18s


kubectl -n cliente2-dev-ns describe pod cliente2-dev-deployment-798b79d588-mwcd2 
  Name:         cliente2-dev-deployment-798b79d588-mwcd2
  Namespace:    cliente2-dev-ns
  Priority:     0
  Node:         node1/192.168.100.11
  Start Time:   Sun, 29 May 2022 15:13:58 +0000
  Labels:       app=flask
                customer=cliente2
                env=dev
                pod-template-hash=798b79d588
  Annotations:  cni.projectcalico.org/podIP: 10.244.166.189/32
                cni.projectcalico.org/podIPs: 10.244.166.189/32
                kubernetes.io/limit-ranger:
                  LimitRanger plugin set: cpu, memory request for container client2-dev-web; cpu, memory limit for container client2-dev-web
  Status:       Running
  IP:           10.244.166.189
  IPs:
    IP:           10.244.166.189
  Controlled By:  ReplicaSet/cliente2-dev-deployment-798b79d588
  Containers:
    client2-dev-web:
      Container ID:   docker://78f2a33dde701c939c838b36b4882c10fa9fc21ca845649c7bd6ad84fc54ece4
      Image:          djoven89/app-lab2:1.0.0
      Image ID:       docker-pullable://djoven89/app-lab2@sha256:707c293b46b85e7c4a6331fedc5a2fbf636145d50963422800b9e7978dc4b625
      Port:           80/TCP
      Host Port:      0/TCP
      State:          Running
        Started:      Sun, 29 May 2022 15:13:59 +0000
      Ready:          True
      Restart Count:  0
      Limits:
        cpu:     200m
        memory:  250Mi
      Requests:
        cpu:        100m
        memory:     150Mi
      Environment:  <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tn4ll (ro)
  Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
  Volumes:
    kube-api-access-tn4ll:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      ConfigMapOptional:       <nil>
      DownwardAPI:             true
  QoS Class:                   Burstable
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                              node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  72s   default-scheduler  Successfully assigned cliente2-dev-ns/cliente2-dev-deployment-798b79d588-mwcd2 to node1
    Normal  Pulled     72s   kubelet            Container image "djoven89/app-lab2:1.0.0" already present on machine
    Normal  Created    71s   kubelet            Created container client2-dev-web
    Normal  Started    71s   kubelet            Started container client2-dev-web
```

Finalmente, confirmamos desde el navegador web como la aplicación vuelve a mostrar la versión anterior.

## Despliegue en el entorno staging

A continuación se indican las acciones a realizar para crear el entorno, desplegar la aplicación, testearla y actualizarla usando el método `canary deployment`.

### 1/4 Preparación del entorno

Lo primero que haremos será crear el `namespace`, `quota` y `limitrange`.

`kubectl apply -f 04-staging-entorno.yaml`

Revisamos que se haya creado el `namespace` y sus requisitos:

```
kubectl get ns -l env=staging
  NAME                  STATUS   AGE
  cliente2-staging-ns   Active   17s


kubectl describe ns cliente2-staging-ns
  Name:         cliente2-staging-ns
  Labels:       app=flask
                customer=client22
                env=staging
                kubernetes.io/metadata.name=cliente2-staging-ns
  Annotations:  customerEmail: daniel@staging.app2.lan
                customerName: Daniel
  Status:       Active

  Resource Quotas
    Name:                   cliente2-staging-quota
    Resource                Used  Hard
    --------                ---   ---
    configmaps              1     6
    limits.cpu              0     1500m
    limits.memory           0     2Gi
    persistentvolumeclaims  0     0
    pods                    0     8
    replicationcontrollers  0     2
    requests.cpu            0     1
    requests.memory         0     1536Mi
    secrets                 1     6
    services                0     2

  Resource Limits
  Type       Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
  ----       --------  ---    ---    ---------------  -------------  -----------------------
  Container  memory    150Mi  250Mi  150Mi            250Mi          -
  Container  cpu       100m   200m   100m             200m           -
```

### 2/4 Despliegue

Una vez que tenemos el entorno listo, crearemos el servicio. El cual tendrá los siguientes requisitos:

* Será de tipo `NodePort`.
* Expondrá el puerto `32021`.
* El puerto del contenedor será `8080`.

`kubectl apply -f 05-staging-service.yaml`

Comprobamos que el servicio haya sido creado por éxito:

```
kubectl -n cliente2-staging-ns get svc -o wide
  NAME                       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
  cliente2-staging-service   NodePort   10.97.130.153   <none>        8080:32021/TCP   10s   app=flask,customer=cliente2,env=staging


kubectl -n cliente2-staging-ns describe svc cliente2-staging-service
  Name:                     cliente2-staging-service
  Namespace:                cliente2-staging-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=staging
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=staging
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.97.130.153
  IPs:                      10.97.130.153
  Port:                     cliente2-staging-web-port  8080/TCP
  TargetPort:               8080/TCP
  NodePort:                 cliente2-staging-web-port  32021/TCP
  Endpoints:                <none>
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>
```

A continuación, crearemos el `deployment` con las siguientes características:

* La aplicación constará de 6 pods en ejecucción.
* Tendrá una etiqueta adicional llamada `track` que para el deployment en ejecución tendrá un valor de `stable` mientras que para el deployment de Canary tendrá como valor `canary`.

`kubectl apply -f 06-staging-deployment-stable.yaml --record`

Revisamos que los objetos hayan sido creados con éxito:

```
kubectl -n cliente2-dev-ns rollout status deployment cliente2-dev-deployment
  deployment "cliente2-dev-deployment" successfully rolled out


kubectl -n cliente2-staging-ns get deploy,rs,pod -o wide
  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS            IMAGES                    SELECTOR
  deployment.apps/cliente2-staging-deployment   6/6     6            6           11s   client2-staging-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,track=stable

  NAME                                                     DESIRED   CURRENT   READY   AGE   CONTAINERS            IMAGES                    SELECTOR
  replicaset.apps/cliente2-staging-deployment-6787fdb4d4   6         6         6       11s   client2-staging-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,pod-template-hash=6787fdb4d4,track=stable

  NAME                                               READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-staging-deployment-6787fdb4d4-6dvh6   1/1     Running   0          11s   10.244.166.191   node1   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-74m7f   1/1     Running   0          11s   10.244.104.1     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-bvwng   1/1     Running   0          11s   10.244.166.190   node1   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-hbpk2   1/1     Running   0          11s   10.244.104.2     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-k22mr   1/1     Running   0          11s   10.244.104.63    node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-lkdrr   1/1     Running   0          11s   10.244.166.129   node1   <none>           <none>


kubectl describe ns cliente2-staging-ns
  Name:         cliente2-staging-ns
  Labels:       app=flask
                customer=client22
                env=staging
                kubernetes.io/metadata.name=cliente2-staging-ns
  Annotations:  customerEmail: daniel@staging.app2.lan
                customerName: Daniel
  Status:       Active

  Resource Quotas
    Name:                   cliente2-staging-quota
    Resource                Used    Hard
    --------                ---     ---
    configmaps              1       6
    limits.cpu              1200m   1500m
    limits.memory           1500Mi  2Gi
    persistentvolumeclaims  0       0
    pods                    6       8
    replicationcontrollers  0       2
    requests.cpu            600m    1
    requests.memory         900Mi   1536Mi
    secrets                 1       6
    services                1       2

  Resource Limits
  Type       Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
  ----       --------  ---    ---    ---------------  -------------  -----------------------
  Container  cpu       100m   200m   100m             200m           -
  Container  memory    150Mi  250Mi  150Mi            250Mi          -


kubectl -n cliente2-staging-ns describe svc cliente2-staging-service
  Name:                     cliente2-staging-service
  Namespace:                cliente2-staging-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=staging
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=staging
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.97.130.153
  IPs:                      10.97.130.153
  Port:                     cliente2-staging-web-port  8080/TCP
  TargetPort:               8080/TCP
  NodePort:                 cliente2-staging-web-port  32021/TCP
  Endpoints:                10.244.104.1:8080,10.244.104.2:8080,10.244.104.63:8080 + 3 more...
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>


kubectl -n cliente2-staging-ns describe pod cliente2-staging-deployment-6787fdb4d4-wh2mv
  Name:         cliente2-staging-deployment-6787fdb4d4-6dvh6
  Namespace:    cliente2-staging-ns
  Priority:     0
  Node:         node1/192.168.100.11
  Start Time:   Sun, 29 May 2022 15:19:54 +0000
  Labels:       app=flask
                customer=cliente2
                env=staging
                pod-template-hash=6787fdb4d4
                track=stable
  Annotations:  cni.projectcalico.org/podIP: 10.244.166.191/32
                cni.projectcalico.org/podIPs: 10.244.166.191/32
                kubernetes.io/limit-ranger:
                  LimitRanger plugin set: cpu, memory request for container client2-staging-web; cpu, memory limit for container client2-staging-web
  Status:       Running
  IP:           10.244.166.191
  IPs:
    IP:           10.244.166.191
  Controlled By:  ReplicaSet/cliente2-staging-deployment-6787fdb4d4
  Containers:
    client2-staging-web:
      Container ID:   docker://487ee44135cabfb85e03250beba5cad0e95b568cfa924b2ba0a00fb317a7a8cb
      Image:          djoven89/app-lab2:1.0.0
      Image ID:       docker-pullable://djoven89/app-lab2@sha256:707c293b46b85e7c4a6331fedc5a2fbf636145d50963422800b9e7978dc4b625
      Port:           80/TCP
      Host Port:      0/TCP
      State:          Running
        Started:      Sun, 29 May 2022 15:19:59 +0000
      Ready:          True
      Restart Count:  0
      Limits:
        cpu:     200m
        memory:  250Mi
      Requests:
        cpu:        100m
        memory:     150Mi
      Environment:  <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-467xn (ro)
  Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
  Volumes:
    kube-api-access-467xn:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      ConfigMapOptional:       <nil>
      DownwardAPI:             true
  QoS Class:                   Burstable
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                              node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  61s   default-scheduler  Successfully assigned cliente2-staging-ns/cliente2-staging-deployment-6787fdb4d4-6dvh6 to node1
    Normal  Pulling    59s   kubelet            Pulling image "djoven89/app-lab2:1.0.0"
    Normal  Pulled     57s   kubelet            Successfully pulled image "djoven89/app-lab2:1.0.0" in 2.745124999s
    Normal  Created    56s   kubelet            Created container client2-staging-web
    Normal  Started    56s   kubelet            Started container client2-staging-web
```

Finalmente, vamos al navegador web y accedemos al servidor `master` y con el puerto `32021`.


### 3/4 Actualizaciones

Para actualizar la aplicación, se seguiran las siguientes instrucciones:

* Se creará un nuevo deployment que tendrá la etiqueta adicional: `track: canary`.
* El deployment comenzará con un único pod.
* El deployment será temporal.
* Se realizarán los siguientes escalados durante la actualización:
  * 5:1
  * 3:3
* Una vez que la nueva versión de la aplicación se confirme como exitosa, reducimos las replicas del deployment con etiqueta `track: canary` a 0 e incrementamos a 6 las replicas del otro deployment.

Lo primero que haremos será reducir las réplicas del actual deployment:

`kubectl -n cliente2-staging-ns scale deployment cliente2-staging-deployment --replicas=5`

Comprobaremos como la reducción fue exitosa:

```
kubectl -n cliente2-staging-ns get deploy,rs,pod -o wide
  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS            IMAGES                    SELECTOR
  deployment.apps/cliente2-staging-deployment   5/5     5            5           76s   client2-staging-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,track=stable

  NAME                                                     DESIRED   CURRENT   READY   AGE   CONTAINERS            IMAGES                    SELECTOR
  replicaset.apps/cliente2-staging-deployment-6787fdb4d4   5         5         5       76s   client2-staging-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,pod-template-hash=6787fdb4d4,track=stable

  NAME                                               READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-staging-deployment-6787fdb4d4-bwwsn   1/1     Running   0          76s   10.244.104.4     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-fmlmc   1/1     Running   0          76s   10.244.166.130   node1   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-ljvdh   1/1     Running   0          76s   10.244.104.3     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-mmr8z   1/1     Running   0          76s   10.244.166.134   node1   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-tqdb6   1/1     Running   0          76s   10.244.166.131   node1   <none>           <none>
```

A continuación creamos el nuevo deployment:

`kubectl apply -f 07-staging-deployment-canary.yaml`

Nuevamente realizamos las comprobaciones:

```
kubectl -n cliente2-staging-ns get deploy,rs,pod -o wide
  NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS                   IMAGES                    SELECTOR
  deployment.apps/cliente2-staging-deployment          5/5     5            5           103s   client2-staging-web          djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,track=stable
  deployment.apps/cliente2-staging-deployment-canary   1/1     1            1           7s     client2-staging-web-canary   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=staging,track=canary

  NAME                                                            DESIRED   CURRENT   READY   AGE    CONTAINERS                   IMAGES                    SELECTOR
  replicaset.apps/cliente2-staging-deployment-6787fdb4d4          5         5         5       103s   client2-staging-web          djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,pod-template-hash=6787fdb4d4,track=stable
  replicaset.apps/cliente2-staging-deployment-canary-7fc84455fc   1         1         1       7s     client2-staging-web-canary   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=staging,pod-template-hash=7fc84455fc,track=canary

  NAME                                                      READY   STATUS    RESTARTS   AGE    IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-staging-deployment-6787fdb4d4-bwwsn          1/1     Running   0          103s   10.244.104.4     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-fmlmc          1/1     Running   0          103s   10.244.166.130   node1   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-ljvdh          1/1     Running   0          103s   10.244.104.3     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-mmr8z          1/1     Running   0          103s   10.244.166.134   node1   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-tqdb6          1/1     Running   0          103s   10.244.166.131   node1   <none>           <none>
  pod/cliente2-staging-deployment-canary-7fc84455fc-cmsp2   1/1     Running   0          7s     10.244.104.5     node2   <none>           <none>


kubectl -n cliente2-staging-ns describe svc cliente2-staging-service
  Name:                     cliente2-staging-service
  Namespace:                cliente2-staging-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=staging
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=staging
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.97.130.153
  IPs:                      10.97.130.153
  Port:                     cliente2-staging-web-port  8080/TCP
  TargetPort:               8080/TCP
  NodePort:                 cliente2-staging-web-port  32021/TCP
  Endpoints:                10.244.104.3:8080,10.244.104.4:8080,10.244.104.5:8080 + 3 more...
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>
```

Después, revisamos desde el navegador web que ĺa nueva versión de la aplicación también es mostrada.

Lo siguiente que haré será reducir a 3 réplicas en deployment `stable` e incrementar a 3 el deployment `canary`.

```
kubectl -n cliente2-staging-ns scale deployment cliente2-staging-deployment --replicas=3
kubectl -n cliente2-staging-ns scale deployment cliente2-staging-deployment-canary --replicas=3
```

Revisamos el estado de los objetos de Kubernetes tras el cambio:

```
kubectl -n cliente2-staging-ns get deploy,rs,pod -o wide
  NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                   IMAGES                    SELECTOR
  deployment.apps/cliente2-staging-deployment          3/3     3            3           3m30s   client2-staging-web          djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,track=stable
  deployment.apps/cliente2-staging-deployment-canary   3/3     3            3           114s    client2-staging-web-canary   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=staging,track=canary

  NAME                                                            DESIRED   CURRENT   READY   AGE     CONTAINERS                   IMAGES                    SELECTOR
  replicaset.apps/cliente2-staging-deployment-6787fdb4d4          3         3         3       3m30s   client2-staging-web          djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,pod-template-hash=6787fdb4d4,track=stable
  replicaset.apps/cliente2-staging-deployment-canary-7fc84455fc   3         3         3       114s    client2-staging-web-canary   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=staging,pod-template-hash=7fc84455fc,track=canary

  NAME                                                      READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-staging-deployment-6787fdb4d4-bwwsn          1/1     Running   0          3m30s   10.244.104.4     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-ljvdh          1/1     Running   0          3m30s   10.244.104.3     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-tqdb6          1/1     Running   0          3m30s   10.244.166.131   node1   <none>           <none>
  pod/cliente2-staging-deployment-canary-7fc84455fc-cmhfc   1/1     Running   0          10s     10.244.166.132   node1   <none>           <none>
  pod/cliente2-staging-deployment-canary-7fc84455fc-cmsp2   1/1     Running   0          114s    10.244.104.5     node2   <none>           <none>
  pod/cliente2-staging-deployment-canary-7fc84455fc-pt6w6   1/1     Running   0          10s     10.244.166.133   node1   <none>           <none>
```

Y nuevamente accedemos al navegador web para revisar que se muestra tanto la nueva como la antigua versión de la aplicación.

Una vez que hayamos confirmado que la nueva versión aplicación es estable para estar en producción, reduciremos a 0 el deployment `canary`.

`kubectl -n cliente2-staging-ns scale deployment cliente2-staging-deployment-canary --replicas=0`

Estableceremos a 6 las réplicas del deployment `stable`:

`kubectl -n cliente2-staging-ns scale deployment cliente2-staging-deployment --replicas=6`

Comprobamos el estado de los objetos:

```
kubectl -n cliente2-staging-ns get deploy,rs,pod -o wide
  NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                   IMAGES                    SELECTOR
  deployment.apps/cliente2-staging-deployment          6/6     6            6           4m27s   client2-staging-web          djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,track=stable
  deployment.apps/cliente2-staging-deployment-canary   0/0     0            0           2m51s   client2-staging-web-canary   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=staging,track=canary

  NAME                                                            DESIRED   CURRENT   READY   AGE     CONTAINERS                   IMAGES                    SELECTOR
  replicaset.apps/cliente2-staging-deployment-6787fdb4d4          6         6         6       4m27s   client2-staging-web          djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,pod-template-hash=6787fdb4d4,track=stable
  replicaset.apps/cliente2-staging-deployment-canary-7fc84455fc   0         0         0       2m51s   client2-staging-web-canary   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=staging,pod-template-hash=7fc84455fc,track=canary

  NAME                                               READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-staging-deployment-6787fdb4d4-bwwsn   1/1     Running   0          4m27s   10.244.104.4     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-g8ltd   1/1     Running   0          8s      10.244.104.7     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-ljvdh   1/1     Running   0          4m27s   10.244.104.3     node2   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-lrcbp   1/1     Running   0          8s      10.244.166.135   node1   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-qzrfk   1/1     Running   0          8s      10.244.166.136   node1   <none>           <none>
  pod/cliente2-staging-deployment-6787fdb4d4-tqdb6   1/1     Running   0          4m27s   10.244.166.131   node1   <none>           <none>
```

Después, eliminamos el deployment de `canary`.

`kubectl delete -f 07-staging-deployment-canary.yaml`

Actualizamos la aplicación del deployment `stable`:

`kubectl -n cliente2-staging-ns set image deployment/cliente2-staging-deployment client2-staging-web=djoven89/app-lab2:1.0.1 --record`

Revisamos los cambios:

```
kubectl -n cliente2-staging-ns get deploy,rs,pod -o wide
  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS            IMAGES                    SELECTOR
  deployment.apps/cliente2-staging-deployment   6/6     6            6           6m51s   client2-staging-web   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=staging,track=stable

  NAME                                                     DESIRED   CURRENT   READY   AGE     CONTAINERS            IMAGES                    SELECTOR
  replicaset.apps/cliente2-staging-deployment-6787fdb4d4   0         0         0       6m51s   client2-staging-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=staging,pod-template-hash=6787fdb4d4,track=stable
  replicaset.apps/cliente2-staging-deployment-8f9876759    6         6         6       115s    client2-staging-web   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=staging,pod-template-hash=8f9876759,track=stable

  NAME                                              READY   STATUS    RESTARTS   AGE    IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-staging-deployment-8f9876759-26hp9   1/1     Running   0          109s   10.244.166.137   node1   <none>           <none>
  pod/cliente2-staging-deployment-8f9876759-64snm   1/1     Running   0          96s    10.244.104.10    node2   <none>           <none>
  pod/cliente2-staging-deployment-8f9876759-9j52k   1/1     Running   0          114s   10.244.104.8     node2   <none>           <none>
  pod/cliente2-staging-deployment-8f9876759-dpqkk   1/1     Running   0          96s    10.244.166.142   node1   <none>           <none>
  pod/cliente2-staging-deployment-8f9876759-kvkvx   1/1     Running   0          111s   10.244.104.9     node2   <none>           <none>
  pod/cliente2-staging-deployment-8f9876759-xns2t   1/1     Running   0          115s   10.244.166.138   node1   <none>           <none>


kubectl -n cliente2-staging-ns describe pod cliente2-staging-deployment-8f9876759-64snm
  Name:         cliente2-staging-deployment-8f9876759-64snm
  Namespace:    cliente2-staging-ns
  Priority:     0
  Node:         node2/192.168.100.12
  Start Time:   Sun, 29 May 2022 15:26:43 +0000
  Labels:       app=flask
                customer=cliente2
                env=staging
                pod-template-hash=8f9876759
                track=stable
  Annotations:  cni.projectcalico.org/podIP: 10.244.104.10/32
                cni.projectcalico.org/podIPs: 10.244.104.10/32
                kubernetes.io/limit-ranger:
                  LimitRanger plugin set: cpu, memory request for container client2-staging-web; cpu, memory limit for container client2-staging-web
  Status:       Running
  IP:           10.244.104.10
  IPs:
    IP:           10.244.104.10
  Controlled By:  ReplicaSet/cliente2-staging-deployment-8f9876759
  Containers:
    client2-staging-web:
      Container ID:   docker://5d62d9653a49f0c2bf40b0e53e483fcebe0947526d94292d84d549bbdc43c552
      Image:          djoven89/app-lab2:1.0.1
      Image ID:       docker-pullable://djoven89/app-lab2@sha256:afb5f4deaae4ade91d7d5a7bfb62e0c964d28736a586aecc7a417bd3682a0c8e
      Port:           80/TCP
      Host Port:      0/TCP
      State:          Running
        Started:      Sun, 29 May 2022 15:26:46 +0000
      Ready:          True
      Restart Count:  0
      Limits:
        cpu:     200m
        memory:  250Mi
      Requests:
        cpu:        100m
        memory:     150Mi
      Environment:  <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gjnpl (ro)
  Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
  Volumes:
    kube-api-access-gjnpl:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      ConfigMapOptional:       <nil>
      DownwardAPI:             true
  QoS Class:                   Burstable
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                              node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  119s  default-scheduler  Successfully assigned cliente2-staging-ns/cliente2-staging-deployment-8f9876759-64snm to node2
    Normal  Pulling    118s  kubelet            Pulling image "djoven89/app-lab2:1.0.1"
    Normal  Pulled     116s  kubelet            Successfully pulled image "djoven89/app-lab2:1.0.1" in 2.002735544s
    Normal  Created    116s  kubelet            Created container client2-staging-web
    Normal  Started    116s  kubelet            Started container client2-staging-web
```

Y finalmente, accedemos al navegador web para confirmar la nueva versión de la aplicación.

### 4/4 Rollback

En caso de que tengamos que realizar un rollback y volver a la versión anterior de la aplicación, podremos usar el comando `rollout`.

Lo primero buscaremos la versión en el historia:

```
kubectl -n cliente2-staging-ns rollout history deployment cliente2-staging-deployment
  deployment.apps/cliente2-staging-deployment
  REVISION  CHANGE-CAUSE
  1         kubectl apply --filename=06-staging-deployment-stable.yaml --record=true
  2         kubectl set image deployment/cliente2-staging-deployment client2-staging-web=djoven89/app-lab2:1.0.1 --namespace=cliente2-staging-ns --record=true
```

Después, restauramos usando la revisión `1` en este caso:

`kubectl -n cliente2-staging-ns rollout undo deployment cliente2-staging-deployment --to-revision=1`

Comprobamos el estado de los objectos tras la acción anterior:

```
kubectl -n cliente2-staging-ns rollout status deployment cliente2-staging-deployment
  deployment "cliente2-staging-deployment" successfully rolled out


kubectl -n cliente2-staging-ns get deploy,rs,pod
  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/cliente2-staging-deployment   6/6     6            6           11m

  NAME                                                     DESIRED   CURRENT   READY   AGE
  replicaset.apps/cliente2-staging-deployment-6787fdb4d4   6         6         6       11m
  replicaset.apps/cliente2-staging-deployment-8f9876759    0         0         0       6m13s

  NAME                                               READY   STATUS    RESTARTS   AGE
  pod/cliente2-staging-deployment-6787fdb4d4-2hlt4   1/1     Running   0          43s
  pod/cliente2-staging-deployment-6787fdb4d4-6dwbc   1/1     Running   0          41s
  pod/cliente2-staging-deployment-6787fdb4d4-86dsh   1/1     Running   0          37s
  pod/cliente2-staging-deployment-6787fdb4d4-nc7c8   1/1     Running   0          32s
  pod/cliente2-staging-deployment-6787fdb4d4-qcxdp   1/1     Running   0          30s
  pod/cliente2-staging-deployment-6787fdb4d4-rgrbf   1/1     Running   0          35s


kubectl -n cliente2-staging-ns describe pod cliente2-staging-deployment-6787fdb4d4-qcxdp
  Name:         cliente2-staging-deployment-6787fdb4d4-qcxdp
  Namespace:    cliente2-staging-ns
  Priority:     0
  Node:         node2/192.168.100.12
  Start Time:   Sun, 29 May 2022 15:32:07 +0000
  Labels:       app=flask
                customer=cliente2
                env=staging
                pod-template-hash=6787fdb4d4
                track=stable
  Annotations:  cni.projectcalico.org/podIP: 10.244.104.15/32
                cni.projectcalico.org/podIPs: 10.244.104.15/32
                kubernetes.io/limit-ranger:
                  LimitRanger plugin set: cpu, memory request for container client2-staging-web; cpu, memory limit for container client2-staging-web
  Status:       Running
  IP:           10.244.104.15
  IPs:
    IP:           10.244.104.15
  Controlled By:  ReplicaSet/cliente2-staging-deployment-6787fdb4d4
  Containers:
    client2-staging-web:
      Container ID:   docker://469a10024008356eb156ace624b8528bfc08454b9bb6465e30b170294ebbcc10
      Image:          djoven89/app-lab2:1.0.0
      Image ID:       docker-pullable://djoven89/app-lab2@sha256:707c293b46b85e7c4a6331fedc5a2fbf636145d50963422800b9e7978dc4b625
      Port:           80/TCP
      Host Port:      0/TCP
      State:          Running
        Started:      Sun, 29 May 2022 15:32:10 +0000
      Ready:          True
      Restart Count:  0
      Limits:
        cpu:     200m
        memory:  250Mi
      Requests:
        cpu:        100m
        memory:     150Mi
      Environment:  <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-psrw7 (ro)
  Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
  Volumes:
    kube-api-access-psrw7:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      ConfigMapOptional:       <nil>
      DownwardAPI:             true
  QoS Class:                   Burstable
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                              node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type    Reason     Age   From               Message
    ----    ------     ----  ----               -------
    Normal  Scheduled  56s   default-scheduler  Successfully assigned cliente2-staging-ns/cliente2-staging-deployment-6787fdb4d4-qcxdp to node2
    Normal  Pulling    55s   kubelet            Pulling image "djoven89/app-lab2:1.0.0"
    Normal  Pulled     53s   kubelet            Successfully pulled image "djoven89/app-lab2:1.0.0" in 2.010712662s
    Normal  Created    53s   kubelet            Created container client2-staging-web
    Normal  Started    53s   kubelet            Started container client2-staging-web
```

Finalmente, confirmamos desde el navegador web como la aplicación vuelve a mostrar la versión anterior.

## Despliegue en el entorno prod

A continuación se indican las acciones a realizar para crear el entorno, desplegar la aplicación, testearla y actualizarla usando el método `blue-green deployment`.

**NOTA:** Dependiendo de los recursos de los nodos worker, es posible que sea necesario eliminar los entornos anteriores.

### 1/4 Preparación del entorno

Lo primero que haremos será crear el `namespace`, `quota` y `limitrange`.

`kubectl apply -f 08-prod-entorno.yaml`

Revisamos que se haya creado el `namespace` y sus requisitos:

```
kubectl get ns -l env=prod
  NAME               STATUS   AGE
  cliente2-prod-ns   Active   72s

kubectl describe ns cliente2-prod-ns
  Name:         cliente2-prod-ns
  Labels:       app=flask
                customer=client22
                env=prod
                kubernetes.io/metadata.name=cliente2-prod-ns
  Annotations:  customerEmail: daniel@prod.app2.lan
                customerName: Daniel
  Status:       Active

  Resource Quotas
    Name:                   cliente2-prod-quota
    Resource                Used  Hard
    --------                ---   ---
    configmaps              1     10
    limits.cpu              0     1500m
    limits.memory           0     2Gi
    persistentvolumeclaims  0     0
    pods                    0     12
    replicationcontrollers  0     2
    requests.cpu            0     1
    requests.memory         0     1536Mi
    secrets                 1     10
    services                0     2

  Resource Limits
  Type       Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
  ----       --------  ---    ---    ---------------  -------------  -----------------------
  Container  cpu       100m   1      100m             1              -
  Container  memory    150Mi  750Mi  150Mi            750Mi          -
```

### 2/4 Despliegue

Una vez que tenemos el entorno listo, crearemos el servicio. El cual tendrá los siguientes requisitos:

* Será de tipo `NodePort`.
* Expondrá el puerto `32022`.
* El puerto del contenedor será `8080`.
* Tendrá como etiqueta adicional, la versión de la aplicación. Por ejemplo: `version: 1.0.0`.

`kubectl apply -f 09-prod-service.yaml`

Comprobamos que el servicio haya sido creado por éxito:

```
kubectl -n cliente2-prod-ns get svc -o wide
  NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
  cliente2-prod-service   NodePort   10.98.105.134   <none>        8080:32022/TCP   23s   app=flask,customer=cliente2,env=prod,version=1.0.0


kubectl -n cliente2-prod-ns describe svc cliente2-prod-service
  Name:                     cliente2-prod-service
  Namespace:                cliente2-prod-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=prod
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=prod,version=1.0.0
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.98.105.134
  IPs:                      10.98.105.134
  Port:                     cliente2-prod-web-port  8080/TCP
  TargetPort:               80/TCP
  NodePort:                 cliente2-prod-web-port  32022/TCP
  Endpoints:                <none>
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>
```

A continuación, crearemos el `deployment` con las siguientes características:

* La aplicación constará de 6 pods en ejecucción.
* Los pods tendrán una etiqueta adicional con el número de versión: `version: 1.0.0`.

`kubectl apply -f 10-prod-deployment-blue.yaml`

Revisamos que los objetos hayan sido creados con éxito:

```
kubectl -n cliente2-staging-ns rollout status deployment cliente2-staging-deployment
  deployment "cliente2-staging-deployment" successfully rolled out


kubectl -n cliente2-prod-ns get deploy,rs,pod -o wide
  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS         IMAGES                    SELECTOR
  deployment.apps/cliente2-prod-deployment-01   6/6     6            6           33s   client2-prod-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=prod,version=1.0.0

  NAME                                                     DESIRED   CURRENT   READY   AGE   CONTAINERS         IMAGES                    SELECTOR
  replicaset.apps/cliente2-prod-deployment-01-84d8c86f95   6         6         6       33s   client2-prod-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=prod,pod-template-hash=84d8c86f95,version=1.0.0

  NAME                                               READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-prod-deployment-01-84d8c86f95-2s59h   1/1     Running   0          32s   10.244.166.140   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-78j5x   1/1     Running   0          33s   10.244.104.18    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-9xk64   1/1     Running   0          33s   10.244.104.19    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-l7gmq   1/1     Running   0          33s   10.244.166.139   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-ssqjt   1/1     Running   0          32s   10.244.166.150   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-xckcm   1/1     Running   0          32s   10.244.104.17    node2   <none>           <none>


kubectl describe ns cliente2-prod-ns
  Name:         cliente2-prod-ns
  Labels:       app=flask
                customer=client22
                env=prod
                kubernetes.io/metadata.name=cliente2-prod-ns
  Annotations:  customerEmail: daniel@prod.app2.lan
                customerName: Daniel
  Status:       Active

  Resource Quotas
    Name:                   cliente2-prod-quota
    Resource                Used   Hard
    --------                ---    ---
    configmaps              1      10
    limits.cpu              600m   1500m
    limits.memory           900Mi  2Gi
    persistentvolumeclaims  0      0
    pods                    6      12
    replicationcontrollers  0      2
    requests.cpu            600m   1
    requests.memory         900Mi  1536Mi
    secrets                 1      10
    services                1      2

  Resource Limits
  Type       Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
  ----       --------  ---    ---    ---------------  -------------  -----------------------
  Container  cpu       100m   1      100m             100m           -
  Container  memory    150Mi  750Mi  150Mi            150Mi          -


kubectl -n cliente2-prod-ns describe svc cliente2-prod-service 
  Name:                     cliente2-prod-service
  Namespace:                cliente2-prod-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=prod
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=prod,version=1.0.0
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.98.105.134
  IPs:                      10.98.105.134
  Port:                     cliente2-prod-web-port  8080/TCP
  TargetPort:               80/TCP
  NodePort:                 cliente2-prod-web-port  32022/TCP
  Endpoints:                10.244.104.17:80,10.244.104.18:80,10.244.104.19:80 + 3 more...
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>


kubectl -n cliente2-prod-ns describe pod cliente2-prod-deployment-01-84d8c86f95-2s59h
  Name:         cliente2-prod-deployment-01-84d8c86f95-2s59h
  Namespace:    cliente2-prod-ns
  Priority:     0
  Node:         node1/192.168.100.11
  Start Time:   Sun, 29 May 2022 15:54:57 +0000
  Labels:       app=flask
                customer=cliente2
                env=prod
                pod-template-hash=84d8c86f95
                version=1.0.0
  Annotations:  cni.projectcalico.org/podIP: 10.244.166.140/32
                cni.projectcalico.org/podIPs: 10.244.166.140/32
                kubernetes.io/limit-ranger:
                  LimitRanger plugin set: cpu, memory request for container client2-prod-web; cpu, memory limit for container client2-prod-web
  Status:       Running
  IP:           10.244.166.140
  IPs:
    IP:           10.244.166.140
  Controlled By:  ReplicaSet/cliente2-prod-deployment-01-84d8c86f95
  Containers:
    client2-prod-web:
      Container ID:   docker://74fcce8b1d8d4365636cb00a399335efb0ba85c4869fe93bc98e4c9f554da608
      Image:          djoven89/app-lab2:1.0.0
      Image ID:       docker-pullable://djoven89/app-lab2@sha256:707c293b46b85e7c4a6331fedc5a2fbf636145d50963422800b9e7978dc4b625
      Port:           80/TCP
      Host Port:      0/TCP
      State:          Running
        Started:      Sun, 29 May 2022 15:55:00 +0000
      Ready:          True
      Restart Count:  0
      Limits:
        cpu:     100m
        memory:  150Mi
      Requests:
        cpu:        100m
        memory:     150Mi
      Environment:  <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ntcfx (ro)
  Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
  Volumes:
    kube-api-access-ntcfx:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      ConfigMapOptional:       <nil>
      DownwardAPI:             true
  QoS Class:                   Guaranteed
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                              node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:
    Type     Reason       Age   From               Message
    ----     ------       ----  ----               -------
    Normal   Scheduled    104s  default-scheduler  Successfully assigned cliente2-prod-ns/cliente2-prod-deployment-01-84d8c86f95-2s59h to node1
    Warning  FailedMount  103s  kubelet            MountVolume.SetUp failed for volume "kube-api-access-ntcfx" : failed to sync configmap cache: timed out waiting for the condition
    Normal   Pulled       102s  kubelet            Container image "djoven89/app-lab2:1.0.0" already present on machine
    Normal   Created      101s  kubelet            Created container client2-prod-web
    Normal   Started      101s  kubelet            Started container client2-prod-web

```

Finalmente, vamos al navegador web y accedemos al servidor `master` y con el puerto `32022`.

### 3/4 Actualización

Para actualizar, creamos otro deployment con las mismas características que el existente, con la diferencia que la etiqueta que contiene la versión será distinta.

`kubectl apply -f 11-prod-deployment-green.yaml`

Comprobamos los recursos creados:

```
kubectl -n cliente2-prod-ns get deploy,rs,pod -o wide
  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS         IMAGES                    SELECTOR
  deployment.apps/cliente2-prod-deployment-01   6/6     6            6           11m     client2-prod-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=prod,version=1.0.0
  deployment.apps/cliente2-prod-deployment-02   6/6     6            6           2m19s   client2-prod-web   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=prod,version=1.0.1

  NAME                                                     DESIRED   CURRENT   READY   AGE     CONTAINERS         IMAGES                    SELECTOR
  replicaset.apps/cliente2-prod-deployment-01-84d8c86f95   6         6         6       11m     client2-prod-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=prod,pod-template-hash=84d8c86f95,version=1.0.0
  replicaset.apps/cliente2-prod-deployment-02-5dcc4ffbdc   6         6         6       2m19s   client2-prod-web   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=prod,pod-template-hash=5dcc4ffbdc,version=1.0.1

  NAME                                               READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-prod-deployment-01-84d8c86f95-2s59h   1/1     Running   0          11m     10.244.166.140   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-78j5x   1/1     Running   0          11m     10.244.104.18    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-9xk64   1/1     Running   0          11m     10.244.104.19    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-l7gmq   1/1     Running   0          11m     10.244.166.139   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-ssqjt   1/1     Running   0          11m     10.244.166.150   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-xckcm   1/1     Running   0          11m     10.244.104.17    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-8npth   1/1     Running   0          2m19s   10.244.166.148   node1   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-9crdw   1/1     Running   0          2m19s   10.244.104.24    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-gc59s   1/1     Running   0          2m19s   10.244.104.13    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-hwfrg   1/1     Running   0          2m19s   10.244.166.145   node1   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-kwd5p   1/1     Running   0          2m19s   10.244.104.23    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-rqkf9   1/1     Running   0          2m19s   10.244.166.146   node1   <none>           <none>
```

Editamos la versión de la aplicación en el selector del servicio:

`kubectl -n cliente2-prod-ns edit svc cliente2-prod-service`

Revisamos el estado del servicio tras el cambio:

```
kubectl -n cliente2-prod-ns describe svc cliente2-prod-service
  Name:                     cliente2-prod-service
  Namespace:                cliente2-prod-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=prod
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=prod,version=1.0.1
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.98.105.134
  IPs:                      10.98.105.134
  Port:                     cliente2-prod-web-port  8080/TCP
  TargetPort:               8080/TCP
  NodePort:                 cliente2-prod-web-port  32022/TCP
  Endpoints:                10.244.104.13:8080,10.244.104.23:8080,10.244.104.24:8080 + 3 more...
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>
```

Después, desde el navegador web revisamos que sólo vemos la nueva versión de la aplicación.

Tan pronto hayamos confirmado que la aplicación es estable, podremos proceder a eliminar el deployment que contiene la versión antigua de la aplicación.

`kubectl delete -f 10-prod-deployment-blue.yaml`

Finalmente, comprobamos que no haya rastro de él:

```
kubectl -n cliente2-prod-ns get deploy,rs,pod -o wide
  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS         IMAGES                    SELECTOR
  deployment.apps/cliente2-prod-deployment-02   6/6     6            6           6m31s   client2-prod-web   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=prod,version=1.0.1

  NAME                                                     DESIRED   CURRENT   READY   AGE     CONTAINERS         IMAGES                    SELECTOR
  replicaset.apps/cliente2-prod-deployment-02-5dcc4ffbdc   6         6         6       6m31s   client2-prod-web   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=prod,pod-template-hash=5dcc4ffbdc,version=1.0.1

  NAME                                               READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-8npth   1/1     Running   0          6m31s   10.244.166.148   node1   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-9crdw   1/1     Running   0          6m31s   10.244.104.24    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-gc59s   1/1     Running   0          6m31s   10.244.104.13    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-hwfrg   1/1     Running   0          6m31s   10.244.166.145   node1   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-kwd5p   1/1     Running   0          6m31s   10.244.104.23    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-rqkf9   1/1     Running   0          6m31s   10.244.166.146   node1   <none>           <none>
```

### 4/4 Rollback

En caso de necesitar realizar un rollback, tendremos que volver a crear el deployment (si lo llegamos a borrar tras confirmar que la aplicación era estable) y modificar el servicio nuevamente.

En caso de que eliminasemos el deployment, volveremos a crearlo:

`kubectl apply -f 10-prod-deployment-blue.yaml`

Comprobamos que los recursos se hayan creado correctamente:

```
kubectl -n cliente2-prod-ns get deploy,rs,pod -o wide
  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS         IMAGES                    SELECTOR
  deployment.apps/cliente2-prod-deployment-01   6/6     6            6           6s    client2-prod-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=prod,version=1.0.0
  deployment.apps/cliente2-prod-deployment-02   6/6     6            6           9m    client2-prod-web   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=prod,version=1.0.1

  NAME                                                     DESIRED   CURRENT   READY   AGE   CONTAINERS         IMAGES                    SELECTOR
  replicaset.apps/cliente2-prod-deployment-01-84d8c86f95   6         6         6       6s    client2-prod-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=prod,pod-template-hash=84d8c86f95,version=1.0.0
  replicaset.apps/cliente2-prod-deployment-02-5dcc4ffbdc   6         6         6       9m    client2-prod-web   djoven89/app-lab2:1.0.1   app=flask,customer=cliente2,env=prod,pod-template-hash=5dcc4ffbdc,version=1.0.1

  NAME                                               READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-prod-deployment-01-84d8c86f95-2c7f2   1/1     Running   0          6s    10.244.166.149   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-7vmgq   1/1     Running   0          6s    10.244.104.22    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-cj2g6   1/1     Running   0          6s    10.244.104.21    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-hqjdd   1/1     Running   0          6s    10.244.104.12    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-k7km7   1/1     Running   0          6s    10.244.166.147   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-zpq6z   1/1     Running   0          6s    10.244.166.152   node1   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-8npth   1/1     Running   0          9m    10.244.166.148   node1   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-9crdw   1/1     Running   0          9m    10.244.104.24    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-gc59s   1/1     Running   0          9m    10.244.104.13    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-hwfrg   1/1     Running   0          9m    10.244.166.145   node1   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-kwd5p   1/1     Running   0          9m    10.244.104.23    node2   <none>           <none>
  pod/cliente2-prod-deployment-02-5dcc4ffbdc-rqkf9   1/1     Running   0          9m    10.244.166.146   node1   <none>           <none>
```

Después, modificamos la etiqueta `version` del selector del servicio e indicamos la versión antigua:

`kubectl -n cliente2-prod-ns edit svc cliente2-prod-service`

Revisamos el estado del servicio tras el cambio:

```
kubectl -n cliente2-prod-ns describe svc cliente2-prod-service
  Name:                     cliente2-prod-service
  Namespace:                cliente2-prod-ns
  Labels:                   app=flask
                            customer=cliente2
                            env=prod
  Annotations:              <none>
  Selector:                 app=flask,customer=cliente2,env=prod,version=1.0.0
  Type:                     NodePort
  IP Family Policy:         SingleStack
  IP Families:              IPv4
  IP:                       10.98.105.134
  IPs:                      10.98.105.134
  Port:                     cliente2-prod-web-port  8080/TCP
  TargetPort:               8080/TCP
  NodePort:                 cliente2-prod-web-port  32022/TCP
  Endpoints:                10.244.104.12:8080,10.244.104.21:8080,10.244.104.22:8080 + 3 more...
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>
```

A continuación, desde el navegador web confirmaremos como la aplicación vuelve a ser la antigua.

Finalmente, eliminaremos el deployment que contiene la nueva versión que no era estable.

`kubectl delete -f 10-prod-deployment-blue.yaml`

Y confirmaremos que los recursos hayan sido borrados:

```
kubectl -n cliente2-prod-ns get deploy,rs,pod -o wide
  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS         IMAGES                    SELECTOR
  deployment.apps/cliente2-prod-deployment-01   6/6     6            6           3m21s   client2-prod-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=prod,version=1.0.0

  NAME                                                     DESIRED   CURRENT   READY   AGE     CONTAINERS         IMAGES                    SELECTOR
  replicaset.apps/cliente2-prod-deployment-01-84d8c86f95   6         6         6       3m21s   client2-prod-web   djoven89/app-lab2:1.0.0   app=flask,customer=cliente2,env=prod,pod-template-hash=84d8c86f95,version=1.0.0

  NAME                                               READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
  pod/cliente2-prod-deployment-01-84d8c86f95-2c7f2   1/1     Running   0          3m21s   10.244.166.149   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-7vmgq   1/1     Running   0          3m21s   10.244.104.22    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-cj2g6   1/1     Running   0          3m21s   10.244.104.21    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-hqjdd   1/1     Running   0          3m21s   10.244.104.12    node2   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-k7km7   1/1     Running   0          3m21s   10.244.166.147   node1   <none>           <none>
  pod/cliente2-prod-deployment-01-84d8c86f95-zpq6z   1/1     Running   0          3m21s   10.244.166.152   node1   <none>           <none>
```

## Limpieza del entorno

Para limpiar el entorno, podemos ejecutar el siguiente bucle dentro del directorio donde se encuentra todos los archivos.

`for i in {11..1}; do kubectl delete -f $i-*.yaml; done`
