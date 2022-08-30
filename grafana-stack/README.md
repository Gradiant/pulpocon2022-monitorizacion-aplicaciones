# Monitorizando nuestras aplicaciones
## PulpoCon 2022

## Requisitos

- Docker versión > 20.10
- Docker compose
- Preferible un entorno Linux

## Monitorizando logs con Grafana + Loki + Promtail

*Modo rápido*
Puedes lanzar todo de una vez, pero queremos ~~rellenar tiempo~~ ver con calma lo que estamos haciendo!

```bash
cd grafana-stack
docker compose -p pulpocon up -d
```

Si tienes compose V1 (el clásico script Python) en lugar del plugin de compose V2 las instrucciones serán con 
docker-compose en lugar de docker compose.

*Ejercicios paso a paso*

### Ejercicio 1: fichero de logs

#### Grafana

Si ya lo lanzaste en el ejercicio de Prometheus (y está en ejecución) puedes saltarte este paso. 

ACCESO:
http://localhost:3000/
- user: pulpocon
- pass: rules

```bash
# Lanzamos Grafana (si ya lo lanzamos en el ejercicio 1 se puede omitir)
docker compose -p pulpocon up -d grafana
```
#### Loki

[loki](https://grafana.com/oss/loki/)

```bash
# Lanzamos Loki
docker compose -p pulpocon up -d loki
```

#### Promtail

[promtail](https://grafana.com/docs/loki/latest/clients/promtail/)

```bash
# Lanzamos Promtail
docker compose -p pulpocon up -d promtail
```

Es necesario montar todos los ficheros que debe ingestar. En este ejercicio necesitamos montar ../logs y configurar 
Promtail para escuchar todos los ficheros `*.log` que aparezcan en la carpeta `logs` del host. (promtail1.yml)

Usaremos un [log de ejemplo](https://github.com/logpai/loghub/tree/master/Apache) de Apache.

Puedes cargar el fichero de log `log_example.log` directamente en la carpeta ../logs o enviar su contenido a esa 
carpeta, línea a línea con una pausa de medio segundo entre líneas, del siguiente modo:
```bash
while read line; do echo $line; sleep 0.5; done < log_example.log > ../logs/test.log
```

Mientras se sube el fichero, entra en [Grafana](http://localhost:3000) e intenta configurar Loki. _Tip: Data sources_

Una vez configurado Loki, trata de explorar los logs.

Ahora prueba a hacer alguna gráfica de conteo de los logs, p. ej:
```promql
count_over_time({job="pulpolog"}[1m])
```
Prueba a filtrar por tipo de log (error, notice,...)

### Ejercicio 2: docker plugin

Para este ejercicio es necesario instalar el plugin de docker de loki

```bash
docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions

# Comprueba la instalación
docker plugin ls
```

#### Servicio generador de logs

Lanza el servicio que genera logs para comprobar el funcionamiento del plugin de docker. 
El formato de logs es [Apache combined](http://fileformats.archiveteam.org/wiki/Combined_Log_Format).
 
Puedes probar también a generarlos en formato json y ver las diferencias.

```bash
docker compose -p pulpocon up -d log-generator
```

Al poco aparecerán nuevos logs en Loki, con etiquetas como compose_service="log-generator".
```promql
sum by(statuscode) (rate({compose_service="log-generator"} | pattern `<hostip> <clientid> <username> [<ts>] "<request>" 
<statuscode> <bytes> "<referer>" "<useragent>"` [1m]))
```

### Ejercicio 3: docker logs

Para agregar los logs de los contenedores docker podemos definir un trabajo en Promtail que use el socket de docker 
para obtener los logs. De esta manera tendremos etiquetas y filtros con la información de los contenedores.
- Para esto promtail necesita acceso de lectura al socket (/var/run/docker.sock).
- promtail2.yml

```bash
docker restart pulpocon-promtail-1
```

Explora los logs que se añaden y trata de hacer alguna gráfica. Pista, para los logs de loki, prometheus, etc logfmt 
permite extraer las etiquetas fácilmente.

### Ejercicio 4: system logs

Para agregar los logs de un sistema con systemd en Loki podemos definir un trabajo en Promtail que use scraper de 
journal. De esta manera tendremos etiquetas y filtros con la información de los servicios de systemd.
- Para esto promtail necesita acceso de lectura a /var/log
- promtail3.yml

Es necesario recrear el contenedor de promtail:

```bash
docker compose  compose -p pulpocon up -d promtail
```

Explora los logs que se añaden.

