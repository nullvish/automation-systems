# automation-systems

Some things are done manually. Those things are done wrong.

This is a collection of production automation workflows — built to eliminate repetitive operations, handle real edge cases, and scale without adding headcount.

---

## Workflows

| Workflow | Description |
|---|---|
| [Automated Email Follow-Up](./workflows/automated-email-followup/) | Picks up new leads from a sheet, generates a personalised follow-up email via LLM, sends it, and logs the thread for inbound reply tracking. Fully scheduled. Zero manual touch. |
| [Weekly AI Business Summary](./workflows/weekly-ai-business-summary/) | Pulls weekly sheet data, computes metrics in pure JS, generates a bar chart, runs Groq analysis, and delivers a formatted HTML report via Gmail. |
| [Daily AI Intel Digest](./workflows/Daily%20AI%20Intel%20Digest/) | Pulls from 10 RSS sources every 3 hours, filters by tier, generates a dark-themed HTML digest via GPT-4.1-mini, and delivers it via Gmail. |
| [Automated Invoice Generator](./workflows/automated-invoice-generator-sender/) | Clones a Google Docs template, merges client data, exports as PDF, and emails the invoice automatically on a schedule. |
| [LinkedIn Content Generator](./workflows/linkedin-content-generator/) | Takes a topic from a sheet, runs 5 sequential Tavily searches across history, mechanics, current state, opportunities, and future — feeds all 25 source chunks into a single LLM prompt, generates a structured post, writes it back to the sheet, and pushes it to Telegram. |


---

## Structure

Each workflow lives in its own folder with:
- The export file — ready to drop into your automation platform
- A `README.md` covering the architecture, data schema, and setup

---

## What you'll find here

Real systems. Different stacks. Whatever the problem demands.

---

*nullvish*


*Building Systems.Breaking Systems.Building Systems*
