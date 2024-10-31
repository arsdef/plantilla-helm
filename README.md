# plantilla-helm

## Estructura 
# mychart/
## ├── charts/
## ├── templates/
### │   ├── deployment.yaml
### │   ├── service.yaml
### │   ├── ingress.yaml
### │   ├── configmap.yaml
### │   ├── secret.yaml
### |   ├── hpa.yaml
### │   ├── _helpers.tpl
### │   └── NOTES.txt
## ├── Chart.yaml
## └── values.yaml


representa la estructura de directorios de un gráfico (chart) de Helm, que es una herramienta de gestión de paquetes para Kubernetes. A continuación, te explico cada componente de la estructura:

mychart/: Es el directorio principal que contiene todos los archivos y subdirectorios del gráfico.

## #charts/:
Este directorio puede contener otros gráficos de Helm que se usan como dependencias del gráfico principal.

## templates/: 

Aquí se encuentran las plantillas de recursos de Kubernetes que Helm utilizará para generar los archivos YAML finales. Cada archivo en este directorio define un recurso que se despliega en el clúster de Kubernetes.

## deployment.yaml: 
Define un despliegue de Kubernetes, que especifica cómo se deben crear y administrar los pods.
A continuación se presenta una explicación clara y concisa de cada parte:

apiVersion: apps/v1: Especifica la versión de la API de Kubernetes para el recurso Deployment.

kind: Deployment: Define el tipo de recurso, que en este caso es un Deployment, que se utiliza para gestionar un conjunto de réplicas de una aplicación.

metadata: Contiene información básica sobre el Deployment, como su nombre y etiquetas. El nombre se genera a partir del nombre del release y se asocia a una etiqueta que se usa para seleccionar recursos.

spec: Describe la especificación del Deployment:

replicas: El número de instancias (réplicas) de la aplicación que se desean ejecutar, tomado de los valores de configuración.
selector: Define cómo Kubernetes debe encontrar las Pods que pertenecen a este Deployment mediante la coincidencia de etiquetas.
template: Define la plantilla de Pod que se utilizará:
metadata: Proporciona el mismo conjunto de etiquetas que se usa en el selector.
spec: Describe la configuración de los contenedores que se ejecutarán en el Pod:
containers: Lista los contenedores que se ejecutan en el Pod. Para cada contenedor se define:
name: El nombre del contenedor.
image: La imagen del contenedor, incluyendo el repositorio y la etiqueta.
imagePullPolicy: Define cuándo se debe descargar la imagen.
ports: Especifica los puertos que los contenedores exponen.
env: Define variables de entorno para los contenedores, iterando sobre una lista de variables desde la configuración.
probes: Incluye definiciones de sondas de prueba para liveness (vida) y readiness (listo) si están habilitadas en los valores de configuración. Estas sondas ayudan a Kubernetes a determinar si el contenedor está funcionando correctamente o listo para recibir tráfico.
resources: Especifica los límites y solicitudes de recursos (CPU y memoria) para el contenedor.
nodeSelector, tolerations, affinity: Estas configuraciones permiten especificar en qué nodos del clúster se deben ejecutar los Pods, considerando tolerancias y afinidades para cumplir con los requisitos de colocación.
Este manifiesto se usa comúnmente junto con Helm, ya que contiene plantillas (indicado por {{ ... }}) que se reemplazan por valores específicos al momento de realizar el despliegue.
## service.yaml:
Define un servicio de Kubernetes, que expone los pods a la red y permite la comunicación interna/externa.
explicación de cada parte:

apiVersion: v1: Esta línea especifica la versión de la API de Kubernetes que se está utilizando. En este caso, se utiliza la versión 1.

kind: Service: Indica que el recurso que se está definiendo es un servicio de Kubernetes. Los servicios permiten exponer aplicaciones en el clúster.

metadata: Esta sección contiene información sobre el servicio que se está creando:

name: {{ .Release.Name }}-service: Definimos el nombre del servicio utilizando una plantilla. {{ .Release.Name }} se reemplaza por el nombre de la liberación (release) del chart de Helm.
spec: Esta sección especifica la configuración del servicio:

type: {{ .Values.service.type }}: Define el tipo de servicio, que puede ser ClusterIP, NodePort, LoadBalancer, etc. Se obtiene de los valores del chart de Helm.
ports: Esta sección define los puertos del servicio:
port: {{ .Values.service.port }}: Especifica el puerto a través del cual se accede al servicio, que se establece a través de los valores del chart.
targetPort: 80: Este es el puerto en el contenedor al que se redirigirán las solicitudes que lleguen al port del servicio. En este caso, se redirigen al puerto 80 del contenedor.
selector: Esto se utiliza para seleccionar los pods que serán parte de este servicio:

app: {{ .Release.Name }}: Define las etiquetas de los pods que el servicio debe seleccionar. Se utiliza también una plantilla para que coincida con el nombre de la liberación.

## ingress.yaml: 
Configura un recurso de Ingress, que gestiona el acceso externo a los servicios en el clúster, generalmente proporcionando balanceo de carga y enrutamiento.
Vamos a desglosarlo y explicarlo por partes:
Condicional if:

yaml
{{- if .Values.ingress.enabled }}  
Este bloque comienza comprobando si la opción ingress.enabled está configurada en true en los valores de configuración. Si no está habilitada, todo el bloque que sigue no se ejecutará.

Definición del Ingress:

yaml
apiVersion: networking.k8s.io/v1  
kind: Ingress  
Se establece que se va a crear un recurso de tipo Ingress, utilizando la API de Kubernetes versión v1.

Metadatos:

yaml
metadata:  
  name: {{ .Release.Name }}-ingress  
  annotations:  
    {{- range $key, $value := .Values.ingress.annotations }}  
    {{ $key }}: {{ $value | quote }}  
    {{- end }}  
Aquí se define el metadato name, que se generará a partir del nombre del lanzamiento (.Release.Name) seguido de -ingress. También se agregan anotaciones que se especifican en .Values.ingress.annotations. Se utiliza un bucle range para iterar sobre cada clave y valor de las anotaciones, aplicando quote para asegurarse de que los valores se manejen como cadenas.

Especificaciones del Ingress:

yaml
spec:  
  rules:  
    - host: {{ .Values.ingress.hostname }}  
      http:  
        paths:  
          - path: /  
            pathType: Prefix  
            backend:  
              service:  
                name: {{ .Release.Name }}-service  
                port:  
                  number: {{ .Values.service.port }}  
En esta sección se definen las reglas del Ingress.

Se especifica host, que toma su valor de .Values.ingress.hostname.
Dentro de http, se define un arreglo de paths. Aquí se configura un solo path (/) con pathType especificado como Prefix, lo que significa que se aceptarán todas las rutas que empiecen con /.
Finalmente, en el backend, se indica que las peticiones se dirigirán a un servicio cuyo nombre se genera a partir de .Release.Name seguido de -service, y el puerto se toma de .Values.service.port.
Cierre del bloque:

yaml
{{- end }}  
Esta línea finaliza el bloque condicional if, cerrando la definición del recurso Ingress.


## configmap.yaml: 
Define un ConfigMap, que permite almacenar datos de configuración que pueden ser utilizados por los pods.
A continuación, se explica cada parte:

apiVersion: v1: Indica la versión de la API de Kubernetes que se está utilizando. En este caso, "v1" se refiere a la primera versión estable de la API.

kind: ConfigMap: Especifica el tipo de recurso que se está creando. Aquí se está creando un "ConfigMap", que se utiliza para almacenar datos de configuración en pares clave-valor.

metadata: Esta sección contiene información sobre el ConfigMap:

name: {{ .Release.Name }}-config: El nombre del ConfigMap se genera dinámicamente usando una plantilla (por el sistema de plantillas de Helm). Aquí, {{ .Release.Name }} se reemplaza con el nombre de la liberación, y se añade '-config' al final.
data: Aquí es donde se definen los datos del ConfigMap. En este caso:

example.property: "value": Este es un par clave-valor. La clave es "example.property" y su valor correspondiente es "value". Esta información puede ser utilizada por aplicaciones que se desplieguen en el clúster de Kubernetes para configuraciones personalizadas.

## secret.yaml:
Define un Secret, que se utiliza para almacenar información sensible, como contraseñas o claves API, de manera segura.
 explicación de  cada parte:

apiVersion: v1: Esta línea especifica la versión de la API de Kubernetes que se está utilizando. En este caso, es la versión 1.

kind: Secret: Indica el tipo de recurso que se está definiendo, que en este caso es un Secret. Los secretos se utilizan para almacenar información sensible, como contraseñas, tokens o claves SSH.

metadata: Este bloque contiene información sobre el objeto, como su nombre y etiquetas.

name: {{ .Release.Name }}-secret: La propiedad name define el nombre del secreto. Aquí se utiliza una plantilla (indicada por {{ }}) que extrae el nombre del lanzamiento (release name) y le añade -secret al final.
type: Opaque: Este campo especifica el tipo de secreto. Opaque es el tipo más común y se utiliza para datos genéricos.

data: Este bloque contiene los datos del secreto.

my-secret-key: {{ .Values.secretKey | b64enc }}: Aquí se define una clave my-secret-key que almacenará los datos del secreto. Se utiliza otra plantilla para acceder a un valor del archivo de valores (.Values.secretKey) y se codifica en Base64 (| b64enc) antes de almacenarlo.

## hpa.yaml:
archivo de configuración en formato YAML utilizado para definir un Horizontal Pod Autoscaler (HPA) en un clúster de Kubernetes, gestionado a través de Helm. A continuación se explica cada parte del código:

Condicional: {{- if .Values.hpa.enabled }}

Esta línea inicia un bloque condicional que verifica si la opción de HPA está habilitada en el archivo de valores de Helm (.Values.hpa.enabled). Si está habilitada, se ejecuta el código que sigue.
Versiones y tipo:

apiVersion: autoscaling/v2
Define la versión de la API que se utiliza para el HPA.
kind: HorizontalPodAutoscaler
Especifica que este recurso es un Horizontal Pod Autoscaler.
Metadatos:

metadata:
Proporciona metadatos sobre el recurso.
name: {{ .Release.Name }}-hpa
Asigna un nombre al HPA usando el nombre de la liberación (.Release.Name) concatenado con -hpa.
Especificaciones:

spec:
Define las especificaciones del HPA.
scaleTargetRef:
Indica el objetivo al que se aplicará el escalado.
apiVersion: apps/v1
Define la versión de la API del objetivo.
kind: Deployment
Especifica que el objetivo es un Deployment.
name: {{ .Release.Name }}-app
Define el nombre del Deployment al que se refiere el HPA, usando el nombre de la liberación y agregando -app.
Réplicas:

minReplicas: {{ .Values.hpa.minReplicas }}
Establece el número mínimo de réplicas que se deben mantener.
maxReplicas: {{ .Values.hpa.maxReplicas }}
Establece el número máximo de réplicas que el HPA puede escalar.
Objetivo de utilización de CPU:

targetCPUUtilizationPercentage: {{ .Values.hpa.targetCPUUtilizationPercentage }}
Define el porcentaje objetivo de utilización de CPU para activar el escalado. Si la utilización de CPU promedio de las réplicas supera este valor, el HPA aumentará el número de réplicas.
Fin del bloque condicional: {{- end }}

Esta línea cierra el bloque condicional que comenzó con if.
## _helpers.tpl:
Contiene funciones auxiliares que se pueden usar en otras plantillas para evitar la duplicación de código.
explicacion de  su uso y propósito:
Centralización de código repetitivo : En lugar de repetir código en múltiples plantillas, _helpers.tplpermite definir funciones que pueden reutilizarse. Esto mejora la legibilidad y facilita el mantenimiento.
Uso de plantillas auxiliares : Se pueden definir funciones auxiliares para personalizar etiquetas, nombres y otras variables de configuración. Por ejemplo, defina nombres consistentes para los recursos en Kubernetes basados ​​en los valores de Helm.
Comodidad en gráficos complejos : En gráficos con múltiples plantillas y dependencias, el archivo _helpers.tplayuda a organizar y mantener el código, especialmente en proyectos complejos con muchos valores configurables.

