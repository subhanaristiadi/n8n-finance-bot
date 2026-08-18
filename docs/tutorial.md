---
title: "Complete Tutorial — Telegram Personal Finance Bot v2.0 (Bynara Edition)"
version: "2.0"
language: "English"
project: "n8n Finance Bot"
---

# Complete Tutorial --- Telegram Personal Finance Bot v2.0 (Bynara Edition)

> Build a personal Telegram bot that automatically records income and
> expenses using AI, then stores the structured data in Google Sheets
> using **Bynara AI Router**, **n8n OpenAI Chat Model**, and **Mistral
> Large**.

This English edition is a rewritten version of the original tutorial,
with the same workflow architecture and configuration logic.

------------------------------------------------------------------------

# 1. About This Tutorial

This tutorial explains how to build a personal finance tracker that
works through Telegram.

You simply send a natural-language message such as:

``` text
bought coffee for 25k
```

or:

``` text
salary this month 7 million
```

The bot uses AI to understand the transaction, converts it into
structured data, stores it in Google Sheets, and sends a confirmation
back to Telegram.

The complete flow is:

``` text
Telegram
   ↓
Telegram Trigger
   ↓
Information Extractor
   ↓
OpenAI Chat Model
   ↓
Bynara AI Router
   ↓
mistral-large
   ↓
Google Sheets
   ↓
Telegram Reply
```

------------------------------------------------------------------------

# 2. Version 2.0 Architecture

Version 1.0 used Groq and `qwen/qwen3-32b`.

Version 2.0 uses:

-   Bynara AI Router
-   n8n OpenAI Chat Model
-   `mistral-large`
-   Google Sheets
-   Telegram
-   Docker Desktop
-   ngrok

The workflow structure itself remains simple:

``` text
Telegram Trigger
      ↓
Information Extractor
      ↓
Google Sheets
      ↓
Telegram Reply
```

The OpenAI Chat Model is used as the AI language model for the
Information Extractor. It is configured to communicate with Bynara
rather than directly with OpenAI.

------------------------------------------------------------------------

# 3. What the Bot Can Do

## Record income

Example:

``` text
salary this month 7 million
```

Expected result:

``` text
Type     : income
Amount   : 7000000
Category : Salary
```

## Record expenses

Example:

``` text
bought gasoline for 150k
```

Expected result:

``` text
Type     : expense
Amount   : 150000
Category : Transport
```

## Understand common Indonesian number formats

The AI can interpret:

``` text
25rb      → 25000
450k      → 450000
2 juta    → 2000000
1.5jt     → 1500000
750 ribu  → 750000
```

## Automatically classify transactions

Examples:

``` text
coffee
    ↓
Food

Grab
    ↓
Transport

internet bill
    ↓
Bills

salary
    ↓
Salary

annual bonus
    ↓
Bonus

buy shoes
    ↓
Shopping
```

------------------------------------------------------------------------

# 4. Example Google Sheets Data

The spreadsheet uses these columns:

  Date             |  Type    |     Amount  | Description   |     Category
  2026-07-04 08:15 |  Expense |      25000 | Buy coffee |         Food
  2026-07-04 12:30 |  Expense |     150000 | Fill up gasoline |    Transport
  2026-07-05 08:00 | Income |     7000000 | July salary  |      Salary

The original workflow maps these five fields directly into Google
Sheets.

------------------------------------------------------------------------

# 5. Prerequisites

## Hardware

-   Windows 10 or Windows 11
-   At least 8 GB RAM
-   Stable internet connection

## Software

-   Docker Desktop
-   Google Chrome
-   Telegram Desktop or Telegram mobile
-   PowerShell

## Accounts

You need:

-   Google account
-   Telegram account
-   Bynara account

------------------------------------------------------------------------

# 6. Credential Checklist

Prepare the following credentials:

  Credential                   Purpose
  ---------------------------- -----------------
  Telegram Bot Token           Telegram Bot
  Telegram Chat ID             Restrict access
  Bynara API Key               AI access
  Google OAuth Client ID       Google Sheets
  Google OAuth Client Secret   Google Sheets
  Google Sheet ID              Database
  n8n username                 n8n login
  n8n password                 n8n login

Never publish API keys, bot tokens, OAuth secrets, or personal Chat IDs
in a public GitHub repository.

