# Changelog

All notable changes to this methodology are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [3.8.0] - 2025-12-31

### Added
- **BACKUP_VERIFICATION.md**: New template with automated PowerShell script to verify all Cloud SQL backups
- **Database Backup & DR section** (SPRINT_1_CHECKLIST.md): Required backup configuration before user testing
- **Backup Configuration section** (DEPLOYMENT_CHECKLIST.md): Verification that backups are enabled and successful
- **Backup questions** (PRE_KICKOFF_CHECKLIST.md): Retention period, PITR, and backup window decisions
- **LL-017**: Database backups must be verified, not assumed

### Changed
- **Sprint 1 Definition of Done**: Now requires "Database backups enabled and verified"
- **README.md**: Added "Backups Verified" principle

### Key Insight
> **Backups must be verified, not assumed.**
> 
> Every database needs: automated backups enabled, PITR for production, retention period set, and at least one successful backup completed.

---

## [3.7.0] - 2025-12-30

### Added
- **Smoke Test Template** (SMOKE_TEST_TEMPLATE.md): Layered smoke tests to verify deployment works before user testing
- **Phase 6: Smoke Testing** (SPRINT_1_CHECKLIST.md): Required verification phase between deployment and user testing
- **Test Types and Purposes** (PROJECT_KICKOFF_TEMPLATE.md): Clarifies smoke vs unit vs integration tests
- **Testing Sequence for New Features**: Write code → Deploy → Smoke test → User test → Unit tests
- **LL-014**: Smoke tests required before user testing (ArtForge "Internal Server Error" incident)
- **LL-015**: Smoke tests vs unit tests serve different purposes
- **LL-016**: Testing phase sequence for new vs existing features

### Changed
- **Sprint 1 Definition of Done**: Now requires smoke tests pass, no 500 errors, user can load app
- **Testing Philosophy**: Added test type matrix and sequence guidance

### Fixed
- "Deployed" being confused with "Working"
- Unit tests blocking deployment when behavior was never verified
- Users discovering 500 errors instead of automated tests catching them

### Key Insight
> **"Deployed" ≠ "Working"**
> 
> Smoke tests fill the gap between deployment and user testing by verifying the deployed app actually works.

---

## [3.6.0] - 2024-12-28

### Added
- **Passkey Authentication Script** (Phase 0.1): Explicit success/failure indicators for each authentication step
- **Responsibility Matrix Clarification** (Phase 1): Emphasizes VS Code AI must execute, not instruct
- **Cloud Migration Script Template** (Appendix C): Executable PowerShell script for automating cloud migration
- **LL-013**: Documents delegation anti-pattern from HarmonyLab migration

### Changed
- **Authentication Flow**: Now includes "AFTER THIS POINT" marker clarifying VS Code AI runs all subsequent gcloud commands
- **Responsibility Matrix**: Added comparison table showing wrong vs. correct VS Code AI behavior

### Fixed
- Ambiguity about who runs gcloud commands after authentication
- VS Code AI interpreting responsibility matrix as "instruct user" rather than "execute directly"

---

## [3.5.0] - 2024-12-27

### Added
- **Meta-project structure**: Organized methodology into dedicated repository
- **Pre-Kickoff Checklist**: Standalone document for questions before kickoff
- **Sprint Checklists**: Separate checklists for Sprint 1 and Sprint 2
- **Deployment Checklist**: GCP verification steps
- **Document Templates**: TEST_PLAN, UI_DESIGN, USER_GUIDE templates
- **ArtForge Example**: Real-world kickoff document for reference

### Changed
- **Repository structure**: From single document to organized folder structure
- **Lessons learned**: Moved to dedicated LESSONS_LEARNED.md with detailed context

---

## [3.4.0] - 2024-12-27

### Added
- **Help Content Strategy** (Section 0.12): External link approach vs embedded docs
- **Definition of Done** (Section 0.16): Explicit completion criteria for both sprints

### Changed
- **Testing Philosophy**: Added distinction between happy path errors and system errors

---

## [3.3.0] - 2024-12-27

### Added
- **Testing Philosophy** (Section 0.11): Guidelines for what to test and error messages
- **Required Documentation** (Section 0.10): TEST_PLAN.md, UI_DESIGN.md, USER_GUIDE.md
- **Document Templates** (Sections 0.13-0.15): Starter templates for required docs

### Changed
- **Responsibility Matrix**: Added TEST_PLAN.md, UI_DESIGN.md, USER_GUIDE.md to Claude deliverables

---

## [3.2.0] - 2024-12-27

### Added
- **Sprint Planning Requirements** (Section 0.9): Two-sprint model defined
  - Sprint 1: Infrastructure & API
  - Sprint 2: Testing, UI/UX, Documentation

### Changed
- **Responsibility Matrix**: Clarified that VS Code AI implements tests in Sprint 2

---

## [3.1.0] - 2024-12-27

### Added
- **GitHub & Deployment Workflow** (Section 0.7): Git init, push, Cloud Run URL retrieval
- **Responsibility Matrix** (Section 0.8): Explicit task assignments

### Fixed
- **LL-007**: Git initialization now in VS Code AI responsibilities
- **LL-001**: Database user creation now in VS Code AI responsibilities

---

## [3.0.0] - 2024-12-27

### Added
- **Development & Testing Environment** (Section 0.5): NO LOCALHOST rule
- **Kickoff Document Variables** (Section 0.6): Standardized placeholders

### Changed
- **Major restructure**: Moved from implicit assumptions to explicit documentation
- **Workflow**: Established cloud-first development pattern

### Fixed
- **LL-006**: Added "No localhost servers" section

---

## [2.0.0] - 2024-12-27

### Added
- **Authentication Flow** (Section 0.3): Exact command order to avoid password prompts
- **SQL Execution Environment** (Section 0.4): Tool selection for GO statements
- **GCP Project Guidance** (Section 0.2): When to use shared vs dedicated projects

### Fixed
- **LL-002**: Authentication order documented
- **LL-003**: Project verification added
- **LL-005**: SQL tool selection documented

---

## [1.0.0] - 2024-12-27

### Added
- **Pre-Kickoff Questions** (Section 0.1): Questions Claude asks before generating kickoff
- **Initial Lessons Learned Log**: First entries from ArtForge development

### Notes
- First formal version of the methodology
- Based on accumulated experience from ArtForge, HarmonyLab planning, Etymython, Super-Flashcards

---

## Version Numbering

- **Major** (X.0.0): Significant restructuring or new phases
- **Minor** (0.X.0): New sections or features added
- **Patch** (0.0.X): Clarifications, typo fixes, minor updates

## Contributing

When adding changes:
1. Add entry under `[Unreleased]` section
2. When releasing, move to versioned section with date
3. Reference Lessons Learned entries (LL-XXX) where applicable
