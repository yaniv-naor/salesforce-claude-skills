# Salesforce Sharing & OWD — SKILL

## When to use this skill
Read this file at the start of any task involving:
- OWD (Organization-Wide Defaults) — sharingModel on objects
- Role Hierarchy — who sees whose records
- Sharing Rules — opening access beyond OWD
- Public Groups & Queues — grouping users for sharing
- Manual Sharing, Apex Managed Sharing
- Permission Set / Profile — View All / Modify All
- Any "user can't see a record" or "access denied" investigation

---

## 1. The Sharing Model — How It Works

Salesforce determines record access by evaluating layers **in order**. The first layer that grants access wins. Access can only be **expanded** going down — never restricted.

```
Layer 1: Object Permissions (Profile / Permission Set)
         → Can this user access this object type at all?
         → If NO object permission → stop. No access.

Layer 2: OWD (Organization-Wide Default)
         → What is the baseline access for records this user does NOT own?

Layer 3: Role Hierarchy
         → Users above in hierarchy see records of users below them (if OWD allows)

Layer 4: Sharing Rules
         → Open access for specific groups/roles beyond OWD

Layer 5: Manual Sharing
         → Record owner manually shares with specific user/group

Layer 6: Apex Managed Sharing
         → Code-driven sharing (custom logic)

Layer 7: View All / Modify All (Profile or Permission Set)
         → Bypass all sharing — full access to all records of this object
```

> ⚠️ **Common mistake:** Giving a user object permissions but leaving OWD=Private without a Sharing Rule → user sees only their own records. Always design OWD + Sharing Rules together.

---

## 2. OWD — Organization-Wide Defaults

### What OWD Controls

OWD sets the **default access** for records a user does NOT own.

| OWD Value | Who can Read | Who can Edit | Notes |
|---|---|---|---|
| `Public Read/Write` | Everyone | Everyone | Most open — avoid for sensitive data |
| `Public Read Only` | Everyone | Owner + Role above + explicit share | Good for reference data |
| `Private` | Owner + Role above + explicit share | Owner + Role above + explicit share | Most restrictive — HR data |
| `ControlledByParent` | Follows parent record | Follows parent record | Master-Detail only |

### Setting OWD in Object Metadata

```xml
<!-- In CustomObject XML -->
<sharingModel>Private</sharingModel>
<sharingModel>Read</sharingModel>          <!-- = Public Read Only -->
<sharingModel>ReadWrite</sharingModel>     <!-- = Public Read/Write -->
<sharingModel>ControlledByParent</sharingModel>
```

> ⚠️ **`Read` in XML = "Public Read Only" in Setup UI.** These are different names for the same value.

### Recommended OWD for this Org

| Object | OWD (XML) | UI Label | Reason |
|---|---|---|---|
| `Employee2` / Person Account | `Private` | Private | Sensitive employee data |
| `Employment__c` | `Private` | Private | HR controlled — use Sharing Rules |
| `Annual_Feedback__c` | `Private` | Private | Sensitive — employee sees own only |
| `Tariff_Line__c` | `ControlledByParent` | Controlled By Parent | Master-Detail under Employment |
| `Monthly_Payment_Control__c` | `ControlledByParent` | Controlled By Parent | Master-Detail under Employment |
| `InternalOrganizationUnit` | `Read` | Public Read Only | Org structure visible to all |
| `Job_Catalog__c` | `Read` | Public Read Only | Everyone reads, HR writes |
| `PurchaseOrder__c` | `Private` | Private | Finance/HR controlled |
| `Account` (Vendor RT) | `Private` | Private | Vendor data — restricted |

### Changing OWD — Risks

> ⚠️ **Changing OWD to more restrictive (e.g., ReadWrite → Private) on an org with existing data immediately blocks access to records users previously saw.** Always:
> 1. Prepare Sharing Rules BEFORE changing OWD
> 2. Deploy Sharing Rules first
> 3. Then change OWD
> 4. Verify access immediately after

> ⚠️ **OWD cannot be set via Metadata API alone for standard objects** (Account, Contact, etc.). For PSS objects and custom objects — use `<sharingModel>` in the object XML.

---

## 3. Role Hierarchy

