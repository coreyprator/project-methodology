# Project Kickoff Template v3.6

## Document Variables

Replace these placeholders throughout this document:

| Variable | Description | Example |
|----------|-------------|---------|
| `{{PROJECT_NAME}}` | Human-readable project name | ArtForge |
| `{{PROJECT_SLUG}}` | Lowercase, hyphenated | artforge |
| `{{GCP_PROJECT_ID}}` | GCP project identifier | super-flashcards-475210 |
| `{{GITHUB_REPO}}` | Repository URL | coreyprator/artforge |
| `{{SQL_INSTANCE}}` | Cloud SQL instance name | flashcards-db |
| `{{SQL_INSTANCE_IP}}` | Instance IP address | 34.xxx.xxx.xxx |
| `{{DATABASE_NAME}}` | Database name | artforge |
| `{{GCS_BUCKET}}` | Storage bucket name | artforge-images |
| `{{CUSTOM_DOMAIN}}` | Optional custom domain | artforge.example.com |

---

## Phase 0: Pre-Kickoff Setup

### 0.1 Authentication Flow (MUST follow this order)

```powershell
# ============================================
# PASSKEY AUTHENTICATION SEQUENCE
# Project Lead runs this ONCE at session start
# ============================================

# Step 1: Browser authentication (opens browser, use Passkey)
gcloud auth login
# ✓ SUCCESS: Browser shows "You are now authenticated"
# ✗ FAILURE: Password prompt in terminal = wrong flow

# Step 2: Application default credentials (opens browser again)
gcloud auth application-default login
# ✓ SUCCESS: "Credentials saved to file"

# Step 3: Set project (should NOT prompt for anything)
gcloud config set project {{GCP_PROJECT_ID}}
# ✓ SUCCESS: "Updated property [core/project]"
# ✗ FAILURE: Password prompt = Step 1 failed, restart

# Step 4: Set quota project
gcloud auth application-default set-quota-project {{GCP_PROJECT_ID}}
# ✓ SUCCESS: "Credentials saved to file"

# Step 5: VERIFY before proceeding
gcloud config get-value project
# Must output: {{GCP_PROJECT_ID}}

gcloud auth list
# Must show your account with asterisk (ACTIVE)

# ============================================
# AFTER THIS POINT: VS Code AI runs all gcloud commands
# Project Lead does NOT run gcloud commands manually
# ============================================
```

> ⚠️ **CRITICAL**: If you see a password prompt at Step 3, STOP. Run `gcloud auth login` again. The Passkey flow should complete in browser without password entry.

### 0.2 Verify Project Configuration

```powershell
# Confirm correct project
gcloud config get-value project
# Should output: {{GCP_PROJECT_ID}}

# Verify authentication
gcloud auth list
# Should show your account as active
```

### 0.3 SQL Execution Environment

| Tool | Use For | GO Statement Support |
|------|---------|---------------------|
| SSMS | Schema creation, complex scripts | ✅ Yes |
| sqlcmd | Batch execution from command line | ✅ Yes |
| Cloud SQL Studio | Quick queries, single statements | ❌ No |

> **Rule**: If your SQL script contains `GO` statements, use SSMS or sqlcmd, NOT Cloud SQL Studio.

### 0.4 Development Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    NO LOCALHOST SERVERS                         │
├─────────────────────────────────────────────────────────────────┤
│  Write Code → Push to GitHub → GitHub Actions → Cloud Run      │
│                                                    ↓            │
│                                          Test on Cloud Run URL  │
└─────────────────────────────────────────────────────────────────┘
```

**Testing URL Format**: `https://{{PROJECT_SLUG}}-xxxxxxxxxx-uc.a.run.app`

Get your Cloud Run URL:
```powershell
gcloud run services describe {{PROJECT_SLUG}} --region=us-central1 --format="value(status.url)"
```

---

## Phase 1: Responsibility Matrix

