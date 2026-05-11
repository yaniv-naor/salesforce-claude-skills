# SF Scratch Org Debug Skill

A diagnostic and troubleshooting skill for Salesforce scratch org creation failures, especially when using automation scripts like `Initialize_scratch_org.py`.

## When to Use This Skill

Use this skill when you encounter:

- Scratch org creation failures
- Initialize_scratch_org.py errors
- Package installation issues during scratch org setup
- Deployment failures to scratch orgs
- Repository cloning/deployment errors
- Authentication issues with DevHub

## Trigger Phrases

The skill activates when you mention:
- "scratch org error"
- "SO failed" / "SO creation failed"
- "scratch org creation"
- "package install error"
- "Initialize_scratch_org.py failed"

## What This Skill Does

The skill follows a structured diagnostic process:

1. **Identifies the error** - Asks for error details and context
2. **Locates failure point** - Determines which step in the process failed
3. **Reads configurations** - Examines relevant config files
4. **Analyzes error type** - Categorizes into 5 common error patterns
5. **Generates diagnostic report** - Provides structured analysis with root cause
6. **Offers configuration fixes** - Suggests corrections (with your approval)
7. **Provides retry strategy** - Gives step-by-step recovery instructions

## What This Skill Won't Do

- Modify your Python automation scripts without permission
- Make configuration changes without explicit approval
- Automatically fix issues without explaining them first

## Key Features

### 1. Generic Approach
Works across any Salesforce project structure. Adapts to your specific:
- Configuration file locations
- Automation script names
- Package and repository setup

### 2. 5 Error Types Covered

1. **Authentication Failure** - DevHub login issues
2. **Scratch Org Creation Failure** - Invalid features, org limits
3. **Package Installation Failure** - Wrong IDs, dependencies
4. **Repository Clone/Deploy Failure** - Git access issues
5. **Deployment Failure** - Metadata conflicts, dependencies

### 3. Structured Output

Every diagnosis includes:
- Issue description
- Root cause analysis
- Recommended solutions (quick fix + proper fix)
- Commands to run
- Prevention tips
- Verification steps

### 4. Safety First

- NEVER modifies Initialize_scratch_org.py or other Python scripts
- ALWAYS asks before modifying configuration files
- Explains what changes will do before making them
- Focuses on guidance rather than automatic fixes

## Example Usage

### Example 1: Basic Error Diagnosis

**You:** "My scratch org creation failed with a package installation error"

**Skill:** 
- Asks for the full error message
- Asks which step failed
- Requests relevant configuration files
- Analyzes the error
- Provides structured diagnostic report
- Offers solutions with commands

### Example 2: Configuration Fix

**You:** "Here's my error: [error details]"

**Skill:**
- Identifies the issue in your config file
- Shows you the problematic configuration
- Explains what needs to change
- Asks: "Would you like me to fix this for you, or would you prefer to do it manually?"
- Waits for your approval before making any changes

### Example 3: Quick Diagnostic

**You:** "Scratch org fails at deployment with conflicts"

**Skill:**
- Identifies deployment conflict issue
- Asks if packages were installed
- Analyzes that fields might exist from packages
- Provides quick fix (--ignore-conflicts) and proper fix (remove duplicates)
- Shows how to verify which metadata comes from packages

## Configuration Files

The skill works with common Salesforce configuration files:

- `config/pre-deploy-actions-scratchorg.json` - Package and repo configuration
- `config/project-scratch-def.json` - Scratch org definition
- `sfdx-project.json` - Project configuration
- Any custom configuration files your project uses

**Note:** File locations may vary. The skill will ask about your project structure if needed.

## Common Solutions Provided

### Authentication Issues
- Re-authentication commands
- DevHub verification
- Config file corrections

### Package Problems
- Package ID verification
- Manual installation testing
- Dependency order fixes

### Feature Issues
- Unsupported feature identification
- Minimal config approach
- Edition compatibility checks

### Deployment Conflicts
- Conflict resolution strategies
- Metadata source identification
- Deployment order optimization

### Repository Access
- SSH key setup
- HTTPS alternative
- Access verification

## Best Practices

### Before Using the Skill
1. Have your error message ready (full text)
2. Know which step failed
3. Have access to your configuration files
4. Note when the last successful creation was

### During Diagnosis
1. Provide complete error messages
2. Share relevant configuration when asked
3. Approve or reject suggested changes explicitly
4. Ask questions if solutions aren't clear

### After Resolution
1. Document what worked for your team
2. Commit working configurations to your repo
3. Test the fix thoroughly
4. Share insights with your team

## Troubleshooting the Skill

If the skill doesn't provide helpful answers:

1. **Provide more context**:
   - Full error message
   - Configuration file contents
   - Recent changes to your setup

2. **Specify your project structure**:
   - Where your config files are
   - What your automation script is called
   - Any custom setup you have

3. **Ask specific questions**:
   - "What does this error code mean?"
   - "How do I verify my package ID?"
   - "What's the correct deployment order?"

## Technical Details

- **Author**: Claude Code
- **Version**: 1.0.0
- **Category**: salesforce-devops
- **License**: MIT
- **Tags**: salesforce, scratch-org, debugging, troubleshooting, devops, generic

## Contributing

If you find issues or have suggestions for improving this skill:

1. Test the skill with real scenarios
2. Document what worked and what didn't
3. Suggest additional error types to cover
4. Provide feedback on clarity of instructions

## Quick Reference

### Diagnostic Commands
```bash
# Check DevHub connection
sf org list

# List all scratch orgs
sf org list --all

# Check org details
sf org display --target-org [alias]

# Verify packages
sf package installed list --target-org [alias]

# Test deployment
sf project deploy start --dry-run --target-org [alias]
```

### Common Fixes
```bash
# Re-authenticate
sf org login web --set-default-dev-hub

# Delete old org
sf org delete scratch --target-org [alias] --no-prompt

# Manual package install
sf package install --package [04t...] --target-org [alias] --wait 10

# Deploy with conflict handling
sf project deploy start --ignore-conflicts --target-org [alias]
```

## Related Skills

- **sf-deploy**: For general Salesforce deployment issues
- **sf-package**: For package development and versioning
- **sf-org**: For org management and configuration

## Support

For issues specific to:
- **Skill functionality**: Check the test cases in tests.md
- **Salesforce CLI**: See official Salesforce CLI documentation
- **Project-specific setup**: Consult your project documentation

## Version History

### 1.0.0 (Current)
- Initial release
- Support for 5 common error types
- Generic project structure support
- Structured diagnostic reports
- Safety rules for file modifications
