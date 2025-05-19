
# myapp-helm-chart/
  ## ├── charts/
  ## ├── values.yaml
  ## ├── templates/
  ## ├── Chart.yaml
charts/: subcharts de Helm si los tienes.

values.yaml: archivo de valores parametrizados.

templates/: plantillas de Kubernetes, como deployment.yaml, service.yaml, etc.

Chart.yaml: archivo que define el chart de Helm.
# Values.yaml

* **`vault`**: Configura la integración con HashiCorp Vault para gestionar secretos. Define si está habilitado, la dirección del servidor de Vault, y cómo obtener el token de autenticación (desde un Secret de Kubernetes). También especifica la ruta dentro de Vault donde se encuentran los secretos de la aplicación.

* **`job`**: Configura un Job de Kubernetes que se encarga de obtener secretos de Vault y almacenarlos en un volumen temporal. Define si el Job está habilitado, su nombre, la imagen de Docker a usar, el comando para obtener los secretos, la política de reinicio y cómo se montan los volúmenes para guardar los secretos obtenidos.

* **`app`**: Configura el Deployment principal de la aplicación. Define el número de réplicas, la imagen de Docker a utilizar (repositorio, etiqueta y política de obtención), y cómo se expone la aplicación dentro del clúster a través de un Service.

* **`resources`**: Define los límites y las solicitudes de recursos (CPU y memoria) para los contenedores de la aplicación, lo cual es importante para la gestión de recursos en Kubernetes.

* **`probes`**: Configura las sondas de `liveness` y `readiness`. Estas sondas permiten a Kubernetes saber si un contenedor está vivo y listo para recibir tráfico, ayudando a mantener la disponibilidad de la aplicación.

* **`hpa`**: Configura el Horizontal Pod Autoscaler (HPA). Si está habilitado, permite que Kubernetes escale automáticamente el número de réplicas de la aplicación basándose en el uso de CPU para manejar la carga.

* **`pvc`**: Configura un PersistentVolumeClaim (PVC). Si está habilitado, solicita almacenamiento persistente para la aplicación, definiendo el nombre del PVC, los modos de acceso y el tamaño del almacenamiento solicitado.

* **`ingress`**: Configura un Ingress. Si está habilitado, permite exponer la aplicación fuera del clúster utilizando un nombre de host específico y puede configurar TLS (HTTPS) para comunicaciones seguras.

* **`k8sSecret`**: Define un Secret de Kubernetes para almacenar secretos directamente en Kubernetes. Especifica el nombre del Secret y los datos a almacenar (los valores deben estar codificados en Base64).

* **`vaultAuth`**: Configura la autenticación automática con Vault utilizando el método de autenticación de Kubernetes. Define si está habilitada, el rol de Vault a utilizar, el ServiceAccount asociado, la ruta del método de autenticación en Vault y cómo obtener el token necesario para la autenticación.

En resumen, este código define la configuración necesaria para desplegar una aplicación en Kubernetes, incluyendo la gestión de secretos con Vault, la obtención de secretos mediante un Job, la configuración del Deployment y el Service de la aplicación, la asignación de recursos, la configuración de sondas, el escalado automático con HPA, el almacenamiento persistente con PVC, la exposición externa con Ingress, la gestión de Secrets de Kubernetes y la autenticación con Vault usando Kubernetes Auth.

# deploiment.yaml