| Task | Project Lead | Claude | VS Code AI |
|------|:------------:|:------:|:----------:|
| Create GitHub repository | ✅ | | |
| Create GCP project (if new) | ✅ | | |
| Create OAuth credentials | ✅ | | |
| Provide API keys | ✅ | | |
| Design system architecture | | ✅ | |
| Create database schema | | ✅ | |
| Design API endpoints | | ✅ | |
| Generate code files | | ✅ | |
| Create TEST_PLAN.md | | ✅ | |
| Create UI_DESIGN.md | | ✅ | |
| Create USER_GUIDE.md | | ✅ | |
| Create database & user in SQL | | | ✅ |
| Store secrets in Secret Manager | | | ✅ |
| Run gcloud commands | | | ✅ |
| Initialize git repository | | | ✅ |
| Create GitHub Actions workflow | | | ✅ |
| Push to GitHub | | | ✅ |
| Debug deployment issues | | | ✅ |
| Implement tests (pytest) | | | ✅ |
| Build frontend | | | ✅ |

### Responsibility Matrix Clarification

**VS Code AI Automation Principle**: When a task is assigned to VS Code AI (✅ in the VS Code AI column), the AI must:

1. **Execute commands directly** — Do not tell Project Lead to run gcloud commands
2. **Create automation scripts** — Wrap multi-step processes in .ps1 scripts
3. **Verify before reporting success** — Check command exit codes and state
4. **Handle errors** — If a command fails, debug and retry before escalating

**The ONLY exception**: If `gcloud config get-value project` returns wrong project, VS Code AI instructs Project Lead to run the Passkey authentication sequence (Phase 0.1).

| ❌ Wrong Behavior | ✅ Correct Behavior |
|-------------------|---------------------|
| "Please run: `gcloud secrets create...`" | Run the command directly in terminal |
| "Paste this value into Secret Manager" | `echo -n "value" \| gcloud secrets create...` |
| "Help debug deployment errors" | Own debugging: check logs, fix issues, retry |
| "You need to set up WIF" | Run WIF setup commands, report completion |

---

## Phase 2: Architecture Design

### 2.1 System Overview

```
[Describe the application purpose and main components]
```

### 2.2 Database Schema

```sql
-- {{DATABASE_NAME}} Database Schema
-- Execute in SSMS (contains GO statements)

USE master;
GO

-- Create database if not exists
IF NOT EXISTS (SELECT * FROM sys.databases WHERE name = '{{DATABASE_NAME}}')
BEGIN
    CREATE DATABASE {{DATABASE_NAME}};
END
GO

USE {{DATABASE_NAME}};
GO

-- Tables go here
```

### 2.3 API Endpoints

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| GET | /health | Health check | No |
| GET | /api/v1/... | ... | Yes |

### 2.4 External Integrations

| Service | Purpose | Credentials Location |
|---------|---------|---------------------|
| Example API | ... | Secret Manager: `{{PROJECT_SLUG}}-api-key` |

### 2.5 GCP Resources Required

| Resource | Name | Notes |
|----------|------|-------|
| Cloud Run Service | {{PROJECT_SLUG}} | us-central1 |
| Cloud SQL Database | {{DATABASE_NAME}} | On {{SQL_INSTANCE}} |
| Secret Manager | {{PROJECT_SLUG}}-* | All project secrets |
| GCS Bucket | {{GCS_BUCKET}} | If needed |

---

## Phase 3: Sprint 1 - Infrastructure & API

### 3.1 VS Code AI Tasks

#### Database Setup
```sql
-- Create database user (run in SSMS)
USE master;
GO

CREATE LOGIN {{PROJECT_SLUG}}_user WITH PASSWORD = '[GENERATE_SECURE_PASSWORD]';
GO

USE {{DATABASE_NAME}};
GO

CREATE USER {{PROJECT_SLUG}}_user FOR LOGIN {{PROJECT_SLUG}}_user;
ALTER ROLE db_datareader ADD MEMBER {{PROJECT_SLUG}}_user;
ALTER ROLE db_datawriter ADD MEMBER {{PROJECT_SLUG}}_user;
GO
```

