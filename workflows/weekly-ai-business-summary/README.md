
# Weekly AI Business Summary — n8n Workflow

Pulls a week of business data from Google Sheets, runs pure JS arithmetic for metrics, generates a bar chart via QuickChart, feeds everything into Groq LLM for a structured analysis, and delivers a fully formatted HTML report to your inbox — every week, automatically.

No AI does the math. The LLM only interprets. Numbers are always accurate.

---

## Flow

```
Schedule Trigger
    └── Pull Sheet Data (Google Sheets)
            └── Calculate Metrics — Pure Math (Code node)
                    └── Build Chart URL (Code node)
                            └── HTTP Request (fetch chart image)
                                    └── Basic LLM Chain ←── Groq (llama-3.3-70b-versatile)
                                            └── Send a message (Gmail)
```

---

## Node Breakdown

| Step | Node | What it does |
|---|---|---|
| 1 | Schedule Trigger | Fires at your set time weekly |
| 2 | Pull Sheet Data | Reads all rows from your sheet |
| 3 | Calculate Metrics | Pure JS — splits 14 days into this week / last week, computes revenue, spend, profit, ROI, WoW delta |
| 4 | Build Chart URL | Constructs a QuickChart bar chart URL from the metrics |
| 5 | HTTP Request | Fetches the chart image |
| 6 | Basic LLM Chain + Groq | Generates structured 7-section business analysis in HTML |
| 7 | Gmail | Sends the full dark-themed HTML report to your recipients |

---

## Google Sheet Schema

Sheet name: `Sheet1`

| Column | Description |
|---|---|
| `DATE` | Entry date — format `YYYY-MM-DD` |
| `NAME` | Job or entry name |
| `REVENUE` | Revenue figure (numeric) |
| `SPENT` | Spend figure (numeric) |

---

## AI Analysis Structure

The LLM delivers exactly 7 sections every week:

1. Weekly Performance Analysis
2. Job-Level Intelligence
3. Spend Efficiency Audit
4. Red Flags
5. One Recommendation
6. Executive Summary
7. Closing

Every claim is anchored to a number. No filler, no generic advice.

---

## Setup

### 1. Credentials

| Placeholder | Replace with |
|---|---|
| `YOUR_GOOGLE_SHEETS_CREDENTIAL_ID` | Google Sheets OAuth2 credential ID in n8n |
| `YOUR_GROQ_CREDENTIAL_ID` | Groq API credential ID in n8n |
| `YOUR_GMAIL_CREDENTIAL_ID` | Gmail OAuth2 credential ID in n8n |

### 2. Google Sheet

- Replace `YOUR_GOOGLE_SHEET_ID` with your actual Sheet ID (from the URL)
- Columns must be exactly: `DATE`, `NAME`, `REVENUE`, `SPENT`

### 3. Recipients & Branding

In the `Send a message` node:
- Replace `YOUR_RECIPIENT_EMAILS` with comma-separated email addresses
- Replace `YOUR SENDER NAME` with the display name for the sender

In the HTML template inside the same node:
- Replace `YOUR COMPANY` (appears in header and footer)

### 4. Schedule

The trigger is set to fire at **17:20** by default. Adjust in the `Schedule Trigger` node to whatever day and time you need.

### 5. Import & Activate

1. n8n → **Workflows → Import from file**
2. Select `workflows/weekly-ai-business-summary.json`
3. Reconnect all credentials
4. Set your schedule
5. Activate

---

## Stack

- **n8n** — workflow engine
- **Google Sheets** — data source
- **Groq** — LLM inference (llama-3.3-70b-versatile)
- **QuickChart** — chart generation (no API key needed)
- **Gmail** — report delivery

## Possible Modifications
- **DataSource** - Xero or QuickBooks can be used
- **LLM** - LLM focused primarily on analysis is suggested
- **Sender** - Twilio or Telegram nodes can be used instead of Gmail
- **Graphs** - Various more graphs can be generated
- **Tweaks** - The analysis can be bundled in pdf and sent with a pdf aswell
---

*nullvish*
