# Salesforce — General Implementation SKILL (formerly PSS)

## When to use this skill
Read this file at the start of ANY Salesforce implementation task, including:
- Metadata deployment (Objects, Fields, Profiles, Validation Rules, Flows, Tabs, FlexiPages)
- Validation Rules — patterns, ISCHANGED, numbering standard
- Flows — best practices, ISNEW/ISCHANGED, bulk safety, Person Accounts
- Salesforce PSS (Public Sector Solutions) objects
- HR system objects: Employee2, Employment, InternalOrganizationUnit, PersonLifeEvent
- Bulk data loading to Salesforce
- Permission Sets and Licenses setup

> 📌 This file grew from PSS-specific to general Salesforce patterns. Validation Rules and Flows are general — not PSS-specific.

---

## 1. PSS Standard Objects — Check Before Creating Custom

**Iron rule:** Before creating any Custom Object, check if PSS already provides an equivalent.

| API Name | Business Name | Notes |
|---|---|---|
| InternalOrganizationUnit | Org Unit | replaces Organization_Unit__c |
| Employee2 | Employee | PSS Person Account |
| Employment__c | Employment | PSS Standard — verify before creating custom |
| PersonLifeEvent | Life Event | birth, marriage, retirement |
| JobRequisition | Open Position | check as replacement for Job_Tender__c |
| JobProfile | Job Profile | check as replacement for Job_Catalog__c |

**To expose PSS objects, assign to Admin first:**
- `Public_Sector_Solutions_Admin`
- `PublicSectorAccessPSL`
- Talent Recruitment Management Specialist

> ⚠️ Without these Permission Sets, PSS objects are invisible in describe and UI — they may appear non-existent. Always assign before checking.

> ⚠️ `Employee_Experience_User` — for employee portal only. Requires separate Experience Cloud License. Do NOT assign to internal HR managers.

---

## 2. Naming Conventions — English API Names Only

| Type | Correct | Wrong |
|---|---|---|
| Custom Object | Employment__c | העסקה__c |
| Custom Field | Hourly_Rate__c | שכר_שעתי__c |
| Picklist Value (API) | Active | פעיל |
| Validation Rule | Validate_Start_Before_End | בדיקת_תאריך |
| Profile | HR_Manager | any Hebrew name |

**Conventions:**
- Custom Objects: `PascalCase__c` → `Employment__c`, `Job_Catalog__c`
- Custom Fields: `PascalCase__c` → `Hourly_Rate__c`, `Start_Date__c`
- Lookup Fields: related object name + `__c` → `Employment__c`, `OrgUnit__c`
- Relationship Names: child plural → `Employments`, `PurchaseOrders`
- Validation Rules: `Verb_Subject_Condition` → `Lock_Past_Tariff_Lines`
- Profiles: `Role_Level` → `HR_Manager`, `Finance_ReadOnly`

**Picklist values in CSV/Bulk API — always API Value, never Hebrew label:**
```
'פעיל'    => 'Active'
'לא פעיל' => 'Inactive'
'עבר'     => 'Past'
```

---

## 3. Profiles, FLS & Object Permissions

### Fields NOT to include in fieldPermissions
| Field Type | Include in FLS? | Reason |
|---|---|---|
| Required Custom Field | ❌ | Error: "You cannot deploy to a required field" |
| MasterDetail Field | ❌ | Always Required in Salesforce |
| Standard Required Field | ❌ | e.g. OrganizationCode, Name |
| Optional Custom Field | ✅ | Any non-required, non-MD field |

### Object Permissions
> ⚠️ `modifyAllRecords=true` requires `allowDelete=true`. Without it: deploy error "Permission Modify All depends on Delete"

```xml
<objectPermissions>
    <allowCreate>true</allowCreate>
    <allowDelete>true</allowDelete>   <!-- required if modifyAllRecords=true -->
    <allowEdit>true</allowEdit>
    <allowRead>true</allowRead>
    <modifyAllRecords>true</modifyAllRecords>
    <object>Employment__c</object>
    <viewAllRecords>true</viewAllRecords>
</objectPermissions>
```

### Tab Visibility in Profile
| Object Type | Format |
|---|---|
| Custom Object | `<tab>Employment__c</tab>` |
| PSS / Standard Object | `<tab>standard-InternalOrganizationUnit</tab>` |
| Standard Salesforce | `<tab>standard-Account</tab>` |

> ⚠️ Do NOT create a Tab file for PSS Standard objects — they already include a Tab. Reference in Profile only.

---

## 4. Validation Rules — Critical Limitations

> ⚠️ `VLOOKUP` requires the first parameter to be a Record Name field ($Name). Cannot use a numeric field (Number, Currency) — error: "Expected Record Name field".
> For uniqueness validation on numeric fields: use Duplicate Rule or Apex Trigger.

### Useful Formulas
| Purpose | Formula |
|---|---|
| Start date before end | `NOT(ISBLANK(End_Date__c)) && Start_Date__c >= End_Date__c` |
| No future date | `NOT(ISBLANK(Date__c)) && Date__c > TODAY()` |
| No negative value | `NOT(ISBLANK(Field__c)) && Field__c < 0` |
| Lock by status | `ISPICKVAL(Status__c,'Past') && ISCHANGED(Rate__c)` |
| Required by RecordType | `$RecordType.DeveloperName = 'Vendor' && ISBLANK(VAT_Number__c)` |
| Exceed ceiling from related | `NOT(ISBLANK(Related__r.Max__c)) && Value__c > Related__r.Max__c` |

---

## 5. Deployment — Order and Methods

### Correct Deploy Order
1. Objects (base objects)
2. Custom Fields (non-Lookup first)
3. Lookup Fields (only after related objects exist)
4. Validation Rules (after fields exist)
5. Custom Tabs (after objects work)
6. Profiles (**always last** — depends on everything)
7. Destructive Changes (only after removing all dependencies)

### Destructive Changes
- Use `destructiveChangesPost.xml` (runs after deploy)
- Do NOT combine `destructiveChanges.xml` + `destructiveChangesPost.xml` in same folder (API >= 33.0)
- Before deleting an Object — first remove all Lookup Fields pointing to it
- Soft-deleted fields (`_del__c` suffix) block Object deletion — requires manual cleanup in Setup

