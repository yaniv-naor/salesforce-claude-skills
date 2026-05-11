---
name: sf-scratch-debug
description: Diagnoses and fixes scratch org creation failures. Use when scratch org creation fails, Initialize_scratch_org.py errors, package installation issues, or deployment failures. Trigger on "scratch org error", "SO failed", "scratch creation", "package install error".
license: MIT
metadata:
  version: 1.0.0
  category: salesforce-devops
  tags: [salesforce, scratch-org, debugging, troubleshooting, devops, generic]
---

# SF Scratch Org Debug

Diagnoses and resolves scratch org creation issues, especially failures in the Initialize_scratch_org.py automation script. Works generically across any Salesforce project with any scratch org setup.

**CRITICAL RULES:**
1. NEVER modify Initialize_scratch_org.py - This file must not be changed
2. ALWAYS ask before modifying configuration files - Get user approval first
3. Focus on guidance - Explain problems and solutions, let user decide on changes

## Instructions

### Step 1: Identify the Error

Ask user to provide:

**Required:**
- Full error message from console
- Which step failed (auth, create SO, install package, deploy, etc.)

**Helpful:**
- Contents of configuration files (pre-deploy-actions-scratchorg.json, project-scratch-def.json, etc.)
- Last successful scratch org creation date (to identify recent changes)
- Project structure (where config files are located)

### Step 2: Locate Error in Process

**Common failure points:**

1. **DevHub Authentication** - `sf org login web --set-default-dev-hub`
2. **Scratch Org Creation** - `sf org create scratch`
3. **Package Installation** - `sf package install`
4. **Repository Cloning** - Git clone of dependent repos
5. **Repository Deployment** - Deploy dependent repos
6. **Main Project Deployment** - `sf project deploy start`
7. **Data Import** - `sf data import`

Identify which step failed from error message.

### Step 3: Read Configuration Files

**Check these files for issues:**

1. **Pre-deploy actions configuration** (typically `config/pre-deploy-actions-scratchorg.json` or similar):
   - DevHub username format
   - Package IDs (04t format)
   - Repository URLs
   - Branch names

2. **Scratch org definition** (typically `config/project-scratch-def.json` or similar):
   - Feature flags
   - Settings
   - Admin email
   - Org preferences

3. **`sfdx-project.json`**:
   - Package dependencies
   - Source API version
   - Package directories

**Note:** Configuration file locations may vary by project. Ask user where their config files are located if not in standard locations.

### Step 4: Analyze Error Type

**Common error patterns:**

#### Error Type 1: Authentication Failure
**Symptoms:**
```
Authentication to devhub failed
Error: Invalid client credentials
```

**Causes:**
- Session expired
- Wrong DevHub username in config
- No DevHub access

**Solutions:**
```
1. Re-authenticate:
   sf org login web --set-default-dev-hub

2. Verify DevHub username:
   sf org list

3. Update configuration file with correct DevHub username
   (typically in pre-deploy-actions-scratchorg.json)

4. Clear authentication cache:
   sf org logout --all
   sf org login web --set-default-dev-hub
```

#### Error Type 2: Scratch Org Creation Failure
**Symptoms:**
```
Error creating scratch org
The requested resource does not exist
SignupRequest: Required fields are missing: [field names]
```

**Causes:**
- Invalid features in project-scratch-def.json
- Feature not available in your org
- Invalid org edition
- Org limit reached

**Solutions:**
```
1. Check scratch org limit:
   sf org list --all
   # Delete old orgs:
   sf org delete scratch --target-org [alias] --no-prompt

2. Validate project-scratch-def.json:
   - Remove unsupported features
   - Check feature compatibility
   - Verify edition (Developer/Enterprise)

3. Common problematic features to remove:
   - "MultiCurrency" (if not available)
   - "Communities" (requires specific edition)
   - "Sites" (check license)

4. Try minimal config first:
   {
     "orgName": "Test Org",
     "edition": "Developer",
     "hasSampleData": false
   }
```

#### Error Type 3: Package Installation Failure
**Symptoms:**
```
ERROR: Package install failed
Installation errors:
1) Component: [Name] - Error: [Description]
```

**Causes:**
- Wrong package ID
- Package version incompatible
- Package dependencies not met
- Namespace conflict

**Solutions:**
```
1. Verify package ID format:
   04t[15-character-ID] CORRECT
   04tXXXXXXXXXXXXXXX CORRECT
   0Ho... WRONG (wrong prefix)

2. Check package version compatibility:
   sf package installed list --target-org [alias]

3. Install packages in correct order:
   # Dependencies first!
   # Example: Base packages must be installed before dependent packages

4. Get latest package version:
   sf package version list --packages [PackageId]

5. Manual installation test:
   sf package install --package [04t...] --target-org [alias] --wait 10
```

#### Error Type 4: Repository Clone/Deploy Failure
**Symptoms:**
```
Failed to clone repository
Permission denied (publickey)
Repository not found
```

**Causes:**
- SSH key not configured
- Wrong repository URL
- Branch doesn't exist
- Access denied to repo

