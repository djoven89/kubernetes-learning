# Certificación CKAD

A continuación se indican los objetivos con sus laboratorios asociados

* Enlace al certificado: https://traininglinuxfoundationorg/certification/certified-kubernetes-application-developer-ckad/
* Versión de Kubernetes: v1.31
* Número de nodos mínimo para los labs: 2

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

1. Init Containers
2. Sidecar Containers
3. Ephemeral Containers

**Choose and use the right workload resource (Deployment, DaemonSet, CronJob, etc)**

1. Deployment
2. DaemonSet
3. CronJob
4. Jobs
5. StatefulSets

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
**Use the Helm package manager to deploy existing packages**
**Kustomize**

## Application Observability and Maintenance

**Understand API deprecations**

**Implement probes and health checks**

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

## Application Environment, Configuration and Security

**Discover and use resources that extend Kubernetes (CRD, Operators)**
**Understand authentication, authorization and admission control**
**Understand requests, limits, quotas**

1. Requests and limits
2. LimitRange
3. ResourceQuota

**Understand ConfigMaps**
**Define resource requirements**
**Create & consume Secrets**
**Understand ServiceAccounts**
**Understand Application Security (SecurityContexts, Capabilities, etc)**

## Services and Networking

* **Demonstrate basic understanding of NetworkPolicies**
* **Provide and troubleshoot access to applications via services**
* **Use Ingress rules to expose applications**