------------------------------------------------------------------------

# 7. Create the Telegram Bot

Telegram bots are created through **@BotFather**.

## Step 1

Open Telegram and search for:

``` text
@BotFather
```

Click **Start**.

## Step 2

Send:

``` text
/newbot
```

Enter a bot name.

Example:

``` text
Personal Finance Bot
```

## Step 3

Choose a unique username ending in `bot`.

Example:

``` text
personal_finance_tracker_bot
```

## Step 4

BotFather will provide a Bot Token.

Example:

``` text
7812345678:AAHxxxxxxxxxxxxxxxx
```

Store the token securely.

------------------------------------------------------------------------

# 8. Get Your Telegram Chat ID

Search for:

``` text
@userinfobot
```

Click **Start**.

The bot will show your Telegram ID.

Example:

``` text
Id
1204164085
```

Use this value in the Telegram Trigger's chat restriction settings.

The reference workflow restricts the Telegram Trigger to a specific Chat
ID so that the personal bot is not openly available to everyone.

------------------------------------------------------------------------

# 9. Create a Bynara API Key

Bynara is the AI provider used in this version.

The workflow accesses Bynara through n8n's OpenAI Chat Model because
Bynara provides an OpenAI-compatible API endpoint.

## Step 1

Open the Bynara website and sign in.

``` text
https://bynara.id
```

## Step 2

Open:

``` text
API Keys
```

Select:

``` text
Create API Key
```

## Step 3

Give the key a descriptive name.

Example:

``` text
Personal Finance Bot
```

## Step 4

Create the key and store it securely.

Example:

``` text
bnr_xxxxxxxxxxxxxxxxx
```

------------------------------------------------------------------------

# 10. Bynara Base URL

Use exactly:

``` text
https://router.bynara.id/v1
```

Do **not** add:

``` text
/chat/completions
```

or:

``` text
/responses
```

The n8n OpenAI Chat Model adds the required endpoint automatically.

------------------------------------------------------------------------

# 11. Create a Google Cloud Project

Google Cloud is required for Google Sheets access.

Open:

``` text
https://console.cloud.google.com
```

Create a new project.

Example:

``` text
n8n-finance
```

------------------------------------------------------------------------

# 12. Enable Google APIs

Inside the Google Cloud project, enable:

``` text
Google Sheets API
```

and:

``` text
Google Drive API
```

Google Drive API is required because Google Sheets files are stored in
Google Drive.

------------------------------------------------------------------------

# 13. Configure OAuth Consent Screen

Open the Google authentication settings.

Choose:

``` text
External
```

Create the application.

Provide:

-   App name
-   Support email
-   Developer contact email

For testing, add the Google account you will use with n8n as a **Test
User**.

------------------------------------------------------------------------

# 14. Create the OAuth Client

Go to:

``` text
Credentials
→
Create Credentials
→
OAuth Client ID
```

Choose:

``` text
Web Application
```

For the initial local configuration, use:

``` text
http://localhost:5678/rest/oauth2-credential/callback
```

The redirect URI will later be changed to the public HTTPS n8n URL.

Save the:

-   Client ID
-   Client Secret

------------------------------------------------------------------------

# 15. Create the Google Sheet

Open Google Sheets and create a blank spreadsheet.

Name it:

``` text
Catatan Keuangan
```

Create these headers in row 1:

  A      B      C        D             E
  ------ ------ -------- ------------- ----------
  Date   Type   Amount   Description   Category

Keep the header names consistent with the n8n mapping. Changing a header
without updating the Google Sheets node can break the workflow.

------------------------------------------------------------------------

# 16. Find the Google Sheet ID

Look at the spreadsheet URL:

``` text
https://docs.google.com/spreadsheets/d/GOOGLE_SHEET_ID/edit
```

Copy the value between `/d/` and `/edit`.

Store it securely.

------------------------------------------------------------------------

# 17. Install Docker Desktop

Download Docker Desktop for Windows.

After installation:

1.  Restart Windows if requested.
2.  Open Docker Desktop.
3.  Wait until Docker Engine is running.

Verify from PowerShell:

``` powershell
docker --version
```

Then:

``` powershell
docker ps
```

------------------------------------------------------------------------

# 18. Install ngrok

