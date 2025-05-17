
myapp-helm-chart/
  ├── charts/
  ├── values.yaml
  ├── templates/
  ├── Chart.yaml
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

Este código es una plantilla de **Helm** para desplegar una aplicación en **Kubernetes**.

-   Comienza con una condición (``) que indica que el resto del código solo se procesará si la variable `app.enabled` en el archivo `values.yaml` es `true`.
-   Define un objeto de Kubernetes de tipo **Deployment** (despliegue).
-   Le asigna un nombre al despliegue usando el nombre de la versión de Helm (`.Release.Name`) y un sufijo (`-myapp`).
-   Establece etiquetas para identificar el despliegue.
-   Configura el número de réplicas (pods) que se ejecutarán, obteniéndolo de `.Values.app.replicaCount`.
-   Define un selector que permite al despliegue encontrar los pods que gestiona, basándose en la etiqueta `app`.
-   Configura la plantilla para los pods:
    -   Les asigna etiquetas para que coincidan con el selector del despliegue.
    -   Define el contenedor principal (`myapp`) dentro del pod:
        -   Especifica la imagen del contenedor, su repositorio y etiqueta, obteniéndolos de `.Values.app.image`.
        -   Define la política de extracción de la imagen (`imagePullPolicy`).
        -   Expone el puerto de la aplicación, tomado de `.Values.app.service.port`.
        -   Configura variables de entorno:
            -   Itera sobre una lista de variables de entorno definidas en `.Values.env`.
            -   Agrega variables de entorno específicas para `VAULT_TOKEN` (obtenida de un Secret de Kubernetes) y `VAULT_ADDR`.
        -   Opcionalmente, configura los sondeos de liveness (para verificar si la aplicación sigue viva) y readiness (para verificar si la aplicación está lista para recibir tráfico), si están habilitados en `.Values.probes`.
        -   Define los límites y solicitudes de recursos (CPU y memoria) para el contenedor.
    -   Permite configurar `nodeSelector`, `tolerations` y `affinity` para controlar dónde se despliegan los pods, obteniendo sus valores de `.Values`.
    -   Establece la política de reinicio de los pods a `Always`.
-   El código termina con la condición de cierre (``).

En resumen, esta plantilla crea un despliegue de Kubernetes para una aplicación, configurando aspectos como el número de réplicas, la imagen del contenedor, variables de entorno, sondeos de salud, recursos y políticas de despliegue, basándose en los valores proporcionados en el archivo `values.yaml` de Helm. Solo se creará el despliegue si `app.enabled` es `true`.

# ConfigMap.yaml

ste código define un ConfigMap en Kubernetes.

apiVersion: v1: Indica la versión de la API de Kubernetes que se está utilizando.
kind: ConfigMap: Especifica que este recurso es un ConfigMap. Los ConfigMaps se usan para almacenar datos de configuración no confidenciales como pares clave-valor.
metadata:: Contiene metadatos para el ConfigMap.
name: -config: Establece el nombre del ConfigMap.  es una plantilla común en Helm que inserta el nombre del lanzamiento actual. Así, el nombre del ConfigMap será el nombre de tu lanzamiento de Helm seguido de -config.
data:: Contiene los datos de configuración que se almacenarán en el ConfigMap.
example.property: value: Es un par clave-valor de ejemplo. example.property es la clave y value es el valor asociado. Puedes añadir múltiples pares clave-valor aquí para almacenar diferentes configuraciones.
En resumen, este código crea un ConfigMap llamado dinámicamente según el nombre de tu lanzamiento de Helm, y almacena una única configuración con la clave example.property y el valor value. Este ConfigMap puede ser luego montado o referenciado po

# hpa.yaml

ste código es una plantilla para crear un Horizontal Pod Autoscaler (HPA) en Kubernetes utilizando Helm.

: Esto es una condición de Helm. Significa que el resto del código dentro de este bloque solo se incluirá si la variable hpa.enabled en tu archivo values.yaml de Helm está establecida a true.
apiVersion: autoscaling/v2 y kind: HorizontalPodAutoscaler: Definen el tipo de recurso de Kubernetes que se está creando, que es un HPA.
metadata:: Contiene información sobre el HPA.
name: -myapp-hpa: Establece el nombre del HPA, usando el nombre de la release de Helm y añadiendo -myapp-hpa.
namespace: : Establece el namespace donde se creará el HPA, usando el namespace de la release de Helm.
spec:: Define la configuración del HPA.
scaleTargetRef:: Especifica el recurso de Kubernetes que el HPA debe escalar.
apiVersion: apps/v1 y kind: Deployment: Indica que el recurso a escalar es un Deployment.
name: -myapp: Especifica el nombre del Deployment a escalar, usando el nombre de la release de Helm y añadiendo -myapp.
minReplicas: : Define el número mínimo de pods que el HPA mantendrá. Este valor se toma de la variable hpa.minReplicas en tu archivo values.yaml.
maxReplicas: : Define el número máximo de pods que el HPA puede escalar. Este valor se toma de la variable hpa.maxReplicas en tu archivo values.yaml.
targetCPUUtilizationPercentage: : Establece el porcentaje de uso de CPU objetivo para los pods. Si el uso promedio de CPU supera este porcentaje, el HPA aumentará el número de pods (hasta maxReplicas). Este valor se toma de la variable hpa.targetCPUUtilizationPercentage en tu archivo values.yaml.
: Cierra el bloque condicional de Helm.
En resumen, esta plantilla crea un HPA que automáticamente ajusta el número de pods de un Deployment llamado [nombre-de-la-release]-myapp en función de la utilización de la CPU, manteniendo el número de pods entre un mínimo y un máximo definidos en tu archivo values.yaml. Solo se creará si la opción HPA está habilitada en tu configuración de Helm.

job.yaml

define un Job de Kubernetes que se ejecuta una vez.

Condición: El Job solo se creará si la variable job.enabled en tus valores de configuración está establecida a true.
Job: Se crea un Job llamado  en el espacio de nombres .
Contenedor: El Job ejecuta un solo contenedor llamado vault.

Utiliza la imagen de Docker especificada en .
Comando: Ejecuta un comando shell (sh -c) que:

Obtiene un secreto de HashiCorp Vault usando el comando vault kv get.
Formatea la salida como JSON.
Utiliza jq para extraer solo los datos del secreto.
Redirige esa salida a un archivo llamado myapp.json dentro del contenedor, en la ruta /mnt/secrets.
Variables de Entorno: Define dos variables de entorno esenciales para conectarse a Vault:

VAULT_TOKEN: Obtiene el token de un secreto de Kubernetes llamado  y utiliza la clave  dentro de ese secreto.
VAULT_ADDR: Establece la dirección de Vault a partir del valor .
Montaje de Volumen: Monta un volumen llamado secret-volume en la ruta /mnt/secrets dentro del contenedor.
Volumen: Define el volumen secret-volume como un emptyDir, lo que significa que es un volumen vacío que existe mientras el Pod del Job esté en ejecución.
Política de Reinicio: La política de reinicio del Pod del Job se establece según el valor de . Si no se especifica, el valor predeterminado es Never (nunca reiniciar si falla).

En resumen, este Job tiene como objetivo conectarse a Vault, descargar un secreto específico y guardarlo en un archivo dentro de un volumen temporal en el Pod que se ejecuta.
