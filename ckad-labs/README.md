# Certificación CKAD

A continuación se indican los objetivos con sus laboratorios asociados

* Enlace al certificado: https://traininglinuxfoundationorg/certification/certified-kubernetes-application-developer-ckad/
* Versión de Kubernetes: v1.31
* Número de nodos mínimo para los labs: 2
* Entorno usado: [Minikube](https://minikube.sigs.k8s.io/docs/) en Ubuntu 20.04
   * Comandos ejecutados para crear el entorno:

      ```sh
      minikube start --driver=docker --memory=4g --cpu=3 --disk-size=10g --cni=calico
      minikube addons enable ingress
      ```

## Application Design and Build

**Define, build and modify container images**

1. Crea un pod con los siguientes requisitos:

   * **Nombre:** `webserver`
   * **Etiquetas:** `type=webserver`, `software=nginx`
   * **Imagen:** `nginx:latest`
   * **Política de restart:** `OnFailure`
   * **Variables de entorno:** `CERT=CKAD`, `YEAR=2024`

2. Actualiza la imagen del pod para use `httpd:latest` sin eliminar el contenedor.

3. Modifica la etiqueta `software=nginx` por `software=httpd`

4. Añade una etiqueta adicional al pod; `env=testing`.

5. Añade la anotación; `description='Simple web server'`

6. Ejecuta el comando `curl -s http://localhost` en el pod sin acceder a él.

7. Elimina el pod a través de la etiqueta `type=webserver`.

8. Crea un pod con los siguientes requisitos:

   * **Etiqueta:** `type=webserver`
   * **Nombre:** `webserver`
      * **Imagen:** `nginx:latest`
   * **Nombre:** `cli`
      * **Imagen:** `alpine/curl`
      * **Comando:** `tail -f /dev/null`

9. Ejecuta una consulta HTTP al contenedor `webserver` desde el contenedor `cli` mediante `curl`.

10. Actualiza la imagen del contenedor `webserver` por `httpd:latest` y verifica su contenedor ejecutando un curl desde el contenedor `cli`.

**Understand multi-container Pod design patterns (eg sidecar, init and others)**

**Choose and use the right workload resource (Deployment, DaemonSet, CronJob, etc)**

* Vídeo explicativo: https://www.youtube.com/watch?v=4MEgCP7h8UU
* Vídeo explicativo: https://www.youtube.com/watch?v=30KAInyvY_o

**Utilize persistent and ephemeral volumes**

1. Crea un volumen persistente con los siguientes requisitos en el nodo principal:

   * **Nombre:** `pv-hostpath`
   * **Tipo:** `hostPath`
   * **Capacidad:** `5M`
   * **Modo de acceso:** `ReadOnlyMany`
   * **Etiqueta:** `env=testing`
   * **Archivo dentro del volumen:** `index.html`
   * **Contenido del archivo:** `<h1>Versión 1.0.0 del Volumen Persistente de tipo hostPath</h1>`

2. Crea un `Persistent Volume Claim` llamado `pvc-hostpath` para el volumen `pv-hostpath`.

3. Crea un deployment con los siguientes requisitos:

   * **Nombre:** `deployment-storage-hostpath`
   * **Réplicas:** 3
   * **Imagens:** `httpd:latest`
   * **Nombre del persistent volume claim:** `pvc-hostpath`
   * **Nombre del volumen:** `volumen-website`
   * **Punto de montaje del volumen persistente:** `/usr/local/apache2/htdocs/`
   * **Nodo:** `principal`

## Application Deployment

**Understand Deployments and how to perform rolling updates**

* Vídeo explicativo: https://www.youtube.com/watch?v=6IPu3WU_M0o

1. Crea un deployment con los siguientes requisitos:

   * **Nombre:** `deployment-website`
   * **Réplicas:** 3
   * **Puerto a exponer:** 8080
   * **Imagens:** `learnk8s/app:1.0.0`

2. Añade una anotación al deployment para que pueda restaurarse una revisión con más información.

3. Añade una etiqueta adicional al deployment llamada `env=staging`.

4. Actualiza la imagen del deployment por `learnk8s/app:2.0.0` y añade otra anotación.

5. Restaura la primera revisión del deployment.

6. Reinicia el deployment.

7. Añade un contenedor adicional al deployment con los siguientes requisitos:

   * **Nombre:** `cli`
   * **Imagen:** `alpine/curl`
   * **Comando:** `tail -f /dev/null`

8. Actualiza las imágenes del deployment por: `learnk8s/app:3.0.0` y `alpine/curl:8.9.1` sin usar `kubectl apply`.

9. Elimina el contener `cli` del deployment.

**Use Kubernetes primitives to implement common deployment strategies (eg blue/green or canary)**

* Vídeo explicativo: https://www.youtube.com/watch?v=lxc4EXZOOvE&feature=youtu.be
* Vídeo explicativo: https://www.youtube.com/watch?v=fWe6k4MmeSg

**Use the Helm package manager to deploy existing packages**
**Kustomize**

* Vídeo explicativo: https://www.youtube.com/watch?v=LWbbL3jZcgo

## Application Observability and Maintenance

**Understand API deprecations**

**Implement probes and health checks**

* Vídeo explicativo: https://www.youtube.com/watch?v=fqfieWP1jY4

1. Crea un pod con un probe de tipo `liveness` con los siguientes requisitos:

   * **Nombre:** `website`
   * **Imagen:** `nginx:latest`
   * **Tipo de liveness:** `HTTP`
   * **Ruta:** `/`
   * **Puerto:** `8080`
   * **Tiempo antes del primer check:** `15`
   * **Timeout:** `10`
   * **Periocidad del los checks:** `15`
   * **Umbral de fallos:** `3`
   * **Política de restart:** `Always`

2. Crea un pod con un probe de tipo `readiness` los siguientes requisitos:

   * **Nombre:** `db-01`
   * **Imagen:** `mysql:8.0`
   * **Variable de entorno:** `MYSQL_ROOT_PASSWORD=ckad-2024`
   * **Tipo de liveness:** `Exec`
   * **Comando:** `mysqladmin ping`
   * **Tiempo antes del primer check:** `30`
   * **Timeout:** `10`
   * **Periocidad del los checks:** `15`
   * **Umbral de fallos:** `3`
   * **Política de restart:** `Always`

3. Crea un pod con un probe de tipo `startup` los siguientes requisitos:

   * **Nombre:** `db-02`
   * **Imagen:** `mysql:8.0`
   * **Variable de entorno:** `MYSQL_ROOT_PASSWORD=ckad-2024`
   * **Tipo de liveness:** `TCP`
   * **Socket:** `3306`
   * **Tiempo antes del primer check:** `30`
   * **Periocidad del los checks:** `10`
   * **Política de restart:** `Always`

**Use built-in CLI tools to monitor Kubernetes applications**

**Utilize container logs**

**Debugging in Kubernetes**

* Vídeo explicativo: https://www.youtube.com/watch?v=hixs2GIGrUw

## Application Environment, Configuration and Security

**Discover and use resources that extend Kubernetes (CRD, Operators)**

**Understand authentication, authorization and admission control**

* Vídeo explicativo: https://www.youtube.com/watch?v=iE9Qb8dHqWI

**Understand requests, limits, quotas**

**Understand ConfigMaps**

1. Crea un configMap llamado `nginx-http-vhost`, el cual debe tener el siguiente contenido:

   ```text
   server {
    listen       8080;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
   }
   ```

2. Crea un configMap llamado `nginx-http-port`, cuyo valor sea: `port=8080`.

3. Crea un deployment con las siguientes requisitos:

   * **Nombre:** `deployment-simple-site`
   * **Imagen:** `nginx:latest`
   * **Réplicas:** `2`
   * **Puerto a exponer:** `8080`
   * **configMaps:**
      * **Nombre:**  `nginx-http-vhost`
         * **Tipo de montaje:** `volumen`
         * **Punto de montaje:** `/etc/nginx/conf.d/default.conf`
      * **Nombre:**  `nginx-http-port`
         * **Tipo de montaje:** `variable`
         * **Nombre de la variable:** `HTTP_PORT`

4. Actualiza el valor de `8080` a `80` en el configMap `nginx-http-port` y confirma con `curl` el cambio.

**Define resource requirements**

**Create & consume Secrets**

1. Crea un secret llamado `secret-db-test-conf` que almacene el contenido de un archivo llamado `.env`, cuyo contenido es:

   ```text
   APP_ROOT_PWD=R00!,2024;
   APP_DBA_USERNAME=dba_app
   APP_DBA_PWD=Db@,2024;
   ```

2. Crea otro secret llamado `secret-db-test-name` que almacene: `APP_DB_NAME=dev_application`.

3. Crea un pod llamado `db-test` cuya imagen sea `mysql:8.0` que exponga el puerto `3306` y que use las siguientes variables obteniendo su valor del secret `secret-db-test-conf` y de `secret-db-test-name`.

   * MYSQL_ROOT_PASSWORD
   * MYSQL_USER
   * MYSQL_PASSWORD
   * MYSQL_DATABASE

4. Crea otro pod llamado `cli-mysql` cuya imagen sea `mysql:8.0`, monta los secrets: `secret-db-test-conf` y `secret-db-test-name` como volúmenes de sólo lectura, conéctate a la base de datos del pod `db-test` usando los secrets montados y crea la base de datos usando los secrets montados e inserta las siguientes sentencias SQL usando los secrets montados.

   ```sql
   CREATE TABLE cursos (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50));
   INSERT INTO cursos (name) VALUES ('CKAD');
   ```

5. Modifica el valor del secret `secret-db-test-name` por `prod_application` y verifica su valor en ambos pods.

**Understand ServiceAccounts**

1. Crea un serviceAccount llamado `sa-report` para el namespace `lab-sa`.

2. Crea un rol llamado `rol-sa-report` que pueda hacer un `get` y `list` sobre los objetos `pods` del namespace `lab-sa`. Después, aplícalo al serviceAccount `sa-report`.

3. Crea un pod llamado `pod-report` cuya imagen sea `alpine/curl`, que pertenezca al namespace `lab-sa` y tenga el serviceAccount `sa-report`.

4. Haz una consulta al API server solicitando los pods en ejecución en el namespace `lab-sa` y almacena el resultado en un archivo fuera del pod llamado `report.json`.

**Understand Application Security (SecurityContexts, Capabilities, etc)**

## Services and Networking

**Demonstrate basic understanding of NetworkPolicies**

**Provide and troubleshoot access to applications via services**

* Vídeo explicativo: https://www.youtube.com/watch?v=RQbc_Yjb9ls

1. Crea un deployment llamado `deployment-http-site`, cuya imagen sea `learnk8s/app:1.0.0`, tenga 3 réplicas y exponga por el puerto `8080`. Adicionalmente, crea un pod llamado `http-cli` cuya imagen sea `alpine/curl` y el comando sea: `tail -f /dev/null`.

2. Crea un servicio llamado `service-http-site` de tipo `ClusterIP` para el deployment `deployment-http-site` que escuche por el puerto `80`.

3. Desde el pod `http-cli` haz una consulta HTTP al deployment `deployment-http-site` a través del servicio `service-http-site`.

**Use Ingress rules to expose applications**

* Vídeo explicativo: https://www.youtube.com/watch?v=1BksUVJ1f5M