### Source Tracking Cache Fix
> ⚠️ "Unchanged" error doesn't mean the field hasn't changed — sometimes cache is wrong.
Override with:
```bash
sf project deploy start --metadata-dir my_metadata_folder --target-org alias
```

### ⚠️ Common Metadata Errors (נסיון מהשטח)

| שגיאה בdeploy | סיבה | תיקון |
|---|---|---|
| `Element 'required' is not valid` על MasterDetail field | `<required>` אינו תקין בשדות MasterDetail inline | הסר את `<required>` — MasterDetail תמיד required |
| `Element 'length' is not valid` על TextArea | `<length>` תקין רק ל-Text, לא ל-TextArea/LongTextArea | הסר את `<length>` |
| `Delete constraint missing` על lookup required | Lookup שהוא required-style צריך `deleteConstraint` מפורש | הוסף `<deleteConstraint>Restrict</deleteConstraint>` |
| `outputAssignments invalid` בCreate Record | `outputAssignments` מיושן ב-Flow XML חדש | החלף ב-`<assignRecordIdToReference>variableName</assignRecordIdToReference>` |
| `Flow XML section order error` | Flow XML דורש סדר מחלקות מסוים לפי Metadata API | סדר נכון: variables → constants → formulas → decisions → loops → recordLookups → recordCreates → recordUpdates → start |

### ⚠️ CustomTab — `<object>` Element אסור

CustomTab עם `<customObject>true</customObject>` **אסור לו לכלול אלמנט `<object>`**. השילוב גורם לשגיאת deploy.

```xml
<!-- ❌ שגוי -->
<CustomTab>
    <customObject>true</customObject>
    <object>Employment_Record__c</object>   ← הסר שורה זו
    <fullName>Employment_Record__c</fullName>
    <label>Employment Records</label>
    <motif>Custom74: Handshake</motif>
</CustomTab>

<!-- ✅ נכון -->
<CustomTab>
    <customObject>true</customObject>
    <fullName>Employment_Record__c</fullName>
    <label>Employment Records</label>
    <motif>Custom74: Handshake</motif>
</CustomTab>
```

### ⚠️ Flow Translations — אסור להוסיף ל-`iw.translation-meta.xml`

Salesforce דוחה entries של Flow label translations בקובץ `iw.translation-meta.xml`. Flow translations דורשים מבנה Metadata API נפרד שאינו מתועד בצורה יציבה.

**כלל:** אל תוסיף `<flows>` entries לקובץ ה-translation. אם נדרשים תרגומי Flow — בצע ידנית דרך Translation Workbench.

Metadata API format (different from SFDX source format):
```
my_folder/
  objects/
    MyObject__c.object    ← single XML containing everything
  package.xml
```

### Changing Lookup referenceTo
> ⚠️ Cannot change `referenceTo` of existing Lookup via Metadata API. Only solution: delete + recreate.
- Sandbox without data: delete → recreate with correct referenceTo
- Production with data: create new field, copy data, remove old

---

## 6. Bulk Data Loading

### Bulk API v2 Requirements
| Requirement | Details |
|---|---|
| CRLF line endings (`\r\n`) | Not LF only |
| Headers in English | API Names, not Hebrew Labels |
| Picklist values | API Values (English), not Hebrew Labels |
| NULL in numeric fields | Replace with empty string `""`, not the word "NULL" |
| LongTextArea | Set to 32768 if content exceeds 255 chars |

### Fix CSV Pattern (Node.js)
```javascript
const fixed = row.map(v => v === 'NULL' ? '' : v);       // clear NULL
const mapped = fixed.map(v => hebrewToApi[v] || v);       // Hebrew → API
const out = rows.map(r => r.join(',')).join('\r\n');        // CRLF
```

---

## 7. Permission Set Licenses (PSLs) — Required per User

| PSL | Who needs it | What it enables |
|---|---|---|
| Industries User | All HR managers | Employee2, Employment, InternalOrganizationUnit, PersonLifeEvent |
| OmniStudio User | All HR managers | FlexCards on Record Pages |
| OmniStudio Admin | System Admin only | FlexCard Designer, DataRaptor, Integration Procedures |
| Salesforce Shield | All HR managers + Admin | View Encrypted Data, Encryption Policy management |

> PSL is assigned automatically when assigning a linked Permission Set — but only if the PSL exists in the org (purchased). If the PSL is not in the org, the Permission Set won't work.

---

## 8. Recommended Architecture

```
InternalOrganizationUnit (org unit)
  └── Employee2 / Person Account (employee)
        └── Employment__c → Employee2 + InternalOrganizationUnit
              ├── Tariff_Line__c (tariff history)
              ├── Monthly_Payment_Control__c (monthly billing control)
              └── Annual_Feedback__c (annual feedback)

Job_Catalog__c / JobProfile (job catalog)
Account (Vendor RecordType) ← vendors
PurchaseOrder__c → Account + InternalOrganizationUnit
```

### OWD + Sharing Rules — Metadata

#### sharingModel ב-CustomObject XML

```xml
<sharingModel>Private</sharingModel>       <!-- רק בעלים + Role Hierarchy -->
<sharingModel>Read</sharingModel>          <!-- כולם קוראים, רק בעלים עורך -->
<sharingModel>ReadWrite</sharingModel>     <!-- כולם קוראים ועורכים -->
<sharingModel>ControlledByParent</sharingModel>  <!-- Master-Detail בלבד — ירושה מהורה -->
```

> ⚠️ שינוי OWD ל-Private על אובייקט עם נתונים קיימים — עלול לחסום גישה מיידית. לבצע עם Sharing Rule מוכנה מראש.

#### Sharing Rules — קובץ נפרד

```
force-app/main/default/sharingRules/
  Annual_Feedback__c.sharingRules-meta.xml   ← שם הקובץ = API name של האובייקט
```

```bash
sf project deploy start --metadata "SharingRules:Annual_Feedback__c" --target-org alias
```

#### XML — Ownership-based (לפי בעלות)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<SharingRules xmlns="http://soap.sforce.com/2006/04/metadata">
    <sharingOwnerRules>
        <fullName>Share_AnnualFeedback_with_HR_Managers</fullName>
        <accessLevel>Read</accessLevel>          <!-- Read | Edit -->
        <label>Share Annual Feedback with HR Managers</label>
        <sharedTo>
            <group>PG_HR_Managers</group>        <!-- Public Group DeveloperName -->
            <!-- OR: <role>HR_Manager</role> -->
            <!-- OR: <roleAndSubordinates>HR_Manager</roleAndSubordinates> -->
            <!-- OR: <allInternalUsers/> -->
        </sharedTo>
        <sharedFrom>
            <allInternalUsers/>                  <!-- רשומות בבעלות כל משתמש פנימי -->
            <!-- OR: <role>RoleName</role> -->
            <!-- OR: <group>GroupName</group> -->
        </sharedFrom>
    </sharingOwnerRules>
