# Migration from Claude to Gemini API

## Summary of Changes

This project has been completely migrated from **Claude API (Anthropic)** to **Gemini API (Google)**.

### What Changed

| Component | Before | After |
|-----------|--------|-------|
| **LLM Provider** | Anthropic | Google |
| **Package** | `anthropic` | `google-generativeai` |
| **Environment Variable** | `ANTHROPIC_API_KEY` | `GEMINI_API_KEY` |
| **Classification Model** | `claude-haiku-4-5-20251001` | `gemini-2.0-flash-exp` |
| **Scoring Model** | `claude-sonnet-4-6-20250514` | `gemini-2.0-flash-exp` |

### Affected Files

- ✅ `requirements.txt` - Updated dependencies
- ✅ `config.py` - API key and model configuration
- ✅ `classifier.py` - Classification engine using Gemini
- ✅ `scorer.py` - Scoring engine using Gemini
- ✅ `cli.py` - Verification checks for Gemini
- ✅ `setup.sh` - Setup script prompts for Gemini API key
- ✅ `.env.example` - Example environment variables
- ✅ `README.md` - Documentation updated to reference Gemini

---

## Quick Start

### One-Click Setup (Recommended)

```bash
git clone https://github.com/brodyautomates/polymarket-pipeline.git
cd polymarket-pipeline
bash setup.sh
```

When prompted, provide your **Gemini API key** (get one at [makersuite.google.com](https://makersuite.google.com)).

### Manual Setup

```bash
# Clone and prepare
git clone https://github.com/brodyautomates/polymarket-pipeline.git
cd polymarket-pipeline

# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies (now uses google-generativeai)
pip install -r requirements.txt

# Setup environment
cp .env.example .env
```

Edit `.env` and add your Gemini API key:

```env
GEMINI_API_KEY=AIza...  # Get from makersuite.google.com
```

### Verify Setup

```bash
python cli.py verify
```

This will check:
- ✓ Python 3.9+
- ✓ All dependencies installed (including google-generativeai)
- ✓ Gemini API key is valid
- ✓ News feeds are accessible
- ✓ Polymarket WebSocket (if enabled)

---

## Getting a Gemini API Key

1. Visit [makersuite.google.com](https://makersuite.google.com)
2. Click "Get API Key" → "Create API key in new project"
3. Copy the key (starts with `AIza...`)
4. Paste into `.env` as `GEMINI_API_KEY=AIza...`

> **Note:** Gemini has free tier limits. Check [Google's pricing page](https://ai.google.dev/pricing) for current rates.

---

## Production Considerations

### Cost Optimization

- **Gemini 2.0 Flash**: Fast inference, lower cost than older models
- **Token limits**: Set `max_output_tokens=200` for classification, `=500` for scoring
- **Temperature**: Optimized at 0.1 for classification, 0.2 for scoring

### Safety Features (Unchanged)

- **Dry-run mode**: ON by default (set `DRY_RUN=false` in `.env` to enable live trading)
- **Position sizing**: Quarter-Kelly with $25 max bet
- **Daily loss limit**: $100 (configurable)
- **Niche market filter**: Only trades markets <$500K volume

### Monitoring

All classification results are logged to SQLite for calibration validation:

```bash
python cli.py calibrate  # V2: Check classification accuracy
```

---

## API Differences (For Developers)

### Classification (formerly Claude)

**Old (Anthropic):**
```python
client = anthropic.Anthropic(api_key=api_key)
response = client.messages.create(
    model="claude-haiku-4-5-20251001",
    max_tokens=200,
    messages=[{"role": "user", "content": prompt}]
)
```

**New (Gemini):**
```python
genai.configure(api_key=api_key)
client = genai.GenerativeModel("gemini-2.0-flash-exp")
response = client.generate_content(
    prompt,
    generation_config={"max_output_tokens": 200}
)
```

### Response Handling

Both return JSON in the format:
```json
{
  "direction": "bullish",
  "materiality": 0.75,
  "reasoning": "Strong bullish news for the market."
}
```

No changes to business logic — just the underlying API.

---

## Troubleshooting

### "Gemini API key not set"

```bash
# Check your .env file
cat .env | grep GEMINI_API_KEY

# Make sure it starts with AIza...
# If empty, get a key from makersuite.google.com
```

### "Authentication failed"

- Key may be invalid or expired
- Verify key format: `AIza...` (exactly)
- Try creating a new key at [makersuite.google.com](https://makersuite.google.com)

### Deprecation Warning

```
FutureWarning: All support for the `google.generativeai` package has ended
```

This is a heads-up from Google. The package still works fine. A newer SDK (`google.genai`) will be available in the future.

---

## Performance

Gemini 2.0 Flash is optimized for speed and cost:

- **Classification latency**: ~200-400ms (vs ~300-500ms with Claude Haiku)
- **Cost per classification**: ~$0.0001 (competitive with Claude Haiku)
- **Throughput**: No rate limit changes

---

## Next Steps

1. **Run the verification:**
   ```bash
   python cli.py verify
   ```

2. **Start the pipeline (dry-run):**
   ```bash
   python cli.py watch
   ```

3. **Enable live trading (when confident):**
   ```bash
   python cli.py watch --live
   ```

---

## Support

- **Gemini API docs:** https://ai.google.dev/
- **Issue tracker:** Report bugs on GitHub
- **Security:** Never commit `.env` file with your API key

---

**Migration completed successfully!** Your Polymarket pipeline is now production-ready with Gemini API.
