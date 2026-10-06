# Linux Server Lab

## Objective

Build and document a lightweight Linux administration lab using an Ubuntu 24.04 Docker container. The goal is to demonstrate practical Linux administration, networking, service deployment, process inspection, and troubleshooting with current hands-on evidence.

---

## Lab environment

| Component | Configuration |
|---|---|
| Host | macOS with Docker Desktop |
| Linux environment | Ubuntu 24.04 container |
| Container name | `linux-lab` |
| Web service | Nginx 1.24.0 |
| Network | Docker bridge networking |

The lab runs inside a container rather than a full virtual machine. This keeps the environment lightweight while still providing a practical space for Linux administration and service troubleshooting.

---

## Build steps completed

### 1. Start the Ubuntu container

```bash
docker run -it --name linux-lab ubuntu:24.04 bash
```

### 2. Install required tools and services

```bash
apt update
apt install nginx iproute2 curl openssh-server -y
```

### 3. Verify Nginx installation

```bash
nginx -v
```

Observed result:

```text
nginx version: nginx/1.24.0 (Ubuntu)
```

### 4. Start Nginx

```bash
nginx
```

### 5. Confirm the service is running

```bash
ps aux | grep nginx
```

The process list showed one Nginx master process and multiple worker processes, confirming that the web server started successfully.

### 6. Test HTTP locally

```bash
curl http://localhost
```

The request returned the default **Welcome to nginx!** page, confirming that the server was responding over HTTP.

### 7. Inspect container networking

```bash
ip addr
```

The container received an address on the Docker bridge network:

```text
172.17.0.2/16
```

### 8. Verify the listening port

```bash
ss -tulpn
```

Observed result:

```text
tcp LISTEN 0 511 0.0.0.0:80 0.0.0.0:* users:(("nginx",pid=3503,fd=5))
tcp LISTEN 0 511 [::]:80 [::]:* users:(("nginx",pid=3503,fd=6))
```

This confirms Nginx is listening on TCP port 80 over both IPv4 and IPv6.

### 9. Validate HTTP headers

```bash
curl -I http://localhost
```

This was used to verify that the local web server returned a successful HTTP response.

### 10. Validate Nginx configuration

```bash
nginx -t
```

This was used to confirm the Nginx configuration syntax was valid.

---

## Troubleshooting performed

### Issue 1 — Nginx installed but HTTP request failed

**Problem**  
Running:

```bash
curl http://localhost
```

initially returned a connection failure.

**Investigation**  
Nginx was installed, but the service had not been started inside the container.

**Resolution**  
Started Nginx manually:

```bash
nginx
```

Then verified it using:

```bash
ps aux | grep nginx
curl http://localhost
```

**Lesson**  
Installing a package does not necessarily mean the service is running. In lightweight container environments, services often need to be started directly because a full init system such as systemd is not running.

### Issue 2 — Docker command unavailable inside the container

**Problem**  
Running `docker --version` from the Ubuntu prompt returned:

```text
bash: docker: command not found
```

**Investigation**  
Docker was running on the macOS host, while the Ubuntu container was only the Linux guest environment.

**Resolution**  
Docker commands were kept on the host, and Linux administration commands were run inside the container.

**Lesson**  
The container is not the Docker host. Understanding that separation is important when troubleshooting containerized environments.

---

## Skills demonstrated

This lab provides current hands-on evidence of:

- Ubuntu/Linux command-line administration
- package installation with APT
- Nginx installation and startup
- process inspection with `ps`
- local HTTP testing with `curl`
- IP/network inspection with `ip addr`
- socket and port inspection with `ss`
- Docker bridge networking
- basic service troubleshooting
- understanding the difference between a container and its Docker host

---

## Evidence status

Completed technical evidence:

- [x] Ubuntu 24.04 container running
- [x] Nginx installed
- [x] Nginx processes verified
- [x] HTTP response verified with `curl`
- [x] Container IP/network inspected
- [x] TCP port 80 verified as listening
- [x] HTTP headers checked
- [x] Nginx configuration tested

Screenshots can be added later if needed, but the commands and observed results already document the completed lab accurately.

---

## Next extension

The next useful additions to this lab are:

- create a non-root Linux user
- practice file ownership and permissions
- customize the Nginx page
- map the container port to the macOS host
- add a second container and test container-to-container communication
- document one intentional failure and recovery

---

## Completion status

**Status: Completed — Phase 1**

This lab now contains real, current hands-on evidence rather than a planned template.
