# Linux Server Lab

## Objective

Build and document a small Linux server environment that demonstrates practical system administration, networking, service deployment, firewall configuration, Docker, and troubleshooting.

This case study will be completed using a fresh Ubuntu Server virtual machine so the repository contains current, reproducible proof of hands-on work.

---

## Lab specification

| Component | Configuration |
|---|---|
| OS | Ubuntu Server 24.04 LTS |
| CPU | 2 vCPU |
| Memory | 2 GB RAM |
| Disk | 20 GB |
| Network | NAT or Bridged |
| Services | SSH, Nginx, UFW, Docker |

---

## Build checklist

### 1. Update the server

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install core services

```bash
sudo apt install nginx openssh-server ufw -y
```

### 3. Verify services

```bash
systemctl status nginx
systemctl status ssh
```

### 4. Configure the firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

### 5. Check network configuration

```bash
ip addr
```

Open the server IP in a browser:

```text
http://SERVER_IP
```

### 6. Install Docker

```bash
sudo apt install docker.io -y
sudo systemctl enable --now docker
```

### 7. Test Docker

```bash
sudo docker run hello-world
```

### 8. Inspect service logs

```bash
journalctl -u nginx --no-pager | tail -20
```

---

## Evidence to capture

Add screenshots only after completing the lab.

Recommended evidence:

- [ ] Ubuntu Server VM running
- [ ] `ip addr` output
- [ ] Nginx default page in browser
- [ ] `ufw status`
- [ ] Docker `hello-world` output
- [ ] `systemctl status nginx`
- [ ] Nginx service logs

Store screenshots in:

```text
screenshots/linux-server/
```

Do not publish passwords, private keys, tokens, public IPs you do not want exposed, or any other sensitive information.

---

## What this lab demonstrates

Once completed, this lab will provide evidence of hands-on work with:

- Ubuntu Server administration
- Linux package management
- systemd service management
- SSH
- Nginx
- host firewall configuration with UFW
- IP/network inspection
- Docker installation and container execution
- Linux logs and basic troubleshooting

---

## Troubleshooting notes

Document real issues encountered during the build here rather than inventing examples.

### Issue 1

**Problem:** Pending

**Investigation:** Pending

**Resolution:** Pending

**Lesson:** Pending

---

## Completion status

**Status:** Planned / ready to build

This page will be updated with real outputs, screenshots, and lessons after the lab is completed.
