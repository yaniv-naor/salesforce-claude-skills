---
name: sf-validate-all
description: Comprehensive pre-deployment validation for Salesforce projects. Checks prefix consistency, test coverage, syntax validation, and deployment readiness. Use before deployment. Trigger on "validate", "check deployment", "ready to deploy", "run validation". Works across any Salesforce project.
license: MIT
metadata:
  version: 2.0.0
  category: salesforce-validation
  tags: [salesforce, validation, deployment, testing, quality-gate, generic]
---

# SF Validate All

Comprehensive pre-deployment validation suite that acts as a quality gate before deploying to any Salesforce org. Checks ONLY modified or new files (via git diff) for prefix compliance, test coverage, syntax errors, and deployment readiness. **Works with any Salesforce project.**

## When to Use

- Before committing code to git
- Before creating a pull request
- Before deployment to any org (dev, staging, production)
- During code review process
- When user says "validate", "check deployment", "ready to deploy", "run validation"
- As part of CI/CD pipeline quality gate

## Core Validation Checks

1. **Prefix Validation** - Ensures all custom components follow naming standards
2. **Test Coverage** - Verifies minimum 70% coverage on changed classes
3. **Syntax Validation** - Compiles all changed files without errors
4. **Deployment Readiness** - Overall go/no-go decision

## Instructions

### Step 1: Identify Changed Files

**IMPORTANT:** Only validate files that are new or modified, not the entire codebase.

**Detect changed files:**
```bash
# Get staged files
git diff --cached --name-only --diff-filter=ACMR

# Get unstaged changes
git diff --name-only --diff-filter=ACMR

# Get untracked files
git ls-files --others --exclude-standard
```

**Filter for Salesforce metadata:**
- Apex classes: `*.cls`
- Triggers: `*.trigger`
- Flows: `*.flow-meta.xml`
- LWC JavaScript: `lwc/**/*.js`
- Custom Objects: `*.object-meta.xml`
- Custom Fields: `fields/*.field-meta.xml`

**Combine all changed files into validation list.**

### Step 2: Prefix Validation

For each changed custom file:

**Use `sf-prefix-detect` skill** to determine if prefix is needed:
- Custom files (Apex, Triggers, LWC, Flows, Objects) → Prefix required
- Standard files (Community controllers, standard objects) → No prefix
- Fields → Depends on parent object (custom vs standard)

**Check file naming** based on detection results.

**Report violations:**
```
PREFIX VIOLATIONS FOUND:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

❌ Missing prefix: MyCustomClass.cls
   💡 Solution: Rename to [PREFIX]_MyCustomClass.cls
   💡 Command: git mv force-app/.../MyCustomClass.cls force-app/.../[PREFIX]_MyCustomClass.cls

❌ Incorrect prefix: WRONG_Handler.cls (project prefix is PROJ_)
   💡 Solution: Rename to PROJ_Handler.cls
   💡 Command: git mv force-app/.../WRONG_Handler.cls force-app/.../PROJ_Handler.cls
```

### Step 3: Test Coverage Validation

For each changed Apex class (excluding test classes):

**1. Find corresponding test class:**
- Pattern 1: `[ClassName]_Test.cls` (preferred)
- Pattern 2: `[ClassName]Test.cls` (alternative)

**2. Run tests** only for changed classes:
```bash
sf apex run test --tests [PREFIX]_ClassName_Test --code-coverage --result-format human
```

**3. Check coverage percentage:**
- Minimum required: ≥70%
- Recommended: ≥85%

**4. Report coverage issues:**
```
TEST COVERAGE ISSUES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️ [PREFIX]_DocumentHandler.cls - Coverage: 68% (below 70%)
   Uncovered lines: 45-52, 78-81
   
   💡 Solution: Add test methods for:
   - handleBeforeInsert() bulk scenario
   - handleAfterUpdate() error handling
   
   💡 Suggested test: testBulkInsertWith200Records()
```

**5. Check test class exists:**
```
❌ [PREFIX]_NewFeature.cls - No test class found
   💡 Solution: Create [PREFIX]_NewFeature_Test.cls
   💡 Command: Use sf-apex-create skill to generate test class
```

### Step 4: Syntax Validation

**Apex Classes & Triggers:**
```bash
# Quick compile check (doesn't deploy)
sf project deploy start --dry-run --ignore-warnings --metadata ApexClass:[PREFIX]_ClassName
```