</SharingRules>
```

#### XML — Criteria-based (לפי ערך שדה)

```xml
<SharingRules xmlns="http://soap.sforce.com/2006/04/metadata">
    <sharingCriteriaRules>
        <fullName>Share_Active_Feedback</fullName>
        <accessLevel>Read</accessLevel>
        <label>Share Active Feedback</label>
        <booleanFilter>1</booleanFilter>
        <criteriaItems>
            <field>Status__c</field>
            <operation>equals</operation>
            <value>Active</value>
        </criteriaItems>
        <sharedTo>
            <roleAndSubordinates>HR_Manager</roleAndSubordinates>
        </sharedTo>
    </sharingCriteriaRules>
</SharingRules>
```

#### איך לגלות שמות Roles ו-Groups בorg

```apex
for(UserRole r : [SELECT DeveloperName, Name FROM UserRole]) {
    System.debug('ROLE: ' + r.DeveloperName);
}
for(Group g : [SELECT DeveloperName, Type FROM Group WHERE Type IN ('Regular','Queue')]) {
    System.debug('GROUP: ' + g.DeveloperName + ' (' + g.Type + ')');
}
```

**Roles בorg זה:** `Finance`, `HR_Manager`, `MOH_Root`
**Public Groups:** `PG_HR_Managers`, `Q1`

#### כללים

- Sharing Rules יכולות רק **לפתוח** גישה — לא לצמצם (OWD קובע את הרצפה)
- Ownership-based = לפי מי הבעלים של הרשומה
- Criteria-based = לפי ערך שדה ברשומה
- `accessLevel`: `Read` או `Edit` (לא `ReadWrite`)
- שם קובץ = API name מדויק של האובייקט (כולל `__c`)

### Recommended OWD
| Object | OWD | Reason |
|---|---|---|
| Annual_Feedback__c | Private | Sensitive — employee sees own only |
| Employment__c | Private | Managed via Sharing Rules |
| PurchaseOrder__c | Private | Access by hierarchy |
| Job_Catalog__c | Public Read Only | Everyone reads, HR writes |
| InternalOrganizationUnit | Public Read Only | Org structure visible to all |

---

## 9. Flows — Best Practices

### Flow Naming Convention
| Type | Format | Example |
|---|---|---|
| Record-Triggered | `RTF_[Object]_[Description]` | `RTF_Employment_StatusOnSave` |
| Scheduled | `SF_[Object]_[Description]` | `SF_Employment_StatusUpdate` |
| Autolaunched | `AF_[Object]_[Description]` | `AF_MonthlyPayment_Recalculate` |
| Screen Flow | `SCR_[Description]` | `SCR_Skills_Assignment` |
| Subflow | `SUB_[Description]` | `SUB_Employment_CalcStatus` |

### Core Rules
| Rule | Detail |
|---|---|
| Entry Criteria | **Always define** — never "Run flow for All records" |
| Before / After Save | Before Save = change fields on same record (no DML). After Save = everything else |
| Fault Path | **Mandatory** on every Create / Update / Delete element — log to Case |
| Element naming | Every element: descriptive name + Description. Never leave "Get_Records_1" |
| Flow description | Must include: what the flow does + what triggers it + entry conditions |
| Subflows | Flow > 15 elements → consider breaking into Subflow |
| Hard coding | Never hard-code Salesforce IDs → use Custom Metadata. Never hard-code picklist values in conditions → use CMT |

### Before Save — Field Assignments
- Use `Assignment` elements writing to `$Record.FieldName__c`
- No DML needed — changes save with the record automatically
- Formula variables: use `expression` with `TODAY()` for date comparisons
- For Lookup fields: assign the related record's **Id** (string) to the lookup field

### After Save — Key Patterns
- Use `RecordLookups` with `getFirstRecordOnly` for single record queries
- Always add `assignNullValuesIfNoRecordsFound=true` when the result drives a Decision
- `IsChanged` operator in Entry Criteria filters avoids unnecessary runs
- For status sync: query all related records first, then decide

### Scheduled Flows
- Filter-based `RecordUpdates` (no loop) for bulk updates — e.g. all Employment where status is wrong
- Add `NotEqualTo current value` filter to avoid updating records that are already correct (reduces DML)
- Use `formula_Today` (`TODAY()`) as formula variable — don't hard-code dates
- Idempotent design: running twice must not create duplicates (check before create)

> ⚠️ **`Frequency=Monthly` is rejected by Salesforce** for scheduled-triggered flow metadata. There is no monthly recurrence option in the Metadata API.
> **Workaround:** Build as a **Daily scheduled flow** with a first-day-of-month Decision:
> ```
> Decision: Is_First_Day_Of_Month
>   DAY(TODAY()) = 1  →  Yes: run logic
>                     →  No: exit immediately
> ```

### Person Accounts — Enabling & Field Registration
> ⚠️ Person Accounts must be enabled BEFORE deploying flows that reference `__pc` fields.
> Enable via: Setup → Account Settings → Enable Person Accounts (irreversible — sandbox first)
> If Contact custom fields were deployed BEFORE enabling Person Accounts:
> 1. Enable Person Accounts
> 2. Redeploy the flows that reference `__pc` fields with `--ignore-conflicts` to force re-validation
> 3. If Flow Builder still shows warnings: open the flow → Save (no changes needed) to refresh

### PersonAccount Fields in Flows
- Custom Contact fields appear as `__pc` suffix when queried on Account in flows: `$Record.Birth_Time__pc`
- Standard person fields (e.g. `PersonBirthdate`) are referenced directly with no suffix
- `PersonContactId` = the Contact Id behind the Person Account — use for `IndividualId` on Employee2

### ⚠️ PersonAccount Record Type — Metadata Address

> **Person Account record type** is addressed in metadata as **`PersonAccount.PersonAccount`**, NOT `Account.PersonAccount`.
>
> If you deploy `force-app/main/default/objects/Account/recordTypes/PersonAccount.recordType-meta.xml`, Salesforce creates a **regular business-account record type** with the same developer name — not a Person Account record type.
> This record type cannot be deleted via API. It can only be set inactive and renamed manually in Setup.
>
> For profile record type visibility, reference `PersonAccount.PersonAccount` and include the `PersonAccount` custom object metadata in the same deployment.

### ⚠️ Apex — Updating Person Account: `Name` Is Not Updateable

When updating an existing Person Account through Apex, **do not DML a queried Account sObject that carries `Name`**.
Salesforce throws `INVALID_FIELD_FOR_INSERT_UPDATE` on `Name` for Person Account updates.

```apex
// ❌ Wrong — queried Account includes Name, causes error
Account existing = [SELECT Id, Name FROM Account WHERE Id = :someId];
existing.National_ID__pc = '123456789';
update existing;

