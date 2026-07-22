# AI Invoice Tracker

An AI tool that reads invoice PDFs from your Gmail, extracts the relevant fields (vendor, amount, tax, dates, currency, and more), and logs them to a Google Sheet automatically. Handles Hebrew and English. Built with n8n + Anthropic Claude Sonnet 5.

> **This is a portfolio project and working reference implementation — not a plug-and-play product.** Setup takes about 45-60 minutes and requires comfort with n8n Cloud + creating OAuth credentials for Gmail and Google Sheets. If you want to try it, follow the setup guide below — it's tested and works, but expect several clicks through menus. If you don't have time for setup, the code and workflow structure show the pattern you can adapt for your own build.

## What it does

Every time an invoice PDF arrives at a labeled Gmail folder, the tool:

1. Downloads the PDF attachment automatically
2. Extracts the text from the PDF
3. Sends the text to Claude AI to check if it really is an invoice, and if so, extract 12 structured fields (vendor, tax ID, invoice number, invoice date, due date, currency, subtotal, tax, total, customer info, and a confidence score)
4. Writes a row to your Google Sheet with all those fields
5. Sends you a confirmation email with the extracted details
6. If Claude can't process something, the failure is routed to an "Error" tab for review

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
- Anthropic API key with $5 in credits (details in Step 3)
- n8n Cloud account, free tier (details in Step 4)
- ~45-60 minutes for first-time setup (n8n Cloud OAuth flows + node configuration)

## Setup guide

**This is the most important part of the README.** Follow all 6 steps in order. Do not skip.

### Step 1: Get your Google Sheet template

The template is a pre-built Excel file with the correct tabs and column headers. Import it into your Google Drive and convert it to a native Google Sheet.

