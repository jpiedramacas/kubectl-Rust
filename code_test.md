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

1. **Instalar Minikube**: Este comando descarga el binario de Minikube y lo instala en tu sistema.

   ```sh
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   ```

   - `curl -LO`: Descarga un archivo desde una URL especificada.
   - `sudo install`: Copia el archivo descargado a `/usr/local/bin` y lo hace ejecutable.

2. **Iniciar Minikube**: Este comando inicia un clúster de Minikube usando Docker como driver.

   ```sh
   minikube start --driver=docker
   ```

   - `minikube start`: Inicia un nuevo clúster de Minikube.
   - `--driver=docker`: Especifica que Docker será usado como el driver para Minikube.

#### Verificación de la Instalación

1. **Comprobar el estado del clúster**: Este comando verifica que Minikube esté corriendo correctamente.

   ```sh
   minikube status
   ```

   - `minikube status`: Muestra el estado actual del clúster de Minikube.

### 2. Construir y Desplegar la Aplicación

#### Crear la Aplicación en Rust

1. **Instalar Rust y Cargo**: Estos comandos instalan Rust, el lenguaje de programación, y Cargo, el gestor de paquetes para Rust.

   ```sh
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   source $HOME/.cargo/env
   ```

   - `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`: Descarga e instala Rust.
   - `source $HOME/.cargo/env`: Configura las variables de entorno necesarias para usar Rust.

2. **Crear un nuevo proyecto en Rust**: Estos comandos crean un nuevo proyecto Rust llamado `hello_world`.

   ```sh
   cargo new hello_world
   cd hello_world
   ```

   - `cargo new hello_world`: Crea un nuevo proyecto Rust en el directorio `hello_world`.
   - `cd hello_world`: Cambia al directorio del nuevo proyecto.

3. **Modificar `src/main.rs`**: Este código define el programa `Hello World` en Rust.

   ```rust
   fn main() {
       println!("Hello, world!");
   }
   ```

#### Crear el Dockerfile

1. **Crear un `Dockerfile` en el directorio del proyecto**: Este Dockerfile define cómo se debe construir la imagen Docker de nuestra aplicación Rust.

   ```Dockerfile
   FROM rust:latest

   WORKDIR /usr/src/myapp
   COPY . .

   RUN cargo install --path .

   CMD ["hello_world"]
   ```

   - `FROM rust:latest`: Usa la imagen oficial de Rust como base.
   - `WORKDIR /usr/src/myapp`: Establece el directorio de trabajo en el contenedor.
   - `COPY . .`: Copia todos los archivos del proyecto en el directorio de trabajo.
   - `RUN cargo install --path .`: Compila y instala la aplicación en el contenedor.
   - `CMD ["hello_world"]`: Define el comando por defecto para ejecutar la aplicación.

2. **Construir la imagen Docker**: Este comando construye la imagen Docker usando el Dockerfile.

   ```sh
   docker build -t hello_world .
   ```

   - `docker build`: Construye una nueva imagen Docker.
   - `-t hello_world`: Etiqueta la imagen como `hello_world`.
   - `.`: Indica que el Dockerfile está en el directorio actual.

#### Desplegar en Kubernetes

1. **Crear un archivo de despliegue `deployment.yaml`**: Este archivo define el despliegue de Kubernetes para nuestra aplicación.

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

   - `apiVersion: apps/v1`: Especifica la versión de la API de Kubernetes.
   - `kind: Deployment`: Define el tipo de recurso como un Deployment.
   - `metadata`: Define la metadata del despliegue, como el nombre.
   - `spec`: Define la especificación del despliegue, incluyendo el número de réplicas y los contenedores.

2. **Aplicar el despliegue**: Este comando crea el despliegue en el clúster de Kubernetes.

   ```sh
   kubectl apply -f deployment.yaml
   ```

   - `kubectl apply`: Aplica la configuración a los recursos de Kubernetes.
   - `-f deployment.yaml`: Especifica el archivo de configuración a aplicar.

3. **Crear un servicio para exponer la aplicación**: Este archivo define un servicio de Kubernetes para nuestra aplicación.

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

   - `apiVersion: v1`: Especifica la versión de la API de Kubernetes.
   - `kind: Service`: Define el tipo de recurso como un Service.
   - `metadata`: Define la metadata del servicio, como el nombre.
   - `spec`: Define la especificación del servicio, incluyendo los puertos y el tipo de servicio.

4. **Aplicar el servicio**: Este comando crea el servicio en el clúster de Kubernetes.

   ```sh
   kubectl apply -f service.yaml
   ```

   - `kubectl apply`: Aplica la configuración a los recursos de Kubernetes.
   - `-f service.yaml`: Especifica el archivo de configuración a aplicar.

### 3. Escaneo de Vulnerabilidades con SonarQube (Opcional)

#### Integración con GitHub Actions

1. **Crear un workflow en `.github/workflows/sonarqube.yml`**: Este archivo define un workflow de GitHub Actions para escanear la aplicación con SonarQube.

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

   - `name`: Define el nombre del workflow.
   - `on: [push]`: Especifica que el workflow se ejecutará en cada push al repositorio.
   - `jobs`: Define los trabajos que se ejecutarán en el workflow.
   - `steps`: Define los pasos del trabajo, incluyendo la configuración de JDK y la ejecución del escaneo con SonarQube.

2. **Configurar las credenciales en GitHub**: Agregar `SONAR_TOKEN` en los secretos del repositorio.

### 4. Autoescalado Basado en la Utilización de CPU (Opcional)

#### Configurar el Autoescalado

1. **Crear una Horizontal Pod Autoscaler**: Este comando crea un escalador horizontal de pods basado en la utilización de CPU.

   ```sh
   kubectl autoscale deployment hello-world-deployment --cpu-percent=50 --min=1 --max=10
   ```

   - `kubectl autoscale`: Comando para crear un escalador horizontal de pods.
   - `--cpu-percent=50`: Especifica que el escalador se activará cuando

 la utilización de CPU supere el 50%.
   - `--min=1`: Define el número mínimo de réplicas.
   - `--max=10`: Define el número máximo de réplicas.

2. **Verificar el autoescalado**: Este comando muestra el estado del escalador horizontal de pods.

   ```sh
   kubectl get hpa
   ```

   - `kubectl get hpa`: Muestra los escaladores horizontales de pods en el clúster.

### Observaciones y Pruebas

1. **Acceder a la aplicación**: Obtener la URL del servicio para acceder a la aplicación.

   ```sh
   minikube service hello-world-service --url
   ```

   - `minikube service`: Comando para interactuar con servicios en Minikube.
   - `--url`: Muestra la URL del servicio.

2. **Probar el autoescalado**: Simular carga en la aplicación para verificar el escalado automático.

### Conclusión

Este README proporciona una guía paso a paso para desplegar una aplicación "Hello World" en un clúster de Kubernetes gestionado por Minikube, con opciones adicionales de escaneo de vulnerabilidades y autoescalado. Siguiendo estas instrucciones, deberías poder configurar el entorno, construir y desplegar la aplicación, y opcionalmente, mejorar la seguridad y escalabilidad del despliegue.
