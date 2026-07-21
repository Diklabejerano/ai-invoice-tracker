# AI Invoice Tracker

An AI tool that reads invoice PDFs from your Gmail, extracts the details automatically, and logs them to a Google Sheet. Handles Hebrew and English. Built with n8n + Anthropic Claude Sonnet 5.

## What it does

Every time an invoice PDF arrives at a labeled Gmail folder, the tool:

1. Downloads the PDF attachment automatically
2. Extracts the text from the PDF
3. Sends the text to Claude AI to check if it's really an invoice, and if so, extract 12 structured fields (vendor, tax ID, invoice number, dates, currency, amounts, tax, customer info)
4. Writes a row to your Google Sheet with all those fields
5. Sends you a confirmation email
6. If Claude can't process something, it's routed to an "Error" tab for review

The tool handles Hebrew (חשבונית מס, חשבונית מס קבלה, חשבונית עסקה) and English invoices side by side.

## Who it's for

Freelancers, subscription-heavy users, small business owners. Anyone tired of losing track of subscription and vendor invoices in their inbox until tax time.

## Why I built it

Losing invoices in your inbox is a real cost. Missed deductions, forgotten renewals, no visibility on total subscription spend. I built this to solve it for myself, then made it configurable so others can use it too.

What you get once it's running:
- Every invoice from your inbox automatically logged in one Google Sheet
- Month-end handoff to your accountant is a single sheet link
- Tax season isn't a scramble through 12 months of email
- Subscription creep visible at a glance (add a SUM formula to see your true monthly spend)

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

- Gmail account
- Google Sheets access (any Google account has this)
- Anthropic API key with $5+ in credits (details in Step 3)
- n8n Cloud account, free tier (details in Step 4)
- ~20 minutes for first-time setup

## Setup guide

Follow these 6 steps in order.

### Step 1: Get your Google Sheet template

The template is a pre-built Excel file. Import it into your Google Drive and convert it to a Google Sheet.

