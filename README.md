# uptime-kuma-docker-monitor
Guide how to add your host into monitoring Uptime-kuma



# Enabling Remote Docker API over TCP (Port 2375) with UFW Protection

This README explains how to **expose the Docker API over TCP (port 2375)** while keeping it **secure using UFW**, so only trusted hosts (e.g., your monitoring server such as Uptime Kuma) can access it.

> ⚠️ **WARNING:** `tcp://0.0.0.0:2375` is a fully unprotected Docker API endpoint (no TLS, no authentication).
> Never expose this port directly to the public internet.
> Access must be restricted using a firewall (UFW) or a private internal network.

---

## 1. Allow SSH and restrict access to port 2375 with UFW

First, ensure SSH access remains open.
Then allow only your trusted IP or subnet to access port 2375.

```bash
sudo ufw allow from < ip of your network > to any port 2375
sudo ufw enable
sudo ufw reload
```

**Explanation:**

* `sudo ufw allow from < ip of your network > to any port 2375`
  Replace `< ip of your network >` with:

  * a single IP (e.g., `10.10.0.5`), or
  * a subnet (e.g., `192.168.0.0/24`).
* `sudo ufw enable` — activates the firewall.
* `sudo ufw reload` — reloads the firewall rules.

---

## 2. Enable Docker API over TCP in the systemd service file

Now configure Docker to listen on both the Unix socket and TCP port 2375.

Open the Docker unit file:

```bash
sudo nano /lib/systemd/system/docker.service
```

Find the line starting with `ExecStart=` and replace it with:

```bash
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 --containerd=/run/containerd/containerd.sock
```

**What this does:**

* `-H unix:///var/run/docker.sock` — keeps local Docker functionality.
* `-H tcp://0.0.0.0:2375` — enables remote API on TCP.
* `--containerd=…` — standard containerd connection.

Save and exit the editor.

Reload systemd and restart Docker:

```bash
systemctl daemon-reload
sudo systemctl restart docker
```

---

## 3. Verify Docker is listening on port 2375

Run:

```bash
sudo netstat -tuln | grep 2375
```

Expected output:

```
tcp        0      0 0.0.0.0:2375            0.0.0.0:*               LISTEN
```

This confirms that the Docker TCP endpoint is active and protected by UFW.

---

## 4. Connecting external services (e.g., Uptime Kuma)

Use the following URL for remote access:

```
http://<your-docker-host-ip>:2375
```

Example for Uptime Kuma:

* **Connection Type:** HTTP
* **Docker Daemon URL:** `http://172.2.2.2:2375`

Ensure your client IP is permitted by UFW rules.

---

## 5. Security recommendations

* **Never expose port 2375 publicly** — it grants full root-level Docker access.
* Restrict access strictly via UFW.
* For production environments, consider:

  * Setting up Docker API with TLS (port 2376),
  * Using a VPN (WireGuard/OpenVPN),
  * Using SSH socket forwarding instead of a direct TCP port.

---

This README provides a clean and minimal setup for controlled environments where a remote Docker API is required and firewall-restricted access is acceptable.