**Flow XML Validation:**
- Parse XML to ensure well-formed
- Check for common issues:
  - Missing `<status>` tag
  - Invalid API version
  - Malformed decision nodes

**Report syntax errors:**
```
SYNTAX ERRORS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

❌ Syntax Error: [PREFIX]_Handler.cls:42
   Error: Unexpected token 'if'
   
   💡 Solution: Check for missing closing braces on line 41
   💡 Review: Ensure proper bracket matching
```

### Step 5: Generate Validation Report

Provide comprehensive validation report with clear READY/NOT READY decision:

```
🔍 SALESFORCE VALIDATION REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Generated: [YYYY-MM-DD HH:MM:SS]
Project: [Project Name]
Branch: [Branch Name]

📊 FILES VALIDATED: [X] files changed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - Apex Classes: [N]
   - Triggers: [N]
   - Flows: [N]
   - LWC: [N]
   - Objects: [N]
   - Fields: [N]

✅ PASSED CHECKS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Prefix validation: [X/X] files compliant
✓ Test coverage: [X/X] classes above 70%
✓ Syntax validation: All files compile successfully

❌ FAILED CHECKS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[List of critical failures with solutions - or "None" if all passed]

⚠️ WARNINGS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[List of warnings (non-blocking issues) - or "None" if all clean]

📝 DEPLOYMENT READINESS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: [✅ READY TO DEPLOY | ❌ NOT READY - FIX ISSUES FIRST]

Critical Issues: [N]
Warnings: [N]

[If NOT READY:]
🚫 Deployment BLOCKED - fix critical issues above first

[If READY:]
✅ All validations passed! Safe to deploy.

📝 NEXT STEPS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Review warnings (if any)
2. Deploy: sf project deploy start -d force-app/main/default
3. Run full test suite: sf apex run test --test-level RunLocalTests
4. Monitor deployment: sf project deploy report
```

## Validation Checklist

Use this internal checklist to ensure all validations are performed:

### Prefix Validation
- [ ] All custom Apex classes have project prefix
- [ ] All triggers have project prefix
- [ ] All custom objects have project prefix
- [ ] All flows have project prefix
- [ ] All LWC folders have lowercase project prefix
- [ ] Fields on standard objects have project prefix
- [ ] Fields on custom objects do NOT have prefix
- [ ] Standard Salesforce files excluded (Community controllers, etc.)

### Test Coverage Validation
- [ ] Every Apex class has corresponding test class
- [ ] Test class naming follows convention (`_Test` or `Test` suffix)
- [ ] All test classes have ≥70% coverage
- [ ] Test classes compile without errors

### Syntax Validation
- [ ] All Apex classes compile successfully
- [ ] All triggers compile successfully
- [ ] All Flow XML files are well-formed
- [ ] No syntax errors in LWC JavaScript
- [ ] No missing dependencies

### Deployment Readiness
- [ ] No critical issues found
- [ ] All required tests exist and pass
- [ ] All files follow naming standards
- [ ] Overall status determined (READY or NOT READY)

## Examples

### Example 1: Clean Validation (All Passed)

**User:** "Validate my changes before I deploy"

**Changed files:**
- `PROJ_DocumentHandler.cls` (modified)
- `PROJ_DocumentHandler_Test.cls` (modified)

**Validation results:**
1. Git diff finds 2 changed files ✅
2. Prefix check: Both have `PROJ_` prefix ✅
3. Test coverage: Run test → 82% coverage ✅
4. Syntax: Dry-run deploy → Success ✅

**Report:**
```
🔍 SALESFORCE VALIDATION REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Generated: 2026-05-05 10:30:15
Project: CRM Enforcement
Branch: feature/document-handler

📊 FILES VALIDATED: 2 files changed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - Apex Classes: 2 (1 main + 1 test)

✅ PASSED CHECKS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Prefix validation: 2/2 files compliant
✓ Test coverage: 1/1 classes above 70% (82%)
✓ Syntax validation: All files compile successfully

❌ FAILED CHECKS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
None

⚠️ WARNINGS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
None

📝 DEPLOYMENT READINESS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: ✅ READY TO DEPLOY

Critical Issues: 0
Warnings: 0

✅ All validations passed! Safe to deploy.

📝 NEXT STEPS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Deploy: sf project deploy start -d force-app/main/default
2. Run full test suite: sf apex run test --test-level RunLocalTests
3. Monitor deployment: sf project deploy report
```

