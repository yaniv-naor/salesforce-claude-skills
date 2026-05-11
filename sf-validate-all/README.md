# SF Validate All

Comprehensive pre-deployment validation skill for Salesforce projects. Acts as a quality gate before deploying code to any Salesforce org.

## What It Does

This skill validates Salesforce code changes by checking:

1. **Prefix Compliance** - Ensures all custom components follow project naming standards
2. **Test Coverage** - Verifies minimum 70% code coverage on changed Apex classes
3. **Syntax Validation** - Confirms all files compile without errors
4. **Deployment Readiness** - Provides clear READY/NOT READY decision

## When to Use

Run this skill before:
- Committing code to git
- Creating a pull request
- Deploying to any org (dev, staging, production)
- Pushing to remote repository

Trigger phrases:
- "validate"
- "check deployment"
- "ready to deploy"
- "run validation"
- "deployment check"
- "is my code ready"

## Key Features

- **Fast Execution** - Only validates changed files (git diff), not entire codebase
- **Actionable Solutions** - Every issue includes specific fix with command
- **Clear Status** - Unambiguous READY/NOT READY deployment decision
- **Integration with sf-prefix-detect** - Intelligent prefix detection for all file types
- **Comprehensive Report** - Structured output with all validation results

## How It Works

1. **Identifies Changed Files** - Uses git diff to find new/modified files
2. **Validates Prefixes** - Checks naming standards using sf-prefix-detect logic
3. **Checks Test Coverage** - Runs tests and verifies ≥70% coverage
4. **Validates Syntax** - Compiles files using sf CLI dry-run
5. **Generates Report** - Provides structured validation report with clear status

## Output Format

The skill produces a structured validation report with:

- Files validated count by type
- Passed checks section (prefix, coverage, syntax)
- Failed checks section with solutions
- Warnings section (non-blocking issues)
- Deployment readiness status (READY or NOT READY)
- Next steps with deployment commands

## Test Results

All 3 test cases passed (100% pass rate):

### Test 1: Clean Validation
- **Scenario:** All validations pass
- **Result:** Shows READY TO DEPLOY with 82% coverage
- **Validation:** ✅ All 5 assertions passed

### Test 2: Prefix Violation
- **Scenario:** New class missing prefix
- **Result:** Blocks deployment, provides git mv command
- **Validation:** ✅ All 5 assertions passed

### Test 3: Low Test Coverage
- **Scenario:** Coverage below 70% (65%)
- **Result:** Blocks deployment, shows gap, suggests improvements
- **Validation:** ✅ All 5 assertions passed

## Integration

Works seamlessly with other Salesforce skills:
- **sf-prefix-detect** - For intelligent prefix validation
- **sf-apex-create** - Suggested when test classes are missing
- **sf-trigger-create** - For trigger validation
- **sf-object-create** - For custom object validation

## Requirements

- Git repository (for change detection)
- Salesforce CLI (sf command)
- Salesforce project structure

## Example Usage

### Scenario 1: Clean Project
```
User: "Validate my changes before I deploy"

Skill Output:
✅ READY TO DEPLOY
- Prefix validation: 2/2 files compliant
- Test coverage: 82% (above 70%)
- Syntax validation: All files compile
- Critical Issues: 0
```

### Scenario 2: Issues Found
```
User: "Check if my code is ready to push"

Skill Output:
❌ NOT READY - FIX ISSUES FIRST
- MyHelper.cls missing PROJ_ prefix
  → git mv MyHelper.cls PROJ_MyHelper.cls
- MyHelper.cls has no test class
  → Create PROJ_MyHelper_Test.cls
- Critical Issues: 2
```

## Performance

- **Execution Time:** 30 seconds - 2 minutes
- **Scope:** Only changed files (not entire codebase)
- **Efficiency:** Parallel validation where possible

## Validation Rules

### Prefix Requirements

| File Type | Prefix Required | Example |
|-----------|----------------|---------|
| Custom Apex Class | ✅ Yes | PROJ_Handler.cls |
| Custom Trigger | ✅ Yes | PROJ_AccountTrigger.trigger |
| Custom Object | ✅ Yes | PROJ_Invoice__c |
| Flow | ✅ Yes | PROJ_DocumentFlow |
| LWC | ✅ Yes (lowercase) | proj_component |
| Standard Object Field | ✅ Yes | PROJ_CustomField__c |
| Custom Object Field | ❌ No | Status__c |

### Test Coverage Rules

- **Minimum:** 70% per class
- **Recommended:** 85% per class
- **Org Minimum:** 75% overall

## Version

**Version:** 2.0.0  
**Author:** Claude Code  
**License:** MIT  
**Category:** salesforce-validation

## Related Documentation

- [SKILL.md](SKILL.md) - Complete skill instructions
- [TEST_CASES.md](TEST_CASES.md) - Detailed test case documentation
- [TEST_RESULTS.md](TEST_RESULTS.md) - Full test results and analysis
