# n8n Finance Bot

> Personal finance tracking bot built with **n8n + Telegram + AI Information Extractor + Google Sheets**.

A lightweight personal automation workflow that turns natural-language Telegram messages into structured financial transactions and stores them automatically in Google Sheets.

---

## ✨ Features

- Record **income** and **expenses** from natural-language messages.
- Extract transaction data automatically with an AI model.
- Normalize transaction amount into a numeric value.
- Classify transactions into predefined categories.
- Save transactions to Google Sheets.
- Automatically timestamp transactions using **Asia/Makassar (UTC+8)**.
- Send a confirmation message back to Telegram.
- Restrict the Telegram Trigger to an authorized chat ID.

---

## 🏗️ Architecture

```text
┌──────────────┐
│   Telegram   │
│    User      │
└──────┬───────┘
       │
       ▼
┌──────────────────────┐
│  Telegram Trigger    │
└──────────┬───────────┘
           │
           ▼
┌────────────────────────────┐
│   Information Extractor    │◄──── Groq Chat Model
│                            │
│  tipe                     │
│  jumlah                   │
│  deskripsi                │
│  kategori                 │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│      Google Sheets         │
│       Append Row           │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│      Telegram Reply        │
└────────────────────────────┘
```

The workflow follows:

```text
Natural Language
      ↓
AI Extraction
      ↓
Structured Transaction
      ↓
Google Sheets
      ↓
Confirmation
```

---

## 🔄 Workflow

The final n8n workflow contains four main steps:

```text
Telegram Trigger
      ↓
Information Extractor
      ↓
Append Row in Sheet
      ↓
Send a text message
```

The AI model is connected to the **Information Extractor** as its language model sub-node.

---

## 🧠 AI Extraction

The Information Extractor reads:

```text
{{ $json.message.text }}
```

and produces four required attributes:

| Field | Type | Description |
|---|---|---|
| `tipe` | String | `pemasukan` or `pengeluaran` |
| `jumlah` | Number | Transaction amount as a number |
| `deskripsi` | String | Short transaction description |
| `kategori` | String | Transaction category |

### Example

Input:

```text
beli nasi 20 ribu
```

Expected structured output:

```json
{
  "tipe": "pengeluaran",
  "jumlah": 20000,
  "deskripsi": "Beli nasi",
  "kategori": "Makanan"
}
```

Another example:

```text
gaji bulan ini 5 juta
```

```json
{
  "tipe": "pemasukan",
  "jumlah": 5000000,
  "deskripsi": "Gaji bulan ini",
  "kategori": "Gaji"
}
```

---

## 🏷️ Transaction Categories

The current workflow defines these categories:

```text
Makanan
Transport
Belanja
Tagihan
Hiburan
Gaji
Bonus
Lainnya
```

Transaction types:

```text
pemasukan
pengeluaran
```

The extractor should stay within these values.

---

## 🤖 AI Model

The final workflow JSON currently connects the Information Extractor to a **Groq Chat Model** configured with:

```text
openai/gpt-oss-20b
```

> **Important:** The repository documentation describes the model configured in the final workflow JSON. If the workflow is migrated to another model, update this section together with the workflow documentation.

---

## 📊 Google Sheets

Transactions are appended to a Google Sheets spreadsheet.

The final workflow uses these columns:

| Column | Purpose |
|---|---|
| `Tanggal` | Transaction timestamp |
| `Tipe` | Income / expense |
| `Jumlah` | Transaction amount |
| `Deskripsi` | Transaction description |
| `Kategori` | Transaction category |
| `Saldo` | Existing spreadsheet field |

The workflow generates the timestamp with:

```javascript
$now.setZone('Asia/Makassar').toFormat('yyyy-MM-dd HH:mm')
```

Therefore the recorded timestamp uses:

```text
Asia/Makassar
UTC+8
```

### Recommended sheet structure

```text
| Tanggal | Tipe | Jumlah | Deskripsi | Kategori | Saldo |
```

The `Saldo` column can be handled by a spreadsheet formula if balance calculation is implemented in the sheet.

---

## ⚠️ Current Workflow Mapping Note

The final workflow JSON currently contains this mapping for the `Kategori` column:

```javascript
={{ $json.output.deskripsi }}
```

while the extractor provides:

```javascript
={{ $json.output.kategori }}
```

This means the current implementation should be reviewed before publishing the workflow as a clean portfolio version.

The intended mapping is:

```javascript
={{ $json.output.kategori }}
```

This README intentionally documents the current workflow behavior rather than silently modifying it.

---

## 💬 Example Usage

Send a normal Telegram message.

### Expense

```text
beli kopi 25rb
```

### Income

```text
gaji bulan ini 5jt
```

### Natural language

```text
tadi saya keluar 35 ribu buat makan siang
```

The workflow converts the message into structured data and appends it to Google Sheets.

---

## 📨 Telegram Confirmation

After the Google Sheets operation, the bot sends a confirmation similar to:

```text
✅ Tercatat!

Tipe       : pengeluaran
Jumlah     : Rp25.000
Deskripsi  : Beli kopi
Kategori   : Makanan

🔗 Buka Google Sheet Catatan Keuangan
```

The reply is sent to the same Telegram chat that submitted the transaction.

---

## 🛠️ Tech Stack

