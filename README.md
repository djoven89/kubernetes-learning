# Kubernetes learning

El objetivo de este repositorio es de aprendizaje propio sobre Kubernetes. Aquí subiré entornos de prueba sobre distintos objetos, conceptos o laboratorios en Kubernetes.

## Laboratorio 1 - Wordpress

En el directorio `lab1-wordpress` se ha creado un laboratorio completo cuya finalidad es aprender las bases de los objetos más comúnes de Kubernetes para el despliegue de una aplicación como Wordpress para un entorno.

Se han usado los siguientes objetos:

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

Además, se ha configurado el cluster con:

* Traefik como Ingress Controller.
* NFS para los PersistentVolumes.
* Metrics server para una monitorización sencilla.

Más información en: `lab1-wordpress/README.md`.

## Laboratorio 2 - Actualizaciones

En el directorio `lab2-updates` se ha creado un laboratorio completo cuya finalidad es aprender las tres metodologías de actualización más comúnes usadas en Kubernetes para actualizar aplicaciones.

Se han usado los siguientes objetos:

* Namespace.
* Service.
* Deployment.

3 entornos distintos, cada uno tiene un tipo de política de actualización.

* (dev) Rolling update.
* (staging) Canary deployment.
* (prod) Blue gree deployment.

Más información en: `lab2-updates/README.md`.

## CKAD

En el directorio `ckad-labs/README.md` se han creados laboratorios para cada uno de los objetivos que conforman la certificación.
