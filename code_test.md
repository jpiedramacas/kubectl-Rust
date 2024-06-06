# Despliegue de Aplicación Rust "Hello World" en Kubernetes con Minikube

Este proyecto tiene como objetivo desplegar una aplicación "Hello World" en Rust utilizando Docker en un clúster de Kubernetes gestionado por Minikube. Además, se puede integrar un escaneo de vulnerabilidades o escalado automático (opcional).

## Objetivo

Desplegar una aplicación "Hello World" en Rust utilizando Docker en un clúster de Kubernetes gestionado por Minikube. Integrar opcionalmente el escaneo de vulnerabilidades o el autoscaling (no obligatorio).

## Requisitos

### 1. Configurar el Clúster de Minikube

- Instalar y configurar Minikube o una alternativa para simular un entorno de Kubernetes local.

### 2. Despliegue de la Aplicación

- Contenerizar y desplegar la aplicación en el clúster de Kubernetes.

### 3. Documentación

Proporcionar un archivo `README.md` con instrucciones claras sobre cómo:

- Configurar un clúster de Kubernetes local (por ejemplo, con Minikube).
- Construir y desplegar la aplicación.
- Ejecutar el escaneo de SonarQube o realizar el escalado automático.

### 4. Autoscaling (Opcional)

- Implementar autoscaling basado en la utilización de CPU utilizando las capacidades de Kubernetes.
- Documentar cómo configurar, probar y observar el autoscaling.

### 5. Escaneo de Vulnerabilidades con SonarQube (Opcional)

- Integrar una acción de GitHub para escanear la imagen Docker en busca de vulnerabilidades como parte del proceso de despliegue.
- Documentar el proceso de configuración y escaneo en el README.

## Criterios de Evaluación

- **Corrección:** La aplicación debe ser correctamente desplegable y accesible en un entorno Docker local.
- **Seguridad:** La imagen Docker debe ser escaneada en busca de vulnerabilidades críticas reportadas por SonarQube (opcional).
- **Calidad del Código:** El código DevOps debe ser limpio, bien organizado y mantenible.
- **Calidad de la Documentación:** La documentación debe guiar claramente los procesos de configuración, despliegue y escaneo.

## Instrucciones de Envío

- Proporcionar un enlace a un repositorio de GitHub que contenga todos los scripts de despliegue, Dockerfile(s), configuración de SonarQube y documentación.
- Asegurarse de que todos los scripts sean ejecutables y que la documentación sea completa y fácil de seguir.

## Configuración

### 1. Iniciar el Clúster de Minikube

Para iniciar un clúster de Minikube, utiliza el siguiente comando:

```sh
minikube start
```

### 2. Configurar el Entorno de Docker para Usar el Daemon de Docker de Minikube

En Windows PowerShell:

```ps1
$Env:DOCKER_TLS_VERIFY = "1"
$Env:DOCKER_HOST = "tcp://<MINIKUBE_IP>:2376"
$Env:DOCKER_CERT_PATH = "C:\Users\TuUsuario\.minikube\certs"
$Env:MINIKUBE_ACTIVE_DOCKERD = "minikube"
```

### 3. Obtener la Dirección IP de Minikube

```sh
minikube ip
```

Reemplaza la dirección IP en la línea 17 del archivo `deployment.yaml`.

### 4. Habilitar el Servidor de Métricas en Minikube

```sh
minikube addons enable metrics-server
```

## Construcción y Despliegue

### 1. Construir la Imagen Docker

```sh
docker build --tag $(minikube ip):5000/hello-world-rust .
```

### 2. Aplicar las Configuraciones de Despliegue y Servicio

```sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## Escalado Automático (Opcional)

### Configurar el Autoscaler Horizontal de Pods (HPA)

```sh
kubectl apply -f hpa.yaml
```

### Monitorear el HPA

```sh
kubectl get hpa hello-world-hpa --watch
kubectl get pods
```

### Eliminar el Generador de Carga

```sh
kubectl delete deployment load-generator
```

## Escaneo de Vulnerabilidades con SonarQube (Opcional)

### Configuración del Flujo de Trabajo de GitHub Actions

En `.github/workflows/docker-image.yaml`:

```yaml
name: Docker Image CI

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

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