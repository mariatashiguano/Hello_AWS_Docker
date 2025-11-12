# Pipeline CI/CD: Docker, GitHub Actions y AWS EC2

[![Estado del Deploy](https://github.com/mariatashiguano/Hello_AWS_Docker/actions/workflows/deploy.yml/badge.svg)](https://github.com/mariatashiguano/Hello_AWS_Docker/actions)

## Resumen del Proyecto

Este repositorio implementa un pipeline de Integración Continua y Despliegue Continuo (CI/CD) para una aplicación web Nginx. El pipeline automatiza la construcción de imágenes Docker y su despliegue en una instancia de AWS EC2.

Este flujo de trabajo se activa automáticamente tras cada `push` a la rama `main`. La instancia EC2 en este proyecto sirve simultáneamente como entorno de desarrollo y servidor de despliegue.

---

## Resumen Tecnológico

* **GitHub Actions**: Orquestador del pipeline de CI/CD.
* **Docker & Docker Hub**: Contenedorización de la aplicación y registro de imágenes.
* **AWS EC2 (Amazon Linux 2023)**: Servidor host para la ejecución del contenedor.
* **Nginx**: Servidor web base.
* **SSH**: Protocolo de comunicación segura entre GitHub Actions y el host EC2.

---

## Arquitectura del Pipeline

El workflow, definido en `.github/workflows/deploy.yml`, se compone de dos trabajos secuenciales:

### 1. Job: `build-and-push`

Este trabajo se ejecuta en un *runner* hospedado por GitHub y gestiona la creación de la imagen.

1.  **Checkout**: Clona el código fuente del repositorio.
2.  **Login to Docker Hub**: Se autentica en Docker Hub utilizando los secretos `DOCKERHUB_USERNAME` y `DOCKERHUB_TOKEN`.
3.  **Build and Push**: Construye la imagen de Docker basada en el `Dockerfile` local y la publica en el registro de Docker Hub con la etiqueta `latest`.

### 2. Job: `deploy`

Este trabajo se ejecuta tras la finalización exitosa de `build-and-push` y gestiona la actualización del servicio en el servidor.

1.  **Deploy to EC2**:
2.  Utiliza la acción `appleboy/ssh-action` para establecer una conexión SSH segura con la instancia EC2, usando los secretos `EC2_HOST`, `EC2_USER` y `EC2_SSH_KEY`.
3.  Tras la autenticación, se ejecuta el siguiente script en el servidor remoto:
    * `docker pull ...`: Descarga la imagen más reciente (`:latest`) desde Docker Hub.
    * `docker stop ...`: Detiene el contenedor actualmente en ejecución (si existe).
    * `docker rm ...`: Elimina la instancia del contenedor anterior.
    * `docker run ...`: Inicia un nuevo contenedor basado en la imagen actualizada, exponiendo el puerto 80.

---

## Configuración de Secretos

La correcta ejecución del pipeline requiere la configuración de las siguientes variables secretas en el repositorio (`Settings > Secrets and variables > Actions`):

* `DOCKERHUB_USERNAME`: El nombre de usuario de Docker Hub.
* `DOCKERHUB_TOKEN`: Un token de acceso generado en Docker Hub con permisos de lectura/escritura.
* `EC2_HOST`: La dirección IP pública o DNS de la instancia EC2.
* `EC2_USER`: El nombre de usuario para la conexión SSH (ej. `ec2-user`).
* `EC2_SSH_KEY`: La clave SSH privada en formato `.pem` para la autenticación en la instancia.

---

## Estructura del Repositorio

/ │ ├── .github/workflows/ │ └── deploy.yml # Definición del pipeline de CI/CD │ ├── Dockerfile # Instrucciones para la imagen de Nginx │ └── index.html # Archivo de la aplicación web

---

## Proceso de Despliegue

1.  Modificar los archivos del proyecto (p.ej., `index.html`) en el entorno de desarrollo (la instancia EC2).
2.  Confirmar y subir los cambios al repositorio:
    ```bash
    git add .
    git commit -m "Mensaje descriptivo del commit"
    git push origin main
    ```
3.  El `push` activará automáticamente el workflow de GitHub Actions.
4.  Es posible monitorear el progreso en la pestaña "Actions" del repositorio.
5.  Al finalizar el workflow, los cambios estarán visibles en la IP pública del servidor EC2.