## Condición de habilitación:
{{- if .Values.app.enabled }}: Este Deployment solo se creará si el valor app.enabled en tus archivos de configuración (values.yaml en Helm) es verdadero.
## Tipo de recurso:
## apiVersion: apps/v1, kind:
Deployment: Define que este código crea un objeto de tipo Deployment en Kubernetes.
## Metadatos:
metadata: Define el nombre ({{ .Release.Name }}-myapp) y etiquetas (app: {{ .Release.Name }}) para el Deployment. {{ .Release.Name }} es una variable que se reemplazará por el nombre del lanzamiento de Helm.
## Especificación del Deployment:
spec: Configura cómo debe funcionar el Deployment.
## replicas:
{{ .Values.app.replicaCount }}: Indica cuántas copias (pods) de la aplicación deben ejecutarse, según el valor app.replicaCount.
## selector:
## matchLabels:
app: {{ .Release.Name }}-myapp: Define qué pods son gestionados por este Deployment, basándose en la etiqueta app.
## template:
Define la plantilla para los pods que se crearán.
## metadata: labels: 
app: {{ .Release.Name }}-myapp: Etiquetas para los pods.
## spec:
Especificación del pod.
## containers:
Define los contenedores dentro del pod.
- name: myapp: Nombre del contenedor.
## image:
"{{ .Values.app.image.repository }}:{{ .Values.app.image.tag }}": Especifica la imagen del contenedor a usar, obteniendo el repositorio y la etiqueta de tus valores de configuración.
## imagePullPolicy:
{{ .Values.app.image.pullPolicy }}: Define cuándo se debe descargar la imagen (por ejemplo, IfNotPresent, Always).
## ports:
- containerPort: {{ .Values.app.service.port }}: Expone el puerto containerPort del contenedor, según el valor app.service.port.
## env:
Define las variables de entorno para el contenedor.
{{- range .Values.env }}: Itera sobre una lista de variables de entorno definidas en values.yaml.
- name: {{ .name }}, value: {{ .value | quote }}: Define variables de entorno con nombre y valor de la lista. | quote asegura que el valor se formatea correctamente.
## VAULT_TOKEN, VAULT_ADDR:
Define variables de entorno específicas para interactuar con HashiCorp Vault, obteniendo el token de un Secret de Kubernetes (secretKeyRef) y la dirección de tus valores de configuración.
## livenessProbe:
-readinessProbe::Configuran los sondeos (probes) de liveness y readiness si están habilitados ({{- if .Values.probes.liveness.enabled }} y {{- if .Values.probes.readiness.enabled }}). Estos comprueban la salud de la aplicación.
-httpGet:: Especifica un sondeo HTTP a una ruta (path) y puerto (port) definidos.
-initialDelaySeconds:, periodSeconds:, failureThreshold:: Configuran el comportamiento de los sondeos (cuándo empezar, con qué frecuencia comprobar, cuántos fallos antes de considerar el pod no saludable).
## resources:
Define los límites y solicitudes de CPU y memoria para el contenedor, según los valores de configuración.
## nodeSelector:
{{ .Values.nodeSelector }}: Asigna los pods a nodos específicos con las etiquetas definidas en nodeSelector.

# ConfigMap.yaml

este código define un ConfigMap en Kubernetes.

## apiVersion: 
### v1:
Indica la versión de la API de Kubernetes que se está utilizando.
## kind: ConfigMap: 
Especifica que este recurso es un ConfigMap. Los ConfigMaps se usan para almacenar datos de configuración no confidenciales como pares clave-valor.
## metadata::
Contiene metadatos para el ConfigMap.
## name:
-config: Establece el nombre del ConfigMap.  es una plantilla común en Helm que inserta el nombre del lanzamiento actual. Así, el nombre del ConfigMap será el nombre de tu lanzamiento de Helm seguido de -config.
## data:
Contiene los datos de configuración que se almacenarán en el ConfigMap.
## example.property:
### value:
Es un par clave-valor de ejemplo. example.property es la clave y value es el valor asociado. Puedes añadir múltiples pares clave-valor aquí para almacenar diferentes configuraciones.
En resumen, este código crea un ConfigMap llamado dinámicamente según el nombre de tu lanzamiento de Helm, y almacena una única configuración con la clave example.property y el valor value. Este ConfigMap puede ser luego montado o referenciado po

# hpa.yaml

