# plantilla-helm

## Estructura 
# mychart/
## ├── charts/
## ├── templates/
### │   ├── deployment.yaml
### │   ├── service.yaml
###│   ├── ingress.yaml
### │   ├── configmap.yaml
### │   ├── secret.yaml
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

apiVersion: apps/v1: Indica la versión de la API de Kubernetes que se está utilizando, en este caso, la versión v1 del grupo de API de aplicaciones.

kind: Deployment: Especifica que el recurso que se está definiendo es un Deployment.

metadata: Contiene información sobre el Deployment:

name: El nombre del Deployment se construye a partir del nombre del lanzamiento (Release) y se le añade "-app".
labels: Se asignan etiquetas para identificar el Deployment, basándose también en el nombre del lanzamiento.
spec: Define la configuración deseada para el Deployment:

replicas: Especifica cuántas copias (réplicas) del contenedor se deben estar ejecutando.
selector: Define cómo seleccionar los Pods que este Deployment debe gestionar, coincidiendo con las etiquetas especificadas.
template: Especifica la plantilla para los Pods que se crearán:
metadata: Contiene etiquetas para el Pod, similar a las del Deployment.
spec: Detalla la especificación del contenedor:
containers: Lista de contenedores que se ejecutarán en el Pod:
name: Nombre del contenedor.
image: Define la imagen del contenedor, que utiliza valores de variables dinámicas como el repositorio y la etiqueta de la imagen.
imagePullPolicy: Política de obtención de la imagen (por ejemplo, si debe actualizar la imagen cada vez).
ports: Especifica los puertos en los que el contenedor está escuchando (en este caso, el puerto 80).
env: Define las variables de entorno, utilizando un bucle para iterar sobre las definiciones de variables de entorno proporcionadas en los valores.
resources: (opcional) Establece los recursos (CPU y memoria) requeridos por el contenedor, si están definidos en los valores.
nodeSelector: Indica en qué nodos debe ejecutarse el Pod, según las etiquetas de los nodos.
tolerations: Configura las tolerancias, permitiendo que el Pod se ejecute en nodos con ciertas "tolerancias" (es decir, capacidades de manejar condiciones específicas).
affinity: Define reglas de afinidad para determinar en qué nodos se deben programar los Pods, en base a las relaciones con otros Pods o nodos.
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
Este archivo define los valores por defecto para las variables que se utilizan en las plantillas. Permite personalizar la configuración del gráfico durante la instalación o actualización.
A continuación se explican las secciones más importantes:

replicaCount: 2
Indica que deben ejecutarse dos réplicas de la aplicación para garantizar disponibilidad y escalabilidad.
image:
repository: myrepo/myapp: Especifica el repositorio de la imagen de contenedor que se va a utilizar. En este caso, se encuentra en "myrepo" y se llama "myapp".
tag: "latest": Indica que se debe utilizar la etiqueta "latest" de la imagen, que normalmente corresponde a la versión más reciente.
pullPolicy: IfNotPresent: Esta política determina que el sistema solo descargará la imagen si no está ya presente en el nodo.
service:
type: ClusterIP: Define que el servicio solo será accesible dentro del clúster de Kubernetes.
port: 80: Especifica que el servicio escuchará en el puerto 80.
ingress:
enabled: true: Indica que el ingreso (Ingress) está habilitado, lo que permite el acceso externo a la aplicación.
hostname: myapp.example.com: Define el nombre de dominio a través del cual se accederá a la aplicación.
annotations: {}: Un campo para agregar anotaciones adicionales, aunque en este caso no se han definido.
resources:
Este campo se utiliza para definir los recursos (CPU, memoria) que se asignarán a los pods. Está vacío, lo que significa que no se han especificado configuraciones.
nodeSelector:
Permite seleccionar en qué nodos debe ejecutarse el pod basado en etiquetas. Está vacío, lo que sugiere que no se han establecido restricciones.
tolerations:
Es permite que los pods sean programados en nodos con ciertas "tolerancias" (puedes pensar en esto como excepciones a los "taints" de los nodos). Aquí también está vacío.
affinity:
Permite especificar reglas avanzadas sobre cómo los pods deben ser asignados a los nodos, en función de etiquetas. Este campo también está vacío.
env:
- name: EXAMPLE_ENV: Define una variable de entorno para el pod.
value: "example-value": Establece el valor de la variable de entorno a "example-value".


En resumen, esta estructura es típica de un gráfico de Helm, donde se gestionan los recursos necesarios para desplegar una aplicación en Kubernetes.
