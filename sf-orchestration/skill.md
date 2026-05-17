# SKILL_ORCHESTRATION_SF — Cross-Domain Enforcement Layer

> **Read this file FIRST before starting any Salesforce implementation task.**
> It does not replace the domain-specific SKILL files — it orchestrates them.
> For each operation below, follow the checklist here, then read the referenced SKILL files for implementation details.

---

## Mental Model — The Two Orthogonal Axes

Every Salesforce access decision passes through **two independent axes**:

### Axis 1: Record Access (Who can see which records?)
`Object Permission → OWD → Role Hierarchy → Sharing Rules → Manual Sharing → Apex Sharing → View All / Modify All`

### Axis 2: Field Access (Who can see which fields on those records?)
`Profile FLS (readable / editable) → Permission Set FLS (additive only)`

**Both axes must be satisfied for a user to read a field value.**
A Sharing Rule that grants record access does NOT grant field access.
FLS that grants field access does NOT grant record access.
These are independent. Always check both.

---

## Dependency Map — What Each Operation Must Trigger

### ✅ CREATE CUSTOM FIELD

| Step | Domain | SKILL File |
|------|--------|------------|
| Define field (type, name, label, required) | Objects | SKILL_OBJECTS_SF.md |
| Add to Profile FLS (readable + editable) | Profiles | SKILL_ROLES_PROFILES_PS_SF.md |
| Add to Permission Set FLS (if feature-specific) | Profiles | SKILL_ROLES_PROFILES_PS_SF.md |
| Add to Page Layout (all relevant Record Types) | UI | SKILL_UI_SF.md |
| Add to Compact Layout (if searchable/summary) | UI | SKILL_UI_SF.md |
| If encrypted → Encryption Policy in Setup (manual) | Encryption | SKILL_SHIELD_ENCRYPTION_SF.md |
| If encrypted → Run "Encrypt Existing Data" if data exists | Encryption | SKILL_SHIELD_ENCRYPTION_SF.md |
| If used in Flow filter → verify operator (encrypted: only = / !=) | Flow | SKILL_FLOW_PATTERNS_SF.md |
| If used in Validation Rule → verify not encrypted VLOOKUP first param | PSS | SKILL PSS SF.md |
| If ExternalId → verify Bulk Load upsert support and encoding | PSS | SKILL PSS SF.md |
| If on Person Account (Contact field) → all references must use `__pc` | PSS | SKILL PSS SF.md |
| Add Hebrew translation | UI | SKILL_UI_SF.md |

**Definition of Done:**
- [ ] Field deployed
- [ ] Profile FLS deployed (at least one profile)
- [ ] Page Layout updated
- [ ] Translation deployed
- [ ] If encrypted: Encryption Policy activated manually in Setup

---

### ✅ CREATE CUSTOM OBJECT

| Step | Domain | SKILL File |
|------|--------|------------|
| Check if PSS already has equivalent object | PSS | SKILL PSS SF.md |
| Define object (API name, label, sharing model) | Objects | SKILL_OBJECTS_SF.md |
| Set OWD deliberately (never leave at default without decision) | Sharing | SKILL_SHARING_SF.md |
| Define Profile object permissions (CRUD + View All / Modify All) | Profiles | SKILL_ROLES_PROFILES_PS_SF.md |
| Define Permission Set (if feature-specific access) | Profiles | SKILL_ROLES_PROFILES_PS_SF.md |
| Create default Page Layout with all required fields | UI | SKILL_UI_SF.md |
| Create Compact Layout | UI | SKILL_UI_SF.md |
| Create Tab (if users need direct navigation) | UI | SKILL_UI_SF.md |
| Add Tab to relevant Lightning Apps | UI | SKILL_UI_SF.md |
| Define Record Types (if needed) | Objects | SKILL_OBJECTS_SF.md |
| Add Hebrew translation (object + fields) | UI | SKILL_UI_SF.md |
| If OWD = Private → define Sharing Rules | Sharing | SKILL_SHARING_SF.md |

