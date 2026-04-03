# Automated Email Follow-Up — n8n Workflow

**Built by nullvish**

A plug-and-play n8n workflow that runs on a schedule, picks up new leads from a Google Sheet, generates a personalised follow-up email via Groq LLM, sends it through Gmail, marks the lead as contacted, and logs the Gmail thread ID for downstream reply tracking.

---

## Flow

```
Schedule Trigger
    └── Read Sheet Rows
            └── Filter – Blank STATUS Only
                    └── AI Agent  ←── Groq LLM (llama-3.3-70b-versatile)
                            └── Build HTML Email
                                    └── Gmail – Send Follow-Up
                                            └── Update Sheet – STATUS + Timestamp
                                                    └── Append to Email Log
```

---

## Node Breakdown

| Step | Node | What it does |
|---|---|---|
| 1 | Schedule Trigger | Fires on your configured interval |
| 2 | Read Sheet Rows | Pulls all rows from the `Leads` tab |
| 3 | Filter – Blank STATUS | Keeps only rows with no STATUS (new leads) |
| 4 | AI Agent + Groq | Generates a personalised email body |
| 5 | Build HTML Email | Wraps the body in a clean HTML template |
| 6 | Gmail – Send Follow-Up | Sends to the lead's email address |
| 7 | Update Sheet | Writes `Follow-Up Sent` + ISO timestamp to STATUS |
| 8 | Append to Email Log | Logs `lead_id`, `thread_id`, `email` for reply tracking |

The `thread_id` logged in step 8 is what an inbound reply-classifier workflow uses to match and route customer replies back to the right lead.

---

## Google Sheet Schema

### `Leads` tab

| Column | Description |
|---|---|
| `NAME` | Lead's name |
| `EMAIL` | Lead's email address |
| `SERVICE` | Type of service or inquiry |
| `LOCATION` | Location (optional) |
| `PHONE NUMBER` | Contact number (optional) |
| `DATE` | Preferred date or submission date |
| `STATUS` | Blank = new. Workflow writes `Follow-Up Sent` here |
| `TIMESTAMP OF EMAIL` | ISO timestamp of when the email was sent |

### `Email Log` tab

| Column | Description |
|---|---|
| `lead_id` | Gmail message ID |
| `thread_id` | Gmail thread ID — for inbound reply matching |
| `email` | Lead's email |
| `original_email` | Same as email (kept for flexibility) |
| `status` | Set to `SENT` on creation |

---

## Setup

### 1. Credentials

Set up these three credentials in n8n, then replace the placeholders in the workflow JSON:

| Placeholder | Replace with |
|---|---|
| `YOUR_GOOGLE_SHEETS_CREDENTIAL_ID` | Your Google Sheets OAuth2 credential ID |
| `YOUR_GMAIL_CREDENTIAL_ID` | Your Gmail OAuth2 credential ID |
| `YOUR_GROQ_CREDENTIAL_ID` | Your Groq API credential ID |

### 2. Google Sheet

- Create a sheet with the schema above
- Replace `YOUR_GOOGLE_SHEET_ID` with your actual Sheet ID (from the URL)
- Tab names must match exactly: `Leads` and `Email Log`

### 3. Branding

In the `Build HTML Email` code node, replace:
- `YOUR COMPANY NAME`
- `YOUR NAME`
- `YOUR COMPANY | YOUR CONTACT`

### 4. AI Prompt

The AI Agent prompt is generic by default. Customise it inside the node to match your business context, tone, and the fields you collect from leads.

### 5. Import & Activate

1. n8n → **Workflows → Import from file**
2. Select `workflows/automated-email-followup.json`
3. Reconnect all credentials
4. Set your desired schedule on the `Schedule Trigger` node
5. Activate

---

## Stack

- **n8n** — workflow engine
- **Google Sheets** — data layer
- **Groq** — LLM inference (llama-3.3-70b-versatile)
- **Gmail** — email delivery

## Possilbe Modifation
- **datasource** -  Xero or QuickBooks can be used as data source.
- **LLM** - LLM selection is a crucial selections for Ai automations . However in this case any available LLM can be used , given the quality of your prompt.
- **Sender** - Twilio based follow ups can be switched in requirement of SMS or whatsapp Based communtions . However one can use Telegram Node as well to send the messages.

---


