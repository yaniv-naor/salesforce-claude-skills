# Salesforce Objects — Standard & Custom SKILL

## When to use this skill
Read this file at the start of any task involving:
- Creating or modifying Custom Objects
- Working with Standard Objects (Account, Contact, Case, etc.)
- Field types, validation, relationships
- Object metadata XML structure
- PSS objects in context of standard Salesforce objects

---

## 1. Object Types — Decision Tree

```
Need to store data?
│
├── Does PSS already have it? → Check SKILL_PSS_SF.md Section 1 first
│     Employee2, Employment__c, InternalOrganizationUnit, PersonLifeEvent, etc.
│
├── Is it a person/company? → Use Person Account (Employee2) or Account (Vendor)
│     Never create a custom "Person" object if Person Accounts are enabled
│
├── Is it a support case / issue? → Use Case (standard)
│
└── None of the above → Create Custom Object
```

---

## 2. Custom Object — Full Metadata

```
force-app/main/default/objects/
  Employment__c/
    Employment__c.object-meta.xml        ← object definition
    fields/
      Start_Date__c.field-meta.xml
      Status__c.field-meta.xml
    validationRules/
      Validate_Start_Before_End.validationRule-meta.xml
    listViews/
      Active_Employments.listView-meta.xml
    compactLayouts/
      Employment_Compact.compactLayout-meta.xml
```

### object-meta.xml — Complete Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObject xmlns="http://soap.sforce.com/2006/04/metadata">

    <!-- Basic info -->
    <label>Employment</label>
    <pluralLabel>Employments</pluralLabel>
    <description>Tracks employee employment records</description>

    <!-- Name field -->
    <nameField>
        <label>Employment Number</label>
        <type>AutoNumber</type>           <!-- Text | AutoNumber -->
        <displayFormat>EMP-{0000}</displayFormat>   <!-- for AutoNumber only -->
    </nameField>

    <!-- Status -->
    <deploymentStatus>Deployed</deploymentStatus>   <!-- Deployed | InDevelopment -->
    <sharingModel>Private</sharingModel>            <!-- Private | Read | ReadWrite | ControlledByParent -->

    <!-- Compact Layout -->
    <compactLayoutAssignment>Employment_Compact</compactLayoutAssignment>

    <!-- Features — ALL default false, set intentionally -->
    <enableSearch>true</enableSearch>       <!-- Global Search — ALWAYS set true unless intentional -->
    <enableReports>true</enableReports>     <!-- Allow Reports -->
    <enableActivities>true</enableActivities>  <!-- Tasks & Events -->
    <enableHistory>true</enableHistory>     <!-- Field History Tracking -->
    <enableBulkApi>true</enableBulkApi>     <!-- Bulk API access -->
    <enableSharing>true</enableSharing>     <!-- Sharing Rules support -->
    <enableStreamingApi>true</enableStreamingApi>   <!-- Platform Events / Streaming -->

</CustomObject>
```

> 🔴 **`enableSearch=false` = object not in Global Search.** Always set `true` unless the object should be hidden from search (e.g. lookup/reference tables).

---

## 3. Field Types — Complete Reference

### Text Fields

```xml
<!-- Short Text (up to 255 chars) -->
<fields>
    <fullName>Description__c</fullName>
    <label>Description</label>
    <type>Text</type>
    <length>255</length>
    <required>false</required>
    <unique>false</unique>
    <externalId>false</externalId>
</fields>

<!-- Long Text (up to 131072 chars) -->
<fields>
    <fullName>Notes__c</fullName>
    <label>Notes</label>
    <type>LongTextArea</type>
    <length>32768</length>      <!-- set to 32768 for content that may exceed 255 -->
    <visibleLines>5</visibleLines>
</fields>

<!-- Auto Number -->
<fields>
    <fullName>Employment_Number__c</fullName>
    <label>Employment Number</label>
    <type>AutoNumber</type>
    <displayFormat>EMP-{0000}</displayFormat>
    <startingNumber>1</startingNumber>
    <externalId>false</externalId>
</fields>

<!-- Unique Text (for external IDs) -->
<fields>
    <fullName>External_ID__c</fullName>
    <label>External ID</label>
    <type>Text</type>
    <length>50</length>
    <unique>true</unique>
    <externalId>true</externalId>
    <caseSensitive>false</caseSensitive>
</fields>
```

### Number Fields

```xml
<!-- Number -->
<fields>
    <fullName>Hourly_Rate__c</fullName>
    <label>Hourly Rate</label>
    <type>Number</type>
    <precision>10</precision>   <!-- total digits -->
    <scale>2</scale>            <!-- decimal places -->
    <required>false</required>
