# Verification Governance

**Purpose**: Establish ownership and access controls for truth sources and verification systems.

> **Key Principle**: The programmer cannot grade their own test. AI agents need external verification.

---

## The Problem

AI agents cannot independently verify their work. They trust their own tests, which may have the same blind spots as their code.

**Example from Etymython**:
- VS Code's roundtrip test: `string_sent == string_received` → TRUE
- Actual database state: Unicode 206 (mojibake) instead of 918 (Greek)
- The test compared corrupted-input to corrupted-output—both had the same bug, so they matched

---

## Truth Source Hierarchy

```
TRUTH SOURCE HIERARCHY:

1. Golden Audit (stored procedures, verification queries) = SINGLE SOURCE OF TRUTH
   - Owned by: Architect (Claude) with Project Lead approval
   - Executed in: SSMS by Project Lead
   - Access for VS Code: READ-ONLY (can execute, cannot modify)
   
2. VS Code's tests = CLAIMS (not proof)
   - Must be verified against Golden Audit
   - Discrepancies = VS Code is wrong by definition
   - Never trust "all tests pass" without external verification

3. Project Lead = Final authority on truth source updates
   - Approves changes to verification logic
   - Runs independent verification
   - Decides when work is complete
```

---

## Access Control Matrix

| Role | Truth Sources | Verification SPs | Application Code | Test Code |
|------|---------------|------------------|------------------|-----------|
| **Project Lead** | Approve changes | Execute in SSMS | Review | Review |
| **Architect (Claude)** | Design, write SQL | Write for PL to execute | Design | Design |
| **VS Code AI** | READ-ONLY | Execute only | Write | Write |

---

## Golden Audit Governance

### Who Can Modify Truth Sources

✅ **Project Lead** (via SSMS): 
- Execute stored procedures written by Architect
- Add new audit requirements
- Modify detection logic after review

✅ **Architect (Claude)**:
- Design audit requirements
- Write SP code for Project Lead to execute
- Analyze audit results
- Recommend threshold changes

❌ **VS Code**:
- READ-ONLY access to audit procedures
- Can EXECUTE verification SPs to check work
- CANNOT modify stored procedures
- CANNOT "fix" audits by changing the truth
- CANNOT adjust thresholds to make tests pass

---

## Why Read-Only for VS Code

If VS Code has write access to truth sources, it may:

| Risk | Example |
|------|---------|
| "Fix" a failing audit | Change detection logic to exclude failing records |
| Modify thresholds | Lower quality bar to make tests pass |
| Add exceptions | Exclude known failures from counts |
| Corrupt verification | Accidentally break the audit system |

**The fundamental problem**: An agent that can modify its own success criteria has no external accountability.

---

## Verification Chain

```
1. VS Code writes code and runs its own tests
   └── VS Code reports: "Tests pass"
   
2. VS Code runs Golden Audit (read-only) to verify
   └── VS Code reports: "Audit results: X violations"
   
3. Project Lead independently runs Golden Audit in SSMS
   └── Project Lead confirms: "Audit results: X violations"
   
4. Results must match
   └── Match: Work may be complete
   └── Mismatch: VS Code's work is wrong by definition
   
5. Project Lead makes final determination
   └── Complete: Move to next task
   └── Not complete: VS Code must fix
```

---

## Implementing Golden Audit

### Standard Audit Stored Procedure Pattern

```sql
CREATE OR ALTER PROCEDURE usp_GoldenAudit
AS
BEGIN
    SET NOCOUNT ON;
    
    -- Audit 1: Data completeness
    SELECT 'COMPLETENESS' as audit_type,
           COUNT(*) as total_records,
           SUM(CASE WHEN required_field IS NULL THEN 1 ELSE 0 END) as violations
    FROM main_table;
    
    -- Audit 2: Data integrity
    SELECT 'INTEGRITY' as audit_type,
           COUNT(*) as total_records,
           SUM(CASE WHEN UNICODE(LEFT(greek_column, 1)) < 900 THEN 1 ELSE 0 END) as violations
    FROM table_with_greek;
    
    -- Audit 3: Referential integrity
    SELECT 'REFERENTIAL' as audit_type,
           COUNT(*) as orphaned_records
    FROM child_table c
    LEFT JOIN parent_table p ON c.parent_id = p.id
    WHERE p.id IS NULL;
    
    -- Summary
    SELECT 'SUMMARY' as audit_type,
           (SELECT COUNT(*) FROM main_table WHERE status = 'complete') as complete,
           (SELECT COUNT(*) FROM main_table WHERE status != 'complete') as incomplete;
END
```

### VS Code Execution (Read-Only)

```python
# VS Code can execute but not modify
def run_golden_audit(connection):
    """Execute Golden Audit - READ ONLY"""
    cursor = connection.cursor()
    cursor.execute("EXEC usp_GoldenAudit")
    
    results = []
    while True:
        rows = cursor.fetchall()
        if rows:
            results.append(rows)
        if not cursor.nextset():
            break
    
    return results
    
# VS Code reports results but cannot change the audit
audit_results = run_golden_audit(conn)
print("Golden Audit Results (read-only verification):")
for result_set in audit_results:
    print(result_set)
```

---

## Discrepancy Resolution

When VS Code's tests pass but Golden Audit fails:

```
DISCREPANCY DETECTED

VS Code claims: "All tests pass"
Golden Audit shows: 15 violations

RESOLUTION:
1. Golden Audit is correct by definition
2. VS Code's tests have a blind spot
3. VS Code must:
   a. Identify why its tests didn't catch the violations
   b. Fix the underlying issue (not the audit)
   c. Re-run Golden Audit to verify
```

**VS Code cannot**:
- Argue the audit is wrong
- Modify the audit to pass
- Request threshold changes to accommodate failures

**VS Code can**:
- Ask Architect to review audit logic (via Project Lead)
- Explain why it believes there's an audit bug
- Request Project Lead verification

---

## Audit Categories

| Category | What It Checks | Owner |
|----------|----------------|-------|
| **Data Completeness** | Required fields populated | Architect designs, PL executes |
| **Data Integrity** | Values within valid ranges | Architect designs, PL executes |
| **Encoding Correctness** | Unicode values correct | Architect designs, PL executes |
| **Referential Integrity** | Foreign keys valid | Architect designs, PL executes |
| **Business Rules** | Domain-specific logic | Architect designs, PL executes |

---

## Project Lead Verification Checklist

Before accepting work as complete:

```
[ ] VS Code reports tests pass
[ ] VS Code has run Golden Audit and reported results
[ ] Project Lead independently runs Golden Audit in SSMS
[ ] Results match VS Code's report
[ ] Zero violations (or acceptable exceptions documented)
[ ] No audit logic was modified during this sprint
```

---

**Template Version**: 3.9  
**Last Updated**: January 2026  
**Methodology**: [coreyprator/project-methodology](https://github.com/coreyprator/project-methodology)