### What Role Hierarchy Does

When OWD is `Private` or `Public Read Only`:
- A user in a **higher** role automatically sees records owned by users **below** them in the hierarchy
- A user in a **lower** role does NOT see records of users above or in sibling roles

### This Org's Role Hierarchy

```
MOH_Root
  ├── HR_Manager
  └── Finance
```

> Records owned by a user with role `HR_Manager` are visible to `MOH_Root` users automatically (via hierarchy), even with OWD=Private.

### Role Hierarchy Does NOT Apply When

- `ControlledByParent` OWD — sharing follows parent record, not hierarchy
- Permission Set has `View All` on the object — hierarchy is irrelevant
- The object has `Grant Access Using Hierarchies = false` (custom objects only, rare)

### Disabling Hierarchy for Custom Objects

```xml
<!-- In CustomObject XML — RARELY used, only for truly flat access models -->
<enableSharing>true</enableSharing>
<!-- Add this to disable role hierarchy grant: -->
<sharingModel>Private</sharingModel>
<!-- Note: "Grant Access Using Hierarchies" checkbox in Setup — cannot be set via metadata -->
```

---

## 4. Sharing Rules

### Core Concepts

- Sharing Rules can only **open** access — never restrict
- OWD is the floor — Sharing Rules can only go above it
- Two types: **Ownership-based** (by record owner) and **Criteria-based** (by field value)
- `accessLevel` must be **greater than or equal to** OWD level

### File Structure

```
force-app/main/default/sharingRules/
  Employment__c.sharingRules-meta.xml        ← filename = object API name
  Annual_Feedback__c.sharingRules-meta.xml
  Job_Catalog__c.sharingRules-meta.xml
```

> ⚠️ **One file per object.** All rules for the same object go in the same file.
> ⚠️ **Filename must exactly match the object API name** including `__c`.
> ⚠️ For PSS standard objects (e.g. `Employee2`) — filename is `Employee2.sharingRules-meta.xml` (no `__c`).

### Deploy Command

```bash
sf project deploy start --metadata "SharingRules:Employment__c" --target-org moh-sandbox
```

### Ownership-Based Sharing Rule

Shares records based on **who owns** the record.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SharingRules xmlns="http://soap.sforce.com/2006/04/metadata">
    <sharingOwnerRules>
        <fullName>Share_Employment_with_HR_Managers</fullName>
        <accessLevel>Edit</accessLevel>        <!-- Read | Edit -->
        <label>Share Employment with HR Managers</label>
        <sharedTo>
            <roleAndSubordinates>HR_Manager</roleAndSubordinates>
        </sharedTo>
        <sharedFrom>
            <allInternalUsers/>
        </sharedFrom>
    </sharingOwnerRules>
</SharingRules>
```

### Criteria-Based Sharing Rule

Shares records based on **field value** on the record itself.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SharingRules xmlns="http://soap.sforce.com/2006/04/metadata">
    <sharingCriteriaRules>
        <fullName>Share_Active_Employment</fullName>
        <accessLevel>Read</accessLevel>
        <label>Share Active Employments</label>
        <booleanFilter>1</booleanFilter>       <!-- "1" = first criteriaItem. "1 AND 2" for multiple -->
        <criteriaItems>
            <field>Status__c</field>
            <operation>equals</operation>
            <value>Active</value>              <!-- API Value — not Hebrew label -->
        </criteriaItems>
        <sharedTo>
            <group>PG_HR_Managers</group>
        </sharedTo>
    </sharingCriteriaRules>
</SharingRules>
```

### Multiple Rules in One File

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SharingRules xmlns="http://soap.sforce.com/2006/04/metadata">

    <!-- Rule 1: Ownership-based -->
    <sharingOwnerRules>
        <fullName>Share_Employment_HR</fullName>
        <accessLevel>Edit</accessLevel>
        <label>Share Employment with HR</label>
        <sharedTo><roleAndSubordinates>HR_Manager</roleAndSubordinates></sharedTo>
        <sharedFrom><allInternalUsers/></sharedFrom>
    </sharingOwnerRules>

    <!-- Rule 2: Criteria-based -->
    <sharingCriteriaRules>
        <fullName>Share_Active_Employment_Finance</fullName>
        <accessLevel>Read</accessLevel>
        <label>Share Active Employment with Finance</label>
        <booleanFilter>1</booleanFilter>
        <criteriaItems>
            <field>Status__c</field>
            <operation>equals</operation>
            <value>Active</value>
        </criteriaItems>
        <sharedTo><role>Finance</role></sharedTo>
    </sharingCriteriaRules>

