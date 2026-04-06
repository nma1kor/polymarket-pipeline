# Production Deployment Checklist

## Pre-Deployment

- [ ] Get Gemini API key from [makersuite.google.com](https://makersuite.google.com)
- [ ] Run `bash setup.sh` for automated one-click setup
- [ ] Run `python cli.py verify` to confirm all systems operational
- [ ] Review `.env` file — ensure no sensitive keys are committed
- [ ] Test in dry-run mode first

## Deployment Steps

### 1. One-Click Setup (Recommended)

```bash
git clone https://github.com/brodyautomates/polymarket-pipeline.git
cd polymarket-pipeline
bash setup.sh
```

The setup script will:
- ✓ Check Python version (3.9+)
- ✓ Create virtual environment
- ✓ Install dependencies (google-generativeai, etc.)
- ✓ Prompt for API keys (Gemini, Twitter, Telegram, Polymarket)
- ✓ Create `.env` file
- ✓ Run verification checks

### 2. Manual Verification

```bash
source .venv/bin/activate
python cli.py verify
```

Expected output:
```
✓ Python 3.9+
✓ All dependencies installed
✓ .env file exists
✓ Gemini API key (verified)
✓ RSS scraper (XX headlines)
✓ (optional) Twitter bearer token set
✓ (optional) Telegram bot token set
✓ (optional) Polymarket API credentials
```

### 3. Dry-Run Mode (Recommended First)

```bash
python cli.py watch
```

This runs the pipeline WITHOUT placing orders. Monitor for 1-2 hours:
- Check classification accuracy
- Verify market matching
- Monitor latency

### 4. Enable Live Trading

Edit `.env`:
```env
DRY_RUN=false           # Enable live trading
MAX_BET_USD=25          # Conservative bet size to start
DAILY_LOSS_LIMIT_USD=100  # Daily stop-loss
```

Start live trading:
```bash
python cli.py watch --live
```

## Production Settings

### Recommended Configuration

```env
# API Keys (required)
GEMINI_API_KEY=AIza...

# News Sources (recommended)
TWITTER_BEARER_TOKEN=...        # Real-time breaking news
TELEGRAM_BOT_TOKEN=...          # Channel monitoring
POLYMARKET_API_KEY=...          # Live trading

# Pipeline Settings (conservative)
DRY_RUN=false                   # Enable trading
MAX_BET_USD=10                  # Start conservative
DAILY_LOSS_LIMIT_USD=50         # Strict limit
EDGE_THRESHOLD=0.10             # Wait for strong signals
SPEED_TARGET_SECONDS=5          # Sub-5s classification

# V2 Market Filter (niche markets only)
MAX_VOLUME_USD=500000           # Only <$500K volume markets
MIN_VOLUME_USD=1000             # Minimum liquidity
MATERIALITY_THRESHOLD=0.6       # Only high-impact news
```

### Safety Limits

- **Max Bet**: Start with $5-10, increase to $25 only after 100+ trades
- **Daily Limit**: $50-100 loss limit prevents catastrophic days
- **Market Filter**: Niche markets (<$500K) have less competition
- **Classification Speed**: <5 seconds ensures news recency

## Monitoring & Operations

### Dashboard (Live)

```bash
python cli.py dashboard
```

Shows:
- Active trades
- P&L by market
- Win rate by category
- Classification latency
- API usage

### Calibration Tracking

```bash
python cli.py calibrate
```

Measures classification accuracy as markets resolve:
- Win rate by source (Twitter, Telegram, RSS)
- Win rate by category (AI, crypto, politics, etc.)
- Materiality correlation
- Confidence regression

### Backtest

```bash
python cli.py backtest
```

Replays strategy against resolved markets from last 30 days.

## Troubleshooting

### No trades executing

```bash
# Check market filter
python cli.py niche --hours 24

# 1. Filter too strict?
#    - Reduce MATERIALITY_THRESHOLD from 0.6 → 0.5
#    - Increase MAX_VOLUME_USD from 500K → 1M

# 2. News not matching markets?
#    - Verify TWITTER_BEARER_TOKEN is set
#    - Check RSS feeds in config.py

# 3. Classifications failing?
#    - Run: python cli.py verify
#    - Check GEMINI_API_KEY is valid
```

### High latency (>10 seconds)

```bash
# 1. Check API rate limits
# 2. Reduce max_output_tokens in classifier.py/scorer.py
# 3. Use gemini-2.0-flash-exp (faster than older models)
```

### API errors

```bash
# Check API key validity
python -c "
import google.generativeai as genai
genai.configure(api_key='YOUR_KEY')
model = genai.GenerativeModel('gemini-2.0-flash-exp')
response = model.generate_content('Test')
print(response.text)
"
```

## Maintenance

### Daily

- [ ] Monitor dashboard for unusual losses
- [ ] Verify API is responding normally
- [ ] Check news streams are active (Twitter, Telegram, RSS)

### Weekly

- [ ] Review calibration metrics
- [ ] Check if market filters need adjustment
- [ ] Verify daily loss limits are reasonable

### Monthly

- [ ] Analyze win rate trends
- [ ] Optimize bet sizing
- [ ] Review category performance
- [ ] Backtest with updated strategy

## Production Support

### Health Checks

All health data is logged to SQLite (`pipeline.db`):

```bash
# Check recent classifications
sqlite3 pipeline.db "SELECT * FROM classifications LIMIT 10"

# Check recent trades
sqlite3 pipeline.db "SELECT * FROM trades LIMIT 10"

# Check calibration
sqlite3 pipeline.db "SELECT source, category, win_rate FROM calibration"
```

### Logs

```bash
# Enable verbose logging (optional)
python cli.py watch --verbose

# View logs
tail -f pipeline.log
```

## Disaster Recovery

### Failed Trade

All trades are reversible via Polymarket CLOB. If a bet goes wrong:

```bash
# Check order status
sqlite3 pipeline.db "SELECT * FROM trades WHERE condition_id='XXX'"

# Manual cancel (if needed)
python -c "
from executor import cancel_order
cancel_order(order_id='XXX')
"
```

### Circuit Breaker

The pipeline auto-stops when `DAILY_LOSS_LIMIT` is reached. To resume:

```bash
# Reset daily counter (manual reset only)
sqlite3 pipeline.db "UPDATE daily_stats SET loss=0 WHERE date=date('now')"
```

## Restart

```bash
# Graceful shutdown
pkill -f "python cli.py watch"

# Resume
python cli.py watch --live
```

---

## Go-Live Checklist

- [ ] Setup.sh completed successfully
- [ ] cli.py verify shows all GREEN
- [ ] Dry-run mode tested for 1-2 hours
- [ ] Calibration looks reasonable (>50% accuracy)
- [ ] Gemini API key is valid
- [ ] .env file NEVER committed to git
- [ ] Conservative bet sizes ($10-25)
- [ ] Daily loss limit set ($50-100)
- [ ] Dashboard monitored daily
- [ ] Back up pipeline.db weekly

---

**Ready for production!** Start with dry-run, scale gradually, monitor closely.
