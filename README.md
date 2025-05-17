Introducción
Esta documentación detalla cómo implementar una aplicación utilizando Helm en Kubernetes, con gestión de secretos a través de Vault y Argo CD para el despliegue y la sincronización continua desde un repositorio Git.

La configuración aborda la creación de un Deployment para una aplicación que se configura con valores parametrizados a través de un archivo values.yaml. Se integra con Vault para gestionar secretos de manera segura, como el token de Vault, y se aprovechan las capacidades de Argo CD para la automatización del despliegue continuo.

Arquitectura y Componentes
Helm:

Se utiliza para empaquetar y gestionar la configuración de Kubernetes de la aplicación.

Los valores de configuración, como imágenes de contenedor, variables de entorno y recursos, se gestionan a través de un archivo values.yaml.

Vault:

Utilizado para almacenar y gestionar secretos (por ejemplo, tokens) de manera segura.

El token de Vault se obtiene desde un Secret de Kubernetes, referenciado en el Deployment de la aplicación.

Argo CD:

Herramienta de entrega continua (CD) para la sincronización y despliegue automático de aplicaciones Kubernetes desde un repositorio Git.

Argo CD gestiona la sincronización de los manifiestos de Kubernetes, incluidos los archivos de Helm y values.yaml.

Estructura del Proyecto
La estructura del proyecto en el repositorio Git debe ser algo similar a esto:

pgsql
Copiar
Editar
myapp-helm-chart/
  ├── charts/
  ├── values.yaml
  ├── templates/
  ├── Chart.yaml
charts/: subcharts de Helm si los tienes.

values.yaml: archivo de valores parametrizados.

templates/: plantillas de Kubernetes, como deployment.yaml, service.yaml, etc.

Chart.yaml: archivo que define el chart de Helm.

Integración con Vault
Configuración de Secretos con Vault
En el archivo values.yaml, se incluye la configuración para gestionar el token de Vault y la dirección del servidor Vault:

yaml
Copiar
Editar
vault:
  enabled: true
  address: "http://vault-address:8200"
  tokenSecretName: "myapp-vault-token-secret"
  tokenSecretKey: "token"
  secretPath: "secret/myapp"
tokenSecretName: Nombre del Kubernetes Secret que contiene el token de Vault.

tokenSecretKey: La clave del secret que contiene el valor del token.

address: La dirección del servidor Vault donde se almacenan los secretos.

secretPath: La ruta del secreto en Vault que será utilizado por la aplicación.

Token de Vault en Kubernetes Secret
El token de Vault debe estar almacenado en un Kubernetes Secret que será referenciado en el Deployment. Ejemplo:

yaml
Copiar
Editar
apiVersion: v1
kind: Secret
metadata:
  name: myapp-vault-token-secret
type: Opaque
data:
  token: <base64-encoded-token>
Este secret contiene el token necesario para interactuar con Vault, y se referenciará en el Deployment.

Helm Chart - Deployment
El manifiesto de deployment.yaml describe cómo se debe configurar el Deployment de la aplicación y cómo interactuar con Vault para obtener el token. El manifiesto se parametriza utilizando los valores del archivo values.yaml.

