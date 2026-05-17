# Salesforce Roles, Profiles & Permission Sets — SKILL

## When to use this skill
Read this file at the start of any task involving:
- Role Hierarchy creation or modification
- Profile creation, editing, deployment
- Permission Sets and Permission Set Groups
- Permission Set Licenses (PSLs)
- "User can't do X" investigations
- Any new object/field/app requiring access setup

> ⚠️ **NOPSS Architecture note:** אם הפרויקט הוא Non-PSS (לא משתמש ב-Employee2 ואין PSS Package), **סעיפי PSL, `Public_Sector_Solutions_Admin`, `Industries`, `Employee2`, ו-OmniStudio אינם רלוונטיים**. עובדים מיוצגים דרך Person Account ישירות. ראה סעיף 7 להרשאות Person Account.

---

## 1. The Access Model — Three Layers

```
Layer 1: PROFILE (mandatory — every user has exactly one)
         → Object CRUD, FLS, Tab visibility, App access, Login hours, IP restrictions
         → Baseline access that applies to ALL users with this profile

Layer 2: PERMISSION SET (additive — user can have many)
         → Same as Profile but additive — can only ADD permissions, never remove
         → Used for role-specific extras without changing the Profile

Layer 3: PERMISSION SET GROUP (bundle of Permission Sets)
         → Assign one group instead of many individual PS
         → Easier management for complex access combinations
```

> ⚠️ **Permission Sets can only ADD to Profile permissions.** If Profile says No Read on an object, a PS cannot grant Read. (Exception: PS can grant object permissions not in Profile.)
> ⚠️ **Actually false for objects:** PS CAN grant object access even if Profile has none. Only field-level access follows Profile baseline.

---

## 2. Role Hierarchy

### What Roles Do

- Determine record visibility (when OWD is Private or Public Read Only)
- Users above in hierarchy see records owned by users below
- Roles are NOT the same as Profiles — a role is about data access, a profile is about feature access

### This Org's Role Hierarchy

```
MOH_Root                    ← System Admin / top-level
  ├── HR_Manager            ← HR team
  └── Finance               ← Finance team
```

### Role Metadata

```
force-app/main/default/roles/
  MOH_Root.role-meta.xml
  HR_Manager.role-meta.xml
  Finance.role-meta.xml
```

```xml
<!-- HR_Manager.role-meta.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<Role xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>HR_Manager</fullName>
    <label>HR Manager</label>
    <parentRole>MOH_Root</parentRole>         <!-- omit for top-level role -->
    <caseAccessLevel>Edit</caseAccessLevel>   <!-- None | Read | Edit -->
    <contactAccessLevel>Edit</contactAccessLevel>
    <opportunityAccessLevel>Edit</opportunityAccessLevel>
    <mayForecastManagerShare>false</mayForecastManagerShare>
</Role>
```

> ⚠️ Roles are usually managed in Setup UI, not via metadata in most orgs. If deploying via metadata, include in `package.xml` under `Role` type.

### Assigning Role to User (Apex / Data Load)

```apex
User u = [SELECT Id FROM User WHERE Username = 'hr@moh.gov.il' LIMIT 1];
UserRole r = [SELECT Id FROM UserRole WHERE DeveloperName = 'HR_Manager' LIMIT 1];
u.UserRoleId = r.Id;
update u;
```

---

## 3. Profiles

### Profile File Structure

```
force-app/main/default/profiles/
  HR_Manager.profile-meta.xml
  System Administrator.profile-meta.xml
  Finance_ReadOnly.profile-meta.xml
```