**Solutions:**
```
1. Test repository access:
   git clone [repo-url]

2. Fix SSH issues:
   # Generate SSH key if needed:
   ssh-keygen -t ed25519 -C "your.email@example.com"

   # Add to ssh-agent:
   ssh-add ~/.ssh/id_ed25519

   # Add public key to Azure DevOps/GitHub

3. Use HTTPS instead of SSH:
   # In config, change:
   "repository_url": "https://dev.azure.com/org/project/_git/repo"
   # Instead of:
   "repository_url": "git@ssh.dev.azure.com:v3/org/project/repo"

4. Verify branch exists:
   git ls-remote [repo-url]
```

#### Error Type 5: Deployment Failure
**Symptoms:**
```
Deploy failed
Component failures:
ApexClass: [Name] - Error: ...
```

**Causes:**
- Missing dependencies
- API version mismatch
- Invalid metadata
- Syntax errors

**Solutions:**
```
1. Check deployment order:
   # Deploy dependencies before main project
   # Example order:
   #   1. Install required packages
   #   2. Deploy base repository (if exists)
   #   3. Deploy main project

2. Validate metadata locally:
   sf project deploy start --dry-run --target-org [alias]

3. Check API version compatibility:
   # In sfdx-project.json:
   "sourceApiVersion": "64.0"  # Match with org version

4. Deploy specific metadata first:
   sf project deploy start --metadata "CustomObject,ApexClass" --target-org [alias]

5. Ignore known issues:
   sf project deploy start --ignore-conflicts --ignore-warnings
```

### Step 5: Generate Diagnostic Report

Provide structured analysis:

```
Scratch Org Debug Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ISSUE DETECTED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Error Type: [Type]
Failed Step: [Step Name]
Error Message: [Full error]

ROOT CAUSE ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Cause: [Explanation of what went wrong]
Impact: [What this prevents]

RECOMMENDED SOLUTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Solution 1 (Quick Fix):
[Step-by-step instructions]
[Commands to run]

Solution 2 (Proper Fix):
[Long-term solution]
[Configuration changes needed]

COMMANDS TO RUN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# [Description]
[command]

# [Description]
[command]

PREVENTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
To avoid this in future:
- [Preventive measure 1]
- [Preventive measure 2]

VERIFICATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
After applying fixes, verify with:
[Verification commands]
```

### Step 6: Offer to Fix Configuration

**CRITICAL:** Never modify files without explicit user approval.

If config file has issues, offer:

```
I've identified issues in your configuration files.

Issues found:
1. [File]: [Issue] - [Explanation]
2. [File]: [Issue] - [Explanation]

Options:
1. Show you the corrected configuration (you apply manually)
2. Explain each fix in detail
3. I can update the files for you (requires your approval)

Note: I will NOT modify Initialize_scratch_org.py or any Python scripts - only configuration files with your permission.

Which option would you prefer?
```

**Rules for file modifications:**
1. Can modify: Configuration JSON files (pre-deploy-actions-scratchorg.json, project-scratch-def.json, etc.)
2. Cannot modify: Initialize_scratch_org.py or any Python automation scripts (guide user to manual changes if needed)
3. Always ask first: Show proposed changes and get approval before editing
4. Explain impact: Tell user what will change and why

### Step 7: Provide Retry Strategy

After fixes suggested:

```
RETRY STRATEGY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Step-by-step retry process:

1. Apply fixes above

2. Clean up failed attempts:
   sf org delete scratch --target-org [alias] --no-prompt

3. Retry with manual steps first:
   # Step 1: Create org
   sf org create scratch -f config/project-scratch-def.json -a test-org -y 7

   # Step 2: Verify org created
   sf org open --target-org test-org

   # Step 3: Install packages manually
   sf package install --package 04t... --target-org test-org --wait 10

   # Step 4: Deploy
   sf project deploy start --target-org test-org

4. If manual steps succeed, retry Python script:
   python config/Initialize_scratch_org.py
   (or wherever your initialization script is located)

5. Monitor for different errors and report back
```

## Examples

### Example 1: Package Installation Error

**User says:** "Scratch org creation failed at package installation"

**Error:**
```
ERROR: Installation errors:
1) Component: CustomObject__c - Error: Object does not exist
```

**Analysis:**
```
Diagnosis:
Package ID might be incorrect or package version outdated

Root Cause:
Package version may have changed or package ID is wrong

Solutions:
1. Verify package ID in DevHub:
   sf package installed list --target-dev-hub [alias]

2. Update config with correct package ID

3. Install package manually to test:
   sf package install --package [new-id] --target-org [test-org]
```

### Example 2: Feature Not Available

**User says:** "Scratch org creation fails with 'feature not available'"

**Error:**
```
SignupRequest: Feature 'Communities' not available for this organization
```

**Analysis:**
```
Diagnosis:
Communities feature requires specific licensing

Solution:
Remove from project-scratch-def.json:

Before:
{
  "features": ["Communities", "Sites"]
}

After:
{
  "features": []
}

Or request Communities enablement from Salesforce if needed for development.
```

