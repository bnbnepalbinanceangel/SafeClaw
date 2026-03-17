# 🦞 SafeClaw — AI-Powered Binance Assistant

SafeClaw is a Telegram-based AI assistant that makes Binance safer, smarter, and more educational for everyone — from complete beginners to active traders worldwide.

**Live Bot:** [@BNBNepalBinanceAIBot](https://t.me/BNBNepalBinanceAIBot)

---

## 🎯 Problem Statement

Crypto trading in emerging markets faces three critical problems:
1. **P2P scams** — No intelligent way to verify safe merchants
2. **Emotional trading** — FOMO and panic cause most retail losses
3. **Knowledge gap** — Beginners have no safe way to practice before using real money

SafeClaw solves all three using OpenClaw AI agents and the full Binance API ecosystem.

---

## ✨ Features

### 🔍 P2P SafeFinder
Find the safest Binance P2P merchants in real time.
- Fetches live ads from Binance P2P API
- Scores each merchant out of 100 (completion rate, orders, verification, spread)
- Flags risky merchants with 🔴 red alerts
- Supports all major fiat currencies globally
```
/p2p — Find safe P2P merchants for your currency
```

### 🛡️ GuardianClaw Pre-Trade Safety
Intercepts every trade before execution.
- Live market data check (price, volume, bid/ask spread)
- FOMO detection — flags emotional trading language
- 3-question psychology check before any trade
- GO / CAUTION / NO-GO verdict with full reasoning
- Calculates exact risk/reward ratios
```
/guard BUY BTC 100 USDT
```

### 💰 SmartDCA — Adaptive Dollar Cost Averaging
Automated crypto accumulation adjusted by market sentiment.
- Fetches Fear & Greed Index before every buy
- Extreme Fear (0-25) → buys 2x your base amount
- Extreme Greed (75-100) → buys 0.5x your base amount
- Works with real, demo, or testnet accounts
- Full trade history and P&L tracking
```
/dca setup — Configure your plan
/dca run — Execute now
/dca status — View performance
```

### 📊 Daily Market Briefing
Your personalized crypto market update.
- Live prices: BTC, ETH, BNB, SOL, XRP
- Top 5 gainers and losers with percentages
- Local fiat P2P rate for your currency
- Market mood analysis (Bullish/Neutral/Bearish)
- Fear & Greed Index context
```
/briefing — Get today's market update
```

### 🎓 SafeClaw Academy
Learn trading using the **real Binance Demo API** — zero risk.
- **Spot simulation** on demo-api.binance.com (real prices, real execution)
- **Futures simulation** on demo-fapi.binance.com with leverage education
- Always shows liquidation price for futures trades
- **Trade evaluator** — scores any decision out of 100
- **Learning resources** — curated Binance Academy articles
- Proactive education — detects FOMO, bad leverage, missing stop losses
```
/simulate BUY BTC 100
/simulate FUTURES LONG BTC 0.01 10x
/evaluate — Score your trade decision
/learn leverage — Educational resources
/academy — Full learning menu
```

### 💎 Yield Monitor
Track your Binance Earn positions and optimize yields.
- Overview of all flexible savings and locked staking
- Current APY rates across all products
- Optimization suggestions when better rates available
- Estimated monthly yield calculation
```
/yield — Full Earn overview
/yield rates — Browse current APY rates
/yield optimize — Get better yield suggestions
```

### 📝 Square Content Engine
AI-powered Binance Square content creation.
- Trending topic discovery from Binance, CoinTelegraph, CoinDesk
- 3 draft variations: Educational (SEO), Viral Hook, Data-Driven (GEO)
- SEO/AEO/GEO optimization built into every draft
- Compliance check — no financial advice, no pump language
- Direct publishing to Binance Square via official API
- Content calendar and scheduling
```
/trending — Discover trending crypto topics
/draft [topic] — Create 3 optimized drafts
/publish — Post to Binance Square
/calendar — View content schedule
```

### 👤 User Onboarding
Personalized setup for every user globally.
- 5-step onboarding wizard (currency, payment, risk profile)
- Supports real, demo, and testnet Binance accounts
- API key validation — checks permissions, rejects withdrawal access
- Per-user completely isolated sessions
- No user can ever access another user's data or keys
```
/start — Begin onboarding
/updatekey — Connect real Binance account
/updatekey-demo — Connect demo account (beginners)
/updatekey-testnet — Connect testnet
/profile — View your settings
```

---

## 🏗️ Architecture
```
Users (Telegram)
      ↓
@BNBNepalBinanceAIBot
      ↓
OpenClaw 2026.3.13 Gateway
(AWS EC2 t3.small — always-on)
      ↓
Per-User Session Isolation (dmScope: per-channel-peer)
      ↓
Claude Haiku 4.5 via OpenRouter
      ↓
┌────────────────────────────────────────┐
│            Skills Layer (10)           │
│  p2p-safefinder    guardianclaw        │
│  smartdca          briefing            │
│  safeclaw-academy  yield-monitor       │
│  user-onboarding   profile             │
│  square-content-engine  api-router     │
└────────────────────────────────────────┘
      ↓
┌────────────────────────────────────────┐
│          Binance API Layer             │
│  api.binance.com      (live trading)   │
│  p2p.binance.com      (P2P data)       │
│  demo-api.binance.com (spot demo)      │
│  demo-fapi.binance.com (futures demo)  │
│  testnet.binance.vision (testnet)      │
│  api.alternative.me   (Fear & Greed)   │
└────────────────────────────────────────┘
```

### Multi-User Security Architecture
```
User A (real account)          User B (demo account)
/updatekey KEY SECRET    →     /updatekey-demo KEY SECRET
        ↓                              ↓
Validated against               Validated against
api.binance.com                 demo-api.binance.com
        ↓                              ↓
Stored in User A's             Stored in User B's
isolated session ONLY          isolated session ONLY
        ↓                              ↓
All trades use                 All trades use
api.binance.com                demo-api.binance.com
```

No user can ever access another user's keys or data.

---

## 🔒 Security

- **SOUL.md constitution** — immutable identity and scope rules (chmod 444)
- **Per-user session isolation** — `dmScope: per-channel-peer`
- **API key validation** — rejects any key with withdrawal permissions
- **Per-user isolated storage** — keys stored in isolated sessions only
- **Scope enforcement** — bot refuses all out-of-scope requests
- **No shared credentials** — server env only holds infrastructure keys

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| Agent Framework | OpenClaw 2026.3.13 |
| AI Model | Claude Haiku 4.5 via OpenRouter |
| Fallback Models | Claude Sonnet 4.5, Llama 3.3 70B free |
| Channel | Telegram Bot API |
| Server | AWS EC2 t3.small (Ubuntu 24.04) |
| Binance APIs | Spot, P2P, Demo, Futures Demo, Testnet, Square |
| Sentiment | Alternative.me Fear & Greed Index |
| News Sources | CoinTelegraph RSS, CoinDesk RSS |

---

## 📦 Skills (10 total)

| Skill | Commands | Description |
|-------|----------|-------------|
| `p2p-safefinder` | `/p2p` | Live P2P merchant safety scoring |
| `guardianclaw` | `/guard` | Pre-trade safety + FOMO detection |
| `smartdca` | `/dca` | Adaptive DCA with Fear & Greed |
| `briefing` | `/briefing` | Daily market update |
| `safeclaw-academy` | `/simulate` `/learn` `/evaluate` | Trading simulation + education |
| `yield-monitor` | `/yield` | Binance Earn tracking + optimization |
| `user-onboarding` | `/start` `/updatekey` | Setup wizard + API validation |
| `profile` | `/profile` | User settings + live balance |
| `square-content-engine` | `/trending` `/draft` `/publish` | Binance Square content |
| `api-router` | (internal) | Per-user account type routing |

---

## 🚀 Self-Hosting
```bash
# 1. Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs jq curl

# 2. Install OpenClaw
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
npm install -g openclaw

# 3. Run onboarding
openclaw onboard

# 4. Clone and deploy skills
git clone https://github.com/BNBNepal/AIBinanceAssistant.git
mkdir -p ~/.openclaw/workspace/skills
cp -r AIBinanceAssistant/skills/* ~/.openclaw/workspace/skills/
cp AIBinanceAssistant/workspace/SOUL.md ~/.openclaw/workspace/SOUL.md
chmod 444 ~/.openclaw/workspace/SOUL.md

# 5. Configure
openclaw config set channels.telegram.allowFrom '["*"]'
openclaw config set channels.telegram.dmPolicy open
openclaw config set channels.telegram.groupPolicy open
openclaw config set agents.defaults.memorySearch.enabled false

# 6. Start
openclaw gateway restart
```

---

## 📋 Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENROUTER_API_KEY` | ✅ | OpenRouter API key for AI model |
| `ENCRYPTION_SECRET` | ✅ | 32-char hex for encryption |

**Note:** Binance API keys are provided by each user via `/updatekey` — never stored at server level.

---

## 📄 License

This project is licensed under the **GNU General Public License v3.0**.
See the [LICENSE](LICENSE) file for full terms.

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

---

*Built with 🦞 OpenClaw*
