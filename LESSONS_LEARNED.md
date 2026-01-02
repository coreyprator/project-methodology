# Lessons Learned Log

This document captures issues encountered during project development, their root causes, and the fixes applied to the methodology.

## Format

Each entry includes:
- **Date**: When the issue was encountered
- **Project**: Which project surfaced the issue
- **Issue**: What went wrong
- **Root Cause**: Why it happened
- **Fix Applied**: How the methodology was updated
- **Template Section**: Where the fix lives

---

## 2024-12 Lessons (v3.5)

### LL-001: VS Code AI Didn't Create Database User

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | VS Code AI generated code expecting database user, but never created the user in SQL Server |
| Root Cause | Responsibility for database user creation was not explicitly assigned |
| Fix Applied | Added "Create database & user in SQL" to VS Code AI responsibilities in the Responsibility Matrix |
| Template Section | Phase 1: Responsibility Matrix |

**Impact**: Deployment failed with authentication errors. Required manual intervention to create the database user.

---

### LL-002: Password Prompt Blocking GCP Commands

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | `gcloud config set project` prompted for password, blocking automated workflows |
| Root Cause | Authentication order was wrong—project was set before browser auth completed |
| Fix Applied | Created Phase 0 with explicit authentication order. Added warning about password prompts. |
| Template Section | Phase 0: Pre-Kickoff Setup, Section 0.1 |

**Impact**: VS Code AI couldn't proceed with gcloud commands. User had to manually intervene with authentication.

**Key Learning**: The correct order is:
1. `gcloud auth login` (browser/Passkey)
2. `gcloud auth application-default login`
3. `gcloud config set project` (should NOT prompt)
4. `gcloud auth application-default set-quota-project`

---

### LL-003: Wrong GCP Project Selected

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | Commands ran against wrong GCP project, creating resources in unexpected location |
| Root Cause | Project was not verified at the start of the session |
| Fix Applied | Added project verification step after authentication |
| Template Section | Phase 0: Pre-Kickoff Setup, Section 0.2 |

**Impact**: Had to clean up resources from wrong project and recreate in correct project.

---

### LL-004: "Project Does Not Exist" Errors

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | HarmonyLab (planning) |
| Issue | Kickoff document referenced GCP project that hadn't been created yet |
| Root Cause | Decision to use new vs. existing project wasn't made upfront |
| Fix Applied | Added pre-kickoff questions about GCP project choice |
| Template Section | Pre-Kickoff Checklist (separate document) |

**Impact**: Wasted time debugging "project not found" errors that were actually planning issues.

---

### LL-005: SQL Scripts Failed in Cloud SQL Studio

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | SQL scripts with GO statements failed in Cloud SQL Studio |
| Root Cause | Cloud SQL Studio doesn't support GO batch separator |
| Fix Applied | Added SQL Execution Environment section specifying which tool to use |
| Template Section | Phase 0: Pre-Kickoff Setup, Section 0.3 |

**Impact**: Schema creation failed. Had to re-run scripts in SSMS.

**Key Learning**: 
- SSMS/sqlcmd: Full T-SQL support including GO
- Cloud SQL Studio: Single statements only

---

### LL-006: VS Code AI Ran Localhost Server

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | VS Code AI started local development server with `uvicorn` instead of deploying to Cloud Run |
| Root Cause | Default developer workflow is local-first; cloud-first not documented |
| Fix Applied | Added "NO LOCALHOST SERVERS" section with cloud-first workflow diagram |
| Template Section | Phase 0: Pre-Kickoff Setup, Section 0.4 |

**Impact**: Testing was done against localhost, then failed on Cloud Run due to environment differences.

**Key Learning**: The workflow is always: Write → Push → Deploy → Test on Cloud Run URL

---

### LL-007: Git Repository Not Initialized

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | VS Code AI generated code but didn't initialize git or push to GitHub |
| Root Cause | Git initialization wasn't in the task list |
| Fix Applied | Added git init, remote add, commit, push to VS Code AI responsibilities |
| Template Section | Phase 1: Responsibility Matrix |

**Impact**: Code sat locally; GitHub Actions never triggered.

---

### LL-008: No Test Plan Created

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | Sprint 1 completed but no tests existed and no plan for what to test |
| Root Cause | TEST_PLAN.md wasn't a required deliverable |
| Fix Applied | Added TEST_PLAN.md to Claude's required deliverables; added to Sprint 1 DoD |
| Template Section | Phase 1: Responsibility Matrix, Phase 3: Sprint 1 DoD |

**Impact**: Sprint 2 started without clarity on what needed testing.

---

