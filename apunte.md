# Docker 🐳🚢

## ¿Qué son los *containers*?

Son procesos que corren con su propio entorno aislado de los demás.

## Container images

### ¿Qué es una *container image*?

Son paquetes con archivos, binarios, bibliotecas, configuraciones del *container*.

Las imágenes se componen de *layers*, las cuáles podemos reutilizar.

Las imágenes son inmutables, pero podemos crear una nueva agregándo una *layer* encima.

Múltiples *containers* puede usar la misma imagen y modificar archivos sin inteferir entre ellos, ya que aunque compartan *image* siguen siendo independientes y aislados. Esto se debe a que la *writable layer* es única en cada *container*.

### ¿Qué son las *layers*?

Son modificaciones al *filesystem* de la *container image*.

### ¿Qué pasa cuando iniciamos un *container*?

1. Se descargan las *layers*.
2. Se extrae cada *layer* en un directorio en el *host*.
3. Al iniciar un *container* desde una *container image* con todas las *layers* de esta se hace un *union filesystem*.
4. Al iniciar el *container* se le pone como *root* este nuevo *filesystem*.

### ¿Cómo creamos una nueva *container image*? → Dockerfiles

Instrucciones comúnes de `Dockerfile`:

- `ARG <arg>` - Especifica los argumentos que usará el `FROM` y solo esta instrucción puede ir antes de este.
- `FROM <image>` - Especifica la imagen base sobre la que construir agregando layers. Para agregarle un nombre al *build stage* usamos `FROM ... AS <NAME>`.
- `WORKDIR <path>` - Ruta en la imagen donde se ejecutarán comandos y ubicarán archivos.
- `COPY <host-path> <image-path>` - Indica que se debe copiar el archivo en `host-path` a `image-path` en la *container image*. Si queremos copiar de otra *build stage* usamos `COPY --from=<NAME> <src> <dest>`.
- `RUN <command>` - Indica al builder que ejecute el siguiente comando.
- `ENV <name> <value>` - Setea una variable de entorno en el container. e.g.:

```dockerfile
ENV app=bar
ENV run=./$app #Si querés usar una variable declarada antes usás '$' como prefijo
```

- `EXPOSE <port-number>` - Setea el puerto por el que será accesible la imagen.
- `USER <user-or-uid>` - Setea el usuario para las siguientes instrucciones.
- `CMD ["<command>", "<arg1>"]` - Setea el comando por defecto que un contenedor que use esta imagen va a correr.

Instrucciones de este archivo se ejecutan en orden.

Para comentarios usar '#'.

Los pasos comúnes de un `Dockerfile` son:

1. Determine your base image
2. Install application dependencies
3. Copy in any relevant source code and/or binaries
4. Configure the final image.

Al igual que con `npm` podemos correr `npm init --yes` para que nos genere un `package.json` por defecto, acá corriendo `docker init` nos crea un `Dockerfile`, `compose.yml` y un `.dockerignore`.

Para correr ese *container* con *Docker Compose* usamos `docker compose up` y para apagarlo `docker compose down`

### *Container image build*

Parado en la carpeta que contiene el `Dockerfile` ejecutándo:

```bash
docker build .
```

Si queremos que la imagen tenga un nombre en vez de un ID podemos usar un *tag*.

### *Container image tag*

Sintaxis de un tag:

```docker
[HOST[:PORT_NUMBER]/]PATH[:TAG]
```

Donde:

- HOST: Es el hostname del *docker registry* donde está la imagen. Por defecto `docker.io`.
- PORT_NUMBER: Es el *docker registry port* si lo hay especificado.
- PATH: Path de la imagen. En *Docker hub* el formato es `[NAMESPACE/]REPOSITORY` donde namespace es el nombre de la organización o del usuario. Por defecto es `library`.
- TAG: Human-readable identifier. Por defecto se usa `latest`.

#### ¿Cómo asignar un *tag*?

- Durante build:

```bash
docker build -t my-username/my-image .
```

- Si ya hiciste el `build` podés usar:

```bash
docker image tag my-username/my-image another-username/another-image:v1
```

### Pushing to *Docker registry*

Tras ejecutar el `build` podemos subirla al *Docker registry* con:

```bash
docker push my-username/my-image
```

Requiere que estés autenticado, para eso usar `docker login`.

### Ver history de cómo se hizo la imagen

```bash
docker image history YOUR_DOCKER_USERNAME/YOUR_DOCKER_IMAGE_TAG
```

