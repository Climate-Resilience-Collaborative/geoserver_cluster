# Geoserver Cluster + Prometheus Monitoring

## Description
This repository provides a scalable and resilient Geoserver cluster setup, along with a comprehensive monitoring stack, all orchestrated using Docker containers. The cluster is designed to handle high loads efficiently.

## Requirements
* Docker + Docker Compose
* 4 GB RAM
* Additional RAM to allocate more RAM per instance or create more instances

## Technology
* Docker
* PostgreSQL
* ActiveMQ Broker
* [Geoserver](https://hub.docker.com/r/geobeyond/geoserver)
* Caddy Load Balancer
* Prometheus
* Grafana

## Setup
Type the following command and docker containers will be up
```
docker compose up
```

## Scaling
To scale up the cluster, add the docker compose snippet with appropriate environemnt variables and replacing the # with number representing geoserver node to the bottom of the [docker-compose.yml](https://github.com/CRI-lab/geoserver_cluster/blob/main/docker-compose.yml)

```
gs-node#:
    image: geobeyond/geoserver:2.20.6
    volumes:
      - ${GEOSERVER_DATA_MNT}:${GEOSERVER_DATA_DIR}
      - ${GEOSERVER_CACHE_MNT}:${GEOSERVER_CACHE_DIR}
    ports:
      - "808#:8080"
    environment: 
    ...
```

To scale down the cluster, remove the docker compose snippet

## Services
| Service  | Port | 
| -------- | -------- |
| Caddy Reverse Proxy   | 8600   |
| Geoserver Instance 1   | 8081   |
| Geoserver Instance 2   | 8082   |
| Grafana Dashboard   | 3000   |
| Prometheus   | 9090   |
| ActiveMQ Broker | 61616   |

## Clustering using JMS Plugin
GeoServer supports clustering using JMS cluster plugin or using the ActiveMQ-broker. 

This setup uses the JMS cluster plugin which uses an embedded broker. A docker-compose.yml
is provided in the clustering folder which simulates the replication using 
a shared data directory.

The environment variables associated with replication are listed below
* `CLUSTERING=True` - Specified whether clustering should be activated.
* `BROKER_URL=tcp://0.0.0.0:61661` - This links to the internal broker provided by the JMS cluster plugin.
This value will be different for (Master-Node)
* `READONLY=disabled` - Determines if the GeoServer instance is Read only
* `RANDOMSTRING=87ee2a9b6802b6da_master` - Used to create a unique CLUSTER_CONFIG_DIR for each instance. Not mandatory as the container can self generate this.
* `INSTANCE_STRING=d8a167a4e61b5415ec263` - Used to differentiate cluster instance names. Not mandatory as the container can self generate this.
* `CLUSTER_DURABILITY=false`
* `TOGGLE_MASTER=true` - Differentiates if the instance will be a Master
* `TOGGLE_SLAVE=true` - Differentiates if the instance will be a Node
* `EMBEDDED_BROKER=disabled` - Should be disabled for the Node
* `CLUSTER_CONNECTION_RETRY_COUNT=10` - How many times try to connect to broker
* `CLUSTER_CONNECTION_MAX_WAIT=500` - Wait time between connection to broker retry (in milliseconds)

## Monitoring
Prometheus collects metrics from Geoserver, RabbitMQ, and other services, and Grafana provides visualization and alerting capabilities. Grafana dashboards can be tailored to track various performance metrics, such as server load, request rates, and response times.

## Grafana Dashboards
The dashboards were pulled from https://github.com/DoTheEvo/selfhosted-apps-docker/tree/master/prometheus_grafana_loki/dashboards
docker-container.md: Monitor docker container metrics
docker-host.md: Monitor docker host metrics
monitoring-services.md: Monitor monitoring services (Prometheus, grafana, node exporter)

## Reference Links
 * Monitoring + Logging: https://github.com/DoTheEvo/selfhosted-apps-docker/blob/master/prometheus_grafana_loki/
 * Cluster: https://github.com/geobeyond/geoserver-clustering-playground
 * Blog: https://medium.com/geobeyond/making-geoserver-fast-vertical-geoserver-clustering-bf9dbdb5d61a
