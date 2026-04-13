# Automated Invoice Generator — n8n Workflow

Picks up new invoice requests from a Google Sheet, clones a Google Docs template, merges client data into it, exports it as a PDF, emails it to the client, and marks the row as completed. Fully scheduled. Zero manual steps.

---

## Flow

```
Schedule Trigger
    └── Get row(s) in sheet (status = blank)
            └── Mark In Progress
                    └── Copy Invoice Template (Google Drive)
                            └── Build Field Map (Code node)
                                    └── Merge Fields into Doc (Google Docs)
                                            └── Export as PDF (Google Drive)
                                                    └── Send Invoice Email (Gmail)
                                                            └── Mark Completed
```

---

## Node Breakdown

| Step | Node | What it does |
|---|---|---|
| 1 | Schedule Trigger | Fires every 5 hours |
| 2 | Get row(s) in sheet | Pulls rows where `status` is blank (new requests) |
| 3 | Mark In Progress | Writes `IN PROGRESS` to status — prevents double-processing |
| 4 | Copy Invoice Template | Clones your Google Docs invoice template in Drive |
| 5 | Build Field Map | JS code node — formats amount, generates invoice number, preps all fields |
| 6 | Merge Fields into Doc | Find-and-replace all `{{placeholders}}` in the cloned doc |
| 7 | Export as PDF | Downloads the filled doc as PDF binary |
| 8 | Send Invoice Email | Emails the PDF as an attachment to the client |
| 9 | Mark Completed | Writes `INVOICE_SENT` to status, matched by Timestamp |

---

## Google Sheet Schema

Sheet name: `Form Responses 1`

| Column | Description |
|---|---|
| `Timestamp` | Form submission timestamp — used as the unique match key for status update |
| `NAME` | Client name |
| `EMAIL` | Client email — invoice is sent here |
| `SERVICE` | Service description that appears on the invoice |
| `AMOUNT` | Numeric amount — formatted to 2 decimal places by the code node |
| `PHONE NUMBER` | Optional contact number |
| `NOTES` | Optional notes |
| `status` | Blank = new. `IN PROGRESS` = being processed. `INVOICE_SENT` = done |

---

## Google Docs Template

The invoice template uses these exact placeholders — the Merge Fields node does a find-and-replace on all of them:

| Placeholder | Replaced with |
|---|---|
| `{{InvoiceNo}}` | Auto-generated `INV-XXXXXX` |
| `{{Date}}` | Current date (long format) |
| `{{Name}}` | Client name |
| `{{Email}}` | Client email |
| `{{Service}}` | Service description |
| `{{Amount}}` | Formatted amount |

Note the leading/trailing spaces on `{{InvoiceNo}}` and `{{Amount}}` — the template must match exactly as written in the node.

---

## Setup

### 1. Credentials

| Placeholder | Replace with |
|---|---|
| `YOUR_GOOGLE_SHEETS_CREDENTIAL_ID` | Google Sheets OAuth2 credential ID in n8n |
| `YOUR_GOOGLE_DRIVE_CREDENTIAL_ID` | Google Drive OAuth2 credential ID in n8n |
| `YOUR_GOOGLE_DOCS_CREDENTIAL_ID` | Google Docs OAuth2 credential ID in n8n |
| `YOUR_GMAIL_CREDENTIAL_ID` | Gmail OAuth2 credential ID in n8n |

### 2. Sheet & Template IDs

| Placeholder | Replace with |
|---|---|
| `YOUR_GOOGLE_SHEET_ID` | Sheet ID from the URL of your invoice sheet |
| `YOUR_INVOICE_TEMPLATE_DOC_ID` | Doc ID of your Google Docs invoice template |

### 3. Sender identity

In the `Send Invoice Email` node, replace `YOUR NAME` in the email body with your name or company name.

### 4. Import & Activate

1. n8n → **Workflows → Import from file**
2. Select `workflows/automated-invoice-generator.json`
3. Reconnect all credentials
4. Set your schedule interval
5. Activate

---

## Stack

- **n8n** — workflow engine
- **Google Sheets** — invoice request queue
- **Google Drive** — template cloning and PDF export
- **Google Docs** — template merge
- **Gmail** — invoice delivery
## Possible Modifications
- **DATA SOURCE** - Instead of Google sheets , other data sources can be used such as Xero or QuickBooks . However for this particular task, google sheets is the best options due to API handling and reliablity.
- **SENDING SOURCE** - Instead of Gmail , other providers like outlook can be used . However Outlook and other providers have high restriction on API calling , so I definately suggest Gmail.


---

*nullvish*