</SharingRules>
```

### sharedTo / sharedFrom — All Options

```xml
<!-- Specific Role only (not subordinates) -->
<role>HR_Manager</role>

<!-- Role + all roles below it in hierarchy -->
<roleAndSubordinates>HR_Manager</roleAndSubordinates>

<!-- Role + subordinates + portal users below (if Experience Cloud exists) -->
<roleAndSubordinatesInternal>HR_Manager</roleAndSubordinatesInternal>

<!-- Public Group -->
<group>PG_HR_Managers</group>

<!-- Queue -->
<queue>Q1</queue>

<!-- All internal users -->
<allInternalUsers/>

<!-- All portal users (Experience Cloud) -->
<allCustomerPortalUsers/>

<!-- Current record owner -->
<!-- (only valid in sharedFrom, not sharedTo) -->
```

### accessLevel Rules

| OWD | Minimum accessLevel in Sharing Rule |
|---|---|
| `Private` | `Read` or `Edit` |
| `Public Read Only` | `Edit` only (Read would be same as OWD — pointless) |
| `Public Read/Write` | Sharing Rules have no effect — already maximum |
| `ControlledByParent` | Sharing Rules not supported |

> ⚠️ **`accessLevel` must be HIGHER than OWD.** If OWD=Private, both Read and Edit are valid. If OWD=Read, only Edit is valid.

### criteriaItems — Operations

| Operation | Meaning |
|---|---|
| `equals` | Field = value |
| `notEqual` | Field ≠ value |
| `contains` | Text contains value |
| `notContain` | Text does not contain value |
| `startsWith` | Text starts with value |
| `greaterThan` | Number/Date > value |
| `greaterOrEqual` | Number/Date >= value |
| `lessThan` | Number/Date < value |
| `lessOrEqual` | Number/Date <= value |

### booleanFilter — Multiple Criteria

```xml
<!-- Two criteria with AND -->
<booleanFilter>1 AND 2</booleanFilter>
<criteriaItems>
    <field>Status__c</field>
    <operation>equals</operation>
    <value>Active</value>
</criteriaItems>
<criteriaItems>
    <field>Employment_Type__c</field>
    <operation>equals</operation>
    <value>FullTime</value>
</criteriaItems>

<!-- Two criteria with OR -->
<booleanFilter>1 OR 2</booleanFilter>

<!-- Complex: (1 AND 2) OR 3 -->
<booleanFilter>(1 AND 2) OR 3</booleanFilter>
```

---

## 5. Public Groups & Queues

### Public Groups — What They Are

A Public Group is a named collection of users, roles, or other groups that can be referenced in Sharing Rules, List Views, and manual sharing.

### Discovering Groups and Roles in the Org

```apex
// Run in Developer Console → Execute Anonymous
for(UserRole r : [SELECT DeveloperName, Name FROM UserRole ORDER BY Name]) {
    System.debug('ROLE: ' + r.DeveloperName + ' | ' + r.Name);
}
for(Group g : [SELECT DeveloperName, Name, Type FROM Group ORDER BY Type, Name]) {
    System.debug('GROUP [' + g.Type + ']: ' + g.DeveloperName + ' | ' + g.Name);
}
```

**This org's Roles:** `MOH_Root`, `HR_Manager`, `Finance`
**This org's Public Groups:** `PG_HR_Managers`
**This org's Queues:** `Q1`

### Public Group Metadata

```
force-app/main/default/groups/
  PG_HR_Managers.group-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Group xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>PG_HR_Managers</fullName>
    <label>HR Managers</label>
    <doesIncludeBosses>false</doesIncludeBosses>   <!-- true = include role hierarchy above members -->
    <members>
        <member>HR_Manager</member>
        <type>Role</type>                          <!-- Role | RoleAndSubordinates | User | Group | Queue -->
    </members>