**Definition of Done:**
- [ ] Object + fields deployed
- [ ] Profile permissions deployed
- [ ] Layout + Tab + App deployed
- [ ] OWD explicitly set
- [ ] Translations deployed
- [ ] If OWD Private: Sharing Rules ready before first data load

---

### ✅ CREATE FLOW (Record-Triggered)

| Step | Domain | SKILL File |
|------|--------|------------|
| Entry criteria handles BOTH new + update (ISNEW + ISCHANGED) | Flow | SKILL_FLOW_PATTERNS_SF.md |
| Every Get Records has null/empty check Decision | Flow | SKILL_FLOW_PATTERNS_SF.md |
| Every DML element has Fault Path | Flow | SKILL_FLOW_PATTERNS_SF.md |
| Fault Path: only Create Record or Send Email (no external calls) | Flow | SKILL_FLOW_PATTERNS_SF.md |
| No SOQL / DML / HTTP calls inside loops | Flow | SKILL_FLOW_PATTERNS_SF.md |
| If reads encrypted field → only = and != operators in filter | Encryption | SKILL_SHIELD_ENCRYPTION_SF.md |
| If writes to field with Validation Rule → Fault Path catches VR error | Flow | SKILL_FLOW_PATTERNS_SF.md |
| If references Person Account fields → use `__pc` suffix | PSS | SKILL PSS SF.md |
| If Get Records on object with OWD=Private → Flow runs in system context, returns all records | Sharing | SKILL_SHARING_SF.md |
| If called from Screen Flow or Subflow → verify error propagation | Flow | SKILL_FLOW_PATTERNS_SF.md |
| If Scheduled Flow runs monthly logic → use Daily + first-day-of-month Decision (Frequency=Monthly rejected by API) | Flow | SKILL_PSS_SF.md |

**Definition of Done:**
- [ ] Entry criteria covers new + update
- [ ] All DML elements have Fault Path
- [ ] All Get Records have null check
- [ ] Bulk safety verified (no operations inside loops)
- [ ] Flow activated

---

### ✅ CREATE LWC COMPONENT

| Step | Domain | SKILL File |
|------|--------|------------|
| 4 files with identical base name (camelCase) | LWC | SKILL_LWC_SF.md |
| js-meta.xml targets defined | LWC | SKILL_LWC_SF.md |
| Apex controllers use `with sharing` | LWC | SKILL_LWC_SF.md |
| Apex read methods use `WITH SECURITY_ENFORCED` or `stripInaccessible()` | LWC | SKILL_LWC_SF.md |
| Add component to FlexiPage (App Builder) | UI | SKILL_UI_SF.md |
| Assign FlexiPage to relevant Profiles | UI | SKILL_UI_SF.md |
| If displays encrypted field → verify FLS `readable=true` on Profile | Encryption | SKILL_SHIELD_ENCRYPTION_SF.md |
| If Hebrew content → add `dir="rtl"` explicitly (Lightning does NOT auto-flip LWC) | UI | SKILL_UI_SF.md |
| If DML in Apex → `cacheable=false`, never `@AuraEnabled(cacheable=true)` on DML | LWC | SKILL_LWC_SF.md |

**Definition of Done:**
- [ ] Component files deployed
- [ ] FlexiPage updated and activated
- [ ] Profile has access to FlexiPage (not just component)
- [ ] Apex FLS enforcement verified

---

### ✅ CREATE VALIDATION RULE

| Step | Domain | SKILL File |
|------|--------|------------|
| Error message format: `[VR-XXX] Hebrew message` (3-digit zero-padded) | PSS | SKILL PSS SF.md |
| ISNEW/ISCHANGED pattern if condition-dependent | PSS | SKILL PSS SF.md |
| VLOOKUP first parameter must be Record Name field ($Name) — never encrypted or numeric | PSS | SKILL PSS SF.md |
| Test with Bulk Data Load (VRs fire during load — can block entire batch) | PSS | SKILL PSS SF.md |
| If Flow writes to field with this VR → Fault Path in Flow will catch the error | Flow | SKILL_FLOW_PATTERNS_SF.md |