### Parar un *container*

Usamos `docker stop <container-id>`.

### Eliminar un *container*

- Si no nos sabemos el *container id* podemos verlo ejecutando `docker ps`.
- Paramos el *container* con `docker stop <container-id>`.
- Ya con este parado ejecutamos `docker rm <container-id>`
- Si querés pararlo y borrarlo de una usá `docker -fr <container id>`


### Comunicación vía puertos

Como los *containers* corren en un entorno aislado, no podemos acceder a ellos directamente sino que debemos hacerlo mediante puertos.

#### Publishing ports

Se realiza durante la creación del *container* usando el flag `-p` o `--publish` con la siguiente sintáxis:

```bash
docker run -d -p HOST_PORT:CONTAINER_PORT nginx
```

- HOST_PORT: Es el puerto de la máquina host del *container* por donde nos queremos comunicar.
- CONTAINER_PORT: Es el puerto dentro del *container* que este escucha.

Si sólo especificás un puerto, ese es el del *container* y *Docker* elige uno para el host, el cuál podemos ver con `docker ps`.

Los puertos en la instrucción `EXPOSE` del `Dockerfile` no son expuestos por default y requieren que usemos el flag `-P` o `--publish-all` con el `docker run`.

Cualquier tráfico que llegue a la máquina del host puede acceder a ese puerto, y por lo tanto, comunicarse con contenedor. No exponer bases de datos.

### Overriding containers config

Si necesitás correr otra instancia de un *container*, pero necesitás hacerlo con otra configuración, ya sea porque necesitás mapearlo a otro puerto para evitar conflictos o porque querés asignarle más recursos, etc. Es necesario poder pisar la configuración del `Dockerfile` y para eso podemos usar los flags del comando `docker run`.

Algunas flags útiles:

- `-p HOST_PORT:CONTAINER_PORT`.
- `-e foo=bar`: Setea variables de entorno, en este caso a la variable `foo` le asigna el valor `bar`. Ejemplo: `docker run -e foo=bar postgres env`. Si tenés hacer muchas asignaciones te conviene crear un nuevo archivo `.env` y pasarlo con --env-file .env por ejemplo : `docker run --env-file .env postgres env`.
- `--memory="<amount>"` y `--cpus=<amount>`: Sirven para especificar los recursos asignados al contenedor. Ejemplo: `docker run -e POSTGRES_PASSWORD=secret --memory="512m" --cpus="0.5" postgres`.
- `--network <myNetworkName>`: Permite conectar el contenedor a una *custom network*.
- `-h <hostName>`: Sirve para cambiar el host.
- `-d` o `--detached`: Indica que el *container* se corra en el *background* y te devuelva el control de la terminal en vez de imprimir los logs en ella.
Para visualizar recursos asignados usar comando `docker stats`.

## Avanzado

### Aprovechando el cache para `builds` más rápidos

- Al hacer `docker build` se ejecutan las instrucciones del `Dockerfile`.
- Cada instrucción te genera una nueva *layer*.
- *Docker* intentará reutilizar *layers* previas almacenadas en el cache para hacerlo más rápido, pero hay que tener en cuenta que algunas modificaciones al `Dockerfile` invalidan en el cache las *layers* afectadas.
- Entre las acciones invalidantes se encuentran:
        - Modificaciones al comando `RUN` invalidan esa *layer*.
        - Modificaciones a los archivos o directorios parámetros de `COPY` y `ADD`.
        - Una vez se invalida una *layer* todas las siguientes también.
- Para evitar reinstalar las dependencias con cada `build`, si las tenemos declaradas en un `package.json` por ser una *Node-based app* primero hacer el `COPY` de ese archivo, luego instalar las dependencias `RUN npm install` y luego hacer el `COPY` de lo demás.

### Usando el archivo .dockerignore

En la misma carpeta del `Dockerfile` creamos un `.dockerignore` con el siguiente contenido: `node_modules`.

### Multi-stage builds

Correr diferentes partes de la secuencia de `build` en distintos entornos de forma concurrente, separando *build environment* de *runtime environment*.

- Vamos a tener múltiples `FROM` en el `Dockerfile`, cada uno puede usar una base distinta. Debemos ponerles nombre con `AS <buildStageName>`, sino se nombran separándose por `FROM` y en orden iniciando desde 0.
- Para correrlos:
        - Usamos `docker build` si queremos correr todas las *build stages*.
        - Usamos `docker build --target <buildStageName> -t <appsTag>`. Si tenemos *BuildKit* activado sólo se construirán las *build stages* de las que dependa `<buildStageName>`, sino se contruirán todas.
