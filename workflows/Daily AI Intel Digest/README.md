
# Daily AI Intel Digest — n8n Workflow

Pulls from 10 RSS sources on a schedule, filters signal from noise by tier, generates a fully formatted dark-themed HTML email via GPT-4.1-mini, and delivers it to your inbox automatically.

No manual curation. No aggregator middleman. Raw feeds → LLM filter → your inbox.

---

## Flow

```
Schedule Trigger
    └── AI Agent ←── 10 RSS Feed Tools (parallel)
            |        ←── OpenAI GPT-4.1-mini
            └── Extract HTML Body
                    └── Send via Gmail
```

---

## Sources

| Tool | Feed |
|---|---|
| Hacker News | AI + LLM + model leak filter (50+ points) |
| TechCrunch AI | General AI news |
| The Decoder | AI analysis and research |
| OpenAI Blog | Model announcements |
| DeepMind Blog | Research publications |
| HuggingFace Blog | Model releases and open-source AI |
| GNews: Hardware | AI chip, VLSI, NPU security news |
| GNews: Automation | Enterprise AI demand signals |
| GNews: LLM Leak | Architecture leaks and model weight exposure |
| GNews: SOTA | State-of-the-art model releases |

---

## Content Tiers

The AI agent filters everything it reads into two tiers before writing:

**T1 — Always include:**
Model leaks, SOTA releases, architecture dumps, chip and hardware security breakthroughs.

**T2 — Fill remaining slots:**
Automation demand, API changes, infrastructure news, enterprise AI adoption.

**Dropped entirely:**
Funding rounds under $50M, PR fluff, repackaged press releases.

---

## Email Structure

The digest ships as a single Gmail-safe HTML email with four sections:

| Section | Content |
|---|---|
| 01 · Lead Story | Single most important story of the day |
| 02 · Deep Dive | 3-paragraph technical breakdown of the lead |
| 03 · Hardware / VLSI Corner | 4-card grid of chip and silicon news |
| 04 · Full Intel Feed | 2 T1 items + 5 T2 items with source links |

All layout is table-based with fully inline styles — renders correctly in Gmail, Outlook, and mobile clients.

---

## Setup

### 1. Credentials

| Placeholder | Replace with |
|---|---|
| `YOUR_OPENAI_CREDENTIAL_ID` | OpenAI API credential ID in n8n |
| `YOUR_GMAIL_CREDENTIAL_ID` | Gmail OAuth2 credential ID in n8n |

### 2. Recipients

In the `Send via Gmail` node, replace `YOUR_RECIPIENT_EMAILS` with comma-separated email addresses.

### 3. Branding

In the AI Agent prompt and HTML template, replace `YOUR COMPANY` with your name or company (appears in header and footer).

### 4. Schedule

Default is every 3 hours. Change the interval in the `Schedule Trigger` node to whatever cadence you need — daily at a fixed time is the recommended setup.

### 5. Import & Activate

1. n8n → **Workflows → Import from file**
2. Select `workflows/daily-ai-intel-digest.json`
3. Reconnect OpenAI and Gmail credentials
4. Set your schedule
5. Activate

---

## Stack

- **n8n** — workflow engine
- **OpenAI GPT-4.1-mini** — content filtering and HTML generation
- **RSS Feed Tool** — native n8n node for feed ingestion
- **Gmail** — delivery

## Modifications

- **News Sources** - You can modify News sources according to your own personal needs . Make it sure that the site allows web scraping , else most of the times an error will be returned
- **HTML** - You can create your own html frontend designs . include that design into AI AGENT prompt
- **Recipients** - Instead of Gmail , you can use Telegram or Twilio based services
- 

---

*nullvish*