// ✅ Correct — build a fresh update object with only the fields you intend to change
Account updateObj = new Account(Id = someId, National_ID__pc = '123456789');
update updateObj;
```

### Hebrew Date Lookup Pattern
```
formula_ConversionKey = TEXT($Record.PersonBirthdate) & "_" & $Record.Birth_Time__pc
→ Get Hebrew_Calendar__c WHERE Conversion_Key__c = formula_ConversionKey
→ Assign $Record.Heb_Birthdate__pc = var_HebrewDate.Id
```
> ⚠️ Hebrew_Calendar__c must be pre-loaded (70 years back, 10 years forward)
> ⚠️ Day_Part__c values in the table must exactly match Birth_Time__pc picklist API values

### Record-Triggered Flow — ISNEW / ISCHANGED Pattern (Critical)

A Flow trigger condition must handle **both creation AND update** scenarios correctly.
`ISCHANGED()` alone fails on new records — fields are not "changed" during insert.

**Always split trigger logic into two clauses:**

```
// Entry Criteria formula (Start node)
OR(
  AND(
    ISNEW(),
    <creation conditions based on field values>  // e.g. NOT(ISBLANK(Status__c))
  ),
  AND(
    NOT(ISNEW()),
    OR(
      ISCHANGED(Status__c),
      ISCHANGED(Employment_Type__c)
    )
  )
)
```

| Scenario | Logic | Example |
|---|---|---|
| On create, field is populated | `ISNEW() AND NOT(ISBLANK(field))` | New Employment with Status filled |
| On update, specific field changed | `NOT(ISNEW()) AND ISCHANGED(field)` | Status changed on existing record |
| Either scenario | `OR(ISNEW_condition, ISCHANGED_condition)` | Most common pattern |

> ⚠️ **Entry Criteria = performance optimization only.** For business correctness, add a **Decision element** as the first node inside the Flow as a safety gate. Never rely solely on Start conditions for logic enforcement.

> ⚠️ Never use "Run flow for All records" (no entry criteria). Always define at least one condition.

### Validation Rules — ISCHANGED + Numbering Standard

**Error message numbering** — from spec standard:
```
Format: [VR-XXX] Hebrew error message
Example: [VR-001] תאריך סיום חייב להיות אחרי תאריך התחלה
```
- Number format: 3-digit zero-padded (`VR-001`, `VR-012`, `VR-100`)
- Numbers are sequential per project — check existing VRs before assigning next number
- The user sees this code in the error message → useful for support/debugging

**Using ISCHANGED in Validation Rules:**
```
// Run only when a specific field changes (not on every save):
AND(
  ISCHANGED(Effective_From__c),           // only fires when this field changes
  Effective_From__c > Effective_To__c    // the actual business rule
)
```

**Combining ISCHANGED with ISNEW:**
```
// Rule should fire on create (always) OR on update only when field changes:
OR(
  AND(ISNEW(), <condition>),             // always check on create
  AND(NOT(ISNEW()), ISCHANGED(field), <condition>)  // only check when field changes
)
```

| Pattern | When to use |
|---|---|
| `ISCHANGED(field)` | Rule only relevant when a specific field changes |
| `AND(NOT(ISNEW()), ISCHANGED(field))` | Update-only rule (skip on creation) |
| `OR(ISNEW(), ISCHANGED(field))` | Rule relevant on creation + when field changes |
| No ISCHANGED | Rule must always run (e.g. data integrity, cross-field) |

**Additional Rules:**
- One rule = one problem. Never combine 5 conditions that check different things
- Check for existing VRs before creating — no duplicate logic
- Rule description: what it checks + why (not just "what")

### Avoid Hard Coding
| Situation | Wrong | Correct |
|---|---|---|
| Salesforce ID in Flow | `IF(Employer__c = "0015000001XYZ")` | Custom Metadata (CMT) |
| Critical Picklist value | `IF(Status__c = "פעיל")` | Custom Metadata (CMT) |
| Configurable parameter | `Number_of_Days = 30` in Flow | Custom Settings |
| UI message text | String literal in Apex/LWC | Custom Label |

### Mandatory Documentation
- Every **Validation Rule** description: what it checks + why it exists (not just "what")
- Every **Flow** description: what it does + what triggers it + entry conditions
- Every **Field** help text: why it exists + how it's derived (if Formula)
- Every **Public Group** description: who the members are + purpose
- Every **Permission Set** description: what permissions + why

### ⚠️ Reports — שתי מגבלות חשובות

**1. מגבלת 40 תווים בשם דוח:**
> שמות דוחות ב-Salesforce Analytics מוגבלים ל-**40 תווים** (label). שם ארוך יותר גורם לשגיאת deploy.

**2. Report Type API names אינם אינטואיטיביים:**
> שמות ה-Report Type ל-custom objects הם generated ולא ניתנים לניחוש:
> - אובייקט בודד: `CustomEntity$PurchaseOrder__c`
> - join בין שני אובייקטים: `CustomEntityCustomEntity$Employment_Record__c$Monthly_Payment_Control__c`
>
> **תמיד** גלה את שם ה-Report Type דרך Analytics API לפני כתיבת report metadata:
> ```bash
> sf analytics report-types list --target-org alias
> ```
> או דרך SOQL:
> ```bash
> sf data query --query "SELECT DeveloperName, Label FROM ReportType WHERE Category = 'other'" --target-org alias
> ```

---

## 10. UI — Record Pages & FlexiPages

### FlexCards (OmniStudio) vs. Related Lists — מה ניתן לדיפלוי

| | FlexCard | Related List רגיל |
|---|---|---|
| Deploy via Metadata API | ❌ ידני — דרך OmniStudio Designer | ✅ אוטומטי — FlexiPage XML |
| נתונים מ-Lookup עמוק | ✅ כן (DataRaptor) | ❌ לא (שדות ישירים בלבד) |
| Conditional Styling | ✅ כן | ❌ לא |

> ✅ **כלל פרקטי:** ממש תמיד עם Related Lists קודם — 90% מהצורך מכוסה, דיפלוי מלא. FlexCards — רק לאחר OmniStudio מוגדר.

### סדר deploy של Record Pages

1. **Compact Layouts** — לפני FlexiPage שמפנה אליהם
2. **Quick Actions** — לפני FlexiPage שמציג אותם בפאנל הפעולות
3. **Page Layouts** — fallback עדיין נדרש גם ב-Lightning
4. **FlexiPages** — אובייקטים פשוטים (אין children) קודם, מורכבים אחרון
5. **Page Assignments** — שיוך FlexiPage לאפליקציה / פרופיל / Record Type

### Compact Layout — Highlights Panel

ה-Compact Layout קובע אילו שדות מוצגים ב-**Highlights Panel** (force:highlightsPanel) בראש ה-Record Page.

#### מבנה קבצים

```
objects/
  Job_Catalog__c/
    compactLayouts/
      JobCatalog_Compact.compactLayout-meta.xml   ← הגדרת השדות
    Job_Catalog__c.object-meta.xml                ← שיוך ה-assignment
