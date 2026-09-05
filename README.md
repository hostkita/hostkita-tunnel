Tentu, berikut adalah teks README yang sudah saya buat. Kamu bisa salin langsung dan tempelkan ke file `README.md` di repository GitHub-mu.

---

```markdown
# 🖥️ Hostkita

[![Docker Pulls](https://img.shields.io/badge/docker-ghcr.io-blue)](https://github.com/clickmamaheti-prog/hostkita/pkgs/container/hostkita)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Railway Deploy](https://img.shields.io/badge/Railway-Deploy-0B0D0E?logo=railway)](https://railway.app)

**Free VPS on Railway** — Ubuntu 24.04 with SSH via `bore.pub` and real‑time push notifications to your phone using [ntfy](https://ntfy.sh/).

Every time the tunnel comes online or reconnects, you receive a notification on your device with the latest SSH port — **no static IP needed**.

---

## 🚀 Quick Deploy — No Fork, No Clone

> The fastest way: deploy a ready‑to‑run image directly, without touching any code.

### 1. Open Railway
- Go to [railway.app](https://railway.app) → login → **New Project** → **Empty Project** → **Add Service** → **Docker Image**
- Enter the image:
  ```
  ghcr.io/clickmamaheti-prog/hostkita:latest
  ```

### 2. Set Environment Variables
In your Railway service → **Variables** tab, add:

| Variable      | Required | Example              | Description                           |
|---------------|:--------:|----------------------|---------------------------------------|
| `ROOT_PASS`   | ✅       | `RahasiaKuat99!`     | SSH root password — **must be changed** |
| `NTFY_TOPIC`  | ✅       | `nama-topic-unik-ku` | ntfy.sh topic for notifications       |
| `PORTS`       | ✅       | `22`                 | Ports to tunnel (default SSH)         |
| `BORE_SERVER` | ❌       | `bore.pub`           | Bore server (default works)           |
| `TZ`          | ❌       | `Asia/Jakarta`       | Container timezone                    |

### 3. Install ntfy on Your Phone
- Download **ntfy** – [Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy) / [iOS](https://apps.apple.com/app/ntfy/id1625396347)
- Subscribe to the topic you set in `NTFY_TOPIC`

### 4. SSH In
After Railway finishes deploying, you'll get a notification like:

```
🖥️ c2026 Gost tunnel linux

━━━━━━━━━━━━━━━━━━━━━━
🟢 VPS Status : ONLINE
⏱️ Uptime     : up X minutes

🌐 Host       : bore.pub
🔐 SSH        : ssh root@bore.pub -p 52341
👤 User       : root
🔑 Password   : RahasiaKuat99!

━━━━━━━━━━━━━━━━━━━━━━
```

Then connect:
```bash
ssh root@bore.pub -p <PORT_FROM_NOTIFICATION>
```

**That's it.** No need to fork or set up GitHub Actions.

---

## 🍴 Deploy with Fork (Optional — for Customisation + Auto‑Deploy)

Fork this repository if you want to:
- Customise the code / configuration
- Have GitHub Actions automatically build a new image on every `main` push
- Get ntfy notifications from GitHub Actions after deploy

### Step 1 — Fork the Repo
Click **Fork** on GitHub. Your fork will be at:
`https://github.com/<your-username>/hostkita`

### Step 2 — Deploy to Railway from GitHub
Railway → **New Project** → **Deploy from GitHub repo** → select your fork.

Railway will automatically detect the `Dockerfile` and `railway.json`.

### Step 3 — Set Railway Variables
Same as in the Quick Deploy table above.

### Step 4 — Set GitHub Secrets (for Actions)
Go to your fork → **Settings** → **Secrets and variables** → **Actions**:

| Secret                    | How to get                                                                 | Description                 |
|---------------------------|----------------------------------------------------------------------------|-----------------------------|
| `RAILWAY_TOKEN`           | Railway → Account Settings → Tokens → New token                           | Railway API token           |
| `RAILWAY_PROJECT_ID`      | Railway → project → Settings → Project ID                                 | Project ID                  |
| `RAILWAY_SERVICE_ID`      | Railway → project → service → Settings → Service ID                       | Service ID                  |
| `RAILWAY_ENVIRONMENT_ID`  | Railway → project → Environments → click env → copy from URL             | Environment ID              |
| `NTFY_TOPIC`              | Same as Railway env                                                       | ntfy topic                  |
| `ROOT_PASS`               | Same as Railway env                                                       | SSH password                |

