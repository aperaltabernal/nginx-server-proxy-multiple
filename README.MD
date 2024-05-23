# Ejercicio de Server Proxy con dos Contenedores

El siguiente laboratorio tiene por objetivo configurar un servidor proxy (*reverse proxy*) en un contenedor, simulando un escenario real, y otro contenedor con los recursos que se desean servir por parte del proxy (*server proxied*).

## 1. Explicación configuración de docker-compose.yml para utilizar variables de entorno en el archivo de configuracion del nginx:

Se debe crear un __volumen__ en el __docker-compose.yml__, en el servicio (en *docker-compose.yml: services: proxy:*) que utilizará las variables, en este caso es el servidor proxy (*reverse proxy*) el cual necesita invocar el otro servidor que se encuentra oculto al mundo exterior (*por ello no se exponen los puertos de este servicio ya que se podría comunicar con el contenedor del servidor proxy el cual se encuentra en la misma red*), quien tiene los recursos a ser servidos. En ese __volumen__ se debe crear un archivo llamado __default.conf.template__, allí se ubica la directiva de un servidor virtual (<code>server {...}</code>) con las variables de entorno que deseamos.

En este ejemplo las variables que deseamos reemplazar son <code>${PROXY_SERVER_PORT}</code>, <code>${PROXIED_SERVER_HOSTNAME}</code> y <code>${PROXIED_SERVER_PORT}</code>, las cuales representan el puerto para exponer el __reverse proxy server__ actual, el *host* donde se ubica el __server proxied__ con los recursos a ser servidos y su puerto respectivamente. Esto es necesario ya que para poder pasar el nombre del host, que en este caso es el nombre del contenedor y el puerto, de una manera más parametrizable, se requiere que este archivo tome los cambios al momento de iniciarse el servidor, en caso contrario tendríamos que ingresar al servidor, sustituir la ip y el host que deseamos y luego recargar el servicio del nginx.

A continuación, el contenido del archivo __default.conf.template__:

```
server {
    listen ${PROXY_SERVER_PORT};
    location / {
        proxy_pass http://${PROXIED_SERVER_HOSTNAME}:${PROXIED_SERVER_PORT};
    }

    location ~ \.(jpg|png)$ {
        root /data/images;
    }
}
```

El volumen debe apuntar a la ruta __/etc/nginx/templates__.

```
    services:
        ...
        proxy:
            ...
            volumes:
            - ./1-server-proxy/templates:/etc/nginx/templates
```

El archivo __/etc/nginx/nginx.conf__ el cual se encuentra dentro del servidor de *nginx* incluye o utiliza los archivos ubicados en el directorio __/etc/nginx/conf.d/__, en este caso es el __default.conf__ el cual es reemplazado y sustituido por el archivo ubicado en la ruta __/etc/nginx/template/__ en este caso __default.conf.template__.

Siempre, al iniciarse el servidor, se ejecutan una serie de *shell commands* (.sh) como __ENTRYPOINTS__ de los cuales el __/docker-entrypoint.d/20-envsubst-on-templates.sh__ ejecuta la instrucción para sustituir todo lo que esté en dichas carpetas con las __variables de entorno__ y luego de reemplazar el archivo lo ubica en la carpeta __/etc/nginx/conf.d/__. A continuación el log:

```
(20-envsubst-on-templates.sh: Running envsubst on /etc/nginx/templates/default.conf.template to /etc/nginx/conf.d/default.conf)
```

## (Opción 1) Agregar variables de entorno al contenedor sin archivo .env:

- Aunque no es necesario, se puede crear el archivo de variables de entorno (.env) en la raiz del proyecto donde está el archivo __docker-compose.yml__.

- En el __docker-compose.yml__, en el servicio que va a necesitar las variables de entorno, agregar las variables con el atributo <code>environment</code>:

```
    services:
        ...
        proxy:
            ...
            environment:
                PROXIED_SERVER_HOSTNAME: ${PROXIED_SERVER_HOSTNAME}
                PROXY_SERVER_PORT: ${PROXY_SERVER_PORT}
```

## (Opción 2) Agregar variables de entorno al contenedor con un archivo .env:

- Crear el archivo de variables de entorno (.env) en la raiz del proyecto donde está el archivo __docker-compose.yml__.

- Se debe colocar dentro del __docker-compose.yml__, en el servicio que va a necesitar el archivo de variables de entorno lo siguiente, donde __.env__ es el archivo creado por nosotros para agregar las variables de entorno:

```
    services:
        ...
        proxy:
            ...
            env_file:
                - .env
```

En este caso, el servicio __proxy__ (*1-server-proxy*) es el contenedor que se debe comunicar con el servicio destino __proxied__ (*2-server-proxied*). Para que el archivo __/etc/nginx/conf.d/default.conf__, en su directiva <code>proxy_pass</code> del <code>location</code> se pueda reemplazar la ip y puerto del servidor __proxied__ realizamos este procedimiento. Ahí es donde deseamos reemplazar las variables de entorno para que sea dinámica la configuración del servidor destino desde el servidor proxy. Al agregar las variables de entorno al servicio, estas quedan establecidas en las variables del sistema del contenedor las cuales se pueden visualizar, ingresando primero a la consola del contenedor mediante la instrucción:
```
docker exec -it <id_contenedor> /sh
```

y ejecutando el siguiente comando:
```
    env
```
Esto mostrará las variables de entorno del contenedor entre ellas las que se pasaron al momento de ejecutarlo.

## 2. Lanzar los contenedores

Para iniciar los contenedores ejecutar el siguiente comando

```
docker compose up -d
```