Este código es una plantilla de Kubernetes para crear un Horizontal Pod Autoscaler (HPA).

{{- if .Values.hpa.enabled }} y {{- end }}: Esto es una condición. El HPA solo se creará si la variable hpa.enabled en tus valores de configuración (generalmente en un archivo values.yaml) está establecida a true.
apiVersion: autoscaling/v2: Especifica la versión de la API de Kubernetes que se está utilizando para el HPA.
kind: HorizontalPodAutoscaler: Indica que estamos creando un recurso HPA.
metadata:: Contiene información sobre el recurso HPA.
name: {{ .Release.Name }}-myapp-hpa: Define el nombre del HPA. Utiliza el nombre de la release de Helm y añade -myapp-hpa.
namespace: {{ .Release.Namespace }}: Especifica el namespace donde se creará el HPA. Utiliza el namespace de la release de Helm.
spec:: Define la configuración del HPA.
scaleTargetRef:: Indica qué recurso debe escalar el HPA.
apiVersion: apps/v1: La versión de la API del recurso objetivo.
kind: Deployment: El tipo de recurso objetivo (en este caso, un Deployment).
name: {{ .Release.Name }}-myapp: El nombre del recurso objetivo a escalar. Utiliza el nombre de la release de Helm y añade -myapp.
minReplicas: {{ .Values.hpa.minReplicas }}: El número mínimo de réplicas (pods) que el HPA mantendrá. Este valor se obtiene de los valores de configuración.
maxReplicas: {{ .Values.hpa.maxReplicas }}: El número máximo de réplicas (pods) al que el HPA puede escalar. Este valor se obtiene de los valores de configuración.
targetCPUUtilizationPercentage: {{ .Values.hpa.targetCPUUtilizationPercentage }}: El porcentaje de utilización de CPU objetivo para escalar. Cuando la utilización media de CPU de los pods excede este porcentaje, el HPA aumenta el número de réplicas. Este valor se obtiene de los valores de configuración.
En resumen, este código crea un HPA que automáticamente ajustará el número de pods de un Deployment llamado [nombre-de-la-release]-myapp basándose en la utilización de CPU, siempre y cuando el HPA esté habilitado en los valores de configuración. El número de pods se mantendrá entre minReplicas y maxReplicas.



# job.yaml

define un Job de Kubernetes que se ejecuta una vez.

## Condición:
El Job solo se creará si la variable job.enabled en tus valores de configuración está establecida a true.
## Job:
Se crea un Job llamado  en el espacio de nombres .
## Contenedor:
El Job ejecuta un solo contenedor llamado vault.
Utiliza la imagen de Docker especificada en .
## Comando:
Ejecuta un comando shell (sh -c) que:

Obtiene un secreto de HashiCorp Vault usando el comando vault kv get.
Formatea la salida como JSON.
Utiliza jq para extraer solo los datos del secreto.
Redirige esa salida a un archivo llamado myapp.json dentro del contenedor, en la ruta /mnt/secrets.
## Variables de Entorno:
Define dos variables de entorno esenciales para conectarse a Vault:
## VAULT_TOKEN:
Obtiene el token de un secreto de Kubernetes llamado  y utiliza la clave  dentro de ese secreto.
## VAULT_ADDR: 
Establece la dirección de Vault a partir del valor .
## Montaje de Volumen:
Monta un volumen llamado secret-volume en la ruta /mnt/secrets dentro del contenedor.
## Volumen:
Define el volumen secret-volume como un emptyDir, lo que significa que es un volumen vacío que existe mientras el Pod del Job esté en ejecución.
## Política de Reinicio:
La política de reinicio del Pod del Job se establece según el valor de . Si no se especifica, el valor predeterminado es Never (nunca reiniciar si falla).

En resumen, este Job tiene como objetivo conectarse a Vault, descargar un secreto específico y guardarlo en un archivo dentro de un volumen temporal en el Pod que se ejecuta.
