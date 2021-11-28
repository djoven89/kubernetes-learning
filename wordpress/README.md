# Aplicación Wordpress

En este ejercicio se pretende simular la creación de un entorno aislado dentro del cluster de Kubernetes para un nuevo cliente, el cual desea alojar allí un Wordpress.

Para este ejercicio se usarán los siguientes objetos de Kubernetes:

* Namespace.
* ResourceQuota.
* LimitRange.
* ConfigMap.
* Secret.
* PersistentVolume con NFS.
* PersistentVolumeClaim.
* Service.
* Deployment.
* Ingress.

Para la realización del ejercicio, se usará el entorno del curso, compuesto por un nodo master y un nodo worker.


## Requisitos previos

* Añadir un disco de al menos 1.5 GB al nodo master.

* Usar la versión `1.21.3` en el cluster.
    * Es tan sencillo como modificar la variable `KUBERNETES_VERSION` del archivo `Vagrantfile` a `1.21.3` y desplegar nuevamente el cluster.


## Preparación del cluster

A continuación se detallan los pasos a realizar para configurar el cluster y dejarlo listo para comenzar con el ejercicio. Las configuraciones previas son:

* Volúmenes que serán usados como persistentVolume a través de NFS.
* Traefik como ingress controller.
* Métricas vía metrics-server.


### Configuración de los volúmenes vía NFS

Tras haber añadido un disco secundario al nodo `master`, se particionará y gestionará a través de LVM. A continuación se configurará el servicio NFS y se compartirán los volúmenes LVM para que sean usados como persistentVolume en Kubernetes.


### Preparando los volúmenes

Lo primero que haremos será particionar el disco recién añadido:

    echo -e 'n\np\n\n\n\nt\n8e\nw' | fdisk /dev/sdb

  **NOTA:** Lo que se hace es crear una partición primaria usando todo el espacio disponible, y se etiqueta como `lvm`.

Después, añadimos la partición del disco a LVM para crear los volúmenes lógicos que compartiremos vía NFS. El volumen group lo llamaremos `vg_nfs`:

    pvcreate /dev/sdb1
    vgcreate vg_nfs /dev/sdb1

Creamos dos volúmenes lógicos en LVM, uno para Mysql y otro Wordpress:

    lvcreate -L 500M -n lv_vol01 vg_nfs
    lvcreate -L 1G -n lv_vol02 vg_nfs

Una vez que tenemos los volúmenes lógicos, creamos el sistema de archivo `ext4` en cada uno de ellos:

    for i in lv_vol0{1..2}; do mkfs -t ext4 /dev/vg_nfs/$i; done

Y montamos los volúmenes en `/srv/`:

    for i in lv_vol0{1..2}; do mkdir -v /srv/$i && mount -t ext4 /dev/vg_nfs/$i /srv/$i; done

Después, persistimos el montaje modificando el archivo `/etc/fstab`:

    for i in lv_vol0{1..2}; do echo "/dev/vg_nfs/$i /srv/$i ext4 defaults 0 0" >> /etc/fstab; done