ngrok provides a public HTTPS tunnel to your local n8n instance.

Download ngrok and place it somewhere such as:

``` text
C:\ngrok\
```

You should have:

``` text
C:\ngrok\ngrok.exe
```

Add your ngrok authentication token:

``` powershell
cd C:\ngrok
.\ngrok.exe config add-authtoken YOUR_TOKEN
```

A successful configuration will report that the authtoken was saved.

------------------------------------------------------------------------

# 19. Run n8n with Docker

Start n8n:

``` powershell
docker run -d ^
--name n8n ^
-p 5678:5678 ^
-v n8n_data:/home/node/.n8n ^
n8nio/n8n
```

Check the container:

``` powershell
docker ps
```

Check the logs:

``` powershell
docker logs n8n
```

When n8n is ready, open:

``` text
http://localhost:5678
```

------------------------------------------------------------------------

# 20. Start ngrok

Open another PowerShell window:

``` powershell
cd C:\ngrok
.\ngrok.exe http 5678
```

You should see a forwarding address similar to:

``` text
https://abc123.ngrok-free.app
```

Keep this terminal open while using the bot.

------------------------------------------------------------------------

# 21. Configure WEBHOOK_URL

Stop the existing container:

``` powershell
docker stop n8n
docker rm n8n
```

Start it again using your ngrok URL:

``` powershell
docker run -d ^
--name n8n ^
-p 5678:5678 ^
-v n8n_data:/home/node/.n8n ^
-e WEBHOOK_URL=https://YOUR_NGROK_URL ^
n8nio/n8n
```

Example:

``` powershell
docker run -d ^
--name n8n ^
-p 5678:5678 ^
-v n8n_data:/home/node/.n8n ^
-e WEBHOOK_URL=https://abc123.ngrok-free.app ^
n8nio/n8n
```

Check:

``` powershell
docker logs n8n
```

------------------------------------------------------------------------

# 22. Update the Google OAuth Redirect URI

Return to Google Cloud.

Open the OAuth Client and replace the local redirect URI with:

``` text
https://YOUR_NGROK_URL/rest/oauth2-credential/callback
```

Example:

``` text
https://abc123.ngrok-free.app/rest/oauth2-credential/callback
```

The URL must:

-   use HTTPS
-   match your current ngrok URL
-   contain `/rest/oauth2-credential/callback`
-   not have an unnecessary trailing slash

------------------------------------------------------------------------

# 23. Create the n8n Account

Open:

``` text
https://YOUR_NGROK_URL
```

Create your n8n owner account.

Use a strong password.

------------------------------------------------------------------------

# 24. Import the Workflow

Create a new workflow or import the existing JSON workflow.

Recommended project structure:

``` text
Bot Catatan Keuangan
│
├── Workflow
│   └── Catatan Keuangan Aris (Bynara).json
│
├── Tutorial
│   └── tutorial-en.md
│
├── Backup
│
└── Screenshot
```

------------------------------------------------------------------------

# 25. Workflow Structure

The final workflow contains four main nodes:

``` text
Telegram Trigger
        ↓
Information Extractor
        ↓
Append Row Google Sheets
        ↓
Telegram Reply
```

The Information Extractor uses:

``` text
OpenAI Chat Model
        ↓
Bynara AI Router
        ↓
mistral-large
```

This architecture is reflected in the final finance workflow
documentation.

------------------------------------------------------------------------

# 26. Configure Telegram Trigger

Add:

``` text
Telegram Trigger
```

Configure the Telegram credential using your Bot Token.

Use:

``` text
Updates:
message
```

Restrict the trigger to your Telegram Chat ID.

The incoming text is available at:

``` javascript
={{ $json.message.text }}
```

------------------------------------------------------------------------

# 27. Configure Information Extractor

Add:

``` text
Information Extractor
```

Connect:

``` text
Telegram Trigger
        ↓
Information Extractor
```

For the **Text** field use:

``` javascript
={{ $json.message.text }}
```

This passes the Telegram message to the AI extractor.

------------------------------------------------------------------------

# 28. Define the Extraction Schema

Create these four attributes.

## Attribute 1 --- type

``` text
Name: type
Type: String
Required: Yes
```

Description:

``` text
Use "income" when money is received.
Use "expense" when money is spent.
```

