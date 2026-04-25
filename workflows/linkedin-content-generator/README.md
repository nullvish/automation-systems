# LinkedIn Content Generator — n8n Workflow

Takes a topic from a Google Sheet, runs five sequential Tavily searches across different research angles, feeds all the results into a single LLM chain, generates a structured LinkedIn post, writes it back to the sheet, and pushes it to Telegram. Fully automated on a daily schedule.

---

## Flow

```
Schedule Trigger (or Manual)
    └── Get row(s) in sheet (STATUS = blank)
            └── HISTORY search (Tavily)
                    └── WORKING search (Tavily)
                            └── CURRENT search (Tavily)
                                    └── OPPORTUNITY search (Tavily)
                                            └── FUTURE search (Tavily)
                                                    └── Basic LLM Chain ←── OpenAI
                                                            └── Append or update row in sheet
                                                                    └── Send a text message (Telegram)
```

---

## Why Linear Chain Instead of Parallel

The five Tavily searches run sequentially — HISTORY → WORKING → CURRENT → OPPORTUNITY → FUTURE — not in parallel branches.

The reason is data dependency at the LLM prompt level.

The Basic LLM Chain prompt references all five search nodes by name using `$('HISTORY').item.json`, `$('CURRENT').item.json`, etc. In n8n, an expression like `$('NodeName').item` resolves to the item that was processed *in the same execution path* as the current node. If the searches ran in parallel branches and were then merged, the item context becomes ambiguous — n8n cannot guarantee which branch's item a given expression resolves to without a Merge node and explicit pairing logic.

Running them linearly means every node in the chain shares the same item context. `$('HISTORY').item` always points to exactly the right search result because there is only one execution path, one item, and no branching that could corrupt the reference.

The tradeoff is execution time — five sequential API calls instead of five parallel ones. For a daily content workflow firing once at 06:00, this is irrelevant. Total runtime is under 30 seconds regardless.

If this were a high-volume pipeline processing hundreds of items per hour, parallel branches with a Merge node would be worth the added complexity. For a daily single-item content workflow, linear is the correct call.

---

## Research Structure

Each Tavily search targets a specific angle of the topic:

| Node | Query angle |
|---|---|
| HISTORY | Origins, background, evolution |
| WORKING | Internal mechanics, system structure, drivers |
| CURRENT | Present state, key players, live trends |
| OPPORTUNITY | Leverage points, risks, challenges |
| FUTURE | Trajectory, predictions, contrarian signals |

5 articles per angle feed into the LLM prompt. 25 source chunks total per post.

---

## Post Structure

The LLM is instructed to follow this exact structure — visible in rhythm, not in labels:

1. **Opening line** — one sentence, no warmup, drops into tension or contradiction
2. **The claim** — one arguable, specific thesis
3. **The evidence** — 3–5 discrete ideas, no connectors
4. **The inversion** — why the obvious read is incomplete
5. **The close** — one inevitable conclusion

No inspirational filler. No generic openers. Every sentence earns its place.

---

## Google Sheet Schema

Sheet name: `Sheet1`

| Column | Description |
|---|---|
| `TOPIC` | The topic to research and write about — used as the match key |
| `STATUS` | Blank = new. `CONTENT_MADE` = processed |
| `CONTENT` | The generated LinkedIn post written back by the workflow |

---

## Setup

### 1. Credentials

| Placeholder | Replace with |
|---|---|
| `YOUR_GOOGLE_SHEETS_CREDENTIAL_ID` | Google Sheets OAuth2 credential ID in n8n |
| `YOUR_OPENAI_CREDENTIAL_ID` | OpenAI API credential ID in n8n |
| `YOUR_TELEGRAM_CREDENTIAL_ID` | Telegram Bot API credential ID in n8n |

### 2. API Keys & IDs

| Placeholder | Replace with |
|---|---|
| `YOUR_TAVILY_API_KEY` | Your Tavily API key (appears in all 5 HTTP Request nodes) |
| `YOUR_GOOGLE_SHEET_ID` | Sheet ID from the URL of your content sheet |
| `YOUR_TELEGRAM_CHAT_ID` | Your Telegram chat or channel ID |

### 3. Model

The workflow uses `gpt-4o-mini` by default. Swap to any OpenAI model in the `OpenAI Chat Model` node. Groq with `llama-3.3-70b-versatile` works as a free alternative — replace the OpenAI node with a Groq Chat Model node.

### 4. Schedule

Default fires daily at 06:00. Adjust in the `Schedule Trigger` node. A manual trigger is also wired in for testing.

### 5. Import & Activate

1. n8n → **Workflows → Import from file**
2. Select `workflows/linkedin-content-generator.json`
3. Reconnect all credentials
4. Replace all API key placeholders
5. Activate

---

## Stack

- **n8n** — workflow engine
- **Tavily** — web search API (5 targeted queries per run)
- **OpenAI** — post generation (gpt-4o-mini)
- **Google Sheets** — topic queue and content storage
- **Telegram** — delivery

---

*nullvish*
