# Blog 5 Recommendations

The Internet of Things has brought with it a slew of advantages. It has, however, increased and altered business and IT risks. Reports of hijacked cameras hacked medical equipment and infiltrated industrial control systems have appeared in recent years. The situation is probably expected to increase when 5G takes hold and gadgets with embedded IoT capabilities arrive. What makes the IoT so hard is that it puts an extra layer of security atop current safeguards. Because the IoT potentially affects everything within a business — and outward to partners and supply chains — it encompasses firmware, operating systems, TCP/IP stacks, network design, data security technologies, and much more.

## Davos
https://docs.linuxserver.io/images/docker-davos

Davos is an FTP automation programme that searches specified host sites for fresh files regularly. It may be set up to do a variety of things, such as listening for particular files to exist at the host location, then downloading and moving them if necessary. It also includes completion alerts and downstream API calls to help move the workflow along.
We can use Davos to push the binary files where we will use to send the Over The Air updates to the individual IoT devices.

An alternative to this Grav which is a Fast, Simple, and Flexible, file-based Web-platform
https://docs.linuxserver.io/images/docker-grav

## Domoticz
https://docs.linuxserver.io/images/docker-domoticz
Domoticz is a Home Automation System that allows you to monitor and set a variety of devices such as lights, switches, and different sensors/meters like temperature, rain, wind, UV, Electra, Gas, and Water, among others. Any mobile device may get notifications or alerts.
We can use this to configure our DHT11 sensor and more sensors if we were to use any of them.

## Duckdns
https://docs.linuxserver.io/images/docker-duckdns
Duckdns is a free service that will point a DNS (duckdns.org subdomains) to an IP address of your choosing. The service is entirely free, and it does not require reactivation or forum postings to be active.
If we are to open up more endpoints like our MQTT endpoint and our dashboards so that they can be reached by a domain name, we can use this service to configure

## Healthchecks
https://docs.linuxserver.io/images/docker-healthchecks
Healthchecks is a watchdog for your cron jobs. It's a web server that listens for pings from your cron jobs, plus a web interface.
This can be used to ensure our services are running by being a watchdog

## Syslog-ng
https://docs.linuxserver.io/images/docker-syslog-ng
syslog-ng allows you to flexibly collect, parse, classify, rewrite and correlate logs from across your infrastructure and store or route them to log analysis tools
This can be used to read our logs files and visualize them

## Wireguard
https://docs.linuxserver.io/images/docker-wireguard
WireGuard is a user-friendly, fast, and contemporary VPN that employs cutting-edge cryptography. Its goal is to be quicker, simpler, leaner, and more helpful than IPsec while avoiding the huge hassle. It aspires to be significantly faster than OpenVPN. WireGuard is a general-purpose VPN that can be used on both embedded interfaces and supercomputers and is suitable for a wide range of scenarios. It was first published for the Linux kernel, but it is now available for Windows, macOS, BSD, iOS, and Android. It's still in the early stages of development, but it's already widely recognised as the most secure, user-friendly, and straightforward VPN option available.
This can be used to ensure security when the user is getting access to his home network using public wifi they can route their traffic through this wireguard server

## Explot IoT
https://gitlab.com/expliot_framework/expliot
This is a pentest framework like Metasploit but specialized for IoT. This can be used to gauge the security of our systems

## Other Ideas
https://github.com/nebgnahz/awesome-iot-hacks
Some of the ideas on how to implement IoT can be found on this awesome IoT hacks.

## Mosquitto
https://mosquitto.org/
Mosquitto is lightweight and is suitable for use on all devices from low power single board computers to full servers. This can be a replacement for RabbitMQ

## Cloud
https://cloud.google.com/solutions/iot/
https://azure.microsoft.com/en-us/services/iot-hub/
https://aws.amazon.com/iot-core/?nc1=h_ls

Allow for an extremely secure and dependable connection between your IoT app and the devices it handles. IoT Hubs offers a cloud-based backend that can link practically any device. With per-device authentication, built-in device management, and scalable provisioning, you can extend your solution from the cloud to the edge.

## Kong
https://konghq.com/install/
Kong is a Microservice API Gateway for Orchestration. Kong provides a versatile abstraction layer for managing API interactions between clients and microservices safely. Also called an API Gateway, API middleware, or Service Mesh in some situations. It was released as an open-source project in 2015, with great performance and extensibility as its main objectives.
If you're developing for the web, mobile, or IoT (Internet of Things), you'll almost certainly require some basic functionality to get your app running. Kong may assist by acting as a gateway (or sidecar) for microservices requests, while also offering load balancing, logging, authentication, rate-limiting, transformations, and more via plugins.

## Traefik
https://traefik.io/traefik/
Traefik is a popular contemporary reverse proxy and load balancer that makes microservice deployment simple. Traefik interacts with your current infrastructure components and automatically and dynamically configures itself.

Traefik is meant to be as simple to use as possible while yet supporting big, complicated installations in a variety of settings and protocols in public, private, and hybrid clouds. It also includes a strong collection of middlewares for load balancing, API gateway, orchestrator ingress, east-west service communication, and more.

We can use it together with Let's Encrypt to ensure we have a secure MQTTS connection with the IoT device and an HTTPS connection with our users.

## Istio
https://istio.io/

Istio is an open-source service mesh that allows teams to connect, secure, control, and watch services. It is built on Envoy and is extendable. Istio is a collaborative effort between IBM and Google, which gave the initial components, and Lyft, which donated Envoy to the Cloud Native Computing Foundation in 2017.

As a result, Istio's feature set has matured and improved. Load balancing, traffic routing, policy development, metrics, and service-to-service authentication are all key Istio capabilities.

## Kubernetes
https://kubernetes.io/
For cloud-native deployments, Kubernetes has become the de-facto standard container orchestration framework. Kubernetes has been used by development teams to help them transition to new microservice architectures and a DevOps culture for continuous integration and deployment.

Many organisations are undergoing digital transformations at the same time. Their mission is to transform the way they interact with their customers, suppliers, and partners. To update their business IT and OT systems, these companies are utilising advances provided by technology such as IoT platforms, big data analytics, and machine learning.
It will automate our deployment, scaling, and management of containerized applications.

## More Resources
1. https://github.com/alvarowolfx/gcloud-ota-arduino-update

## References
1. https://www.syslog-ng.com/products/open-source-log-management/
2. https://libredd.it/r/Traefik?sort=hot