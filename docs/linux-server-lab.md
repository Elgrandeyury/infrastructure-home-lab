# Linux Server Lab

## Objective

Build and document a lightweight Linux administration lab using an Ubuntu 24.04 Docker container. The goal is to demonstrate practical Linux administration, networking, service deployment, process inspection, user/permission management, and troubleshooting with current hands-on evidence.

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

### Issue 3 — Mistyped shell command

**Problem**  
While exiting the `labuser` shell, `exsit` was typed instead of `exit`.

**Resolution**  
The correct command was entered:

```bash
exit
```

**Lesson**  
Small command-line mistakes are easy to diagnose when the shell returns a clear `command not found` message.

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
- Linux user creation
- file and directory ownership
- permission-aware file access
- switching between root and non-root users
- basic service and shell troubleshooting
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
- [x] Non-root user created
- [x] File ownership changed from root to `labuser`
- [x] Non-root write access verified

Screenshots can be added later if needed, but the commands and observed results already document the completed lab accurately.

---

## Next extension

The next useful additions to this lab are:

- customize the Nginx page
- map the container port to the macOS host
- open the page from the Mac browser
- add a second container and test container-to-container communication
- intentionally break a service or network configuration and recover it

---

## Completion status

**Status: Completed — Phase 2**

This lab now includes real hands-on evidence for both Linux service administration and user/permission management.