Finalmente, comprobamos que todo sea correcto:

    mount -a

    df -h /srv/*

        Filesystem                   Size  Used Avail Use% Mounted on
        /dev/mapper/vg_nfs-lv_vol01  469M  768K  433M   1% /srv/lv_vol01
        /dev/mapper/vg_nfs-lv_vol02  976M  2.6M  907M   1% /srv/lv_vol02


### Configuración del servicio NFS

Lo primero que haremos será instalar el paquete `nfs-kernel-server`:

    apt-get update
    apt-get install nfs-kernel-server -y

Después, iniciamos el servicio y lo habilitamos:

    systemctl start nfs-server
    systemctl enable nfs-server

A continuación, establecemos los permisos a los directorios que queremos usar como PV en Kubernetes:

    for i in /srv/lv_vol0{1..2}; do chmod 0777 $i; chown nobody:nogroup $i; done

Lo siguiente será añadir estos recursos a NFS, para ello, habrá que modificar el archivo `/etc/exports`:

    for i in /srv/lv_vol0{1..2}; do echo "$i 192.168.100.0/24(rw,sync,no_root_squash,no_subtree_check)" >> /etc/exports; done

Finalmente, publicamos los recursos recién configurados:

    exportfs -var


## Configuración del ingress controller

Utilizaremos [Traefik 1.7] para la configuración del ingress controller.

[Traefik 1.7]: https://doc.traefik.io/traefik/v1.7/user-guide/kubernetes/#kubernetes-ingress-controller

Para implementarlo, simplemente tendremos que importar el siguiente manifiesto:

    kubectl apply -f 01-app-ingress-controller.yaml

Una vez que haya sido importado, podremos comprobar que se haya creado un pod en cada nodo (salvo en el master):

    kubectl --namespace=kube-system get pods -l name=traefik-ingress-lb -o wide

        NAME                               READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
        traefik-ingress-controller-8k6jd   1/1     Running   0          15s   10.244.104.1     node2   <none>           <none>
        traefik-ingress-controller-lwjkz   1/1     Running   0          15s   10.244.166.129   node1   <none>           <none>

También podremos verificar que tenemos acceso al dashboard de Traefik a través de las siguientes URLS:

* http://192.168.100.11:8080
* http://192.168.100.12:8080


## Configuración de las métricas

Para una monitorización sencilla de los nodos y los pods, usaremos [metrics-server].

[metrics-server]: https://github.com/kubernetes-sigs/metrics-server

Para su implementación, tendremos que importar el manifiesto:

    kubectl apply -f 02-app-metrics-server.yaml

Después, revisaremos que se hayan creado los objetos correctamente:

    kubectl -n kube-system get sa,service,deploy,pod -l k8s-app=metrics-server -o wide

        NAME                            SECRETS   AGE
        serviceaccount/metrics-server   1         87s

        NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
        service/metrics-server   ClusterIP   10.109.163.142   <none>        443/TCP   86s   k8s-app=metrics-server

        NAME                             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS       IMAGES                                            SELECTOR
        deployment.apps/metrics-server   1/1     1            1           86s   metrics-server   k8s.gcr.io/metrics-server/metrics-server:v0.5.2   k8s-app=metrics-server

        NAME                                  READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
        pod/metrics-server-5b6dd75459-zjmjn   1/1     Running   0          86s   10.244.166.130   node1   <none>           <none>


Finalmente, verificaremos que podemos obtener las las métricas de los nodos y pods:

    kubectl top nodes --sort-by=cpu

        NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
        master   297m         14%    1194Mi          63%
        node1    76m          7%     790Mi           41%
        node2    86m          8%     800Mi           42%

    kubectl top pods -A --sort-by=memory

        NAMESPACE     NAME                                      CPU(cores)   MEMORY(bytes)
        kube-system   kube-apiserver-master                     90m          356Mi
        kube-system   kube-controller-manager-master            35m          71Mi
        kube-system   etcd-master                               18m          41Mi
        kube-system   calico-node-bgn82                         35m          40Mi
        kube-system   calico-node-qwt24                         26m          38Mi
        kube-system   calico-node-rg79q                         26m          37Mi
        kube-system   kube-proxy-w4kwl                          1m           29Mi
        kube-system   kube-scheduler-master                     5m           22Mi
        kube-system   coredns-558bd4d5db-fnqmg                  4m           17Mi
        kube-system   metrics-server-5b6dd75459-qzn2n           5m           15Mi
        kube-system   coredns-558bd4d5db-ppxh8                  4m           15Mi
        kube-system   kube-proxy-dsncb                          1m           11Mi
        kube-system   kube-proxy-kw664                          1m           11Mi
        kube-system   calico-kube-controllers-f4c65d86b-gqsqt   1m           7Mi


## Creación de la aplicación

La aplicación estará compuesta por:

* Mysql.
* PHPMyAdmin.
* Wordpress.

Además, se pretende aislar al cliente en un namespace y limitar los recursos que puede usar.

En las siguientes secciones se irá detallando en orden la construcción completa del entorno del cliente.


## 1/5 Creación de los persistent volumen

La primera tarea que haremos será crear los persistent volumes, ya que estos no tienen un namespace asociado.

Se crearán 2 persistent volumes, uno para Mysql y otro para Apache. Sus característica son:

**Mysql:**

* El recurso tendrá las siguientes etiquetas:
    * volume  = vol01
    * app     = wordpress
    * tier    = mysql
    * version = 1.0.0.
* El PV se llamará `nfs-mysql`.
* Tendrá una capacidad de 500 MB.
* Sus datos deberán persistir siempre.
* Sólo podrá acceder un único pod al PV.
* La ruta al NFS será `/srv/lv_vol01`.
* El PV será reclamado vía nombre, el cual será `nfs-mysql`.

**Apache:**

* El recurso tendrá las siguientes etiquetas:
    * volume  = vol01
    * app     = wordpress
    * tier    = apache
    * version = 1.0.0.
* El PV se llamará `nfs-apache`.
* Tendrá una capacidad de 1 GB.
* Sus datos deberán persistir siempre.
* Sólo podrá acceder un único pod.
* La ruta al NFS será `/srv/lv_vol02`.
* El PV será reclamado vía nombre, el cual será `nfs-apache`.

Si se configuró correctamente, se debería de tener un resultado similar al siguiente:

    kubectl get pv -o wide

        NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   REASON   AGE   VOLUMEMODE
        nfs-apache   1Gi        RWO            Retain           Available                       nfs-apache              54s   Filesystem
        nfs-mysql    500Mi      RWO            Retain           Available                       nfs-mysql               55s   Filesystem

**La solución** se encuentra en el archivo `03-app-persistent-volume.yaml`.


## 2/5 Aislamiento del cliente

Queremos poder aislar el entorno del cliente en nuestro cluster y limitar el uso de recursos que puede usar. Para ello, usaremos los objetos: `namespace`, `quota` y `limitrange`.

Las características a implementar para cada objeto son:

**Namespace:**

* Se llamará `cliente1-app`.

* Tendrá las siguientes etiquetas:
    * customer  = cliente11
    * app       = wordpress

* Tendrá las siguientes anotaciones:
    * customerName  = Daniel
    * customerEmail = daniel@app.lan

**Quotas:**

* Se llamará `cliente1-quota`.

* Tendrá las siguientes etiquetas:
    * customer  = cliente11
    * app       = wordpress

* El máximo de recursos que podrá usar son:
    * 8 pods.
    * 6 replicas.
    * 6 servicios.
    * 12 secretos.
    * 12 configmaps.
    * 4 persistent volume claims.
    * 1.5 core de CPU como limit.
    * 1 core de CPU como request.
    * 1.5 GB de RAM como limit.
    * 1 GG de RAM como request.

**LimitRange:**

* Se llamará `cliente1-limitrange`

* Tendrá las siguientes etiquetas:
    * customer  = cliente11
    * app       = wordpress

* Los valores por defecto para pods y pvcs:
    * Requests:
        * CPU: 0.1
        * RAM: 150 MB
        * PVC: 100 MB
    * Limits:
        * CPU: 0.5
        * RAM: 500 MB
        * PVC: 1 GB

* Los valores mínimos para pods y pvcs:
    * CPU: 0.1
    * RAM: 150 MB
    * PVC: 100 MB

* Los valores máximos para pods y pvcs:
    * CPU: 0.5
    * RAM: 500 MB
    * PVC: 1 GB

Si todo fue bien, se deberá tener un resultado como el siguiente:

    kubectl describe namespaces cliente1-app

        Name:         cliente1-app
        Labels:       app=wordpress
                    customer=cliente1
                    kubernetes.io/metadata.name=cliente1-app
        Annotations:  customerEmail: daniel@app.lan
                    customerName: Daniel
        Status:       Active

        Resource Quotas
        Name:                   cliente1-quota
        Resource                Used  Hard
        --------                ---   ---
        configmaps              1     18
        limits.cpu              0     1500m
        limits.memory           0     2Gi
        persistentvolumeclaims  0     3
        pods                    0     6
        replicationcontrollers  0     2
        requests.cpu            0     1
        requests.memory         0     1Gi
        secrets                 1     18
        services                0     2

        Resource Limits
        Type                   Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
        ----                   --------  ---    ---    ---------------  -------------  -----------------------
        Container              cpu       200m   500m   200m             500m           -
        Container              memory    150Mi  500Mi  150Mi            500Mi          -
        PersistentVolumeClaim  storage   100Mi  2Gi    100Mi            2Gi            -

**La solución** se encuentra en el archivo `04-app-namespace.yaml`.


## 3/5 Despliegue de Mysql

La primera aplicación que configuraremos será Mysql, el cual se quiere que el almacenamiento sea persistente vía NFS. A continuación los requsitos a cumplir:

* Todos los objetos tienen que tener las siguientes etiquetas:
    * app     = wordpress
    * tier    = mysql
    * version = 1.0.0

* Crear un configmap llamado `mysql-configmap` que contenga la siguiente información:
    * db_host = mysql-service
    * db_port = 3306
    * db_name = db_app
    * db_user = dba_app

* Crear un secret llamado `mysql-secret` para:
    * db_root_password = root
    * db_user_password = dba

* Crear un servicio llamado `mysql-service` de tipo `ClusterIp`, usando el puerto `3306/TCP` llamado `mysql-port`. Además deberá de tener un selector cuyas etiquetas sean:
    * app  = wordpress
    * tier = mysql
    * version = 1.0.0

* Crear un persisent volume clain llamado `mysql-pvc` que tenga las siguientes características:
    * Se ha de vincular con el persistent volume `nfs-mysql`.
    * Se necesitarán 500MB de espacio.
    * Sólo podrá ser usado por un único pod.

* El pod tendrá deberá de cumplir con:
    * Usar la imagen [mysql:8].
    * El contenedor deberá de llamarse `mysql`.
    * Persisir el directorio `/var/lib/mysql/`.
    * Usar el puerto `3306/TCP` y llamarlo `mysql-port`.
    * Definir las variables de entorno cuyo valores deberá ser obtenido por el ConfigMap y el Secret:
        * MYSQL_HOST
        * MYSQL_PORT
        * MYSQL_DATABASE
        * MYSQL_ROOT_PASSWORD
        * MYSQL_USER
        * MYSQL_PASSWORD
    * Tendrá un request de recursos de:
        * CPU = 0.2
        * RAM = 400Mi

[mysql:8]: https://hub.docker.com/_/mysql

Si todo fue bien, se deberá tener un resultado como el siguiente:

    kubectl get cm,secret,pvc,deploy,pod,service -l app=wordpress

        NAME                        DATA   AGE
        configmap/mysql-configmap   4      3m14s

        NAME                  TYPE     DATA   AGE
        secret/mysql-secret   Opaque   2      3m14s

        NAME                              STATUS   VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS   AGE
        persistentvolumeclaim/mysql-pvc   Bound    nfs-mysql   500Mi      RWO            nfs-mysql      3m14s

        NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/mysql-deployment   1/1     1            1           3m14s

        NAME                                    READY   STATUS    RESTARTS   AGE
        pod/mysql-deployment-76948df4f6-lw7pj   1/1     Running   0          3m14s

        NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
        service/mysql-service   ClusterIP   10.99.79.95   <none>        3306/TCP   3m14s

**La solución** se encuentra en el archivo `05-app-mysql.yaml`.


## 4/5 Despliegue de PHPmyadmin

La siguiente aplicación a desplegar es PHPMyAdmin, la cual debe cumplir con los siguientes requisitos:

* Todos los objetos tienen que tener las siguientes etiquetas:
    * app  = wordpress
    * tier = phpmyadmin
    * version = 1.0.0

* Crear un servicio llamado `phpmyadmin-service` de tipo `ClusterIp`, usando el puerto `80/TCP` llamado `phpmyadmin-port`. Además deberá de tener un selector cuyas etiquetas sean:
    * app  = wordpress
    * tier = phpmyadmin
    * version = 1.0.0

* El pod tendrá deberá de cumplir con:
    * Usar la imagen [phpmyadmin:5.1.1-apache].
    * El contenedor deberá de llamarse `phpmyadmin`.
    * Usar el puerto `80/TCP`.
    * Definir las variables de entorno cuyo dos últimos valores deberá ser obtenido por el ConfigMap:
        * PMA_ARBITRARY = 1
        * PMA_VERBOSE = 1
        * PMA_HOST
        * PMA_PORT

[phpmyadmin:5.1.1-apache]: https://hub.docker.com/_/phpmyadmin

* Crear un ingress llamado `phpmyadmin-ingress` el cual sea accesible vía las siguientes URLs:
    * http://phpmyadmin.app.lan.
    * http://phpmyadmin.app.lan.192.168.100.11.nip.io
    * http://phpmyadmin.app.lan.192.168.100.12.nip.io

Si todo fue bien, se deberá tener un resultado como el siguiente:

    kubectl get deploy,pod,service,ingress -l tier=phpmyadmin

        NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/phpmyadmin-deployment   1/1     1            1           9m25s

        NAME                                         READY   STATUS    RESTARTS   AGE
        pod/phpmyadmin-deployment-545cf8fdc8-7gxd2   1/1     Running   0          9m25s

        NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
        service/phpmyadmin-service   ClusterIP   10.108.159.248   <none>        80/TCP    9m25s

        NAME                                           CLASS    HOSTS                                                                                                  ADDRESS   PORTS   AGE
        ingress.networking.k8s.io/phpmyadmin-ingress   <none>   phpmyadmin.app.lan,phpmyadmin.app.lan.192.168.100.11.nip.io,phpmyadmin.app.lan.192.168.100.12.nip.io             80      6m5s


**La solución** se encuentra en el archivo `06-app-phpmyadmin.yaml`.


## 5/5 Despliegue de Wordpress

La última aplicación a desplegar será el propio Wordpress. A continuación los requisitos a cumplir:

* Todos los objetos tienen que tener las siguientes etiquetas:
    * app  = wordpress
    * tier = apache
    * version = 1.0.0

* Crear un servicio llamado `apache-service` de tipo `ClusterIp`, usando el puerto `80/TCP` llamado `apache-port`. Además deberá de tener un selector cuyas etiquetas sean:
    * app  = wordpress
    * tier = apache
    * version = 1.0.0

* Crear un persisent volume claim llamado `apache-pvc` que tenga las siguientes características:
    * Se ha de vincular con el persistent volume `nfs-apache`.
    * Se necesitarán 1Gi de espacio.
    * Sólo podrá ser usado por un único pod.

* El pod tendrá deberá de cumplir con:
    * Usar la imagen [wordpress:php7.4-apache].
    * El contenedor deberá de llamarse `apache`.
    * Usar el puerto `80/TCP`.
    * Persisir el directorio `/var/www/html/`.
    * Variables de entorno cuyo valores deberá ser obtenido por el ConfigMap y el Secret:
        * WORDPRESS_DB_HOST
        * WORDPRESS_DB_NAME
        * WORDPRESS_DB_USER
        * WORDPRESS_DB_PASSWORD
    * Tendrá un request de recursos de:
        * CPU = 0.5
        * RAM = 400Mi

[wordpress:php7.4-apache]: https://hub.docker.com/_/wordpress

* Crear un ingress llamado `apache-ingress` el cual sea accesible vía las siguientes URLs:
    * http://app.lan.
    * http://app.lan.192.168.100.11.nip.io
    * http://app.lan.192.168.100.12.nip.io

Si todo fue bien, se deberá tener un resultado como el siguiente:

    kubectl get deploy,pod,service,ingress -l tier=apache -o wide

        NAME                               STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
        persistentvolumeclaim/apache-pvc   Bound    nfs-apache   1Gi        RWO            nfs-apache     17s   Filesystem

        NAME                                READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                    SELECTOR
        deployment.apps/apache-deployment   1/1     1            1           17s   apache       wordpress:php7.4-apache   app=wordpress,tier=apache,version=1.0.0

        NAME                                     READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
        pod/apache-deployment-5c5776cd65-s5q55   1/1     Running   0          17s   10.244.166.132   node1   <none>           <none>

        NAME                     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE   SELECTOR
        service/apache-service   NodePort   10.101.24.166   <none>        80:30081/TCP   17s   app=wordpress,tier=apache,version=1.0.0


**La solución** se encuentra en el archivo `07-app-apache.yaml`.

## Resultado final

Una vez que todo haya sido desplegado, se deberían de obtener resultados similares a los siguientes:

* **Objetos creados:**

      kubectl get cm,secret,deploy,pod,service,ingress -o wide -n cliente1-app -l app=wordpress

        NAME                        DATA   AGE
        configmap/mysql-configmap   4      4m19s

        NAME                  TYPE     DATA   AGE
        secret/mysql-secret   Opaque   2      4m19s

        NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                    SELECTOR
        deployment.apps/apache-deployment       1/1     1            1           2m12s   apache       wordpress:php7.4-apache   app=wordpress,tier=apache,version=1.0.0
        deployment.apps/mysql-deployment        1/1     1            1           4m19s   mysql        mysql:8                   version=1.0.0
        deployment.apps/phpmyadmin-deployment   1/1     1            1           3m25s   phpmyadmin   phpmyadmin:5.1.1-apache   app=wordpress,tier=phpmyadmin,version=1.0.0

        NAME                                         READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
        pod/apache-deployment-6f75686b75-mm6rf       1/1     Running   0          2m12s   10.244.104.9     node2   <none>           <none>
        pod/mysql-deployment-596f8bcdcc-jvnt4        1/1     Running   0          4m19s   10.244.104.8     node2   <none>           <none>
        pod/phpmyadmin-deployment-545cf8fdc8-2gr2d   1/1     Running   0          3m25s   10.244.166.134   node1   <none>           <none>

        NAME                         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE     SELECTOR
        service/apache-service       NodePort    10.99.12.172   <none>        80:32732/TCP   2m12s   app=wordpress,tier=apache,version=1.0.0
        service/mysql-service        ClusterIP   10.99.44.242   <none>        3306/TCP       4m19s   tier=mysql
        service/phpmyadmin-service   ClusterIP   10.100.90.84   <none>        80/TCP         3m25s   app=wordpress,tier=phpmyadmin,version=1.0.0

        NAME                                           CLASS    HOSTS                                                                                                  ADDRESS   PORTS   AGE
        ingress.networking.k8s.io/apache-ingress       <none>   app.lan,app.lan.192.168.100.11.nip.io,app.lan.192.168.100.12.nip.io                                              80      2m12s
        ingress.networking.k8s.io/phpmyadmin-ingress   <none>   phpmyadmin.app.lan,phpmyadmin.app.lan.192.168.100.11.nip.io,phpmyadmin.app.lan.192.168.100.12.nip.io             80      3m25s

* **Uso de recursos:**

      kubectl -n cliente1-app describe ns cliente1-app

        Name:         cliente1-app
        Labels:       app=wordpress
                    customer=cliente1
                    kubernetes.io/metadata.name=cliente1-app
        Annotations:  customerEmail: daniel@app.lan
                    customerName: Daniel
        Status:       Active

        Resource Quotas
        Name:                   cliente1-quota
        Resource                Used    Hard
        --------                ---     ---
        configmaps              2       18
        limits.cpu              1500m   1500m
        limits.memory           1500Mi  2Gi
        persistentvolumeclaims  2       4
        pods                    3       6
        replicationcontrollers  0       2
        requests.cpu            800m    1
        requests.memory         950Mi   1Gi
        requests.storage        1524Mi  2Gi
        secrets                 2       18
        services                3       6

        Resource Limits
        Type                   Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
        ----                   --------  ---    ---    ---------------  -------------  -----------------------
        Container              cpu       100m   500m   100m             500m           -
        Container              memory    150Mi  500Mi  150Mi            500Mi          -
        PersistentVolumeClaim  storage   100Mi  2Gi    100Mi            2Gi            -

* **Métricas sobre el namespace:**

      kubectl top pods -n cliente1-app --use-protocol-buffers

        NAME                                     CPU(cores)   MEMORY(bytes)
        apache-deployment-6f75686b75-mm6rf       42m          72Mi
        mysql-deployment-596f8bcdcc-jvnt4        39m          369Mi
        phpmyadmin-deployment-545cf8fdc8-2gr2d   1m           11Mi

* **Métricas de los nodos:**

      kubectl top nodes --use-protocol-buffers

        NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
        master   325m         16%    1247Mi          66%
        node1    97m          9%     788Mi           41%
        node2    112m         11%    1147Mi          60%


* **Uso de los volúmenes NFS:**

      du -hs /srv/lv_vol0*

        58M	/srv/lv_vol01
        203M	/srv/lv_vol02