1. **Download the template** from this repo: [AI_Invoice_Tracker_Template.xlsx](https://github.com/Diklabejerano/ai-invoice-tracker/blob/main/AI_Invoice_Tracker_Template.xlsx). Click the link, then click the download icon in the top right of GitHub's file view to save it to your computer.
2. **Open Google Drive** in your browser: [drive.google.com](https://drive.google.com)
3. **Upload the file:** click **+ New** (top left) > **File upload** > select the `.xlsx` file you just downloaded
4. **Open with Google Sheets:** once uploaded, right-click the file in Drive > **Open with** > **Google Sheets**
5. **Convert to Google Sheets format:** in the Sheets menu, click **File** > **Save as Google Sheets**. A new native Google Sheet is created. You can delete the original .xlsx after this.
6. **Rename** the new Sheet to something like "AI Invoice Tracker".

You now have your own template. It has 2 tabs: `Sheet1` (successful invoices land here) and `Error` (extraction errors land here). Keep this tab open — you'll need to point n8n to it in Step 7.

### Step 2: Create a Gmail label

1. Open Gmail
2. In the left sidebar, scroll to the bottom of the labels list
3. Click **+ Create new label**
4. Name it `invoices` (or any name — just remember it)
5. Click **Create**

When invoice emails arrive in your inbox, add this label to them. You can do this manually, or set up a Gmail filter to auto-apply the label based on rules (e.g., emails from certain senders, or with "invoice" in the subject).

### Step 3: Get your Anthropic API key

**Part A — create the account and add credits**

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Sign up (Google login or email works)
3. Once logged in, click **Settings** in the left sidebar
4. Click **Billing**
5. Click **Add to credit balance**
6. Add **$5** (the minimum)
7. Enter payment method, confirm
8. **IMPORTANT:** on the billing page, look for **Auto-reload**. **Leave it OFF (or click Skip for now).** You don't want it to charge your card automatically if you run out. You can always add more later manually.

After paying, stay on the Anthropic Console. Do not close the tab.

**Part B — create your API key**

1. In the same Console, click **Settings** > **API Keys** in the left sidebar
2. Click **Create Key** (top right)
3. Give it a name like `n8n-invoice-tracker` and click **Create Key**
4. A popup appears with your key — a very long string starting with `sk-ant-api03-...`
5. Click the **Copy** button next to the key
6. **Paste it immediately** into a text file (Notepad) — you will NEVER see this key again once you close the popup
7. Save the text file somewhere safe on your computer
8. **Security note:** treat this key like a password. Do not share it, do not commit it to GitHub, do not paste it into public places. Anyone with this key can make API calls that charge your account.

### Step 4: Sign up for n8n Cloud

1. Go to [n8n.io](https://n8n.io)
2. Click **Get started for free** (top right)
3. If you already have an account, sign in. If not, sign up (Google login works)
4. If new: check your email for a verification code, enter it
5. After signup, you land on n8n Cloud's home page

You now have your own n8n instance running in the cloud.

### Step 5: Set up n8n credentials for Gmail and Google Sheets

n8n needs permission to read your Gmail and write to your Google Sheet. You create these credentials once, then any workflow you build can use them.

**Create Gmail credential:**

1. In n8n, click **Credentials** in the left sidebar
2. Click **Add credential** (or the **+** button, top right)
3. In the search box, type `Gmail`
4. Click **Gmail OAuth2 API**
5. n8n shows a popup with instructions. Click **Sign in with Google**
6. A Google popup opens. Sign in with the SAME Google account whose Gmail you want to monitor. Grant the permissions n8n asks for.
7. When you're back in n8n, name this credential `Gmail OAuth2` and click **Save**

**Create Google Sheets credential:**

1. Back in Credentials, click **Add credential** again
2. Type `Google Sheets`
3. Click **Google Sheets OAuth2 API**
4. Repeat the sign-in flow with the SAME Google account
5. Name it `Google Sheets OAuth2` and click **Save**

You now have two credentials saved. Both will appear whenever a node needs them.

### Step 6: Import the workflow

1. In n8n, click **Workflows** in the left sidebar
2. Click **Add workflow** (top right — this creates a new empty workflow)
3. The empty workflow canvas opens
4. Look at the top-right of the canvas. Click the **three-dot menu** (`⋯`) or **hamburger menu** (three horizontal lines)
5. Click **Import from File** (or "Import Workflow")
6. Choose `AI_Invoice_Tracker_workflow.json` from wherever you downloaded it
7. The full workflow appears on the canvas with all 9 nodes

**If you can't find "Import from File" in the menu:** some n8n Cloud versions moved it. Alternative:
1. Open `AI_Invoice_Tracker_workflow.json` in a text editor (Notepad works)
2. Select all (Ctrl+A) and copy (Ctrl+C)
3. Go back to the empty n8n workflow canvas
4. Paste (Ctrl+V) directly on the canvas — n8n will parse the JSON and build the workflow

### Step 7: Configure each node with your values

Click each node in the workflow and update these fields. Save (Ctrl+S) as you go.

| Node | What to change |
|---|---|
| Gmail Trigger | Credential: pick `Gmail OAuth2` from the dropdown. Then in Filters > Label, pick the label you created in Step 2. |
| Get a message | Credential: `Gmail OAuth2` |
| HTTP Request | Scroll to Send Headers > find the row with Name `x-api-key`. Replace `YOUR_ANTHROPIC_API_KEY_HERE` in the Value field with your actual key from Step 3. |
| Append row in sheet | Credential: `Google Sheets OAuth2`. Document: pick YOUR copy of the template. Sheet Name: pick the main tab. |
| Append row in sheet1 | Credential: `Google Sheets OAuth2`. Document: same sheet. Sheet Name: pick the `Error` tab. |
| Send a message | Credential: `Gmail OAuth2`. In the sendTo field, replace `YOUR_NOTIFICATION_EMAIL_HERE` with your email address. |

### Step 8: Activate the workflow and test

1. At the top-right of the workflow, toggle **Active** to ON
2. Send yourself a test email: attach any PDF invoice, apply the Gmail label from Step 2
3. Wait ~1 minute for the Gmail Trigger to poll for new emails
4. Check your Google Sheet — a new row should appear in the main tab with the extracted fields
5. Check your inbox — a confirmation email should arrive

If it worked, you're done. The workflow will now run automatically on every labeled invoice.

## Test with sample invoices

`test_invoices_anonymized/` contains 6 synthetic invoices (3 English + 3 Hebrew, mixed types and currencies). All names and amounts are fictional. Copy any file's text into a Word doc, export as PDF, email to yourself with the label.

## Troubleshooting

- **HTTP Request returns 401.** API key wrong or no credits. Check the key you pasted matches your Anthropic Console key exactly, and check Settings > Billing shows credits.
- **"Cannot read properties of undefined".** Response structure changed. Check the model name in the "Code in JavaScript" node is a current Claude model.
- **Google Sheets rate limit.** Increase the Wait node from 3 to 5+ seconds.
- **No PDF attachment found.** Email had no PDF. Skipped by design.
- **Nothing lands in sheet.** Check Executions log. Usually: workflow not Active, or a Google Sheets node points to the wrong document.
- **Import from File is missing.** Use the paste-JSON alternative in Step 6.

## Tech stack

n8n . Anthropic Claude Sonnet 5 . Gmail . Google Sheets

## License

MIT

## About

Built by Dikla Cohen Bejerano. CPA and MIS Director based in Israel. I build practical AI tools that solve real workflow problems. [LinkedIn](https://www.linkedin.com/in/dikla-cohen-3945364a/)

