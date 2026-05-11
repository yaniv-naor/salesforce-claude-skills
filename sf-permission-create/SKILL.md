---
name: sf-permission-create
description: Creates Salesforce Permission Sets with validation and prefix support. Handles object/field/apex/tab permissions, validation rules, and common deployment errors. Use when creating permission sets, granting access, configuring security. Trigger on "create permission", "new permission set", "grant access", "add permissions", "permission for", "access to object/field". Works across any Salesforce project.
license: MIT
metadata:
  version: 1.0.0
  category: salesforce-development
  tags: [salesforce, permissions, security, metadata, generic]
---

# SF Permission Create

Creates production-grade Salesforce Permission Sets with proper structure, validation, and project prefix support. **Works with any Salesforce project** regardless of naming conventions or structure.

## When to Use

- Creating new Permission Sets
- Granting object CRUD permissions
- Configuring field-level security (FLS)
- Setting up user permissions
- Configuring tab visibility
- Granting Apex/Visualforce access
- Adding agent access (Agentforce)

## Instructions

### Step 1: Detect Project Prefix

Use `sf-prefix-detect` skill to determine the project prefix.

**Quick fallback (if skill unavailable):**
1. Get working directory name
2. Apply transformation:
   - `Crm[Name]` → `[NAME]_`
   - `[Name]` → `[NAME]_`
3. Validate: 2-10 uppercase characters + underscore

**Note:** Prefix is used to identify custom objects, fields, tabs, and Apex classes that need the prefix qualifier.

### Step 2: Gather Permission Set Information

Ask user for (or infer from request):

**Required:**
- **Permission Set Name** (API name, no spaces)
- **Label** (Display name for admins)
- **Description** (Purpose and intended audience)

**Optional:**
- Objects to grant access to
- Fields requiring FLS
- User permissions needed
- Tab visibility settings
- Apex classes/Visualforce pages
- Application visibility
- License type (default: Salesforce)

**Auto-detected:**
- API version: Read from `sfdx-project.json` or default to `60.0`

### Step 3: Create Base Permission Set Structure

Generate the base XML with core properties:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>YourPermissionSetName</fullName>
    <label>Display Name for Administrators</label>
    <description>Clear description of purpose and intended audience</description>
    <hasActivationRequired>false</hasActivationRequired>
    <license>Salesforce</license>
</PermissionSet>
```

**Naming conventions:**
- **API name**: Descriptive, no spaces (e.g., `Sales_Manager_Access`)
- **Label**: Human-readable (e.g., "Sales Manager Access")
- **Description**: Include purpose, target users, and scope

### Step 4: Add Object Permissions

For each object that needs access, add CRUD permissions:

```xml
<objectPermissions>
    <allowCreate>true</allowCreate>
    <allowRead>true</allowRead>
    <allowEdit>true</allowEdit>
    <allowDelete>false</allowDelete>
    <modifyAllRecords>false</modifyAllRecords>
    <viewAllRecords>false</viewAllRecords>
    <object>ObjectName</object>
</objectPermissions>
```

**Important rules:**
- **Custom objects**: Must include `__c` suffix and apply project prefix if created by user
  - Example: `[PREFIX]_Document__c`
- **Standard objects**: Use exact API name (Account, Contact, Opportunity, etc.)
- **View All vs Modify All**: Carefully consider security implications
- **viewAllFields**: Alternative to individual field permissions if all fields should be visible

**Object naming with prefix:**
1. For custom objects created by user: `[PREFIX]_ObjectName__c`
2. For standard objects: No prefix (e.g., `Account`, `Contact`)
3. For managed package objects: Keep original prefix (e.g., `IMOF_Log__c`)

### Step 5: Add Field-Level Security

**CRITICAL:** Never add required fields to field permissions. This causes deployment failure.

```xml
<fieldPermissions>
    <editable>true</editable>
    <readable>true</readable>
    <field>ObjectName.FieldName__c</field>
</fieldPermissions>
```

**Validation checklist:**
- [ ] Field exists on the object
- [ ] Field is NOT required (`<required>true</required>` in field metadata)
- [ ] Formula fields are NOT marked as editable
- [ ] Master-detail fields are NOT included (always required)
- [ ] Format is `ObjectName.FieldName`
- [ ] Custom fields end with `__c`

**Field naming with prefix:**
1. **Custom field on standard object**: `[PREFIX]_FieldName__c`
   - Example: `Account.[PREFIX]_CustomField__c`
2. **Custom field on custom object**: No prefix on field (object already has it)
   - Example: `[PREFIX]_Document__c.Status__c`
3. **Standard field on standard object**: No prefix
   - Example: `Account.BillingCity`

**When to use viewAllFields:**
- If all custom fields should be visible, set `<viewAllFields>true</viewAllFields>` in objectPermissions
- This is simpler than listing every field individually
- Still need explicit fieldPermissions for sensitive fields requiring edit access

### Step 6: Add User Permissions

Grant system-level permissions:

```xml
<userPermissions>
    <enabled>true</enabled>
    <name>ApiEnabled</name>