#### Secret Manager Setup
```powershell
# Database connection string
echo -n "DRIVER={ODBC Driver 18 for SQL Server};SERVER={{SQL_INSTANCE_IP}};DATABASE={{DATABASE_NAME}};UID={{PROJECT_SLUG}}_user;PWD=[PASSWORD];TrustServerCertificate=yes" | `
gcloud secrets create {{PROJECT_SLUG}}-db-connection --data-file=-

# Grant Cloud Run access
gcloud secrets add-iam-policy-binding {{PROJECT_SLUG}}-db-connection `
    --member="serviceAccount:{{GCP_PROJECT_NUMBER}}-compute@developer.gserviceaccount.com" `
    --role="roles/secretmanager.secretAccessor"
```

#### Project Structure
```
{{PROJECT_SLUG}}/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI application
│   ├── config.py            # Settings from Secret Manager
│   ├── database.py          # Database connection
│   ├── models.py            # Pydantic models
│   └── routers/
│       └── api.py           # API routes
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # Pytest fixtures
│   └── test_api.py          # API tests
├── .github/
│   └── workflows/
│       └── deploy.yml       # GitHub Actions
├── Dockerfile
├── requirements.txt
├── .gitignore
└── README.md
```

#### GitHub Actions Workflow
```yaml
# .github/workflows/deploy.yml
name: Deploy to Cloud Run

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write
    
    steps:
      - uses: actions/checkout@v4
      
      - id: auth
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.WIF_PROVIDER }}
          service_account: ${{ secrets.WIF_SERVICE_ACCOUNT }}
      
      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v2
      
      - name: Build and Deploy
        run: |
          gcloud run deploy {{PROJECT_SLUG}} \
            --source . \
            --region us-central1 \
            --allow-unauthenticated \
            --set-secrets="DB_CONNECTION={{PROJECT_SLUG}}-db-connection:latest"
```

### 3.2 Sprint 1 Definition of Done

- [ ] Database created with tables and user
- [ ] All secrets stored in Secret Manager
- [ ] API endpoints accessible on Cloud Run URL
- [ ] Authentication working (if applicable)
- [ ] Database CRUD operations succeed
- [ ] GitHub Actions deploys successfully on push
- [ ] TEST_PLAN.md created
- [ ] UI_DESIGN.md created
- [ ] USER_GUIDE.md created

---

## Phase 4: Sprint 2 - Testing, UI & Documentation

### 4.1 Testing Philosophy

> **If it's in the requirements or user documentation, it MUST be tested.**

| Category | What to Test |
|----------|--------------|
| Happy Path | Every documented feature works as described |
| Obvious Exceptions | User-friendly error messages for common mistakes |

#### Error Message Guidelines

```python
# ❌ BAD - Exposes internal implementation
raise HTTPException(status_code=500, detail="NoneType has no attribute 'id'")

# ✅ GOOD - User-friendly, actionable
raise HTTPException(status_code=404, detail="Collection not found. It may have been deleted.")
```

System/infrastructure errors (database down, API timeout) can bubble up as-is—users have AI assistants to help interpret technical errors.

#### Coverage Requirements

```yaml
# In GitHub Actions
- name: Run tests
  run: |
    pytest --cov=app --cov-report=xml --cov-fail-under=70
```

### 4.2 Help Content Strategy

| Approach | Implementation |
|----------|----------------|
| Main Help | Link to GitHub docs: `https://github.com/{{GITHUB_REPO}}/blob/main/docs/USER_GUIDE.md` |
| Tooltips | Inline hints for complex fields |
| Welcome | First-time user message explaining core workflow |

**Why external links?** Documentation can be updated without redeploying the application.

### 4.3 Sprint 2 Definition of Done

