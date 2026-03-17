---
name: api-router
description: Internal routing layer for Binance API calls. Always use this to determine the correct API endpoint and credentials for the current user. Never hardcode API endpoints in other skills.
---

# API Router 🔀

Determines the correct Binance API endpoint and credentials for each user.

## How to Use in Every Skill

Before making ANY Binance API call, check the user's account type from their session memory:
```
if user has binance_account_type = "live":
    BASE_URL = "https://api.binance.com"
    FUTURES_URL = "https://fapi.binance.com"
    USER_KEY = user's live key from session
    USER_SECRET = user's live secret from session

elif user has binance_account_type = "demo":
    BASE_URL = "https://demo-api.binance.com"
    FUTURES_URL = "https://demo-fapi.binance.com"
    USER_KEY = user's demo key from session
    USER_SECRET = user's demo secret from session

elif user has binance_account_type = "testnet":
    BASE_URL = "https://testnet.binance.vision"
    FUTURES_URL = "https://testnet.binancefuture.com"
    USER_KEY = user's testnet key from session
    USER_SECRET = user's testnet secret from session

else:
    Show: "Please run /updatekey to connect your Binance account"
    Do NOT proceed with API call
```

## Account Type Labels
```
live     → 🟢 Real Binance Account
demo     → 🟡 Binance Demo Account
testnet  → 🔵 Binance Testnet
none     → ⚪ Not configured
```

## Always Show Account Type in Responses

When executing any trade or API call always show:
```
🔑 Account: [🟢 Real / 🟡 Demo / 🔵 Testnet]
```

## No Key Configured Response
```
⚠️ No Binance account connected.

To use this feature:
- /updatekey — Real Binance account
- /updatekey-demo — Practice account (RECOMMENDED for beginners)
- /updatekey-testnet — Developer testnet

P2P search and briefings work without an API key.
```

## Security Rules

- NEVER use server env vars for user trades
- ALWAYS read keys from user's session memory
- ALWAYS show which account type is active
- Server env vars are ONLY for infrastructure (OpenRouter, encryption)