</fields>

<!-- Currency -->
<fields>
    <fullName>Monthly_Salary__c</fullName>
    <label>Monthly Salary</label>
    <type>Currency</type>
    <precision>16</precision>
    <scale>2</scale>
</fields>

<!-- Percent -->
<fields>
    <fullName>Tax_Rate__c</fullName>
    <label>Tax Rate</label>
    <type>Percent</type>
    <precision>5</precision>
    <scale>2</scale>
</fields>
```

### Date Fields

```xml
<!-- Date -->
<fields>
    <fullName>Start_Date__c</fullName>
    <label>Start Date</label>
    <type>Date</type>
    <required>false</required>
</fields>

<!-- DateTime -->
<fields>
    <fullName>Last_Action_DateTime__c</fullName>
    <label>Last Action</label>
    <type>DateTime</type>
</fields>
```

### Picklist Fields

```xml
<!-- Standard Picklist -->
<fields>
    <fullName>Status__c</fullName>
    <label>Status</label>
    <type>Picklist</type>
    <required>false</required>
    <valueSet>
        <restricted>true</restricted>   <!-- true = only defined values allowed -->
        <valueSetDefinition>
            <sorted>false</sorted>
            <value>
                <fullName>Active</fullName>
                <default>true</default>
                <label>Active</label>   <!-- English — Hebrew via Translation -->
            </value>
            <value>
                <fullName>Inactive</fullName>
                <default>false</default>
                <label>Inactive</label>
            </value>
            <value>
                <fullName>Past</fullName>
                <default>false</default>
                <label>Past</label>
            </value>
        </valueSetDefinition>
    </valueSet>
</fields>

<!-- Multi-Select Picklist -->
<fields>
    <fullName>Skills__c</fullName>
    <label>Skills</label>
    <type>MultiselectPicklist</type>
    <visibleLines>5</visibleLines>
    <valueSet>
        <restricted>true</restricted>
        <valueSetDefinition>
            <sorted>false</sorted>
            <value><fullName>Management</fullName><default>false</default><label>Management</label></value>
            <value><fullName>Technical</fullName><default>false</default><label>Technical</label></value>
        </valueSetDefinition>
    </valueSet>
</fields>
```

> ⚠️ **Picklist API values (fullName) must be English.** Hebrew is set via Translation only.
> ⚠️ **`restricted=true`** = only listed values can be saved. Always use `true` unless legacy data migration requires open values.

### Relationship Fields

```xml
<!-- Lookup (optional relationship) -->
<fields>
    <fullName>Employee__c</fullName>
    <label>Employee</label>
    <type>Lookup</type>
    <referenceTo>Employee2</referenceTo>    <!-- PSS object — no __c -->
    <relationshipName>Employments</relationshipName>   <!-- plural, used as __r name -->
    <relationshipLabel>Employments</relationshipLabel>
    <required>false</required>
    <deleteConstraint>SetNull</deleteConstraint>   <!-- SetNull | Restrict | Cascade -->
</fields>

<!-- Master-Detail (required relationship, child deleted with parent) -->
<fields>
    <fullName>Employment__c</fullName>
    <label>Employment</label>
    <type>MasterDetail</type>
    <referenceTo>Employment__c</referenceTo>
    <relationshipName>Tariff_Lines</relationshipName>
    <relationshipLabel>Tariff Lines</relationshipLabel>
    <relationshipOrder>0</relationshipOrder>   <!-- 0 = primary master (up to 2 masters) -->
    <writeRequiresMasterRead>false</writeRequiresMasterRead>
    <reparentableMasterDetail>false</reparentableMasterDetail>
</fields>
```

| deleteConstraint | Behavior when parent deleted |
|---|---|
| `SetNull` | Lookup field set to null (default) |
| `Restrict` | Cannot delete parent if children exist |
| `Cascade` | Child records deleted with parent |

> ⚠️ **Cannot change `referenceTo` via Metadata API.** Delete + recreate is the only way.
> ⚠️ **MasterDetail fields are always required.** Never add them to `fieldPermissions` in Profile.

### Formula Fields

```xml
<fields>
    <fullName>Employment_Duration_Days__c</fullName>
    <label>Duration (Days)</label>
    <type>Number</type>
    <precision>10</precision>
    <scale>0</scale>
    <formula>IF(ISBLANK(End_Date__c), TODAY() - Start_Date__c, End_Date__c - Start_Date__c)</formula>
    <formulaTreatBlanksAs>BlankAsZero</formulaTreatBlanksAs>
