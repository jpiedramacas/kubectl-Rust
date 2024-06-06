# Rust on Kubernetes with Minikube

Este repositorio proporciona una guía detallada sobre cómo desplegar y escalar una aplicación Rust en Kubernetes usando Minikube. A continuación, se explica cada paso, comando y código necesario para lograrlo.

## Prerrequisitos

Antes de comenzar, asegúrate de tener instalados los siguientes componentes en tu entorno de desarrollo:

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Docker](https://docs.docker.com/get-docker/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## Configuración

### 1. Iniciar el clúster de Minikube

Para iniciar un clúster de Minikube, utiliza el siguiente comando:

```sh
minikube start
```

### 2. Configurar el entorno de Docker para usar el daemon de Docker de Minikube

Ejecuta el siguiente comando para que Docker use el daemon de Docker de Minikube:

```sh
minikube docker-env
```

### 3. Obtener la dirección IP de Minikube y reemplazarla en el archivo `deployment.yaml`

Para obtener la dirección IP de Minikube, utiliza:

```sh
minikube ip
```

Luego, reemplaza la dirección IP en la línea 17 del archivo `deployment.yaml`.

### 4. Habilitar el servidor de métricas en Minikube

Para habilitar el servidor de métricas, ejecuta:

```sh
minikube addons enable metrics-server
```

## Construcción y Despliegue

### 1. Construir la imagen Docker

Construye la imagen Docker de la aplicación Rust:

```sh
docker build --tag $(minikube ip):5000/hello-world-rust .
```

### 2. Aplicar las configuraciones de despliegue y servicio

Despliega la aplicación y el servicio en el clúster de Kubernetes:

```sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## Escalado

### Configurar el Autoscaler Horizontal de Pods (HPA)

Aplica la configuración de HPA para habilitar el escalado automático basado en la utilización de CPU:

```sh
kubectl apply -f hpa.yaml
```

## Pruebas de Carga

### Aplicar el despliegue del generador de carga

Para generar carga en la aplicación y probar el escalado automático, aplica la configuración del generador de carga:

```sh
kubectl apply -f load-generator.yaml
```

## Monitoreo

### Monitorear el HPA

Monitorea el HPA para ver el incremento en la utilización de CPU y el escalado de los pods:

```sh
kubectl get hpa hello-world-hpa --watch
```

Espera unos minutos hasta que el Autoscaler aumente el número de pods, luego verifica el número de nuevos pods creados:

```sh
kubectl get pods
```

### Eliminar el generador de carga

Una vez que las pruebas con el HPA hayan finalizado, elimina el generador de carga para reducir el número de pods:

```sh
kubectl delete deployment load-generator
```

## Escaneo de Vulnerabilidades con Trivy

Para garantizar la seguridad de la imagen Docker, este repositorio incluye un flujo de trabajo de GitHub Actions para la integración continua de las imágenes Docker. El flujo de trabajo se activa en los `push` y `pull requests` a la rama principal. Construye la imagen Docker y la escanea en busca de vulnerabilidades utilizando el escáner de código abierto "Trivy" de Aqua Security.

Aquí está la configuración del flujo de trabajo ubicada en `.github/workflows/docker-image.yaml`:

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
    - uses: actions/checkout@v4
    - name: Test the Docker image
      run: docker build . --file Dockerfile --tag hello-world-rust && docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image hello-world-rust:latest
```

## Estructura del Repositorio

- `.github/workflows`: Contiene los flujos de trabajo de GitHub Actions.
- `src`: Directorio de código fuente de la aplicación Rust.
- `Cargo.toml`: Archivo de configuración de Rust.
- `Dockerfile`: Define cómo construir la imagen Docker de la aplicación.
- `README.md`: Este archivo, contiene la documentación del proyecto.
- `deployment.yaml`: Configuración del despliegue de Kubernetes.
- `hpa.yaml`: Configuración del Autoscaler Horizontal de Pods.
- `load-generator.yaml`: Configuración del generador de carga.
- `service.yaml`: Configuración del servicio de Kubernetes.

Este README proporciona una guía completa y detallada para desplegar y escalar una aplicación Rust en Kubernetes utilizando Minikube, asegurando que los desarrolladores puedan seguir cada paso de manera eficiente y efectiva.
