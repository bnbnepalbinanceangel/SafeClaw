---
name: square-content-engine
description: Binance Square content creation, SEO/AEO/GEO optimization, trending topic discovery, approval workflow, and publishing engine. Trigger when user sends /trending, /draft, /calendar, /publish, /schedule, /analytics, /content, asks to write a Binance Square post, find trending crypto topics, create content, schedule posts, or grow their Binance Square presence. Also trigger on "post to square", "draft a post", "what's trending in crypto".
---

# Square Content Engine 📝

AI-powered Binance Square content creation with trending discovery, SEO/AEO/GEO optimization, approval workflow, and direct publishing via official Binance Square API.

## Binance Square Publishing API

### Endpoint
```
POST https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add
```

### Required Headers
```
X-Square-OpenAPI-Key: [user's square api key]
Content-Type: application/json
clienttype: binanceSkill
```

### Request Body
```json
{
  "bodyTextOnly": "Your post content with #hashtags"
}
```

### Publish Command
```bash
SQUARE_KEY="${BINANCE_SQUARE_API_KEY}"
CONTENT="Your post content here"

curl -s -X POST "https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add" \
  -H "X-Square-OpenAPI-Key: ${SQUARE_KEY}" \
  -H "Content-Type: application/json" \
  -H "clienttype: binanceSkill" \
  -d "{\"bodyTextOnly\": \"${CONTENT}\"}" | jq '{code: .code, id: .data.id, message: .message}'
```

### Success Response
```json
{
  "code": "000000",
  "data": { "id": "298177291743282" }
}
```
Post URL: https://www.binance.com/square/post/{id}

### Error Codes
- 000000 → Success
- 10005 → Identity verification required
- 20002 / 20022 → Sensitive words detected
- 20013 → Content too long
- 220003 → API key not found
- 220004 → API key expired
- 220009 → Daily limit exceeded (100 posts/day)
- 220010 → Unsupported content type
- 2000001 → Account permanently blocked

## API Key Setup

When user first runs /content or /publish without a configured key:
1. Tell them: "You need a Binance Square Creator API key."
2. Guide them:
   - Go to Binance Square Creator Center
   - Click Create API Key
   - Copy the key
   - Send: /setkey [YOUR_SQUARE_KEY]
3. Store key as BINANCE_SQUARE_API_KEY in env
4. NEVER display full key — show only first 5 + last 4 chars: abc12...xyz9
5. Key is publishing-only — cannot access assets or trading

## Commands

- /trending — Discover trending crypto topics right now
- /draft [topic] — Generate 3 optimized post drafts
- /calendar — View content schedule
- /publish [draft_id] — Post approved draft now
- /schedule [draft_id] [time] — Schedule for later
- /analytics — View post performance
- /setkey [key] — Configure Square API key
- /content — Full menu

## /trending Command

Fetch from all sources simultaneously:

### Source 1 — BTC market data (always relevant)
```bash
curl -s "https://api.binance.com/api/v3/ticker/24hr?symbols=%5B%22BTCUSDT%22,%22ETHUSDT%22,%22BNBUSDT%22%5D" | jq '[.[] | {symbol: .symbol, change: .priceChangePercent, price: .lastPrice}]'
```

### Source 2 — Top movers (content angles)
```bash
curl -s "https://api.binance.com/api/v3/ticker/24hr" | jq '[.[] | select(.symbol | endswith("USDT")) | {symbol: .symbol, change: (.priceChangePercent | tonumber)}] | sort_by(.change) | reverse | .[0:5]'
```

### Source 3 — Fear & Greed sentiment
```bash
curl -s "https://api.alternative.me/fng/?limit=1" | jq '{sentiment: .data[0].value_classification, value: .data[0].value}'
```

### Source 4 — CoinTelegraph headlines
```bash
curl -s "https://cointelegraph.com/rss" | grep -o '<title><!\[CDATA\[[^\]]*\]\]></title>' | head -5 | sed 's/<title><!\[CDATA\[//g' | sed 's/\]\]><\/title>//g'
```

### Source 5 — CoinDesk headlines
```bash
curl -s "https://www.coindesk.com/arc/outboundfeeds/rss/" | grep -o '<title>[^<]*</title>' | grep -v "CoinDesk" | head -5 | sed 's/<title>//g' | sed 's/<\/title>//g'
```