</fields>

<!-- Text formula -->
<fields>
    <fullName>Full_Status__c</fullName>
    <label>Full Status</label>
    <type>Text</type>
    <length>255</length>
    <formula>Name &amp; " - " &amp; TEXT(Status__c)</formula>   <!-- & must be &amp; in XML -->
    <formulaTreatBlanksAs>BlankAsBlank</formulaTreatBlanksAs>
</fields>
```

> ⚠️ Formula fields are **read-only.** Never add them to `fieldPermissions` in Profile (treat like required fields).
> ⚠️ `&` in formula XML must be written as `&amp;`.

### Checkbox

```xml
<fields>
    <fullName>Is_Active__c</fullName>
    <label>Is Active</label>
    <type>Checkbox</type>
    <defaultValue>false</defaultValue>
</fields>
```

---

## 4. Standard Objects — Key Behaviors

### Account

| Context | Behavior |
|---|---|
| Person Accounts enabled | Account can be a "person" (Employee2 is a Person Account) |
| RecordType: Vendor | Used for vendor companies in this org |
| Standard fields | Name (required), OwnerId, RecordTypeId |

> ⚠️ **Never create a custom "person" or "company" object if Account covers the use case.**

### Contact

| Context | Behavior |
|---|---|
| Person Accounts enabled | Contact is the "back side" of a Person Account |
| `PersonContactId` | The Contact Id behind Employee2 — used for IndividualId |
| Custom fields on Contact | Appear as `__pc` fields on the Account/Employee2 in Flows |

### Case

Used for error logging and support tickets. In this org: Fault Path in Flows logs errors to Case.

```apex
// Flow fault path pattern — log to Case
Case errorCase = new Case(
    Subject = 'Flow Error: ' + flowName,
    Description = errorMessage,
    Status = 'New',
    Origin = 'Flow'
);
insert errorCase;
```

### User

```
Standard fields: Id, Name, Username, Email, ProfileId, UserRoleId, IsActive
Permission Set assignment: PermissionSetAssignment object
PSL assignment: PermissionSetLicenseAssign object
```

---

## 5. Record Types

### What Record Types Do

- Allow different Picklist values per RecordType on the same object
- Allow different Page Layouts per RecordType per Profile
- Allow different business processes (for Opportunity, Case, Lead)
- Identified in code via `RecordType.DeveloperName`

### Record Type Metadata

```
force-app/main/default/objects/Employment__c/
  recordTypes/
    Standard_Employment.recordType-meta.xml
    Vendor_Employment.recordType-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<RecordType xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Standard_Employment</fullName>
    <active>true</active>
    <description>Standard employment record</description>
    <label>Standard Employment</label>
    <picklistValues>
        <picklist>Status__c</picklist>
        <values>
            <fullName>Active</fullName>
            <default>true</default>
        </values>
        <values>
            <fullName>Inactive</fullName>
            <default>false</default>
        </values>
    </picklistValues>
</RecordType>
```

> ⚠️ Picklist values in RecordType = **subset** of values defined on the field. Cannot add new values here.
> ⚠️ RecordType must be in `recordTypeVisibilities` in Profile to be selectable.

### Profile RecordType Permissions — MANDATORY

```xml
<!-- In Profile XML -->
<recordTypeVisibilities>
    <default>true</default>      <!-- one RecordType must be default per object per Profile -->
    <recordType>Employment__c.Standard_Employment</recordType>
    <visible>true</visible>
</recordTypeVisibilities>
<recordTypeVisibilities>
    <default>false</default>
    <recordType>Employment__c.Vendor_Employment</recordType>
    <visible>true</visible>
</recordTypeVisibilities>

<!-- Different Layout per RecordType -->
<layoutAssignments>
    <layout>Employment__c-Standard Employment Layout</layout>
    <object>Employment__c</object>
    <recordType>Employment__c.Standard_Employment</recordType>
</layoutAssignments>
<layoutAssignments>
    <layout>Employment__c-Vendor Employment Layout</layout>
    <object>Employment__c</object>
    <recordType>Employment__c.Vendor_Employment</recordType>
</layoutAssignments>
```

### Referencing RecordType in Code

```apex
// Apex
Id rtId = Schema.SObjectType.Employment__c.getRecordTypeInfosByDeveloperName()
              .get('Standard_Employment').getRecordTypeId();