### Example 2: Validation with Issues

**User:** "Check if my code is ready to push"

**Changed files:**
- `MyHelper.cls` (new - missing prefix)
- `PROJ_BatchJob.cls` (modified)
- `PROJ_BatchJobTest.cls` (modified)

**Validation results:**
1. Git diff finds 3 files
2. Prefix check:
   - ❌ `MyHelper.cls` missing prefix
   - ✅ `PROJ_BatchJob.cls` has prefix
   - ✅ `PROJ_BatchJobTest.cls` has prefix
3. Test coverage:
   - ❌ `MyHelper.cls` has no test class
   - ⚠️ `PROJ_BatchJob.cls` → 65% coverage (below 70%)

**Report:**
```
🔍 SALESFORCE VALIDATION REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Generated: 2026-05-05 10:45:22
Project: CRM Enforcement
Branch: feature/batch-job

📊 FILES VALIDATED: 3 files changed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - Apex Classes: 3 (2 main + 1 test)

✅ PASSED CHECKS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Prefix validation: 2/3 files compliant
✓ Syntax validation: All files compile successfully

❌ FAILED CHECKS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PREFIX VIOLATIONS:
1. MyHelper.cls - Missing prefix PROJ_
   💡 Solution: Rename to PROJ_MyHelper.cls
   💡 Command: git mv force-app/.../MyHelper.cls force-app/.../PROJ_MyHelper.cls

TEST COVERAGE ISSUES:
2. MyHelper.cls - No test class found
   💡 Solution: Create PROJ_MyHelper_Test.cls
   💡 Command: Use /sf-apex-create skill to generate test class

3. PROJ_BatchJob.cls - Coverage 65% (below 70%)
   Uncovered lines: 45-52, 78-81
   💡 Solution: Add tests for execute() method bulk scenario
   💡 Current: 65% | Required: 70% | Gap: 5%

⚠️ WARNINGS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
None

📝 DEPLOYMENT READINESS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: ❌ NOT READY - FIX ISSUES FIRST

Critical Issues: 3
Warnings: 0

🚫 Deployment BLOCKED - fix critical issues above first

📝 REQUIRED ACTIONS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Rename MyHelper.cls to PROJ_MyHelper.cls
2. Create test class PROJ_MyHelper_Test.cls
3. Improve PROJ_BatchJob_Test.cls coverage to at least 70%
4. Re-run validation after fixes
```

### Example 3: Multiple File Types

**User:** "Validate all my changes"

**Changed files:**
- `PROJ_Invoice__c.object-meta.xml` (new object)
- `PROJ_Invoice__c/fields/Status__c.field-meta.xml` (new field)
- `PROJ_InvoiceHandler.cls` (new)
- `PROJ_InvoiceHandler_Test.cls` (new)
- `proj_invoiceForm` (new LWC)

**Validation results:**
1. Prefix check: All files compliant ✅
2. Test coverage: 88% ✅
3. Syntax: All compile ✅

**Report:**
```
🔍 SALESFORCE VALIDATION REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Generated: 2026-05-05 11:15:33
Project: CRM Enforcement
Branch: feature/invoice-module

📊 FILES VALIDATED: 5 files changed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   - Apex Classes: 2 (1 main + 1 test)
   - Custom Objects: 1
   - Custom Fields: 1
   - LWC: 1

✅ PASSED CHECKS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Prefix validation: 5/5 files compliant
✓ Test coverage: 1/1 classes above 70% (88%)
✓ Syntax validation: All files compile successfully
✓ Naming conventions: All follow standards

❌ FAILED CHECKS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
None

⚠️ WARNINGS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
None

📝 DEPLOYMENT READINESS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: ✅ READY TO DEPLOY

Critical Issues: 0
Warnings: 0

✅ All validations passed! Safe to deploy.

📝 NEXT STEPS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Deploy: sf project deploy start -d force-app/main/default
2. Run full test suite: sf apex run test --test-level RunLocalTests
3. Monitor deployment: sf project deploy report
```

## Important Notes

### What This Skill Does

- ✅ Validates only changed files (fast)
- ✅ Checks prefix compliance using sf-prefix-detect
- ✅ Verifies test coverage for changed classes (70% minimum)
- ✅ Validates syntax (compile check)
- ✅ Provides clear READY/NOT READY decision
- ✅ Generates comprehensive report with actionable fixes
- ✅ Acts as quality gate before deployment