### Example 3: Deployment Conflicts

**User says:** "Initialize script fails at deployment with conflicts"

**Error:**
```
Component conflicts detected
- CustomField: Status__c already exists
```

**Analysis:**
```
Diagnosis:
Field exists from package or previous deployment

Solutions:
1. Quick fix - ignore conflicts:
   Edit Python script or run:
   sf project deploy start --ignore-conflicts

2. Proper fix - check if field from package:
   # If from package, remove from local metadata
   # If local, ensure it's the right version

3. Clean org and retry:
   sf org delete scratch --target-org [alias]
   python config/Initialize_scratch_org.py
```

## Important Notes

### Python Script Flow

The `Initialize_scratch_org.py` script (or similar automation) typically follows this sequence:

1. Check DevHub authentication
2. Authenticate if needed
3. Prompt for SO name and duration
4. Create scratch org
5. Install packages (from config)
6. Clone dependent repositories (from config)
7. Deploy dependent repositories
8. Deploy main project
9. Import data (if configured)

**Any step failure stops the process.**

### Configuration Files Reference

**Pre-deploy actions configuration structure** (typically `pre-deploy-actions-scratchorg.json`):
```json
{
  "devhub_username": "devhub@example.com",
  "package": [
    {
      "name": "PackageName",
      "id": "04tJ7000000cMrSIAU"
    }
  ],
  "repository": [
    {
      "name": "DependentRepo",
      "repository_url": "https://...",
      "branch": "master",
      "folder": "DependentRepo"
    }
  ],
  "data": []
}
```

**Note:** Configuration structure may vary by project. Adapt to the specific project's configuration format.

### Common Gotchas

1. **Package order matters** - Install dependencies before dependents
2. **Branch names** - Ensure branch exists before cloning
3. **SSH vs HTTPS** - HTTPS more reliable for automation
4. **Org limits** - Can't create more than daily/total limit
5. **Features** - Not all features available in all editions

## Troubleshooting Reference

### Quick Diagnostic Commands

```bash
# Check DevHub connection
sf org list

# List active scratch orgs
sf org list --all

# Check scratch org details
sf org display --target-org [alias]

# Verify package installation
sf package installed list --target-org [alias]

# Test deployment
sf project deploy start --dry-run --target-org [alias]

# View deployment status
sf project deploy report --target-org [alias]

# Check org limits
sf limits api display --target-org [alias]
```

### Log Files to Check

```
.sfdx/sfdx.log - SF CLI logs
[repo-folder]/.git/config - Cloned repo info
Python console output - Full error stack trace
```

### When All Else Fails

```
1. Create minimal scratch org manually:
   sf org create scratch -f config/project-scratch-def.json -a minimal-test

2. If that works, add complexity incrementally:
   - Add one feature at a time
   - Install one package at a time
   - Deploy one metadata type at a time

3. Identify exactly which step breaks

4. Focus fix on that specific step
```

## Prevention Best Practices

### Before Running Script

1. **Test configuration**:
   ```bash
   # Validate JSON
   python -m json.tool config/pre-deploy-actions-scratchorg.json
   ```

2. **Verify package IDs** (check in DevHub)

3. **Test repository access**:
   ```bash
   git ls-remote [repo-url]
   ```

4. **Check DevHub limits**:
   ```bash
   sf org list --all
   # Delete old orgs if near limit
   ```

### After Successful Creation

1. **Document working configuration** (commit to repo)
2. **Note package versions** that worked
3. **Save scratch org for reference** (don't delete immediately)
4. **Share working config** with team

### Regular Maintenance

1. **Update package versions** quarterly
2. **Test scratch org creation** before major releases
3. **Keep Python script updated** with latest sf CLI commands
4. **Review and clean old orgs** weekly

## Reference

### SF CLI Scratch Org Commands

```bash
# Create
sf org create scratch -f config/project-scratch-def.json -a [alias] -y [days]

# Delete
sf org delete scratch --target-org [alias] --no-prompt

# List
sf org list --all

# Open
sf org open --target-org [alias]

# Display info
sf org display --target-org [alias]

# Generate password
sf org generate password --target-org [alias]
```

### Common Error Codes

| Error Code | Meaning | Common Fix |
|------------|---------|------------|
| UNKNOWN_EXCEPTION | Generic error | Check logs for details |
| INVALID_FIELD | Field doesn't exist | Check API version |
| REQUIRED_FIELD_MISSING | Missing required field | Add required fields |
| DUPLICATE_VALUE | Value already exists | Check unique constraints |
| CANNOT_INSERT_UPDATE | DML restriction | Check triggers/validation |

## Adaptation Notes

This skill works generically across any Salesforce project. When using:

1. **Ask about project structure** - Config file locations may vary
2. **Identify automation scripts** - May be named differently than Initialize_scratch_org.py
3. **Check configuration format** - JSON structure may differ between projects
4. **Verify CLI tools** - Some projects may use older sfdx commands vs sf commands

Always adapt instructions to the specific project's setup while following the core diagnostic approach.