### Profile XML — Complete Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Profile xmlns="http://soap.sforce.com/2006/04/metadata">

    <!-- ═══ APP ACCESS ═══ -->
    <applicationVisibilities>
        <application>HR_App</application>
        <default>true</default>
        <visible>true</visible>
    </applicationVisibilities>

    <!-- ═══ TAB VISIBILITY ═══ -->
    <tabVisibilities>
        <tab>Employment__c</tab>
        <visibility>DefaultOn</visibility>   <!-- DefaultOn | DefaultOff | Hidden -->
    </tabVisibilities>
    <tabVisibilities>
        <tab>standard-InternalOrganizationUnit</tab>
        <visibility>DefaultOn</visibility>
    </tabVisibilities>
    <tabVisibilities>
        <tab>standard-report</tab>
        <visibility>DefaultOn</visibility>
    </tabVisibilities>

    <!-- ═══ OBJECT PERMISSIONS ═══ -->
    <objectPermissions>
        <allowCreate>true</allowCreate>
        <allowDelete>true</allowDelete>       <!-- required if modifyAllRecords=true -->
        <allowEdit>true</allowEdit>
        <allowRead>true</allowRead>
        <modifyAllRecords>false</modifyAllRecords>
        <object>Employment__c</object>
        <viewAllRecords>false</viewAllRecords>
    </objectPermissions>

    <!-- ═══ FIELD PERMISSIONS (FLS) ═══ -->
    <!-- Only optional fields — never required, MasterDetail, or Formula -->
    <fieldPermissions>
        <editable>true</editable>
        <field>Employment__c.Hourly_Rate__c</field>
        <readable>true</readable>
    </fieldPermissions>
    <fieldPermissions>
        <editable>false</editable>
        <field>Employment__c.Confidential_Notes__c</field>
        <readable>true</readable>
    </fieldPermissions>

    <!-- ═══ LAYOUT ASSIGNMENTS ═══ -->
    <layoutAssignments>
        <layout>Employment__c-Employment Layout</layout>
        <object>Employment__c</object>
    </layoutAssignments>
    <!-- With RecordType: -->
    <layoutAssignments>
        <layout>Employment__c-Vendor Layout</layout>
        <object>Employment__c</object>
        <recordType>Employment__c.Vendor</recordType>
    </layoutAssignments>

    <!-- ═══ RECORD TYPE VISIBILITY ═══ -->
    <recordTypeVisibilities>
        <default>true</default>
        <recordType>Employment__c.Standard</recordType>
        <visible>true</visible>
    </recordTypeVisibilities>

    <!-- ═══ PAGE ACCESS (Visualforce — rarely needed) ═══ -->
    <!-- <pageAccesses>
        <apexPage>MyVFPage</apexPage>
        <enabled>true</enabled>
    </pageAccesses> -->

    <!-- ═══ APEX CLASS ACCESS ═══ -->
    <classAccesses>
        <apexClass>EmploymentController</apexClass>
        <enabled>true</enabled>
    </classAccesses>

    <!-- ═══ USER LICENSE ═══ -->
    <userLicense>Salesforce</userLicense>

