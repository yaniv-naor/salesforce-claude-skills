---
name: sf-prefix-detect
description: Detects and validates project prefix for Salesforce components in any project. Handles standard objects, community files, custom fields, and edge cases. Determines when prefix is required vs optional. Use when creating any Salesforce component, when prefix is unclear, or when user asks "what prefix", "need prefix", "should I add prefix". Works across any Salesforce project structure.
license: MIT
metadata:
  version: 2.0.0
  category: salesforce-validation
  tags: [salesforce, naming, prefix, standards, validation, generic]
---

# SF Prefix Detect

Intelligent prefix detection and validation for Salesforce components. Determines project prefix and advises whether prefix is needed for specific file types. **Works with any Salesforce project**, regardless of naming conventions.

## Instructions

### Step 1: Detect Project Prefix

**Method 1: From folder name (most common)**

1. Get the current working directory name (the root project folder)
2. Apply transformation patterns:
   ```
   Pattern 1: Crm[ProjectName] → Extract [ProjectName] → Uppercase
   Examples:
   - CrmAbc → "Rtn" → "ABC_"
   - CrmXyz → "Enforcement" → "XYZ_"
   - CrmDef → "Gcr" → "DEF_"
   
   Pattern 2: [ProjectName] → Uppercase
   Examples:
   - MyProject → "MYPROJECT_"
   - Salesforce → "SALESFORCE_"
   ```

3. Validate prefix format:
   - Length: 2-10 characters
   - Format: Uppercase letters and/or numbers
   - Must end with underscore (`_`)

**Method 2: Scan existing files**

If folder name is unclear or doesn't follow conventions:

1. Search for custom Apex classes (`.cls` files)
2. Look for pattern: `[A-Z0-9]+_*.cls`
3. Extract the most common prefix
4. Example findings:
   - Files: `ABC_Handler.cls`, `ABC_Util.cls`, `ABC_Test.cls`
   - Detected prefix: `ABC_`

**Method 3: Ask user**

If both methods fail:
```
"I couldn't detect a project prefix. What prefix should be used for this project?
Common formats: ABC_, ABC_, DEF_, MYPROJ_"
```

### Step 2: Determine Component Type

Identify what the user is creating or modifying:

**Component types:**
1. **Custom Apex Class** - User-created `.cls` file
2. **Custom Trigger** - User-created `.trigger` file
3. **Standard Community/Site File** - Salesforce-provided controllers
4. **Lightning Web Component** - LWC folder structure
5. **Flow** - `.flow-meta.xml` file
6. **Custom Object** - `*__c.object-meta.xml`
7. **Custom Field on Custom Object** - Field ending with `__c` on object ending with `__c`
8. **Custom Field on Standard Object** - Field ending with `__c` on standard object
9. **Standard Object** - Objects without `__c` suffix

### Step 3: Apply Prefix Rules

**RULE 1: Custom Apex Classes & Triggers → PREFIX REQUIRED**

All user-created Apex code needs the project prefix.

```
✅ Examples:
- [PREFIX]_DocumentHandler.cls
- [PREFIX]_AccountTrigger.trigger
- [PREFIX]_StringUtils.cls
- [PREFIX]_BatchJob.cls
```

**RULE 2: Standard Salesforce Files → NO PREFIX**

Files provided by Salesforce (Community, Sites, Auth) keep their original names.

```
❌ NO PREFIX:
- CommunitiesLandingController.cls
- CommunitiesLoginController.cls
- SiteLoginController.cls
- ForgotPasswordController.cls
- MyProfilePageController.cls
- ChangePasswordController.cls
- MicrobatchSelfRegController.cls
```

**Detection pattern:**
- Starts with: `Communities*`, `Site*`, `Forgot*`, `Lightning*`, `MyProfile*`, `ChangePassword*`, `Microbatch*`

**RULE 3: Standard Salesforce Objects → NO PREFIX**

Standard objects (provided by Salesforce) never get prefixes.

```
❌ NO PREFIX:
- Account.object-meta.xml
- Contact.object-meta.xml
- Opportunity.object-meta.xml
- Case.object-meta.xml
- User.object-meta.xml
```