1. **Download the template file** from this repo: [AI_Invoice_Tracker_Template.xlsx](https://github.com/Diklabejerano/ai-invoice-tracker/blob/main/AI_Invoice_Tracker_Template.xlsx). Click the link, then click the download icon at the top-right of GitHub's file view to save it to your computer.
2. **Open Google Drive** in your browser: [drive.google.com](https://drive.google.com)
3. **Upload the file:** click **+ New** (top-left) > **File upload** > select the `.xlsx` file you just downloaded
4. **Open with Google Sheets:** in Drive, right-click the file you just uploaded > **Open with** > **Google Sheets**
5. **Convert to Google Sheets format:** in the Sheets menu bar, click **File** > **Save as Google Sheets**. A new native Google Sheet is created. You can delete the original .xlsx after this if you want.
6. **Rename** the new Sheet to something like "AI Invoice Tracker".

You now have your own template with 2 tabs: `Sheet1` (successful invoices land here) and `Error` (extraction errors land here). Keep this tab open — you'll point n8n to it in Step 7.

### Step 2: Create a Gmail label AND a filter to auto-apply it

**Important:** the workflow only processes emails that ALREADY have the label applied. The label is not applied automatically by Gmail. You need to either apply it manually to each invoice email, OR create a Gmail filter that auto-applies the label to matching emails.

**Part A — create the label:**

1. Open Gmail
2. In the left sidebar, scroll to the bottom of the labels list
3. Click **+ Create new label**
4. Name it `invoices` (or any name — just remember it)
5. Click **Create**

**Part B — create a filter to auto-apply the label (recommended, so you don't have to label each email manually):**

1. In Gmail, click the gear icon (top-right) > **See all settings**
2. Click the **Filters and Blocked Addresses** tab
3. Click **Create a new filter**
4. In the **Has the words** field, enter: `invoice OR receipt OR חשבונית` (add more terms as needed)
5. Check the **Has attachment** box (so it only matches emails with attachments — usually PDFs)
6. Click **Create filter** at the bottom
7. On the next screen, check **Apply the label:** and select `invoices` from the dropdown
8. Click **Create filter**

From now on, any incoming email matching those criteria gets the label automatically, and the workflow processes it.

**Part C — for testing right now:**

If you want to test immediately without waiting for a filter to trigger, manually apply the label to any invoice email that's already in your inbox:
1. Open the email
2. Click the label icon at the top (looks like a tag)
3. Select `invoices` from the list

The workflow will detect and process it within 1 minute.

### Step 3: Get your Anthropic API key

This step has 2 parts on the same website (console.anthropic.com). Complete both.

**Part A — create the account and add credits**

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Sign up (Google login or email — Google is faster)
3. Once logged in, click **Settings** in the left sidebar
4. Click **Billing**
5. Click **Add to credit balance** (or similar button)
6. Add **$5** (the minimum amount)
7. Enter your payment method, confirm the payment
8. **CRITICAL: turn OFF auto-reload.** On the billing page or in a follow-up prompt, Anthropic may ask about auto-reload. **Click "Skip for now"** or set the toggle to OFF. You don't want your card to be charged automatically. You can always add more manually later.

**Stay on the Anthropic Console after paying. Do not close the tab.**

**Part B — create your API key (still on the same console)**

1. In the same Console, click **Settings** in the left sidebar
2. Click **API Keys**
3. Click **Create Key** (top-right of the page)
4. Give the key a name like `n8n-invoice-tracker`
5. Click **Create Key**
6. A popup shows your key — a long string starting with `sk-ant-api03-...`
7. Click the **Copy** button next to the key
8. **Paste it immediately into a text file (Notepad).** You will NEVER see this key again after you close the popup. If you lose it, you'll have to create a new one.
9. Save the text file somewhere safe on your computer.
10. **Security warning:** treat this key like a password. Never share it, never post it publicly, never commit it to GitHub. Anyone with this key can make API calls that charge your account.

Now you can close the popup. Your key is created and saved.

### Step 4: Sign up for n8n and create your Gmail + Google Sheets credentials

**Part A — sign up for n8n Cloud:**

1. Go to [n8n.io](https://n8n.io) and click **Get started for free** (top-right)
2. Sign up (email or Google login). If new: check your email for the verification code n8n sends, enter it in the browser
3. Answer or skip the onboarding questions. You land on n8n Cloud's home page

**Part B — create the Gmail credential:**

1. In n8n's left sidebar, click **Credentials**
2. Click **Add credential** (top-right)
3. Search `Gmail` > click **Gmail OAuth2 API**
4. Click **Sign in with Google**, sign in with the account whose Gmail you want to monitor, grant permissions
5. Back in n8n, name it `Gmail OAuth2` and click **Save**

**Part C — create the Google Sheets credential:**

1. Back in Credentials, click **Add credential** again
2. Search `Google Sheets` > click **Google Sheets OAuth2 API**
3. Repeat the Sign-in-with-Google flow using the SAME Google account
4. Name it `Google Sheets OAuth2` and click **Save**

You now have both credentials saved. They'll appear as dropdown options in the next step.

### Step 5: Import the workflow and configure the nodes

**Part A — download the workflow file:**

1. Open the workflow file in GitHub: click [AI_Invoice_Tracker_workflow.json](https://github.com/Diklabejerano/ai-invoice-tracker/blob/main/AI_Invoice_Tracker_workflow.json)
2. On the file view page, click the **Download raw file** button (top-right, icon looks like a download arrow)
3. The file downloads to your Downloads folder

**Part B — import into n8n:**

1. In n8n, click **Workflows** > **Add workflow** — a new empty workflow canvas opens
2. In the top-right of the canvas, click the three-dot menu (`⋯`) or hamburger icon > **Import from File**
3. Select `AI_Invoice_Tracker_workflow.json` from Downloads
4. The full workflow appears with all 9 nodes

Fallback if "Import from File" isn't visible: open the downloaded `.json` file with Notepad, select all (Ctrl+A), copy (Ctrl+C), then paste onto the empty canvas in n8n.

**Part C — configure each node:**

Click each node in the workflow and update the fields as listed below. Save (Ctrl+S) as you go.

| Node | What to change |
|---|---|
| **Gmail Trigger** | Credential: pick `Gmail OAuth2` from the dropdown. Then in Filters > Label, pick the label you created in Step 2. |
| **Get a message** | Credential: `Gmail OAuth2` |
| **HTTP Request** | Scroll to Send Headers. Find the row with Name `x-api-key`. In the Value field, replace `YOUR_ANTHROPIC_API_KEY_HERE` with your actual key from Step 3. |
| **Append row in sheet** (main) | Credential: `Google Sheets OAuth2`. Document: pick your Google Sheet copy from Step 1. Sheet Name: pick `Sheet1` (main tab). **Then scroll to Columns section, set "Mapping Column Mode" to "Map Each Column Automatically".** |
| **Append row in sheet1** (Error tab) | Credential: `Google Sheets OAuth2`. Document: same Google Sheet. Sheet Name: pick `Error`. **Then scroll to Columns section, keep "Define Below" mapping mode and confirm the 4 columns are mapped: timestamp → `{{ $now }}`, sender_email → `{{ $('Get a message').item.json.from }}`, error_message → `{{ JSON.stringify($json.error) }}`, raw_response → `{{ JSON.stringify($json) }}`.** |
| **Send a message** | Credential: `Gmail OAuth2`. In the sendTo field, replace `YOUR_NOTIFICATION_EMAIL_HERE` with your email address (where you want confirmations sent). |

### Step 6: Activate the workflow and test

1. At the top-right of the workflow canvas, toggle **Active** to ON
2. Send yourself a test email: attach any PDF invoice, apply the Gmail label from Step 2
3. Wait ~1 minute for the Gmail Trigger to poll for new emails
4. Check your Google Sheet — a new row should appear in `Sheet1` with the extracted fields
5. Check your inbox — a confirmation email should arrive within a few seconds after the sheet row appears

If it worked, you're done. The workflow will now run automatically on every labeled invoice.

## Test with sample invoices

`test_invoices_anonymized/` contains 6 synthetic invoices (3 English + 3 Hebrew, mixed types and currencies). All names and amounts are fictional. Copy any file's text into a Word doc, export as PDF, email to yourself with the label.

## Troubleshooting

- **HTTP Request returns 401.** API key wrong or no credits. Check the key you pasted matches your Anthropic Console key exactly, and check Settings > Billing shows credits.
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