## NOTES.txt: 
Un archivo de texto que puede contener notas informativas sobre el gráfico, que se muestran después de la instalación del gráfico.

## Chart.yaml:
Este archivo contiene metadatos sobre el gráfico, como el nombre, versión, y descripción.
A continuación, se explica cada línea:

apiVersion: v2: Indica la versión de la API de Helm que se está utilizando. En este caso, se está usando la versión 2.

name: mychart: Este es el nombre del chart. "mychart" es un nombre genérico que puedes cambiar para identificar tu chart.

description: Un chart genérico para una aplicación en Kubernetes.: Proporciona una breve descripción del chart. En este caso, se especifica que es un chart genérico destinado a una aplicación en Kubernetes.

version: 0.1.0: Esta es la versión del chart en sí. Se utiliza para el control de versiones del chart y seguir su evolución.

appVersion: "1.0": Indica la versión de la aplicación que se está empaquetando dentro de este chart. En este caso, la aplicación tiene la versión 1.0.

En resumen, este archivo describe un chart de Helm que se utilizará para desplegar una aplicación en Kubernetes, proporcionando información sobre su nombre, descripción y versiones.

## values.yaml:
replicaCount: 2

Define el número de réplicas del pod que se deben crear. En este caso, se crean 2 réplicas.
image:

repository: Especifica el repositorio de la imagen del contenedor, en este caso, myrepo/myapp.
tag: Indica la etiqueta de la imagen, que se establece como "latest".
pullPolicy: Define la política de descarga de la imagen. "IfNotPresent" significa que se descargará la imagen solo si no está presente en el nodo.
service:

type: Tipo de servicio, "ClusterIP" significa que el servicio será accesible solo dentro del clúster.
port: El puerto en el que estará expuesto el servicio, que es el 80.
ingress:

enabled: Si está habilitado o no, aquí está habilitado.
hostname: El nombre de dominio bajo el cual se accederá a la aplicación (myapp.example.com).
annotations: Permite añadir anotaciones adicionales al recurso de ingreso.
resources:

limits: Define los recursos máximos que puede usar el pod, en este caso, 500 milicores de CPU y 512 MiB de memoria.
requests: Establece la cantidad mínima de recursos que se garantiza para el pod, 200 milicores de CPU y 256 MiB de memoria.
probes:

liveness: Configura la sonda de vivacidad para verificar si la aplicación está viva.
initialDelaySeconds: Espera 10 segundos antes de iniciar la comprobación.
periodSeconds: Comprueba cada 10 segundos.
failureThreshold: Permite 3 fallos antes de reiniciar el pod.
path: URL a consultar para la verificación.
readiness: Configura la sonda de disponibilidad.
initialDelaySeconds: Espera 5 segundos antes de iniciar la comprobación.
periodSeconds: Comprueba cada 10 segundos.
failureThreshold: Permite 3 fallos antes de considerar el pod no listo.
path: URL a consultar para la verificación.
hpa (Horizontal Pod Autoscaler):

enabled: Activa el escalador automático.
minReplicas: Número mínimo de réplicas que se mantendrán, en este caso 2.
maxReplicas: Número máximo de réplicas que pueden escalarse, aquí 10.
targetCPUUtilizationPercentage: Porcentaje de utilización de CPU que se utilizará como objetivo para la escala automática, 80%.
nodeSelector, tolerations, affinity:

Secciones para controlar cómo se programan los pods en los nodos del clúster, pero están vacías aquí, lo que significa que no hay restricciones específicas.
env:

Define las variables de entorno que se pasarán al pod.
name: Nombre de la variable (EXAMPLE_ENV).
value: Valor que tendrá la variable ("example-value").
En resumen, este archivo define la configuración necesaria para desplegar una aplicación en Kubernetes, incluyendo el número de réplicas, información del contenedor, configuración del servicio, sonda de salud, escala
