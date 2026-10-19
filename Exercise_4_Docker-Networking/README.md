# 🐳 Docker Networking with Multiple Containers

## 🎯 Objective

Understand Docker networking by connecting a multi-container application on a custom **bridge network** and verifying that the containers can reach each other by name.

## 🧩 Scenario

A web application made of three containers on one bridge network:

- **Flask** web server (container 1)
- **MySQL** database (container 2)
- **Redis** cache (container 3)

## 🛠️ Environment

- **OS:** Windows 11 (AMD64), commands run in PowerShell
- **Engine:** Docker Desktop

## 📁 Files

| File | Purpose |
|------|---------|
| `app.py` | Minimal Flask REST API (`/about` endpoint) |
| `requirements.txt` | Pins `Flask==2.0.1` + `Werkzeug==2.0.1` (compatible versions) |
| `Dockerfile` | Builds a `python:3.9-slim` image for the Flask app |

## 📋 Tasks Performed

**1. Create a bridge network**
```powershell
docker network create --driver bridge my-bridge-net
```

**2. Verify the network**
```powershell
docker network ls
```

**3. Inspect the network**
```powershell
docker network inspect my-bridge-net
# Subnet 172.21.0.0/16, Gateway 172.21.0.1
```

**4. Build the Flask image and launch all three containers on the network**
```powershell
docker build -t flask-api .
docker run -d --name mysql --net my-bridge-net -e MYSQL_ROOT_PASSWORD=root mysql:latest
docker run -d --name redis --net my-bridge-net redis:7-alpine
docker run -d --name flask --net my-bridge-net -p 5001:5001 flask-api
docker ps        # all 3 containers Up
```

**5. Test connectivity — from inside the Flask container, ping the others by name**
```powershell
docker exec -it flask bash
```
```bash
apt-get update && apt-get install -y iputils-ping
ping -c 4 mysql     # replies from mysql (172.21.0.3)
ping -c 4 redis     # replies from redis (172.21.0.4)
exit
```

**6. Clean up**
```powershell
docker stop flask mysql redis
docker rm flask mysql redis
docker network rm my-bridge-net
```

## 📸 Output

**1. Bridge network created, listed, and inspected:**

![network](./network-output.png)

**2. All three containers running on the network:**

![containers](./containers-output.png)

**3. Connectivity test — Flask pinging MySQL and Redis by name (0% packet loss):**

![connectivity](./connectivity-output.png)

## 🧠 Notes & Fixes

A few adjustments were needed to make the exercise run on this Windows/AMD64 setup:

- **`host='0.0.0.0'` in `app.py`** — so the Flask app is reachable outside the container (via `-p 5001:5001`).
- **Pinned `Werkzeug==2.0.1`** — Flask 2.0.1 crashes on startup with newer Werkzeug versions.
- **`-e MYSQL_ROOT_PASSWORD=root`** — `mysql:latest` exits immediately without a root password set.
- **`iputils-ping` installed** — the slim Python image has no `ping` by default.
- **`redis:7-alpine` instead of `redis:latest`** — the `redis:latest` image resolved to a broken/mismatched build on this machine (`exec format error`); the pinned Alpine build runs correctly.

## ❓ Q&A

1. **Purpose of `--net` in `docker run`?** Connects the container to the specified network.
2. **How do containers communicate on the same network?** By container **name** (Docker provides automatic DNS) or by IP.
3. **Bridge vs host network?** A **bridge** network isolates containers on a private virtual network (they talk to each other through it); a **host** network shares the host's network stack directly.
4. **How to expose a container port to the host?** Use the `-p` flag, e.g. `-p 5001:5001`.

## ✅ Result

Created a custom bridge network, ran Flask, MySQL, and Redis containers on it, and confirmed the containers resolve and reach each other by name — demonstrating Docker's built-in container networking and DNS.
