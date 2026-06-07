# 🖥️ HomeServer Lab: From Legacy Hardware to Modern Docker Environment

A detailed documentation of my journey setting up a lightweight, production-ready home server using a legacy Intel H61 motherboard, 8GB RAM, and a 112GB SSD — no new hardware required.

## 🎯 Project Goals

Transform older hardware into a self-hosted stack:

- 🎬 **Jellyfin** — Private Netflix-style media server
- ☁️ **Nextcloud** — Personal cloud storage (Google Drive replacement)
- 🌐 **Apache / Portfolio** — Self-hosted web development environment
- 🔒 **Tailscale** — Secure remote access without opening router ports
- 📦 **CasaOS + Portainer** — Lightweight Docker management GUI

---

## 🔧 Hardware Stack

| Component | Spec |
|-----------|------|
| Motherboard | Intel H61 Series |
| CPU | Intel Pentium (2nd/3rd Gen) |
| RAM | 8GB DDR3 |
| Storage | 112GB SATA SSD |
| OS | Debian 13 "Trixie" — Minimal/Netinst (No GUI) |

This severely limited hardware expansion. A planned **NVIDIA GTX 1050 / GTX 1650** upgrade for Jellyfin NVENC hardware transcoding was **abandoned** — those GPUs draw up to 75W under load, which would have pushed the system dangerously over the 80W total output limit and risked hardware failure.

**Decision:** Run CPU-based transcoding only. Jellyfin handles this adequately for 1–2 streams at 1080p on the Pentium.

---

## 🧠 Architecture Decisions

### Why NOT Proxmox?

Proxmox was the initial plan for running isolated VMs. It was abandoned because:

- The hypervisor itself consumes ~1–2GB RAM at idle
- With only 8GB total, VM overhead left too little headroom for Jellyfin + Nextcloud + other services running simultaneously
- Bare-metal Debian + Docker achieves the same isolation with a fraction of the RAM cost

### Why Debian 13 (Trixie) Minimal?

- Zero GUI overhead — no GNOME, no Xorg, no desktop bloat
- Ships with modern kernel and `systemd`
- Stable, well-documented, and Docker-compatible out of the box
- Netinst ISO is ~400MB — only installs what you explicitly choose

### Why CasaOS instead of raw Docker?

- Provides a clean web UI for monitoring system resources, Docker containers, and disk usage
- App Store with one-click installs for Jellyfin, Nextcloud, etc.
- Runs entirely in Docker itself — minimal RAM footprint (~100MB)
- Portainer is used alongside it for advanced Docker Compose stack management

---

## 📋 Installation & Configuration — Step by Step

### Step 1: OS Installation — The "Single Partition" Pivot

**First attempt (wrong):** Let the Debian installer auto-partition. It created separate partitions:

```
/var    →   4GB   ← Docker installs to /var/lib/docker — immediately hit the wall
/       →  10GB
/srv    →  95GB   ← Space sitting unused
```

This caused Docker container pulls and Nextcloud data to crash into the 4GB `/var` ceiling while 95GB sat idle on `/srv`.

**Fix — Reinstalled with correct partitioning:**

During the Debian installer, at the partitioning screen:

1. Select **"Guided — use entire disk"**
2. Choose **"All files in one partition"**

This creates a single `~100GB+` root partition at `/dev/sda1` mounted at `/`. Docker, Nextcloud data, Jellyfin media — everything draws from the same pool.

**Other install settings used:**

| Setting | Value |
|--------|-------|
| Hostname | `homeserver` |
| Domain name | *(left blank — avoids local DNS confusion)* |
| Root password | *(set separately)* |
| Non-root user | `badal` |
| Software selection | ✅ SSH Server, ✅ Standard System Utilities only |
| Unchecked | ❌ Debian Desktop Environment, ❌ GNOME |

---

### Step 2: Fix the SSH "Man-in-the-Middle" Warning

After reinstalling the OS, trying to SSH back in from the client PC threw:

```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

This is expected and harmless — the server's SSH identity key changed because it was reinstalled. The client still has the old key cached.

**Fix (run on the CLIENT machine, not the server):**

```bash
ssh-keygen -f "/home/badal/.ssh/known_hosts" -R "192.168.100.185"
```

Replace `192.168.100.185` with your server's actual local IP. Then SSH in again normally — it will prompt you to accept the new key fingerprint.

---

### Step 3: Install sudo and curl (Minimal Debian doesn't include them)

Debian Netinst ships with almost nothing. Running the CasaOS install script immediately fails:

```
sudo: command not found
curl: command not found
```

**Fix:**

```bash
# 1. Switch to root
su -

# 2. Update package lists
apt update

# 3. Install required tools
apt install sudo curl -y

# 4. Add your user to the sudo group
usermod -aG sudo badal

