# Linux Server Lab

## Objective

Build and document a lightweight Linux administration lab using an Ubuntu 24.04 Docker container. The goal is to demonstrate practical Linux administration, networking, service deployment, process inspection, user/permission management, container port publishing, and troubleshooting with current hands-on evidence.

---

## Lab environment

| Component | Configuration |
|---|---|
| Host | macOS with Docker Desktop |
| Linux environment | Ubuntu 24.04 container |
| Container name | `linux-lab` |
| Web service | Nginx 1.24.0 |
| Network | Docker bridge networking |
| Published port | Host `8080` → Container `80` |

The lab runs inside a container rather than a full virtual machine. This keeps the environment lightweight while still providing a practical space for Linux administration, web service deployment, networking, and troubleshooting.

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

### 1. Create a custom web page

Inside the Ubuntu container, replace the default Nginx page with a simple lab page:

```bash
echo '<h1>Turki Infrastructure Lab</h1><p>Nginx running inside Docker on Ubuntu 24.04.</p>' > /var/www/html/index.html
```

### 2. Recreate the container with published port mapping

From the macOS host:

```bash
docker stop linux-lab
docker rm linux-lab
docker run -it --name linux-lab -p 8080:80 ubuntu:24.04 bash
```

This maps TCP port `8080` on the Mac host to TCP port `80` inside the Ubuntu container.

### 3. Install and start Nginx in the recreated container

```bash
apt update
apt install nginx -y
```

Create the custom page again:

```bash
echo '<h1>Turki Infrastructure Lab</h1><p>Nginx running inside Docker on Ubuntu 24.04.</p>' > /var/www/html/index.html
```

Start Nginx:

```bash
nginx
```

### 4. Verify access from the host browser

The page was successfully opened from the macOS host at:

```text
http://localhost:8080
```

This verifies host-to-container connectivity through Docker port publishing and confirms that Nginx inside the container is reachable from outside the container namespace.

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
- custom web content deployment
- process inspection with `ps`
- local HTTP testing with `curl`
- IP/network inspection with `ip addr`
- socket and port inspection with `ss`
- Docker bridge networking
- Docker host-to-container port publishing
- host-to-container connectivity testing
- Linux user creation
- file and directory ownership
- permission-aware file access
- switching between root and non-root users
- basic service and shell troubleshooting
- understanding the separation between a container and its Docker host

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
- [x] Web service opened successfully from the macOS host browser

Screenshots can be added later if needed. A browser screenshot of the custom page would be the strongest visual proof for this phase.

---

## Next extension

The next useful additions to this lab are:

- create a user-defined Docker network
- add a second container
- verify container-to-container DNS and HTTP communication
- intentionally break connectivity or service configuration and recover it
- optionally add persistent storage with a Docker volume

---

## Completion status

**Status: Completed — Phase 3**

This lab now demonstrates Linux administration, Nginx service deployment, user and permission management, and practical Docker networking from host to container.
