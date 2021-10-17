# Blog 4 Second half of the 12-factor app

## Introduction

## 7. Port binding

80 & 8010

The 12-factor application must be self-contained and bind to a port that is provided in the environment variable. It can't rely on a web container like tomcat or unicorn being injected; instead, it must incorporate a server like jetty or thin. Requests from a public-facing hostname are sent through the execution environment to the port-bound web process.
With most embedded web servers, this is a simple task. If you're already utilising an external web server, supporting an embedded server within your application may take extra effort.

For the frontend application, IoT application, we are binding port 80 so that the end-user can interface with the IoT device web interface. This is so if they want to config the WIFI username and passwords or if they want to manually upgrade or downgrade the application version running in their device. This is done by the following.

```cpp
WebServer server(80);
```
### i. RabbitMQ

RabbitMQ nodes bind to ports (open server TCP sockets) to accept client and CLI tool connections. Some of the available ports you can bind are:
- 5672, 5671: used by AMQP 0-9-1 and AMQP 1.0 clients without and with TLS
- 15672: HTTP API clients, management UI and rabbitmqadmin (only if the management plugin is enabled)
- 1883, 8883: MQTT clients without and with TLS, if the MQTT plugin is enabled
- 15675: MQTT-over-WebSockets clients (only if the Web MQTT plugin is enabled)
- 15692: Prometheus metrics (only if the Prometheus plugin is enabled)

```yaml
rabbitmq:
build: rabbitmq/
image: mqtt-rabbitmq
container_name: "mqtt"
ports:
    - "15672:15672"
    - "5672:5672"
    - "1883:1883"
```

### i. InfluxDB

Some open ports you can bind when using the InfluxDB service are:
- 8086: The default port that runs the InfluxDB HTTP service.
- 8088: The default port used by the RPC service for RPC calls made by the CLI for backup and restore operations (influxdb backup and influxdb restore)

Disabled ports
- 2003 The default port that runs the Graphite service. 
- 4242 The default port that runs the OpenTSDB service.
- 8089 The default port that runs the UDP service.


```yaml
influxdb:
image: influxdb:1.7-alpine
container_name: "influxdb"
ports:
    # The API for InfluxDB is served on port 8086
    - "8086:8086"
    - "8082:8082"
    # UDP Port
    - "8089:8089/udp"
```

### iii. Kapacitor

Kapacitor exposes port 9092 which is the API port for the service.

```yaml
kapacitor:
image: kapacitor:1.5-alpine
container_name: "kapacitor"
ports:
    # The API for Kapacitor is served on port 9092
    - "9092:9092"
```

### iv. Chronograf

Chronograf has a WebUI port which is `8888`. If you want to visualise your data you bind that port and open it when the application is running

```yaml
chronograf:
image: chronograf:1.7-alpine
container_name: "chronograf"
ports:
    # The WebUI for Chronograf is served on port 8888
    - "8888:8888"
```

### v. Grafana

Grafana has a WebUI port which is `3000`. If you want to visualise your data you bind that port and open it when the application is running

```yaml
grafana:
image: grafana/grafana:latest
container_name: "grafana"
ports:
    - '3000:3000'
```

### vi. Portainer

Portainer has a WebUI port which is `8000` and `9443`. One of them is HTTPS while the other is HTTP. If you want to visualise your data you bind that port and open it when the application is running

```yaml
portainer:
image: portainer/portainer-ce:latest
container_name: portainer
ports:
    - '8000:8000'
    - '9443:9443'
```

### vii. Kibana

Kibana has a WebUI port which is `5601`. If you want to visualise your data you bind that port and open it when the application is running

```yaml
kibana:
build: kibana/
container_name: "kibana"
ports:
    - "5601:5601"
```

### viii. Logstash

```yaml
logstash:
build: logstash/
container_name: "logstash"
ports:
    - "5000:5000"

```

### ix. Elasticsearch

```yaml
elasticsearch:
build: elasticsearch/
container_name: "elasticsearch"
ports:
    - "9200:9200"
    - "9300:9300"
```

### x. nginx

Nginx exposes port 80 you have to bind it to another port so that you can serve those requests through it.

```yaml
nginx:
image: nginx:latest
container_name: "nginx"
ports:
    - "8080:80"
```
## 8. Concurrency

Keep your sessions in Redis or DB

