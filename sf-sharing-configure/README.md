# SF Sharing Configure - Quick Reference

Configure Salesforce sharing rules (Ownership-based and Criteria-based) with proper structure and validation. Works across any Salesforce project.

## Quick Start

**Trigger phrases:**
- "create sharing rule"
- "ownership rule"
- "criteria sharing"
- "grant access"
- "share records"

## Sharing Rule Types

### 1. Ownership-Based Rules
Share records based on who owns them.

**Use when:**
- "Share records owned by X to Y"
- "Give managers access to team member records"
- "Share by role/group ownership"

**Example:**
```xml
<sharingOwnerRules>
    <fullName>PREFIX_ShareToManagers</fullName>
    <accessLevel>Read</accessLevel>
    <label>Share to Managers</label>
    <sharedTo>
        <role>Manager</role>
    </sharedTo>
    <sharedFrom>
        <group>PREFIX_AllUsers</group>
    </sharedFrom>
</sharingOwnerRules>
```

### 2. Criteria-Based Rules
Share records based on field values.

**Use when:**
- "Share records where Status = Active"
- "Share high priority cases"
- "Share records matching criteria"

**Example:**
```xml
<sharingCriteriaRules>
    <fullName>PREFIX_ActiveRecords</fullName>
    <accessLevel>Edit</accessLevel>
    <label>Share Active Records</label>
    <sharedTo>
        <group>PREFIX_Team</group>
    </sharedTo>
    <criteriaItems>
        <field>Status__c</field>
        <operation>equals</operation>
        <value>Active</value>
    </criteriaItems>
    <includeRecordsOwnedByAll>true</includeRecordsOwnedByAll>
</sharingCriteriaRules>
```

### 3. Guest User Rules
Share to guest users in communities/sites.

**Use when:**
- "Share to guest users"
- "Public site access"
- "Community guest access"

**Example:**
```xml
<sharingGuestRules>
    <fullName>PREFIX_GuestAccess</fullName>
    <accessLevel>Read</accessLevel>
    <label>Guest User Access</label>
    <sharedTo>
        <guestUser>SiteName</guestUser>
    </sharedTo>
    <criteriaItems>
        <field>IsPublic__c</field>
        <operation>equals</operation>
        <value>True</value>
    </criteriaItems>
    <includeHVUOwnedRecords>false</includeHVUOwnedRecords>
</sharingGuestRules>
```

## Key Concepts

### Access Levels
- **Read** - View record and fields
- **Edit** - View and edit record

### sharedTo Options
- `<role>RoleName</role>` - Specific role only
- `<roleAndSubordinates>RoleName</roleAndSubordinates>` - Role + subordinates in hierarchy
- `<group>GroupName</group>` - Public group

### Criteria Operations
- `equals` - Field equals value
- `notEqual` - Field not equal
- `lessThan` / `greaterThan` - Numeric/date comparison
- `lessOrEqual` / `greaterOrEqual` - Inclusive comparison
- `contains` / `notContain` - Text contains
- `startsWith` - Text starts with

### Boolean Filters
Combine multiple criteria with logic:
- `1 AND 2` - Both conditions must be true
- `1 OR 2` - Either condition can be true
- `(1 OR 2) AND 3` - Complex nested logic

## File Structure

```
force-app/main/default/sharingRules/
├── Account.sharingRules-meta.xml           # Standard object
└── PREFIX_CustomObject__c.sharingRules-meta.xml  # Custom object
```

## Critical Rules

### ✅ Objects That Support Sharing Rules
- `Private` sharing model
- `Public Read Only` sharing model
- `ReadWrite` sharing model

### ❌ Objects That DON'T Support Sharing Rules
- `ControlledByParent` sharing model (child objects with Master-Detail)
- `Public Read/Write` sharing model (everyone has access)

### Naming Conventions
- **Rule name:** `[PREFIX]_RuleName`
- **Custom object file:** `[PREFIX]_Object__c.sharingRules-meta.xml`
- **Standard object file:** `ObjectName.sharingRules-meta.xml`
- **Custom field:** `FieldName__c` (has __c suffix)
- **Group name:** `[PREFIX]_GroupName` (apply prefix)
- **Role name:** Use Developer Name/API Name (not Label)

## Common Patterns

### Pattern: Role Hierarchy
```xml
<sharedTo>
    <roleAndSubordinates>RegionalManager</roleAndSubordinates>
</sharedTo>
```
Includes role AND all subordinate roles.

### Pattern: Multiple Criteria (AND)
```xml
<booleanFilter>1 AND 2</booleanFilter>
<criteriaItems>
    <field>Status__c</field>
    <operation>equals</operation>
    <value>Active</value>
</criteriaItems>
<criteriaItems>
    <field>Priority__c</field>
    <operation>equals</operation>
    <value>High</value>
</criteriaItems>
```

