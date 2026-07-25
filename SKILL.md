---
name: 9router
description: Configure, test, and use 9Router multi-provider LLM router for chat, image gen, TTS, STT, web search, and web fetch. Supports combo models with auto-fallback round-robin across providers (ByNara, Gemini, OpenRouter, etc.). Use when setting up 9Router API, debugging provider routing, testing endpoint connectivity, or using 9Router models/endpoints.
version: 1.0.0
author: Hermes Agent
license: MIT
---

# 9Router — Multi-Provider Router

## What is 9Router
Proxy/router that chains multiple AI providers behind one endpoint. Model `combo` automatically round-robins + fallbacks across all enabled providers. Single URL + single key gives access to dozens of models.

## Setup

### 1. Set credentials in .env
```bash
# Add to ~/.hermes/.env
NINEROUTER_URL=https://YOUR_TUNNEL_URL/v1
NINEROUTER_KEY=sk-YOUR_KEY
source ~/.hermes/.env
```

### 2. Verify connection
```bash
curl -s "$NINEROUTER_URL/v1/models" -H "Authorization: Bearer $NINEROUTER_KEY" | head -c 500
# Should return JSON with available models including "combo"
```

### 3. List available models by category
```bash
# All models
curl -s "$NINEROUTER_URL/v1/models" -H "Authorization: Bearer $NINEROUTER_KEY" | jq '.data[].id'

# Image generation models
curl -s "$NINEROUTER_URL/v1/models/image" -H "Authorization: Bearer $NINEROUTER_KEY" | jq '.data[].id'

# Web search models
curl -s "$NINEROUTER_URL/v1/models/web" -H "Authorization: Bearer $NINEROUTER_KEY" | jq '.data[] | select(.kind=="webSearch") | .id'

# TTS models
curl -s "$NINEROUTER_URL/v1/models/tts" -H "Authorization: Bearer $NINEROUTER_KEY" | jq '.data[].id'

# STT models
curl -s "$NINEROUTER_URL/v1/models/stt" -H "Authorization: Bearer $NINEROUTER_KEY" | jq '.data[].id'
```

## Endpoints

### Chat / Completions
```bash
POST $NINEROUTER_URL/v1/chat/completions
# OpenAI format
curl -X POST "$NINEROUTER_URL/v1/chat/completions" \
  -H "Authorization: Bearer $NINEROUTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"combo","messages":[{"role":"user","content":"Hi"}],"max_tokens":10}'

# Anthropic format (if supported)
curl -X POST "$NINEROUTER_URL/v1/messages" \
  -H "Authorization: Bearer $NINEROUTER_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"model":"bynara/claude-opus-4.8","max_tokens":1024,"messages":[{"role":"user","content":"Hi"}]}'
```

### Image Generation
```bash
POST $NINEROUTER_URL/v1/images/generations
curl -X POST "$NINEROUTER_URL/v1/images/generations?response_format=binary" \
  -H "Authorization: Bearer $NINEROUTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gemini/gemini-3-pro-image-preview","prompt":"neon jellyfish","size":"1024x1024"}' \
  --output out.png

# Available image models: bynara/gpt-5.4, gemini/*, openai/dall-e-3, black-forest-labs/flux, etc.
```

### Text-to-Speech (TTS)
```bash
POST $NINEROUTER_URL/v1/audio/speech
curl -X POST "$NINEROUTER_URL/v1/audio/speech" \
  -H "Authorization: Bearer $NINEROUTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"openai/tts-1","input":"Hello world"}' --output speech.mp3

# Voice options: openai/tts-1, el/<voice_id>, edge-tts/vi-VN-HoaiMyNeural, google-tts/vi
# edge-tts supports Indonesian natively without auth
```

### Speech-to-Text (STT)
```bash
POST $NINEROUTER_URL/v1/audio/transcriptions
curl -X POST "$NINEROUTER_URL/v1/audio/transcriptions" \
  -H "Authorization: Bearer $NINEROUTER_KEY" \
  -F "model=openai/whisper-1" \
  -F "file=@audio.mp3" \
  -F "language=id"

# Models: whisper-1, groq/whisper-large-v3, deepgram/nova-3, gemini-2.5-flash
```