**Definition of Done:**
- [ ] VR deployed and activated
- [ ] Error message in Hebrew with correct numbering
- [ ] Tested on new record AND on update
- [ ] Bulk Load tested if data migration planned

---

### ✅ CREATE REPORT / DASHBOARD

#### Step 1 — Before Writing Any Metadata

| Check | Why |
|---|---|
| Is `enableReports=true` on the source object? | If not → object not available in Report Type builder at all |
| Does a Custom Report Type (CRT) already exist for this join? | Reuse before creating a new one |
| Is the source object's OWD = Private? | Report will only show records the running user can access — plan Sharing Rules accordingly |

#### Step 2 — Report Folder

| Step | Rule | SKILL File |
|------|------|------------|
| Folder metadata file is **SIBLING** of folder directory (NOT inside it) | Salesforce computes fullName incorrectly if placed inside | SKILL_REPORTS_DASHBOARDS_SF.md |
| `accessType=Public` requires at least one `<sharedTo>` entry | Without it: folder created but invisible to everyone | SKILL_REPORTS_DASHBOARDS_SF.md |
| `publicFolderAccess`: `Read` = view only, `ReadWrite` = edit/create | Default: ReadWrite — tighten if needed | SKILL_REPORTS_DASHBOARDS_SF.md |
| Deploy folder BEFORE any reports inside it | Reports reference their folder in `fullName` | SKILL_REPORTS_DASHBOARDS_SF.md |

#### Step 3 — Custom Report Type (if needed)

| Step | Rule | SKILL File |
|------|------|------------|
| `reportType` element **MUST include `__c` suffix** | `EMP_RT__c` not `EMP_RT` — deploy error otherwise | SKILL_REPORTS_DASHBOARDS_SF.md |
| Deploy CRT BEFORE any reports that use it | Reports reference CRT by fullName | SKILL_REPORTS_DASHBOARDS_SF.md |

#### Step 4 — Report

| Step | Rule | SKILL File |
|------|------|------------|
| All CRT field references use `ObjectName$FieldName` format | `Employment__c$Status__c` not `Status__c` — applies to columns, filters, sorts, groupings | SKILL_REPORTS_DASHBOARDS_SF.md |
| Non-date groupings need `<dateGranularity>Day</dateGranularity>` | Picklist / Text groupings silently fail without it | SKILL_REPORTS_DASHBOARDS_SF.md |
| Filter values use API values (English), not Hebrew labels | `Active` not `פעיל` — filter matches nothing otherwise | SKILL_REPORTS_DASHBOARDS_SF.md |
| If Tabular + used in Dashboard → must have `<rowLimit>` | Without it: dashboard component cannot use this report | SKILL_REPORTS_DASHBOARDS_SF.md |
| Prefer Summary format for dashboard components | Summary with grouping always works; Tabular requires extra setup | SKILL_REPORTS_DASHBOARDS_SF.md |
| If encrypted field in filter → only `=` operator | CONTAINS / LIKE → runtime error or 0 results | SKILL_SHIELD_ENCRYPTION_SF.md |
| If user lacks FLS Read on a field → column shows blank (no error) | Silent — always verify FLS if column is unexpectedly empty | SKILL_ROLES_PROFILES_PS_SF.md |

#### Step 5 — Dashboard Folder

| Step | Rule |
|---|---|
| Same SIBLING rule as Report Folders | Folder metadata file must be outside the folder directory |
| Deploy Dashboard Folder BEFORE dashboards inside it | Same reason as Report Folders |

#### Step 6 — Dashboard