### LL-009: No UI Design Document

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | Frontend development started without any UI design specification |
| Root Cause | UI_DESIGN.md wasn't a required deliverable |
| Fix Applied | Added UI_DESIGN.md to Claude's required deliverables |
| Template Section | Phase 1: Responsibility Matrix |

**Impact**: Frontend had to be reworked multiple times due to unclear requirements.

---

### LL-010: Users Confused by Application

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | Users didn't understand how to use the application |
| Root Cause | No user documentation existed |
| Fix Applied | Added USER_GUIDE.md to Claude's required deliverables |
| Template Section | Phase 1: Responsibility Matrix |

**Impact**: Support burden on developer; users abandoned application.

---

### LL-011: Technical Errors Exposed to Users

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | Users saw errors like `NoneType has no attribute 'id'` |
| Root Cause | No guidelines for user-friendly error messages |
| Fix Applied | Added Testing Philosophy section with error message guidelines |
| Template Section | Phase 4: Sprint 2, Section 4.1 |

**Impact**: Users confused by technical errors; poor user experience.

**Key Learning**:
- Happy path errors: User-friendly messages
- System errors: Can bubble up (users have AI to explain)

---

### LL-012: Help Content Embedded in Application

| Field | Value |
|-------|-------|
| Date | 2024-12-27 |
| Project | ArtForge |
| Issue | Help documentation was embedded in React components |
| Root Cause | No strategy defined for help content |
| Fix Applied | Changed to external link strategy—help links to GitHub docs |
| Template Section | Phase 4: Sprint 2, Section 4.2 |

**Impact**: Every documentation fix required a full redeploy.

**Key Learning**: External help links allow doc updates without app deployment.

---

## 2024-12 Lessons (v3.6)

### LL-013: VS Code AI Delegated Automation to Project Lead

| Field | Value |
|-------|-------|
| Date | 2024-12-28 |
| Project | HarmonyLab |
| Issue | VS Code AI told Project Lead to manually create secrets and run gcloud commands instead of automating |
| Root Cause | Responsibility matrix listed tasks but didn't emphasize automation over instruction |
| Fix Applied | Added "Responsibility Matrix Clarification" section; created cloud migration script template |
| Template Section | Phase 1: Responsibility Matrix, Appendix C: Cloud Migration Script |

**Impact**: Project Lead had to repeatedly redirect VS Code AI to automate tasks. Slowed migration by 2+ hours.

**Key Learning**: 
- VS Code AI must execute, not instruct
- Provide executable script templates, not command lists
- The only manual step is Passkey authentication (Phase 0.1)

---

## 2025-12 Lessons (v3.7)

### LL-014: Smoke Tests Before User Testing

| Field | Value |
|-------|-------|
| Date | 2025-12-30 |
| Project | ArtForge |
| Issue | User attempted to test ArtForge UI but received "Internal Server Error: Unexpected token 'I'" - could not test any functionality |
| Root Cause | Sprint 1 Definition of Done only checked "code deployed" - no verification that the deployed app actually works |
| Fix Applied | Added "Deployment Verification" phase with layered smoke tests between deployment and user testing |
| Template Section | SPRINT_1_CHECKLIST.md Phase 5, SMOKE_TEST_TEMPLATE.md |

**Impact**: 
- User testing completely blocked
- Debug cycle added: diagnose → fix → redeploy → test
- Confusion about whether unit tests or deployment was the problem

**Key Learning**: "Deployed" ≠ "Working". Smoke tests verify the deployed app works before users touch it.

---

### LL-015: Smoke Tests vs Unit Tests - Different Purposes

| Field | Value |
|-------|-------|
| Date | 2025-12-30 |
| Project | ArtForge |
| Issue | Team was stuck deciding between "fix 21 failing unit tests" vs "deploy anyway" - neither option addressed the real problem |
| Root Cause | Confusion about test types: unit tests verify code logic with mocks, smoke tests verify deployed app works |
| Fix Applied | Added "Test Types and Purposes" section clarifying when each test type is required |
| Template Section | PROJECT_KICKOFF_TEMPLATE.md Phase 4.1 Testing Philosophy |

**Impact**:
- Hours spent on wrong priority
- Unit tests written for unverified behavior
- No tests for actual deployment health

**Key Learning**: 
- Smoke tests: Run against deployed URL, verify app works
- Unit tests: Run against mocks, verify code logic
- The 21 failing unit tests were testing *assumed* behavior - the app had never been verified to work

---

### LL-016: Testing Phase Sequence

