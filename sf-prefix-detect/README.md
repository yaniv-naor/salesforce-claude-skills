# sf-prefix-detect

> Intelligent prefix detection and validation for Salesforce components across any project

## Overview

The **sf-prefix-detect** skill automatically detects your Salesforce project's naming prefix and determines whether specific components require that prefix. It works across any Salesforce project structure, regardless of naming conventions or folder organization.

### Key Features

- **Automatic Prefix Detection** - Extracts prefix from folder name or scans existing files
- **8 Comprehensive Rules** - Covers all Salesforce component types (Apex, Triggers, LWC, Flows, Objects, Fields)
- **Project-Agnostic** - Works with any Salesforce project structure
- **Clear Decision Logic** - Explains which rule applies and why
- **Edge Case Handling** - Handles ambiguous names, multiple prefixes, and managed packages
- **Standard File Recognition** - Identifies Community/Sites controllers that should never be prefixed

## When to Use

Use this skill whenever you need to:

- Create a new Salesforce component
- Determine if a component needs a prefix
- Validate naming conventions
- Review code for proper prefixing
- Onboard new team members to project standards

### Triggering Phrases

The skill activates when users ask:
- "What prefix should I use?"
- "Does this need a prefix?"
- "Should I add a prefix to [component]?"
- "What's the project prefix?"

### Called By Other Skills

This skill is commonly invoked by:
- `sf-apex-create`
- `sf-trigger-create`
- `sf-lwc-create`
- `sf-object-create`
- `sf-field-create`
- `sf-validate-all`

## Quick Start

### Example 1: Detect Project Prefix

**User:** "What prefix should I use?"

**Skill Output:**
```
📊 PREFIX DETECTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Working Directory: p:\CrmXyz
Detection Method: Folder name pattern matching

Pattern Matched: Crm[ProjectName]
Extracted Value: Enforcement
Transformation: XYZ_

Detected Project Prefix: XYZ_
LWC Prefix (lowercase): enforcement_
```

### Example 2: Check if Custom Apex Needs Prefix

**User:** "Should DocumentHandler.cls have a prefix?"

**Skill Output:**
```
📊 PREFIX ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Project: CrmXyz
Project Prefix: XYZ_
Component: DocumentHandler.cls
Type: Custom Apex Class

DECISION: ✅ PREFIX REQUIRED

Reasoning:
- Custom Apex class created by user
- All custom Apex code requires project prefix

Rule Applied: RULE 1 - Custom Apex Classes & Triggers

Suggested Name: XYZ_DocumentHandler.cls
```

### Example 3: Check Community Controller

**User:** "Does CommunitiesLandingController need a prefix?"

**Skill Output:**
```
📊 PREFIX ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Project: CrmXyz
Project Prefix: XYZ_
Component: CommunitiesLandingController.cls
Type: Standard Salesforce Community Controller

DECISION: ❌ NO PREFIX NEEDED

Reasoning:
- Standard Salesforce file - starts with "Communities"
- Adding a prefix would break Salesforce functionality

Rule Applied: RULE 2 - Standard Salesforce Files

Suggested Name: CommunitiesLandingController.cls (keep as-is)
```

## The 8 Prefix Rules

| Rule | Component Type | Prefix Required? | Example |
|------|----------------|------------------|---------|
| 1 | Custom Apex/Triggers | ✅ Yes | `PREFIX_Handler.cls` |
| 2 | Standard Salesforce Files | ❌ No | `CommunitiesLandingController.cls` |
| 3 | Standard Objects | ❌ No | `Account.object-meta.xml` |
| 4 | Custom Objects | ✅ Yes | `PREFIX_MyObject__c` |
| 5 | Fields on Custom Objects | ❌ No | `Status__c` (object already prefixed) |
| 6 | Fields on Standard Objects | ✅ Yes | `PREFIX_CustomField__c` |
| 7 | Lightning Web Components | ✅ Yes (lowercase) | `prefix_component/` |
| 8 | Flows | ✅ Yes | `PREFIX_MyFlow.flow-meta.xml` |

## Detection Methods

The skill uses three detection methods in order:

### Method 1: Folder Name Pattern (Primary)

Extracts prefix from project folder name:
- `CrmAbc` → `ABC_`
- `CrmXyz` → `XYZ_`
- `MyProject` → `MYPROJECT_`