- [ ] pytest test suite implemented
- [ ] All tests pass in CI
- [ ] Coverage ≥ 70%
- [ ] Frontend matches UI_DESIGN.md
- [ ] User can complete core workflow end-to-end
- [ ] Help link accessible and working
- [ ] Error messages are user-friendly
- [ ] Welcome/onboarding message displays for new users

---

## Phase 5: Deployment Verification

### 5.1 Post-Deployment Checklist

```powershell
# Get Cloud Run URL
$URL = gcloud run services describe {{PROJECT_SLUG}} --region=us-central1 --format="value(status.url)"

# Health check
curl "$URL/health"

# API check (adjust endpoint as needed)
curl "$URL/api/v1/status"
```

### 5.2 Verify Secrets Access

```powershell
# List secrets
gcloud secrets list --filter="name:{{PROJECT_SLUG}}"

# Verify IAM binding
gcloud secrets get-iam-policy {{PROJECT_SLUG}}-db-connection
```

### 5.3 Verify Database Connectivity

From Cloud Run logs:
```powershell
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name={{PROJECT_SLUG}}" --limit=50
```

---

## Appendix A: Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Password prompt during `gcloud config set project` | Auth not completed | Run `gcloud auth login` first |
| SQL script fails in Cloud SQL Studio | GO statements not supported | Use SSMS or sqlcmd |
| "Secret not found" error | IAM binding missing | Add secretAccessor role to compute service account |
| Cloud Run deployment fails | Dockerfile error | Check build logs with `gcloud builds list` |
| Database connection timeout | IP not authorized | Verify Cloud SQL authorized networks |

---

## Appendix B: Reference Commands

### GCP Authentication
```powershell
gcloud auth login
gcloud auth application-default login
gcloud config set project {{GCP_PROJECT_ID}}
gcloud auth application-default set-quota-project {{GCP_PROJECT_ID}}
```

### Cloud Run
```powershell
# Deploy
gcloud run deploy {{PROJECT_SLUG}} --source . --region us-central1

# Get URL
gcloud run services describe {{PROJECT_SLUG}} --region=us-central1 --format="value(status.url)"

# View logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name={{PROJECT_SLUG}}" --limit=50
```

### Secret Manager
```powershell
# Create secret
echo -n "secret-value" | gcloud secrets create {{PROJECT_SLUG}}-secret-name --data-file=-

# Grant access
gcloud secrets add-iam-policy-binding {{PROJECT_SLUG}}-secret-name \
    --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
    --role="roles/secretmanager.secretAccessor"

# Read secret (for verification)
gcloud secrets versions access latest --secret={{PROJECT_SLUG}}-secret-name
```

### Git/GitHub
```powershell
git init
git remote add origin https://github.com/{{GITHUB_REPO}}.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

---

## Appendix C: Cloud Migration Script Template

Use this script template for migrating localhost projects to cloud-first deployment. VS Code AI should customize and execute this script directly.

```powershell
# {{PROJECT_SLUG}}-Cloud-Migration.ps1
# VS Code AI: Run this script to complete cloud migration
# Per project-methodology v3.6

param(
    [string]$ProjectId = "{{GCP_PROJECT_ID}}",
    [string]$ProjectSlug = "{{PROJECT_SLUG}}",
    [string]$Region = "us-central1",
    [string]$SqlInstanceIp = "{{SQL_INSTANCE_IP}}",
    [string]$DatabaseName = "{{DATABASE_NAME}}",
    [string]$DatabaseUser = "{{PROJECT_SLUG}}_user",
    [string]$DatabasePassword = "{{SECURE_PASSWORD}}"
)

Write-Host "========================================" -ForegroundColor Cyan
Write-Host "  $ProjectSlug Cloud Migration Script" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan

# ========================================
# Step 1: Verify Authentication
# ========================================
Write-Host "`nStep 1: Verifying GCP authentication..." -ForegroundColor Yellow