### Pattern: Multiple Criteria (OR)
```xml
<booleanFilter>1 OR 2</booleanFilter>
<criteriaItems>
    <field>Priority__c</field>
    <operation>equals</operation>
    <value>High</value>
</criteriaItems>
<criteriaItems>
    <field>Type__c</field>
    <operation>equals</operation>
    <value>Urgent</value>
</criteriaItems>
```

### Pattern: Account with Related Objects
```xml
<sharingCriteriaRules>
    <fullName>PREFIX_AccountSharing</fullName>
    <accessLevel>Edit</accessLevel>
    <accountSettings>
        <caseAccessLevel>Read</caseAccessLevel>
        <contactAccessLevel>Read</contactAccessLevel>
        <opportunityAccessLevel>None</opportunityAccessLevel>
    </accountSettings>
    <label>Account Sharing</label>
    <!-- ... criteria ... -->
</sharingCriteriaRules>
```
Only for Account objects. Controls related object access.

## Common Errors

### Error: ControlledByParent
```
Cannot create sharing rules for object with sharing model ControlledByParent
```
**Solution:** Don't create sharing rules on child objects. Create them on parent object instead.

### Error: Invalid role/group
```
Invalid role or public group: RoleName
```
**Solution:** 
- Use Developer Name (API name), not Label
- Verify role/group exists in org
- Check for typos

### Error: Invalid field
```
Field [FieldName] does not exist on object [ObjectName]
```
**Solution:**
- Verify field exists
- Add __c suffix to custom fields
- Check spelling

### Error: Boolean filter
```
Invalid boolean filter: References non-existent criterion 4
```
**Solution:**
- Count criteria (starts at 1)
- Filter can only reference existing criteria
- Check for typos in filter

## Deployment

1. **Deploy sharing rules:**
   ```bash
   sf project deploy start -d force-app/main/default/sharingRules
   ```

2. **Verify in org:**
   - Setup → Sharing Settings → [Object] Sharing Rules
   - Check rule appears in list

3. **Recalculate sharing (if needed):**
   - Setup → Sharing Settings → [Object]
   - Click "Recalculate" button

4. **Test access:**
   - Login as user in target role/group
   - Verify can access shared records
   - Check sharing button on record detail page

## Best Practices

1. **Use role hierarchy** when possible (automatic, no rules needed)
2. **Consolidate rules** - Use OR boolean filter instead of multiple similar rules
3. **Use public groups** to share to multiple roles (one rule shared to group)
4. **Test thoroughly** - Login as different users to verify access
5. **Document business logic** - Use clear labels and descriptions
6. **Monitor performance** - Many rules can slow sharing calculations
7. **Defer calculation** - Use "Defer Calculation" when creating multiple rules at once

## Limits

- Maximum **300 sharing rules per object** (combined ownership + criteria + guest)
- Maximum **50 criteria per rule**

## Related Skills

- `sf-object-create` - Create objects before creating sharing rules
- `sf-prefix-detect` - Detect project prefix for naming
- `sf-permission-create` - Create permission sets for user permissions

## Examples

### Example 1: Share by Ownership
**Request:** "Share documents owned by all users to managers"

**Creates:**
- Ownership rule
- sharedFrom: AllUsers group
- sharedTo: Managers role
- Access: Read

### Example 2: Share by Criteria
**Request:** "Share active enforcement cases to the enforcement team"

**Creates:**
- Criteria rule
- Criteria: Status__c equals Active
- sharedTo: EnforcementTeam group
- Access: Edit

### Example 3: Complex Criteria
**Request:** "Share cases where Status is Active AND Priority is High to leadership"

**Creates:**
- Criteria rule with multiple conditions
- Boolean filter: "1 AND 2"
- Two criteriaItems
- sharedTo: Leadership role

## Troubleshooting

**Q: Object doesn't support sharing rules?**
A: Check object's sharing model. If ControlledByParent, create rules on parent object instead.

**Q: Role/group name not working?**
A: Use Developer Name (API name) from Setup, not the Label shown in UI.

**Q: Field criteria not working?**
A: Ensure custom fields have __c suffix and field exists on object.

**Q: Boolean filter error?**
A: Filter can only reference existing criteria by number. Check criteria count matches filter.

**Q: Multiple rules on same object?**
A: Yes! Multiple rules can exist in same file. All are evaluated (additive access).

---

**For detailed documentation, see:** `SKILL.md`
**For test cases, see:** `TEST_CASES.md`

**Version:** 1.0.0
**Author:** Claude Code
