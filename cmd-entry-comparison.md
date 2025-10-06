# Docker Instruction Comparison & Lab Exercises

This document summarizes all the Docker hands-on exercises and comparisons executed so far, including CMD, ENTRYPOINT, COPY, ADD, and practical testing with Python apps.

---

## 1️⃣ Files Used

**app1.py**
```python
# app1.py
print("🚀 This is the DEFAULT app — app1.py is running.")
```

**app2.py**
```python
print("✨ This is the CUSTOM app — app2.py is running instead.")
```

**cmd Dockerfile**
```dockerfile
FROM ubuntu
COPY . .
CMD ["echo", "Hello"]
```

**entry Dockerfile**
```dockerfile
FROM ubuntu:latest
WORKDIR /app
ENTRYPOINT ["echo", "Hello from ENTRYPOINT"]
```

**entry1 Dockerfile**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY . .
ENTRYPOINT ["python"]
CMD ["app1.py"]
```

---

## 2️⃣ Experiments and Observations

### CMD Example (`cmd` image)
| Command | Output / Behavior |
|---------|------------------|
| `docker run cmd` | Hello (default CMD) |
| `docker run cmd San` | Error: San not found (CMD replaced completely) |
| `docker run cmd echo san` | san (CMD replaced with `echo san`) |
| `docker run cmd python3 san` | Error: python3 not installed |
| `docker run cmd ls` | Shows directory listing (CMD replaced) |
| `docker run cmd cat cmd` | Shows content of cmd file |

### ENTRYPOINT Example (`entry` image)
| Command | Output / Behavior |
|---------|------------------|
| `docker run entry` | Hello from ENTRYPOINT |
| `docker run entry san` | Hello from ENTRYPOINT san (arguments appended) |
| `docker run entry "Iam learning docker"` | Hello from ENTRYPOINT Iam learning docker |

### ENTRYPOINT + CMD Example (`entry1` image)
| Command | Output / Behavior |
|---------|------------------|
| `docker run entry1` | 🚀 This is the DEFAULT app — app1.py is running. (CMD used as default argument) |
| `docker run entry1 app2.py` | ✨ This is the CUSTOM app — app2.py is running instead. (CMD overridden) |
| `docker run entry1 ls` | Error: python cannot open /app/ls (ENTRYPOINT fixed, CMD replaced) |
| `docker run --entrypoint cat entry1 /etc/os-release` | Shows OS release info (temporary ENTRYPOINT override) |

---

## 3️⃣ COPY vs ADD

### Files for testing
- `hello.txt` — simple text file
- `inside.txt` inside `archive.tar.gz` — tar archive

### COPY Example
```dockerfile
FROM alpine
WORKDIR /app
COPY hello.txt .
COPY archive.tar.gz .
CMD ["ls", "-l"]
```
- Output: `hello.txt archive.tar.gz` (tar not extracted)

### ADD Example
```dockerfile
FROM alpine
WORKDIR /app
ADD hello.txt .
ADD archive.tar.gz .
CMD ["ls", "-l"]
```
- Output: `hello.txt inside.txt` (tar extracted automatically)

### Key Points
| Feature | COPY | ADD |
|---------|------|-----|
| Copies files/folders | ✅ | ✅ |
| Auto-extract tar | ❌ | ✅ |
| Supports URLs | ❌ | ✅ |
| Recommended use | General copy | Use only if tar extraction or URL needed |

---

## 4️⃣ Observations Summary

1. **CMD**: default command, fully replaced by `docker run <args>`.
2. **ENTRYPOINT**: fixed executable, arguments appended.
3. **ENTRYPOINT + CMD**: best practice, fixed executable + default arguments.
4. **COPY vs ADD**: COPY for regular files; ADD for tar extraction or remote URLs.
5. **--entrypoint**: allows temporary override of ENTRYPOINT without rebuilding.

---

This document now includes **all the Docker tasks and experiments executed so far** with relevant outputs and behavior notes.