### Web Search
```bash
POST $NINEROUTER_URL/v1/search
curl -X POST "$NINEROUTER_URL/v1/search" \
  -H "Authorization: Bearer $NINEROUTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"tavily","query":"latest crypto trends","max_results":5}'

# Providers: tavily, brave-search, exa, serper, perplexity, linkup, youcom, searxng (noAuth)
# Combo: "search-combo" chains providers with auto-fallback
```

### Web Fetch (URL → Markdown)
```bash
POST $NINEROUTER_URL/v1/web/fetch
curl -X POST "$NINEROUTER_URL/v1/web/fetch" \
  -H "Authorization: Bearer $NINEROUTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"jina-reader","url":"https://example.com","format":"markdown"}'

# Providers: firecrawl (JS-rendered), jina-reader (fastest free), tavily, exa
# Combo: "fetch-combo" chains providers with auto-fallback
```

## Key Provider Quirks

### ByNara (default provider on this VPS)
- Heavyweight: gpt-5.6, claude-opus-4.8, grok-4.5, kimi-k3, qwen3.7-max, deepseek-v4-pro
- Some models have vision (`gc/gemini-*`, `bynara/gpt-5.6-luna`, `bynara/claude-*`)
- Combo model uses ByNara as base + auto-fallback to other providers

### Gemini (Google Cloud)
- Vision + search + tools in one model
- `gc/gemini-3.1-pro-preview` — latest pro, 1M context window
- `gc/gemini-3-flash-preview` — faster, cheaper

### OpenRouter Free
- Multiple free models still available: `openrouter/google/gemma-4-31b-it:free`, `nemotron-3-ultra:free`, `laguna-xs-2.1:free`
- These are NOT deprecated like OpenRouter.org (which moved to paid)

### Edge TTS (Indonesian voices)
- `edge-tts/vi-VN-HoaiMyNeural` — free, no auth needed
- Other Indo voices: `vi-VN-LanVy`, `vi-VN-An`, etc.

## Troubleshooting

### Connection fails
```bash
# Test URL connectivity
curl -sv "$NINEROUTER_URL/v1/models" -H "Authorization: Bearer $NINEROUTER_KEY" 2>&1 | tail -20
# Check if tunnel is alive: ping -c 3 rccqndg.abc-tunnel.us
# If tunnel down, wait for 9Router restart
```

### Model not found
```bash
# Verify model exists before using
curl -s "$NINEROUTER_URL/v1/models" -H "Authorization: Bearer $NINEROUTER_KEY" | \
  jq '.data[] | select(.id | contains("MODEL_NAME")) | .id'
```

### Combo model falling back too much
- Combo auto-rotates through providers. If one is down, it skips to next.
- Can force specific provider: replace `"model":"combo"` with `"model":"bynara/gpt-5.6"` directly.
- Rate limits per provider vary — heavy usage on one provider triggers fallback more often.

### Gateway needs restart after config change
- After changing `model.default` or provider in `config.yaml`, gateway must restart.
- Use `_HERMES_GATEWAY=0 hermes gateway restart` if inside gateway process.
- Or restart from outside SSH session.

## Integration with Hermes
After 9Router setup, bot uses it via `hermes config set`:
```bash
hermes config set model.default "combo"
hermes config set model.provider "custom:9router"
hermes config set custom_providers '[{"name":"9router","base_url":"'$NINEROUTER_URL'","api_key_env":"NINEROUTER_KEY","type":"openai"}]'
```

Then restart gateway. Bot will route all chat through 9Router combo.

## Full Endpoint Reference
See `references/endpoints.md` for condensed docs on all 6 endpoint categories (chat, image, TTS, STT, web search, web fetch) with provider quirks, model formats, and example requests.

## VPS systemd Setup
See `references/systemd-setup.md` for step-by-step systemd service files for both 9router and Hermes gateway, including the critical pitfall that systemd does NOT expand `$VAR` in ExecStart (must hardcode paths).

## Deprecation Notes
- IAMHC: all 75+ keys retired (503/down), removed from .env
- OpenRouter free models: deprecated (moved to paid), `:free` slugs return errors
- Primary fallback was IAMHC image gen — now use 9Router `/v1/images/generations` for image tasks