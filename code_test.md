# README.md

## Descripción del Proyecto

Este proyecto tiene como objetivo desplegar una aplicación "Hello World" escrita en Rust utilizando Docker en un clúster de Kubernetes gestionado por Minikube. Adicionalmente, se puede integrar un escaneo de vulnerabilidades o autoescalado (opcional).

## Requisitos

- Instalar y configurar Minikube para simular un entorno Kubernetes local.
- Contenerizar y desplegar la aplicación en el clúster de Kubernetes.
- Proveer una documentación clara con instrucciones para:
  - Configurar un clúster de Kubernetes local (por ejemplo, con Minikube).
  - Construir y desplegar la aplicación.
  - Ejecutar el escaneo con SonarQube o realizar el autoescalado (opcional).
- Implementar autoescalado basado en la utilización de CPU usando capacidades de Kubernetes (opcional).
- Integrar SonarQube en un Github Action para escanear la imagen Docker en busca de vulnerabilidades como parte del proceso de despliegue (opcional).

## Instrucciones

### 1. Configurar Minikube

#### Instalación de Minikube

1. **Instalar Minikube**:

   ```sh
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   ```

2. **Iniciar Minikube**:

   ```sh
   minikube start --driver=docker
   ```

#### Verificación de la Instalación

1. **Comprobar el estado del clúster**:

   ```sh
   minikube status
   ```

### 2. Construir y Desplegar la Aplicación

#### Crear la Aplicación en Rust

1. **Instalar Rust y Cargo**:

   ```sh
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   source $HOME/.cargo/env
   ```

2. **Crear un nuevo proyecto en Rust**:

   ```sh
   cargo new hello_world
   cd hello_world
   ```

3. **Modificar `src/main.rs`**:

   ```rust
   fn main() {
       println!("Hello, world!");
   }
   ```

#### Crear el Dockerfile

1. **Crear un `Dockerfile` en el directorio del proyecto**:

   ```Dockerfile
   FROM rust:latest

   WORKDIR /usr/src/myapp
   COPY . .

   RUN cargo install --path .

   CMD ["hello_world"]
   ```

2. **Construir la imagen Docker**:

   ```sh
   docker build -t hello_world .
   ```

#### Desplegar en Kubernetes

1. **Crear un archivo de despliegue `deployment.yaml`**:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hello-world-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: hello-world
     template:
       metadata:
         labels:
           app: hello-world
       spec:
         containers:
         - name: hello-world
           image: hello_world:latest
           ports:
           - containerPort: 8080
   ```

2. **Aplicar el despliegue**:

   ```sh
   kubectl apply -f deployment.yaml
   ```

3. **Crear un servicio para exponer la aplicación**:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: hello-world-service
   spec:
     selector:
       app: hello-world
     ports:
       - protocol: "TCP"
         port: 80
         targetPort: 8080
     type: LoadBalancer
   ```

4. **Aplicar el servicio**:

   ```sh
   kubectl apply -f service.yaml
   ```

### 3. Escaneo de Vulnerabilidades con SonarQube (Opcional)

#### Integración con GitHub Actions

1. **Crear un workflow en `.github/workflows/sonarqube.yml`**:

   ```yaml
   name: SonarQube Scan

   on: [push]

   jobs:
     sonar:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v2
         - name: Set up JDK 11
           uses: actions/setup-java@v1
           with:
             java-version: '11'
         - name: SonarQube Scan
           env:
             SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
           run: |
             sonar-scanner \
               -Dsonar.projectKey=my_project \
               -Dsonar.sources=. \
               -Dsonar.host.url=https://sonarqube.example.com \
               -Dsonar.login=$SONAR_TOKEN
   ```

2. **Configurar las credenciales en GitHub**: Agregar `SONAR_TOKEN` en los secretos del repositorio.

### 4. Autoescalado Basado en la Utilización de CPU (Opcional)

#### Configurar el Autoescalado

1. **Crear una Horizontal Pod Autoscaler**:

   ```sh
   kubectl autoscale deployment hello-world-deployment --cpu-percent=50 --min=1 --max=10
   ```

2. **Verificar el autoescalado**:

   ```sh
   kubectl get hpa
   ```

### Observaciones y Pruebas

1. **Acceder a la aplicación**:

   Obtener la URL del servicio:

   ```sh
   minikube service hello-world-service --url
   ```

2. **Probar el autoescalado**:

   Simular carga en la aplicación para verificar el escalado automático.

### Conclusión

Este README proporciona una guía paso a paso para desplegar una aplicación "Hello World" en un clúster de Kubernetes gestionado por Minikube, con opciones adicionales de escaneo de vulnerabilidades y autoescalado. Siguiendo estas instrucciones, deberías poder configurar el entorno, construir y desplegar la aplicación, y opcionalmente, mejorar la seguridad y escalabilidad del despliegue.