After secrets are set, every push to `main` will:
1. Build a new Docker image
2. Push to GHCR (`ghcr.io/<username>/hostkita:latest`)
3. Trigger Railway redeploy
4. Send an ntfy notification (via GitHub Actions)

> **Without GitHub Secrets:** Railway still works normally. Notifications from inside the container (`tunnel.sh`) keep working — only the Actions‑based ones are skipped.

---

## 📲 All Notification Formats

### Tunnel Activated (startup)
```
🟢 VPS Status : ONLINE
🔐 SSH        : ssh root@bore.pub -p <PORT>
🔑 Password   : <ROOT_PASS>
```

### Reconnection (bore disconnects then reconnects)
```
🔄 VPS Status : RECONNECTED
🔐 SSH        : ssh root@bore.pub -p <NEW_PORT>
```

### Watchdog Alerts
```
⚠️ VPS Status : sshd DOWN       ← sshd crashed (Supervisor auto‑restarts)
⚠️ VPS Status : TUNNEL DOWN     ← bore crashed (Supervisor auto‑restarts)
```

---

## 🔧 Multi‑Port Tunnel

Set `PORTS` with a comma‑separated list:

```
PORTS=22,3000,8080
```

The first port (22) is always used for SSH.

---

## 🏗️ Repository Structure

```
hostkita/
├── Dockerfile                      # Ubuntu 24.04 + bore + SSH + Supervisor
├── railway.json                    # Railway deploy config + health check
├── .env.example                    # Example environment variables
├── config/
│   ├── supervisord.conf            # Supervisor (PID 1) — manages all services
│   ├── conf.d/
│   │   ├── startup.conf            # Runs startup.sh once at boot
│   │   ├── sshd.conf               # SSH daemon
│   │   ├── tunnel.conf             # Bore tunnel manager
│   │   ├── health.conf             # HTTP health check (/health)
│   │   └── cron.conf               # Cron daemon
│   └── sshd_banner.txt             # Banner shown on SSH login
└── scripts/
    ├── startup.sh                  # Init: set password, SSH port, create env file
    ├── tunnel.sh                   # Bore tunnel manager + ntfy notifications
    ├── watchdog.sh                 # Extra monitoring + ntfy alerts (optional)
    ├── health.py                   # HTTP server /health and /status
    └── notify.sh                   # Helper: send manual ntfy notifications
```

---

## 🩺 Health Check

Railway uses the `/health` endpoint to check container health:

```bash
curl https://<railway-domain>/health
```

---

## 🔍 Troubleshooting

### No ntfy notifications
- Ensure `NTFY_TOPIC` in Railway **exactly matches** the topic in the ntfy app.
- Test manually after SSH:
  ```bash
  curl -d "Test" "https://ntfy.sh/<NTFY_TOPIC>"
  ```
- Check Railway logs for `[tunnel] ntfy sent` or `[tunnel] ntfy failed`.

### SSH "Connection refused"
- The bore port **changes on every Railway restart** — check the latest notification or Railway logs.
- Look for `Tunnel ready: local :22 → bore.pub:XXXXX` in the logs.

### Container keeps restarting
- Ensure `ROOT_PASS`, `NTFY_TOPIC`, and `PORTS` are set.
- Health check may take ~30 seconds after the container starts.

### GitHub Actions failing
- Verify all 4 Railway secrets are correct.
- The workflow uses `continue-on-error: true` — it won't fail the build if one secret is wrong.

---

## 📋 Changelog

### v3.3
- ntfy notification format changed to structured status card
- GitHub Actions no longer crash on Railway API failures (`continue-on-error`)

### v3.2
- Fix: ntfy in Railway container uses `--retry 5 --retry-delay 3`
- Fix: replaced tmpfile with direct `-d body`

### v3.1
- Fix: capture bore stdout to parse real port (not `???`)

### v3.0
- Supervisor as PID 1
- Multi‑port bore tunnel support
- Health check endpoint

---

## 📄 License

MIT © [clickmamaheti-prog](https://github.com/clickmamaheti-prog)
```

---