# 5. Exit root and log back in to apply the group change
exit
# Then SSH back in as 'badal'
```

Verify it worked:

```bash
sudo whoami
# Should output: root
```

---

### Step 4: Install CasaOS

CasaOS acts as the main dashboard — it provides a GUI for managing Docker apps, monitoring RAM/CPU/storage, and installing services from its App Store.

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

Once installed, access the dashboard at:

```
http://<your-server-local-ip>:80
```

Example: `http://192.168.100.185`

> **Note:** CasaOS will guide you through creating a dashboard account on first launch.

---

### Step 5: Install Tailscale (Secure Remote Access)

Tailscale creates an encrypted mesh VPN between your devices. You can SSH into or access the server from anywhere in the world — without opening a single port on your router.

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Authenticate and bring the node online:

```bash
sudo tailscale up
```

This outputs a URL — open it in a browser and sign in with your Google/GitHub/Microsoft account. After authentication, your server gets a stable Tailscale IP (e.g., `100.x.x.x`) that works from any network.

**Verify the connection:**

```bash
tailscale status
```

You can now SSH to your server from a phone or laptop using the Tailscale IP even on mobile data:

```bash
ssh badal@100.x.x.x
```

---

### Step 6: Install Portainer

Portainer is installed from the **CasaOS App Store** (one-click). It provides advanced Docker management — custom Docker Compose stacks, container logs, network management, and environment variables.

Access it at:

```
http://<server-ip>:9000
```

Use Portainer for anything that needs a custom `docker-compose.yml` (e.g., deploying an Apache web server for a portfolio site).

---

### Step 7: Install Jellyfin & Nextcloud

Both are installed via the **CasaOS App Store** (one-click install).

| Service | Default Port | Purpose |
|---------|-------------|---------|
| Jellyfin | `8096` | Media server (movies, TV, music) |
| Nextcloud | `your-chosen-port` | Private cloud storage + office suite |

Access after install:

- Jellyfin: `http://<server-ip>:8096`
- Nextcloud: `http://<server-ip>:<port>`

> **Jellyfin transcoding note:** Hardware transcoding (NVENC) is unavailable due to the 80W PSU constraint. Software transcoding is enabled instead — adequate for 1–2 simultaneous 1080p streams.

---

## 🌐 Network Topology

```
Internet
    │
    └── Router (Home Network: 192.168.100.x)
            │
            ├── Client PC (SSH client)
            │
            └── Home Server (192.168.100.185)
                    │
                    ├── CasaOS Dashboard     :80
                    ├── Portainer            :9000
                    ├── Jellyfin             :8096
                    └── Nextcloud            :your-port

Remote Access (any network, any device):
    └── Tailscale VPN → Server Tailscale IP (100.x.x.x)
```

---

## 📦 Services Summary

| Service | Install Method | Port | Purpose |
|---------|---------------|------|---------|
| CasaOS | `curl` script | 80 | Docker GUI + App Store |
| Tailscale | `curl` script | — | Mesh VPN / Remote Access |
| Portainer | CasaOS App Store | 9000 | Advanced Docker management |
| Jellyfin | CasaOS App Store | 8096 | Media streaming |
| Nextcloud | CasaOS App Store | — | Private cloud |

---

## 🚧 Current Status & Roadmap

- [x] Debian 13 minimal install (single partition)
- [x] SSH configured and tested
- [x] sudo + curl installed
- [x] CasaOS installed and accessible
- [x] Tailscale configured (remote access working)
- [x] Portainer installed
- [ ] Jellyfin configured with media library
- [ ] Nextcloud configured with external storage
- [ ] Apache portfolio site deployed via Portainer Docker Compose
- [ ] Domain + HTTPS via Cloudflare Tunnel or Nginx Proxy Manager
- [ ] Automated backups (Nextcloud data + config)

---

## 💡 Lessons Learned

- **Always use "All files in one partition"** on a single-drive server. Separate partitions create invisible walls Docker will hit immediately.
- **Debian Netinst is genuinely minimal** — `sudo`, `curl`, `vim` are all missing. Budget 5 minutes to install essentials before anything else.
- **The SSH host key warning after reinstall is harmless** — just clear the old key with `ssh-keygen -R`.
- **8GB RAM is workable** — but only if you avoid hypervisor overhead. Bare metal + Docker is the right call here.
- **Check your PSU before adding any GPU** — 80W total output leaves almost no headroom beyond CPU + SSD + RAM.

---

## 📝 References

- [CasaOS Official](https://casaos.io)
- [Tailscale Docs](https://tailscale.com/kb)
- [Jellyfin Documentation](https://jellyfin.org/docs)
- [Nextcloud Hub](https://nextcloud.com)
- [Portainer Documentation](https://docs.portainer.io)
- [Debian Installation Guide](https://www.debian.org/releases/stable/installmanual)