```

#### XML — CompactLayout

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CompactLayout xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>JobCatalog_Compact</fullName>
    <fields>Name</fields>          <!-- שדה ראשון = הכי בולט -->
    <fields>Status__c</fields>
    <fields>Cluster__c</fields>
    <fields>Level__c</fields>
    <fields>Job_Number__c</fields>
    <fields>Max_Rate__c</fields>
    <fields>Max_Bid_Rate__c</fields>
    <label>Job Catalog</label>
</CompactLayout>
```

#### XML — object-meta.xml (כולל כל ה-enable flags החשובים)

```xml
<CustomObject xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>Job Catalog</label>
    <pluralLabel>Job Catalog</pluralLabel>
    <nameField>
        <label>Job Title</label>
        <type>Text</type>
    </nameField>
    <deploymentStatus>Deployed</deploymentStatus>
    <sharingModel>ReadWrite</sharingModel>
    <compactLayoutAssignment>JobCatalog_Compact</compactLayoutAssignment>

    <!-- ⚠️ חשוב: ברירת מחדל false — לשנות בכוונה -->
    <enableSearch>true</enableSearch>      <!-- Allow Search — חובה לGlobal Search! -->
    <enableReports>true</enableReports>    <!-- Allow Reports -->
    <enableActivities>true</enableActivities>  <!-- Tasks & Events -->
    <enableHistory>true</enableHistory>    <!-- Field History Tracking -->
    <enableBulkApi>true</enableBulkApi>
    <enableSharing>true</enableSharing>
</CustomObject>
```

> 🔴 **`enableSearch>false` = האובייקט לא מופיע ב-Global Search בכלל.**
> שגיאת SOSL: *"entity type X does not support search"* — תיקון: `enableSearch>true` + deploy.

#### deploy

```bash
sf project deploy start \
  --metadata "CompactLayout:Job_Catalog__c.JobCatalog_Compact" \
             "CustomObject:Job_Catalog__c" \
  --target-org moh-sandbox
```

> ⚠️ **חשוב**: deploy מטרגט `CompactLayout:ObjectName.CompactLayoutName` — שני חלקים עם נקודה.

**כללים:**
- `fullName` = שם הקובץ (ללא `.compactLayout-meta.xml`)
- `compactLayoutAssignment` ב-object XML = `fullName` של הקומפקט
- שדה ראשון ברשימה = מוצג הכי בולט בהיילייטס
- מקסימום 10 שדות (Salesforce מגביל)
- שדות Required נכללים ללא FLS מיוחד (כמו בכל מקום)

### FlexiPage XML — פורמט נכון לאובייקט Custom (v66.0)

#### שגיאות נפוצות ותיקוניהן

| בעיה | שגיאה | פתרון |
|---|---|---|
| שם קובץ עם אותיות גדולות | deploy fails | `.flexipage-meta.xml` (lowercase p) |
| `<label>` בFlexiPage | `Element label invalid` | השתמש ב-`<masterLabel>` |
| `<pageTemplate>` | `Not valid in version 66.0` | `<template><name>...</name></template>` |
| שם FlexiPage עם `__c` | `Cannot create component with namespace` | קרא `Job_Catalog_Record_Page` לא `Job_Catalog__c_Record_Page` |
| `mode>Replace` ללא parentFlexiPage | `parent region enabling that mode doesn't exist` | השמט `<mode>` לחלוטין |
| `mode>Append` ללא parentFlexiPage | `parent region enabling that mode doesn't exist` | השמט `<mode>` לחלוטין |
| `force:recordDetail` | `couldn't retrieve design time component info` | השתמש ב-`force:detailPanel` |
| FLS לשדה required | `You cannot deploy to a required field` | Required fields תמיד נגישים — אסור להוסיף להם FLS בפרופיל |
| `relatedListApiName=JobTenders` | `Could not find related list` | לcustom lookups הוסף `__r`: `JobTenders__r` |
| URL Object Manager עם `__c` | `character not allowed` | השתמש ב-Object ID במקום: `/ObjectManager/01IWm000000BLh7/PageLayouts/...` |

#### Template נכון לאובייקט Custom (standalone, ללא parentFlexiPage)

```
flexipage:recordHomeTemplateDesktop
```
Region names: `header`, `main`, `sidebar`
**אין להוסיף `<mode>`** — שדה mode אינו קיים ב-standalone pages.

#### XML מלא ועובד

