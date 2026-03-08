# KingShot Auto Gift Code Redeemer

Automatically detects new gift codes from kingshot.net and redeems them
for all your players on ks-giftcode.centurygame.com — fully hands-free.

---

## How it works

```
Every 15 minutes:
  → Fetch https://kingshot.net/api/gift-codes
  → Compare with seen_codes.json
  → If NEW code found:
      → Loop through all playerIDs.txt
      → Selenium redeems the code on centurygame.com per player
      → Log results
      → Mark code as done (won't retry same code)
```

---

## Setup

### 1. Install Python dependencies
```bash
pip install -r requirements.txt
```

### 2. Install Google Chrome
The script uses Chrome in headless mode. Install it:
- **Ubuntu/Debian:**
  ```bash
  wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
  echo "deb http://dl.google.com/linux/chrome/deb/ stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list
  sudo apt update && sudo apt install -y google-chrome-stable
  ```
- **Windows/Mac:** Download from https://www.google.com/chrome/

### 3. Add your players
Edit `playerIDs.txt` — one player per line:
```
8767319   SgtSlayer
1234567   WarriorKing
```
(The name is optional, just for logging)

### 4. Run
```bash
python main.py
```

---

## Files

| File | Purpose |
|------|---------|
| `main.py` | Scheduler — polls API, detects new codes |
| `redeemer.py` | Selenium logic — redeems on centurygame.com |
| `playerIDs.txt` | Your player IDs (you edit this) |
| `seen_codes.json` | Auto-created — tracks redeemed codes |
| `logs/` | Daily log files |
| `screenshots/` | Auto-saved on errors for debugging |

---

## Deployment (24/7 running)

### Option A — DigitalOcean Droplet (Recommended, ~$4/month)
1. Create a $4/mo Ubuntu 24.04 droplet at https://digitalocean.com
2. SSH in, clone/upload your files
3. Install Chrome + Python deps (see above)
4. Run with `screen` or `nohup` so it survives logout:
   ```bash
   # Using screen (easiest):
   screen -S kingshot
   python main.py
   # Press Ctrl+A then D to detach
   # Reattach anytime: screen -r kingshot
   ```
5. Or set it up as a systemd service (see below) for auto-restart on reboot

### Option B — systemd service (best for VPS, auto-restarts)
Create `/etc/systemd/system/kingshot.service`:
```ini
[Unit]
Description=KingShot Auto Gift Code Redeemer
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/kingshot_auto
ExecStart=/usr/bin/python3 /home/ubuntu/kingshot_auto/main.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
Then:
```bash
sudo systemctl enable kingshot
sudo systemctl start kingshot
sudo systemctl status kingshot   # check it's running
```

### Option C — Railway.app (Free tier, easiest)
1. Push your code to a GitHub repo
2. Connect repo to https://railway.app
3. Add a `Procfile`:
   ```
   worker: python main.py
   ```
4. Railway will run it 24/7 for free (within limits)
> Note: Railway free tier has ~500 hrs/month — enough for this use case.

### Option C — Your own PC (simplest but not reliable)
Just run `python main.py` — but it stops when your PC is off.

---

## Configuration

Edit the top of `main.py` to change:

| Setting | Default | Description |
|---------|---------|-------------|
| `CHECK_INTERVAL` | 15 | Minutes between API polls |

Edit the top of `redeemer.py` to change:

| Setting | Default | Description |
|---------|---------|-------------|
| `POST_LOGIN_WAIT` | 3 | Seconds to wait after login loads |
| `POST_CONFIRM_WAIT` | 3 | Seconds to wait after clicking Confirm |
| `BETWEEN_PLAYERS` | 2 | Pause between each player |
| `headless` | True | Set False to see the browser window |

---

## Troubleshooting

- **Error screenshots** are saved to `screenshots/` automatically
- **Logs** are in `logs/run_YYYYMMDD.log`
- If the site changes its HTML, the XPATHs in `redeemer.py` may need updating
- Set `headless=False` in `redeemer.py` → `build_driver()` call to watch the browser live
