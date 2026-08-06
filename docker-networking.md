# Docker Networking Basics

Hands-on notes from the Docker networking lab performed in WSL2.

---

## Goal

Run an Nginx container, publish it to the host, and understand port mapping, network namespaces, and Docker bridge networking.

---

## Run an Nginx container

```bash
docker run -d --name web1 -p 8080:80 nginx
```

* `-d` = detached mode
* `--name web1` = container name
* `-p 8080:80` = publish host port 8080 to container port 80

---

## Verify container status

```bash
docker ps
```

Observed status:

```text
Up
```

---

## Access the web server

Working:

```text
http://localhost:8080
```

Result: Nginx welcome page displayed.

Not working:

```text
http://localhost:80
```

Reason: nothing was listening on host port 80.

---

## Port mapping syntax

```text
HOST_PORT:CONTAINER_PORT
```

Example:

```text
8080:80
```

Traffic sent to host port **8080** is forwarded to container port **80**.

---

## View published ports

```bash
docker port web1
```

Output:

```text
80/tcp -> 0.0.0.0:8080
80/tcp -> [::]:8080
```

This means the service is published on both IPv4 and IPv6 host addresses.

---

## Test from inside the container

```bash
docker exec web1 sh -c 'curl -I http://localhost:80 | head -n 1'
```

Output:

```text
HTTP/1.1 200 OK
```

Inside the container, `localhost` refers to the container itself.

---

## Host listener verification

```bash
ss -tulpn | grep ':80 '
```

No output → host port 80 unused.

```bash
ss -tulpn | grep ':8080 '
```

Output showed a listener on port 8080.

This proves the host is listening on 8080, not on 80.

---

## Multiple containers using port 80

Started a second container:

```bash
docker run -d --name web2 -p 8081:80 nginx
```

Both containers were running simultaneously:

* `web1` → host 8080 → container 80
* `web2` → host 8081 → container 80

---

## Container IP addresses

```bash
docker inspect web1 --format '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
docker inspect web2 --format '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

Output:

```text
web1 = 172.17.0.2
web2 = 172.17.0.3
```

Each container received a different IP address on the Docker bridge network.

---

## Network mode

```bash
docker inspect web1 --format 'IP={{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}} NetworkMode={{.HostConfig.NetworkMode}}'
```

Output:

```text
IP=172.17.0.2 NetworkMode=bridge
```

Docker used the default **bridge** network.

---

## Docker logs

```bash
docker logs web1 | tail -n 5
```

Observed:

* Nginx startup messages
* HTTP `200` for `/`
* HTTP `404` for `/favicon.ico`

The favicon 404 is normal because the file does not exist in the default Nginx image.

---

## What I learned

* Containers have isolated network namespaces.
* Each container gets its own IP address on the bridge network.
* Container ports can be reused across containers.
* Host ports must be unique on the host.
* Docker publishes services using port forwarding/NAT.
* `docker logs` is useful for troubleshooting containerized applications.

---

## Key takeaway

**Each Docker container has its own network namespace and IP address. Multiple containers can listen on port 80 simultaneously because the ports exist in different namespaces, while Docker exposes them externally by mapping different host ports to each container port using bridge networking and NAT.**