| Step | Rule | SKILL File |
|------|------|------------|
| All source reports must be deployed before the dashboard | Dashboard references reports — deploy order matters | SKILL_REPORTS_DASHBOARDS_SF.md |
| `dashboardType=SpecifiedUser` requires `<runningUser>` email | Without it: deploy error | SKILL_REPORTS_DASHBOARDS_SF.md |
| `runningUser` must have Read access to all objects in all source reports | Otherwise dashboard shows blank data | SKILL_REPORTS_DASHBOARDS_SF.md |
| `dashboardType=LoggedInUser` → each viewer sees only their own accessible records | Correct for personal dashboards, not team views | SKILL_REPORTS_DASHBOARDS_SF.md |

#### Step 7 — Profile Permissions

| Step | Rule |
|---|---|
| Add `standard-report` tab visibility to Profile | Without it: users cannot see the Reports tab at all |
| Add `standard-dashboard` tab visibility to Profile | Without it: users cannot see the Dashboards tab at all |
| User must have object `allowRead=true` for objects in the report | FLS + object permission both required |

#### Full Deploy Order for Reports/Dashboards

```
1. Object metadata with enableReports=true
2. Custom Report Types (CRT)
3. Report Folders         ← SIBLING placement
4. Dashboard Folders      ← SIBLING placement
5. Reports                ← inside their folder directories
6. Dashboards             ← after their source reports
7. Profiles               ← tab visibility (standard-report, standard-dashboard)
```

> ⚠️ **Silent deploy failures:** Salesforce can report "Succeeded" while silently skipping a component. Always verify after deploy:
> `sf data query --query "SELECT Id, Name FROM Dashboard WHERE FolderName = 'FolderName'"`

**Definition of Done:**
- [ ] `enableReports=true` on source object
- [ ] CRT deployed (if custom)
- [ ] Report Folder deployed with correct `sharedTo`
- [ ] Reports deployed inside folder
- [ ] Dashboard Folder deployed (if dashboard included)
- [ ] Dashboard deployed after source reports
- [ ] Profile `standard-report` + `standard-dashboard` tab visibility deployed
- [ ] Verified post-deploy: folder visible, report returns data, dashboard displays correctly

---

### ✅ CHANGE OWD (to more restrictive)

> **This is the highest-risk operation. It takes effect immediately and is not reversible by rollback.**

| Step | Domain | SKILL File |
|------|--------|------------|
| Create Sharing Rules BEFORE changing OWD | Sharing | SKILL_SHARING_SF.md |
| Verify all existing Sharing Rules still valid with new OWD level | Sharing | SKILL_SHARING_SF.md |
| Notify stakeholders — users lose access to records immediately | Sharing | SKILL_SHARING_SF.md |
| Verify Reports still return expected data with new OWD | Reports | SKILL_REPORTS_DASHBOARDS_SF.md |
| Verify Flows using Get Records still return expected records | Flow | SKILL_FLOW_PATTERNS_SF.md |
| Flows run in system context by default — Get Records returns ALL records regardless of OWD | Flow | SKILL_FLOW_PATTERNS_SF.md |

**Definition of Done:**
- [ ] Sharing Rules deployed and verified FIRST
- [ ] OWD changed
- [ ] Post-change verification: check Reports, check user record access

---

### ✅ DEPLOY TO ORG

**Canonical Deploy Order:**

```
1.  Objects (standard + custom, no Lookup fields yet)
2.  Fields (non-Lookup fields first)
3.  Lookup / MasterDetail fields (after referenced objects exist)
4.  Record Types
5.  Validation Rules
6.  Flows (after all fields they reference exist)
7.  Custom Labels
8.  Tabs
9.  Quick Actions
10. Page Layouts
11. Compact Layouts
12. FlexiPages
13. Lightning Apps
14. Profiles
15. Permission Sets
16. Permission Set Groups
17. Translations
18. Report Types
19. Report Folders + Dashboard Folders
20. Reports
21. Dashboards
```

