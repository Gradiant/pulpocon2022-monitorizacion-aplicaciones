# Monitorizando nuestras aplicaciones
## PulpoCon 2022

## Requisitos

- Docker versión > 20.10
- Docker compose
- Preferible un entorno Linux


## Monitorizando logs con Kibana + ElasticSearch + Logstash + Filebeat

*Modo rápido*
Puedes lanzar todo de una vez, pero queremos ~~rellenar tiempo~~ ver con calma lo que estamos haciendo!

```bash
cd elastic-stack
docker compose -p pulpocon up -d
```

Si tienes compose V1 (el clásico script Python) en lugar del plugin de compose V2 las instrucciones serán con 
docker-compose en lugar de docker compose.

*Ejercicios paso a paso*

### Ejercicio 1: fichero de logs

```bash
# Lanzamos Kibana
docker compose -p pulpocon up -d kibana

# Lanzamos ElasticSearch
docker compose -p pulpocon up -d elasticsearch

# Lanzamos Logstash
docker compose -p pulpocon up -d logstash

# Lanzamos Filebeat
docker compose -p pulpocon up -d filebeat
```

Filebeat se ha configurado para escuchar todos los ficheros `*.log` que aparezcan en la carpeta `logs` del host. Puedes enviar el fichero de log `log_example.log`, línea a línea con una pausa de medio segundo entre líneas del siguiente modo:
```bash
while read line; do echo $line; sleep 0.5; done < log_example.log > ../logs/test.log
```

Mientras se sube el fichero, entra en [Kibana](http://localhost:5601) e intenta configurar encontrar los logs. _Tip: Discover_

Una vez visualizados los logs, trata de crear alguna visualización. _Tip: Dashboard_

### Ejercicio 2: docker logs

Para este ejercicio es necesario instalar el plugin de docker de elastic

```bash
docker plugin install elastic/elastic-logging-plugin:8.4.1 --alias elastic --grant-all-permissions

# Comprueba la instalación
docker plugin ls
```

#### Servicio generador de logs

Lanza el servicio que genera logs para comprobar el funcionamiento del plugin de docker. 
El formato de logs es [Apache combined](http://fileformats.archiveteam.org/wiki/Combined_Log_Format).

```bash
docker compose -p pulpocon up -d log-generator
```

Al poco aparecerán nuevos logs en Elastic con etiquetas como container.name=pulpocon-log-generator-1