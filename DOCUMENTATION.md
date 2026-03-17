# SafeClaw — Technical Documentation

## Table of Contents
1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Skills Reference](#skills-reference)
4. [API Integrations](#api-integrations)
5. [Security Model](#security-model)
6. [User Flows](#user-flows)
7. [Deployment Guide](#deployment-guide)
8. [Configuration Reference](#configuration-reference)
9. [Troubleshooting](#troubleshooting)

---

## 1. Overview

SafeClaw is a multi-user AI assistant built on OpenClaw that provides:
- Real-time P2P merchant safety scoring
- Pre-trade psychological and technical safety checks
- Adaptive Dollar Cost Averaging
- Trading simulation using real Binance Demo API
- Crypto education with curated resources
- Automated Binance Square content creation

### Key Design Principles
- **Per-user isolation** — every user has completely independent sessions
- **Bring your own keys** — users connect their own Binance accounts
- **Scope-locked** — bot refuses anything outside defined features
- **Multi-account** — supports real, demo, and testnet simultaneously

---

## 2. System Architecture

### Component Stack
```
┌─────────────────────────────────────────────┐
│              Telegram Interface              │
│         @BNBNepalBinanceAIBot               │
└─────────────────┬───────────────────────────┘
                  │ Telegram Bot API (polling)
┌─────────────────▼───────────────────────────┐
│           OpenClaw Gateway                   │
│         Port 18789 (loopback)               │
│         Systemd service (auto-start)         │
└─────────────────┬───────────────────────────┘
                  │ Per-peer session routing
┌─────────────────▼───────────────────────────┐
│         Session Isolation Layer              │
│    dmScope: per-channel-peer                │
│    Each Telegram user ID = isolated session  │
└─────────────────┬───────────────────────────┘
                  │ Agent execution
┌─────────────────▼───────────────────────────┐
│           AI Model Layer                     │
│    Primary: claude-haiku-4.5 (OpenRouter)   │
│    Fallback 1: claude-sonnet-4-5            │
│    Fallback 2: llama-3.3-70b-instruct:free  │
└─────────────────┬───────────────────────────┘
                  │ Skill execution
┌─────────────────▼───────────────────────────┐
│              Skills Layer                    │
│    10 skills in ~/.openclaw/workspace/skills │
│    Each skill = SKILL.md with instructions  │
└─────────────────┬───────────────────────────┘
                  │ API calls
┌─────────────────▼───────────────────────────┐
│           External APIs                      │
│    Binance (live/demo/testnet/P2P/Square)   │
│    OpenRouter (AI inference)                │
│    Alternative.me (Fear & Greed)            │
│    CoinTelegraph / CoinDesk (RSS)           │
└─────────────────────────────────────────────┘
```

### File Structure
```
~/.openclaw/
├── openclaw.json              # Main config
├── workspace/
│   ├── SOUL.md               # Bot identity + security rules (read-only)
│   ├── AGENTS.md             # Agent definitions
│   ├── IDENTITY.md           # Bot personality
│   └── skills/               # All 10 skills
│       ├── api-router/
│       ├── briefing/
│       ├── guardianclaw/
│       ├── p2p-safefinder/
│       ├── profile/
│       ├── safeclaw-academy/
│       ├── smartdca/
│       ├── square-content-engine/
│       ├── user-onboarding/
│       └── yield-monitor/
└── agents/
    └── main/
        └── sessions/          # Per-user isolated sessions
```

---

## 3. Skills Reference

### api-router
**Purpose:** Internal routing layer — determines correct Binance endpoint per user.

**Account type routing:**
| Type | Spot URL | Futures URL |
|------|----------|-------------|
| live | api.binance.com | fapi.binance.com |
| demo | demo-api.binance.com | demo-fapi.binance.com |
| testnet | testnet.binance.vision | testnet.binancefuture.com |

**Used by:** smartdca, safeclaw-academy, yield-monitor, profile

---

### p2p-safefinder
**Purpose:** Find and score safe P2P merchants.

**Scoring model (100 points):**
| Factor | Weight | Safe threshold | Reject threshold |
|--------|--------|----------------|------------------|
| Completion rate | 35% | >98% | <90% |
| Total orders | 25% | >500 | <10 |
| Verified status | 20% | merchant | user |
| Price spread | 20% | <1% | >3% |

**API used:** POST p2p.binance.com/bapi/c2c/v2/friendly/c2c/adv/search

**Required headers:** User-Agent, Accept-Encoding (gzip), Origin, Referer

---

### guardianclaw
**Purpose:** Pre-trade safety interception.

**Checks performed:**
1. 24h volume (>$1M = safe)
2. Price change (<20% = not a pump)
3. Bid/ask spread (<0.1% = liquid)
4. Psychology questionnaire (3 questions)

**FOMO detection phrases:**
- "moon", "pump", "everyone is buying"
- "just saw on Twitter", "quick profit", "can't miss"
- "all in", "last chance", "guaranteed"

**Scoring:**
- 75-100: GO
- 50-74: CAUTION
- 0-49: NO-GO

---

### smartdca
**Purpose:** Adaptive automated DCA.

**Fear & Greed multipliers:**
| Index | Classification | Multiplier |
|-------|---------------|------------|
| 0-25 | Extreme Fear | 2.0x |
| 26-45 | Fear | 1.5x |
| 46-55 | Neutral | 1.0x |
| 56-75 | Greed | 0.75x |
| 76-100 | Extreme Greed | 0.5x |

**Session data stored:**
- dca_asset, dca_amount, dca_interval
- dca_adaptive, dca_status, dca_next_run
- dca_history (last 10 trades)

---

### safeclaw-academy
**Purpose:** Trading simulation and education.

**Simulation environments:**
- Spot: Uses user's demo/testnet account
- Futures: Uses demo-fapi.binance.com or testnet
- Never uses real account for simulation

**Liquidation formulas:**
- LONG: entry × (1 - 1/leverage)
- SHORT: entry × (1 + 1/leverage)

**Education scoring (100 points):**
| Factor | Weight |
|--------|--------|
| Entry reasoning | 30% |
| Market timing | 25% |
| Position sizing | 25% |
| Risk management | 20% |

---

### yield-monitor
**Purpose:** Binance Earn position tracking.

**APIs used:**
- GET /sapi/v1/lending/union/account (overview)
- GET /sapi/v1/lending/daily/product/list (rates)
- GET /sapi/v1/staking/position (locked positions)

**Only works with real (live) accounts.**

---

### square-content-engine
**Purpose:** Binance Square content creation and publishing.

**Publishing API:**
- POST https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add
- Headers: X-Square-OpenAPI-Key, Content-Type, clienttype: binanceSkill
- Limit: 100 posts/day per key

**Draft types:**
1. Educational (SEO) — 300-400 chars, structured, Binance Academy links
2. Viral Hook — 100-150 chars, emotional hook, high shareability
3. Data-Driven (GEO) — 200-250 chars, citation-ready, AI-optimized

**Content sources:**
- Binance market data (api.binance.com)
- Fear & Greed (api.alternative.me)
- CoinTelegraph RSS
- CoinDesk RSS

---

## 4. API Integrations

### Binance APIs Used

| Endpoint | Purpose | Auth Required |
|----------|---------|---------------|
| GET /api/v3/ticker/24hr | Price + market data | No |
| GET /api/v3/ticker/price | Current price | No |
| GET /api/v3/depth | Order book | No |
| GET /api/v3/trades | Recent trades | No |
| GET /api/v3/account | Balance + permissions | Yes (user key) |
| POST /api/v3/order | Place spot order | Yes (user key) |
| POST /fapi/v1/leverage | Set futures leverage | Yes (user key) |
| POST /fapi/v1/order | Place futures order | Yes (user key) |
| GET /fapi/v2/account | Futures balance | Yes (user key) |
| GET /fapi/v2/positionRisk | Open positions | Yes (user key) |
| GET /sapi/v1/lending/* | Earn positions | Yes (user key) |
| POST p2p/.../adv/search | P2P merchant list | No |
| POST pgc/.../content/add | Square posting | Square key |

### Request Signing (HMAC SHA256)
```bash
TIMESTAMP=$(date +%s%3N)
QUERY="param1=val1&param2=val2&timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s "${BASE_URL}/endpoint?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}"
```

---

## 5. Security Model

### SOUL.md Constitution
Located at `~/.openclaw/workspace/SOUL.md` with permissions 444 (read-only).

Enforces:
- Bot identity cannot be changed
- Only allowed Binance endpoints can be called
- No shell commands except curl to whitelisted domains
- No cross-user data access
- Mandatory trade confirmation
- 3-strikes violation tracking

### Per-User Key Isolation
```
Server env vars (infrastructure only):
  OPENROUTER_API_KEY — AI inference
  ENCRYPTION_SECRET — key encryption

User session memory (per user, isolated):
  binance_account_type — live/demo/testnet
  binance_live_key — user's real key
  binance_live_secret — user's real secret
  binance_demo_key — user's demo key
  binance_demo_secret — user's demo secret
  binance_testnet_key — user's testnet key
  binance_testnet_secret — user's testnet secret
  binance_square_key — user's Square key
```

### API Key Validation Rules
- SPOT permission → required
- WITHDRAWALS permission → instant reject
- Key must be 64 chars alphanumeric
- Validated against live Binance API before storing
- User always warned to delete /updatekey message

---

## 6. User Flows

### New User Onboarding
```
/start
  → Currency selection (NPR/INR/NGN/PHP/USD/Other)
  → Payment methods (based on currency)
  → Risk profile (Conservative/Moderate/Aggressive)
  → Account type (Real/Demo/Testnet/Skip)
  → API key instructions
  → /updatekey or /updatekey-demo
  → Validation + storage
  → Ready confirmation
```

### P2P Merchant Search
```
/p2p
  → Fetch live ads (Binance P2P API)
  → Score each merchant (100-point model)
  → Rank by safety score
  → Display top 3 with flags
  → Recommend safest with reasoning
```

### GuardianClaw Trade Check
```
/guard BUY BTC 100 USDT
  → Fetch 24h market data
  → Check volume, spread, momentum
  → Ask 3 psychology questions
  → Calculate safety score
  → GO/CAUTION/NO-GO verdict
  → If YES → execute via user's account
```

### SmartDCA Execution
```
/dca run
  → Read user's account type
  → Fetch Fear & Greed Index
  → Calculate adaptive amount
  → Run GuardianClaw quick check
  → Confirm with user
  → Execute via correct endpoint
  → Record to session history
```

### Academy Simulation
```
/simulate BUY BTC 100
  → Check user has demo/testnet account
  → Verify demo balance
  → Execute real order on demo-api.binance.com
  → Show execution confirmation
  → Offer /evaluate feedback
```

---

## 7. Deployment Guide

### Prerequisites
- Ubuntu 22.04 or 24.04
- Node.js 22+
- 2GB RAM minimum (t3.small or equivalent)
- OpenRouter account + API key
- Telegram bot token (from @BotFather)

### Production Setup
```bash
# System dependencies
sudo apt update && sudo apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs jq curl git

# Add swap (recommended for t3.small)
sudo dd if=/dev/zero of=/swapfile bs=128M count=16
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# OpenClaw
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
npm install -g openclaw

# Run onboarding wizard
NODE_OPTIONS="--max-old-space-size=1024" openclaw onboard

# Deploy skills
git clone https://github.com/BNBNepal/AIBinanceAssistant.git
mkdir -p ~/.openclaw/workspace/skills
cp -r AIBinanceAssistant/skills/* ~/.openclaw/workspace/skills/
cp AIBinanceAssistant/workspace/SOUL.md ~/.openclaw/workspace/SOUL.md
chmod 444 ~/.openclaw/workspace/SOUL.md

# Configure
openclaw config set channels.telegram.allowFrom '["*"]'
openclaw config set channels.telegram.dmPolicy open
openclaw config set channels.telegram.groupPolicy open
openclaw config set agents.defaults.memorySearch.enabled false
openclaw config set env.ENCRYPTION_SECRET "$(openssl rand -hex 16)"

# Start
openclaw gateway restart
```

### Update Procedure
```bash
cd ~/AIBinanceAssistant
git pull
cp -r skills/* ~/.openclaw/workspace/skills/
cp workspace/SOUL.md ~/.openclaw/workspace/SOUL.md
chmod 444 ~/.openclaw/workspace/SOUL.md
openclaw gateway restart
```

---

## 8. Configuration Reference

### openclaw.json Key Settings
```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/anthropic/claude-haiku-4.5",
        "fallbacks": [
          "openrouter/anthropic/claude-sonnet-4-5",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free"
        ]
      },
      "contextTokens": 50000,
      "memorySearch": { "enabled": false }
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "open",
      "allowFrom": ["*"],
      "groupPolicy": "open",
      "streaming": "off"
    }
  },
  "session": {
    "dmScope": "per-channel-peer"
  }
}
```

### SOUL.md Allowed Endpoints
- api.binance.com
- p2p.binance.com
- demo-api.binance.com
- demo-fapi.binance.com
- testnet.binance.vision
- testnet.binancefuture.com
- api.alternative.me
- cointelegraph.com/rss
- coindesk.com/arc/outboundfeeds/rss

---

## 9. Troubleshooting

### Bot not responding
```bash
# Check gateway status
openclaw gateway status

# Check logs
openclaw logs --follow

# Restart
openclaw gateway restart
```

### DNS issues (WSL2 only)
```bash
sudo chmod 644 /etc/resolv.conf
printf "nameserver 8.8.8.8\nnameserver 1.1.1.1\n" | sudo tee /etc/resolv.conf
sudo chmod 444 /etc/resolv.conf
```

### Out of memory
```bash
NODE_OPTIONS="--max-old-space-size=1024" openclaw gateway restart
```

### Session not resetting
```bash
rm -rf ~/.openclaw/agents/main/sessions/
mkdir -p ~/.openclaw/agents/main/sessions/
openclaw gateway restart
```

### P2P API returning empty
Binance P2P requires compressed requests:
```bash
curl -s --compressed -X POST "https://p2p.binance.com/..." \
  -H "Accept-Encoding: gzip, deflate, br"
```

### Rate limits (OpenRouter free tier)
- Free tier: 20 req/min, 200 req/day
- Solution: Add credits to OpenRouter account
- Recommended model: claude-haiku-4.5 (~$0.001/message)