</Profile>
```

### FLS Rules — What to Include

| Field Type | Include in fieldPermissions? | Reason |
|---|---|---|
| Optional custom field | ✅ Yes | Needs explicit FLS |
| Required custom field | ❌ No | Deploy error: "cannot deploy to required field" |
| MasterDetail field | ❌ No | Always required — implicit access |
| Formula field | ❌ No | Read-only, always visible |
| Standard required field (Name, etc.) | ❌ No | Always accessible |

### Tab Visibility Values

| Value | Meaning |
|---|---|
| `DefaultOn` | Visible by default, user can hide |
| `DefaultOff` | Hidden by default, user can show |
| `Hidden` | Always hidden, user cannot change |

### Profile Naming Convention

| Profile | Purpose |
|---|---|
| `HR_Manager` | Full HR access — Employment, Employee2, Feedback |
| `Finance_ReadOnly` | Read-only access to financial objects |
| `System Administrator` | Full org access (standard — do not rename) |

---

## 4. Permission Sets

### When to Use Permission Set vs. Profile

| Use Profile for | Use Permission Set for |
|---|---|
| Baseline access all users of this type need | Extra access specific users need |
| Tab visibility, App default | Temporary elevated access |
| Login hours, IP restrictions | Feature-specific access (Shield, PSS, OmniStudio) |
| Layout assignments | Access that crosses profile types |

### Permission Set Metadata

```
force-app/main/default/permissionsets/
  PSS_HR_Access.permissionset-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PermissionSet xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>PSS_HR_Access</fullName>
    <description>Grants access to PSS HR objects for HR managers</description>
    <label>PSS HR Access</label>
    <license>Industries</license>   <!-- Link to PSL — omit if no PSL required -->

    <!-- Object permissions -->
    <objectPermissions>
        <allowCreate>true</allowCreate>
        <allowDelete>false</allowDelete>
        <allowEdit>true</allowEdit>
        <allowRead>true</allowRead>
        <modifyAllRecords>false</modifyAllRecords>
        <object>Annual_Feedback__c</object>
        <viewAllRecords>false</viewAllRecords>
    </objectPermissions>

    <!-- Field permissions -->
    <fieldPermissions>
        <editable>true</editable>
        <field>Annual_Feedback__c.Score__c</field>
        <readable>true</readable>
    </fieldPermissions>

    <!-- Apex class access -->
    <classAccesses>
        <apexClass>FeedbackController</apexClass>
        <enabled>true</enabled>
    </classAccesses>

</PermissionSet>
```

### Assigning Permission Set to User

```apex
// Apex
PermissionSet ps = [SELECT Id FROM PermissionSet WHERE Name = 'PSS_HR_Access' LIMIT 1];
User u = [SELECT Id FROM User WHERE Username = 'hr@moh.gov.il' LIMIT 1];
insert new PermissionSetAssignment(PermissionSetId = ps.Id, AssigneeId = u.Id);
```

```bash
# CLI
sf org assign permset --name PSS_HR_Access --on-behalf-of hr@moh.gov.il --target-org moh-sandbox
```

---

## 5. Permission Set Licenses (PSLs)

### What PSLs Are

PSLs are purchased add-ons that unlock specific platform features. A Permission Set that grants PSL-dependent features only works if the PSL exists in the org AND is assigned to the user.

### PSLs in This Org

| PSL Name | API Name | Who Needs It | What It Unlocks |
|---|---|---|---|
| Industries | `Industries` | All HR managers | Employee2, Employment, InternalOrganizationUnit, PersonLifeEvent |
| OmniStudio User | `OmniStudioUser` | All HR managers | FlexCards on Record Pages |
| OmniStudio Admin | `OmniStudioAdmin` | System Admin only | FlexCard Designer, DataRaptor, Integration Procedures |
| Salesforce Shield | `Shield` | Admin + HR managers | View Encrypted Data, manage Encryption Policy |

### Assigning PSL — Two Ways

**Via Permission Set (recommended):**
When you assign a Permission Set that is linked to a PSL (`<license>` tag), Salesforce automatically assigns the PSL to the user — IF the PSL exists in the org.

**Directly:**
```apex
PermissionSetLicense psl = [SELECT Id FROM PermissionSetLicense 
                             WHERE DeveloperName = 'Industries' LIMIT 1];
