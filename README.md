# Service Discovery - Consul

open two terminals:

- first terminal
- - Starting an agent

```
$ docker-compose up -d
$ docker exec -it consulserver01 sh
-----
/ # consul agent -dev
```

- second terminal
- - show nodes up using consul, own rest api and show by own DNS server

```
$ docker exec -it consulserver01 sh
-----
/ # consul members
/ # curl localhost:8500/v1/catalog/nodes
/ # apk -U add bind-tools
/ # dig @localhost -p 8600
/ # dig @localhost -p 8600  consulserver01.node.consul
/ # dig @localhost -p 8600  consulserver01.node.consul +short
```

## Comunication between them

One terminal for each machine up, in this example, it'll gonna be 3. But first of all, down your compose, in this example we'll use agent as server mode instead of dev

- All three containers:

```
$ docker exec -it consulserver{n} sh
------
/ # # Getting your container IP Address
/ # ifconfig
/ # mkdir /etc/consul.d
/ # mkdir /var/lib/consul
/ # consul agent -config-dir=/etc/consul.d
```

- Open another terminal and, in first consulserver container, to join, if not joined:

```
$ docker-compose exec -it consulserver01 sh
-----
/ # consul join <IP_FROM_OTHERS_CONTAINERS>
```

- Open two more terminal and join container 02 and 04, and join first container

```
/ # consul join <IP_FIRST_CONTAINER>
```

Using the following command in any container will return the three members

```
/ # consul members
```

## Creating a client

- Enter client and create

```
$ docker exec -it consulclient01 sh
-----
/ # # Save your IP Address
/ # ifconfig
/ # mkdir /var/lib/consul
/ # consul agent -bind=<IP_ADDR> -data-dir=/var/lib/consul -config-dir=/etc/consul.d -retry-join=<IP_CONTAINER_SERVER01> -retry-join=<IP_CONTAINER_SERVER02>
-retry-join=<IP_CONTAINER_SERVER03>
```

- Open other terminal and access client container to add the client to cluster and verify the members

```
$ docker exec -it consulclient01 sh
-----
/ # consul join <IP_FIRST_CONSUL_SERVER>
/ # consul members
```

To sync services registered use, in client:

```
/ # consul reload
```

To check service "nginx" is running, in any container in same cluster:

```
/ # apk -U add bind-tools
/ # dig @localhost -p 8600 nginx.service.consul
/ # curl localhost:8500/v1/catalog/services
/ # consul catalog nodes -service nginx
```

## For Production

Use encryption and TLS configured into each server and client for more security
