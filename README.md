# wahoo-fit-sync

`wahoo-fit-sync` is a lightweight, containerized tool that connects to the [Wahoo Fitness Cloud API](https://developers.wahooligan.com/) to automatically download your workout activities as `.FIT` files into a local directory.

It features **incremental syncing** and **deduplication**, ensuring only new workouts are downloaded and previously downloaded activities are never re-downloaded.

---

## 🌟 Key Features

* **Incremental Sync**: Scans your Wahoo workout history and downloads only new/recent activities.
* **Deduplication**: Tracks downloaded workout IDs in a persistent JSON database (`/data/config/sync_history.json`) and skips existing files.
* **Automatic Interval Syncing**: Background daemon mode automatically checks for new workouts every X minutes (e.g. `SYNC_INTERVAL_MINUTES=60`).
* **Built-in Web Interface**: Includes a clean dashboard at `http://localhost:8080` for one-click OAuth authorization, viewing sync status, and triggering manual syncs on demand.
* **Persistent Token Refresh**: Automatically handles OAuth 2.0 access token renewals using saved refresh tokens without requiring repeated logins.
* **Docker Ready**: Pre-configured with `Dockerfile` and `docker-compose.yml` mapping `./activities` for `.FIT` file storage.

---

## 🚀 Quick Start Guide

### Step 1: Register Wahoo Developer Application

1. Go to the [Wahoo Developer Portal](https://developers.wahooligan.com/applications) and sign in.
2. Click **Create Application**.
3. Fill in the required fields:
   * **App Name**: `wahoo-fit-sync` (or your preferred name)
   * **Redirect URI**: `http://localhost:8080/callback` *(or your server's custom domain/IP)*
   * **Webhook URI**: Leave blank (or re-enter `http://localhost:8080/callback` if required)
4. Submit the application and copy your **Client ID** and **Client Secret**.

---

### Step 2: Configure Environment Variables

Create or edit the `.env` file in the project root:

```ini
WAHOO_CLIENT_ID=your_client_id_here
WAHOO_CLIENT_SECRET=your_client_secret_here
WAHOO_REDIRECT_URI=http://localhost:8080/callback
SYNC_INTERVAL_MINUTES=60
PORT=8080
```

---

### Step 3: Start with Docker Compose

Run the container using Docker Compose:

```bash
docker compose up -d --build
```

Or using standard Docker CLI:

```bash
docker build -t wahoo-fit-sync .
docker run -d \
  --name wahoo-fit-sync \
  -p 8080:8080 \
  --env-file .env \
  -v $(pwd)/config:/data/config \
  -v $(pwd)/activities:/data/downloads \
  wahoo-fit-sync
```

---

### Step 4: One-Click Authentication

1. Open your web browser and navigate to **`http://localhost:8080`**.
2. Click the **Connect Wahoo Account** button.
3. Authorize the application on Wahoo's website.
4. You will be redirected back to the dashboard, and `wahoo-fit-sync` will immediately perform its initial sync!

All downloaded `.FIT` files will be saved in the `./activities` folder on your host machine.

---

## ⚙️ Configuration Options

| Environment Variable | Default | Description |
| :--- | :--- | :--- |
| `WAHOO_CLIENT_ID` | *Required* | Client ID from Wahoo Developer Portal |
| `WAHOO_CLIENT_SECRET` | *Required* | Client Secret from Wahoo Developer Portal |
| `WAHOO_REDIRECT_URI` | `http://localhost:8080/callback` | OAuth redirect URI matching Wahoo App settings |
| `SYNC_INTERVAL_MINUTES` | `60` | Background sync frequency in minutes (`0` disables background scheduler) |
| `PORT` | `8080` | Port for web server dashboard & OAuth callback |
| `DATA_DIR` | `/data` | Internal container path for config and downloads |

---

## 📁 Directory Structure

```text
wahoo-fit-sync/
├── activities/            # Local directory where .FIT files are saved
│   ├── 2026-07-28_workout_1234567.fit
│   └── 2026-07-25_workout_1234566.fit
├── config/                # Persistent authentication tokens & sync history
│   ├── tokens.json
│   └── sync_history.json
├── app/
│   ├── __init__.py
│   ├── main.py            # Flask Web UI & server routes
│   ├── scheduler.py       # Background interval sync scheduler
│   ├── sync.py            # Incremental sync & deduplication logic
│   └── wahoo_client.py    # Wahoo API & OAuth client
├── .env                   # Local configuration
├── docker-compose.yml
├── Dockerfile
└── requirements.txt
```

---

## 🧪 License

MIT License. Free for personal and non-commercial use.
