---
name: user-onboarding
description: Handle new user onboarding, API key setup, and profile configuration. Trigger when user sends /start, /setup, /updatekey, /updatekey-demo, /updatekey-testnet, /updatekey-square, or mentions setting up their account, adding API key, or configuring preferences.
---

# User Onboarding 🦞

Per-user isolated API key management. Every user has their OWN keys.

## CRITICAL — Per-User Key Isolation

- NEVER use server env vars for user Binance keys
- Each user's keys stored ONLY in their own session memory
- No user can ever access another user's keys
- Server env vars only for: OpenRouter, Encryption

## Commands

- `/start` — Welcome + onboarding wizard
- `/updatekey [KEY] [SECRET]` — Connect real Binance account
- `/updatekey-demo [KEY] [SECRET]` — Connect Binance Demo account
- `/updatekey-testnet [KEY] [SECRET]` — Connect Binance Testnet
- `/updatekey-square [KEY]` — Connect Binance Square posting
- `/profile` — View settings and balance
- `/help` — All commands

## Account Types

### Real Account (live trading)
- Endpoint: https://api.binance.com
- Command: /updatekey
- Permissions: Read + Spot Trade only
- NEVER: Withdrawals permission

### Demo Account (practice — recommended for beginners)
- Endpoint: https://demo-api.binance.com
- Command: /updatekey-demo
- Get keys: demo.binance.com
- Balance: 5,000 USDT provided by Binance

### Testnet (developers)
- Endpoint: https://testnet.binance.vision
- Command: /updatekey-testnet
- Get keys: testnet.binance.vision

## Onboarding Flow

### Step 1 — Welcome
Greet user, explain 5 features.
Ask: "What is your local currency?"
Options: NPR / INR / NGN / PHP / USD / Other

### Step 2 — Payment Methods
Based on currency:
- NPR: Khalti / eSewa / Fonepay / Bank Transfer
- INR: UPI / IMPS / Bank Transfer
- NGN: Bank Transfer / Opay
- Others: Bank Transfer

### Step 3 — Risk Profile
Options: Conservative / Moderate / Aggressive

### Step 4 — Account Type Selection
Ask: "Which Binance account do you want to connect?"
```
1. 🟢 Real Account — Live trading
2. 🟡 Demo Account — Practice (RECOMMENDED for beginners)
3. 🔵 Testnet — For developers
4. ⏭️ Skip — Use P2P and briefings only
```

### Step 5 — Setup Instructions by Type

Real Account:
```
1. Binance → Profile → API Management
2. Create key → Label: SafeClaw
3. Enable: Read Info + Spot Trade ONLY
4. Disable: Withdrawals
5. IP restrict: 3.109.214.201
6. Send: /updatekey YOUR_KEY YOUR_SECRET
```

Demo Account:
```
1. Go to demo.binance.com
2. API Management → Create Key
3. Label: SafeClaw-Demo
4. Send: /updatekey-demo YOUR_KEY YOUR_SECRET
You get 5,000 USDT to practice!
```

Testnet:
```
1. Go to testnet.binance.vision
2. Login with GitHub
3. Generate HMAC key
4. Send: /updatekey-testnet YOUR_KEY YOUR_SECRET
```

## /updatekey — Real Account Validation
```bash
USER_KEY="FIRST_ARGUMENT"
USER_SECRET="SECOND_ARGUMENT"
TIMESTAMP=$(date +%s%3N)
QUERY="timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s "https://api.binance.com/api/v3/account?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '{permissions: .permissions, canTrade: .canTrade}'
```

Checks:
- SPOT found → valid
- WITHDRAWALS found → REJECT + warn
- Error → invalid key

Store in session:
```
binance_live_key: [key]
binance_live_secret: [secret]
binance_account_type: live
binance_key_status: configured
```

## /updatekey-demo — Demo Account Validation
```bash
USER_KEY="FIRST_ARGUMENT"
USER_SECRET="SECOND_ARGUMENT"
TIMESTAMP=$(date +%s%3N)
QUERY="timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s "https://demo-api.binance.com/api/v3/account?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '{canTrade: .canTrade, permissions: .permissions}'
```

Store in session:
```
binance_demo_key: [key]
binance_demo_secret: [secret]
binance_account_type: demo
binance_key_status: configured
```

Confirm: "🟡 Demo account connected! Practice with $5,000 USDT."

## /updatekey-testnet — Testnet Validation
```bash
USER_KEY="FIRST_ARGUMENT"
USER_SECRET="SECOND_ARGUMENT"
TIMESTAMP=$(date +%s%3N)
QUERY="timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "$QUERY" | openssl dgst -sha256 -hmac "$USER_SECRET" | cut -d' ' -f2)
curl -s "https://testnet.binance.vision/api/v3/account?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${USER_KEY}" | jq '{canTrade: .canTrade}'
```

Store in session:
```
binance_testnet_key: [key]
binance_testnet_secret: [secret]
binance_account_type: testnet
binance_key_status: configured
```

## /updatekey-square — Square Posting Key

Store in session:
```
binance_square_key: [key]
square_key_status: configured
```

Show only: first5...last4 format.
Confirm: "✅ Binance Square connected. You can now post content."

## Security Rules

- NEVER echo API key or secret in any message
- NEVER store keys in server env vars
- NEVER share keys between users
- Always warn user to delete /updatekey message from chat
- Reject any key with withdrawal permissions
- Show only: apiKeyStatus configured/not set

## /help Command
```
🦞 SafeClaw Commands:

🔧 Setup:
/start — Begin setup
/updatekey — Real Binance account
/updatekey-demo — Demo account (beginners)
/updatekey-testnet — Testnet (developers)
/updatekey-square — Square content posting
/profile — View your settings

📊 Trading:
/p2p — Safe P2P merchants
/guard — Pre-trade safety check
/dca — SmartDCA setup
/briefing — Market update

🎓 Academy:
/simulate — Practice trading
/learn [topic] — Education
/evaluate — Score your trades

📝 Content:
/trending — Trending topics
/draft [topic] — Create Square post
/publish — Post to Square
```
