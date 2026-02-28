# ClawBigin — Bigin CRM Skill for OpenClaw

**Project Challenge:** Build a comprehensive Bigin CRM skill for OpenClaw — the lightweight, pipeline-focused CRM for small businesses.

---

## 🎯 Why Bigin First?

**Bigin is the perfect MVP target** for your first CRM skill:

| Advantage | Explanation |
|-----------|-------------|
| **Simpler** | Fewer modules to implement (no complex Lead→Account→Contact→Opportunity flow) |
| **Faster to MVP** | Core features: Pipelines, Contacts, Companies, Tasks |
| **Clearer workflow** | Everything revolves around pipelines — intuitive for sales teams |
| **Same API patterns** | Everything you learn transfers directly to Zoho CRM later |
| **Real-world need** | Small businesses use Bigin but lack CLI tooling |

**Once you master Bigin, extending to full Zoho CRM is trivial** — just add more modules.

---

## 🔍 Bigin vs Zoho CRM API

### API Endpoint Difference
| Product | Base URL |
|---------|----------|
| **Zoho CRM** | `https://www.zohoapis.com/crm/v2/` |
| **Bigin** | `https://www.zohoapis.com/bigin/v2/` |

**Simply swap `crm` → `bigin` in the base URL.**

### Module Mapping

| Bigin Module | API Name | Zoho CRM Equivalent | Notes |
|--------------|----------|---------------------|-------|
| **Pipelines** | `Pipelines` | `Deals` | Core module — everything flows through here |
| **Contacts** | `Contacts` | `Contacts` | Same |
| **Companies** | `Accounts` | `Accounts` | Called "Companies" in UI, "Accounts" in API |
| **Products** | `Products` | `Products` | Same |
| **Tasks** | `Tasks` | `Tasks` | Same |
| **Events** | `Events` | `Events` | Same |
| **Calls** | `Calls` | `Calls` | Same |
| **Notes** | `Notes` | `Notes` | Same |
| **Attachments** | `Attachments` | `Attachments` | Same |

**Key Difference:** Bigin has **no separate "Leads" module** — leads go directly into Pipelines.

### OAuth Scopes
```python
# Zoho CRM scopes
"ZohoCRM.modules.ALL,ZohoCRM.settings.ALL"

# Bigin scopes
"ZohoBigin.modules.ALL,ZohoBigin.settings.ALL"
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────┐
│         OpenClaw Agent                  │
│  (Your personal sales assistant)        │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│      Bigin CRM Skill (Python)           │
│  - OAuth2 authentication                │
│  - REST API v2 wrapper                  │
│  - Pipeline automation                  │
│  - Contact/Company management           │
└─────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│        Bigin CRM REST API v2            │
│  - Pipelines (core sales module)        │
│  - Contacts & Companies                 │
│  - Tasks, Events, Calls                 │
│  - Products & Notes                     │
└─────────────────────────────────────────┘
```

---

## 🛠️ Core Features

### 1. Authentication & Setup
```bash
# One-time OAuth setup
bigin auth --client-id "1000.xxx" --client-secret "xxx"
# Opens browser for Bigin/Zoho login
# Stores tokens securely in ~/.openclaw/credentials/bigin-crm.json

# Check auth status
bigin auth:whoami
```

### 2. Pipeline Management (Core Feature)
```bash
# Create a pipeline entry (like a deal/opportunity)
bigin pipeline create --contact-id 12345 --company-id 67890 \
  --stage "Initial Contact" --amount 50000 \
  --closing-date "2026-03-15" --owner "sales@yourcompany.com"

# Update pipeline stage
bigin pipeline update --id 12345678 --stage "Negotiation" \
  --amount 75000 --probability 70

# Move to next stage
bigin pipeline advance --id 12345678

# Mark as won/lost
bigin pipeline win --id 12345678
bigin pipeline lose --id 12345678 --reason "Budget constraints"

# List all pipelines
bigin pipeline list --stage "Proposal" --owner "me" --limit 50

# Search pipelines
bigin pipeline search --query "company:Acme" --stage "Open"

# Get pipeline details with history
bigin pipeline get --id 12345678 --include-history
```

