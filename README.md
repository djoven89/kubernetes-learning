# Kubernetes learning

El objetivo de este repositorio es de aprendizaje propio sobre Kubernetes. Aquí subiré entornos de prueba sobre distintos objetos, conceptos o laboratorios en Kubernetes.

## Laboratorio sobre Wordpress

En el directorio `wordpress` se ha creado un laboratorio completo cuya finalidad es aprender las bases de los objetos más comúnes de Kubernetes para el despliegue de una aplicación como Wordpress para un entorno.

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

Más información en: `wordpress/README.md`.