Deployment para la Aplicación
yaml
Copiar
Editar
{{- if .Values.app.enabled }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-myapp
  labels:
    app: {{ .Release.Name }}
spec:
  replicas: {{ .Values.app.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-myapp
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-myapp
    spec:
      containers:
        - name: app
          image: "{{ .Values.app.image.repository }}:{{ .Values.app.image.tag }}"
          imagePullPolicy: {{ .Values.app.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.app.service.port }}
          env:
            - name: VAULT_TOKEN
              valueFrom:
                secretKeyRef:
                  name: {{ .Values.vault.tokenSecretName }}
                  key: {{ .Values.vault.tokenSecretKey }}
            - name: VAULT_ADDR
              value: {{ .Values.vault.address }}
            {{- range .Values.env }}
            - name: {{ .name }}
              value: {{ .value | quote }}
            {{- end }}
          {{- if .Values.probes.liveness.enabled }}
          livenessProbe:
            httpGet:
              path: {{ .Values.probes.liveness.path }}
              port: 80
            initialDelaySeconds: {{ .Values.probes.liveness.initialDelaySeconds }}
            periodSeconds: {{ .Values.probes.liveness.periodSeconds }}
            failureThreshold: {{ .Values.probes.liveness.failureThreshold }}
          {{- end }}
          {{- if .Values.probes.readiness.enabled }}
          readinessProbe:
            httpGet:
              path: {{ .Values.probes.readiness.path }}
              port: 80
            initialDelaySeconds: {{ .Values.probes.readiness.initialDelaySeconds }}
            periodSeconds: {{ .Values.probes.readiness.periodSeconds }}
            failureThreshold: {{ .Values.probes.readiness.failureThreshold }}
          {{- end }}
          resources:
            limits:
              cpu: {{ .Values.resources.limits.cpu }}
              memory: {{ .Values.resources.limits.memory }}
            requests:
              cpu: {{ .Values.resources.requests.cpu }}
              memory: {{ .Values.resources.requests.memory }}
      nodeSelector: {{ .Values.nodeSelector | default {} }}
      tolerations: {{ .Values.tolerations | default [] }}
      affinity: {{ .Values.affinity | default {} }}
      restartPolicy: Always
{{- end }}
Explicación:
Contenedores: El contenedor utiliza los valores de values.yaml para definir la imagen, el puerto y las variables de entorno.

Vault: Se pasan las credenciales de Vault a través de variables de entorno, utilizando un Kubernetes Secret.

Probes (Liveness y Readiness): Se configuran probes de Kubernetes para la aplicación si están habilitados.

Recursos: Se gestionan los recursos (CPU, memoria) basados en lo que esté configurado en values.yaml.

Estrategia de despliegue: Las configuraciones de nodeSelector, tolerations y affinity permiten un despliegue más específico de la aplicación según los requisitos de infraestructura.

Integración con Argo CD
Configuración de Argo CD para Deploy con Helm
Para sincronizar y desplegar el chart de Helm con Argo CD, debes agregar el repositorio donde está alojado el chart y la configuración de Helm.

Agregar Repositorio de Helm a Argo CD: En la interfaz de usuario de Argo CD o usando la CLI, agrega el repositorio de Helm:

bash
Copiar
Editar
argocd repo add <repo-name> --type helm --url <helm-repo-url> --name <name> --username <username> --password <password>
Crear Aplicación en Argo CD: Al crear la aplicación en Argo CD, debes referenciar el repositorio Git donde se encuentra el chart y el archivo values.yaml.

Con la CLI de Argo CD:

bash
Copiar
Editar
argocd app create myapp \
  --repo https://github.com/myorg/myapp-helm-chart \
  --path myapp-helm-chart \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --helm-set-file values.yaml \
  --sync-policy automated
También puedes crear la aplicación directamente desde la interfaz web de Argo CD, especificando el repositorio y el archivo values.yaml.

Sincronización Automática: Puedes habilitar la sincronización automática para que Argo CD aplique cambios en Kubernetes cada vez que se detecten cambios en el repositorio de Git, incluidos los archivos de valores o manifiestos.

Conclusión
Este enfoque proporciona una solución robusta y segura para el despliegue de aplicaciones en Kubernetes utilizando Helm, gestionando secretos a través de Vault y automatizando el proceso con Argo CD. Esto permite la implementación continua de aplicaciones mientras se asegura que los secretos y otros valores sensibles se gestionen de manera segura.



1. Estructura General del Helm Chart
El Helm Chart propuesto tiene una estructura básica de un chart de Helm con los siguientes componentes:

Chart.yaml: Define el chart de Helm.

values.yaml: Contiene los valores que serán parametrizados y utilizados en las plantillas de Kubernetes.

templates/: Contiene las plantillas de los manifiestos de Kubernetes como deployment.yaml, service.yaml, ingress.yaml, etc.

2. Archivo values.yaml
El archivo values.yaml contiene los valores que se usan para parametrizar los manifiestos de Kubernetes en las plantillas. Estos valores se pueden modificar para ajustar la configuración de la aplicación y el despliegue.

Estructura del values.yaml
yaml
Copiar
Editar
vault:
  enabled: true
  address: "http://vault-address:8200"  # Dirección de Vault
  tokenSecretName: "myapp-vault-token-secret"  # Nombre del Secret que contiene el token de Vault
  tokenSecretKey: "token"  # Clave en el Secret que contiene el token
  secretPath: "secret/myapp"  # Ruta del secreto en Vault

app:
  enabled: true
  replicaCount: 2
  image:
    repository: "myrepo/myapp"
    tag: "latest"
    pullPolicy: IfNotPresent
  service:
    port: 80

resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
  requests:
    cpu: "200m"
    memory: "256Mi"

probes:
  liveness:
    enabled: true
    initialDelaySeconds: 10
    periodSeconds: 10
    failureThreshold: 3
    path: /health
  readiness:
    enabled: true
    initialDelaySeconds: 5
    periodSeconds: 10
    failureThreshold: 3
    path: /ready

nodeSelector: {}
tolerations: []
affinity: {}

env:
  - name: EXAMPLE_ENV
    value: "example-value"
Explicación de los valores en values.yaml:
Vault:

vault.enabled: Si se establece en true, habilita la integración con Vault.

vault.address: Dirección del servidor de Vault para obtener los secretos.

vault.tokenSecretName: Nombre del Secret de Kubernetes que contiene el token de Vault.

vault.tokenSecretKey: Clave del Secret donde se encuentra el token.

vault.secretPath: Ruta del secreto dentro de Vault para acceder a los datos.

app:

app.enabled: Habilita o deshabilita la creación del Deployment de la aplicación.

app.replicaCount: Número de réplicas de la aplicación a desplegar.

app.image: Parámetros relacionados con la imagen Docker de la aplicación (repositorio, etiqueta y política de obtención).

app.service.port: Puerto del servicio de la aplicación.

probes:

liveness: Configura la comprobación de salud para verificar si el contenedor está funcionando.

readiness: Configura la comprobación de disponibilidad para verificar si el contenedor está listo para recibir tráfico.

Recursos:

Define los límites y solicitudes de recursos como CPU y memoria para el contenedor.

affinity, nodeSelector, tolerations:

Permiten personalizar el comportamiento de programación del pod en los nodos del clúster.

env:

Variables de entorno personalizadas que serán configuradas dentro del contenedor.

3. Plantilla de deployment.yaml
El archivo deployment.yaml es la plantilla que define cómo se debe desplegar la aplicación en Kubernetes. Aquí se usa el archivo values.yaml para parametrizar el despliegue.

Manifiesto de Deployment
yaml
Copiar
Editar
{{- if .Values.app.enabled }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-myapp
  labels:
    app: {{ .Release.Name }}
spec:
  replicas: {{ .Values.app.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-myapp
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-myapp
    spec:
      containers:
        - name: app
          image: "{{ .Values.app.image.repository }}:{{ .Values.app.image.tag }}"
          imagePullPolicy: {{ .Values.app.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.app.service.port }}
          env:
            - name: VAULT_TOKEN
              valueFrom:
                secretKeyRef:
                  name: {{ .Values.vault.tokenSecretName }}
                  key: {{ .Values.vault.tokenSecretKey }}
            - name: VAULT_ADDR
              value: {{ .Values.vault.address }}
            {{- range .Values.env }}
            - name: {{ .name }}
              value: {{ .value | quote }}
            {{- end }}
          {{- if .Values.probes.liveness.enabled }}
          livenessProbe:
            httpGet:
              path: {{ .Values.probes.liveness.path }}
              port: 80
            initialDelaySeconds: {{ .Values.probes.liveness.initialDelaySeconds }}
            periodSeconds: {{ .Values.probes.liveness.periodSeconds }}
            failureThreshold: {{ .Values.probes.liveness.failureThreshold }}
          {{- end }}
          {{- if .Values.probes.readiness.enabled }}
          readinessProbe:
            httpGet:
              path: {{ .Values.probes.readiness.path }}
              port: 80
            initialDelaySeconds: {{ .Values.probes.readiness.initialDelaySeconds }}
            periodSeconds: {{ .Values.probes.readiness.periodSeconds }}
            failureThreshold: {{ .Values.probes.readiness.failureThreshold }}
          {{- end }}
          resources:
            limits:
              cpu: {{ .Values.resources.limits.cpu }}
              memory: {{ .Values.resources.limits.memory }}
            requests:
              cpu: {{ .Values.resources.requests.cpu }}
              memory: {{ .Values.resources.requests.memory }}
      nodeSelector: {{ .Values.nodeSelector | default {} }}
      tolerations: {{ .Values.tolerations | default [] }}
      affinity: {{ .Values.affinity | default {} }}
      restartPolicy: Always
{{- end }}
Explicación de deployment.yaml:
Contenedor de la Aplicación:

image: El contenedor se configura con los valores del archivo values.yaml para definir el repositorio y la etiqueta de la imagen Docker.

env: Se añaden las variables de entorno para Vault, donde se utiliza un Secret de Kubernetes para obtener el token de Vault.

livenessProbe y readinessProbe: Estas se configuran si están habilitadas en el archivo values.yaml, y se utilizan para verificar si la aplicación está funcionando y lista para recibir tráfico.

Recursos:

Se especifican los límites y solicitudes de recursos como CPU y memoria que la aplicación necesita.

Programación:

nodeSelector, tolerations y affinity: Permiten definir reglas de programación para los pods en nodos específicos del clúster.

4. Configuración de Secretos en Kubernetes para Vault
El token de Vault se almacena como un Secret en Kubernetes. Este Secret será referenciado en el Deployment para que el contenedor pueda acceder a los secretos de Vault de forma segura.

Ejemplo de Kubernetes Secret para Vault
yaml
Copiar
Editar
apiVersion: v1
kind: Secret
metadata:
  name: myapp-vault-token-secret
type: Opaque
data:
  token: <base64-encoded-token>
En este ejemplo, el valor de token debe ser un token codificado en base64, que se almacena en un Secret de Kubernetes.

5. Uso de Argo CD para Despliegue Continuo
Sincronización con Argo CD
Argo CD es una herramienta de CD (Entrega Continua) que se utiliza para gestionar el despliegue y la sincronización de aplicaciones en Kubernetes directamente desde un repositorio Git.

Automatización de despliegues: Argo CD puede configurarse para que se sincronice automáticamente con el repositorio donde está almacenado el Helm Chart y el archivo values.yaml. Esto asegura que cualquier cambio en el código o en la configuración se refleje inmediatamente en el clúster de Kubernetes.
