---
name: briefing
description: Provide daily Binance market briefings with price data, top movers, and trading insights. Trigger when user sends /briefing, asks for market update, today's prices, crypto news summary, or market overview.
---

# Daily Market Briefing 📊

Fetch live market data from Binance and deliver a concise daily briefing.

## /briefing Command

When user sends /briefing, fetch and format a complete market update.

### Step 1 — Fetch BTC, ETH, BNB prices
```bash
curl -s "https://api.binance.com/api/v3/ticker/24hr?symbols=%5B%22BTCUSDT%22,%22ETHUSDT%22,%22BNBUSDT%22,%22SOLUSDT%22,%22XRPUSDT%22%5D" | jq '[.[] | {
  symbol: .symbol,
  price: .lastPrice,
  change: .priceChangePercent,
  high: .highPrice,
  low: .lowPrice,
  volume: .quoteVolume
}]'
```

### Step 2 — Fetch top gainers
```bash
curl -s "https://api.binance.com/api/v3/ticker/24hr" | jq '[.[] | select(.symbol | endswith("USDT")) | select((.priceChangePercent | tonumber) > 0) | {symbol: .symbol, change: .priceChangePercent, price: .lastPrice}] | sort_by(.change | tonumber) | reverse | .[0:5]'
```

### Step 3 — Fetch top losers
```bash
curl -s "https://api.binance.com/api/v3/ticker/24hr" | jq '[.[] | select(.symbol | endswith("USDT")) | select((.priceChangePercent | tonumber) < 0) | {symbol: .symbol, change: .priceChangePercent, price: .lastPrice}] | sort_by(.change | tonumber) | .[0:5]'
```

### Step 4 — Fetch NPR/USDT P2P rate
```bash
curl -s --compressed -X POST "https://p2p.binance.com/bapi/c2c/v2/friendly/c2c/adv/search" \
  -H "Content-Type: application/json" \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
  -H "Accept-Encoding: gzip, deflate, br" \
  -H "Origin: https://p2p.binance.com" \
  -H "Referer: https://p2p.binance.com/" \
  -d '{"asset":"USDT","fiat":"NPR","merchantCheck":false,"page":1,"payTypes":[],"rows":3,"tradeType":"BUY"}' \
  | jq '[.data[] | {name: .advertiser.nickName, price: .adv.price}]'
```

### Step 5 — Format briefing

Always format like this:
```
📊 SafeClaw Daily Briefing
🗓 [DATE] | 🕐 [TIME] NPT

━━━ Major Prices ━━━
₿ BTC: $XX,XXX (+X.X%)
Ξ ETH: $X,XXX (+X.X%)
⬡ BNB: $XXX (+X.X%)
◎ SOL: $XXX (+X.X%)

━━━ Top Gainers 🚀 ━━━
- XXXUSDT: +XX.X%
- XXXUSDT: +XX.X%
- XXXUSDT: +XX.X%

━━━ Top Losers 📉 ━━━
- XXXUSDT: -XX.X%
- XXXUSDT: -XX.X%

━━━ NPR P2P Rate 🇳🇵 ━━━
- Best USDT rate: XXX NPR
- Top merchant: [name]

━━━ Market Mood ━━━
[Brief 2-line analysis based on BTC change]

/p2p — Find safe merchants
/guard — Check a trade
```

## Market Mood Logic

- BTC > +3% → "🟢 Bullish — market showing strong momentum"
- BTC -1% to +3% → "🟡 Neutral — sideways consolidation"
- BTC < -3% → "🔴 Bearish — caution advised, good DCA opportunity"

## Scheduled Briefings (future)

When cron is configured, send briefing daily at 7:00 AM NPT (01:15 UTC).
Cron expression: 15 1 * * *