### Trending Response Format
```
🔥 Trending Now — [DATE TIME NPT]

📊 Market Pulse:
- BTC: $XX,XXX (+X.X%) | ETH: $X,XXX (+X.X%)
- Market Mood: [Sentiment] (Fear & Greed: XX)

📰 Breaking Headlines:
1. [Headline 1 from CT/CD]
2. [Headline 2]
3. [Headline 3]

🚀 Top Movers:
- [Token]: +XX.X% — potential content angle
- [Token]: +XX.X% — potential content angle

💡 Best Content Opportunities Right Now:
1. [Topic] — [why trending] — [angle: educational/market/safety]
2. [Topic] — [why trending] — [angle]
3. [Topic] — [why trending] — [angle]

Ready to draft?
/draft 1 — [Topic 1]
/draft 2 — [Topic 2]
/draft 3 — [Topic 3]
/draft [your own topic]
```

## /draft Command

Usage: /draft P2P safety Nepal
Or: /draft BTC market update
Or: /draft 1 (from trending list)

### Always Generate 3 Variations

**Draft A — Educational (SEO-focused)**
- 300-400 characters (Square limit aware)
- Teaches one specific thing
- Starts with a question or surprising fact
- Includes numbered list
- AEO: answers a specific question AI assistants get asked
- Ends with Binance Academy link when relevant

**Draft B — Viral Hook (engagement-focused)**
- 100-150 characters
- Powerful hook in first line
- Bold claim backed by data
- One clear takeaway
- Strong call to action

**Draft C — Data-Driven (GEO-focused)**
- 200-250 characters
- Specific numbers and percentages
- Cites "Binance data" or market data
- Structured for AI to cite/quote
- Clear factual statements

### SEO/AEO/GEO Rules

**SEO:**
- Primary keyword in first 10 words
- Natural keyword density
- Searchable title-style opening

**AEO (Answer Engine Optimization):**
- Directly answers: "What is X?", "How to Y?", "Why does Z?"
- Use definition structure for key terms
- Numbered lists AI assistants can parse
- Include the question itself in the post

**GEO (Generative Engine Optimization):**
- Write factual statements AI will cite
- Format: "According to Binance, [specific fact]"
- Include date-stamped data: "As of [date], BTC is..."
- Authoritative, citation-worthy tone
- Avoid opinions — state facts AI can reference

### Compliance Filter (run on every draft)
Before showing any draft, verify:
- ❌ No price predictions as facts → reject
- ❌ No "you should buy/sell" → reject
- ❌ No "moon", "guaranteed", "100x" → reject
- ❌ No competitor names → reject
- ✅ Add "Not financial advice. DYOR." to market posts
- ✅ Educational framing: "historically", "data shows"

### Hashtag Selection (always 5-7)
Always include: #AIBinance #Binance
Add based on topic:
- BTC content: #Bitcoin #BTC #Crypto
- ETH content: #Ethereum #ETH #DeFi
- P2P content: #BinanceP2P #P2PTrading
- Education: #CryptoEducation #DYOR #LearnCrypto
- Nepal: #CryptoNepal #Nepal #NPR
- Market: #CryptoNews #CryptoMarket #Trading
- SafeClaw: #SafeClaw #OpenClaw

### Draft Response Format
```
📝 3 Drafts Ready: [TOPIC]
━━━━━━━━━━━━━━━━━━━━━━━

🅰️ Educational (SEO)
─────────────────────
[Full draft text — 300-400 chars]

#AIBinance #Binance [+ 3-5 more]
─────────────────────
SEO: [primary keyword] | AEO: answers "[question]"
Est. reach: High | Best time: 9am NPT

━━━━━━━━━━━━━━━━━━━━━━━
🅱️ Viral Hook
─────────────────────
[Full draft text — 100-150 chars]

#AIBinance #Binance [+ 3-5 more]
─────────────────────
Hook strength: X/10 | Best time: 6pm NPT

━━━━━━━━━━━━━━━━━━━━━━━
🅲 Data-Driven (GEO)
─────────────────────
[Full draft text — 200-250 chars]

#AIBinance #Binance [+ 3-5 more]
─────────────────────
GEO: citation-ready | AEO score: XX/100

━━━━━━━━━━━━━━━━━━━━━━━
Actions:
- "post A" — Publish Draft A now
- "post B" — Publish Draft B now
- "post C" — Publish Draft C now
- "schedule A 9am" — Schedule A for 9am
- "edit A: [changes]" — Modify Draft A
```

## Approval Workflow

