# 🛡️ Custom Phish Reporter

A lightweight Outlook add-in that lets users report suspicious emails to their security team with one click — no authentication required.

**Status:** Proof of Concept / Learning Project

**Live page:** [https://mimecast-scott.github.io/custom-phish-reporter/](https://mimecast-scott.github.io/custom-phish-reporter/)

---

## How It Works

1. User opens a suspicious email in Outlook
2. Clicks the **Report Phish** button in the ribbon
3. A taskpane opens showing the email details (sender, subject, date)
4. User confirms the report
5. The add-in packages the full `.eml` file + reporter identity and POSTs it to your API
6. User sees a confirmation message

No login screen. The reporter's email address is pulled from `Office.context.mailbox.userProfile.emailAddress`, which Outlook provides automatically.

---

## What Gets Submitted

The report is sent as a JSON payload:

```json
{
  "reporter": "jane.doe@company.com",
  "reporterName": "Jane Doe",
  "messageId": "<abc123@mail.company.com>",
  "subject": "Urgent: Verify your account",
  "sender": "suspicious@phishing-domain.com",
  "receivedAt": "2026-03-18T09:15:00.000Z",
  "emlBase64": "base64-encoded-full-eml-file..."
}
```

---

## Repository Structure

```
custom-phish-reporter/
├── manifest.xml          ← sideload this into Outlook
├── functions.html        ← Office.js function shim
├── index.html            ← project landing page
├── README.md             ← you are here
├── taskpane/
│   └── report.html       ← the reporting UI
└── assets/
    └── icon-*.png        ← add-in icons (see below)
```

---

## Platform Support

| Platform         | Status    |
|------------------|-----------|
| Outlook Desktop  | ✅ Supported |
| Outlook Web (OWA)| ✅ Supported |
| Outlook Mobile   | ✅ Supported |
| Mailbox API      | Requires v1.8+ |

---

## Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/mimecast-scott/custom-phish-reporter.git
cd custom-phish-reporter
```

### 2. Generate a GUID

Go to [guidgenerator.com](https://www.guidgenerator.com/) and replace the placeholder `XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX` in `manifest.xml` with your new GUID.

### 3. Add icon assets

Create an `assets/` folder and add PNG icons at these sizes (in pixels):

**Desktop:** 16, 32, 64, 80, 128

**Mobile:** 25, 48, 50, 75, 96, 144

A single square source image resized to each dimension is fine for PoC.

### 4. Set your API endpoint

In `taskpane/report.html`, update the `API_ENDPOINT` variable:

```js
const API_ENDPOINT = "https://your-server.com/api/report-phish";
```

### 5. Enable GitHub Pages

Go to **Settings → Pages → Source** and select `main` branch, `/ (root)`. Your site will be live at `https://mimecast-scott.github.io/custom-phish-reporter/`.

### 6. Sideload in Outlook

**Outlook on the web:**
1. Go to [Outlook on the web](https://outlook.office.com)
2. Open any email
3. Click **···** (more actions) → **Get Add-ins**
4. Click **My add-ins** → **Add a custom add-in** → **Add from file**
5. Upload `manifest.xml`

**Outlook desktop:**
1. Go to **File → Manage Add-ins** (opens in browser)
2. Follow the same steps as above

---

## Architecture

```
┌─────────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│                     │     │                      │     │                 │
│   Outlook Client    │────▶│   GitHub Pages       │────▶│   Your API      │
│                     │     │                      │     │                 │
│  - Loads manifest   │     │  - report.html       │     │  - Receives     │
│  - Shows ribbon btn │     │  - functions.html    │     │    JSON payload │
│  - Opens taskpane   │     │  - Office.js reads   │     │  - Stores .eml  │
│                     │     │    email + reporter   │     │  - Alerts SOC   │
└─────────────────────┘     └──────────────────────┘     └─────────────────┘
         ▲                                                        │
         │              Reporter identity from                    │
         └──────────── Office.context.mailbox.userProfile ────────┘
                        (no auth needed)
```

---

## Security Considerations

This PoC has **no authentication** on the API endpoint. Before going to production:

- **Domain validation** — Only accept reports from known email domains on the server side
- **Rate limiting** — Prevent abuse by limiting submissions per user/IP
- **Origin checking** — Validate the `Origin` header on incoming requests
- **HTTPS only** — Both GitHub Pages and your API must use HTTPS
- **Tenant lock** — Consider deploying via Microsoft 365 Admin Centre to restrict the add-in to your organisation

---

## Differences from Mimecast Add-in

| Feature             | Mimecast                      | This PoC                        |
|---------------------|-------------------------------|---------------------------------|
| Authentication      | Mimecast login required       | None (uses Office.js profile)   |
| Buttons             | Report, Managed Senders, Hold | Report only                     |
| Hosting             | Mimecast servers              | GitHub Pages                    |
| Backend             | Mimecast platform             | Your own API endpoint           |
| Scope               | Full email security suite     | Phish reporting only            |

---

## Next Steps for Production

- [ ] Set up a backend API (Azure Function, AWS Lambda, Power Automate, etc.)
- [ ] Add server-side domain validation
- [ ] Generate proper icon assets at all sizes
- [ ] Replace the placeholder GUID in `manifest.xml`
- [ ] Test on desktop, web, and mobile
- [ ] Deploy org-wide via Microsoft 365 Admin Centre
- [ ] Add feedback loop — notify reporters of investigation outcomes

---

## License

This project is for learning and proof-of-concept purposes.