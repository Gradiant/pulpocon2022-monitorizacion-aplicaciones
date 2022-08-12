# Monitorizando nuestras aplicaciones
## PulpoCon 2022

## Kibana + ElasticSearch + Logstash + Filebeat
```bash
# Lanzamos Kibana
docker-compose up -d kibana

# Lanzamos Loki
docker-compose up -d loki

# Lanzamos Promtail
docker-compose up -d promtail
```

Tip: Si quieres puedes lanzar todo de una sola vez, pero queríamos alagar el taller :wink:
```bash
docker-compose up -d
```

Filebeat se ha configurado para escuchar todos los ficheros `*.log` que aparezcan en la carpeta `logs` del host. Puedes enviar el fichero de log `log_example.log`, línea a línea con una pausa de medio segundo entre líneas del siguiente modo:
```bash
while read line; do echo $line; sleep 0.5; done < log_example.log > ../logs/test.log
```

Mientras se sube el fichero, entra en [Kibana](http://localhost:5601) e intenta configurar encontrar los logs. _Tip: Discover_

Una vez visualizados los logs, trata de crear alguna visualización. _Tip: Dashboard_
