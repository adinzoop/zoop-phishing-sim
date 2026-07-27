# Zoop Phishing Simulation

Internal security awareness training simulation for Zoop employees.
**Authorized use only — run only within the Zoop organization.**

---

## Scenario

Simulated Google Workspace security alert claiming suspicious sign-in from an unknown Linux device.
Employees who click are taken to a fake Google login page. Credential submission redirects to an
awareness training page explaining the simulation.

**Goals tracked:**
- Email open rate (via tracking pixel)
- Link click-through rate
- Credential submission rate

---

## Project Structure

```
zoop-phishing-sim/
├── email-templates/
│   ├── google-workspace-alert.html   # HTML phishing email
│   └── google-workspace-alert.txt    # Plain-text fallback
├── landing-pages/
│   └── google-login.html             # Fake Google login (GoPhish credential capture)
├── awareness/
│   └── training-page.html            # Shown after credential submission
├── gophish-config/
│   └── campaign-template.json        # GoPhish campaign settings (fill in your values)
└── scripts/
    └── import-users.py               # Auto-import Google Workspace users into GoPhish
```

---

## Prerequisites

- [GoPhish](https://getgophish.com) installed and running (default: `localhost:3333`)
- Python 3.8+ with dependencies:
  ```bash
  pip install requests google-auth google-auth-httplib2 google-api-python-client
  ```
- A Google Workspace **service account** with domain-wide delegation
  (scope: `https://www.googleapis.com/auth/admin.directory.user.readonly`)
- An SMTP relay or transactional email provider (e.g. SendGrid, Mailgun)

---

## Setup

### 1. Install GoPhish

```bash
# macOS (Homebrew)
brew install gophish

# Or download from https://github.com/gophish/gophish/releases
```

Start it:
```bash
./gophish
# Admin panel: https://localhost:3333  (default creds: admin / gophish)
```

### 2. Import employees from Google Workspace

Place your service account JSON key at `scripts/service-account.json`, then:

```bash
export GOPHISH_API_KEY="paste-your-gophish-api-key-here"
export GOPHISH_URL="http://localhost:3333"
python3 scripts/import-users.py
```

This creates a GoPhish group called **"All Zoop Employees"** with all users from `zoop.one`.

### 3. Configure the campaign

Edit `gophish-config/campaign-template.json` and fill in:
- `sending_profile.host` — your SMTP server
- `sending_profile.username` / `.password` — SMTP credentials
- `landing_page.redirect_url` — URL where `awareness/training-page.html` is hosted
- `url` — public URL of your GoPhish server (employees' browsers must reach this)
- `launch_date` / `send_by_date` — when to send

### 4. Load templates into GoPhish

In the GoPhish admin UI:
1. **Email Templates** → New → paste `email-templates/google-workspace-alert.html`
2. **Landing Pages** → New → paste `landing-pages/google-login.html`
   - Enable "Capture Submitted Data" and "Capture Passwords"
   - Set redirect URL to your awareness page
3. **Users & Groups** → verify the imported group
4. **Sending Profiles** → add your SMTP config
5. **Campaigns** → New → wire everything together and launch

### 5. Monitor results

GoPhish dashboard shows real-time:
- Emails sent / opened
- Links clicked
- Credentials submitted (stored locally in GoPhish — delete after review)
- Time-to-click distribution

---

## After the Campaign

1. Export the results CSV from GoPhish before deleting the campaign.
2. Send a follow-up email to **all** employees with the awareness page link.
3. Send a targeted follow-up to employees who clicked/submitted with mandatory training.
4. Delete captured credentials from GoPhish immediately after reviewing metrics.

---

## Legal & Ethics Checklist

- [ ] Written authorization from company leadership obtained
- [ ] Simulation scoped to Zoop-owned accounts only
- [ ] IT / legal team notified before launch
- [ ] Captured credentials deleted immediately after the campaign
- [ ] Results shared only with relevant stakeholders (not used punitively)

---

## Contact

Security questions: security@zoop.one