```

```xml
<!-- Validation Rule -->
$RecordType.DeveloperName = 'Vendor_Employment' && ISBLANK(VAT_Number__c)
```

```javascript
// Flow — Decision element
{!$Record.RecordType.DeveloperName} equals Standard_Employment
```

---

## 6. Deploy Order for Objects

```
1. Parent objects (objects with no dependencies)
2. Child objects that reference parents (Lookup/MasterDetail)
3. Fields — non-Lookup first, then Lookup fields
4. Record Types (after picklist fields exist)
5. Validation Rules (after fields exist)
6. Compact Layouts
7. Page Layouts (after fields and Record Types)
8. List Views
9. Profiles (ALWAYS LAST)
```

---

## 7. Common Mistakes

| Mistake | What Happens | Fix |
|---|---|---|
| Hebrew in field API name (`fullName`) | Deploy error or data issues | English API names only — always |
| Hebrew in field `<label>` | English users also see Hebrew | English in `<label>`, Hebrew in `objectTranslation/<fields><label>` |
| Hebrew in object `<label>` / `<pluralLabel>` | English users also see Hebrew | English in object metadata, Hebrew in `objectTranslation/<caseValues>` |
| Hebrew in picklist `<fullName>` | Flows/Apex must use Hebrew strings, breaks SOQL readability | English API values in `<fullName>`, Hebrew labels in `objectTranslation/<picklistValues>` |
| Hebrew in picklist `<label>` | English users see Hebrew picklist options | English in `<label>`, Hebrew in `objectTranslation/<picklistValues><translation>` |
| `enableSearch=false` | Object missing from Global Search | Set `true` |
| Required field in Profile FLS | Deploy error | Never add required/MD/Formula fields to fieldPermissions |
| Changing `referenceTo` on existing Lookup | Metadata API error | Delete + recreate field |
| RecordType not in Profile `recordTypeVisibilities` | Users can't create that RecordType | Add to every relevant Profile |
| MasterDetail `deleteConstraint` | N/A — MasterDetail always cascades | Use Lookup if Cascade not desired |
| `restricted=false` on picklist | Old/garbage values accepted | Set `restricted=true` |
| Two RecordTypes both `default=true` in Profile | Deploy error | Only one default per object per Profile |
| AutoNumber field modified after records exist | Cannot change format | Plan AutoNumber format before first record |

---

## 8. Cross-Domain Interactions (Critical — Often Missed)

### Changing Field Type — Cascade Impact
Changing a field's data type affects every component that references it:

| Change | Downstream Impact |
|---|---|
| Text → Lookup | All Flows filtering on this field must update; Reports referencing the field lose the column; Validation Rules using it may need rewrite |
| Lookup → MasterDetail | All existing Sharing Rules on child object become invalid; `deleteConstraint` changes to cascade; OWD changes to `ControlledByParent` if you choose |
| Required=false → Required=true | Profile FLS entries for this field (if any) cause deploy error → remove from Profile FLS; Bulk Load must always include this field |
| Text → Encrypted | Existing data NOT encrypted automatically → run "Encrypt Existing Data" in Setup; all SOQL filters must switch to `=` only; VLOOKUP cannot use it as first param |
| Picklist: adding value | Safe. No downstream impact. |
| Picklist: removing value | Existing records retain old value; VRs comparing to old value still fire; Reports may show blank for old value |

**Rule: Before changing any field type, audit: Flows, Validation Rules, Reports, Apex, LWC — every reference.**

### OWD and Field-Level Security Interaction
- OWD controls **who can see which records**
- FLS controls **which fields are visible on those records**
- They are independent. A user can have OWD access to a record but FLS blocks all fields → they see the record exists but all fields are blank
- Setting OWD = Private without configuring Sharing Rules → users can create records but cannot see others' records
- Setting `enableSearch=true` on Private OWD object → Global Search returns results, but only for records the user has access to

### Encrypted Fields in Object Design
- Supported types: Text, Phone, Email, URL, TextArea (Short) only
- NOT supported: LongTextArea, Formula, Number, AutoNumber, Lookup, Checkbox, Date, Datetime
- Cannot be used as the first parameter in VLOOKUP (in Validation Rules)
- Cannot be used in ORDER BY, GROUP BY, HAVING, LIKE in SOQL
- Can be used as ExternalId for Bulk Load upsert (exact case-insensitive match)
- After enabling encryption on an existing field: existing data is NOT encrypted → must manually run "Encrypt Existing Data" in Setup → Platform Encryption

### Object Search and Reporting Implications
- `enableSearch=true`: object appears in Global Search and can be used in List Views
- `enableReports=true`: object appears in Report Type builder
- `enableHistory=true`: enables Field History Tracking (requires separate field selection per field)
- If `enableReports=false` → cannot create Custom Report Types (CRT) using this object → set to `true` from start
