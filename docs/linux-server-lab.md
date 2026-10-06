# Linux Server Lab

## Objective

Build and document a lightweight Linux administration lab using Ubuntu 24.04 containers. The goal is to demonstrate practical Linux administration, networking, service deployment, process inspection, user/permission management, container networking, and troubleshooting with current hands-on evidence.

---

## Lab environment

| Component | Configuration |
|---|---|
| Host | macOS with Docker Desktop |
| Linux environment | Ubuntu 24.04 containers |
| Web service | Nginx |
| Default network | Docker bridge |
| Custom network | `lab-network` |
| Published port | Host `8080` → Container `80` |

The lab runs inside containers rather than a full virtual machine. This keeps the environment lightweight while still providing a practical space for Linux administration, web service deployment, Docker networking, and troubleshooting.

---

## Phase 1 — Linux service and networking basics

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

Observed result:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

## Phase 2 — Linux users, ownership, and permissions

A non-root user was created and used to verify file ownership and write access.

```bash
useradd -m labuser
passwd labuser
mkdir /lab-data
touch /lab-data/test.txt
chown -R labuser:labuser /lab-data
su - labuser
```

Observed ownership change:

```text
-rw-r--r-- 1 root root 0 Oct 6 10:00 test.txt
-rw-r--r-- 1 labuser labuser 0 Oct 6 10:00 test.txt
```

Write access was verified with:

```bash
echo "Linux permissions lab" > /lab-data/test.txt
cat /lab-data/test.txt
```

Observed result:

```text
Linux permissions lab
```

---

## Phase 3 — Custom Nginx page and host-to-container access

A custom Nginx page was created and exposed from the container to the macOS host using Docker port publishing.

```bash
docker run -it --name linux-lab -p 8080:80 ubuntu:24.04 bash
```

The custom page was accessible from the host at:

```text
http://localhost:8080
```

This verified host-to-container connectivity through Docker port publishing.

---

## Phase 4 — Container-to-container networking

### 1. Create a user-defined Docker network

```bash
docker network create lab-network
```

### 2. Start an Nginx server on the custom network

```bash
docker run -dit --name web-server --network lab-network nginx
```

### 3. Start a client container on the same network

```bash
docker run -it --name client --network lab-network ubuntu:24.04 bash
```

Inside the client container:

```bash
apt update
apt install curl iproute2 -y
```

### 4. Test HTTP communication by container name

```bash
curl http://web-server
```

Observed result: the request returned the default Nginx HTML page.

### 5. Verify Docker DNS resolution

```bash
getent hosts web-server
```

Observed result:

```text
172.18.0.2      web-server
```

This confirmed container-to-container HTTP communication and Docker internal DNS resolution.

---

## Phase 5 — Failure, diagnosis, and recovery

A real connectivity failure was created intentionally to prove troubleshooting and recovery capability.

### 1. Break connectivity

From the macOS host:

```bash
docker network disconnect lab-network web-server
```

The `web-server` container was removed from the custom Docker network.

### 2. Test from the client container

```bash
docker start -ai client
curl http://web-server
```

Observed failure:

```text
curl: (6) Could not resolve host: web-server
```

This showed that the client could no longer resolve or reach the `web-server` service because the server was no longer attached to `lab-network`.

### 3. Restore connectivity

After exiting the client, reconnect the server from the macOS host:

```bash
docker network connect lab-network web-server
```

### 4. Verify recovery

Re-enter the client container:

```bash
docker start -ai client
curl http://web-server
```

Observed result: the Nginx HTML page was returned successfully again.

### What this proves

This test demonstrates a complete troubleshooting cycle:

1. known-good communication
2. intentional network failure
3. observable DNS/connectivity error
4. network configuration repair
5. successful service recovery

---

## Troubleshooting performed

### Issue 1 — Nginx installed but HTTP request failed

Nginx was installed but not running. Starting it manually with `nginx` resolved the issue.

### Issue 2 — Docker command unavailable inside the container

`docker --version` returned `command not found` inside Ubuntu. Docker runs on the macOS host, not inside the container.

### Issue 3 — Mistyped shell command

`exsit` was typed instead of `exit`; the shell returned `command not found` and the correct command was entered.

### Issue 4 — Duplicate Docker container name

Attempting to run `web-server` a second time returned a container name conflict. The original container had already been created successfully.

### Issue 5 — Linux networking commands run on macOS host

Commands such as `apt`, `getent`, and `ip addr` were initially run from the macOS shell. They were then run inside the Ubuntu client container, where they worked correctly.

### Issue 6 — Container DNS/connectivity failure

Disconnecting `web-server` from `lab-network` caused:

```text
curl: (6) Could not resolve host: web-server
```

Reconnecting the server to the network restored DNS resolution and HTTP communication immediately.

---

## Skills demonstrated

This lab provides current hands-on evidence of:

- Ubuntu/Linux command-line administration
- APT package management
- Nginx installation and startup
- Nginx configuration validation
- process inspection
- HTTP testing with `curl`
- IP/network inspection
- socket and port inspection
- Linux user creation
- file ownership and permissions
- root vs non-root access
- Docker bridge networking
- Docker user-defined networks
- host-to-container port publishing
- container-to-container HTTP communication
- Docker internal DNS/service-name resolution
- intentional failure testing
- connectivity diagnosis
- network recovery
- basic service, shell, and container troubleshooting

---

## Evidence status

- [x] Ubuntu 24.04 container running
- [x] Nginx installed and verified
- [x] Nginx processes inspected
- [x] HTTP response verified
- [x] Container IP/network inspected
- [x] TCP port 80 verified
- [x] Nginx configuration validated
- [x] Non-root user created
- [x] Ownership and write access verified
- [x] Custom Nginx page created
- [x] Host port `8080:80` published
- [x] Web service accessed from macOS host
- [x] User-defined Docker network created
- [x] Second container created
- [x] Container-to-container HTTP verified
- [x] Docker DNS resolution verified
- [x] Connectivity intentionally broken
- [x] Failure observed and diagnosed
- [x] Network restored
- [x] Service communication recovered

---

## Completion status

**Status: Completed**

The lab now demonstrates Linux administration, service deployment, permissions, Docker networking, DNS-based container communication, and a complete failure-to-recovery troubleshooting workflow with real hands-on evidence.