```xml
<?xml version="1.0" encoding="UTF-8"?>
<FlexiPage xmlns="http://soap.sforce.com/2006/04/metadata">
    <flexiPageRegions>
        <itemInstances>
            <componentInstance>
                <componentInstanceProperties>
                    <name>collapsed</name>
                    <value>false</value>
                </componentInstanceProperties>
                <componentInstanceProperties>
                    <name>numVisibleActions</name>
                    <value>3</value>
                </componentInstanceProperties>
                <componentName>force:highlightsPanel</componentName>
                <identifier>force_highlightsPanel</identifier>
            </componentInstance>
        </itemInstances>
        <name>header</name>
        <type>Region</type>
    </flexiPageRegions>
    <flexiPageRegions>
        <itemInstances>
            <componentInstance>
                <componentName>force:detailPanel</componentName>
                <identifier>force_detailPanel</identifier>
            </componentInstance>
        </itemInstances>
        <name>main</name>
        <type>Region</type>
    </flexiPageRegions>
    <flexiPageRegions>
        <itemInstances>
            <componentInstance>
                <componentName>force:relatedListContainer</componentName>
                <identifier>force_relatedListContainer</identifier>
            </componentInstance>
        </itemInstances>
        <name>sidebar</name>
        <type>Region</type>
    </flexiPageRegions>
    <fullName>MyObject_Record_Page</fullName>
    <masterLabel>My Object Record Page</masterLabel>
    <template>
        <name>flexipage:recordHomeTemplateDesktop</name>
        <properties>
            <name>enablePageActionConfig</name>
            <value>false</value>
        </properties>
    </template>
    <sobjectType>MyObject__c</sobjectType>
    <type>RecordPage</type>
</FlexiPage>
```

#### Console Pages (Service Cloud) — הבדל ממה שלמעלה

| | Standalone | Console (e.g. Case) |
|---|---|---|
| Template | `flexipage:recordHomeTemplateDesktop` | `flexipage:recordHomeThreeColTemplateDesktop` |
| parentFlexiPage | אין | נדרש (e.g. `support__Case_rec_L_3col`) |
| mode בregions | **אין** | `Replace` |
| Regions | `header`, `main`, `sidebar` | `main`, `leftsidebar`, `rightsidebar` + Facets |
| force:detailPanel | כן | כן |

#### גילוי template נכון — שיטה אמינה

אם נדרש template חדש שאינו ידוע:
1. `sf org open --path "/lightning/setup/ObjectManager/OBJECT/LightningRecordPages/view"`
2. ב-App Builder: New → בחר template → Save
3. `sf project retrieve start --metadata "FlexiPage:NAME" --target-org moh-sandbox`
4. קרא ה-XML שחזר — שם ה-template מופיע ב-`<template><name>`

#### Dynamic Forms — חלופה ל-force:detailPanel

App Builder יוצר fieldInstances (Dynamic Forms) במקום force:detailPanel:
```xml
<!-- חלופה מתקדמת — שדות ישירות בFlexiPage (ללא תלות ב-Page Layout) -->
<flexiPageRegions>
    <itemInstances>
        <fieldInstance>
            <fieldInstanceProperties>
                <name>uiBehavior</name>
                <value>none</value>
            </fieldInstanceProperties>
            <fieldItem>Record.FieldName__c</fieldItem>
            <identifier>RecordFieldName_cField</identifier>
        </fieldInstance>
    </itemInstances>
    <name>Facet-UUID</name>
    <type>Facet</type>
</flexiPageRegions>
```
> ⚠️ **Dynamic Forms דורש "Upgrade Now" חד-פעמי ב-App Builder.** ה-`flexipage:fieldSection` שנפרוס מציג את השדות, אך **שדות ניתנים לגרירה** (Dynamic Forms אמיתי) מופעל רק אחרי לחיצת "Upgrade Now" ב-App Builder. לחיצה זו לא משנה את ה-XML (retrieve לפני ואחרי זהה), אך מפעילה מנגנון backend. **לא ניתן לעקוף דרך metadata בלבד.**

#### Related List ב-FlexiPage — שני רכיבים

| רכיב | שם | מאפיינים |
|---|---|---|
| **Related List – Single** (בסיסי) | `force:relatedListSingleContainer` | שדות ברירת מחדל, ללא שליטה |
| **Dynamic Related List – Single** (מומלץ) | `lst:dynamicRelatedList` | בחירת שדות, מיון, מספר שורות, כפתורים |

> ✅ **תמיד להשתמש ב-`lst:dynamicRelatedList`** — הוא ה-upgrade שמאפשר שליטה מלאה.

##### lst:dynamicRelatedList — XML מלא

```xml
<componentInstance>
    <componentInstanceProperties>
        <name>actionNames</name>
        <valueList>
            <valueListItems>
                <value>MassChangeOwner</value>  <!-- פעולות כפתור — ניתן להסיר/לשנות -->
            </valueListItems>
            <valueListItems>
                <value>New</value>
            </valueListItems>
        </valueList>
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>maxRecordsToDisplay</name>
        <value>10</value>  <!-- מספר שורות מקסימלי -->
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>parentFieldApiName</name>
        <value>Job_Catalog__c.Id</value>  <!-- ParentObject.Id -->
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>relatedListApiName</name>
        <value>JobTenders__r</value>  <!-- relationshipName + __r לcustom -->
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>relatedListDisplayType</name>
        <value>ADVGRID</value>  <!-- תמיד ADVGRID לdynamic -->
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>relatedListFieldAliases</name>
        <valueList>
            <valueListItems>
                <value>NAME</value>  <!-- שדות לתצוגה — NAME תמיד ראשון -->
            </valueListItems>
            <valueListItems>
                <value>Tender__c</value>  <!-- API name של כל שדה נוסף -->
            </valueListItems>
        </valueList>
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>relatedListLabel</name>
        <value>Job Tenders</value>  <!-- תווית לתצוגה -->
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>showActionBar</name>
        <value>true</value>
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>sortFieldAlias</name>
        <value>__DEFAULT__</value>  <!-- או API name של שדה למיון -->
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>sortFieldOrder</name>
        <value>Default</value>  <!-- Ascending / Descending / Default -->
    </componentInstanceProperties>
    <componentName>lst:dynamicRelatedList</componentName>
    <identifier>lst_dynamicRelatedList</identifier>
</componentInstance>
```

**פרמטרים עיקריים:**
- `relatedListFieldAliases` — `valueList` של שדות. `NAME` תמיד ראשון, אחריו API names.
- `sortFieldAlias` — `__DEFAULT__` או API name של שדה (למשל `CreatedDate`)
- `sortFieldOrder` — `Ascending` / `Descending` / `Default`
- `maxRecordsToDisplay` — ברירת מחדל 10
- `actionNames` — valueList של פעולות בכפתורי הכותרת. ניתן לרוקן לרשימה ריקה.

