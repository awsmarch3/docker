# Docker Storage Lab – `--mount` Scenarios

This section covers the **same experiments** as the `-v` syntax but using the **`--mount`** flag for explicit and production-grade usage.

---

## 1️⃣ Docker Volumes with `--mount`

### **Scenario 1 – Named Volume**

```bash
docker volume create nginx-data
docker run -d -p 8810:80 \
  --mount type=volume,source=nginx-data,target=/usr/share/nginx/html \
  nginx
```

**Observation:**

* Volume `nginx-data` created automatically if not exists.
* Persistent even after container removal.

---

### **Scenario 2 – Read-Only Volume**

```bash
docker run -d -p 8812:80 \
  --mount type=volume,source=nginx-data,target=/usr/share/nginx/html,readonly \
  nginx
```

**Observation:**

* Container cannot modify or delete files inside mount.
* Volume content still editable from host under `/var/lib/docker/volumes/nginx-data/_data`.

---

### **Scenario 3 – Volume with Specific UID/GID**

```bash
docker run -d -p 8813:80 \
  --user 101:101 \
  --mount type=volume,source=nginx-data,target=/usr/share/nginx/html \
  nginx
```

**Observation:**

* Mounted data writable only by specified UID/GID in container.
* Root on host still has full control.

---

## 2️⃣ Bind Mounts with `--mount`

### **Scenario 1 – Empty Host Directory**

```bash
mkdir -p /root/docker-vol
docker run -d -p 8880:80 \
  --mount type=bind,source=/root/docker-vol,target=/usr/share/nginx/html \
  nginx
```

**Observation:**

* Empty host directory hides container’s default HTML → results in “403 Forbidden.”
* Adding `index.html` in host path fixes it.

---

### **Scenario 2 – Write from Container**

```bash
docker exec -it <container_id> bash
cd /usr/share/nginx/html
echo "<h2>Created inside container</h2>" > container.html
```

**Observation:**

* File visible on host at `/root/docker-vol/container.html`.
* Persists after container restart or recreation (if same mount reused).

---

### **Scenario 3 – Read-Only Bind Mount**

```bash
docker run -d -p 8881:80 \
  --mount type=bind,source=/root/docker-vol,target=/usr/share/nginx/html,readonly \
  nginx
```

**Observation:**

* Nginx can serve files but not modify them.
* Host can update files directly.

---

## 3️⃣ tmpfs Mounts with `--mount`

### **Scenario 1 – Basic tmpfs**

```bash
docker run -d -p 8890:80 --name nginx-tmpfs \
  --mount type=tmpfs,target=/usr/share/nginx/html \
  nginx
docker exec -it nginx-tmpfs bash
echo "<h1>Hello from tmpfs</h1>" > /usr/share/nginx/html/index.html
```

**Observation:**

* Data exists only in memory and disappears on container removal.
* tmpfs not listed under `docker volume ls`.

---

### **Scenario 2 – tmpfs with Size Limit**

```bash
docker run -d -p 8891:80 --name nginx-tmpfs2 \
  --mount type=tmpfs,target=/usr/share/nginx/html,tmpfs-size=64m \
  nginx
```

**Observation:**

* Limits tmpfs mount to 64 MB.
* Files vanish after container removal or restart.

---

## **Key Takeaways (for --mount)**

| Storage Type | Persistence | Shared        | Ephemeral | Use Case            | Flag Used             |
| ------------ | ----------- | ------------- | --------- | ------------------- | --------------------- |
| Volume       | ✅           | ✅             | ❌         | Persistent app data | `--mount type=volume` |
| Bind Mount   | ✅           | ✅ (with host) | ❌         | Local dev, testing  | `--mount type=bind`   |
| tmpfs        | ❌           | ❌             | ✅         | Cache, sessions     | `--mount type=tmpfs`  |

---

✅ **Tip:**
Use `--mount` when scripting, automating, or using advanced mount options.
Use `-v` for quick, one-off testing.