## Attribute 2 --- amount

``` text
Name: amount
Type: Number
Required: Yes
```

Description:

``` text
Return numbers only.
Examples: 25000, 500000, 5000000.
Convert Indonesian expressions such as 25rb, 2 juta, and 1.5jt into numbers.
```

## Attribute 3 --- description

``` text
Name: description
Type: String
Required: Yes
```

Description:

``` text
A short and clear description of the transaction.
```

## Attribute 4 --- category

``` text
Name: category
Type: String
Required: Yes
```

Description:

``` text
Choose one:
Food, Transport, Shopping, Bills, Entertainment, Salary, Bonus, Other.
```

Clear schema descriptions improve parsing consistency.

------------------------------------------------------------------------

# 29. Add the OpenAI Chat Model

Inside the Information Extractor, add:

``` text
OpenAI Chat Model
```

Create a new credential.

Even though the credential is named **OpenAI**, this credential is
configured to use Bynara.

------------------------------------------------------------------------

# 30. Configure the Bynara OpenAI Credential

Set:

``` text
API Key:
YOUR_BYNARA_API_KEY
```

Set:

``` text
Base URL:
https://router.bynara.id/v1
```

Leave Organization and Project empty unless your setup specifically
requires them.

Most importantly:

``` text
Use Responses API: OFF
```

Do not enable it.

Bynara is being accessed through the Chat Completions-compatible route.
If Responses API is enabled, n8n attempts to call `/v1/responses`, which
can produce an endpoint-not-found error.

------------------------------------------------------------------------

# 31. Select the AI Model

Choose:

``` text
mistral-large
```

The model is used for transaction extraction.

Test the model before continuing.

Example input:

``` text
salary 5 million
```

Expected result:

``` json
{
  "type": "income",
  "amount": 5000000,
  "description": "Salary",
  "category": "Salary"
}
```

Another test:

``` text
filled up gasoline for 100k
```

Expected:

``` json
{
  "type": "expense",
  "amount": 100000,
  "description": "Fill up gasoline",
  "category": "Transport"
}
```

------------------------------------------------------------------------

# 32. Add Google Sheets

Add:

``` text
Google Sheets
```

Select:

``` text
Append Row
```

Connect:

``` text
Information Extractor
        ↓
Google Sheets
```

Create or select your Google Sheets OAuth credential.

Sign in using the Google account that owns or can edit the spreadsheet.

------------------------------------------------------------------------

# 33. Select the Spreadsheet

Choose your finance spreadsheet.

Example:

``` text
Catatan Keuangan
```

Select the appropriate sheet.

The mapping should correspond to:

``` text
Date
Type
Amount
Description
Category
```

The reference workflow uses a timestamp expression for the Date field
and maps the four AI fields into the remaining columns.

------------------------------------------------------------------------

# 34. Date and Time Expression

For the Date field, the reference workflow uses:

``` javascript
={{ $now.setZone('Asia/Makassar').toFormat('yyyy-MM-dd HH:mm') }}
```

This records the current date and time using the configured timezone.

If your deployment requires a different timezone, adjust the timezone
accordingly.

------------------------------------------------------------------------

# 35. Add Telegram Reply

Add:

``` text
Telegram
```

Use the Telegram credential.

Set the Chat ID with:

``` javascript
={{ $('Telegram Trigger').item.json.message.chat.id }}
```

A useful response format is:

``` text
✅ Recorded!

Type       : Expense
Amount     : Rp25,000
Description: Buy coffee
Category   : Food
```

You can also add a link to your Google Sheet.

Example:

``` text
🔗 Open Finance Spreadsheet
https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/edit
```

The reference workflow uses the Telegram chat ID from the original
trigger and formats the amount with the Indonesian locale.

------------------------------------------------------------------------

# 36. Complete Workflow

The final workflow should look like:

``` text
Telegram
   ↓
Telegram Trigger
   ↓
Information Extractor
   ↓
Google Sheets
   ↓
Telegram Reply
```

AI connection:

``` text
OpenAI Chat Model
        ↓
Information Extractor

OpenAI Chat Model
        ↓
Bynara AI Router
        ↓
mistral-large
```

------------------------------------------------------------------------

# 37. End-to-End Testing

Run the workflow in test mode and send messages through Telegram.