</userPermissions>
```

**Common user permissions:**
- `ApiEnabled`: API access
- `ViewSetup`: View Setup menu
- `ManageUsers`: User management
- `RunReports`: Report execution
- `EditTask`: Edit tasks
- `EditEvent`: Edit events

**Security review required for:**
- `ViewAllData`: Read all records (bypass sharing)
- `ModifyAllData`: Edit all records (bypass sharing)
- `ManageUsers`: User administration
- `ViewEncryptedData`: See encrypted fields

### Step 7: Configure Tab Visibility

Make tabs visible to users:

```xml
<tabSettings>
    <tab>TabName</tab>
    <visibility>Visible</visibility>
</tabSettings>
```

**Tab visibility options:**
- `Visible`: Tab appears in the visible tabs for its app (can be customized)
- `Available`: Tab available on All Tabs page, user can customize
- `None`: Not visible

**Tab naming rules:**
- **Custom object tabs**: MUST include `__c` suffix and apply prefix
  - Example: `[PREFIX]_CustomObject__c`
- **Standard object tabs**: Use format `standard-ObjectName`
  - Example: `standard-Account`, `standard-Contact`
- **Custom tabs (not objects)**: May or may not need prefix, check existing tab API name

### Step 8: Add Apex and Visualforce Access (Optional)

Grant access to custom code:

```xml
<classAccesses>
    <apexClass>ClassName</apexClass>
    <enabled>true</enabled>
</classAccesses>
<pageAccesses>
    <apexPage>PageName</apexPage>
    <enabled>true</enabled>
</pageAccesses>
```

**Apex class naming with prefix:**
- **Custom classes**: Apply project prefix
  - Example: `[PREFIX]_DocumentHandler`
- **Standard Salesforce classes**: No prefix
  - Example: `CommunitiesLandingController`

### Step 9: Add Application Visibility (Optional)

Make applications visible:

```xml
<applicationVisibilities>
    <application>AppName</application>
    <visible>true</visible>
</applicationVisibilities>
```

**Application naming:**
- Custom applications may need prefix (check API name)
- Standard applications use Salesforce name

### Step 10: Add Agent Access (Optional)

Enable Agentforce Employee Agent access:

```xml
<agentAccesses>
    <agentName>Sales_Assistant_Agent</agentName>
    <enabled>true</enabled>
</agentAccesses>
```

**Requirements:**
- Agent name must match existing Agentforce Employee Agent developer name
- Only applicable in orgs with Agentforce enabled

### Step 11: Validate Permission Set

Run validation checks before saving:

**Required fields:**
- [ ] `fullName` is set
- [ ] `label` is set
- [ ] `description` is set

**Permission validation:**
- [ ] No required fields in `<fieldPermissions>`
- [ ] No duplicate permissions
- [ ] Custom object/field/tab names include `__c` suffix
- [ ] Project prefix applied correctly to custom components
- [ ] Standard object/field names are correct

**Common errors to prevent:**
- Required field in fieldPermissions → DEPLOYMENT FAILURE
- Missing `__c` on custom objects/fields → DEPLOYMENT FAILURE
- Incorrect API names → DEPLOYMENT FAILURE
- Wrong prefix or no prefix → DEPLOYMENT FAILURE

### Step 12: Save Permission Set

Save to standard Salesforce structure:

```
force-app/main/default/permissionsets/
└── PermissionSetName.permissionset-meta.xml
```

### Step 13: Provide Summary

```
✅ Created Permission Set: [PermissionSetName].permissionset-meta.xml

📋 Permission Set Details:
- API Name: [PermissionSetName]
- Label: [Label]
- License: [License Type]

📊 Permissions Granted:
- Object Permissions: [count] objects
- Field Permissions: [count] fields
- User Permissions: [count] permissions
- Tab Visibility: [count] tabs
- Apex Classes: [count] classes

✅ Validation:
- No required fields in field permissions
- All API names validated
- Project prefix applied correctly

