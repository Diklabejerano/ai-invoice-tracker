# Test Invoices — Anonymized

These are synthetic invoices for testing the AI Invoice Tracker. All names, tax IDs, amounts, and details are 100% fictional. Do not treat any of them as real business records.

## What each file exercises

| File | Language | Type | Notes |
|---|---|---|---|
| 01_english_saas_subscription.txt | English | SaaS monthly subscription | Small amount, no tax |
| 02_hebrew_consulting.txt | Hebrew | Consulting services | With 17% Israeli VAT |
| 03_english_multiple_line_items.txt | English | Design services | Multiple line items, GBP currency, 20% UK VAT |
| 04_hebrew_subscription.txt | Hebrew | Subscription (חשבונית מס קבלה) | Combined tax invoice + receipt format |
| 05_english_freelance_hourly.txt | English | Freelance dev work | EUR, hourly rate, German VAT |
| 06_hebrew_utility_bill.txt | Hebrew | Utility (internet) bill | Israeli utility format |

## How to test

Option 1 — Copy the text into a PDF (any word processor → export as PDF), then email the PDF to your configured Gmail label. Workflow will fire and process it.

Option 2 — Adapt the workflow to accept plain text input for testing without PDF conversion.