**How to identify:**
- Object name does NOT end with `__c`
- Common standard objects: Account, Contact, Lead, Opportunity, Case, User, etc.

**RULE 4: Custom Objects → PREFIX REQUIRED**

Objects created by users need the project prefix.

```
✅ Examples:
- [PREFIX]_Chapter__c
- [PREFIX]_Document__c
- [PREFIX]_CustomObject__c
```

**How to identify:**
- Filename ends with `__c.object-meta.xml`
- Not a field (no parent object context)

**RULE 5: Custom Fields on Custom Objects → NO PREFIX**

When a custom object already has a prefix, its fields don't need one.

```
❌ NO PREFIX:
- Field Status__c on [PREFIX]_Document__c
- Field Priority__c on [PREFIX]_Chapter__c
- Field Value__c on [PREFIX]_CustomObject__c

Reasoning: The object already has the prefix, fields inherit the namespace.
```

**RULE 6: Custom Fields on Standard Objects → PREFIX REQUIRED**

When adding custom fields to standard objects, use the prefix.

```
✅ Examples:
- [PREFIX]_CustomField__c on Account
- [PREFIX]_ExtraInfo__c on Contact
- [PREFIX]_Notes__c on Opportunity

Reasoning: Standard objects have no prefix, so custom fields need identification.
```

**RULE 7: Lightning Web Components → PREFIX REQUIRED (lowercase)**

LWC folders use lowercase prefix.

```
✅ Examples:
- [prefix]_documentViewer/
- [prefix]_header/
- [prefix]_customComponent/

Format: [prefix]_[componentName] (camelCase after prefix)
Example: If project prefix is ABC_, LWC prefix is rtn_
```

**RULE 8: Flows → PREFIX REQUIRED**

All custom Flows need the project prefix.

```
✅ Examples:
- [PREFIX]_AddTagFlow.flow-meta.xml
- [PREFIX]_DocumentAfterInsertFlow.flow-meta.xml
- [PREFIX]_ValidationFlow.flow-meta.xml
```

### Step 4: Handle Edge Cases

**Edge Case 1: Ambiguous File Name**

If the file name could be standard OR custom, ask for clarification.

```
Example: "LoginController"
Could be:
1. SiteLoginController (standard) → No prefix
2. Custom login handler → [PREFIX]_LoginController

Action: "Is this a custom controller you're creating, or a standard Salesforce community controller?"
```

**Edge Case 2: Multiple Prefixes Detected**

If scanning finds different prefixes in the same project:

```
Found:
- ABC_DocumentHandler.cls
- ABC_LogHandler.cls
- MyCustomClass.cls (no prefix)

Action: "Detected multiple prefixes (ABC_, ABC_). Which prefix should be used for new files?"

Possible causes:
- Package dependencies (keep their original prefixes)
- Migration from another project
- Inconsistent naming (needs cleanup)
```

**Edge Case 3: Managed Package Files**

Files from installed packages keep their original prefix.

```
Example: IMOF_Log__c, IMOF_LogHandler
Pattern: Different prefix than project prefix

Action: "This appears to be from a managed package ([PACKAGE_PREFIX]_).
Package files should keep their original prefix."
```

**Edge Case 4: Test Classes**

Test classes follow the same rules as the classes they test.

```
✅ Examples:
- [PREFIX]_DocumentHandler.cls → [PREFIX]_DocumentHandler_Test.cls
- CommunitiesLoginController.cls → CommunitiesLoginControllerTest.cls (no prefix)
```

### Step 5: Provide Decision with Clear Reasoning

Always explain the decision in a structured format:

```
📊 PREFIX ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Project: [Project Name]
Project Prefix: [PREFIX]_
Component: [ComponentName]
Type: [Component Type]

DECISION: [✅ PREFIX REQUIRED | ❌ NO PREFIX NEEDED]

Reasoning:
[Specific explanation of why prefix is/isn't needed]

Rule Applied: [Rule #: Description]

Suggested Name: [FinalName]
```

### Step 6: Handle User Uncertainty

If the user is unsure, provide decision-making guidance:

```
❓ NEED HELP DECIDING?

Quick questions to ask yourself:

1. Did you create this file, or is it from Salesforce?
   → You created it = Prefix needed
   → From Salesforce = No prefix

2. Is this a Community/Sites standard controller?
   → Yes = No prefix
   → No = Prefix needed

3. Is this a field on a custom object (e.g., [PREFIX]_MyObject__c)?
   → Yes = No prefix (object already has it)
   → No, it's on a standard object = Prefix needed

4. Does the filename start with "Communities", "Site", "Forgot", etc.?
   → Yes = No prefix (standard Salesforce file)
   → No = Prefix needed

Still unsure? Share the filename and context, and I'll analyze it!
```

## Examples

### Example 1: Custom Apex Class (Generic Project)

**User asks:** "Should MyDocumentHelper need a prefix?"

**Analysis:**
```
Project: MyProject
Project Prefix: MYPROJECT_
Component: MyDocumentHelper.cls
Type: Custom Apex Class

DECISION: ✅ PREFIX REQUIRED

Reasoning:
- Custom Apex class created by user (not from Salesforce)
- Not a standard community/site controller
- No special Salesforce naming pattern detected

Rule Applied: RULE 1 - Custom Apex → PREFIX REQUIRED

Suggested Name: MYPROJECT_MyDocumentHelper.cls
```

### Example 2: Community Controller (Any Project)

**User asks:** "Do I need a prefix for CommunitiesLandingController?"

**Analysis:**
```
Project: AnyProject
Project Prefix: ANY_
Component: CommunitiesLandingController.cls
Type: Standard Salesforce Community Controller

DECISION: ❌ NO PREFIX NEEDED

Reasoning:
- Filename starts with "Communities" - this is a Salesforce standard controller
- Standard Salesforce files maintain their original names
- Adding a prefix would break Salesforce functionality

Rule Applied: RULE 2 - Standard Salesforce Files → NO PREFIX

Suggested Name: CommunitiesLandingController.cls (keep as-is)
```

### Example 3: Field on Custom Object

**User asks:** "Should Status__c field on PROJ_Document__c have a prefix?"

**Analysis:**
```
Project: ProjectX
Project Prefix: PROJ_
Component: Status__c (custom field)
Parent Object: PROJ_Document__c (custom object with prefix)
Type: Custom field on custom object

DECISION: ❌ NO PREFIX NEEDED

Reasoning:
- Parent object (PROJ_Document__c) already has the project prefix
- Fields on custom objects inherit the namespace from their parent
- Adding prefix to field would be redundant: PROJ_Status__c on PROJ_Document__c

Rule Applied: RULE 5 - Custom Object Fields → NO PREFIX

Suggested Name: Status__c (no prefix)
```

### Example 4: Field on Standard Object

**User asks:** "Should ExtraInfo__c on Account have a prefix?"

**Analysis:**
```
Project: CompanyX
Project Prefix: COMPANYX_
Component: ExtraInfo__c (custom field)
Parent Object: Account (standard Salesforce object)
Type: Custom field on standard object

DECISION: ✅ PREFIX REQUIRED

Reasoning:
- Parent object (Account) is a standard Salesforce object with no prefix
- Custom fields on standard objects need project identification
- Without prefix, unclear which project added this field

Rule Applied: RULE 6 - Standard Object Fields → PREFIX REQUIRED

Suggested Name: COMPANYX_ExtraInfo__c
```

### Example 5: Lightning Web Component

**User asks:** "What about my documentViewer component?"

**Analysis:**
```
Project: MyOrg
Project Prefix: MYORG_
Component: documentViewer (LWC)
Type: Lightning Web Component

DECISION: ✅ PREFIX REQUIRED (lowercase)

Reasoning:
- Custom Lightning Web Component created by user
- LWC naming convention uses lowercase prefix
- Project prefix MYORG_ becomes myorg_ for LWC

Rule Applied: RULE 7 - LWC → PREFIX REQUIRED (lowercase)

Suggested Name: myorg_documentViewer/
```

### Example 6: Multiple Projects Example

**User asks:** "What prefix should I use?" (in CrmXyz project)

