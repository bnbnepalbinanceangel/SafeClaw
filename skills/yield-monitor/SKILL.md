---
name: yield-monitor
description: Monitor Binance Earn yields, staking positions, and savings products. Trigger when user sends /yield, /earn, asks about their Binance Earn positions, staking rewards, flexible savings, locked products, or passive income from crypto.
---

# Yield Monitor 💎

Track and optimize your Binance Earn positions.

## CRITICAL — Per-User Account Routing

Only works with real Binance account (live):
```
if binance_account_type == "live":
    BASE_URL = "https://api.binance.com"
    USER_KEY = binance_live_key from session
    USER_SECRET = binance_live_secret from session
else:
    Show: "Yield monitoring requires a real Binance account.
    Run /updatekey to connect your live account."
    STOP
```

## Commands

- /yield — Overview of all Earn positions
- /yield flexible — Flexible savings positions
- /yield locked — Locked staking positions
- /yield rates — Best current APY rates
- /yield optimize — Suggest better yield opportunities

## /yield — Full Earn Overview

### Fetch flexible savings positions
```bash
USER_KEY="[from session]"
USER_SECRET="[from session]"
TIMESTAMP=$(date +%s%3N)
QUERY="timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s "https://api.binance.com/sapi/v1/lending/union/account?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '{
    totalAmountInUSDT: .totalAmountInUSDT,
    totalFixedAmountInUSDT: .totalFixedAmountInUSDT,
    totalFlexibleInUSDT: .totalFlexibleInUSDT
  }'
```

### Fetch current flexible savings rates
```bash
curl -s "https://api.binance.com/sapi/v1/lending/daily/product/list?status=SUBSCRIBABLE&asset=USDT" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '[.[0:5] | .[] | {asset: .asset, latestAnnualInterestRate: .latestAnnualInterestRate}]'
```

### Fetch locked staking positions
```bash
TIMESTAMP=$(date +%s%3N)
QUERY="timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s "https://api.binance.com/sapi/v1/staking/position?product=STAKING&timestamp=${TIMESTAMP}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '[.[0:5] | .[] | {asset: .asset, amount: .amount, apy: .apy, endTime: .endTime}]'
```

## /yield Response Format
```
💎 Your Binance Earn Overview

🔑 Account: 🟢 Real Account

━━━ Summary ━━━
- Total in Earn: $XX,XXX USDT
- Flexible savings: $X,XXX
- Locked staking: $X,XXX
- Est. monthly yield: ~$XX

━━━ Active Positions ━━━
📦 Flexible Savings:
- USDT: X,XXX @ X.XX% APY
- BNB: X.XX @ X.XX% APY

🔒 Locked Staking:
- BTC: X.XXXX @ X.XX% APY (unlocks: [date])
- ETH: X.XX @ X.XX% APY (unlocks: [date])

━━━ Best Rates Right Now ━━━
- USDT Flexible: X.XX% APY
- BNB Locked 30d: X.XX% APY
- ETH Locked 60d: X.XX% APY

💡 Optimization Tip:
[Personalized suggestion based on their holdings]

/yield optimize — See better opportunities
/yield rates — Browse all current rates
```

## /yield rates — Browse Current Rates
```bash
curl -s "https://api.binance.com/sapi/v1/lending/daily/product/list?status=SUBSCRIBABLE" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '[.[] | select(.latestAnnualInterestRate != "0") | {
    asset: .asset,
    apy: .latestAnnualInterestRate,
    minAmount: .minPurchaseAmount
  }] | sort_by(.apy | tonumber) | reverse | .[0:10]'
```

## /yield optimize — Suggestions

Compare user's current positions vs available rates.
Suggest moving funds if APY improvement > 1%.

Format:
```
💡 Yield Optimization

Current: USDT flexible @ X.XX% APY
Better option: USDT locked 30d @ X.XX% APY
Improvement: +X.XX% APY
Extra monthly: +$X.XX on $X,XXX

Want to switch? I can guide you through it.
(Note: locked products have fixed terms)
```

## Education — What is Binance Earn?

When user asks about Earn for first time:
```
💎 Binance Earn lets your crypto work for you:

🟢 Flexible Savings:
- Deposit anytime, withdraw anytime
- Lower APY (0.5-3%)
- Good for funds you might need

🔒 Locked Staking:
- Lock for 30/60/90/120 days
- Higher APY (2-15%+)
- Cannot withdraw early

📚 Learn more: academy.binance.com/en/articles/what-is-binance-earn
```

## Security Rules

- Only works with real account — never demo/testnet
- Never execute any Earn transactions without explicit confirmation
- Always show APY as annual (not daily)
- Always mention lock periods for locked products
