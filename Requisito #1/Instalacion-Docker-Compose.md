# Levantar Clúster de MongoDB con `Docker-compose.yml`

Para levantar el clúster MongoDB usando el archivo `docker-compose.yml`, se debe hacer lo siguiente:

- Descarga la imagen de MongoDB: Si aún no lo has hecho, descarga la imagen de Docker de MongoDB ejecutando en tu terminal:

```
docker pull mongo:latest
```

<p align="center">
<img src="https://github.com/IAlfaro404/lab2-BDA-MongoDB/blob/main/Requisito%20%231/SalidaTerminalInstalacionDockerCompose2.png?raw=true" alt="Este debe ser el resultado, 1." style="display: block; margin: 0 auto;"/></p>

- Navega hasta el directorio desde el terminal, y donde está archivo `docker-compose.yml`:

-  Levanta los contenedores: Ejecuta lo siguiente para iniciar todos los servicios definidos en el archivo en modo detached (segundo plano):

```
docker-compose up -d
```
Una vez que los contenedores estén en ejecución, Se debe proceder con la configuración del Replica Set y la creación de la base de datos en `Instalación-Manual.md` desde el paso #2.

<p align="center">
<img src="https://github.com/IAlfaro404/lab2-BDA-MongoDB/blob/main/Requisito%20%231/SalidaTerminalInstalacionDockerCompose1.png?raw=true" alt="Este debe ser el resultado, 2." style="display: block; margin: 0 auto;"/></p>
  
---

## Explicación de `Docker-compose.yml`

Este documento de Docker Compose (`docker-compose.yml`) está diseñado para desplegar un clúster de MongoDB con un Replica Set de tres nodos. Un Replica Set en MongoDB es un grupo de procesos mongod que mantienen el mismo conjunto de datos, proporcionando alta disponibilidad y redundancia de datos.

El archivo se divide en varias secciones clave:

- `services`: Define los contenedores individuales que se ejecutarán: `mongo1`, `mongo2` y `mongo3`, cada uno representando un nodo de MongoDB.

- `networks`: Define las redes personalizadas que los servicios pueden usar para comunicarse entre sí.

- `volumes`: Define los volúmenes de Docker que se utilizarán para persistir los datos de los contenedores, asegurando que los datos no se pierdan cuando los contenedores se detienen o se eliminan.

### Sección `services` (Contenedores MongoDB):

Cada servicio (`mongo1`, `mongo2`, `mongo3`) tiene una configuración similar pero con algunas diferencias clave:

- `image: mongo:latest`: Especifica la imagen de Docker que se utilizará para crear el contenedor. En este caso, la última versión oficial de MongoDB.

- `container_name: mongo1` (o `mongo2`, `mongo3`): Asigna un nombre específico al contenedor.

- `ports: - "3001:27017"` (o `3002`, `3003`): Mapea un puerto del host (máquina local) al puerto `27017` dentro del contenedor. Esto permite acceder a cada instancia de MongoDB desde fuera del entorno Docker.

    - mongo1 usa el puerto `3001`.

    - mongo2 usa el puerto `3002`.

    - mongo3 usa el puerto `3003`.

- `networks: - mongodb-lab2-grupo2`: Conecta cada contenedor a la red personalizada mongodb-lab2-grupo2. Esto permite que los contenedores se comuniquen entre sí utilizando sus nombres de servicio (por ejemplo, mongo1 puede comunicarse con mongo2 usando mongo2:27017).

- `command: mongod --replSet rs`: Este es el comando que se ejecuta cuando el contenedor se inicia. `mongod` es el proceso del demonio de MongoDB, y `--replSet rs` lo inicializa como parte de un Replica Set llamado `rs`. 

- `healthcheck`: Define cómo Docker puede verificar la salud de cada servicio.

    - `test: ["CMD", "mongosh", "--eval", "db.runCommand({ ping: 1 }).ok"]`: Ejecuta un comando mongosh dentro del contenedor para verificar si la base de datos responde a un comando ping. Si ok es 1, el servicio se considera saludable.

    - `interval: 10s`: Frecuencia con la que se ejecuta el chequeo de salud.

    - `timeout: 5s`: Tiempo máximo para que el chequeo de salud se complete.

    - `retries: 5`: Número de reintentos antes de que el servicio se considere no saludable.

- `depends_on` (para `mongo2` y `mongo3`):

    - `mongo1: condition: service_healthy`: Indica que `mongo2` y `mongo3` no se iniciarán hasta que el servicio mongo1 esté en estado healthy. Esto es importante para asegurar que el nodo primario potencial esté listo antes de que los secundarios intenten unirse.

    - `volumes: - mongo1-data:/data/db` (o `mongo2-data`, `mongo3-data`): Monta un volumen con nombre (`mongo1-data`, etc.) en el directorio `/data/db` dentro del contenedor. Este es el directorio predeterminado donde MongoDB almacena sus datos. El uso de volúmenes con nombre asegura que los datos persistan incluso si los contenedores se eliminan y se vuelven a crear.

### Sección `networks`:

- `mongodb-lab2-grupo2:`: Define la red personalizada.

    - `driver: bridge`: Especifica el tipo de controlador de red. bridge es el tipo predeterminado y más común para redes personalizadas, creando una red aislada para los contenedores.

### Sección `volumes`:

- `mongo1-data:`, `mongo2-data:`, `mongo3-data:`: Declara los volúmenes con nombre que se utilizarán. Docker gestionará la creación y el almacenamiento de estos volúmenes en el sistema de archivos del host, garantizando la persistencia de los datos de MongoDB para cada nodo.