### Method 2: File Scanning (Fallback)

Scans existing `.cls` files for common prefix patterns:
- Finds: `ABC_Handler.cls`, `ABC_Util.cls`
- Detects: `ABC_` as project prefix

### Method 3: User Input (Last Resort)

Asks user directly if both methods fail:
```
"I couldn't detect a project prefix. What prefix should be used?
Common formats: ABC_, XYZ_, DEF_, MYPROJ_"
```

## Installation

1. Copy the skill to your project:
   ```bash
   cp -r sf-prefix-detect .claude/skills/
   ```

2. Verify installation:
   ```bash
   ls .claude/skills/sf-prefix-detect/
   # Should show: SKILL.md, README.md, TEST_CASES.md, TEST_RESULTS.md
   ```

3. The skill is now available for use in Claude Code

## Testing

The skill includes comprehensive test coverage:

- **Test Case 1:** Prefix detection from folder name
- **Test Case 2:** Custom Apex class validation
- **Test Case 3:** Community controller (standard file) validation
- **Test Case 4:** Custom field on standard object
- **Test Case 5:** Custom field on custom object

**Test Results:** 5/5 passed (100% success rate)

See [TEST_CASES.md](./TEST_CASES.md) for detailed test scenarios and [TEST_RESULTS.md](./TEST_RESULTS.md) for results.

## Configuration

No configuration required. The skill works out-of-the-box with any Salesforce project.

### Optional: Custom Prefix Rules

If your project has custom naming conventions, document them in `.claude/CLAUDE.md`:

```markdown
## Custom Naming Conventions
- Exception: LoginHandler uses no prefix (legacy code)
- Exception: All batch jobs use BATCH_ prefix instead of project prefix
```

The skill respects user decisions and can learn project-specific patterns.

## File Structure

```
sf-prefix-detect/
├── SKILL.md           # Main skill instructions (660 lines)
├── README.md          # This file (user documentation)
├── TEST_CASES.md      # Test case definitions
└── TEST_RESULTS.md    # Test execution results
```

## Troubleshooting

### Issue: Prefix not detected

**Cause:** Folder name doesn't follow known patterns

**Solution:**
1. Run file scan: Look for `*_*.cls` patterns
2. Check CLAUDE.md for documented prefix
3. Specify prefix manually when prompted

### Issue: Conflicting recommendations

**Cause:** Project-specific conventions differ from standard rules

**Solution:**
1. Follow your project's conventions
2. Document exceptions in CLAUDE.md
3. The skill will respect your decision

### Issue: Multiple prefixes detected

**Cause:** Managed packages or legacy code

**Solution:**
- Identify project prefix (most common in custom files)
- Identify package prefixes (from installed packages)
- Keep package files with original prefix

## Best Practices

1. **Always check before creating** - Use skill before creating any new component
2. **Document exceptions** - Add project-specific rules to CLAUDE.md
3. **Stay consistent** - Follow the skill's recommendations for consistency
4. **Review legacy code** - Use skill to audit existing files for proper prefixing
5. **Onboard with skill** - Train new team members using the skill's clear explanations

## Technical Details

- **Version:** 2.0.0
- **Category:** salesforce-validation
- **License:** MIT
- **Author:** Claude Code
- **Lines of Code:** ~660 lines
- **Last Updated:** 2026-05-05

## Related Skills

- **sf-apex-create** - Creates Apex classes with proper prefix
- **sf-trigger-create** - Creates triggers with proper prefix
- **sf-lwc-create** - Creates LWC components with lowercase prefix
- **sf-object-create** - Creates custom objects with proper prefix
- **sf-field-create** - Creates fields with context-aware prefix logic
- **sf-validate-all** - Validates all Salesforce components in project

## Contributing

To improve this skill:

1. Test edge cases and document findings
2. Add new rules for additional component types
3. Submit improvements via project standards
4. Update test cases as rules evolve

## Support

For questions or issues:

1. Check [SKILL.md](./SKILL.md) for detailed documentation
2. Review [TEST_CASES.md](./TEST_CASES.md) for examples
3. Consult project's CLAUDE.md for project-specific guidance
4. Ask Claude Code to invoke the skill with your specific scenario

## License

MIT License - Free to use and modify for your Salesforce projects.

---

**Generated by Claude Code**  
**Part of the Claude Code Salesforce Skill Suite**