> ⚠️ **Column rejection (נסיון מהשטח):** Salesforce דוחה שדות מסוימים ב-`relatedListFieldAliases` גם כשהשדה קיים על האובייקט הילד. לדוגמה: `Monthly_Payment_Control__c` נדחה כעמודה ב-`Payment_Splits__r`. אין שגיאה ברורה — הדרך לגלות היא dry-run. הסר עמודות שנדחות.

#### flexipage:fieldSection — שדות Header ללא Dynamic Forms

ניתן להציג כמה שדות ספציפיים בראש ה-FlexiPage מחוץ ל-`force:detailPanel`, בלי לדרוש "Upgrade Now":

```xml
<componentInstance>
    <componentInstanceProperties>
        <name>fields</name>
        <valueList>
            <valueListItems><value>Employer__c</value></valueListItems>
            <valueListItems><value>Effective_To__c</value></valueListItems>
            <valueListItems><value>Daily_Hours__c</value></valueListItems>
        </valueList>
    </componentInstanceProperties>
    <componentInstanceProperties>
        <name>showLabel</name>
        <value>true</value>
    </componentInstanceProperties>
    <componentName>flexipage:fieldSection</componentName>
    <identifier>flexipage_fieldSection</identifier>
</componentInstance>
```

> שונה מ-Dynamic Forms: שדות ב-`flexipage:fieldSection` אינם ניתנים לגרירה ב-App Builder. Dynamic Forms האמיתי דורש "Upgrade Now" חד-פעמי ב-App Builder ואינו ניתן לאקטיבציה דרך metadata בלבד.

#### relatedList ב-Page Layout XML — פורמט נכון

```xml
<!-- Custom child object: ChildObject__c.LookupFieldName__c -->
<relatedLists>
    <fields>NAME</fields>
    <relatedList>Job_Tender__c.Catalog_Job__c</relatedList>
</relatedLists>

<!-- Files (כמעט לכל האובייקטים) -->
<relatedLists>
    <relatedList>RelatedFileList</relatedList>
</relatedLists>

<!-- Standard related lists -->
<relatedLists>
    <relatedList>RelatedActivityList</relatedList>
</relatedLists>
<relatedLists>
    <relatedList>RelatedHistoryList</relatedList>
</relatedLists>
```

| Related List | פורמט ב-XML | הערות |
|---|---|---|
| Custom child object | `ChildObject__c.LookupField__c` | השדה שמצביע לאובייקט ההורה |
| Files | `RelatedFileList` | להוסיף כמעט לכל אובייקט |
| Activities | `RelatedActivityList` | Tasks + Events |
| History | `RelatedHistoryList` | Field History Tracking |
| Standard objects | `Related[ObjectName]List` | e.g. `RelatedContactList` |

> ⚠️ **FlexiPage vs Layout**: הפורמטים שונים!
> - FlexiPage (`lst:dynamicRelatedList`): `relatedListApiName=JobTenders__r`
> - Layout XML: `relatedList=Job_Tender__c.Catalog_Job__c`