📝 Next Steps:
1. Review permission set in [file_path]
2. Deploy: sf project deploy start -d force-app/main/default/permissionsets
3. Assign to users: Setup → Permission Sets → [Label] → Manage Assignments
4. Test with target user

📍 File Created:
- [file_path:line_number]
```

## Permission Set Patterns

### Pattern 1: Basic Object Access

**Scenario:** Grant read/create/edit on custom object

```xml
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Document_User</fullName>
    <label>Document User Access</label>
    <description>Allows users to view, create, and edit documents</description>
    
    <objectPermissions>
        <allowCreate>true</allowCreate>
        <allowRead>true</allowRead>
        <allowEdit>true</allowEdit>
        <allowDelete>false</allowDelete>
        <modifyAllRecords>false</modifyAllRecords>
        <viewAllRecords>false</viewAllRecords>
        <object>PREFIX_Document__c</object>
    </objectPermissions>
    
    <tabSettings>
        <tab>PREFIX_Document__c</tab>
        <visibility>Visible</visibility>
    </tabSettings>
</PermissionSet>
```

### Pattern 2: Field-Level Security

**Scenario:** Grant access to sensitive fields

```xml
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Account_Sensitive_Fields</fullName>
    <label>Account Sensitive Fields Access</label>
    <description>Grants access to sensitive fields on Account</description>
    
    <!-- Read access on Account already granted by profile -->
    
    <fieldPermissions>
        <editable>true</editable>
        <readable>true</readable>
        <field>Account.PREFIX_SSN__c</field>
    </fieldPermissions>
    
    <fieldPermissions>
        <editable>false</editable>
        <readable>true</readable>
        <field>Account.PREFIX_CreditScore__c</field>
    </fieldPermissions>
</PermissionSet>
```

### Pattern 3: User Permissions

**Scenario:** Grant system-level capabilities

```xml
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>API_User</fullName>
    <label>API User Access</label>
    <description>Allows integration user to access Salesforce via API</description>
    
    <userPermissions>
        <enabled>true</enabled>
        <name>ApiEnabled</name>
    </userPermissions>
    
    <userPermissions>
        <enabled>true</enabled>
        <name>RunReports</name>
    </userPermissions>
</PermissionSet>
```

### Pattern 4: Apex Access

**Scenario:** Grant access to custom Apex classes

```xml
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Document_Processor</fullName>
    <label>Document Processor Access</label>
    <description>Allows access to document processing functionality</description>
    
    <classAccesses>
        <apexClass>PREFIX_DocumentHandlerCtrl</apexClass>
        <enabled>true</enabled>
    </classAccesses>
    
    <classAccesses>
        <apexClass>PREFIX_DocumentService</apexClass>
        <enabled>true</enabled>
    </classAccesses>
    
    <objectPermissions>
        <allowCreate>true</allowCreate>
        <allowRead>true</allowRead>
        <allowEdit>true</allowEdit>
        <allowDelete>false</allowDelete>
        <modifyAllRecords>false</modifyAllRecords>
        <viewAllRecords>false</viewAllRecords>
        <object>PREFIX_Document__c</object>
    </objectPermissions>
</PermissionSet>
```

### Pattern 5: Comprehensive Access

**Scenario:** Grant full access to a feature (object + fields + tabs + apex)

```xml
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Document_Admin</fullName>
    <label>Document Administrator</label>
    <description>Full administrative access to Document management feature</description>
    
    <objectPermissions>
        <allowCreate>true</allowCreate>
        <allowRead>true</allowRead>
        <allowEdit>true</allowEdit>
        <allowDelete>true</allowDelete>
        <modifyAllRecords>true</modifyAllRecords>
        <viewAllRecords>true</viewAllRecords>
        <viewAllFields>true</viewAllFields>
        <object>PREFIX_Document__c</object>
    </objectPermissions>
    
    <tabSettings>
        <tab>PREFIX_Document__c</tab>
        <visibility>Visible</visibility>
    </tabSettings>
    
    <classAccesses>
        <apexClass>PREFIX_DocumentHandlerCtrl</apexClass>
        <enabled>true</enabled>
    </classAccesses>
    
    <applicationVisibilities>
        <application>PREFIX_DocumentApp</application>
        <visible>true</visible>
    </applicationVisibilities>
</PermissionSet>
```

## Validation Rules

### Rule 1: Required Fields

**Never add required fields to fieldPermissions.**

**How to check if field is required:**
1. Read field metadata: `force-app/main/default/objects/ObjectName/fields/FieldName__c.field-meta.xml`
2. Look for `<required>true</required>`
3. Master-detail relationships are always required
4. Formula fields cannot be edited

**Example of required field metadata:**
```xml
<CustomField xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>RequiredField__c</fullName>
    <required>true</required>