</Group>
```

> ⚠️ Public Groups are **not** deployed automatically — they must be in `package.xml` and deployed explicitly.
> ⚠️ If a Sharing Rule references a Group that doesn't exist → deploy error.
> **Always deploy Groups before Sharing Rules.**

### Deploy Order for Sharing

```
1. Roles (UserRole — managed via Setup, not metadata in most cases)
2. Public Groups
3. OWD changes (sharingModel in CustomObject)
4. Sharing Rules
```

---

## 6. View All / Modify All

### When to Use vs. Sharing Rules

| Mechanism | Use when |
|---|---|
| Sharing Rules | Most users — controlled access based on ownership or criteria |
| `View All` on Profile/PS | Admin-level users who need to see ALL records regardless |
| `Modify All` on Profile/PS | Admin-level users who need to edit/delete ALL records |

### In Profile XML

```xml
<objectPermissions>
    <allowCreate>true</allowCreate>
    <allowDelete>true</allowDelete>     <!-- required if modifyAllRecords=true -->
    <allowEdit>true</allowEdit>
    <allowRead>true</allowRead>
    <modifyAllRecords>true</modifyAllRecords>   <!-- bypasses ALL sharing for this object -->
    <object>Employment__c</object>
    <viewAllRecords>true</viewAllRecords>
</objectPermissions>
```

> ⚠️ `modifyAllRecords=true` requires `allowDelete=true`. Without it: deploy error.
> ⚠️ `View All` / `Modify All` bypass Sharing Rules, OWD, and Role Hierarchy completely.
> Use only for System Admin and HR Admin profiles — not regular HR managers.

### In Permission Set

Same XML structure as Profile. Assign Permission Set to specific users who need elevated access without changing their Profile.

---

## 7. Manual Sharing

Manual Sharing is done via the UI (Share button on record) or via Apex. It cannot be deployed via metadata.

**When it's useful:**
- One-off exceptions — a record that needs to be shared with a specific user
- Not a systemic pattern — if you need it for many records, use Sharing Rules instead

**Via Apex (for automation):**
```apex
// Share an Employment record with a specific User
Employment__Share share = new Employment__Share();
share.ParentId = employmentId;
share.UserOrGroupId = targetUserId;
share.AccessLevel = 'Read';           // Read | Edit
share.RowCause = Schema.Employment__Share.RowCause.Manual;
insert share;
```

> ⚠️ Apex Managed Sharing requires a custom `RowCause` (Sharing Reason) defined on the object. `Manual` is built-in. For custom sharing reasons, define them in the object metadata.

---

## 8. ControlledByParent — Master-Detail Sharing

When a child object has `ControlledByParent` OWD:
- The child record's access is **identical** to its parent's access
- No Sharing Rules can be defined on the child object
- No manual sharing on the child object
- If you can see/edit the parent → you can see/edit the child

**Implications for this org:**

| Child Object | Parent | Access follows |
|---|---|---|
| `Tariff_Line__c` | `Employment__c` | If user can see the Employment → sees all its Tariff Lines |
| `Monthly_Payment_Control__c` | `Employment__c` | Same |
| `Annual_Feedback__c` | `Employment__c` (if MD) | Same — IF set as Master-Detail |

> ⚠️ `Annual_Feedback__c` is currently `Private` (Lookup, not Master-Detail). If it becomes Master-Detail — change `sharingModel` to `ControlledByParent` and remove Sharing Rules.

---

## 9. Troubleshooting — "User Can't See Record"

Step-by-step diagnosis:

```
1. Does the user have Object Permission (Read) on this object?
   → Check Profile + all assigned Permission Sets
   → If NO → add objectPermissions

2. What is the OWD for this object?
   → Setup → Sharing Settings → [Object Name]
   → If Private → proceed to step 3

3. Does the user own the record?
   → If YES → should see it. If not seeing it → object permission issue (step 1)
   → If NO → proceed to step 4

4. Is the record owner in a Role BELOW the user in hierarchy?
   → If YES → user should see it via hierarchy
   → If NO → proceed to step 5