### 3. Contact Management
```bash
# Create contact
bigin contact create --first-name "John" --last-name "Doe" \
  --email "john@company.com" --phone "+91-98765-43210" \
  --company "Acme Inc" --source "Website"

# Bulk import from CSV
bigin contact import --file contacts.csv --mapping mapping.json

# Search contacts
bigin contact search --query "company:Acme" --limit 100

# Update contact
bigin contact update --id 87654321 --phone "+91-99999-88888"

# Get contact with associated pipelines
bigin contact get --id 87654321 --include-pipelines
```

### 4. Company Management
```bash
# Create company
bigin company create --name "Acme Inc" --industry "Technology" \
  --website "https://acme.com" --employees 50 \
  --address "123 Business Park, Bengaluru"

# Search companies
bigin company search --query "industry:Technology"

# Get company with all contacts and pipelines
bigin company get --id 67890 --include-contacts --include-pipelines
```

### 5. Task & Activity Management
```bash
# Create follow-up task
bigin task create --related-to pipeline:12345678 \
  --subject "Send proposal" --due "2026-02-25" --priority "High"

# Create event (meeting)
bigin event create --related-to contact:87654321 \
  --title "Product Demo" --start "2026-02-24 14:00" --duration 30 \
  --location "Zoom"

# Log a call
bigin call create --related-to contact:87654321 \
  --subject "Discovery call" --duration 15 \
  --outcome "Interested, follow-up scheduled"

# List upcoming tasks
bigin task list --due-before "2026-02-28" --status "Open"

# Complete task
bigin task complete --id 54321
```

### 6. Pipeline Automation
```bash
# Auto-assign unassigned pipelines (round-robin)
bigin pipeline assign --unassigned --round-robin

# Create follow-up tasks for stale pipelines
bigin automation follow-up --stale-days 7 --create-tasks

# Move pipelines based on activity
bigin automation advance --auto-advance --criteria "proposal-sent-and-7-days"

# Bulk update stage
bigin pipeline bulk-update --stage "Negotiation" \
  --new-stage "Closed Won" --criteria "probability-gt-80"
```

### 7. Reporting & Analytics
```bash
# Pipeline report
bigin report pipeline --by-stage --by-owner --output pipeline-report.csv

# Sales performance
bigin report performance --owner "sales@company.com" \
  --month "2026-02" --output performance.json

# Forecast (weighted by probability)
bigin forecast --month "2026-03" --output forecast.csv

# Activity report
bigin report activity --user "me" --week "2026-08" \
  --include-calls --include-tasks --include-events
```

### 8. AI-Powered Features
```python
# Auto-enrich contact from email
"When I receive an email from a new sender, create contact and check for existing company"

# Smart pipeline scoring
"Score all open pipelines based on: last activity, email replies, stage age, company size"

# Follow-up reminders
"Which contacts haven't been contacted in 7 days? Create tasks for them."

# Meeting prep
"Before my 2 PM demo, give me: contact history, active pipelines, last 3 emails, company details"

# Pipeline health check
"Identify pipelines stuck in same stage for >14 days and suggest next actions"
```

### 9. Integration with Zoho Email Skill
```bash
# Unified workflow: Email → Bigin
# 1. Receive email from prospect
# 2. Extract sender info → Create/update contact
# 3. Check if company exists → Create if new
# 4. Create pipeline entry if none exists
# 5. Assign to sales rep
# 6. Set follow-up task
# 7. Reply with acknowledgment
```

---

## 📁 Project Structure