</CustomField>
```

**If this field appears in fieldPermissions → DEPLOYMENT FAILS**

### Rule 2: API Names

Validate all API names before adding:

**Custom objects:**
- Must end with `__c`
- Apply project prefix for user-created objects
- Example: `PREFIX_Document__c`

**Custom fields:**
- Must end with `__c`
- Apply prefix based on parent object type
- Example: `Account.PREFIX_CustomField__c` (custom field on standard object)
- Example: `PREFIX_Document__c.Status__c` (custom field on custom object)

**Standard objects/fields:**
- No `__c` suffix
- No prefix
- Example: `Account.BillingCity`

**Custom tabs:**
- Custom object tabs: Match object API name with `__c`
- Standard object tabs: Use `standard-` prefix
- Example: `PREFIX_Document__c` (custom object tab)
- Example: `standard-Account` (standard object tab)

### Rule 3: No Duplicates

Each permission should appear only once:
- Only one objectPermissions block per object
- Only one fieldPermissions block per field
- Only one userPermissions block per permission name

### Rule 4: Least Privilege

Follow security best practices:
- Grant minimum permissions needed
- Avoid `ViewAllData` / `ModifyAllData` unless absolutely necessary
- Use `viewAllRecords` / `modifyAllRecords` sparingly
- Document why elevated permissions are needed

## Common Deployment Failures

### Error 1: Field is required

**Error message:**
```
Cannot set field level security for a required field
```

**Cause:** Field in `<fieldPermissions>` has `<required>true</required>` in its metadata

**Solution:**
1. Remove the field from fieldPermissions
2. Required fields are accessible by default (users with object access can see/edit them)

### Error 2: Invalid API name

**Error message:**
```
Field [ObjectName].[FieldName] does not exist
```

**Cause:** 
- Typo in field name
- Missing `__c` suffix on custom field
- Missing prefix on custom component

**Solution:**
1. Verify field exists in object metadata
2. Check correct API name including `__c` suffix
3. Apply project prefix to custom components

### Error 3: Missing __c suffix

**Error message:**
```
Custom field must end with __c
```

**Cause:** Custom field/object/tab missing `__c` suffix

**Solution:**
1. Add `__c` suffix to custom components
2. Verify with Glob tool: `force-app/main/default/objects/*/fields/*.field-meta.xml`

### Error 4: Invalid object

**Error message:**
```
Object [ObjectName] does not exist
```

**Cause:**
- Object not deployed yet
- Typo in object name
- Wrong prefix

**Solution:**
1. Deploy object first
2. Verify object API name
3. Check project prefix is correct

## Modifying Existing Permission Sets

When user asks to modify or update an existing permission set:

### Step 1: Read Existing Permission Set

1. Use Read tool to examine current permissions
2. Identify what needs to change
3. Check for existing structure

### Step 2: Make Changes

Common modifications:
- **Add object permission:** Insert new `<objectPermissions>` block
- **Add field permission:** Insert new `<fieldPermissions>` block (validate not required)
- **Add user permission:** Insert new `<userPermissions>` block
- **Add tab visibility:** Insert new `<tabSettings>` block
- **Remove permission:** Delete the corresponding block

### Step 3: Validate Changes

- [ ] No required fields added
- [ ] API names correct
- [ ] Prefix applied correctly
- [ ] No duplicates created

### Step 4: Save and Summarize

```
✅ Updated Permission Set: [PermissionSetName].permissionset-meta.xml

📊 Changes:
- Added: [list of added permissions]
- Removed: [list of removed permissions]
- Modified: [list of modified permissions]

📝 Next Steps:
1. Review changes in [file_path]
2. Deploy: sf project deploy start -d force-app/main/default/permissionsets
3. Test with assigned users

📍 File Modified:
- [file_path:line_numbers]
```

## Examples

### Example 1: Basic Permission Set

**User:** "Create a permission set for Document users"

**Actions:**
1. Detect prefix: `PROJ_`
2. Gather requirements:
   - Name: `Document_User`
   - Label: "Document User Access"
   - Object: `PROJ_Document__c` (read, create, edit)
   - Tab: `PROJ_Document__c`
3. Create permission set with object permissions and tab visibility
4. Validate: No required fields, API names correct
5. Save to `force-app/main/default/permissionsets/Document_User.permissionset-meta.xml`

**Result:** Permission set ready for deployment.

### Example 2: Field-Level Security

**User:** "Grant access to SSN field on Account"

**Actions:**
1. Detect prefix: `PROJ_`
2. Field: `Account.PROJ_SSN__c` (custom field on standard object)
3. Validate field is not required:
   - Read metadata: `force-app/main/default/objects/Account/fields/PROJ_SSN__c.field-meta.xml`
   - Confirm `<required>false</required>` or no required tag
4. Create permission set with field permissions
5. Save file

**Result:** Permission set grants FLS on sensitive field.

### Example 3: Comprehensive Access

**User:** "Create admin access for Document management"

**Actions:**
1. Detect prefix: `PROJ_`
2. Gather comprehensive requirements:
   - Object: `PROJ_Document__c` (full CRUD + View/Modify All)
   - Fields: All fields (use `viewAllFields`)
   - Tab: `PROJ_Document__c`
   - Apex: `PROJ_DocumentHandlerCtrl`, `PROJ_DocumentService`
   - App: `PROJ_DocumentApp`
3. Create permission set with all components
4. Add security note in description about elevated permissions
5. Validate and save

**Result:** Complete administrative permission set.

## Troubleshooting

### Issue: Prefix unclear

**Solution:** Use sf-prefix-detect skill or ask user directly

### Issue: Don't know if field is required

**Solution:**
1. Read field metadata file
2. Check for `<required>true</required>`
3. Master-detail relationships are always required
4. If metadata doesn't exist, field might not be deployed yet

### Issue: Permission set already exists

**Solution:**
1. Read existing file
2. Ask user:
   - Add to existing permission set
   - Create new permission set with different name
   - Overwrite (show current content first)

### Issue: Object doesn't exist yet

**Solution:**
1. Check if object metadata exists
2. If not, suggest creating object first with sf-object-create skill
3. Or proceed with permission set creation knowing object must be deployed together

### Issue: Tab name unclear

**Solution:**
1. For custom object tabs: Use object API name (e.g., `PREFIX_Object__c`)
2. For standard object tabs: Use `standard-ObjectName` format
3. For custom tabs (non-object): Search for tab metadata file to get API name

## Reference

**File structure:**
```
force-app/main/default/permissionsets/
└── PermissionSetName.permissionset-meta.xml
```

**Permission types:**

| Type | Element | Purpose |
|------|---------|---------|
| Object CRUD | `<objectPermissions>` | Create, Read, Edit, Delete access |
| Field FLS | `<fieldPermissions>` | Field-level read/edit access |
| User Permissions | `<userPermissions>` | System-level capabilities |
| Tab Visibility | `<tabSettings>` | Tab display settings |
| Apex Access | `<classAccesses>` | Apex class execution |
| Visualforce Access | `<pageAccesses>` | VF page viewing |
| App Visibility | `<applicationVisibilities>` | Application access |
| Agent Access | `<agentAccesses>` | Agentforce agent access |

**Naming patterns:**

| Component | Needs Prefix? | Format | Example |
|-----------|---------------|--------|---------|
| Custom Object | Yes | `[PREFIX]_Object__c` | `PROJ_Document__c` |
| Standard Object | No | `ObjectName` | `Account` |
| Custom Field (Standard Object) | Yes | `Object.[PREFIX]_Field__c` | `Account.PROJ_Field__c` |
| Custom Field (Custom Object) | No | `[PREFIX]_Object__c.Field__c` | `PROJ_Document__c.Status__c` |
| Custom Tab (Object) | Yes | `[PREFIX]_Object__c` | `PROJ_Document__c` |
| Standard Tab | No | `standard-ObjectName` | `standard-Account` |
| Custom Apex | Yes | `[PREFIX]_ClassName` | `PROJ_Handler` |
| Standard Apex | No | `ClassName` | `CommunitiesLanding*` |

**Common user permissions:**
- `ApiEnabled` - API access
- `ViewSetup` - View Setup
- `ManageUsers` - User management
- `RunReports` - Run reports
- `EditTask` - Edit tasks
- `EditEvent` - Edit events
- `ViewAllData` - View all data (SECURITY RISK)
- `ModifyAllData` - Modify all data (SECURITY RISK)

**Tab visibility values:**
- `Visible` - Tab visible in app
- `Available` - Tab available for customization
- `None` - Tab hidden

---

**Generated by Claude Code**
**Last Updated:** 2026-05-05
**Version:** 1.0.0 (Claude Code)
