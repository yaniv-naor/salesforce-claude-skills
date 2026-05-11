# SF Permission Create

**Version:** 1.0.0  
**Author:** Claude Code  
**Category:** Salesforce Development  
**License:** MIT

## Overview

Creates production-grade Salesforce Permission Sets with proper validation, prefix support, and comprehensive error prevention. Works across any Salesforce project regardless of naming conventions or structure.

## When to Use This Skill

Use sf-permission-create when:
- Creating new Permission Sets
- Granting object CRUD permissions
- Configuring field-level security (FLS)
- Setting up user permissions
- Configuring tab visibility
- Granting Apex/Visualforce access
- Adding Agentforce agent access

## Trigger Phrases

The skill activates when you mention:
- "create permission set"
- "new permission"
- "grant access"
- "add permissions"
- "permission for [object/field]"
- "access to [object/field]"

## Key Features

### 1. Automatic Prefix Detection
- Detects project prefix from folder name or existing files
- Applies prefix correctly to custom components
- Knows when NOT to apply prefix (standard objects, fields on custom objects)

### 2. Comprehensive Validation
- Prevents adding required fields (causes deployment failure)
- Validates API names for correctness
- Checks for __c suffixes on custom components
- Prevents duplicate permissions

### 3. Multiple Permission Types
- Object permissions (CRUD)
- Field-level security
- User permissions
- Tab visibility
- Apex/Visualforce access
- Application visibility
- Agent access (Agentforce)

### 4. Security Best Practices
- Warns about elevated permissions (ViewAllData, ModifyAllData)
- Documents security considerations in descriptions
- Follows least privilege principle
- Clear permission explanations

### 5. Error Prevention
- Validates fields are not required before adding
- Checks field metadata when available
- Provides clear error messages
- Suggests corrections for common mistakes

## Quick Start

### Example 1: Basic Object Permission

```
User: "Create a permission set for Document users with read and edit access"

Skill will:
1. Detect prefix (e.g., ABC_)
2. Create permission set with object permissions
3. Add tab visibility
4. Validate all API names
5. Save to correct location
```

### Example 2: Field-Level Security

```
User: "Grant access to SSN field on Account, make it editable"

Skill will:
1. Detect it's a custom field on standard object
2. Apply prefix: Account.ABC_SSN__c
3. Validate field is not required
4. Create permission set with field permissions
5. Set editable=true, readable=true
```

### Example 3: Comprehensive Permissions

```
User: "Create admin permission set for Document management"

Skill will:
1. Add full CRUD on object
2. Add viewAllFields for field access
3. Add tab visibility
4. Add Apex class access
5. Add user permissions (if requested)
6. Add application visibility
7. Document elevated permissions
```

## Permission Types Reference

### Object Permissions
- `allowCreate`: Create new records
- `allowRead`: View records
- `allowEdit`: Edit records
- `allowDelete`: Delete records
- `viewAllRecords`: View all records (bypasses sharing)
- `modifyAllRecords`: Edit all records (bypasses sharing)
- `viewAllFields`: View all fields (alternative to individual field permissions)

### Field Permissions
- `readable`: User can view field
- `editable`: User can edit field
- **WARNING**: Never add required fields (deployment fails)

### User Permissions
Common permissions:
- `ApiEnabled`: API access
- `ViewSetup`: View Setup menu
- `RunReports`: Run reports
- `EditTask`: Edit tasks
- `EditEvent`: Edit events

Security-sensitive:
- `ViewAllData`: View all data (requires justification)
- `ModifyAllData`: Edit all data (requires justification)
- `ManageUsers`: User management (requires justification)

### Tab Visibility
- `Visible`: Tab visible in app
- `Available`: Tab available for customization
- `None`: Tab hidden

## Naming Conventions

### Custom Objects
Format: `[PREFIX]_ObjectName__c`  
Example: `ABC_Document__c`

### Standard Objects
Format: `ObjectName` (no prefix)  
Example: `Account`, `Contact`, `Opportunity`

### Custom Fields on Standard Objects
Format: `ObjectName.[PREFIX]_FieldName__c`  
Example: `Account.ABC_SSN__c`

### Custom Fields on Custom Objects
Format: `[PREFIX]_ObjectName__c.FieldName__c` (no prefix on field)  
Example: `ABC_Document__c.Status__c`

### Custom Tabs
Object tabs: Match object API name  
Example: `ABC_Document__c`

Standard object tabs: Use `standard-` prefix  
Example: `standard-Account`

### Apex Classes
Format: `[PREFIX]_ClassName`  
Example: `ABC_DocumentHandler`

## Validation Rules

### Rule 1: No Required Fields
Never add required fields to fieldPermissions. This causes deployment failure.

**How to check:**
- Field metadata contains `<required>true</required>`
- Master-detail relationships are always required
- Standard Name field is typically required

### Rule 2: Correct API Names
- Custom components must end with `__c`
- Standard components have no suffix
- Prefix applied to user-created custom components

### Rule 3: No Duplicates
Each permission should appear only once in the permission set.

### Rule 4: Least Privilege
Grant minimum permissions needed. Document justification for elevated permissions.

## Common Deployment Failures (Prevented by Skill)

| Error | Cause | Prevention |
|-------|-------|-----------|
| "Cannot set FLS for required field" | Required field in fieldPermissions | Validates field is not required before adding |
| "Field does not exist" | Wrong API name or missing __c | Validates API naming conventions |
| "Object does not exist" | Wrong object name or missing prefix | Applies correct prefix based on object type |
| "Invalid custom object" | Missing __c suffix | Ensures custom objects have __c suffix |