User u = [SELECT Id FROM User WHERE Username = 'hr@moh.gov.il' LIMIT 1];
insert new PermissionSetLicenseAssign(PermissionSetLicenseId = psl.Id, AssigneeId = u.Id);
```

> ⚠️ **If the PSL is not purchased in the org, the Permission Set assignment will fail silently or throw an error.** Always verify PSL exists before assigning.
> ⚠️ **`Employee_Experience_User` PSL** — for employee portal (Experience Cloud) ONLY. Do NOT assign to internal HR managers.

### Mandatory Permission Sets for This Org

| Permission Set | Who Gets It | Why |
|---|---|---|
| `Public_Sector_Solutions_Admin` | System Admin | Makes PSS objects visible |
| `PublicSectorAccessPSL` | System Admin | PSS object access |
| `Industries` (linked PS) | All HR managers | PSS data access |
| `OmniStudio User` (linked PS) | All HR managers | FlexCards |
| `Shield` (linked PS) | Admin + HR managers | Encrypted field access |

> ⚠️ Without `Public_Sector_Solutions_Admin` + `PublicSectorAccessPSL` on Admin, PSS objects are invisible in Setup, describe calls, and SOQL — they appear non-existent.

---

## 6. Permission Set Groups

### What They Are

A bundle of Permission Sets deployed and assigned as one unit.

```
force-app/main/default/permissionsetgroups/
  HR_Manager_Access.permissionsetgroup-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PermissionSetGroup xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>HR_Manager_Access</fullName>
    <description>Full access bundle for HR managers</description>
    <label>HR Manager Access</label>
    <permissionSets>PSS_HR_Access</permissionSets>
    <permissionSets>OmniStudio_User_Access</permissionSets>
    <permissionSets>Shield_User_Access</permissionSets>
    <mutingPermissionSets>Remove_Delete_Access</mutingPermissionSets>   <!-- optional: mute specific permissions -->
</PermissionSetGroup>
```

---

## 7. New Object/Field — Permissions Checklist

> ⚠️ **Every new object or field must be followed by a permissions deploy in the same task.**

**New Custom Object:**
- [ ] `objectPermissions` added to every relevant Profile
- [ ] `tabVisibilities` added (if Tab created)
- [ ] `layoutAssignments` added
- [ ] `recordTypeVisibilities` added (if Record Types exist)
- [ ] Relevant Permission Sets updated

**New Optional Field:**
- [ ] `fieldPermissions` added to every relevant Profile (readable + editable)
- [ ] `fieldPermissions` added to relevant Permission Sets

**New App:**
- [ ] `applicationVisibilities` added to every relevant Profile

**New Apex Class:**
- [ ] `classAccesses` added to relevant Profiles / Permission Sets

### ⚠️ Person Account — חובה הרשאות Account AND Contact

כאשר הישות "עובד" מיוצגת דרך **Person Account** (ולא Employee2), פרופיל חייב לכלול `objectPermissions` גם על **Account** וגם על **Contact**. הרשאת Account בלבד אינה מספיקה.

```xml
<objectPermissions>
    <allowCreate>true</allowCreate>
    <allowDelete>false</allowDelete>
    <allowEdit>true</allowEdit>
    <allowRead>true</allowRead>
    <modifyAllRecords>false</modifyAllRecords>
    <object>Account</object>
    <viewAllRecords>true</viewAllRecords>
</objectPermissions>
<!-- Contact permissions required for Person Account access -->
<objectPermissions>
    <allowCreate>false</allowCreate>
    <allowDelete>false</allowDelete>
    <allowEdit>false</allowEdit>
    <allowRead>true</allowRead>
    <modifyAllRecords>false</modifyAllRecords>
    <object>Contact</object>
    <viewAllRecords>true</viewAllRecords>
