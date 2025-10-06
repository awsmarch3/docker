# Docker Storage Lab – Volumes, Bind Mounts, tmpfs

This document summarizes all the Docker storage types we explored, with **hands-on tasks, commands, and observations**. It is structured for lab exercises and reference.

---

## 1️⃣ Docker Volumes

### **Scenario 1 – Named Volume**
```bash
docker volume create nginx-data
docker run -d -p 8610:80 -v nginx-data:/usr/share/nginx/html nginx
```
**Observation:**
- Data persists even after container removal.
- Files inside `/usr/share/nginx/html` in container reflect on volume.

### **Scenario 2 – Read-Only Volume**
```bash
docker run -d -p 8612:80 -v nginx-data:/usr/share/nginx/html:ro nginx
```
**Observation:**
- Container cannot write to volume.
- Host can still modify `_data` files manually.

### **Scenario 3 – Container User Isolation**
```bash
docker run -d -p 8613:80 -v nginx-data:/usr/share/nginx/html --user 101:101 nginx
```
**Observation:**
- Container can write to volume with specific user UID/GID.
- Host root still can access volume.

---

## 2️⃣ Bind Mounts

### **Scenario 1 – Empty Host Directory**
```bash
mkdir -p /root/docker-vol
docker run -d -p 8780:80 -v /root/docker-vol:/usr/share/nginx/html nginx
```
**Observation:**
- Empty host directory overrides container content → 403 forbidden.
- Adding `index.html` to host fixes it.

### **Scenario 2 – Write from Container**
```bash
docker exec -it <container> bash
cd /usr/share/nginx/html
echo "<h2>Created inside container</h2>" > container.html
```
**Observation:**
- Files created inside container appear on host directory.
- Persistent across container restarts.

### **Scenario 3 – Read-Only Bind Mount**
```bash
docker run -d -p 8781:80 -v /root/docker-vol:/usr/share/nginx/html:ro nginx
```
**Observation:**
- Container cannot modify files.
- Host can still modify content.

---

## 3️⃣ tmpfs Mounts

### **Scenario 1 – Basic tmpfs**
```bash
docker run -d -p 8790:80 --name nginx-tmpfs --tmpfs /usr/share/nginx/html:rw nginx
docker exec -it nginx-tmpfs bash
echo "<h1>Hello from tmpfs</h1>" > index.html
```
**Observation:**
- Data exists in container and is served by Nginx.
- After container removal, all files are lost.
- tmpfs does not appear in `docker volume ls`.

### **Scenario 2 – Multiple tmpfs Files**
```bash
docker run -d -p 8791:80 --name nginx-tmpfs2 --tmpfs /usr/share/nginx/html:rw nginx
docker exec -it nginx-tmpfs2 bash
echo "<h2>file1 in tmpfs</h2>" > file1.html
echo "<h2>file2 in tmpfs</h2>" > file2.html
```
**Observation:**
- Files accessible inside container.
- All files are ephemeral and lost on container removal.


## **Key Takeaways**

| Storage Type | Persistence | Shared | Ephemeral | Use Case |
|--------------|------------|--------|-----------|----------|
| Volume | ✅ | ✅ (single host) | ❌ | Persistent container data |
| Bind Mount | ✅ | ✅ (host only) | ❌ | Dev/test with host files |
| tmpfs | ❌ | ❌ | ✅ | Caches, sessions, ephemeral/sensitive data |


---

This lab can be used for **practical exercises**, **testing scenarios**, or **production architecture planning** for Docker storage types.

