# Monitorizando nuestras aplicaciones
## PulpoCon 2022

## Grafana + Loki + Promtail
```bash
# Lanzamos Grafana
docker-compose up -d grafana

# Lanzamos Loki
docker-compose up -d loki

# Lanzamos Promtail
docker-compose up -d promtail
```

Tip: Si quieres puedes lanzar todo de una sola vez, pero queríamos alagar el taller :wink:
```bash
docker-compose up -d
```

Promtail se ha configurado para escuchar todos los ficheros `*.log` que aparezcan en la carpeta `logs` del host. Puedes enviar el fichero de log `log_example.log`, línea a línea con una pausa de medio segundo entre líneas del siguiente modo:
```bash
while read line; do echo $line; sleep 0.5; done < log_example.log > ../logs/test.log
```

Mientras se sube el fichero, entra en [Grafana](http://localhost:3000) e intenta configurar Loki. _Tip: Data sources_

Una vez configurado Loki, trata de mostrar los logs.

Ahora prueba a hacer alguna gráfica de conteo de los logs, p. ej:
```promql
count_over_time({job="pulpolog"}[1m])
```