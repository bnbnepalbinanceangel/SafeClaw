---
name: profile
description: Show user their current SafeClaw profile, settings, and Binance account summary. Trigger when user sends /profile, asks to see their settings, account info, balance, or portfolio overview.
---

# Profile 👤

Show the user their SafeClaw settings and Binance account summary.

## /profile Command

When user sends /profile:

### Step 1 — Show saved settings from USER.md
Display:
- Fiat currency
- Payment methods
- Risk profile
- API key status (configured/not set — NEVER show actual key)
- Member since date

### Step 2 — Fetch live Binance balance (if API key configured)

For testnet:
```bash
API_KEY="users_api_key"
SECRET="users_secret"
TIMESTAMP=$(date +%s%3N)
QUERY="timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$SECRET" | cut -d' ' -f2)

curl -s "https://testnet.binance.vision/api/v3/account?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${API_KEY}" | jq '[.balances[] | select((.free | tonumber) > 0) | {
    asset: .asset,
    free: .free,
    locked: .locked
  }] | .[0:10]'
```

For live:
```bash
curl -s "https://api.binance.com/api/v3/account?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${API_KEY}" | jq '[.balances[] | select((.free | tonumber) > 0.001) | {
    asset: .asset,
    free: .free
  }] | .[0:10]'
```

### Step 3 — Fetch current prices for held assets
```bash
curl -s "https://api.binance.com/api/v3/ticker/price" | jq '[.[] | select(.symbol | test("USDT$"))]'
```

### Step 4 — Format and display profile

Show this format:
```
👤 Your SafeClaw Profile

⚙️ Settings:
- Currency: NPR
- Payment: Khalti, eSewa  
- Risk: Moderate
- API Key: ✅ Configured

💼 Binance Balance:
- BTC: 1.00
- USDT: 10,000.00
- ETH: 1.00
- BNB: 1.00

📊 Quick Stats:
- Total assets: X
- Setup: Complete ✅

Use /help to see all commands.
```

## /settings Command

Allow user to update their profile:
- /settings currency NPR
- /settings risk conservative
- /settings payments Khalti eSewa

Update USER.md accordingly.

## Security Rules

- NEVER display API key or secret
- Only show apiKeyStatus: configured or not set
- Show balances only if API key is valid
- If API key missing: prompt user to run /updatekey
