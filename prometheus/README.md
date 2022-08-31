# Monitorizando nuestras aplicaciones en PulpoCon 2022

## Requisitos

- Docker versión > 20.10
- Docker compose
- Preferible un entorno Linux

## Monitorizando sistemas con Prometheus

En esta carpeta encontrarás el fichero compose para levantar la infraestructura para monitorizar tu máquina.

*Modo rápido*
```bash
cd prometheus
docker compose -p pulpocon up -d
```

Si se tiene compose V1 (el clásico script Python) en lugar del plugin de compose V2 las instrucciones serán con 
docker-compose en lugar de docker compose. Y los nombres de los contenedores llevarán _ en lugar de - (i.e. 
pulpocon_prometheus_1).

*Ejercicios paso a paso*

### Ejercicio 1: monitorizar el nodo

#### Alertmanager

[alertmanager](https://github.com/prometheus/alertmanager)

Emite alertas según las reglas configuradas en Prometheus.

```bash
docker compose -p pulpocon up -d alertmanager
```

- alermanager/config.yml para editar cómo se alerta

Comprueba la interfaz disponible en http://localhost:9093/

#### Node exporter
[node-exporter](https://github.com/prometheus/node_exporter)

Exporta métricas de la máquina para que se pueden scrapear. Las deja disponibles en http://localhost:9100/metrics

```bash
docker compose -p pulpocon up -d node-exporter
```

En algunos sistemas puede ser necesario añadir SYS_TIME para recolectar tiempos.

(Para Windows se debe instalar https://github.com/prometheus-community/windows_exporter)

#### Prometheus

[prometheus](https://prometheus.io/)

- Interfaz disponible en http://localhost:9090/
- Métricas en http://localhost:9090/metrics

```bash
docker compose -p pulpocon up -d prometheus
```

Configuración
- prometheus.yml
- alert_rules.yml

Jobs de monitorización (sección scrape_config en prometheus.yml):
- métricas de Prometheus
- métricas del nodo

#### Visualización básica

Prometheus expression browser: http://localhost:9090/

- Status
  - targets 
  - config
- Alerts
  - high_load activa (en alert_rules se ha configurado un umbral bajo)
- Graph
  - tiempo de CPU
    - filtro por cpu: node_cpu_seconds_total{cpu="0",mode="user"}
    - todas las cpus: node_cpu_seconds_total{job="node",mode="user"}
    - pasar del valor acumulado a promedio temporal: irate(node_cpu_seconds_total{job="node",mode="user"}[30s])

#### Visualización Grafana

```bash
docker compose -p pulpocon up -d grafana
```

ACCESO:
http://localhost:3000/
- user: pulpocon
- pass: rules

Primero hay que configurar el datasource (Settings): http://prometheus:9090

Después se puede explorar (Explore), usando el builder de ayuda o raw queries en modo Code:
```promql
node_cpu_seconds_total{cpu="0",mode="user"}
irate(node_cpu_seconds_total{cpu="0",mode="user"}[30s])
```

Por último se pueden usar dashboards. Lo más cómodo es importarlos desde grafana.com:
- 1860 : node explorer
- 3662: prometheus


### Ejercicio 2: monitorizar docker

#### cAdvisor

[cadvisor](https://github.com/google/cadvisor) Container advisor

Recoge métricas de los contenedores docker. Disponibles en http://localhost:8080/metrics

```bash
docker compose -p pulpocon up -d cadvisor
```

En http://localhost:8080/containers/ se puede navegar por los contenedores y ver gráficas e información sobre ellos.

#### Prometheus

Es necesario añadir a la configuración de Prometheus la tarea de scrapeo de este servicio. Fichero config/prometheus2.yaml

```bash
docker restart pulpocon-prometheus-1
```

#### Grafana

Explorar:

```promql
irate(container_cpu_usage_seconds_total{name="pulpocon-node-exporter-1"}[1m])
```
Tip: Legend {{name}}-{{cpu}}

Importar
- 15894: docker
- 179: docker & host

### Ejercicio 3: monitorizar docker daemon

Para poder monitorizar el docker daemon es necesario editar:
```bash
sudo nano /etc/docker/daemon.json
```
```json
{
 "metrics-addr" : "0.0.0.0:9323",
 "experimental" : true
}
```

```bash
sudo systemctl restart docker
```

Comprueba las métricas en http://172.17.0.1:9323/metrics

#### Prometheus

En la configuración de prometheus se debe añadir un job de docker que debe tener como target la <ip del 
host>:9323. Fichero config/prometheus3.yaml
Es necesario modificar el despliegue para incluir privilegios (user root) en el servicio de Prometheus. 
Además se añade el host de docker para facilitar la resolución de su IP (Esto funciona automáticamente en el 
despliegue para Docker > 20.10 en linux usando el compose proporcionado y descomentando extra_hosts)

Ejemplo de métricas añadidas:
```promql
- builder_builds_failed_total{job="docker"}
- engine_daemon_container_states_containers{state="running"}
```

### Ejercicios extra: monitorizar los nuevos servicios para logs

Se pueden añadir las métricas de los servicios que se van desplegando para la agregación de logs:
- http://loki:3100/metrics
- http://promtail:9080/metrics

Tip:
```promql
  - job_name: 'new-service'
    scrape_interval: 5s
    static_configs:
      - targets: ['new-service:new-service-port']
```
Import dashboard 10004

## Extras

### Portainer 
Interfaz para ayudar a gestionar los recursos docker en https://localhost:9443. 

En la primera ejecución es necesario asignar un usuario y contraseña, y configurar el entorno local.

```bash
 docker run -d -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