## Test 1 --- Coffee

``` text
bought coffee for 25k
```

Expected:

``` text
Type: expense
Amount: 25000
Category: Food
```

## Test 2 --- Salary

``` text
salary 5 million
```

Expected:

``` text
Type: income
Amount: 5000000
Category: Salary
```

## Test 3 --- Gasoline

``` text
filled up gasoline for 100k
```

Expected:

``` text
Category: Transport
```

## Test 4 --- Electricity

``` text
paid electricity bill 350k
```

Expected:

``` text
Category: Bills
```

## Test 5 --- Pizza

``` text
pizza hut 150k
```

Expected:

``` text
Category: Food
```

The original tutorial uses these same end-to-end test cases to validate
the parser, Google Sheets insertion, and Telegram response.

------------------------------------------------------------------------

# 38. Production Checklist

Before activating the workflow:

-   [ ] Telegram Bot works
-   [ ] Telegram Trigger receives messages
-   [ ] Chat ID restriction is correct
-   [ ] Information Extractor returns structured JSON
-   [ ] OpenAI Chat Model is connected
-   [ ] Bynara API key is valid
-   [ ] Base URL is `https://router.bynara.id/v1`
-   [ ] Model is `mistral-large`
-   [ ] Use Responses API is OFF
-   [ ] Google Sheets credential works
-   [ ] Google Sheets receives rows
-   [ ] Telegram sends confirmations
-   [ ] Workflow is Active
-   [ ] Workflow JSON has been backed up

------------------------------------------------------------------------

# 39. Activate the Workflow

Once all tests pass, switch the workflow to:

``` text
Active
```

The bot will now process Telegram messages automatically.

------------------------------------------------------------------------

# 40. Backup the Workflow

Export the workflow regularly.

Recommended backup structure:

``` text
Backup
├── Catatan Keuangan Aris (Bynara).json
├── Catatan Keuangan Aris (Bynara)-v2.json
└── Catatan Keuangan Aris (Bynara)-v3.json
```

Keep backups outside the n8n container as well.

Do not store credentials inside public workflow files.

------------------------------------------------------------------------

# 41. Troubleshooting

## Problem: Google Sheets does not receive data

Check:

-   Spreadsheet ID
-   Google OAuth credential
-   Spreadsheet headers
-   Column mapping
-   Google Sheets permissions

------------------------------------------------------------------------

## Problem: Telegram does not reply

Check:

-   Telegram Bot Token
-   Chat ID
-   Telegram credential
-   Workflow status
-   Telegram Trigger webhook

------------------------------------------------------------------------

## Problem: Information Extractor fails

Check:

-   Schema attributes
-   Attribute descriptions
-   AI credential
-   Bynara API key
-   Model name

------------------------------------------------------------------------

## Problem: Endpoint not found

Error:

``` text
The resource you are requesting could not be found.

The requested endpoint does not exist.
```

Likely cause:

``` text
Use Responses API = ON
```

Solution:

``` text
Use Responses API = OFF
```

Then execute the node again.

------------------------------------------------------------------------

## Problem: 401 Unauthorized

Likely cause:

``` text
Invalid Bynara API key
```

Solution:

-   verify the API key
-   recreate the key if necessary
-   update the n8n credential

------------------------------------------------------------------------

## Problem: 404 Not Found

Check the Base URL.

Correct:

``` text
https://router.bynara.id/v1
```

Incorrect:

``` text
https://router.bynara.id/v1/chat/completions
```

Incorrect:

``` text
https://router.bynara.id/v1/responses
```

------------------------------------------------------------------------

## Problem: Model Not Found

Verify that the selected model is available in your Bynara account.

For this tutorial:

``` text
mistral-large
```

------------------------------------------------------------------------

# 42. Best Practices

## 1. Do not casually rename spreadsheet headers

The workflow maps fields by column name.

If you change:

``` text
Amount
```

to:

``` text
Nominal
```

update the Google Sheets mapping as well.

------------------------------------------------------------------------

## 2. Keep one spreadsheet per bot

Recommended:

``` text
Finance Bot
Daily Activity Bot
English Vocabulary Bot
```

Separate databases make backup and analysis easier.

------------------------------------------------------------------------

## 3. Keep the schema stable

Current schema:

``` text
type
amount
description
category
```

