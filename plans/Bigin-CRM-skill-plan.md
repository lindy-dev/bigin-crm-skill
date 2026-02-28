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

## 🛠️ Core Features to Build

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

## 🔧 Technical Implementation Reference

### Core Python Module
```python
# scripts/bigin_crm.py
import requests
import json
from datetime import datetime

class BiginCRM:
    def __init__(self, auth_token, dc='com'):
        """
        Initialize Bigin CRM client
        dc: Data center - com, eu, in, au, etc.
        """
        self.base_url = f"https://www.zohoapis.{dc}/bigin/v2"
        self.headers = {
            "Authorization": f"Zoho-oauthtoken {auth_token}",
            "Content-Type": "application/json"
        }
    
    # ==================== PIPELINES (CORE) ====================
    
    def create_pipeline(self, data):
        """Create a pipeline entry (equivalent to Deal in CRM)"""
        url = f"{self.base_url}/Pipelines"
        payload = {"data": [data]}
        response = requests.post(url, headers=self.headers, json=payload)
        return response.json()
    
    def get_pipelines(self, stage=None, owner=None, limit=200):
        """Get pipeline records with optional filters"""
        url = f"{self.base_url}/Pipelines"
        params = {"per_page": limit}
        
        # Build criteria string
        criteria = []
        if stage:
            criteria.append(f"(Stage:equals:{stage})")
        if owner:
            criteria.append(f"(Owner:equals:{owner})")
        
        if criteria:
            params["criteria"] = " and ".join(criteria)
        
        response = requests.get(url, headers=self.headers, params=params)
        return response.json()
    
    def update_pipeline(self, pipeline_id, data):
        """Update a pipeline record (e.g., change stage)"""
        url = f"{self.base_url}/Pipelines/{pipeline_id}"
        payload = {"data": [data]}
        response = requests.put(url, headers=self.headers, json=payload)
        return response.json()
    
    def advance_pipeline(self, pipeline_id, new_stage=None):
        """Move pipeline to next stage or specified stage"""
        update_data = {}
        if new_stage:
            update_data["Stage"] = new_stage
        # Could also implement logic to determine next stage automatically
        return self.update_pipeline(pipeline_id, update_data)
    
    # ==================== CONTACTS ====================
    
    def create_contact(self, data):
        """Create a contact"""
        url = f"{self.base_url}/Contacts"
        payload = {"data": [data]}
        response = requests.post(url, headers=self.headers, json=payload)
        return response.json()
    
    def get_contacts(self, criteria=None, limit=200):
        """Get contacts with optional criteria"""
        url = f"{self.base_url}/Contacts"
        params = {"per_page": limit}
        if criteria:
            params["criteria"] = criteria
        response = requests.get(url, headers=self.headers, params=params)
        return response.json()
    
    def search_contacts(self, query):
        """Search contacts by keyword"""
        url = f"{self.base_url}/Contacts/search?word={query}"
        response = requests.get(url, headers=self.headers)
        return response.json()
    
    # ==================== COMPANIES (ACCOUNTS) ====================
    
    def create_company(self, data):
        """Create a company (uses Accounts API)"""
        url = f"{self.base_url}/Accounts"
        payload = {"data": [data]}
        response = requests.post(url, headers=self.headers, json=payload)
        return response.json()
    
    def get_companies(self, criteria=None, limit=200):
        """Get companies"""
        url = f"{self.base_url}/Accounts"
        params = {"per_page": limit}
        if criteria:
            params["criteria"] = criteria
        response = requests.get(url, headers=self.headers, params=params)
        return response.json()
    
    # ==================== TASKS ====================
    
    def create_task(self, subject, related_to=None, due_date=None, owner=None):
        """Create a follow-up task"""
        data = {"Subject": subject}
        
        if due_date:
            data["Due_Date"] = due_date
        if owner:
            data["Owner"] = {"name": owner}
        if related_to:
            # related_to format: "module:record_id" e.g., "Pipelines:12345"
            module, record_id = related_to.split(":")
            data["What_Id"] = {"id": record_id}
        
        url = f"{self.base_url}/Tasks"
        payload = {"data": [data]}
        response = requests.post(url, headers=self.headers, json=payload)
        return response.json()
    
    def get_tasks(self, due_before=None, status="Open", limit=200):
        """Get tasks with filters"""
        url = f"{self.base_url}/Tasks"
        params = {"per_page": limit}
        
        criteria = [f"(Status:equals:{status})"]
        if due_before:
            criteria.append(f"(Due_Date:less_than:{due_before})")
        
        params["criteria"] = " and ".join(criteria)
        response = requests.get(url, headers=self.headers, params=params)
        return response.json()
    
    # ==================== BULK OPERATIONS ====================
    
    def bulk_import_contacts(self, file_path):
        """Import contacts from CSV/JSON"""
        # Uses Bigin bulk API
        pass
    
    # ==================== REPORTING ====================
    
    def get_pipeline_report(self, by_stage=True, by_owner=True):
        """Generate pipeline summary report"""
        pipelines = self.get_pipelines(limit=200)
        # Process and aggregate data
        report = {
            "total_count": 0,
            "by_stage": {},
            "by_owner": {},
            "total_value": 0
        }
        # ... aggregation logic
        return report
```