NEVER auto-post without explicit user confirmation.

Flow:
1. Show all 3 drafts
2. User picks: "post A" or "schedule B 9am"
3. Show final preview: "Posting this to Binance Square:"
4. Ask: "Confirm? YES or NO"
5. Only on YES → execute the curl command
6. Show success: post URL https://www.binance.com/square/post/{id}
7. On error → show error code meaning and suggest fix

## /publish — Execute Post

When user confirms posting:
```bash
SQUARE_KEY="${BINANCE_SQUARE_API_KEY}"
# Escape special characters in content
CONTENT=$(echo "[POST_CONTENT]" | python3 -c "import sys,json; print(json.dumps(sys.stdin.read().strip()))")

curl -s -X POST "https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add" \
  -H "X-Square-OpenAPI-Key: ${SQUARE_KEY}" \
  -H "Content-Type: application/json" \
  -H "clienttype: binanceSkill" \
  -d "{\"bodyTextOnly\": ${CONTENT}}" | jq '{code: .code, postId: .data.id, message: .message}'
```

On success (code: "000000"):
```
✅ Posted to Binance Square!

🔗 View your post:
https://www.binance.com/square/post/{id}

📊 Track performance in /analytics
💡 Next post: /trending to find new topics
```

## /calendar Command

Show weekly content plan:
```
📅 Content Calendar

━━━ This Week ━━━
Mon 9am  — Market update (SCHEDULED ⏰)
Tue 9am  — Educational: P2P safety (DRAFT READY 📝)
Wed 6pm  — Viral: BTC milestone (PENDING ⏳)
Thu 9am  — SafeClaw feature highlight
Fri 6pm  — Weekly recap
Sat 10am — Community poll
Sun 9am  — Binance Academy spotlight

━━━ Content Mix Target ━━━
📚 Educational: 40%
📊 Market updates: 30%
🦞 SafeClaw/product: 20%
👥 Community: 10%

━━━ Actions ━━━
/draft [topic] — Add new content
/schedule [draft] [time] — Schedule a post
```

## /analytics Command
```
📊 Square Analytics

━━━ Last 7 Days ━━━
Posts published: X
Total views: X,XXX
Avg views/post: XXX
Best post: "[title]" (X,XXX views)

━━━ Best Performing Types ━━━
1. Educational: avg XXX views
2. Market updates: avg XXX views
3. Viral hooks: avg XXX views

━━━ Optimal Posting Times ━━━
Weekdays: 9am, 1pm, 6pm NPT
Weekends: 10am, 7pm NPT

━━━ Top Hashtags ━━━
#AIBinance — XX posts
#BinanceP2P — XX posts

━━━ Recommendations ━━━
- Post more [type] — highest engagement
- Best day: [day]
- Next trending topic: /trending
```

## Post Templates

### Template 1 — P2P Safety (Nepal)
```
🇳🇵 Nepal P2P traders — avoid these red flags:

❌ Completion rate below 90% → skip
❌ Less than 50 total orders → risky
❌ No verified badge → caution
❌ Price spread >3% → overpriced

✅ Safe merchants: 98%+ completion, 500+ orders

Powered by SafeClaw 🦞 — AI safety scoring for Binance P2P

Not financial advice. DYOR.
#BinanceP2P #Nepal #CryptoNepal #AIBinance #Binance
```

### Template 2 — Daily Briefing
```
📊 Crypto Briefing [DATE]

₿ BTC: $XX,XXX (+X.X%)
Ξ ETH: $X,XXX (+X.X%)
😨 Fear & Greed: XX ([classification])

Top mover: [TOKEN] +XX.X%
NPR/USDT: XXX NPR (Binance P2P)

[1 line market insight]

Not financial advice. DYOR. 🦞
#Binance #Bitcoin #CryptoNews #AIBinance
```

### Template 3 — Educational
```
❓ [Question about crypto topic]?

Here's the simple answer:

[Clear 1-2 sentence explanation]

Key points:
1️⃣ [Point 1]
2️⃣ [Point 2]
3️⃣ [Point 3]

Learn more: academy.binance.com 📚

#CryptoEducation #Binance #DYOR #AIBinance
```

## Security Rules

- NEVER display full Square API key
- Show only: first5...last4 format
- Key is publish-only — not linked to funds
- Regenerate immediately if exposed
- Always human approval before posting
- Max 100 posts per day (platform limit)
- Text-only posts supported currently