If you rename:

``` text
amount
```

to:

``` text
nominal
```

you must also update the downstream mapping.

------------------------------------------------------------------------

## 4. Use clear schema descriptions

For example:

``` text
Return numbers only.
Examples: 25000, 500000, 5000000.
```

is more useful than:

``` text
Transaction amount
```

Clear descriptions improve model consistency.

------------------------------------------------------------------------

## 5. Prefer structured extraction

Use the Information Extractor instead of asking the AI to return
free-form text.

The desired structure is:

``` json
{
  "type": "expense",
  "amount": 25000,
  "description": "Buy coffee",
  "category": "Food"
}
```

------------------------------------------------------------------------

## 6. Keep the AI provider configurable

Using the n8n OpenAI Chat Model with an OpenAI-compatible provider makes
it easier to change models or providers later without redesigning the
entire workflow.

------------------------------------------------------------------------

# 43. Future Improvements

The basic bot can be extended with additional functionality.

## Delete a transaction

Example:

``` text
delete the coffee expense
```

Possible workflow operations:

``` text
Search Rows
    ↓
Update Row
or
Delete Row
```

## Daily summary

A Schedule Trigger can send a nightly summary such as:

``` text
Today's Expenses

Rp250,000

5 transactions

Top category:
Food
```

## Monthly summary

A scheduled workflow can send:

-   total income
-   total expenses
-   balance
-   category breakdown
-   charts
-   AI-generated insights

These extensions are also discussed in the original tutorial.

------------------------------------------------------------------------

# 44. Useful Docker Commands

Check running containers:

``` powershell
docker ps
```

Check all containers:

``` powershell
docker ps -a
```

Start n8n:

``` powershell
docker start n8n
```

Stop n8n:

``` powershell
docker stop n8n
```

Restart n8n:

``` powershell
docker restart n8n
```

Remove the container:

``` powershell
docker rm n8n
```

View logs:

``` powershell
docker logs n8n
```

Follow logs in real time:

``` powershell
docker logs -f n8n
```

------------------------------------------------------------------------

# 45. Important Note About ngrok

If you use the free ngrok setup, the public URL can change.

When the URL changes:

1.  start ngrok again
2.  copy the new HTTPS URL
3.  update `WEBHOOK_URL`
4.  update the Google OAuth redirect URI
5.  restart n8n if necessary
6.  verify the Telegram Trigger

This is especially important after restarting your computer.

------------------------------------------------------------------------

# 46. Final Architecture

``` text
                    Telegram
                       │
                       ▼
               Telegram Trigger
                       │
                       ▼
             Information Extractor
                       │
                       ▼
              Google Sheets
                       │
                       ▼
                Telegram Reply

AI Layer:

        Information Extractor
                 ▲
                 │
         OpenAI Chat Model
                 │
                 ▼
        Bynara AI Router
                 │
                 ▼
            mistral-large
```

------------------------------------------------------------------------

# 47. Conclusion

Congratulations.

You have built a personal Telegram Finance Bot using:

-   Telegram Bot
-   n8n
-   Information Extractor
-   OpenAI Chat Model
-   Bynara AI Router
-   `mistral-large`
-   Google Sheets
-   Docker
-   ngrok

The bot can:

-   receive natural-language transactions
-   classify income and expenses
-   normalize Indonesian currency expressions
-   generate structured JSON
-   save transactions to Google Sheets
-   send confirmation messages through Telegram

The architecture is intentionally modular, making it easier to replace
the AI model or provider in the future without redesigning the entire
automation.

------------------------------------------------------------------------

# Quick Reference

### Telegram

``` text
Bot Token
Chat ID
```

### Bynara

``` text
API Key
Base URL:
https://router.bynara.id/v1
```

### Model

``` text
mistral-large
```

### OpenAI Chat Model

``` text
Use Responses API:
OFF
```

### Information Extractor

``` text
type
amount
description
category
```

### Google Sheets

``` text
Date
Type
Amount
Description
Category
```

### Main workflow

``` text
Telegram Trigger
↓
Information Extractor
↓
Google Sheets
↓
Telegram Reply
```

------------------------------------------------------------------------

# End of Tutorial

**Telegram Personal Finance Bot v2.0 --- Bynara Edition**

Happy building!
