---
name: p2p-safefinder
description: Find and score safe Binance P2P merchants. Use when user asks to find P2P merchants, buy/sell crypto via P2P, check merchant safety, or find best P2P rates. Supports NPR, INR, NGN, USD and all major fiat currencies.
---

# P2P SafeFinder 🔍

Find the safest Binance P2P merchants with real-time safety scoring.

## Fetch Live P2P Merchants
```bash
curl -s --compressed -X POST "https://p2p.binance.com/bapi/c2c/v2/friendly/c2c/adv/search" \
  -H "Content-Type: application/json" \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
  -H "Accept: */*" \
  -H "Accept-Encoding: gzip, deflate, br" \
  -H "Origin: https://p2p.binance.com" \
  -H "Referer: https://p2p.binance.com/" \
  -d '{
    "asset": "USDT",
    "fiat": "NPR",
    "merchantCheck": false,
    "page": 1,
    "payTypes": [],
    "rows": 10,
    "tradeType": "BUY"
  }' | jq '[.data[] | {
    name: .advertiser.nickName,
    price: .adv.price,
    minAmount: .adv.minSingleTransAmount,
    maxAmount: .adv.maxSingleTransAmount,
    completionRate: .advertiser.monthFinishRate,
    totalOrders: .advertiser.monthOrderCount,
    isVerified: .advertiser.userType,
    payMethods: [.adv.tradeMethods[].tradeMethodName]
  }]'
```

## Safety Scoring Model

Score each merchant out of 100:

| Factor | Weight | Good | Warning |
|--------|--------|------|---------|
| Completion rate | 35% | >98% | <95% |
| Total orders | 25% | >500 | <50 |
| Account verified | 20% | merchant | user |
| Price vs market | 20% | <1% spread | >3% spread |

## Score Calculation
```
completion_score = completionRate * 35
orders_score = min(totalOrders/500, 1) * 25
verified_score = (isVerified == "merchant") ? 20 : 0
total = completion_score + orders_score + verified_score
```

## Risk Flags

- 🔴 completionRate < 0.90 → REJECT
- 🔴 totalOrders < 10 → REJECT
- 🟡 isVerified == "user" → CAUTION
- 🟡 completionRate < 0.95 → CAUTION
- 🟢 completionRate > 0.98 AND totalOrders > 100 → SAFE

## Always Do This

1. Run the curl command to get live data
2. Calculate safety score for each merchant
3. Rank by score highest to lowest
4. Show top 3 with scores and flags
5. Recommend the safest one with reason

## Supported Fiat Currencies

NPR, INR, NGN, PHP, VND, BDT, PKR, USD, EUR, GBP

## DNS Fix Required

If curl returns no data, the DNS may need fixing:
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