**Analysis:**
```
Working Directory: P:\CrmXyz
Detection Method: Folder name pattern matching

Pattern: Crm[ProjectName]
Extracted: Enforcement
Transformation: XYZ_

Validation:
✅ Length: 12 characters (within 2-10 range - acceptable)
✅ Format: Uppercase letters
✅ Ends with underscore

Project Prefix: XYZ_

Alternative (if too long): ENFOR_ or ABC_ (user choice)

For LWC: enforcement_ (lowercase)
```

## Important Notes

### When to Use This Skill

**Always use sf-prefix-detect:**
- ✅ Before creating any new Salesforce component
- ✅ When unsure if prefix is needed
- ✅ When renaming existing files
- ✅ During code reviews
- ✅ When onboarding new team members
- ✅ When migrating code between projects

**This skill is typically called by:**
- sf-apex-create
- sf-trigger-create
- sf-lwc-create
- sf-object-create
- sf-field-create
- sf-validate-all

### Prefix Format Standards

**Apex/Triggers/Flows/Custom Objects:**
- Format: `[PREFIX]_[Name]`
- Example: `MYPROJECT_DocumentHandler`
- Case: PascalCase for name after prefix
- Prefix: Always uppercase

**Lightning Web Components:**
- Format: `[prefix]_[name]`
- Example: `myproject_documentViewer`
- Case: camelCase for name after prefix
- Prefix: Always lowercase (converted from uppercase project prefix)

**Custom Fields:**
- On standard objects: `[PREFIX]_[FieldName]__c`
  - Example: `MYPROJECT_CustomField__c`
- On custom objects: `[FieldName]__c` (no prefix)
  - Example: `Status__c` on `MYPROJECT_Document__c`

### Generic Detection Algorithm

The detection algorithm works for any project structure:

```
STEP 1: Detect Project Prefix
├─ Try: Extract from folder name
│  ├─ Pattern: Crm[Name] → [NAME]_
│  └─ Pattern: [Name] → [NAME]_
├─ Try: Scan existing files for common prefix
│  └─ Find most frequent [PREFIX]_*.cls pattern
└─ Fallback: Ask user for prefix

STEP 2: Identify Component Type
└─ Classify: Apex/Trigger/LWC/Flow/Object/Field/Standard

STEP 3: Apply Appropriate Rule (1-8)
└─ Match component type to rule

STEP 4: Handle Edge Cases
├─ Ambiguous names → Ask user
├─ Multiple prefixes → Clarify
└─ Package files → Preserve original

STEP 5: Return Decision + Reasoning
└─ Explain which rule applies and why
```

This algorithm is **project-agnostic** and works regardless of:
- Project naming conventions
- Folder structure
- Existing file patterns
- Team size or organization

## Troubleshooting

### Issue: Can't determine prefix from folder name

**Cause:** Folder name doesn't follow recognizable pattern

**Solution:**
1. Run file scan: `find . -name "*_*.cls" -type f`
2. Extract common patterns: `ABC_*, XYZ_*, etc.`
3. Identify most frequent prefix
4. If no custom files exist, ask user directly
5. Document chosen prefix for future reference

### Issue: Folder name is too generic

**Example:** Working directory is just "Salesforce" or "Project"

**Solution:**
1. Check for CLAUDE.md or project docs with prefix info
2. Scan existing files for patterns
3. Look for package.xml or sfdx-project.json for clues
4. Ask user: "What prefix does your team use for this project?"

### Issue: File name is ambiguous

**Example:** "LoginHandler" could be custom or standard

**Solution:**
1. Search project for similar files
2. Check if other Community/Site files exist
3. Present both options:
   ```
   Option A: Custom handler → [PREFIX]_LoginHandler
   Option B: Standard Salesforce → LoginHandler (or SiteLoginHandler)
   ```
4. Let user decide based on their intent

### Issue: Multiple prefixes detected

**Example:** Found `ABC_*, DEF_*, XYZ_*` in same project

**Possible causes:**
- **Managed packages**: External packages (keep their prefix)
- **Migration**: Old code from different project
- **Multi-team**: Different teams using different prefixes

**Solution:**
1. Identify which is the **project prefix** (most common in custom files)
2. Identify which are **package prefixes** (from installed packages)
3. Ask user to confirm project prefix
4. Document: "Project prefix: [PREFIX]_, Package prefixes: [PKG1]_, [PKG2]_"

### Issue: User disagrees with recommendation

