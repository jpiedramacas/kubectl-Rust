# Rust on Kubernetes with Minikube

Este repositorio proporciona una guía detallada sobre cómo desplegar y escalar una aplicación Rust en Kubernetes usando Minikube. A continuación, se explica cada paso, comando y archivo necesario para lograrlo.
[INSTRUCCIONES](https://github.com/jpiedramacas/kubectl-rust/blob/main/code_test.md)

## Estructura del Repositorio

- `.github/workflows`: Contiene los flujos de trabajo de GitHub Actions.
- `src`: Directorio de código fuente de la aplicación Rust.
- `Cargo.toml`: Archivo de configuración de Rust, especifica las dependencias y la configuración del proyecto.
- `Dockerfile`: Define cómo construir la imagen Docker de la aplicación.
- `README.md`: Este archivo, contiene la documentación del proyecto.
- `deployment.yaml`: Configuración del despliegue de Kubernetes.
- `hpa.yaml`: Configuración del Autoscaler Horizontal de Pods.
- `load-generator.yaml`: Configuración del generador de carga.
- `service.yaml`: Configuración del servicio de Kubernetes.

## Prerrequisitos

Antes de comenzar, asegúrate de tener instalados los siguientes componentes en tu entorno de desarrollo:

- [Minikube](https://minikube.sigs.k8s.io/docs/start/): Herramienta que permite ejecutar un clúster de Kubernetes localmente.
- [Docker](https://docs.docker.com/get-docker/): Plataforma para desarrollar, enviar y ejecutar aplicaciones dentro de contenedores.
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/): Herramienta de línea de comandos para interactuar con clústeres de Kubernetes.


## Configuración

### 1. Iniciar el clúster de Minikube

Para iniciar un clúster de Minikube, utiliza el siguiente comando:

```bash
minikube start
```

Este comando inicializa un clúster de Kubernetes en tu máquina local usando Minikube. Si es la primera vez que ejecutas este comando, Minikube descargará una máquina virtual y configurará Kubernetes en ella.

### 2. Configurar el entorno de Docker para usar el daemon de Docker de Minikube

Para configurar Docker para que use el daemon de Docker que Minikube está utilizando, ejecuta el siguiente comando:

```bash
eval $(minikube docker-env)
```

Este comando imprime una serie de variables de entorno que configuran Docker para que use el daemon de Docker dentro de Minikube.

Para Windows PowerShell, configura manualmente las variables de entorno proporcionadas por el comando anterior. Copia y pega los comandos generados por `minikube docker-env`:

```powershell
$Env:DOCKER_TLS_VERIFY = "1"
$Env:DOCKER_HOST = "tcp://<MINIKUBE_IP>:2376"
$Env:DOCKER_CERT_PATH = "C:\Users\CursosTardes\.minikube\certs"
$Env:MINIKUBE_ACTIVE_DOCKERD = "minikube"
```

### 3. Obtener la dirección IP de Minikube y reemplazarla en el archivo deployment.yaml

Para obtener la dirección IP de Minikube, utiliza:

```bash
minikube ip
```

Luego, reemplaza la dirección IP en la línea 17 del archivo `deployment.yaml` con la IP obtenida.

### 4. Habilitar el servidor de métricas en Minikube

Para habilitar el servidor de métricas, ejecuta:

```bash
minikube addons enable metrics-server
```

Este comando habilita el servidor de métricas en Minikube, necesario para el autoscaling basado en la utilización de recursos.

## Construcción y Despliegue

### 1. Construir la imagen Docker

Construye la imagen Docker de la aplicación Rust con el siguiente comando:

```bash
docker build --tag $(minikube ip):5000/hello-world-rust .
```

Este comando construye una imagen Docker y la etiqueta con la dirección IP de Minikube y el nombre especificado.

### 2. Aplicar las configuraciones de despliegue y servicio

Despliega la aplicación y el servicio en el clúster de Kubernetes utilizando los siguientes comandos:

```bash
kubectl apply -f deployment.yaml
```

```bash
kubectl apply -f service.yaml
```

- `deployment.yaml`: Define la configuración del despliegue de la aplicación en Kubernetes, incluyendo el número de réplicas y la imagen Docker a usar.
- `service.yaml`: Define el servicio de Kubernetes que expone la aplicación desplegada, permitiendo el acceso desde fuera del clúster.

## Escalado

### Configurar el Autoscaler Horizontal de Pods (HPA)

Aplica la configuración de HPA para habilitar el escalado automático basado en la utilización de CPU:

```bash
kubectl apply -f hpa.yaml
```

- `hpa.yaml`: Define el Autoscaler Horizontal de Pods, que automáticamente escala la cantidad de réplicas de la aplicación basada en la utilización de CPU.

## Pruebas de Carga

### Aplicar el despliegue del generador de carga

Para generar carga en la aplicación y probar el escalado automático, aplica la configuración del generador de carga:

```bash
kubectl apply -f load-generator.yaml
```

- `load-generator.yaml`: Despliega un generador de carga que envía solicitudes a la aplicación para simular tráfico y probar el escalado.

## Monitoreo

### Monitorear el HPA

Monitorea el HPA para ver el incremento en la utilización de CPU y el escalado de los pods:

```bash
kubectl get hpa hello-world-hpa --watch
```

Espera unos minutos hasta que el Autoscaler aumente el número de pods, luego verifica el número de nuevos pods creados:

```bash
kubectl get pods
```

### Eliminar el generador de carga

Una vez que las pruebas con el HPA hayan finalizado, elimina el generador de carga para reducir el número de pods:

```bash
kubectl delete deployment load-generator
```

## Desplegar

```bash
kubectl get service
minikube service hello-world-service
```

```bash
kubectl get pods
kubectl port-forward pod/hello-world-deployment
```

## Conclusión

Siguiendo estos pasos, habrás configurado exitosamente un clúster de Minikube, desplegado una aplicación, habilitado el escalado automático basado en métricas y probado el sistema bajo carga. Cada comando y archivo de configuración juega un rol crucial en el correcto funcionamiento del clúster y la aplicación desplegada.



# Escaneo de Vulnerabilidades con Trivy en GitHub Actions

Este documento proporciona una guía detallada para configurar un flujo de trabajo de GitHub Actions que construye una imagen Docker y la escanea en busca de vulnerabilidades utilizando Trivy, un escáner de seguridad de código abierto de Aqua Security.

## Requisitos Previos

- Cuenta en GitHub y un repositorio con los archivos de configuración necesarios.
- Dockerfile presente en el repositorio.
- Conocimiento básico de YAML y GitHub Actions.

## Configuración del Flujo de Trabajo

### 1. Crear el Archivo de Flujo de Trabajo

El archivo de flujo de trabajo de GitHub Actions debe ubicarse en el directorio `.github/workflows/` dentro de tu repositorio. Crea un archivo llamado `docker-image.yaml` en esa ubicación.

### 2. Configurar el Flujo de Trabajo

Aquí está la configuración completa del flujo de trabajo `docker-image.yaml`:

```yaml
name: Docker Image CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - name: Check out the repository
      uses: actions/checkout@v4

    - name: Build the Docker image
      run: docker build . --file Dockerfile --tag hello-world-rust

    - name: Scan the Docker image for vulnerabilities
      run: docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image hello-world-rust:latest
```

### Explicación de la Configuración

#### `name: Docker Image CI`

Este es el nombre del flujo de trabajo. Puedes personalizarlo según tus necesidades.

#### `on`

Define los eventos que activarán el flujo de trabajo. En este caso, el flujo de trabajo se activa en:

- `push` a la rama principal (`main`).
- `pull_request` hacia la rama principal (`main`).

#### `jobs`

Define los trabajos que se ejecutarán como parte del flujo de trabajo. Aquí tenemos un solo trabajo llamado `test`.

#### `runs-on: ubuntu-latest`

Especifica el sistema operativo en el que se ejecutará el trabajo. En este caso, se usa la última versión de Ubuntu.

#### `steps`

Lista las acciones individuales que componen el trabajo:

1. **Check out the repository**: Utiliza la acción `actions/checkout@v4` para clonar el repositorio en el entorno de ejecución de GitHub Actions.

    ```yaml
    - name: Check out the repository
      uses: actions/checkout@v4
    ```

2. **Build the Docker image**: Construye la imagen Docker utilizando el Dockerfile presente en el repositorio.

    ```yaml
    - name: Build the Docker image
      run: docker build . --file Dockerfile --tag hello-world-rust
    ```

3. **Scan the Docker image for vulnerabilities**: Ejecuta Trivy para escanear la imagen Docker en busca de vulnerabilidades.

    ```yaml
    - name: Scan the Docker image for vulnerabilities
      run: docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image hello-world-rust:latest
    ```

    - `--rm`: Elimina el contenedor Trivy después de que finalice la ejecución.
    - `-v /var/run/docker.sock:/var/run/docker.sock`: Monta el socket Docker dentro del contenedor Trivy para que pueda comunicarse con el daemon Docker.
    - `aquasec/trivy image hello-world-rust:latest`: Ejecuta Trivy para escanear la imagen `hello-world-rust:latest`.

## Conclusión

Siguiendo estos pasos, habrás configurado un flujo de trabajo de GitHub Actions que construye una imagen Docker y la escanea en busca de vulnerabilidades utilizando Trivy. Este proceso garantiza que las imágenes Docker en tu repositorio sean seguras y libres de vulnerabilidades conocidas. Cada paso en la configuración del flujo de trabajo tiene un propósito específico y es crucial para el correcto funcionamiento de la integración continua y la seguridad de la imagen.