```
bigin-crm-skill/
├── SKILL.md                          # Skill documentation
├── config/
│   ├── bigin-config.json             # API endpoints (bigin/v2)
│   └── oauth-config.json             # Client ID/secret template
├── scripts/
│   ├── bigin_crm.py                  # Main Python module
│   ├── auth.py                       # OAuth2 flow (ZohoBigin scopes)
│   ├── pipelines.py                  # Pipeline operations (CORE!)
│   ├── contacts.py                   # Contact management
│   ├── companies.py                  # Company/Account management
│   ├── tasks.py                      # Task management
│   ├── events.py                     # Event/meeting management
│   ├── calls.py                      # Call logging
│   ├── reports.py                    # Analytics & reporting
│   └── automation.py                 # Pipeline automation
├── examples/
│   ├── bulk-import-contacts.csv      # Sample import file
│   ├── pipeline-mapping.json         # Pipeline stage mapping
│   └── automation-workflows/         # Pre-built workflows
│       ├── auto-assign.yaml
│       ├── follow-up-reminders.yaml
│       └── stage-advancement.yaml
├── tests/
│   ├── test_pipelines.py             # Pipeline tests
│   ├── test_contacts.py              # Contact tests
│   └── test_automation.py            # Automation tests
└── README.md                         # Installation & usage
```

---

## 🎯 Success Criteria

### Functional
- [ ] OAuth2 authentication with auto-refresh
- [ ] Full Pipeline CRUD (create, read, update, stage changes)
- [ ] Contact & Company management
- [ ] Task/Event/Call activity tracking
- [ ] Bulk import from CSV
- [ ] Search with criteria/filters
- [ ] Pipeline automation (auto-assign, reminders)

### Reporting
- [ ] Pipeline summary (by stage, by owner)
- [ ] Sales performance metrics
- [ ] Weighted forecast
- [ ] Activity reports

### Integration
- [ ] Works as OpenClaw skill
- [ ] CLI commands executable from terminal
- [ ] Integrates with `zoho-email-integration` skill

### Quality
- [ ] Comprehensive error handling
- [ ] Unit tests with >80% coverage
- [ ] Full documentation
- [ ] Example workflows provided

---

## 💼 Portfolio Value

**Why This Project Stands Out:**

1. **Real integration** — Working with enterprise CRM APIs
2. **Full stack** — Python backend, CLI interface, AI features
3. **Business value** — Automates actual sales workflows for small businesses
4. **Open source contribution** — First Bigin skill for OpenClaw ecosystem
5. **Foundation for growth** — Easy to extend to full Zoho CRM later

**Skills Demonstrated:**
- OAuth2 implementation
- REST API integration (Zoho/Bigin)
- Enterprise software automation
- CLI tool development
- Sales workflow design
- AI-powered business tools

---

## 📚 Resources

### Bigin API Documentation
- **Bigin API v2 Overview:** https://www.bigin.com/developer/docs/apis/v2/
- **Modules API:** https://www.bigin.com/developer/docs/apis/v2/modules-api.html
- **Bulk APIs:** https://www.bigin.com/developer/docs/apis/v2/bulk-read/overview.html
- **OAuth Guide:** https://www.bigin.com/developer/docs/apis/v2/oauth-overview.html
- **Notification APIs:** https://www.bigin.com/developer/docs/apis/v2/notifications/overview.html

### Zoho OAuth Console
- **API Console:** https://api-console.zoho.com/
- Create "Server-based Application" for OAuth2

---

## 🚀 Getting Started

### Prerequisites
- Bigin account (developer sandbox recommended)
- Python 3.8+
- `requests` library (`pip install requests`)

### Step 1: Create Zoho OAuth App
1. Go to https://api-console.zoho.com/
2. Click "Add Client" → "Server-based Application"
3. Set redirect URI: `http://localhost:8888/callback`
4. Select scopes: `ZohoBigin.modules.ALL`, `ZohoBigin.settings.ALL`
5. Note down Client ID and Client Secret

### Step 2: Initialize Project
```bash
mkdir -p ~/.openclaw/skills/bigin-crm-skill
cd ~/.openclaw/skills/bigin-crm-skill
touch SKILL.md README.md
mkdir -p scripts config examples tests
```

### Step 3: Build Auth Module First
Start with `scripts/auth.py` — OAuth is the foundation. Once auth works, everything else follows.

### Step 4: Focus on Pipelines
**Pipelines are the core module** — get this working first, then add Contacts, Companies, Tasks.

---

*Created: Saturday, February 21st, 2026*  
*Challenge Level: 🏆 Intermediate (MVP) → Advanced (Full)*