### What This Skill Does NOT Do

- ❌ Does NOT validate unchanged files
- ❌ Does NOT perform deep code analysis (use PMD/SonarQube)
- ❌ Does NOT run all org tests (only changed classes)
- ❌ Does NOT fix issues automatically
- ❌ Does NOT deploy code (validation only)
- ❌ Does NOT check permission sets/profiles

### When to Use This Skill

**Always use before:**
- Committing code to git
- Creating pull request
- Deploying to any org
- Pushing to remote repository

**Also use when user says:**
- "Validate"
- "Check deployment"
- "Ready to deploy"
- "Run validation"
- "Deployment check"
- "Is my code ready"
- "Pre-deployment check"

### Integration with Git Workflow

Typical workflow:

```bash
# 1. Make changes to Apex/Flows/LWC
[Make changes...]

# 2. Stage changes
git add .

# 3. Run validation
/sf-validate-all

# 4. Fix any issues if NOT READY
[Fix issues if any...]

# 5. Re-run validation
/sf-validate-all

# 6. Commit when READY
git commit -m "feat: add invoice processing"

# 7. Deploy
sf project deploy start -d force-app/main/default
```

## Validation Rules

### Prefix Rules (By File Type)

| File Type | Prefix Required | Example |
|-----------|----------------|---------|
| Custom Apex Class | ✅ Yes | `PROJ_Handler.cls` |
| Custom Trigger | ✅ Yes | `PROJ_AccountTrigger.trigger` |
| Custom Object | ✅ Yes | `PROJ_Invoice__c.object-meta.xml` |
| Custom Field (Standard Object) | ✅ Yes | `PROJ_CustomField__c` on `Account` |
| Custom Field (Custom Object) | ❌ No | `Status__c` on `PROJ_Invoice__c` |
| Flow | ✅ Yes | `PROJ_DocumentFlow.flow-meta.xml` |
| LWC | ✅ Yes (lowercase) | `proj_component/` |
| Community Controller | ❌ No | `CommunitiesLandingController.cls` |
| Standard Object | ❌ No | `Account.object-meta.xml` |

### Test Coverage Rules

| Metric | Minimum | Recommended | Optimal |
|--------|---------|-------------|---------|
| Class Coverage | 70% | 85% | 95%+ |
| Org Coverage | 75% | 85% | 90%+ |

**Coverage requirements:**
- Every Apex class (except test classes) must have test class
- Test class must achieve ≥70% coverage
- Bulk tests with 200+ records recommended
- Negative test scenarios recommended

## Performance Optimization

This skill is optimized for speed:

**1. Git diff first:**
- Only check changed files (not entire codebase)
- Dramatically reduces validation time

**2. Targeted tests:**
- Only run tests for changed classes
- Skip full org test suite

**3. Quick syntax:**
- Use dry-run deploys (no actual deployment)
- Fast feedback loop

**Expected execution time:** 30 seconds - 2 minutes (depending on number of changed files)

## Troubleshooting

### Issue: Git not initialized

**Error:** `fatal: not a git repository`

**Solution:**
- This skill requires git to identify changed files
- Initialize git: `git init`
- Or manually specify files to validate

### Issue: Test coverage inconsistent

**Cause:** Tests not deployed to target org yet

**Solution:**
1. Deploy test classes first
2. Run tests: `sf apex run test --test-level RunLocalTests`
3. Then run validation

### Issue: False positive on prefix

**Cause:** Standard Salesforce file flagged incorrectly

**Solution:**
1. Verify file is actually standard (not custom)
2. Use sf-prefix-detect skill for accurate detection
3. Check file name pattern (Communities*, Site*, etc.)

## Quick Commands Reference

```bash
# Find changed files
git diff --name-only
git status --short

# Check single class syntax
sf project deploy start --dry-run --metadata ApexClass:[PREFIX]_ClassName

# Run specific test with coverage
sf apex run test --tests [PREFIX]_ClassName_Test --code-coverage --result-format human

# Validate full deployment (dry-run)
sf project deploy start --dry-run --test-level RunLocalTests

# Deploy validated changes
sf project deploy start -d force-app/main/default
```

---

**Generated by Claude Code**
**Last Updated:** 2026-05-05
**Version:** 2.0.0 (Generic/Multi-Project Support)
