# Deploy to Render

## Quick Setup (5 minutes)

### 1. Go to Render Dashboard
- Visit https://dashboard.render.com
- Sign up with GitHub account `nma1kor`

### 2. Connect Your Fork
- Click **New → Web Service**
- Select **Docker**
- Choose repository: `nma1kor/polymarket-pipeline`
- Branch: `nma1kor-save`
- Click **Connect**

### 3. Configure Environment
- Click **Add Environment Variable** for each:
  - `GEMINI_API_KEY` → Your API key (get from https://makersuite.google.com)
  - `TWITTER_BEARER_TOKEN` → Twitter API key (optional)
  - `TELEGRAM_BOT_TOKEN` → Telegram bot token (optional)
  - `POLYMARKET_API_KEY` → Polymarket credentials (optional)
  - `DRY_RUN` → `true` (to test without trading)

### 4. Deploy
- Click **Create Web Service**
- Render builds & deploys automatically (~3-5 min)
- Your app runs at: `https://polymarket-pipeline.onrender.com`

### 5. View Logs
- Go to **Logs** tab in Render dashboard
- Watch the pipeline run in real-time

---

## What's Running

- ✅ `python cli.py watch` — Monitors news streams continuously
- ✅ SQLite database persists trades on disk
- ✅ Free tier never stops (unlike other providers)

---

## Access Dashboard Anywhere

Once deployed, open your browser:
```
https://polymarket-pipeline.onrender.com/dashboard
```

Or SSH into logs to see output:
```bash
render logs polymarket-pipeline
```

---

## Cost

- ✅ **FREE** on Render free tier
- Includes: 750 compute hours/month (enough for 24/7 operation)
- Includes: 100 GB bandwidth/month

---

## Troubleshooting

### App keeps spinning down?
- Upgrade to paid plan (~$7/mo) for guaranteed uptime
- Or use Render's "restart" button to force restart

### Missing environment variables?
- Check Render dashboard **Environment** tab
- Make sure all required keys are set

### Database errors?
- Check `/app/data` disk space in Render dashboard
- Render provides 1 GB free

---

## Local Testing Before Deploy

Test locally to ensure config works:

```bash
source .venv/bin/activate
cp .env.example .env
# Add your GEMINI_API_KEY to .env
python cli.py watch --dry-run
```