| Field | Value |
|-------|-------|
| Date | 2025-12-30 |
| Project | ArtForge |
| Issue | Tests were written before confirming the feature worked, then blocked deployment when they failed |
| Root Cause | Assumed sequence was: Write code → Write tests → Deploy → User test |
| Fix Applied | Documented correct sequence for new features vs existing features |
| Template Section | PROJECT_KICKOFF_TEMPLATE.md Phase 4.1 Testing Philosophy |

**Impact**: 
- Circular dependency: Can't deploy because tests fail, can't verify behavior because not deployed
- Tests based on assumptions, not reality

**Key Learning**: For NEW features:
```
Write code → Deploy → Smoke test → User test → Write unit tests based on verified behavior
```
For EXISTING features: Unit tests are blocking (prevent regression).

---

### LL-017: Database Backups Not Verified

| Field | Value |
|-------|-------|
| Date | 2025-12-31 |
| Project | All Projects |
| Issue | Databases deployed to production without verified backup and disaster recovery configuration |
| Root Cause | No backup verification step in Sprint 1 checklist or deployment verification |
| Fix Applied | Added backup configuration to Sprint 1 checklist, DEPLOYMENT_CHECKLIST, PRE_KICKOFF_CHECKLIST; created BACKUP_VERIFICATION.md with automated verification script |
| Template Section | SPRINT_1_CHECKLIST.md, DEPLOYMENT_CHECKLIST.md, PRE_KICKOFF_CHECKLIST.md, BACKUP_VERIFICATION.md |

**Impact**: 
- Risk of data loss without recovery option
- No disaster recovery capability for production databases
- Potential compliance/audit issues

**Key Learning**: 
- Backups must be verified, not assumed
- Every database needs: automated backups enabled, PITR for production, verified retention period
- Automated verification script catches gaps across all instances

---

## 2026-01 Lessons (v3.9)

### LL-018: RTFM - AI Agents Don't Read Project Documentation

| Field | Value |
|-------|-------|
| Date | 2026-01-01 |
| Project | Etymython |
| Issue | VS Code built Sprint 4 content generation without consulting the database schema. Result: queries that don't match actual table structures, JOINs that fail, misunderstanding of data relationships |
| Root Cause | AI agents start coding immediately without reviewing existing documentation, schemas, or prior work |
| Fix Applied | Added Pre-Work Checklist (mandatory before writing code) and Violation Response Protocol |
| Template Section | PROJECT_KICKOFF_TEMPLATE.md Phase 3, VIOLATION_RESPONSE.md |

**Impact**: 
- Queries joining tables incorrectly
- Scripts assuming wrong column names
- Hours of debugging preventable issues

**Key Learning**: AI agents must be explicitly required to read documentation before coding. Add "Confirm understanding with Project Lead before proceeding" as a gate.

---

### LL-019: Automation First - State What TO DO, Not What NOT To Do

| Field | Value |
|-------|-------|
| Date | 2026-01-01 |
| Project | Etymython |
| Issue | AI agents repeatedly perform manual operations (password prompts, manual data entry, interactive confirmations) despite methodology prohibiting them |
| Root Cause | Methodology stated what NOT to do ("don't use getpass") leaving infinite wrong choices. Should state exactly what TO DO (one correct choice) |
| Fix Applied | Rewrote automation requirements as positive instructions with specific code examples |
| Template Section | PROJECT_KICKOFF_TEMPLATE.md Phase 1 Responsibility Matrix Clarification |

**Impact**: 
- Scripts hanging waiting for password input
- Manual API key entry requests blocking automation
- Interactive confirmations breaking CI/CD

**Key Learning**: 
- **Parent Principle**: Anything that CAN be automated SHOULD be automated
- Give one correct way, not infinite wrong ways to avoid

---

### LL-020: AI Lacks Metacognition - External Verification Required

| Field | Value |
|-------|-------|
| Date | 2026-01-01 |
| Project | Etymython |
| Issue | AI agent reported "ROUNDTRIP SUCCESS" when the test was actually failing. The test compared corrupted-input to corrupted-output, both having the same bug, so they matched |
| Root Cause | AI cannot evaluate its own output quality. It trusts its own tests, which may have the same blind spots as its code |
| Fix Applied | Established Golden Audit governance model - truth sources owned by Architect, read-only for VS Code |
| Template Section | VERIFICATION_GOVERNANCE.md |

**Impact**: 
- False success reports masking real failures
- Corrupted data (Unicode 206 mojibake) passing "tests"
- Project Lead time wasted investigating "passing" tests

**Key Learning**: 
- **The programmer cannot grade their own test**
- VS Code's tests = CLAIMS (not proof)
- Golden Audit = SINGLE SOURCE OF TRUTH (read-only for VS Code)