- Podés crear una nueva *build stage* usando otra previa como base.
- También usar imágenes externas como parámetro del `--from=`, ya sea el nombre de una imagen local, un tag de una imagen local o en un *Docker registry* por ejemplo:

```Docker
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

Para activar *BuildKit* usamos como prefijo del `docker build` a `DOCKER_BUILDKIT=1 docker build`.

Conviene tener distintas *build stages* para no mezclar las de desarrollo con las de producción. De este modo una vez ya tenemos todo compilado, testeado y listo podemos en otra *build stage* tomar los ejecutables de esa dirección. Así también reducimos el tamaño del archivo final de producción.

### Controlled networks

Al correr un *container* este se conecta a una *network* especial llamada *bridge network* la cuál permite que los *containers* se comuniquen entre sí sin romper si aislamiento.

Podemos utilizar una *network* nueva definida por nosotros para facilitar la comunicación entre contenedores.

Con el comando `docker network inspect` vemos qué contenedor está conectado a cuál *network*.

#### Crear una *custom network*

- Para crear una *network* usamos: `docker network create <myNetworkName>` y chequeamos que esté ejecutando `docker network ls`.
- Para conectarse a una *network* al correr un *container* usamos el flag `--network <myNetworkName>`.
e.g.:
`docker run -d -e POSTGRES_PASSWORD=secret -p 5434:5432 --network mynetwork postgres`.

Si usamos *Docker Compose* no hace falta que definamos una *network* ya que por defecto este crea una y utiliza los nombres de los *containers/services* como *network alias*.

#### Diferencias entre *default bridge* y *custom network*

- Los *containers* conectados a *default bridge* se pueden comunicar pero solo por IP address. Mientras que en *custom networks* se pueden comunicar por alias o nombre.
- Un *container* conectado a *default bridge* está expuesto a que cualquier otro *container* no relacionado se comunique con él vía IP. Mientras que en una *custom network* solo se permite la comunicación dentro de ella.

### *Container volumes*: Persistencia de datos del *container*

Al borrar un *container* se borran con él todos sus datos, también al apagarlo. En el caso de tener un base de datos en uno, querríamos que al reiniciarla esta no haya perdido los datos.

Los *container volumes* son un mecanismo de almacenamiento persistente más allá del ciclo de vida de un *container*.

#### ¿Cómo crear un volumen?

Para crear un volumen ejecutamos  `docker volume create <volumeName>`, y al iniciar un *container* le montamos ese volúmen mediante el flag `-v <volumeName:/carpeta>`.
        - Si no existe el volumen `<volumeName>`, *Docker* lo crea.
        - Todos los archivos que se escriban a `/carpeta` se guardarán en el volumen y si borrás un *container* e iniciás otro con ese volumen los datos estarán ahí.
        - Se puede montar el mismo volumen en múltiples *containers*.

#### Manejo de volúmenes

- `docker volume ls`: Lista los volúmenes.
- `docker volume rm <volume-name-or-id>`: Borra el volumen y solo funciona si no está montado a ningún contenedor.
- `docker volume prune`: Borra todos los volúmenes no usados/montados.
- `docker volume inspect <volume-name-or-id>`: Nos permite ver información del volumen en formato JSON, como "Mountpoint", que es donde se encuentran los datos del volumen en el disco del host.

## Compartir archivos locales con *container*

Si necesitás que un *container* acceda a información en el host pero que no se almacene en el *container* por razones de seguridad, entonces podés usar los métodos para compartir archivos entre host y *container*:

- Volumes ← Si querés que las modificaciones de datos realizadas en el *container* persistan.
- Mounts ← Si hay archivos del *host system* que querés compartir con el *container*. Permite especificar *host location*, también `type=` permite elegir entre *volume* y *bind*, con `src=` y `target=` indicamos fuente en *host* y destino en *container*.

E.g.: `docker run -it --mount type=bind,src=.,target=/src ubuntu`

Tenemos los flags `-v`/`--volume` y `--mount` del comando `docker run`. El `--mount` es el recomendado por ser más granular. Tener en cuenta que de no existir la carpeta `-v` la crea pero  `--mount` no, tampoco sucede con *Docker Compose*.

Ejemplo dando acceso al *container* a `/HOST/PATH`: `docker run -v /HOST/PATH:/CONTAINER/PATH -it nginx`

Ejemplo con `--mount`
`docker run --mount type=bind,source=/HOST/PATH,target=/CONTAINER/PATH,readonly nginx`

En *Docker Compose* debemos indicar los volúmenes con `volumes:` a nivel general, y en la configuración de cada *container/service* ponemos en `volumes:` en *mountpoint*. Por ejemplo:

```yml
services:
  app:
    # The app service definition
  mysql:
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql

volumes:
  todo-mysql-data:
```

### Restringiendo permisos a archivos compartidos

Al final del *container directory* podemos usar flags para restringir los permisos que tendrá el *container* sobre el *host directory*.

Estos pueden ser `:ro` para *read-only*, `:rw` para *read-write*, en el cuál los cambios realizados por el *container* se ven reflejados en el *host directory*.

Ejemplo:
`docker run -v HOST-DIRECTORY:/CONTAINER-DIRECTORY:rw nginx`

## Aplicaciones *multi-container*

Buena práctica: "Cada contenedor debe realizar una única cosa y hacerla bien"

Para esto nos ayudamos de *Docker Compose* que:

- Nos permite definir toda la aplicación *multi-container* en un sólo archivo `compose.yaml`, conteniendo las configuraciones para cada *container*. 
- Permite evitar tener que correr un `docker run` por cada *container*.
- Facilita el manejo de variables de entorno, *networks*, volúmenes persistentes, etc.

### Sin Docker Compose

- Primero hacemos el `docker build` de cada *container*.
- Luego creamos una *network* con `docker network create <nombreNetwork>`.
- Corremos `docker run` en cada *container* pero conectándolo a la *network* con el flag `--network <nombreNetwork>` y también les damos un alias en esta con `--network-alias <alias>`.

### Con Docker Compose

- En el *root* del directorio del proyecto creamos un archivo `compose.yml`.
- Seguimos la siguiente estructura:

```yml
services:
  container1:
    ...
  container2:
    ...
  containern:
    ...
```

- Lo que pongamos como nombre, en este caso containeri será el network alias de ese container

- En el cuerpo de cada *container* completamos con su configuración. [Ejemplo de un Docker Compose *multi-container*](./nginx-node-redis/compose.yml)

## Compartir imágenes vía Docker Registry

- Crear un repositiorio en ([*Docker Hub*](https://hub.docker.com/?_gl=1*12686r3*_gcl_au*MTg4MDg5NTYzOC4xNzc4NzcxNTQz*_ga*MTk5NTQ0OTYyOC4xNzc4NzcxNTQ0*_ga_XJWPQMJYHQ*czE3Nzg4NjQ3MTMkbzExJGcxJHQxNzc4ODY2NDE0JGo1OSRsMCRoMA..))
- Al crearlo *Docker Hub* nos va a sugerir un comando para pushear la imagen al repositorio. 
- Copiar dicho comando en la terminal pero asignándo un *tag name*.
- Si al ejecutarlo nos tira un error que contenga el siguiente mensaje "...An image does not exist locally with the tag: `docker/<repositoryName>`". Debemos ver si al ejecutar `docker image ls` la imagen que queremos subir no tiene el nombre `docker/<repositoryName>`. Para solucionarlo hacemos lo siguiente:
        - Nos loggeamos en *Docker Hub* con nuestro *Docker ID*: `docker login <YOUR-USER-NAME>`.
        - Cambiar el nombre de la imagen: `docker tag <currentTag> <YOUR-USER-NAME/currentTag>`, donde `currentTag` debe ser igual a `repositoryName`.
        - Volver a correr el comando de `docker push` que dio el error.

Ahora podemos descargar esta imagen en cualquier otra computadora y correrla sin problemas.

## Command breakdowns

Si tenemos que ejecutar un comando muy largo en terminal, podemos separarlo con \ y seguir en siguiente línea. Ejemplo:

```bash
docker run -dp 127.0.0.1:3000:3000 \
    -w /app --mount type=bind,src=.,target=/app \
    node:24-alpine \
    sh -c "npm install && npm run dev"
```

## Utilizar un *container* como watch montando el código fuente en él

Podés crear un *container* al cuál le pasás el directorio que querés ejecutar con un *bind* y luego corrés en él el script con `nodemon`.

Luego para ver sus logs corrés `docker logs -f <container-id>`.

