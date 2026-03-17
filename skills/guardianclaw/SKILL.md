---
name: guardianclaw
description: Pre-trade safety check for any crypto trade. Trigger when user sends /guard, asks to check a trade, wants to verify a token, asks if a trade is safe, or mentions buying/selling a specific token. Checks price, volume, liquidity, and psychological risk factors.
---

# GuardianClaw 🛡️

Pre-trade safety interceptor. Check any trade before executing it.

## /guard Command

Usage: /guard BUY BTC 100 USDT
Or: /guard SELL ETH 0.5
Or just: /guard BTCUSDT

When triggered, run all 4 safety checks and return a GO/NO-GO verdict.

## Check 1 — Token Price + 24h Data
```bash
SYMBOL="BTCUSDT"
curl -s "https://api.binance.com/api/v3/ticker/24hr?symbol=${SYMBOL}" | jq '{
  symbol: .symbol,
  price: .lastPrice,
  change24h: .priceChangePercent,
  high24h: .highPrice,
  low24h: .lowPrice,
  volume24h: .quoteVolume,
  trades24h: .count
}'
```

## Check 2 — Order Book Depth (Liquidity)
```bash
curl -s "https://api.binance.com/api/v3/depth?symbol=${SYMBOL}&limit=5" | jq '{
  topBid: .bids[0],
  topAsk: .asks[0],
  spread: ((.asks[0][0] | tonumber) - (.bids[0][0] | tonumber))
}'
```

## Check 3 — Recent Trades (Momentum)
```bash
curl -s "https://api.binance.com/api/v3/trades?symbol=${SYMBOL}&limit=10" | jq '[.[] | {price: .price, qty: .qty, isBuyerMaker: .isBuyerMaker}]'
```

## Check 4 — Psychology Check

Ask user 3 quick questions:
1. "Why are you making this trade?" (expect: strategy/plan vs FOMO)
2. "Is this part of your DCA plan or a new position?"
3. "What's your exit plan if price drops 20%?"

Flag FOMO if user says: moon, pump, everyone is buying, just saw, quick profit, can't miss

## Safety Scoring

Score trade out of 100:

| Factor | Weight | Safe | Warning |
|--------|--------|------|---------|
| 24h volume | 25% | >$1M | <$100k |
| Price change | 25% | -5% to +10% | >+20% (FOMO zone) |
| Bid/ask spread | 25% | <0.1% | >1% |
| Psychology | 25% | planned | FOMO signals |

## Verdict Logic

- Score 75-100 → 🟢 GO — Trade looks safe
- Score 50-74 → 🟡 CAUTION — Proceed carefully
- Score 0-49 → 🔴 NO-GO — High risk detected

## Response Format

Always respond with:
```
🛡️ GuardianClaw Safety Check
Token: BTCUSDT

📊 Market Data:
- Price: $71,530
- 24h Change: +1.19%
- 24h Volume: $2.1B
- Bid/Ask Spread: 0.01%

🧠 Risk Assessment:
- Liquidity: ✅ Excellent
- Momentum: ✅ Healthy
- Volatility: ✅ Normal
- Psychology: ✅ Planned trade

🎯 Safety Score: 87/100
Verdict: 🟢 GO — Trade looks safe

⚡ Recommendation:
[1-2 line specific advice]

Proceed? Reply YES to execute or NO to cancel.
```

## FOMO Detection Phrases

Flag these as FOMO signals:
- "everyone is buying", "going to moon", "pump incoming"
- "just saw on Twitter/X", "quick profit", "can't miss this"
- "all in", "last chance", "guaranteed"

When FOMO detected → always add:
"⚠️ FOMO Alert: Your reasoning shows emotional trading signals.
Consider: Is this in your original plan? Start with 25% of intended size."

## After Verdict

If user says YES → confirm trade details before executing
If user says NO → acknowledge and suggest better entry point
If NO-GO → explain exactly what the risk is