$currentProject = gcloud config get-value project 2>$null
if ($currentProject -ne $ProjectId) {
    Write-Host "  ✗ Wrong project: $currentProject" -ForegroundColor Red
    Write-Host "  Project Lead must run Passkey authentication sequence first." -ForegroundColor Red
    Write-Host "  See: PROJECT_KICKOFF_TEMPLATE.md Phase 0.1" -ForegroundColor Gray
    exit 1
}
Write-Host "  ✓ Authenticated to project: $currentProject" -ForegroundColor Green

# ========================================
# Step 2: Create/Update Secrets
# ========================================
Write-Host "`nStep 2: Configuring Secret Manager..." -ForegroundColor Yellow

$secrets = @{
    "$ProjectSlug-db-server" = $SqlInstanceIp
    "$ProjectSlug-db-name" = $DatabaseName
    "$ProjectSlug-db-user" = $DatabaseUser
    "$ProjectSlug-db-password" = $DatabasePassword
}

foreach ($secretName in $secrets.Keys) {
    $secretValue = $secrets[$secretName]
    $exists = gcloud secrets describe $secretName --project=$ProjectId 2>$null
    
    if ($exists) {
        Write-Host "  Updating secret '$secretName'..." -ForegroundColor Cyan
        $secretValue | gcloud secrets versions add $secretName --data-file=- --project=$ProjectId 2>$null
    } else {
        Write-Host "  Creating secret '$secretName'..." -ForegroundColor Cyan
        $secretValue | gcloud secrets create $secretName --data-file=- --project=$ProjectId 2>$null
    }
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "    ✓ $secretName" -ForegroundColor Green
    } else {
        Write-Host "    ✗ Failed: $secretName" -ForegroundColor Red
    }
}

# ========================================
# Step 3: Grant IAM Access
# ========================================
Write-Host "`nStep 3: Granting IAM access..." -ForegroundColor Yellow

$PROJECT_NUMBER = (gcloud projects describe $ProjectId --format="value(projectNumber)")
$SA = "$PROJECT_NUMBER-compute@developer.gserviceaccount.com"

foreach ($secretName in $secrets.Keys) {
    gcloud secrets add-iam-policy-binding $secretName `
        --member="serviceAccount:$SA" `
        --role="roles/secretmanager.secretAccessor" `
        --project=$ProjectId 2>$null | Out-Null
    Write-Host "  ✓ IAM binding: $secretName" -ForegroundColor Green
}

# ========================================
# Step 4: Verify Infrastructure Files
# ========================================
Write-Host "`nStep 4: Checking infrastructure files..." -ForegroundColor Yellow

$requiredFiles = @(
    @{ Path = "Dockerfile"; Description = "Container definition" },
    @{ Path = ".github/workflows/deploy.yml"; Description = "CI/CD pipeline" },
    @{ Path = "requirements.txt"; Description = "Python dependencies" }
)

foreach ($file in $requiredFiles) {
    if (Test-Path $file.Path) {
        Write-Host "  ✓ $($file.Path)" -ForegroundColor Green
    } else {
        Write-Host "  ✗ MISSING: $($file.Path) - $($file.Description)" -ForegroundColor Red
    }
}

# ========================================
# Step 5: Summary
# ========================================
Write-Host "`n========================================" -ForegroundColor Cyan
Write-Host "  Migration Complete" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan
Write-Host "`nNext steps:" -ForegroundColor Yellow
Write-Host "  1. git add . && git commit -m 'Cloud migration' && git push" -ForegroundColor White
Write-Host "  2. Monitor: https://github.com/{{GITHUB_REPO}}/actions" -ForegroundColor White
Write-Host "  3. Test: gcloud run services describe $ProjectSlug --region=$Region --format='value(status.url)'" -ForegroundColor White
```

---

**Template Version**: 3.6  
**Last Updated**: December 2024  
**Methodology**: [coreyprator/project-methodology](https://github.com/coreyprator/project-methodology)