---

### LL-021: AI Memory Loss - Context Doesn't Persist

| Field | Value |
|-------|-------|
| Date | 2026-01-01 |
| Project | Etymython |
| Issue | VS Code repeatedly forgets established patterns, configurations, and decisions from earlier in the same project or conversation |
| Root Cause | AI context windows are limited. Long conversations lose early context. Separate chat sessions have no shared memory |
| Fix Applied | Added context reinforcement strategies: explicit citations, instruction templates, periodic reminders |
| Template Section | PROJECT_KICKOFF_TEMPLATE.md, VIOLATION_RESPONSE.md |

**Impact**: 
- Forgetting passkey authentication after being told multiple times
- Reverting to default patterns instead of project-specific ones
- Disproven theories persisting across sessions

**Key Learning**: 
- Use explicit citations: "Per PROJECT_METHODOLOGY.md section 4.2..."
- Every ~10 exchanges, restate critical constraints
- When violated, require lookup and confirmation before proceeding

---

### LL-022: Unit Before Bulk - Testing Sequence

| Field | Value |
|-------|-------|
| Date | 2026-01-01 |
| Project | Etymython |
| Issue | VS Code jumped from "code written" directly to "bulk execution" of 56 records, causing widespread data corruption |
| Root Cause | No enforced testing sequence requiring single-record verification before batch operations |
| Fix Applied | Mandatory testing sequence: Smoke → Unit (1 record) → Small Batch (3-5) → Bulk. Added bulk operation checklist |
| Template Section | SPRINT_1_CHECKLIST.md, TEST_PLAN_TEMPLATE.md |

**Impact**: 
- Bulk operations applying bugs to all records simultaneously
- No rollback possible when entire dataset corrupted
- Multiple recovery attempts needed

**Key Learning**: 
```
NEVER skip from "code written" to "bulk execution"
Sequence: Smoke test → Unit test (1 record) → Small batch (3-5) → Bulk
```

---

### LL-023: Unicode/Encoding Requires Special Attention

| Field | Value |
|-------|-------|
| Date | 2026-01-01 |
| Project | Etymython |
| Issue | Greek Unicode text corrupted multiple times due to encoding mismatches between Python, pyodbc, and SQL Server |
| Root Cause | String comparison doesn't catch encoding bugs. SQL Server NVARCHAR uses UTF-16, Python uses UTF-8, pyodbc configuration is non-obvious |
| Fix Applied | Added Unicode handling requirements: verify with UNICODE() not string comparison, UTF-16LE encoding for pyodbc |
| Template Section | UNICODE_HANDLING.md |

**Impact**: 
- Original `Ζεύς` (Unicode 918) became `????` then `Î–ÎµÏÏ‚` (mojibake)
- Each "fix" made data worse
- 56 records corrupted before root cause found

**Key Learning**: 
- Test with UNICODE values: `SELECT UNICODE(LEFT(column, 1))` 
- Greek should be 913-1023 (basic) or 7936-8191 (extended)
- If you see 63, 206, 195, 225 = encoding bug
- pyodbc setup: `conn.setencoding(encoding='utf-16-le')`

---

### LL-024: Error Recovery - Don't Make It Worse

| Field | Value |
|-------|-------|
| Date | 2026-01-01 |
| Project | Etymython |
| Issue | Each "fix" attempt made the data worse, not better. Pressure to show progress led to untested fixes that compounded problems |
| Root Cause | No error recovery protocol. Fixes applied to all affected records without isolated testing |
| Fix Applied | Error Recovery Protocol: STOP → PRESERVE → ISOLATE → VERIFY → ROLLBACK PLAN |
| Template Section | PROJECT_KICKOFF_TEMPLATE.md Appendix D |

**Impact**: 
- Original: Some records had `????` (data lost but recoverable)
- After fix attempt: All 56 records had mojibake (worse than before)
- Multiple attempts before finally working

**Key Learning**: 
```
BACKUP BEFORE FIX: SELECT * INTO table_backup_YYYYMMDD FROM table;
NEVER apply untested fixes to all affected records simultaneously.
```

---

## Summary Statistics

| Metric | Value |
|--------|-------|
| Total Lessons | 24 |
| Template Sections Updated | 15 |
| New Sections Added | 12 |
| Projects Contributing | ArtForge, HarmonyLab, Etymython |

## How to Add New Lessons

1. Create entry with the next LL number (e.g., LL-013)
2. Fill in all fields in the table format
3. Add Impact and Key Learning sections if significant
4. Update the Summary Statistics
5. Update the relevant template section
6. Increment version in CHANGELOG.md
