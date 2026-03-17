---
name: safeclaw-academy
description: Trading simulator using Binance Demo API, trade evaluation, and educational resources. Trigger when user sends /simulate, /evaluate, /learn, /academy, asks to practice trading, wants to understand crypto concepts, asks about spot or futures trading, leverage, stop loss, liquidation, DCA, or any trading topic.
---

# SafeClaw Academy 🎓

Learn crypto trading safely. Uses user's own Demo or Testnet account.

## CRITICAL — Per-User Account Routing

For /simulate, check user's account type:
```
if binance_account_type == "demo":
    SPOT_URL = "https://demo-api.binance.com"
    FUTURES_URL = "https://demo-fapi.binance.com"
    USER_KEY = binance_demo_key from session
    USER_SECRET = binance_demo_secret from session
    ACCOUNT_LABEL = "🟡 Binance Demo"

elif binance_account_type == "testnet":
    SPOT_URL = "https://testnet.binance.vision"
    FUTURES_URL = "https://testnet.binancefuture.com"
    USER_KEY = binance_testnet_key from session
    USER_SECRET = binance_testnet_secret from session
    ACCOUNT_LABEL = "🔵 Binance Testnet"

elif binance_account_type == "live":
    Show warning: "⚠️ /simulate uses Demo or Testnet only — not your real account.
    Please run /updatekey-demo to get a practice account with $5,000 USDT."
    STOP

else:
    Show: "Please run /updatekey-demo to connect a demo account first.
    Get free $5,000 USDT at demo.binance.com"
    STOP
```

## Commands

- /simulate BUY BTC 100 — Spot trade
- /simulate SELL ETH 0.5 — Spot sell
- /simulate FUTURES LONG BTC 0.01 10x — Futures long
- /simulate FUTURES SHORT BTC 0.01 5x — Futures short
- /evaluate — Score any trade decision
- /learn [topic] — Educational resources
- /academy — Full learning menu

## /simulate — Spot Trade

### Check balance first
```bash
USER_KEY="[from session]"
USER_SECRET="[from session]"
SPOT_URL="[from account type]"
TIMESTAMP=$(date +%s%3N)
QUERY="timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s "${SPOT_URL}/api/v3/account?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '[.balances[] | select((.free | tonumber) > 0) | {asset: .asset, free: .free}]'
```

### Execute spot trade
```bash
SYMBOL="BTCUSDT"
SIDE="BUY"
QUOTE_QTY="100"
TIMESTAMP=$(date +%s%3N)
QUERY="symbol=${SYMBOL}&side=${SIDE}&type=MARKET&quoteOrderQty=${QUOTE_QTY}&timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s -X POST "${SPOT_URL}/api/v3/order?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '{
    orderId: .orderId,
    status: .status,
    executedQty: .executedQty,
    cummulativeQuoteQty: .cummulativeQuoteQty,
    avgPrice: .fills[0].price
  }'
```

### Trade confirmation
```
✅ Demo Trade Executed

🏦 [ACCOUNT_LABEL] — Real prices, zero real money risk
📊 Order:
- Symbol: BTCUSDT
- Side: BUY
- Spent: 100 USDT
- Received: 0.001398 BTC
- Price: $71,530
- Order ID: XXXXXXX

📚 Next:
- /evaluate — Score this decision
- /learn market orders — Understand what happened
```

## /simulate — Futures Trade

### Set leverage
```bash
TIMESTAMP=$(date +%s%3N)
QUERY="symbol=BTCUSDT&leverage=10&timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s -X POST "${FUTURES_URL}/fapi/v1/leverage?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '{symbol: .symbol, leverage: .leverage}'
```

### Place futures order
```bash
TIMESTAMP=$(date +%s%3N)
QUERY="symbol=BTCUSDT&side=BUY&type=MARKET&quantity=0.01&timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s -X POST "${FUTURES_URL}/fapi/v1/order?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '{orderId: .orderId, status: .status, avgPrice: .avgPrice, executedQty: .executedQty}'
```

### Always show futures education
```
⚠️ Futures Education:
- Leverage: 10x
- Entry: $XX,XXX
- Liquidation: $XX,XXX (10% drop = 100% margin loss)
- Beginners: start with 2x max
```

Liquidation:
- LONG: entry * (1 - 1/leverage)
- SHORT: entry * (1 + 1/leverage)

## /evaluate Command

Score any trade decision out of 100.

| Factor | Weight | Good | Bad |
|--------|--------|------|-----|
| Entry reasoning | 30% | Planned | FOMO |
| Market timing | 25% | Fear zone | Greed spike |
| Position size | 25% | <5% portfolio | >20% |
| Risk management | 20% | Has stop loss | No exit plan |

## /learn Command

Authoritative sources only:

- stop loss → academy.binance.com/en/articles/what-is-a-stop-loss-order
- leverage → academy.binance.com/en/articles/what-is-leverage-in-crypto-trading
- futures → academy.binance.com/en/articles/a-complete-guide-to-cryptocurrency-trading-for-beginners
- DCA → academy.binance.com/en/articles/dollar-cost-averaging-dca-explained
- technical analysis → academy.binance.com/en/articles/a-beginners-guide-to-technical-analysis
- candlestick → academy.binance.com/en/articles/a-beginners-guide-to-candlestick-charts
- FOMO → academy.binance.com/en/articles/fomo-and-fud-in-crypto
- liquidation → academy.binance.com/en/articles/what-is-liquidation
- P2P → academy.binance.com/en/articles/what-is-peer-to-peer-p2p-trading
- risk management → academy.binance.com/en/articles/risk-management-in-trading
- spot trading → academy.binance.com/en/articles/crypto-spot-trading-101
- market order → academy.binance.com/en/articles/market-order-vs-limit-order

Response format:
```
📚 SafeClaw Academy: [TOPIC]

🎯 Plain English:
[2-3 sentences]

🔑 Key Points:
- Point 1
- Point 2
- Point 3

⚠️ Common Mistakes:
- Mistake 1
- Mistake 2

📖 Full Guide: [URL]
🎮 Practice: /simulate
🔗 Related: /learn [topic1] | /learn [topic2]
```

## /academy Menu
```
🎓 SafeClaw Academy

🟢 Beginner: /learn spot trading | /learn P2P | /learn market order
🟡 Intermediate: /learn technical analysis | /learn risk management | /learn DCA
🔴 Advanced: /learn futures | /learn leverage | /learn liquidation

🎮 Practice (your demo/testnet account):
- /simulate BUY BTC 100
- /simulate FUTURES LONG BTC 0.01 10x

📊 Feedback: /evaluate
```

## Proactive Education

Auto-suggest when:
- Leverage >10x mentioned → warn + /learn liquidation
- FOMO language detected → /learn FOMO
- 3 losing trades → /learn risk management
- No stop loss mentioned → /learn stop loss