**Cause:** Project-specific conventions or legacy decisions

**Solution:**
1. Respect the user's decision
2. Ask if this exception should be documented
3. Consider adding to project's CLAUDE.md:
   ```
   ## Custom Naming Conventions
   - Exception: [FileName] uses [PREFIX] because [reason]
   ```
4. Update project-specific rules if pattern exists

## Reference

### Standard Salesforce File Patterns (Never Prefix These)

**Community/Portal Controllers:**
- `Communities*` - CommunitiesLandingController, CommunitiesLoginController, etc.
- `Site*` - SiteLoginController, SiteRegisterController, etc.
- `Microbatch*` - MicrobatchSelfRegController, etc.

**Authentication/Security:**
- `Forgot*` - ForgotPasswordController, etc.
- `ChangePassword*` - ChangePasswordController, etc.
- `MyProfile*` - MyProfilePageController, etc.
- `Lightning*` - LightningForgotPasswordController, etc.

**Standard Salesforce Objects (Partial List):**
- **Core CRM**: Account, Contact, Lead, Opportunity, Case
- **Security**: User, Profile, PermissionSet, PermissionSetGroup
- **Sales**: Campaign, Contract, Order, Product2, Pricebook2, Quote
- **Service**: Case, LiveChatTranscript, MessagingSession
- **Activities**: Task, Event, EmailMessage
- **Content**: Attachment, ContentDocument, ContentVersion
- **Platform**: CustomObject, CustomField, ApexClass, ApexTrigger

Full list: Any object without `__c` suffix is standard.

### Quick Reference Table

| Component Type | Prefix Needed? | Format | Example |
|----------------|----------------|--------|---------|
| Custom Apex Class | ✅ Yes | `[PREFIX]_Name` | `PROJ_Handler.cls` |
| Custom Trigger | ✅ Yes | `[PREFIX]_Name` | `PROJ_Trigger.trigger` |
| Community/Site File | ❌ No | `Original Name` | `CommunitiesLanding*.cls` |
| Standard Object | ❌ No | `Original Name` | `Account.object-meta.xml` |
| Custom Object | ✅ Yes | `[PREFIX]_Name__c` | `PROJ_MyObject__c` |
| Field on Custom Object | ❌ No | `FieldName__c` | `Status__c` |
| Field on Standard Object | ✅ Yes | `[PREFIX]_Field__c` | `PROJ_Field__c` |
| LWC | ✅ Yes (lowercase) | `[prefix]_name` | `proj_component/` |
| Flow | ✅ Yes | `[PREFIX]_Name` | `PROJ_Flow.flow-meta.xml` |
| Test Class | Same as tested class | Follow class rules | `PROJ_Handler_Test.cls` |

### Prefix Transformation Examples

| Project Folder | Detected Prefix | LWC Prefix |
|----------------|-----------------|------------|
| CrmAbc | ABC_ | rtn_ |
| CrmXyz | XYZ_ | enforcement_ |
| CrmDef | DEF_ | gcr_ |
| CrmGhi | GHI_ | gim_ |
| MyProject | MYPROJECT_ | myproject_ |
| SalesforceApp | SALESFORCEAPP_ | salesforceapp_ |

### Decision Tree

```
START: Need to name a Salesforce component
│
├─ Is it from Salesforce (Community/Site/Standard)?
│  ├─ YES → NO PREFIX NEEDED
│  └─ NO → Continue
│
├─ Is it a custom object?
│  ├─ YES → PREFIX REQUIRED ([PREFIX]_Name__c)
│  └─ NO → Continue
│
├─ Is it a field?
│  ├─ On Standard Object → PREFIX REQUIRED ([PREFIX]_Field__c)
│  └─ On Custom Object → NO PREFIX (Field__c)
│
├─ Is it Apex/Trigger/Flow?
│  └─ YES → PREFIX REQUIRED ([PREFIX]_Name)
│
├─ Is it LWC?
│  └─ YES → PREFIX REQUIRED lowercase ([prefix]_name)
│
└─ UNSURE? → Ask user for clarification
```

---

**Generated by Claude Code**
**Last Updated:** 2026-05-05
**Version:** 2.0.0 (Generic/Multi-Project Support)