**Rules:**
- Never deploy UI (FlexiPage, App) without also deploying Profiles that control access in the same package
- Destructive changes: use `destructiveChangesPost.xml`, remove Lookup dependencies before deleting objects
- If Person Accounts involved: must be enabled in org BEFORE deploying any flow referencing `__pc` fields
- PSL assignment must happen BEFORE assigning the Permission Set that depends on it

> ⚠️ **Tab Visibility Rule:** Profile-only deploy/retrieve can silently omit `tabVisibilities`. When fixing or setting tab visibility, always deploy `CustomTab` metadata in the **same package** as Profiles and the App. Without `CustomTab` in the package, tab visibility changes may not reach the org even though the Profile XML is correct locally.

> ⚠️ **FlexiPage Activation:** Org-default activation is stored in the **object's `object-meta.xml`** as `actionOverrides` (not in the FlexiPage file). Always deploy FlexiPage + CustomObject (with actionOverrides) together. An object without activation returns `type=Default` on `View` action; with activation it returns `type=Flexipage` and `content=<PageName>`.

---

## Cross-Domain Interaction Rules

These rules fall between files. They are not in any single SKILL file.

### Flow + Validation Rule
- If a Flow DML fails because a Validation Rule blocked the save → the Flow **Fault Path catches the error**
- The Fault Path `{!$Flow.FaultMessage}` will contain the VR error text
- Multiple VRs on the same object: Salesforce evaluates all active VRs. If any fail, the save is blocked. Order is not guaranteed.

### Flow + FLS
- Record-Triggered Flows run in **system context** (not user context) by default
- FLS does NOT block Flow from reading or writing fields
- Exception: if Flow calls Apex that uses `WITH SECURITY_ENFORCED` → FLS is enforced on that Apex query

### Flow + Sharing (OWD)
- Record-Triggered Flows run in system context → Get Records returns ALL records, ignoring OWD/Sharing Rules
- Screen Flows run in user context → Get Records respects OWD and Sharing Rules
- Autolaunched Flows called from Apex (without `System.runAs`) → system context

### Flow + Encrypted Field
- Get Records filter on encrypted field: only `=` and `!=` operators
- Cannot use CONTAINS, STARTS WITH, IN on encrypted fields in Flow filter
- Flow can write to encrypted field without restriction

### LWC + FLS
- FLS is NOT automatically enforced in Apex called from LWC
- Must explicitly use `WITH SECURITY_ENFORCED` or `Security.stripInaccessible()` in Apex
- If FLS not enforced: user without Read access can still see field value via LWC → **security hole**

### LWC + Sharing
- Apex with `with sharing` respects OWD + Sharing Rules → user only sees records they have access to
- Apex with `without sharing` bypasses Sharing Rules → use only when explicitly needed and documented

### Encrypted Field + Validation Rule
- Cannot use encrypted field as **first parameter** in `VLOOKUP` (must be Record Name / $Name field)
- Can use encrypted field in other formula positions (e.g., `IF(field__c = "value", ...)`)

### Encrypted Field + Reports
- Can filter by encrypted field using `=` only
- Cannot use CONTAINS, LIKE, starts with in Report filters on encrypted fields
- Encrypted field values display normally to users with FLS `readable=true`

### Encrypted Field + Bulk Load
- Can upsert using encrypted ExternalId field
- Matching uses exact case-insensitive comparison (Deterministic CI)
- Encryption Policy must be activated before load — fields not yet encrypted will not match

### Sharing Rule + FLS
- Sharing Rule grants **record access** (can see the record exists)
- FLS grants **field access** (can see field values on that record)
- If Sharing Rule grants Read on record but FLS denies Read on field → user sees the record but field shows blank
- Both must be satisfied independently

### OWD Private + Report
- Report shows only records the running user has access to (Sharing Rules applied)
- "Run as specified user" in Dashboard → dashboard data reflects that user's access level