5. Is there a Sharing Rule that covers this user + this record?
   → Check sharingRules file for this object
   → Verify the rule's criteria match the record's field values (for criteria-based)
   → Verify the rule's sharedTo includes this user's role/group
   → If NO matching rule → add one

6. Does the user have View All / Modify All on this object?
   → If YES → should see everything. If still not → object permission issue

7. Is the object ControlledByParent?
   → Check if user can see the PARENT record
   → If can't see parent → fix parent sharing first
```

---

## 10. Common Mistakes

| Mistake | What Happens | Fix |
|---|---|---|
| Sharing Rule `accessLevel` same as OWD | Rule has no effect | accessLevel must be higher than OWD |
| Sharing Rule on `ControlledByParent` object | Deploy error | Remove rule — sharing follows parent |
| Group referenced in rule doesn't exist | Deploy error | Deploy Group before Sharing Rules |
| `modifyAllRecords=true` without `allowDelete=true` | Deploy error | Add `allowDelete=true` |
| Changed OWD to Private without preparing Sharing Rules | Users lose access immediately | Prepare rules BEFORE changing OWD |
| Criteria-based rule uses Hebrew picklist label | Rule never matches | Use API value (e.g. `Active` not `פעיל`) |
| Multiple Sharing Rules files for same object | Deploy conflict | One file per object — merge all rules |
| Forgot to include `sharingRules` in `package.xml` | Rules not deployed | Add `<members>ObjectName__c</members>` under `SharingRules` type |
| `Read` in XML vs "Public Read Only" in UI | Confusion | Same thing — `Read` in XML = "Public Read Only" in UI |

---

## 11. Cross-Domain Interactions (Critical — Often Missed)

### Sharing + FLS — Independent Axes
- A Sharing Rule grants access to a **record** — it does not grant access to **fields** on that record
- User A has Sharing Rule granting Read on Employee record → but if FLS denies Read on `Salary__c` → `Salary__c` is blank for User A
- Always verify both: record-level (Sharing) and field-level (FLS) for each access requirement

### Encrypted Fields in Criteria-Based Sharing Rules
- **Cannot use an encrypted field as a criteria condition** in a criteria-based Sharing Rule
- Encrypted fields are not available in the Sharing Rule criteria field picker
- Workaround: use a non-encrypted formula field or text field that mirrors the encrypted field's category/status
- Example: cannot filter "where National_ID__c = X" → instead use a Status or Department field as criteria

### Cascade: Changing Lookup to MasterDetail
- If you change a Lookup field to MasterDetail:
  - OWD on the child object automatically switches to `ControlledByParent`
  - All existing Sharing Rules on the child object become invalid and must be removed (or they error on deploy)
  - `deleteConstraint` changes from configurable → always cascade (parent delete cascades to child)
- Before making this change: delete all Sharing Rules on the child object first

### OWD Change — Preparation Checklist
Changing OWD to a MORE restrictive setting (e.g., Public Read Only → Private):
1. **Create Sharing Rules FIRST** (before changing OWD) — otherwise users lose access the moment OWD changes
2. Notify users — access loss is immediate, not on next login
3. Verify Integration User has explicit Sharing Rule access (Integration User may not be in Role Hierarchy)
4. Verify Flows — Record-Triggered Flows run in system context (unaffected), Screen Flows run as user (affected)
5. Verify Reports — all reports on this object will show fewer records for non-admin users
6. Verify Batch Apex — runs as system context unless `Database.QueryLocator` uses `with sharing`

### Integration User and Sharing
- Integration Users typically have no Role (no Role Hierarchy access)
- For objects with OWD = Private: Integration User needs explicit Sharing Rule (e.g., `sharedTo: allInternalUsers` or a dedicated group)
- Without Sharing Rule: Integration User can CREATE records but cannot READ, UPDATE, or DELETE records they don't own
- `View All` / `Modify All` on the Profile bypasses all sharing — use this for Integration Profiles on private objects

### Manual Sharing — Not Deployable via Metadata
- Manual Sharing (sharing a specific record with a specific user) cannot be deployed via Metadata API
- Must be done via Apex (`Database.insert(new ObjectShare(...))`) or via UI
- Recalculating sharing: `Database.convertLead()` or `System.resetSharingRules()` — use carefully in production