</objectPermissions>
```

> לפחות `allowRead=true` על Contact נדרש כדי ש-Salesforce יאפשר גישה ל-Person Account בכלל.

---

## 8. Troubleshooting — "User Can't Do X"

| Symptom | Likely Cause | Fix |
|---|---|---|
| "Insufficient Privileges" on object | No `objectPermissions` | Add to Profile or PS |
| Field visible but not editable | FLS `editable=false` | Set `editable=true` in Profile |
| Field not visible at all | FLS `readable=false` | Set `readable=true` in Profile |
| Tab not visible | `tabVisibilities` missing or `Hidden` | Add/change `tabVisibilities` |
| App not in App Launcher | `applicationVisibilities` missing | Add to Profile |
| Can't create a RecordType | `recordTypeVisibilities` missing | Add to Profile |
| PSS objects not visible | PSS Permission Sets not assigned | Assign `Public_Sector_Solutions_Admin` + `PublicSectorAccessPSL` |
| FlexCard not loading | OmniStudio PSL not assigned | Assign OmniStudio User PS |
| Encrypted field shows masked | Shield PSL not assigned | Assign Shield PS |

---

## 9. Common Mistakes

| Mistake | What Happens | Fix |
|---|---|---|
| Adding required field to FLS | Deploy error | Remove from fieldPermissions |
| `modifyAllRecords=true` without `allowDelete=true` | Deploy error | Add `allowDelete=true` |
| Two apps with `default=true` in same Profile | Deploy error | Only one default app per Profile |
| Two RecordTypes with `default=true` for same object in Profile | Deploy error | Only one default RT per object per Profile |
| Deploying PS that references PSL not in org | Silent fail or error | Verify PSL purchased first |
| Assigning `Employee_Experience_User` to internal users | Wrong license consumed | Only for Experience Cloud portal users |
| Profile deployed without all objects it references | Deploy error on missing objects | Deploy objects first, Profile last |
| Forgetting classAccesses for new Apex used by LWC | LWC Apex call throws "not authorized" | Add classAccesses to Profile/PS |

---

## 10. Cross-Domain Interactions (Critical — Often Missed)

### FLS + Sharing — They Are Independent
- **Sharing Rules** determine which records a user can access (record-level)
- **FLS** determines which fields a user can see on those records (field-level)
- Both must be satisfied independently:
  - Sharing Rule grants Record Read + FLS grants Field Read → user sees field value ✅
  - Sharing Rule grants Record Read + FLS denies Field Read → user sees the record but field is blank ⚠️
  - Sharing Rule denies Record Read + FLS grants Field Read → user cannot access the record at all ❌
- Never assume Sharing Rule access implies field access

### Changing a Field to Required — Profile FLS Cascade
- If a field is currently Optional and has FLS entries in Profile metadata (`<fieldPermissions>`) → changing it to Required causes **deploy error**
- Required fields must NOT appear in `<fieldPermissions>` in Profile metadata
- Checklist when making a field required:
  1. Remove field from `<fieldPermissions>` in all Profile metadata files
  2. Remove field from Permission Set `<fieldPermissions>` if present
  3. Ensure all Bulk Load CSV files include this field (required field cannot be empty)
  4. Update any Flows/Apex that create records — must populate this field

### Permission Set Cannot Reduce Profile Permissions
- Permission Sets are **additive only**: they can grant additional access, never remove access granted by Profile
- Exception: **object-level access** — a PS CAN grant access to an object that the Profile has no access to
- Field-level: if Profile grants `readable=true` on a field, no PS can remove that
- If a user needs LESS access than their Profile → assign a different Profile; Permission Sets cannot restrict

### Apex Class Access (classAccesses)
- Every Apex class used by LWC (`@AuraEnabled`) must be in the Profile's `<classAccesses>`:
  ```xml
  <classAccesses>
      <apexClass>EmployeeController</apexClass>
      <enabled>true</enabled>
  </classAccesses>
  ```
- If missing → LWC Apex call throws "Method is not visible" at runtime (not at deploy time)
- Always add `classAccesses` to Profile AND relevant Permission Sets when creating a new Apex controller

### FLS + Encrypted Fields
- Encrypted field is readable to a user if their Profile has FLS `readable=true` on that field
- No separate "ViewEncryptedData" permission is required (as of Summer '22 / API v55+)
- If FLS `readable=false` → encrypted field returns null, same as any other unreadable field
- `editable=true` required for users who need to modify the encrypted field value

### Permission Set Assignment Order (PSL Dependency)
```
Correct order:
1. Assign required PSL (e.g., PublicSectorAccessPSL, OmniStudio User PSL, Shield PSL)
2. Assign Permission Set Group
3. Assign individual Permission Sets

If PSL missing → PSS objects invisible in Setup, SOQL returns 0 rows, describe returns no fields
```
