# Salesforce UI — SKILL

## When to use this skill
Read this file at the start of any task involving:
- Page Layouts, Record Pages (FlexiPage), Compact Layouts
- Lightning Apps, Navigation, Tabs
- List Views, Search Layouts
- Translations, Labels, Hebrew UI
- UI Permissions — who sees what, Tab visibility, App access
- Any time a new UI component is created — permissions must be closed in the same deploy

---

## GOLDEN RULE — UI Permissions Checklist

> ⚠️ **Every UI component created must be completed in the same deploy.**
> Claude Code tends to create the component and forget to assign permissions. This is wrong.

For every new UI element, complete this checklist **before** marking the task done:

| Component | What must be added |
|---|---|
| New Custom Tab | `tabVisibilities` in every relevant Profile |
| New FlexiPage | `pageAccesses` in Profile OR FlexiPage Assignment by App/Profile |
| New Lightning App | `applicationVisibilities` in every relevant Profile |
| New List View | Sharing set (Public, Queue, or Roles) |
| New Quick Action | Added to Page Layout AND FlexiPage |
| New Report / Dashboard Folder | Folder sharing to relevant Public Group / Role |

**Pattern for closing permissions in same deploy:**
```
1. Create component metadata
2. Add permissions to Profile XML (same folder, same deploy)
3. Deploy together in one command
```

---

## 1. Translations & Hebrew UI

### 🔴 Golden Rule — English in Metadata, Hebrew in Translations ONLY

> **`<label>` inside any metadata file (object, field, tab, app, layout section) is the DEFAULT label.**
> It is shown to ALL users when no language-specific translation exists for them.
>
> - ✅ `<label>Employment</label>` in `.tab-meta.xml` + `<label>העסקות</label>` in `iw.translation` → correct: Hebrew users see Hebrew, English users see English
> - ❌ `<label>העסקות</label>` in `.tab-meta.xml` → WRONG: everyone sees Hebrew regardless of language
>
> **Never write Hebrew directly in any `<label>` tag in metadata files.**

### Architecture — Three Distinct Mechanisms

Salesforce Hebrew translation uses **three separate mechanisms** — each for a different scope:

| Mechanism | Metadata Type | File Location | What it translates |
|---|---|---|---|
| **Object + Field + Picklist translations** | `CustomObjectTranslation` | `objectTranslations/ObjName__c-iw.objectTranslation-meta.xml` | Object label, field labels, picklist values, validation rules, layout sections |
| **App + Tab + Quick Action translations** | `Translations` | `translations/iw.translation-meta.xml` | App name, tab names, quick actions, custom labels |
| **Direct label** (single-language only) | Field/Object metadata | `objects/ObjName__c/...` | English label shown as-is — breaks in Translation Workbench |