Because a 12-factor solely employs stateless processes, it may expand out by adding processes. A 12-factor application can contain several process types, such as browser processes, background worker processes, or clock processes (for cron-like scheduled activities) (for cron-like scheduled jobs).
As each process type is scaled separately, each logical process would become its own Docker container as well. We’ve previously seen developing a web process; other processes are pretty similar. Scaling out usually only entails deploying new instances of the container. (However, scaling out the clock processes is typically not recommended because they frequently create events that you want to be scheduled as singletons inside your infrastructure.)

## 9. Disposability

Thank Docker and Makefile!

Processes in a 12-factor app can be launched or halted (with a SIGTERM) at any moment. As a result, reducing starting time and gracefully shutting down is critical. When a web service receives a SIGTERM, for example, it should cease listening on the HTTP port, wait for in-flight requests to complete, and then depart. Processes should be resistant to an unexpected death in the same way; for example, worker processes should have a robust queuing backend.
You should choose a web server that can gracefully shut down. This is one of the more difficult aspects of choosing a web server, at least for many of the popular Python HTTP servers I've tested.

```makefile
GH_REPO = https://github.com/JKUATSES/2021-project-hack.git
C_DIR = chronograf/
I_DIR = influxdb/
K_DIR = kapacitor/
G_DIR = grafana/

clean: stop remove-files

all: clean pull build start

# This pull the images for the docker images
pull:
    docker-compose pull
# This builds any images which are supposed to be built while running docker compose
build:
    docker-compose build
# This starts up the containers. We change the read and write rights for the rabbitmq file the start it up
start:
    chmod 755 rabbitmq/cluster-entrypoint.sh
    docker-compose up -d
# This stops the containers while maintaining their volumes
stop:
    docker-compose down
# This prunes the images containers, volumes and networks. It also removes the data files that were created when the docker containers spun up into existence
remove-files:
    docker system prune -a
    docker volume prune
    [ -d $(C_DIR) ] && echo exists && rm -rf $(C_DIR) $(I_DIR) $(K_DIR) || echo not exists

# This watches the conatiner logs
logs:
    watch docker-compose logs
# Removes the data bases and creates new directories
clear_db:
    sudo rm -rf $(C_DIR)/data $(G_DIR)/data $(G_DIR)/provisioning $(I_DIR)/data $(K_DIR)/data elasticsearch/data logstash/data kibana/data
    mkdir $(K_DIR)/data $(C_DIR)/data $(I_DIR)/data elasticsearch/data logstash/data kibana/data
```
## 10. Dev/Prod parity

Yup
The goal of a 12-factor app is to reduce the gap between development and production as short as possible. The amount of time code spends in development but not in production is reduced with continuous deployment. Developers can deploy their code in production, just as they do in their local development environments, using a self-serve platform. Using the same underpinning services (databases, caches, queues, etc) in development as production lowers the number of minor issues that emerge in discrepancies across technologies or integrations.

## 11. Logs

Yup
Consider logs to be a continuous stream of time-ordered events gathered from all active processes and services. A 12-factor app is unconcerned with how its output is processed. It just publishes its output to the stdout stream instead. This output is collected, collated, and routed to its eventual destination by the execution environment (s).
Most logging frameworks enable logging to stderr/stdout by default or provide a simple way to switch from file-based logging to one of these streams. The execution environment of a 12-factor programme is required to capture these streams and manage them as the platform mandates.

Elasticsearch, Logstash, and Kibana are the three open-source tools that make up the ELK Stack. The ELK stack offers centralised logging to help discover server or application issues. It allows you to search through all of your logs in one location. It also aids in the discovery of faults across several servers by linking logs during a specified period.

The letter E stands for ElasticSearch, which is a log storage system.
The letter L stands for LogStash, which is utilised for both transporting and processing and storing logs.
Kibana (pronounced "kibana") is a visualisation tool (a web interface) that runs on Nginx or Apache.

## 12. Admin Processes

Run admin/management tasks as one-off processes

Any 12-factor app administration or management activities should be executed as one-off processes within the deployment's execution environment. This process runs against a release and utilises the same codebase and configurations as any other process in that release, as well as the same dependency isolation methods as the long-running processes.
This is a characteristic of the execution environment in which your programme runs. This may be simple if you're using a Docker-like containerized solution.
We will be using portainer to manage our Admin processes

## Acknowledgment
1. [Kelvin Gitu](https://twitter.com/GituKelvin/)