> 📌 **Files reminder**: יש להוסיף `RelatedFileList` לכל layout של אובייקטים ראשיים (Employment, Job_Catalog, MPC, Account, Case, InternalOrganizationUnit וכד').

#### FLS בפרופיל — כלל חשוב
**שדות Required** (`<required>true</required>`) — **אסור** להוסיף להם `fieldPermissions` בפרופיל.
Salesforce ידחה את הdeploy עם: `You cannot deploy to a required field`.
Required fields תמיד גלויים ועריכתיים לכל המשתמשים — אין צורך בFLS.

**שדות שמותר ב-FLS:** Optional fields, Picklist fields (לא required), Lookup fields (לא required).

### Quick Action שמפעיל Screen Flow

> ❌ **DEPLOYMENT LIMITATION (v66):** `type=Flow` Quick Actions **לא ניתנות לפריסה דרך Metadata API**.
> - `<flow>FlowApiName</flow>` — אלמנט לא תקין (XML schema error)
> - `<flowDefinition>FlowApiName</flowDefinition>` — שגיאת runtime: "לא ניתן להגדיר את השדה לסוג זרימה"
> - **יש ליצור ידנית דרך Setup UI:**
>   1. Setup → Object Manager → [Object] → Buttons, Links, and Actions → New Action
>   2. Action Type: **Flow** → בחר Flow → Label → Save
>   3. הוסף ל-Page Layout דרך Object Manager → Page Layouts
> - **חלופה דרך metadata:** LWC עם `<lightning-flow flow-api-name="FlowApiName">` מוטמע בRecord Page

```xml
<!-- force-app/main/default/quickActions/ObjectName.ActionName.quickAction-meta.xml -->
<!-- ⚠️ שים לב: אין אלמנט <flow> — הוא לא תקין. יצירה דרך UI בלבד -->
<QuickAction xmlns="http://soap.sforce.com/2006/04/metadata">
    <actionSubtype>ScreenAction</actionSubtype>
    <label>תווית הכפתור</label>
    <optionsCreateFeedItem>false</optionsCreateFeedItem>
    <targetObject>Account</targetObject>
    <type>Flow</type>
</QuickAction>
```

> ⚠️ `type=Flow` + `actionSubtype=ScreenAction` — לפלואו מסוג `processType=Flow` (Screen Flow) בלבד. Autolaunched Flow אינו ניתן לקריאה ישירה — יש לעטוף ב-Screen Flow.

### Autolaunched Flow מכפתור — עטיפה ב-Screen Flow

כאשר יש כפתור שאמור להפעיל Autolaunched Flow (AF_*):
1. צור `SCR_*` פשוט: מסך אישור → Subflow → מסך הצלחה
2. צור Quick Action מסוג `ScreenAction` שמפנה ל-`SCR_*`
3. הוסף ל-FlexiPage או Page Layout

### FLS — כלל לכל שדה חדש

בכל יצירת שדה חדש (optional, לא required, לא MasterDetail):
```xml
<!-- מוסיפים לשני הפרופילים באותו deploy -->
<fieldPermissions>
    <editable>true</editable>
    <field>ObjectName__c.FieldName__c</field>
    <readable>true</readable>
</fieldPermissions>
```
**שדות שאסור להכניס ל-FLS:** required=true, MasterDetail, Formula fields.

---

## 11. Lightning App (CustomApplication)

### מבנה קבצים
```
force-app/main/default/
  applications/
    MOH_HR.app-meta.xml
  staticresources/
    MOH_Logo.resource              ← קובץ PNG/JPG עצמו
    MOH_Logo.resource-meta.xml     ← מטה
```

### XML — CustomApplication (Lightning, לא Console)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomApplication xmlns="http://soap.sforce.com/2006/04/metadata">
    <defaultLandingTab>Job_Catalog__c</defaultLandingTab>
    <description>תיאור האפליקציה</description>
    <formFactors>Large</formFactors>
    <formFactors>Small</formFactors>
    <isNavAutoTempTabsDisabled>false</isNavAutoTempTabsDisabled>
    <isNavPersonalizationDisabled>false</isNavPersonalizationDisabled>
    <label>HR - משרד הבריאות</label>
    <logo>MOH_Logo</logo>
    <navType>Standard</navType>
    <tabs>Job_Catalog__c</tabs>
    <tabs>Job_Tender__c</tabs>
    <tabs>Tender__c</tabs>
    <tabs>Annual_Feedback__c</tabs>
    <uiType>Lightning</uiType>
</CustomApplication>
```

### XML — StaticResource (לוגו)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<StaticResource xmlns="http://soap.sforce.com/2006/04/metadata">
    <cacheControl>Public</cacheControl>
    <contentType>image/png</contentType>
    <description>MOH Logo</description>
</StaticResource>
```

### כללים קריטיים

| שגיאה | פתרון |
|---|---|
| `logoUrl invalid` | שדה לוגו ב-Lightning App הוא `<logo>`, לא `<logoUrl>` |
| `FormFactors is required` | חובה להוסיף `<formFactors>Large</formFactors>` + `<formFactors>Small</formFactors>` |
| `navType=Console` | לאפליקציה רגילה (לא console) השתמש `<navType>Standard</navType>` |

### Deploy

```bash
# קודם Static Resource, אחר כך Application
sf project deploy start --metadata "StaticResource:MOH_Logo" --target-org moh-sandbox
sf project deploy start --metadata "CustomApplication:MOH_HR" --target-org moh-sandbox
```

> ⚠️ `<logo>` מפנה לשם ה-Static Resource בלבד (ללא `/resource/`). הלוגו מוצג כ-icon קטן באפליקציה ב-App Launcher.
> 
> ⚠️ תמונת רקע (Background Image) אינה נתמכת ב-CustomApplication metadata — ניתן להגדיר בלבד דרך App Manager UI.

---

## 12. Go-Live Checklist

- [ ] All Profiles received CRUD + FLS on all relevant objects
- [ ] PSS Permission Sets (Public_Sector_Solutions_Admin + PublicSectorAccessPSL) assigned to Admin
- [ ] OWD configured before first data load
- [ ] Validation Rules tested with edge cases (blank, null, boundary dates)
- [ ] Custom Tabs visible in App Launcher
- [ ] Lookup Fields point to correct object (PSS Standard preferred over Custom)
- [ ] Picklist API Values documented and given to data team before load
- [ ] Hebrew Reference Data (Hebrew date table) loaded before production data
- [ ] Deploy order verified — Profiles always last
- [ ] Permission Set Licenses (Industries User, OmniStudio User, Shield) assigned to all relevant users


---

## 13. Cross-Domain Interactions (Critical — Often Missed)

### Validation Rules + Flow — Fault Path Catches VR Errors
- When a Flow executes a DML (Create/Update Record) on an object that has active Validation Rules:
  - If the VR condition is true → the save is blocked
  - The Flow **Fault Path catches this as an error** — `{!$Flow.FaultMessage}` contains the VR message
  - If no Fault Path → the entire transaction rolls back with an uncaught error
- **Multiple VRs on same object**: all active VRs are evaluated simultaneously; if any fail, the save is blocked
- VR evaluation order is not guaranteed — do not write VRs that depend on each other's order
- Testing pattern: after adding a new VR, trigger it via a Flow or Bulk Load to verify Fault Path handles it correctly

### Validation Rules + Bulk Data Load
- Validation Rules fire during Bulk Data Load (Data Loader, SFDC CLI `data upsert`)
- A VR failure causes the **entire batch row to fail** (not just the field) → row appears in failed-records CSV
- Common trap: VR that checks a cross-object condition (VLOOKUP) — related record may not exist during load sequence
- Load order matters: load parent/lookup records BEFORE child records that have cross-object VRs
- To temporarily bypass VRs during a one-time load: deactivate VR → load → reactivate. Never leave deactivated.

### Bulk Load + Sharing (OWD = Private)
- Integration User needs record access to UPDATE or DELETE records it doesn't own
- If OWD = Private and Integration User has no Role → it can only see records it created
- Fix: give Integration User's Profile `View All` / `Modify All` on the object, OR create a Sharing Rule to `allInternalUsers`, OR add Integration User to a Public Group that has Sharing Rule access
- Best practice for Integration Profiles: set `View All` + `Modify All` on all objects they must load into

### Validation Rules — Evaluation Behavior on Bypass
- Flows run in system context → Validation Rules STILL fire (system context does not bypass VRs)
- Apex with `DML.Database.insert(..., false)` (allOrNothing=false) → VR failures appear in SaveResult, no exception
- `Database.insert(..., true)` (default) → VR failure throws exception → Fault Path or try/catch required

### PSS Objects — Always Check Before Custom
Before creating any Custom Object, verify PSS does not already provide it:
```
PSS Standard Objects (check these first):
Employee2          → Employee / Person record
Employment         → Employment position / job assignment
InternalOrganizationUnit → Department / Org Unit
PersonLifeEvent    → HR events (leave, absence, lifecycle events)
JobRequisition     → Open job position / vacancy
JobProfile         → Job role definition / job catalog

If PSS provides it: use it. Never create a custom duplicate.
```

### Naming Conventions — Iron Rules
- API names: English only, no spaces, underscore allowed. Example: `Employee_Status__c`
- Object labels (displayed in UI): Hebrew via Translation only — not in `<label>` tag in multilingual orgs
- Validation Rule names: `[VR-XXX]` format in error message (3-digit zero-padded), human-readable Hebrew message
- Picklist values: API values in English (`Active`, `Inactive`); Hebrew via Translation
- Flow names: descriptive, include object name and trigger type. Example: `Employee_OnCreate_SetDefaults`
