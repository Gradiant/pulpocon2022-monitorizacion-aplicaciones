# Monitorizando nuestras aplicaciones en PulpoCon 2022

Ejemplos sencillos de como monitorizar el sistema y las propias aplicaciones (logs y métricas) con los stacks Open 
Sources más comunes. 

## Requisitos

- Docker versión > 20.10
- Docker compose
- Preferible un entorno Linux

## Ejercicio Prometheus

En la carpeta `prometheus` encontrarás el fichero compose para levantar la infraestructura para monitorizar 
tu máquina.

## Ejercicio Grafana Stack vs Elastic Stack

- En la carpeta `elastic-stack` encontrarás el fichero compose para levantar la infraestructura para analizar los 
  logs utilizando el stack de Kibana + Elasticsearch + Logstash + Filebeat.

- En la carpeta `grafana-stack` encontrarás el fichero compose para levantar la infraestructura para analizar los 
  logs utilizando el stack de Grafana + Loki + Promtail.


### Demo de observability con ELK

https://demo.elastic.co/app/observability/