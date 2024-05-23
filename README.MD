# Ejercicio de Server Proxy con dos Contenedores

El siguiente laboratorio tiene por objetivo configurar un servidor proxy (reverse proxy) en un contenedor (simulando un escenario real) y otro contenedor con los recursos que se desean servir por parte del proxy.

## 1.  Para sustituir variables en el archivo de configuracion del nginx:

Se debe crear un volumen en el __docker-compose.yml__, en el servicio que utilizará las variables, ene sa carpeta se debe crearun archivo llamado __default.conf.template__, allí se debe ubicar la directiva de un servidor virtual con las variables de entorno que deseamos.

El volumen debe apuntar a la ruta __/etc/nginx/templates__.

El archivo __/etc/nginx/nginx.conf__ incluye o utiliza los archivos ubicados en el directorio __/etc/nginx/conf.d/__, en este caso es el __default.conf__ el cual es reemplazado y substituido por el archivo ubicado en la ruta __/etc/nginx/template/__ en este caso __default.conf.template__.

Siempre, al iniciarse el servidor, se ejecutan una serie de shell (.sh) como ENTRYPOINTS de los cuales el __/docker-entrypoint.d/20-envsubst-on-templates.sh__ ejecuta la instrucción para sustituir todo lo que esté en dichas carpetas con las variables de entorno. A continuación el log:

```
(20-envsubst-on-templates.sh: Running envsubst on /etc/nginx/templates/default.conf.template to /etc/nginx/conf.d/default.conf)
```

## 2. Para agregar variables de entorno al contenedor sin archivo .env:

- Aunque no es necesario, pero se puede crear el archivo de variables de entorno (.env) en la raiz del proyecto donde está el archivo __docker-compose.yml__.

- En el __docker-compose.yml__, en el servicio que va a necesitar las variables de entorno agregar las variables con el atributo <code>environments</code>:

```
    proxy:
        ...
        environment:
            PROXIED_SERVER_HOSTNAME: ${PROXIED_SERVER_HOSTNAME}
            PROXY_SERVER_PORT: ${PROXY_SERVER_PORT}
```

## Para agregar variables de entorno al contenedor con un archivo .env:

- Crear el archivo de variables de entorno (.env) en la raiz del proyecto donde está el archivo __docker-compose.yml__.

- Se debe colocar en el __docker-compose.yml__, en el servicio que va a necesitar el archivo de variables de entorno lo siguiente:

```
    proxy:
        ...
        env_file:
            - .env
```

En este caso, el servicio __proxy__ (1-server-proxy) es el contenedor que se debe comunicar con el servicio destino __proxied__ (2-server-proxied) para que pueda incorporar en su archivo __/etc/nginx/nginx.conf__, en la directiva <code>proxy_pass</code> del <code>location</code> la ip y puerto del servidor __proxied__. Ahí es donde deseamos reemplazar las variables de entorno para que sea dinámica la configuración del servidor destino desde el servidor proxy.

## Lanzar los contenedores

Para iniciar los contenedores ejecutar el siguiente comando

```
docker compose up -d
```