### Profile FLS + Page Layout
- If Profile FLS = not readable → field is **hidden entirely** from Layout (user cannot see it at all)
- If Profile FLS = readable but not editable → field shows as **read-only** on Layout regardless of Layout setting
- If Profile FLS = readable + editable → Layout controls whether field is editable or read-only
- Layout field settings cannot override Profile FLS — Profile always wins

### Person Account + Custom Fields
- Custom fields on Contact object appear on Person Account as `FieldName__pc` (not `__c`)
- All Flow references, Apex queries, Validation Rules, and Reports must use `__pc` suffix
- Person Accounts must be enabled in the org BEFORE deploying any metadata that references `__pc` fields

### PSL + Permission Set Dependency
- `Public_Sector_Solutions_Admin` PSL must be assigned before `OmniSupervisor`, `OmniStudio User/Admin`
- Assignment order: PSL first → Permission Set Groups → Permission Sets
- If PSL missing: PSS objects invisible in Setup, describe calls return empty, SOQL returns 0 rows

---

## Anti-Patterns — Never Do These

| Anti-Pattern | Risk | Correct Approach |
|---|---|---|
| Deploy UI (FlexiPage) without Profile permissions in same package | Users see broken page or error | Always deploy Profiles with UI components |
| Change OWD to Private without creating Sharing Rules first | Immediate access loss for all users | Create Sharing Rules first, then change OWD |
| VLOOKUP with encrypted field as first param | Runtime error on every save | Use Record Name ($Name) as first VLOOKUP param |
| Flow filter with CONTAINS on encrypted field | Runtime error | Use only = or != on encrypted fields |
| Apex `without sharing` on public-facing LWC | Security bypass | Use `with sharing` unless explicitly documented |
| Deploy Person Account Flow before enabling Person Accounts | Deploy error | Enable Person Accounts in org first |
| Assign Permission Set before its required PSL | Silent failure or error | Assign PSL first, then PS |
| Deploy destructive changes without removing Lookup dependencies | Deploy error | Remove dependent fields/objects first |
| Set OWD to ControlledByParent on object that has Sharing Rules | Sharing Rules become invalid | Delete Sharing Rules before changing to ControlledByParent |
| Create Custom Object that duplicates PSS standard object | Technical debt, integration issues | Always check PSS objects first |
| Deploy Profile tab visibility without including CustomTab in package | Tab remains hidden in org even if Profile XML is correct | Include CustomTab in same deploy as Profile + App |
| Deploy FlexiPage without updating object actionOverrides | Default Record Page shown — FlexiPage has no effect | Add `type=Flexipage` actionOverrides to object-meta.xml for Large and Small |
| Deploy Account/PersonAccount record type as `Account.PersonAccount` | Creates a business-account record type, not Person Account | Use `PersonAccount.PersonAccount` metadata addressing |
| DML on queried Person Account that carries Name field in Apex | `INVALID_FIELD_FOR_INSERT_UPDATE` on Name | Construct fresh `new Account(Id=..., Field=...)` update object |

---

## Quick Reference — Which SKILL File for What

| Topic | File |
|---|---|
| Flow entry criteria, bulk safety, fault paths | SKILL_FLOW_PATTERNS_SF.md |
| LWC structure, Apex integration, RTL | SKILL_LWC_SF.md |
| Object/field types, relationships, Record Types, deploy order | SKILL_OBJECTS_SF.md |
| Reports, Dashboards, CRT, folders | SKILL_REPORTS_DASHBOARDS_SF.md |
| Profiles, Permission Sets, PSL, FLS | SKILL_ROLES_PROFILES_PS_SF.md |
| OWD, Sharing Rules, Role Hierarchy | SKILL_SHARING_SF.md |
| Deterministic CI encryption, SOQL limitations | SKILL_SHIELD_ENCRYPTION_SF.md |
| Page Layouts, FlexiPages, Tabs, Apps, Translations | SKILL_UI_SF.md |
| PSS objects, naming conventions, Validation Rules, Bulk Load | SKILL PSS SF.md |
| Cross-domain dependencies, Dependency Map, Definition of Done | **This file** |
