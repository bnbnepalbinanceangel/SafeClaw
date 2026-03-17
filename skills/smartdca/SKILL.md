---
name: smartdca
description: Smart Dollar Cost Averaging for Binance. Trigger when user sends /dca, asks to set up recurring buys, automate purchases, schedule crypto buying, or asks about DCA strategy. Supports real, demo, and testnet accounts per user.
---

# SmartDCA 💰

Automated Dollar Cost Averaging. Works with real, demo, or testnet accounts.

## CRITICAL — Per-User Account Routing

Before executing ANY trade, check user's account type from session memory:
```
if binance_account_type == "live":
    BASE_URL = "https://api.binance.com"
    USER_KEY = binance_live_key from session
    USER_SECRET = binance_live_secret from session
    ACCOUNT_LABEL = "🟢 Real Account"

elif binance_account_type == "demo":
    BASE_URL = "https://demo-api.binance.com"
    USER_KEY = binance_demo_key from session
    USER_SECRET = binance_demo_secret from session
    ACCOUNT_LABEL = "🟡 Demo Account"

elif binance_account_type == "testnet":
    BASE_URL = "https://testnet.binance.vision"
    USER_KEY = binance_testnet_key from session
    USER_SECRET = binance_testnet_secret from session
    ACCOUNT_LABEL = "🔵 Testnet"

else:
    Show: "Please run /updatekey first to connect your Binance account"
    STOP — do not execute any trade
```

## Commands

- /dca setup — Configure DCA plan
- /dca status — View current plan + performance
- /dca run — Execute one DCA buy now
- /dca stop — Pause DCA
- /dca history — View past trades

## /dca setup Flow

Ask user:
1. "Which asset? BTC / ETH / BNB / SOL"
2. "How much USDT per buy? (e.g. 10, 25, 50, 100)"
3. "How often? Daily / Weekly / Monthly"
4. "Adaptive mode? (buys more during fear, less during greed)"

Save to session:
```
dca_asset: BTC
dca_amount: 50
dca_interval: weekly
dca_adaptive: true
dca_status: active
dca_next_run: [date]
```

## /dca run — Execute DCA Buy

### Step 1 — Determine user account type
Read binance_account_type from session.
If not set → ask user to run /updatekey first.

### Step 2 — Check Fear & Greed (Adaptive Mode)
```bash
curl -s "https://api.alternative.me/fng/?limit=1" | jq '{
  value: .data[0].value,
  classification: .data[0].value_classification
}'
```

### Step 3 — Calculate Adaptive Amount
```
Fear 0-25 (Extreme Fear) → 2x amount
Fear 26-45 (Fear) → 1.5x amount
Fear 46-55 (Neutral) → 1x amount
Fear 56-75 (Greed) → 0.75x amount
Fear 76-100 (Extreme Greed) → 0.5x amount
```

### Step 4 — Get Current Price
```bash
curl -s "https://api.binance.com/api/v3/ticker/price?symbol=BTCUSDT" | jq '.price'
```

### Step 5 — GuardianClaw Safety Check
Check 24h price change is not >20% spike before executing.

### Step 6 — Execute Buy using user's account
```bash
USER_KEY="[from user session]"
USER_SECRET="[from user session]"
BASE_URL="[from account type]"
SYMBOL="BTCUSDT"
QUOTE_QTY="[calculated adaptive amount]"

TIMESTAMP=$(date +%s%3N)
QUERY="symbol=${SYMBOL}&side=BUY&type=MARKET&quoteOrderQty=${QUOTE_QTY}&timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)

curl -s -X POST "${BASE_URL}/api/v3/order?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '{
    orderId: .orderId,
    symbol: .symbol,
    status: .status,
    executedQty: .executedQty,
    cummulativeQuoteQty: .cummulativeQuoteQty
  }'
```

### Step 7 — Confirmation Format
```
✅ DCA Buy Executed

🔑 Account: [🟢 Real / 🟡 Demo / 🔵 Testnet]
📊 Order:
- Asset: BTC
- Spent: XX USDT (1.5x adaptive — Fear index: 32)
- Received: 0.000XXX BTC
- Price: $XX,XXX
- Order ID: XXXXXXX

📈 DCA Stats:
- Total invested: XXX USDT
- Total BTC: 0.00XXXX
- Avg buy price: $XX,XXX
- Current price: $XX,XXX
- P&L: +/-X%

⏰ Next run: [date]
```

## /dca status

Show plan + performance from session data.
Always show which account type is active.

## /dca history

Show last 10 trades from session memory with:
date, asset, spent, received, price, fear index, adaptive multiplier

## Safety Rules

- ALWAYS confirm with user before executing on real account
- ALWAYS show account type label before any trade
- NEVER execute if no account connected
- NEVER use server env vars for user keys
- Log every trade to session memory
