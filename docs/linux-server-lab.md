# Linux Server Lab

## Objective

Build and document a lightweight Linux administration lab using an Ubuntu 24.04 Docker container. The goal is to demonstrate practical Linux administration, networking, service deployment, process inspection, user/permission management, container networking, and troubleshooting with current hands-on evidence.

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

### 1. Create a non-root user

```bash
useradd -m labuser
passwd labuser
```

A new user named `labuser` was created with its own home directory and password.

### 2. Create a shared test directory and file

```bash
mkdir /lab-data
touch /lab-data/test.txt
```

Initial ownership:

```text
-rw-r--r-- 1 root root 0 Oct 6 10:00 test.txt
```

This showed that the file was initially owned by `root`.

### 3. Change ownership to the non-root user

```bash
chown -R labuser:labuser /lab-data
```

Ownership after the change:

```text
-rw-r--r-- 1 labuser labuser 0 Oct 6 10:00 test.txt
```

This confirms the file and directory were successfully reassigned to the new user and group.

### 4. Switch to the non-root user

```bash
su - labuser
```

### 5. Write to the file as `labuser`

```bash
echo "Linux permissions lab" > /lab-data/test.txt
cat /lab-data/test.txt
```

Observed result:

```text
Linux permissions lab
```

This confirms the user could modify the file after ownership was changed.

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

From the macOS host:

```bash
docker network create lab-network
```

The network was created successfully.

### 2. Start an Nginx server on the custom network

```bash
docker run -dit --name web-server --network lab-network nginx
```

Docker pulled the Nginx image and started the `web-server` container successfully.

### 3. Start a client container on the same network

```bash
docker run -it --name client --network lab-network ubuntu:24.04 bash
```

Inside the client container, the required tools were installed:

```bash
apt update
apt install curl iproute2 -y
```

### 4. Test HTTP communication by container name

```bash
curl http://web-server
```

Observed result: the request returned the default Nginx HTML page.

This confirms that the `client` container can reach the `web-server` container over the custom Docker network.

### 5. Verify Docker DNS resolution

```bash
getent hosts web-server
```

Observed result:

```text
172.18.0.2      web-server
```

This confirms Docker's built-in DNS resolved the service name `web-server` to the container's network IP address.

---

## Troubleshooting performed

### Issue 1 — Nginx installed but HTTP request failed

Nginx was installed but not running. Starting it manually with `nginx` resolved the issue.

### Issue 2 — Docker command unavailable inside the container

`docker --version` returned `command not found` inside Ubuntu. Docker runs on the macOS host, not inside the container.

### Issue 3 — Mistyped shell command

`exsit` was typed instead of `exit`; the shell returned `command not found` and the correct command was entered.

### Issue 4 — Duplicate Docker container name

Attempting to run `web-server` a second time returned:

```text
Conflict. The container name "/web-server" is already in use
```

The original `web-server` container had already been created successfully, so there was no need to create it again.

### Issue 5 — Linux networking commands run on macOS host

Commands such as `apt`, `getent`, and `ip addr` were initially run from the macOS shell and returned `command not found` or failed DNS resolution.

The commands were then run inside the Ubuntu `client` container, where they worked correctly.

**Lesson:** Host commands and container commands must be run in the correct environment.

---

## Skills demonstrated

This lab now provides current hands-on evidence of:

- Ubuntu/Linux command-line administration
- package installation with APT
- Nginx installation and startup
- Nginx configuration validation
- process inspection with `ps`
- local HTTP testing with `curl`
- IP/network inspection with `ip addr`
- socket and port inspection with `ss`
- Docker bridge networking
- Docker user-defined networks
- host-to-container port publishing
- container-to-container HTTP communication
- Docker internal DNS/service-name resolution
- Linux user creation
- file and directory ownership
- permission-aware file access
- switching between root and non-root users
- basic service, shell, and container troubleshooting

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
- [x] Non-root user created
- [x] File ownership changed from root to `labuser`
- [x] Non-root write access verified
- [x] Custom Nginx page created
- [x] Docker port `8080:80` published
- [x] Web service opened from macOS host
- [x] Custom Docker network created
- [x] Second container created
- [x] Container-to-container HTTP communication verified
- [x] Docker DNS resolution verified (`web-server` → `172.18.0.2`)

---

## Final extension

One final phase remains to make the lab complete:

- intentionally break container connectivity or a service
- diagnose the failure
- restore service
- optionally demonstrate persistent storage with a Docker volume

---

## Completion status

**Status: Completed — Phase 4**

The lab now demonstrates Linux administration, Nginx service deployment, user/permission management, host-to-container networking, and container-to-container Docker networking with real troubleshooting evidence.
