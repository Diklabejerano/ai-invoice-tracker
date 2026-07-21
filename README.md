# AI Invoice Tracker

A tool I built to solve a real personal pain and made configurable so others can use it too. It watches your Gmail for invoices, reads them with AI, and logs the details into a Google Sheet automatically. Handles Hebrew and English. Built with n8n + Anthropic Claude Sonnet 5.

## Who it's for

Freelancers, subscription-heavy users, and small business owners. Anyone who wants every invoice hitting their inbox captured automatically in one Google Sheet.

## Why I built it

Losing invoices in your inbox is a real cost. Missed deductions at tax time, forgotten renewals, blind spots on subscription spend. I built this to solve the problem for myself, then made it configurable so others can use it too.

What you get once it's running:

- Every invoice from your inbox automatically logged in one Google Sheet
- Month-end handoff to your accountant is a single sheet link
- Tax season isn't a scramble through 12 months of email
- Subscription creep visible at a glance (add a SUM formula, see your true monthly spend)

## What it does

1. Watches a labeled folder in your Gmail for emails with PDF attachments
2. Extracts text from the PDF
3. Sends the text to Claude to classify (invoice or not) and extract 12 structured fields
4. If it's an invoice, writes a row to your Google Sheet and sends a confirmation email
5. If it's not an invoice, it's skipped
6. If extraction fails, it's routed to an "Error" tab for review

Recognizes Hebrew (חשבונית מס, חשבונית מס קבלה, חשבונית עסקה) and English tax invoices.

## How it works

```
Gmail label -> Get message + PDF -> Extract text -> Claude API
                                                        |
                                                        v
                                             Response has error? -- yes --> Error tab
                                                        |
                                                        no
                                                        v
                                              Is it an invoice? -- no --> skipped
                                                        |
                                                        yes
                                                        v
                                     Main sheet -> Wait 3s -> Confirmation email
```

## Prerequisites

- **n8n account.** Free tier works. [n8n.io](https://n8n.io)
- **Anthropic API key with $5+ credits.** [console.anthropic.com](https://console.anthropic.com). Cost is ~1-2 cents per invoice.
- **Google account** with Gmail and Google Sheets access
- **~20 minutes** for first-time setup

## Setup guide

### 1. Copy the Google Sheet template

**[Click here to copy the template](https://docs.google.com/spreadsheets/d/1Hk7WcoGkmCYSkHXpuJpANkaxbhffSWU1Y2gEJAp--7E/copy)**

Google will prompt "Make a copy". Click it. The template has 2 tabs: main sheet (successful invoices) and Error (extraction errors).

### 2. Create a Gmail label

Gmail > left sidebar > **+ Create new label**. Name it (e.g., `invoices`). Apply this label to invoice emails (manually or with a Gmail filter).

### 3. Get your Anthropic API key

[console.anthropic.com](https://console.anthropic.com) > sign up > Settings > Billing > add $5+ credits > Settings > API Keys > **Create Key**. **Copy it immediately** (you can only see it once).

### 4. Set up n8n credentials (do this once)

In n8n > **Credentials** > **New**:

- **Gmail OAuth2 API.** Sign in with your Google account, name it `Gmail OAuth2`
- **Google Sheets OAuth2 API.** Same account, name it `Google Sheets OAuth2`

### 5. Import the workflow and configure each node

**Import:** Workflows > **Add workflow** > menu (top-right three dots) > **Import from File** > select `AI_Invoice_Tracker_workflow.json`.

**Configure each node** (click each, update as listed, save):

| Node | What to change |
|---|---|
| Gmail Trigger | Credential: `Gmail OAuth2` . Filters > Label: pick your label from Step 2 |
| Get a message | Credential: `Gmail OAuth2` |
| HTTP Request | Header `x-api-key` value: replace `YOUR_ANTHROPIC_API_KEY_HERE` with your key from Step 3 |
| Append row in sheet (main) | Credential: `Google Sheets OAuth2` . Document: your sheet copy . Sheet Name: main tab |
| Append row in sheet1 (Error tab) | Credential: `Google Sheets OAuth2` . Document: same sheet . Sheet Name: `Error` |
| Send a message | Credential: `Gmail OAuth2` . sendTo: replace `YOUR_NOTIFICATION_EMAIL_HERE` with your email |

### 6. Activate and test

Save. Toggle **Active** (top-right). Email yourself a PDF invoice. Apply the Gmail label. Wait ~1 minute. Check your sheet and inbox.

## Test with sample invoices

`test_invoices_anonymized/` contains 6 synthetic invoices (3 English + 3 Hebrew, mixed types and currencies). All names and amounts are fictional. Copy any file's text into a Word doc, export as PDF, email to yourself.

## Troubleshooting

- **HTTP Request returns 401.** API key wrong or no credits. Recheck at console.anthropic.com.
- **"Cannot read properties of undefined".** Response structure changed. Check the model name in the "Code in JavaScript" node is a current Claude model.
- **Google Sheets rate limit.** Increase the Wait node from 3 to 5+ seconds.
- **No PDF attachment found.** Email had no PDF. Skipped by design.
- **Nothing lands in sheet.** Check Executions log. Usually: workflow not Active, or a Google Sheets node points to the wrong document.

## Tech stack

n8n . Anthropic Claude Sonnet 5 . Gmail . Google Sheets

## License

MIT

## About

Built by Dikla Cohen Bejerano. CPA and MIS Director based in Israel. I build practical AI tools that solve real workflow problems. [LinkedIn](https://www.linkedin.com/in/dikla-cohen-3945364a/)