| Technology | Role |
|---|---|
| [n8n](https://n8n.io/) | Workflow automation |
| Telegram Bot | User interface |
| Groq | AI model provider |
| AI Information Extractor | Structured extraction |
| Google Sheets | Transaction database |
| Docker | n8n runtime |
| ngrok | HTTPS tunnel during local development |

---

## 📁 Repository Structure

```text
n8n-finance-bot/
│
├── README.md
│
├── workflow/
│   └── catatan-keuangan.json
│
├── docs/
│   └── TUTORIAL.md
│
├── prompts/
│   └── information-extractor.md
│
├── screenshots/
│   └── ...
│
└── .gitignore
```

---

## 🚀 Setup

### Prerequisites

You need:

- n8n
- Telegram Bot
- Telegram Bot Token
- Groq API credential
- Google account
- Google Sheets
- Google Sheets credential/service account
- HTTPS webhook URL when running n8n locally

---

### 1. Create Telegram Bot

Create a Telegram bot using BotFather and obtain its Bot Token.

Configure the Telegram credential in n8n.

For a private personal bot, configure the Telegram Trigger to accept only the intended chat ID.

---

### 2. Prepare Google Sheets

Create a spreadsheet with:

```text
Tanggal
Tipe
Jumlah
Deskripsi
Kategori
Saldo
```

Connect the spreadsheet to n8n using the appropriate Google credential.

---

### 3. Configure AI Credential

Configure the Groq credential in n8n.

The final workflow currently uses:

```text
openai/gpt-oss-20b
```

through the Groq Chat Model node.

---

### 4. Import the Workflow

Import:

```text
workflow/catatan-keuangan.json
```

into n8n.

After importing, review:

- Telegram credential
- Groq credential
- Google Sheets credential
- Spreadsheet
- Sheet
- Chat ID restriction
- AI model
- column mapping

Do not assume credentials from the exported workflow will work in another n8n instance.

---

### 5. Test the Workflow

Use a simple test:

```text
beli nasi 20 ribu
```

Verify:

```text
Telegram Trigger
       ↓
Information Extractor
       ↓
Google Sheets
       ↓
Telegram Reply
```

Check all three outputs:

1. AI structured output
2. New Google Sheets row
3. Telegram confirmation

---

## 🧪 Test Cases

### Test 1 — Simple expense

```text
beli nasi 20 ribu
```

Expected:

```text
tipe = pengeluaran
jumlah = 20000
kategori = Makanan
```

### Test 2 — Income

```text
dapat bonus 500 ribu
```

Expected:

```text
tipe = pemasukan
jumlah = 500000
kategori = Bonus
```

### Test 3 — Natural language

```text
tadi saya keluar 35 ribu buat makan siang
```

The bot should extract the amount and classify the transaction appropriately.

---

## 🔧 Troubleshooting

### AI does not produce output

Check:

- Telegram input field
- `message.text`
- Information Extractor schema
- Groq credential
- Groq Chat Model connection
- model availability

---

### Google Sheets does not receive a row

Check:

- Google credential
- Spreadsheet ID
- Sheet name
- column headers
- Information Extractor output
- Google Sheets mapping

---

### Category is wrong

First inspect:

```text
Information Extractor → output → kategori
```

Then inspect the Google Sheets mapping.

Make sure the `Kategori` column uses:

```javascript
={{ $json.output.kategori }}
```

rather than:

```javascript
={{ $json.output.deskripsi }}
```

---

### Telegram does not reply

Check:

- Telegram credential
- Telegram chat ID
- execution status
- Google Sheets node result
- Telegram Send Message node

---

## 🔐 Security

Never commit real credentials to this repository.

Do not publish:

```text
Telegram Bot Token
Groq API Key
Google OAuth secrets
Service Account private keys
n8n credentials
Personal access tokens
Private spreadsheet credentials
```

The exported workflow should be sanitized before being committed to a public repository.

Use placeholders such as:

```text
YOUR_TELEGRAM_CREDENTIAL
YOUR_GROQ_CREDENTIAL
YOUR_GOOGLE_CREDENTIAL
YOUR_SPREADSHEET_ID
```

If a secret has already been exposed:

```text
REVOKE
   ↓
ROTATE
   ↓
UPDATE CREDENTIAL
   ↓
TEST
```

---

## 🧩 Design Principles

This project demonstrates a simple but reusable automation pattern:

```text
INPUT
  ↓
NORMALIZE
  ↓
UNDERSTAND
  ↓
STRUCTURE
  ↓
STORE
  ↓
RESPOND
```

The AI is responsible for understanding natural language.

n8n is responsible for orchestration.

Google Sheets is responsible for structured storage.

Telegram is responsible for the user interface.

---

## 🔮 Future Improvements

Potential improvements include:

- stricter category validation
- duplicate transaction detection
- transaction editing/deletion
- monthly summaries
- spending analysis
- budget alerts
- dashboard integration
- automated financial reports
- improved error handling
- validation before writing to Google Sheets
- separate production and development workflows

---

## 📚 Documentation

Detailed implementation instructions are available in:

```text
docs/TUTORIAL.md
```

The tutorial covers:

- n8n setup
- Telegram configuration
- AI Information Extractor
- Google Sheets configuration
- credential setup
- workflow testing
- troubleshooting
- maintenance

---

## 📄 License

This project is provided as a personal automation project and learning/portfolio reference.

Adapt the workflow, credentials, spreadsheet structure, and AI configuration to your own environment before deployment.