> ⚠️ **Key distinction:**
> - **Object name (singular/plural/definite article)** → `CustomObjectTranslation` → `<caseValues>` (NO root `<label>` — it's invalid)
> - **Field label translation** → `CustomObjectTranslation` → `<fields>` → `<label>` (must come BEFORE `<name>`)
> - **Tab label translation** → `iw.translation-meta.xml` → `<customTabs>`
> - **Object name gender** → `CustomObjectTranslation` → `<gender>` (Masculine/Feminine/Neuter)
> - **Name field label** → `CustomObjectTranslation` → `<nameFieldLabel>`
> 
> These are different files and different elements, even though they're all "translations."

> ⚠️ Hebrew labels set directly in object/field `<label>` tags work in English orgs but break in multilingual orgs. Always use Translations for Hebrew.

### Enabling Hebrew in the Org

**Step 1 — Manual (required first):**
```
Setup → Translation Language Settings → Add Language → Hebrew (iw)
```

After enabling, Translation Workbench becomes active and `.translation-meta.xml` files can be deployed.

> ⚠️ Translation Workbench must be enabled in the org BEFORE deploying translation metadata. Deploying `objectTranslation` files to an org without Hebrew enabled will cause an error.

### Translation File Structure

```
force-app/main/default/
  translations/
    iw.translation-meta.xml                              ← App, Tab, Quick Action names
  objectTranslations/
    Employment__c-iw.objectTranslation-meta.xml          ← Employment fields + picklists
    Job_Catalog__c-iw.objectTranslation-meta.xml
    Annual_Feedback__c-iw.objectTranslation-meta.xml
    Hebrew_Calendar__c-iw.objectTranslation-meta.xml
    Skill__c-iw.objectTranslation-meta.xml
    EmployeeSkill__c-iw.objectTranslation-meta.xml
    Tariff_Line__c-iw.objectTranslation-meta.xml
    Tariff_Adjustment__c-iw.objectTranslation-meta.xml
    Monthly_Payment_Control__c-iw.objectTranslation-meta.xml
    Monthly_Adjustment_Snapshot__c-iw.objectTranslation-meta.xml
    PurchaseOrder__c-iw.objectTranslation-meta.xml
    OrderPriority__c-iw.objectTranslation-meta.xml
    MerkavaOrderSnapshot__c-iw.objectTranslation-meta.xml
    Tender__c-iw.objectTranslation-meta.xml
    Job_Tender__c-iw.objectTranslation-meta.xml
    Account-iw.objectTranslation-meta.xml                ← Supplier custom fields
    Employee2-iw.objectTranslation-meta.xml              ← Employee (PersonAccount) label
```

### objectTranslation — Full Working Example (Verified for Hebrew)

**Element order is MANDATORY — must be alphabetical:**
`caseValues` → `fields` → `gender` → `nameFieldLabel`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObjectTranslation xmlns="http://soap.sforce.com/2006/04/metadata">

    <!-- Object name forms (singular/plural/definite article) -->
    <!-- article enum for Hebrew: None (indefinite) | Definite (with ה') -->
    <caseValues>
        <article>None</article>
        <plural>false</plural>
        <value>משרה</value>           <!-- singular indefinite -->
    </caseValues>
    <caseValues>
        <article>None</article>
        <plural>true</plural>
        <value>משרות</value>          <!-- plural indefinite -->
    </caseValues>
    <caseValues>
        <article>Definite</article>
        <plural>false</plural>
        <value>המשרה</value>          <!-- singular definite (ה' הידיעה) -->
    </caseValues>
    <caseValues>
        <article>Definite</article>
        <plural>true</plural>
        <value>המשרות</value>         <!-- plural definite -->
    </caseValues>

    <!-- Field translations: <label> MUST come before <name> -->
    <fields>
        <label>תאריך התחלה</label>
        <name>Start_Date__c</name>
    </fields>
    <fields>
        <label>סטטוס</label>
        <name>Status__c</name>
        <!-- Picklist translations ONLY if the picklist values have English labels -->
        <!-- If picklist labels are already in Hebrew in field definitions → SKIP picklistValues -->
        <!-- <picklistValues><masterLabel>Active</masterLabel><translation>פעיל</translation></picklistValues> -->
    </fields>

    <!-- Gender: Masculine | Feminine | Neuter -->
    <gender>Feminine</gender>

    <!-- Name field label (the "Record Name" field shown in Rename Tabs and Labels) -->
    <nameFieldLabel>מזהה משרה</nameFieldLabel>

    <!-- Validation Rule translations (optional) -->
    <!-- <validationRules>
        <name>Validate_Start_Before_End</name>
        <errorMessage>תאריך סיום חייב להיות אחרי תאריך התחלה</errorMessage>
    </validationRules> -->

</CustomObjectTranslation>
```

### Translation — Lessons Learned (Critical)

| What | Rule | Why |
|---|---|---|
| Root `<label>` | ❌ NOT valid | Object name goes via `<caseValues>`, not `<label>` |
| `<startsWith>` | ❌ NOT valid for Hebrew | Salesforce error: "Cannot specify startsWith for language HEBREW" |
| `<article>` enum | `None` / `Definite` only | Not `true`/`false`, not `Indefinite` |
| Field `<label>` position | Must come **before** `<name>` | Schema validation error if after |
| `<picklistValues>` | Skip if labels already in Hebrew | Only needed when translating English→Hebrew |
| `<language>` in iw.translation | ❌ NOT valid | Language inferred from filename (`-iw`) |
| Element ordering | Alphabetical | caseValues → fields → gender → nameFieldLabel |
| `<gender>` | Valid: Masculine / Feminine / Neuter | English default = Neuter |

### iw.translation-meta.xml — Standard Labels

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Translations xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- NO <language> element — it is inferred from the filename (iw.translation-meta.xml) -->

    <!-- Custom App translations -->
    <customApplications>
        <label>מערכת משאבי אנוש</label>
        <name>HR_App</name>
    </customApplications>

    <!-- Custom Tab translations -->
    <customTabs>
        <label>העסקות</label>
        <name>Employment__c</name>
    </customTabs>

    <!-- Quick Action translations -->
    <quickActions>
        <label>הקצאת כישורים</label>
        <name>Employee2.Assign_Skills</name>
    </quickActions>

    <!-- Custom Label translations (used in LWC / Flow) -->
    <customLabels>
        <label>שמירה הצליחה</label>
        <name>Save_Success_Message</name>
    </customLabels>

</Translations>
```

### RTL (Right-to-Left) — Hebrew Display

Salesforce Lightning UI automatically flips to RTL when the user's language is Hebrew (`iw`). This affects:
- Field label alignment
- Page layout direction
- Navigation

> ⚠️ **LWC components do NOT automatically flip to RTL.** Must add `dir="rtl"` in HTML or use CSS `direction: rtl`.

```html
<!-- LWC template — Hebrew RTL -->
<template>
    <div dir="rtl" class="slds-p-around_medium">
        <lightning-input label="שם עובד" value={employeeName}></lightning-input>
    </div>
</template>
```

### Custom Labels — for use in Flow & LWC

```
Setup → Custom Labels → New
```

Or via metadata:
```xml
<!-- force-app/main/default/labels/CustomLabels.labels-meta.xml -->
<CustomLabels xmlns="http://soap.sforce.com/2006/04/metadata">
    <labels>
        <fullName>Save_Success_Message</fullName>
        <language>en_US</language>
        <protected>false</protected>
        <shortDescription>Save Success Message</shortDescription>
        <value>Record saved successfully</value>
    </labels>
</CustomLabels>
```

In Flow: `{!$Label.c.Save_Success_Message}`
In LWC: `import SAVE_MSG from '@salesforce/label/c.Save_Success_Message';`

---

## 2. Lightning Apps — Structure & Permissions

### App Structure (NavigationMixin)

```
force-app/main/default/applications/
  HR_App.app-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomApplication xmlns="http://soap.sforce.com/2006/04/metadata">
    <defaultLandingTab>Employment__c</defaultLandingTab>
    <description>HR Management System</description>
    <formFactors>Large</formFactors>     <!-- Large = Desktop, Small = Mobile -->
    <isNavAutoTempTabsDisabled>false</isNavAutoTempTabsDisabled>
    <isNavPersonalizationDisabled>false</isNavPersonalizationDisabled>
    <label>מערכת משאבי אנוש</label>
    <navType>Standard</navType>          <!-- Standard | Console -->
    <tabs>standard-home</tabs>
    <tabs>Employment__c</tabs>
    <tabs>Employee2</tabs>
    <tabs>standard-InternalOrganizationUnit</tabs>
    <tabs>Job_Catalog__c</tabs>
    <tabs>standard-report</tabs>
    <tabs>standard-dashboard</tabs>
    <uiType>Lightning</uiType>
    <utilityBar>HR_App_UtilityBar</utilityBar>   <!-- optional -->
</CustomApplication>
```

### Logo — Lightning Experience Limitation

> ⚠️ **`<logo>` ב-CustomApplication metadata עובד לאפליקציות Classic בלבד.**
> ב-Lightning Experience, הלוגו ב-App Launcher **אינו נקרא מה-metadata** — חייבים להגדיר דרך UI:
> 
> **Setup → App Manager → [שם האפליקציה] → Edit → Branding → Upload Image**
> 
> ניתן לכלול `<logo>MOH_Logo</logo>` ב-XML (Static Resource name) — זה לא גורם לשגיאה,
> אבל לא יוצג ב-Lightning. הוא יוצג רק אם המשתמש עובר ל-Classic.

### App Permissions in Profile — MANDATORY

> ⚠️ Creating an App without assigning it in the Profile = no one can see it in the App Launcher.

```xml
<!-- In Profile XML -->
<applicationVisibilities>
    <application>HR_App</application>
    <default>true</default>     <!-- true = opens automatically for this profile -->
    <visible>true</visible>
</applicationVisibilities>
```

**Rules:**
- Only **one** app can have `<default>true</default>` per Profile
- `<visible>false</visible>` hides the app from App Launcher
- Must be added to every Profile that needs access

### Tab Visibility in App vs. Profile

Tabs can be controlled in two places:

| Where | Effect |
|---|---|
| In `<tabs>` of App XML | Tab appears in this app's navigation |
| In Profile `<tabVisibilities>` | Tab is visible/hidden globally for this profile |

Both must be set. A tab in the App but hidden in Profile = user won't see it.

```xml
<!-- Profile tabVisibilities -->
<tabVisibilities>
    <tab>Employment__c</tab>
    <visibility>DefaultOn</visibility>   <!-- DefaultOn | DefaultOff | Hidden -->
</tabVisibilities>
<tabVisibilities>
    <tab>standard-InternalOrganizationUnit</tab>
    <visibility>DefaultOn</visibility>
</tabVisibilities>
```

| Visibility Value | Meaning |
|---|---|
| `DefaultOn` | Tab visible by default, user can hide |
| `DefaultOff` | Tab hidden by default, user can show |
| `Hidden` | Tab completely hidden, user cannot show |

---

## 3. Custom Tabs

### ⚠️ CRITICAL: Label Language Rule

> **`<label>` in tab metadata = the DEFAULT label shown to ALL users regardless of language.**
> 
> - **English in `<label>`** → English users see English, Hebrew users see Hebrew (via translation)
> - **Hebrew in `<label>`** → EVERYONE sees Hebrew, including English-language users — **WRONG**
>
> **Rule: Always write English in `<label>` in all metadata files (tabs, fields, objects, apps).**
> **Hebrew belongs ONLY in `iw.translation-meta.xml` and `objectTranslation` files.**

### Tab File — Custom Object

```
force-app/main/default/tabs/Employment__c.tab-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomTab xmlns="http://soap.sforce.com/2006/04/metadata">
    <customObject>true</customObject>
    <fullName>Employment__c</fullName>
    <label>Employment</label>          <!-- MUST be English — Hebrew goes in iw.translation-meta.xml ONLY -->
    <motif>Custom74: Handshake</motif>  <!-- Icon -->
</CustomTab>
```

Then in `translations/iw.translation-meta.xml`:
```xml
<customTabs>
    <label>העסקות</label>              <!-- Hebrew shown ONLY to Hebrew-language users -->
    <name>Employment__c</name>
</customTabs>
```

### Available Icons (motif)

Common useful icons:
- `Custom74: Handshake` — agreements, employment
- `Custom32: People` — employees, HR
- `Custom18: Building` — organization
- `Custom2: Briefcase` — jobs
- `Custom15: Dollar` — financial
- `Custom60: Books` — catalog

> Full list: Setup → Tabs → New → Icon selector

### Tab Rules

| Object Type | Tab File Needed | Profile Reference |
|---|---|---|
| Custom Object (`__c`) | ✅ Yes — create `.tab-meta.xml` | `<tab>Employment__c</tab>` |
| PSS Standard Object | ❌ No — already exists | `<tab>standard-InternalOrganizationUnit</tab>` |
| Standard Salesforce Object | ❌ No | `<tab>standard-Account</tab>` |
| Report | ❌ No | `<tab>standard-report</tab>` |
| Dashboard | ❌ No | `<tab>standard-dashboard</tab>` |

### ⚠️ CRITICAL: Tab Visibility — Must Deploy CustomTab + Profile + App Together

> **Profile-only deploy/retrieve can silently omit `tabVisibilities`.**
> Even if the Profile XML locally has correct `DefaultOn` visibility, a deploy that includes only Profiles may not write `tabVisibilities` to the org.

**Rule:** To fix or set tab visibility, always deploy the relevant `CustomTab` metadata in the **same package** as the Profile and App. This forces Salesforce to evaluate tab visibility as part of the combined deployment.

```bash
# Correct — all three components together
sf project deploy start \
  --metadata "Profile:HR_Manager" \
             "Profile:HR_ReadOnly" \
             "CustomApplication:MOH_HR" \
             "CustomTab:Employment_Record__c" \
             "CustomTab:PurchaseOrder__c" \
  --target-org alias
```

> ⚠️ Do NOT include `<object>` element inside a CustomTab `<customObject>true</customObject>` file — it causes a deploy error. Use only `<customObject>`, `<fullName>`, `<label>`, and `<motif>`.

---

## 4. Page Layouts

### Page Layout — Complete Structure

```
force-app/main/default/layouts/
  Employment__c-Employment Layout.layout-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Layout xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Employment__c-Employment Layout</fullName>

    <!-- Quick Action buttons at top of page -->
    <quickActionList>
        <quickActionListItems>
            <quickActionName>FeedItem.TextPost</quickActionName>
        </quickActionListItems>
        <quickActionListItems>
            <quickActionName>Employment__c.Assign_Skills</quickActionName>
        </quickActionListItems>
    </quickActionList>

    <!-- Sections -->
    <layoutSections>
        <customLabel>false</customLabel>
        <detailHeading>true</detailHeading>
        <editHeading>true</editHeading>
        <label>General Information</label>   <!-- English — translated via objectTranslation -->
        <layoutColumns>
            <layoutItems>
                <behavior>Required</behavior>    <!-- Required | Edit | Readonly -->
                <field>Employee__c</field>
            </layoutItems>
            <layoutItems>
                <behavior>Edit</behavior>
                <field>Start_Date__c</field>
            </layoutItems>
        </layoutColumns>
        <layoutColumns>
            <layoutItems>
                <behavior>Edit</behavior>
                <field>Status__c</field>
            </layoutItems>
            <layoutItems>
                <behavior>Edit</behavior>
                <field>End_Date__c</field>
            </layoutItems>
        </layoutColumns>
        <style>TwoColumnsTopToBottom</style>   <!-- OneColumn | TwoColumnsTopToBottom | TwoColumnsLeftToRight -->
    </layoutSections>

    <!-- Related Lists -->
    <relatedLists>
        <fields>NAME</fields>
        <relatedList>Tariff_Line__c.Employment__c</relatedList>   <!-- Custom: ChildObject.LookupField -->
    </relatedLists>
    <relatedLists>
        <relatedList>RelatedFileList</relatedList>    <!-- Files — always include -->
    </relatedLists>
    <relatedLists>
        <relatedList>RelatedActivityList</relatedList>  <!-- Tasks & Events — if enableActivities=true -->
    </relatedLists>
    <relatedLists>
        <relatedList>RelatedHistoryList</relatedList>   <!-- if enableHistory=true -->
    </relatedLists>

    <!-- Show Submit for Approval if Approval Process exists -->
    <showSubmitAndSaveButton>false</showSubmitAndSaveButton>

</Layout>
```

### Layout Assignment in Profile — MANDATORY

> ⚠️ Deploying a Layout without assigning it in the Profile = Salesforce uses the default layout.

```xml
<!-- In Profile XML -->
<layoutAssignments>
    <layout>Employment__c-Employment Layout</layout>
    <object>Employment__c</object>
</layoutAssignments>
```

For RecordType-specific layouts:
```xml
<layoutAssignments>
    <layout>Employment__c-Vendor Layout</layout>
    <object>Employment__c</object>
    <recordType>Employment__c.Vendor</recordType>
</layoutAssignments>
```

### Layout Section Styles

| Style | Description |
|---|---|
| `OneColumn` | Single column — full width |
| `TwoColumnsTopToBottom` | Two columns, items fill top-to-bottom |
| `TwoColumnsLeftToRight` | Two columns, items fill left-to-right |
| `CustomLinks` | Section for custom links only |

### Field Behavior in Layout

| Behavior | Meaning |
|---|---|
| `Required` | Mandatory in layout (visual indicator) — does NOT replace field-level required |
| `Edit` | Editable in Edit mode |
| `Readonly` | Visible but not editable |

---

## 5. List Views

### List View Metadata

```
force-app/main/default/objects/Employment__c/listViews/
  Active_Employments.listView-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ListView xmlns="http://soap.sforce.com/2006/04/metadata">
    <fullName>Active_Employments</fullName>
    <label>Active Employments</label>
    <filterScope>Everything</filterScope>   <!-- see table below -->
    <filters>
        <field>Status__c</field>
        <operation>equals</operation>       <!-- equals | notEqual | contains | greaterThan | etc. -->
        <value>Active</value>               <!-- API Value — not Hebrew label -->
    </filters>
    <columns>Name</columns>
    <columns>Employee__c</columns>
    <columns>Start_Date__c</columns>
    <columns>Status__c</columns>
    <columns>InternalOrganizationUnit__c</columns>
    <sharedTo>
        <allInternalUsers/>                 <!-- see sharing options below -->
    </sharedTo>
</ListView>
```

### filterScope Values

| Value | Meaning |
|---|---|
| `Everything` | All records the user can see |
| `Mine` | Records owned by current user |
| `MyTeam` | Records owned by user's team |
| `Queue` | Records in a specific Queue |
| `Delegated` | Delegated records |

### List View Sharing (sharedTo)

```xml
<!-- All internal users -->
<sharedTo><allInternalUsers/></sharedTo>

<!-- Specific Role -->
<sharedTo><role>HR_Manager</role></sharedTo>

<!-- Role and subordinates -->
<sharedTo><roleAndSubordinates>HR_Manager</roleAndSubordinates></sharedTo>

<!-- Public Group -->
<sharedTo><group>PG_HR_Managers</group></sharedTo>

<!-- Only owner (private) -->
<sharedTo><me/></sharedTo>
```

> ⚠️ **List Views with `<me/>` sharing are private — only the creator sees them.** For production, always use `<allInternalUsers/>` or a specific group/role unless specifically required to be private.

### Search Layouts — Which fields appear in Search Results

```xml
<!-- In object-meta.xml or separate searchLayouts -->
<searchLayouts>
    <searchResultsAdditionalFields>Status__c</searchResultsAdditionalFields>
    <searchResultsAdditionalFields>Employee__c</searchResultsAdditionalFields>
    <searchResultsAdditionalFields>Start_Date__c</searchResultsAdditionalFields>
    <lookupDialogsAdditionalFields>Status__c</lookupDialogsAdditionalFields>
    <lookupDialogsAdditionalFields>Employee__c</lookupDialogsAdditionalFields>
    <lookupFilterFields>Name</lookupFilterFields>
    <lookupFilterFields>Status__c</lookupFilterFields>
</searchLayouts>
```

| Tag | Where it appears |
|---|---|
| `searchResultsAdditionalFields` | Global Search results columns |
| `lookupDialogsAdditionalFields` | Lookup popup columns |
| `lookupFilterFields` | Lookup popup search fields |

---

## 6. Record Pages (FlexiPage) — Full Reference

> See also SKILL_PSS_SF.md Section 10 for detailed FlexiPage XML patterns.

### FlexiPage Assignment — MANDATORY

> ⚠️ Deploying a FlexiPage without assigning it = Salesforce uses the default Record Page.

Assignment can be:
1. **Org Default** — applies to all profiles and apps
2. **App Default** — applies when viewed in a specific app
3. **App + Profile** — most specific, highest priority

```
force-app/main/default/flexipages/
  Employment_Record_Page.flexipage-meta.xml
```

### FlexiPage Org-Default Activation — Exact Metadata Mechanism

> 📌 **Org-default activation is NOT stored in the FlexiPage file itself.**
> It is stored in the **object's `object-meta.xml`** as `actionOverrides` for the `View` action.

```xml
<!-- force-app/main/default/objects/Employment_Record__c/Employment_Record__c.object-meta.xml -->
<actionOverrides>
    <actionName>View</actionName>
    <content>Employment_Record_Record_Page</content>
    <formFactor>Large</formFactor>
    <skipRecordTypeSelect>false</skipRecordTypeSelect>
    <type>Flexipage</type>
</actionOverrides>
<actionOverrides>
    <actionName>View</actionName>
    <content>Employment_Record_Record_Page</content>
    <formFactor>Small</formFactor>
    <skipRecordTypeSelect>false</skipRecordTypeSelect>
    <type>Flexipage</type>
</actionOverrides>
```

**How to verify activation exists in the org:**
- Retrieve the object metadata and check if `View` action overrides exist with `type=Flexipage` and `content=<PageName>`.
- An object WITHOUT activation returns `type=Default` with no `content` field.
- An object WITH activation returns `type=Flexipage` and `content` pointing to the FlexiPage name.

**Two components must be deployed together:**
1. `FlexiPage` — the page structure.
2. `CustomObject` (with `actionOverrides`) — the activation.

> ⚠️ When retrieving object metadata for this purpose, review diffs carefully. Salesforce may return unrelated default action overrides, base-language label diffs, and field/list-view churn. Keep only the intentional activation metadata.

> 📌 **Practical rule:** After deploying a FlexiPage, always run:
> ```bash
> sf org open --path "/lightning/setup/ObjectManager/OBJECT_API/LightningRecordPages/view" --target-org moh-sandbox
> ```
> Then assign the page as Org Default or App Default via the UI, and retrieve the assignment.

### FlexiPage Checklist — Every New Page

- [ ] `<fullName>` does NOT contain `__c`
- [ ] `<masterLabel>` used (not `<label>`)
- [ ] Template is `flexipage:recordHomeTemplateDesktop` for standalone custom objects
- [ ] `<mode>` tag is ABSENT for standalone pages — `mode=Replace` and `mode=Append` cause deploy errors in standalone (non-Console) pages
- [ ] `force:detailPanel` used (not `force:recordDetail`)
- [ ] `lst:dynamicRelatedList` used for related lists (not `force:relatedListSingleContainer`)
- [ ] `actionOverrides` added to object-meta.xml for Large and Small form factors (org-default activation)
- [ ] Dry-run before real deploy — some related list field aliases are silently rejected even when the field exists
- [ ] Page assigned in App Builder after deploy
- [ ] Quick Actions added to both Page Layout AND FlexiPage

---

## 7. Quick Actions — Complete Reference

### Types of Quick Actions

| Type | Use case | `<type>` | `<actionSubtype>` |
|---|---|---|---|
| Create Record | Create related record | `Create` | — |
| Update Record | Update current record | `Update` | — |
| Screen Flow | Launch a Flow from button | `Flow` | `ScreenAction` |
| Log a Call | Activity — call log | `LogACall` | — |
| Send Email | Activity — email | `SendEmail` | — |

### Quick Action File

```
force-app/main/default/quickActions/
  Employment__c.Assign_Skills.quickAction-meta.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<QuickAction xmlns="http://soap.sforce.com/2006/04/metadata">
    <actionSubtype>ScreenAction</actionSubtype>
    <label>הקצאת כישורים</label>
    <optionsCreateFeedItem>false</optionsCreateFeedItem>
    <targetObject>Employment__c</targetObject>
    <type>Flow</type>
</QuickAction>
```

> ⚠️ **DEPLOYMENT LIMITATION (v66):** `type=Flow` Quick Actions **cannot be deployed via Metadata API**.
> - `<flow>FlowApiName</flow>` — invalid element (XML schema error)
> - `<flowDefinition>FlowApiName</flowDefinition>` — invalid element (runtime error: "לא ניתן להגדיר את השדה לסוג זרימה")
> - **Must be created manually via Setup UI:**
>   1. Setup → Object Manager → [Object] → Buttons, Links, and Actions → New Action
>   2. Action Type: **Flow**
>   3. Flow: בחר מהרשימה
>   4. Label + Save
>   5. Add to Page Layout via Object Manager → Page Layouts
> - **Alternative via metadata:** Use LWC component with `<lightning-flow flow-api-name="FlowApiName">` embedded in the record page — avoids the QuickAction limitation entirely.

### Quick Action Placement — Two Places Required

> ⚠️ Quick Action must appear in BOTH Page Layout AND FlexiPage to show correctly.

**In Page Layout XML:**
```xml
<quickActionList>
    <quickActionListItems>
        <quickActionName>Employment__c.Assign_Skills</quickActionName>
    </quickActionListItems>
</quickActionList>
```

**In FlexiPage XML (force:highlightsPanel controls action buttons):**
The `force:highlightsPanel` component automatically shows actions from the Page Layout.
No additional XML needed in FlexiPage — actions come from the Layout.

---

## 8. Profiles — Full UI Permissions Summary

Every new UI element requires a corresponding update in the Profile. Here is the complete map:

```xml
<Profile xmlns="http://soap.sforce.com/2006/04/metadata">

    <!-- App visibility -->
    <applicationVisibilities>
        <application>HR_App</application>
        <default>true</default>
        <visible>true</visible>
    </applicationVisibilities>

    <!-- Tab visibility -->
    <tabVisibilities>
        <tab>Employment__c</tab>
        <visibility>DefaultOn</visibility>
    </tabVisibilities>
    <tabVisibilities>
        <tab>standard-InternalOrganizationUnit</tab>
        <visibility>DefaultOn</visibility>
    </tabVisibilities>

    <!-- Object permissions -->
    <objectPermissions>
        <allowCreate>true</allowCreate>
        <allowDelete>true</allowDelete>
        <allowEdit>true</allowEdit>
        <allowRead>true</allowRead>
        <modifyAllRecords>true</modifyAllRecords>
        <object>Employment__c</object>
        <viewAllRecords>true</viewAllRecords>
    </objectPermissions>

    <!-- Field permissions (optional fields only — never required/MasterDetail) -->
    <fieldPermissions>
        <editable>true</editable>
        <field>Employment__c.Hourly_Rate__c</field>
        <readable>true</readable>
    </fieldPermissions>

    <!-- Layout assignment -->
    <layoutAssignments>
        <layout>Employment__c-Employment Layout</layout>
        <object>Employment__c</object>
    </layoutAssignments>

    <!-- Record Type visibility (if Record Types exist) -->
    <recordTypeVisibilities>
        <default>true</default>
        <recordType>Employment__c.Standard</recordType>
        <visible>true</visible>
    </recordTypeVisibilities>

</Profile>
```

### Profile Permissions — What Blocks What

| Missing permission | Symptom |
|---|---|
| No `objectPermissions` | Object not accessible — "Insufficient Privileges" |
| No `tabVisibilities` | Tab not visible in navigation |
| No `applicationVisibilities` | App not in App Launcher |
| No `layoutAssignments` | Default layout used instead of custom |
| No `fieldPermissions` on optional field | Field visible but not editable |
| No `recordTypeVisibilities` | Record Type not selectable on create |

---

## 9. Deployment — UI Components Order

```
1. Custom Labels (if new — used by LWC/Flow)
2. Object changes (enableSearch, enableReports, enableActivities)
3. Custom Tabs (before App or Profile references them)
4. Quick Actions (before Layout or FlexiPage references them)
5. Page Layouts (before Profile layout assignments)
6. Compact Layouts (before FlexiPage highlights panel)
7. FlexiPages (after all components they reference)
8. Lightning Apps (after Tabs exist)
9. Profiles (ALWAYS LAST — references everything above)
10. Translations / objectTranslations (can deploy after Profiles)
11. FlexiPage Assignment (manual via App Builder OR retrieve after UI assignment)
```

> ⚠️ **Translations deploy AFTER Profiles** — they depend on fields, picklists, and layouts existing.

---

## 10. Common Mistakes Checklist

| Mistake | What happens | Fix |
|---|---|---|
| Tab created but not in Profile | Tab invisible to users | Add `tabVisibilities` to Profile |
| App created but not in Profile | App invisible in App Launcher | Add `applicationVisibilities` to Profile |
| Layout deployed but not assigned in Profile | Default layout shown | Add `layoutAssignments` to Profile |
| FlexiPage deployed but not assigned | Default Record Page shown | Assign via App Builder + retrieve |
| Quick Action only in Layout, not FlexiPage | Action missing from Lightning page | Add to both |
| Quick Action only in FlexiPage | Works but inconsistent | Add to both |
| List View with `<me/>` sharing | Only creator sees it | Change to `<allInternalUsers/>` |
| Hebrew in `<label>` of field | English users also see Hebrew | Use English in `<label>`, Hebrew in `objectTranslation/<fields><label>` |
| Hebrew in `<label>` of tab | English users also see Hebrew | Use English in `<label>`, Hebrew in `iw.translation-meta.xml/<customTabs><label>` |
| Hebrew in `<label>` of app | English users also see Hebrew | Use English in `<label>`, Hebrew in `iw.translation-meta.xml/<customApplications><label>` |
| LWC without `dir="rtl"` | Hebrew text displays LTR | Add `dir="rtl"` to root element |
| `enableSearch=false` on object | Object missing from Global Search | Set `enableSearch>true` + deploy |
| RecordType exists but not in `recordTypeVisibilities` | Profile can't create that RecordType | Add `recordTypeVisibilities` to Profile |

---

## 11. Cross-Domain Interactions (Critical — Often Missed)

### FLS and Page Layout — Profile Always Wins
The interaction between Profile FLS and Page Layout field behavior:

| Profile FLS | Layout Setting | What User Sees |
|---|---|---|
| readable=false, editable=false | Any | Field **hidden entirely** — user cannot see it |
| readable=true, editable=false | Editable | Field displayed as **read-only** (Profile overrides Layout) |
| readable=true, editable=true | Required | Field displayed as required (Layout takes effect) |
| readable=true, editable=true | Read-Only | Field displayed as read-only (Layout takes effect) |
| readable=true, editable=true | Editable | Field displayed as editable |

**Rule: Page Layout controls display style; Profile FLS controls visibility. If FLS says hidden → Layout is irrelevant.**

### Cascade: Deleting a Quick Action
- If a Quick Action is deleted:
  - It is automatically removed from Page Layouts → safe
  - It is **NOT** automatically removed from FlexiPages → FlexiPage references broken action → runtime error for users
- Before deleting a Quick Action:
  1. Find all FlexiPages referencing it (retrieve all FlexiPages, grep for action API name)
  2. Remove from each FlexiPage
  3. Deploy updated FlexiPages
  4. Then deploy destructive change for the Quick Action

### List Views and Sharing (OWD)
- List Views with sharing `<me/>` → only visible to the user who created them
- List Views with `<allInternalUsers/>` → visible to all internal users
- If OWD = Private: List View "All Employees" → shows only records the running user has access to (Sharing Rules applied)
- List View does NOT bypass Sharing — it just defines filter/columns, not record access

### FlexiPage Not Assigned — Silent Failure
- A FlexiPage deployed but not assigned to an app/record type via App Builder → Salesforce uses the **default Record Page**
- Users see the wrong page with no error
- After deploying a FlexiPage: always retrieve it from App Builder (after assignment) to capture the assignment metadata

### Translations — Must Deploy After All Referenced Metadata
Translation deploy order:
```
1. Objects + Fields (label in English in metadata)
2. Picklist values (API names in English)
3. Tabs, Quick Actions, Apps
4. Profiles
5. Translations (objectTranslation files — Hebrew labels)
```
- Deploying translations before the field exists → deploy error
- Deploying translations with wrong `<name>` element order (must be: caseValues → fields → gender → nameFieldLabel) → deploy error

### RTL — What Lightning Auto-Flips vs What It Does Not
| Component | RTL Auto-Flip | Action Required |
|---|---|---|
| Standard Lightning fields on record page | ✅ Yes (when org/user lang = Hebrew) | None |
| Standard `lightning-input` component in LWC | ✅ Yes | None |
| Custom HTML in LWC (divs, spans) | ❌ No | Add `dir="rtl"` to container |
| CSS `text-align: left` in LWC | ❌ No | Override to `text-align: right` |
| `lightning-datatable` | ✅ Yes | None |
| Custom CSS positioning | ❌ No | Use logical CSS properties or explicit RTL overrides |
