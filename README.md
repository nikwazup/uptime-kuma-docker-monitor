# uptime-kuma-docker-monitor
Guide how to add your host into monitoring Uptime-kuma



# Enabling Remote Docker API over TCP (Port 2375) with UFW Protection

This README explains how to **expose the Docker API over TCP (port 2375)** while keeping it **secure using UFW**, so only trusted hosts (e.g., your monitoring server such as Uptime Kuma) can access it.

> ⚠️ WARNING: `tcp://0.0.0.0:2375` is a **fully unprotected Docker API endpoint** (no TLS, no authentication).  
> Never expose this port to the public internet.  
> Access *must* be restricted using a firewall (UFW) or a private network.

---

## 1. Allow SSH and restrict access to port 2375 with UFW

First, make sure SSH remains accessible.  
Then allow only your trusted IP/subnet to access port 2375.

```bash
sudo ufw allow ssh
sudo ufw allow from < ip of your network > to any port 2375
sudo ufw enable
sudo ufw reload