## File Structure

Permission sets are saved to:
```
force-app/main/default/permissionsets/
└── PermissionSetName.permissionset-meta.xml
```

## Integration with Other Skills

### sf-prefix-detect
Called automatically to determine project prefix. Ensures consistent naming across all Salesforce components.

### sf-object-create
If creating permission for object that doesn't exist, suggests creating object first.

### sf-field-create
If granting FLS for field that doesn't exist, suggests creating field first.

### sf-validate-all
Can validate permission sets as part of overall project validation.

## Examples

### Example 1: Document User Permission Set

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Document_User</fullName>
    <label>Document User Access</label>
    <description>Allows users to view, create, and edit documents</description>
    <hasActivationRequired>false</hasActivationRequired>
    <license>Salesforce</license>
    <objectPermissions>
        <allowCreate>true</allowCreate>
        <allowRead>true</allowRead>
        <allowEdit>true</allowEdit>
        <allowDelete>false</allowDelete>
        <modifyAllRecords>false</modifyAllRecords>
        <viewAllRecords>false</viewAllRecords>
        <object>ABC_Document__c</object>
    </objectPermissions>
    <tabSettings>
        <tab>ABC_Document__c</tab>
        <visibility>Visible</visibility>
    </tabSettings>
</PermissionSet>
```

### Example 2: Field-Level Security

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Account_Sensitive_Fields</fullName>
    <label>Account Sensitive Fields Access</label>
    <description>Grants access to sensitive fields on Account</description>
    <hasActivationRequired>false</hasActivationRequired>
    <license>Salesforce</license>
    <fieldPermissions>
        <editable>true</editable>
        <readable>true</readable>
        <field>Account.ABC_SSN__c</field>
    </fieldPermissions>
    <fieldPermissions>
        <editable>false</editable>
        <readable>true</readable>
        <field>Account.ABC_CreditScore__c</field>
    </fieldPermissions>
</PermissionSet>
```

### Example 3: Admin Access with Multiple Permissions

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Document_Admin</fullName>
    <label>Document Administrator</label>
    <description>Full administrative access to Document management. 
    SECURITY NOTE: Includes elevated permissions (modifyAllRecords).</description>
    <hasActivationRequired>false</hasActivationRequired>
    <license>Salesforce</license>
    <applicationVisibilities>
        <application>ABC_Enforcement</application>
        <visible>true</visible>
    </applicationVisibilities>
    <classAccesses>
        <apexClass>ABC_DocumentHandler</apexClass>
        <enabled>true</enabled>
    </classAccesses>
    <objectPermissions>
        <allowCreate>true</allowCreate>
        <allowRead>true</allowRead>
        <allowEdit>true</allowEdit>
        <allowDelete>true</allowDelete>
        <modifyAllRecords>true</modifyAllRecords>
        <viewAllRecords>true</viewAllRecords>
        <viewAllFields>true</viewAllFields>
        <object>ABC_Document__c</object>
    </objectPermissions>
    <tabSettings>
        <tab>ABC_Document__c</tab>
        <visibility>Visible</visibility>
    </tabSettings>
    <userPermissions>
        <enabled>true</enabled>
        <name>ApiEnabled</name>
    </userPermissions>
</PermissionSet>
```

## Troubleshooting

### Issue: "Can't determine project prefix"
**Solution:** Skill will ask for prefix directly, or you can specify it in the request.

### Issue: "Field is required"
**Solution:** Skill validates and rejects required fields. Use viewAllFields on object instead.

### Issue: "Permission set already exists"
**Solution:** Skill will ask to add to existing, create with different name, or overwrite.

### Issue: "Wrong prefix applied"
**Solution:** Verify working directory and existing file patterns. Skill may need manual prefix specification.

## Deployment

After creating permission set:

```bash
# Deploy to org
sf project deploy start -d force-app/main/default/permissionsets

# Assign to users
Setup → Permission Sets → [Your Permission Set] → Manage Assignments
```

## Testing

The skill includes comprehensive test cases in `TEST_CASES.md`:
1. Basic object permissions
2. Field-level security
3. Comprehensive permissions
4. Modifying existing permission sets
5. Validation error handling
6. Prefix detection and application
7. Tab visibility patterns
8. Security-sensitive permissions

See `TEST_RESULTS.md` for validation results.

## Best Practices

1. **Start Small**: Create focused permission sets for specific features
2. **Least Privilege**: Grant minimum permissions needed
3. **Clear Descriptions**: Document purpose and intended users
4. **Validate Before Deploy**: Use skill validation to prevent errors
5. **Test Thoroughly**: Test with target users before production
6. **Document Elevated Permissions**: Always explain why ViewAllData/ModifyAllData needed
7. **Use viewAllFields**: Instead of listing every field individually
8. **Group Related Permissions**: Keep related permissions in same permission set

## Links

- **SKILL.md**: Complete skill instructions
- **TEST_CASES.md**: Test scenarios
- **TEST_RESULTS.md**: Validation results
- **Salesforce Docs**: [Permission Sets](https://help.salesforce.com/s/articleView?id=sf.perm_sets_overview.htm)

## Support

For issues or questions:
1. Check SKILL.md troubleshooting section
2. Review TEST_CASES.md for examples
3. Consult Claude Code

---

**Generated by Claude Code**  
**Skill Version:** 1.0.0  
**Last Updated:** 2026-05-05