### Authentication Flow
```python
# scripts/auth.py
import webbrowser
import http.server
import socketserver
import urllib.parse
import json
import os

class BiginOAuth:
    def __init__(self, client_id, client_secret, dc='com'):
        self.client_id = client_id
        self.client_secret = client_secret
        self.dc = dc
        self.redirect_uri = "http://localhost:8888/callback"
        # Bigin-specific scopes
        self.scope = "ZohoBigin.modules.ALL,ZohoBigin.settings.ALL"
        self.token_file = os.path.expanduser("~/.openclaw/credentials/bigin-crm.json")
    
    def get_auth_url(self):
        """Generate authorization URL for Bigin"""
        return (
            f"https://accounts.zoho.{self.dc}/oauth/v2/auth?"
            f"scope={self.scope}&"
            f"client_id={self.client_id}&"
            f"response_type=code&"
            f"access_type=offline&"
            f"redirect_uri={self.redirect_uri}"
        )
    
    def start_auth_flow(self):
        """Start local server and authenticate"""
        webbrowser.open(self.get_auth_url())
        # Start local server to catch callback
        # Exchange code for tokens
        # Save to token_file
        pass
    
    def refresh_token(self, refresh_token):
        """Refresh expired access token"""
        url = f"https://accounts.zoho.{self.dc}/oauth/v2/token"
        data = {
            "refresh_token": refresh_token,
            "client_id": self.client_id,
            "client_secret": self.client_secret,
            "grant_type": "refresh_token"
        }
        response = requests.post(url, data=data)
        return response.json()
    
    def get_access_token(self):
        """Get valid access token (refresh if needed)"""
        # Load from file
        # Check expiry
        # Refresh if expired
        # Return valid token
        pass
```

---

## 📅 Implementation Plan (4-5 Weeks)

### Week 1: Foundation & Pipeline Core
- [ ] Set up Bigin developer account (https://www.bigin.com/)
- [ ] Create OAuth app in Zoho API Console
- [ ] Build authentication module (OAuth2 flow)
- [ ] Implement Pipeline CRUD operations (create, read, update)
- [ ] Test basic API calls

### Week 2: Contacts & Companies
- [ ] Implement Contact management (create, update, search)
- [ ] Implement Company/Account management
- [ ] Bulk import from CSV
- [ ] Link Contacts to Companies
- [ ] Link Contacts to Pipelines

### Week 3: Activities & Automation
- [ ] Task management (create, complete, list)
- [ ] Event/Meeting management
- [ ] Call logging
- [ ] Pipeline automation (auto-assign, stage advancement)
- [ ] Follow-up reminders

### Week 4: Reporting & AI Features
- [ ] Pipeline reports (by stage, by owner)
- [ ] Sales performance analytics
- [ ] Forecasting (weighted pipeline)
- [ ] Activity reports
- [ ] AI-powered features (lead scoring, meeting prep)

### Week 5: Polish & Integration
- [ ] CLI command structure
- [ ] Error handling & validation
- [ ] Unit tests (target 80%+ coverage)
- [ ] Documentation (SKILL.md, README.md)
- [ ] Example workflows
- [ ] Integration with zoho-email skill
- [ ] Demo video
- [ ] Publish to ClawHub

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

### Similar Skills for Reference
- **zoho-email-integration** — Reference for Zoho OAuth patterns
  ```bash
  clawhub install zoho-email-integration
  ```

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

## 🎉 Challenge Ready?

This is a **substantial but achievable project** that will:
- Teach you enterprise API integration
- Give you a unique portfolio piece (first Bigin skill!)
- Solve a real problem for small business sales teams
- Build foundation for full Zoho CRM skill later

**Estimated effort:** 35-45 hours over 4-5 weeks  
**Difficulty:** Intermediate  
**Impact:** High — fills genuine gap in ecosystem  
**Extensibility:** Easy to add Zoho CRM modules later

**Key insight:** Start with **Pipelines** (core), then **Contacts** and **Tasks**. Everything else is optional for MVP.

---

*Created: Saturday, February 21st, 2026*  
*Challenge Level: 🏆 Intermediate (MVP) → Advanced (Full)*
