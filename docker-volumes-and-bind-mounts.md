# Docker Volumes and Bind Mounts

Hands-on notes from the Docker storage lab performed on WSL2.

---

# Goal

Learn the practical difference between:

* **Named volumes** (Docker-managed persistent storage)
* **Bind mounts** (host-managed storage)

---

# Named Volume Lab

## Create a volume

```bash
docker volume create nginx-data
docker volume ls
```

Output:

```text
DRIVER    VOLUME NAME
local     nginx-data
```

---

## Start nginx with the volume attached

```bash
docker run -d --name nginx-vol -p 8082:80 \
  -v nginx-data:/usr/share/nginx/html nginx
```

---

## Create a web page inside the container

```bash
docker exec nginx-vol sh -c \
  'echo "<h1>Persistent Page</h1>" > /usr/share/nginx/html/index.html'
```

Browser:

* `http://localhost:8082`

Result:

```html
<h1>Persistent Page</h1>
```

---

## Remove the container

```bash
docker rm -f nginx-vol
```

---

## Recreate a new container with the same volume

```bash
docker run -d --name nginx-vol2 -p 8082:80 \
  -v nginx-data:/usr/share/nginx/html nginx
```

Browser still showed:

```html
<h1>Persistent Page</h1>
```

### Observation

The file survived container deletion because it was stored in the **volume**, not in the container writable layer.

---

## Inspect the volume

```bash
docker volume inspect nginx-data
```

Important field:

```json
"Mountpoint": "/var/lib/docker/volumes/nginx-data/_data"
```

This shows where Docker stored the volume data on the host.

---

# Bind Mount Lab

## Create a host directory

```bash
mkdir -p ~/web-content
echo '<h1>Host Bind Mount</h1>' > ~/web-content/index.html
```

Verify:

```bash
ls -l ~/web-content
```

---

## Start nginx with a bind mount

```bash
docker run -d --name nginx-bind -p 8083:80 \
  -v ~/web-content:/usr/share/nginx/html nginx
```

Browser:

* `http://localhost:8083`

Result:

```html
<h1>Host Bind Mount</h1>
```

---

## Modify the file on the host

```bash
echo '<h1>Updated From Host</h1>' > ~/web-content/index.html
cat ~/web-content/index.html
```

Output:

```html
<h1>Updated From Host</h1>
```

After refreshing the browser, nginx immediately showed:

```html
<h1>Updated From Host</h1>
```

### Observation

Changes made on the host were visible instantly inside the running container without restarting it.

---

# Comparison

| Feature                          | Named Volume                | Bind Mount      |
| -------------------------------- | --------------------------- | --------------- |
| Host path chosen by              | Docker                      | User            |
| Example                          | `nginx-data`                | `~/web-content` |
| Stored under `/var/lib/docker`   | Yes (default)               | No              |
| Good for persistent app data     | Yes                         | Sometimes       |
| Good for editing files from host | No                          | Yes             |
| Host changes visible immediately | Usually not edited directly | Yes             |
| Survives container deletion      | Yes                         | Yes             |

---

# Key Notes

* A **named volume** is Docker-managed persistent storage.
* A **bind mount** uses a host directory chosen by the user.
* Deleting a container does **not** delete a named volume.
* Bind mounts are commonly used during development so code changes appear immediately inside containers.
* Docker-managed data is typically stored under `/var/lib/docker` on Linux hosts.
* If a running container is using a named volume, changes made directly to the volume directory on the host are usually visible inside the container immediately because both paths reference the same underlying files.
* In practice, manually editing Docker-managed volume directories is generally avoided; bind mounts are preferred when regular host-side editing is required.

---

# Cleanup Commands

Remove lab containers:

```bash
docker rm -f web1 web2 nginx-bind nginx-vol2
```

Remove stopped containers:

```bash
docker container prune
```

Check remaining volumes:

```bash
docker volume ls
```

---

# What I Learned

* I can create and inspect Docker volumes.
* I verified that volume data persists across container deletion.
* I verified that bind mounts reflect host file changes immediately.
* I understood the practical difference between Docker-managed storage and host-managed storage.
* I practiced cleaning up temporary Docker resources after completing a lab.

