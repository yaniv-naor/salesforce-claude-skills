# Salesforce Flow — Production Patterns SKILL

## When to use this skill
Read this file at the start of ANY Flow task — before writing a single element.
This file defines how an experienced developer thinks about Flow, not just how to build one.

---

## 0. The Senior Developer Mindset — Before Building Any Flow

> ⚠️ **Claude Code tends to build the "happy path" only.** A production Flow must handle every path.

Before building, ask these questions for EVERY Flow:

```
1. What happens if the lookup finds NO records?
2. What happens if a field is BLANK/NULL?
3. What happens if this runs on record CREATE (data already set) vs UPDATE?
4. What happens if 200 records trigger this at once (bulk)?
5. What happens if an external call or DML fails?
6. What happens if the Flow runs twice on the same record?
7. Is there a condition where nothing should happen? (exit early)
```

**The pattern: always check before acting.**

```
❌ Junior: Get Record → Update Record
✅ Senior: Get Record → [Found?] → Yes: Check if update needed → Update
                                 → No: Exit gracefully (do nothing)
```

---

## 1. Record-Triggered Flow — Production Patterns

### Trigger Configuration — When to Use Each

| Trigger | Run When | Use For |
|---|---|---|
| `A record is created` | New record only | Set defaults, create related records |
| `A record is updated` | Existing record changed | Sync fields, notify on change |
| `A record is created or updated` | Both | Most automation — use with `$Record.Prior` checks |
| `A record is deleted` | Before delete | Archive, cleanup related records |

> ⚠️ **"Created or Updated" + no condition = runs on EVERY save.** Always add entry conditions.

### Entry Condition — Always Required

```
Entry Condition (Run Flow When):
  Status__c  Changes To  Active
  → Add condition: ONLY run when value actually changed
  → Use $Record.Status__c IS_CHANGED (available in "created or updated")
```

> ⚠️ Without entry condition — Flow runs on every field save even if the triggering field didn't change. This causes:
> - Infinite loops
> - Unnecessary DML
> - Performance issues on bulk operations

### Before-Save vs After-Save — Decision Rules

| | Before-Save | After-Save |
|---|---|---|
| Update SAME record fields | ✅ Yes — no DML needed | ❌ No — causes extra DML |
| Create/Update RELATED records | ❌ No | ✅ Yes |
| Send Email / Call External API | ❌ No | ✅ Yes |
| Check $Record.Id (new record) | ❌ Not available | ✅ Available |
| Performance | Faster | Slower |

**Rule: Always prefer Before-Save for same-record field updates.**

### The "Already Set on Create" Problem

> ⚠️ Common mistake: Flow triggers on CREATE to set a field, but the field was already populated by the user or an import. The Flow overwrites it.

**Pattern — Check before setting:**

```
Decision: Is_Field_Already_Set
  Condition: {!$Record.Employment_Type__c} IS NULL  OR  IS BLANK
  → Yes (null/blank): Set the default value
  → No (already has value): Exit — do nothing
```

```
// In metadata terms — Decision element:
Rule: Field_Is_Blank
  {!$Record.Employment_Type__c}  Is Null  true
  → Outcome: SET_DEFAULT

Rule: Field_Already_Set (default outcome)
  → Outcome: EXIT (no further elements)
```

### The "Don't Update if Not Changed" Pattern

> ⚠️ Common mistake: Flow always runs an Update Record even if nothing changed → unnecessary DML, audit trail noise, recursion risk.

```
Decision: Did_Value_Change
  {!$Record.Status__c}  Does Not Equal  {!$Record__Prior.Status__c}
  → Yes (changed): Proceed to update
  → No (same value): Exit
```

### Avoiding Infinite Loops

Record-Triggered Flows can trigger themselves if they update the same object.

**Prevention patterns:**

```
Option 1: Entry Condition uses IS_CHANGED
  → Flow only runs when a specific field changes
  → After Flow updates other fields, those don't retrigger (different fields)

Option 2: Add a "Flow ran" checkbox field
  → Flow_Processed__c (Checkbox, default false)
  → Entry condition: Flow_Processed__c = false
  → Flow sets Flow_Processed__c = true at end
  → ⚠️ Resets on next real user change — add to entry condition logic

Option 3: Before-Save for same-record updates
  → Before-Save Flows don't retrigger after-save flows
```

---

## 2. Get Records — Production Patterns

### Always Check "Records Found" After Get Records

> ⚠️ **Most common Claude Code mistake: Get Records → immediately use the result without checking if anything was found.**

```
❌ Wrong:
Get Records (Employment__c where Employee__c = {!recordId})
→ Assignment: var_Employment = {!Get_Employment}
→ Update Record: {!var_Employment.Status__c} = 'Active'
// If nothing found → NullPointerException or updates wrong record

✅ Correct:
Get Records (Employment__c where Employee__c = {!recordId})
→ Decision: Was_Employment_Found
    {!Get_Employment} IS NULL  →  false outcome: EXIT or LOG
    {!Get_Employment} IS NOT NULL  →  true outcome: CONTINUE
→ Update Record: {!Get_Employment}
```

### Get Records — Configuration Checklist

```
□ How many records: "First record only" vs "All records"
□ Filter condition: always specific — never get all records of an object
□ Sort order: define explicitly if order matters
□ Store in: "Automatically store all fields" for simple cases
           "Choose fields" for performance on large objects
□ After element: ALWAYS add Decision to check if null
```

### "First Record Only" vs "All Records"

```
First Record Only → returns SObject variable (or null)
  Check: {!Get_Employment} IS NULL

All Records → returns Collection variable (never null, but may be empty)
  Check: {!Get_Employments.size()} = 0
  OR use Decision: Count({!Get_Employments}) equals 0
```

### Lookup on Encrypted Field

```
Get Records
  Filter: National_ID__pc  Equals  {!inputNationalId}
  → Works with Deterministic CI encryption — exact match only
  → ⚠️ Contains / Starts With → runtime error on encrypted field
  → Always Decision after: was record found?
```

---

## 3. Update / Create Records — Production Patterns

### Update Record — Only Update What Changed

```
❌ Wrong: Update ALL fields on record
✅ Correct: Update only the specific fields the Flow is responsible for

// In Update Records element:
Set Field Values (not "Use all values from trigger or record variable"):
  Status__c = Active
  Last_Updated_By_Flow__c = {!$Flow.CurrentDateTime}
  // Only touch fields this Flow owns
```

### Create Record — Check for Duplicate First

> ⚠️ Flow triggered on Create should always check if a related record already exists before creating another.

```
Pattern:
1. Get Records: find existing related record
2. Decision: Already_Exists
   {!Get_Existing} IS NOT NULL → Yes: Exit (or update instead of create)
   {!Get_Existing} IS NULL → No: Create new record
```

### Update Record — Avoid Null Overwrites

```
❌ Wrong: Assign variable, leave field empty, update record → field gets nulled
✅ Correct: Only include fields in Update that you intentionally want to set

// If a field should only be set conditionally:
Decision: Should_Set_Field
  {!inputValue} IS NOT NULL
  → Yes: include field in Update
  → No: don't include field → existing value preserved
```

---

## 4. Fault Path — Every DML and External Call Must Have One

> ⚠️ **Claude Code often skips Fault Path. This is unacceptable in production.**
> Any element that can fail MUST have a Fault Path connected.

### Elements That MUST Have Fault Paths

```
✅ Update Records
✅ Create Records  
✅ Delete Records
✅ HTTP Callout / External Service
✅ Send Email (sometimes)
✅ Apex Action
```

### Fault Path Pattern — Log to Case

```
Fault Path → Create Record (Case)
  Subject: 'Flow Error: ' + {!$Flow.CurrentInterviewGuid}
  Description: {!$Flow.FaultMessage}
  Status: New
  Origin: Flow
  Priority: High
```

### Fault Path Pattern — Send Email to Admin

```
Fault Path → Send Email
  To: admin@moh.gov.il
  Subject: 'Flow Failure — ' + {!$Flow.InterviewLabel}
  Body: Error: {!$Flow.FaultMessage}
        Record: {!$Record.Id}
        Time: {!$Flow.CurrentDateTime}
```

### Fault Path Pattern — Custom Notification

```
Fault Path → Assignment (build error message)
  var_ErrorMsg = 'שגיאה בתהליך: ' + {!$Flow.FaultMessage}
→ Create Record (Error_Log__c)
  Flow_Name__c = {!$Flow.InterviewLabel}
  Error_Message__c = {!var_ErrorMsg}
  Record_Id__c = {!$Record.Id}
  Timestamp__c = {!$Flow.CurrentDateTime}
```

> ⚠️ **The Fault Path itself should NOT fail.** Keep it simple — only Create Record or Send Email. Never another external call.
> ⚠️ **After logging the error** — decide: should the transaction roll back (re-throw) or continue? In most HR cases: log + continue (don't break the user's save).

---

## 5. Bulk Safety — Flows on 200+ Records

> ⚠️ Record-Triggered Flows are bulkified automatically — but only if you don't fight it.

### What Salesforce Does Automatically

```
✅ Get Records inside Record-Triggered Flow → bulkified (one SOQL for all records)
✅ Update Records inside Record-Triggered Flow → bulkified (one DML for all records)
✅ Loop + Update Collection → processed in bulk
```

### What Breaks Bulk

```
❌ SOQL inside a Loop → 200 records = 200 SOQL queries → governor limit error
❌ DML inside a Loop → 200 records = 200 DML statements → governor limit error
❌ HTTP Callout inside a Loop → not allowed at all
```

### The Correct Bulk Pattern

```
❌ Wrong:
Loop over records
  → Get Records (inside loop) ← SOQL in loop
  → Update Record (inside loop) ← DML in loop

✅ Correct:
Get Records (outside loop — get ALL related records at once)
→ Loop over trigger records
  → Assignment: add to collection
→ Update Records (outside loop — single DML for collection)
```

### Scheduled Flow — Bulk Pattern

```
Scheduled Flow processes up to 2000 records per batch.
Always use:
  Get Records → All Records (with filter)
  Loop → process each
  Update Records Collection (outside loop)
  
⚠️ Never use "First Record Only" in Scheduled Flow if you need to process multiple.
```

---

## 6. Decision Element — Defensive Patterns

### The "Safe Exit" Pattern

Every Flow should have a clear exit for cases where nothing should happen:

```
Start
→ Entry Condition (on trigger)
→ Decision: Should_Flow_Run
    Condition 1: {!$Record.Status__c} equals Active
    Condition 2: {!$Record.Employee__c} IS NOT NULL
    → All conditions met: CONTINUE
    → Any condition fails: END (no action, no error)
```

### Null Check Before Formula or Assignment

```
❌ Wrong:
Assignment: var_Name = {!Get_Employee.FirstName} + ' ' + {!Get_Employee.LastName}
// If Get_Employee is null → error

✅ Correct:
Decision: Employee_Found
  {!Get_Employee} IS NULL → EXIT
  {!Get_Employee} IS NOT NULL → CONTINUE
Assignment: var_Name = {!Get_Employee.FirstName} + ' ' + {!Get_Employee.LastName}
```

### Picklist Value Check — Use API Value

```
❌ Wrong: {!$Record.Status__c} equals פעיל
✅ Correct: {!$Record.Status__c} equals Active
// Always compare to API value, not Hebrew translated label
```

---

## 7. Screen Flow — Production Patterns

### Input Validation Before Submit

```
Every Screen Flow with user input must validate:
1. Required fields — mark as Required in Screen component
2. Format validation — use Decision after screen to validate
3. Duplicate check — Get Records before creating

Pattern:
Screen (input)
→ Decision: Input_Valid
    {!input_NationalId} IS NOT NULL
    LEN({!input_NationalId}) = 9
    → Valid: continue
    → Invalid: Screen (show error message) → loop back
→ Get Records: check for duplicate
→ Decision: Duplicate_Found
    → Yes: Screen (show "record already exists")
    → No: Create Record
```

### Screen Flow — Don't Lose Data on Navigation Back

```
⚠️ If user clicks "Previous" in a multi-screen Flow, input variables reset.
Fix: Store inputs in Assignment elements as the user progresses.
Use persistent variables (defined outside screen components).
```

---

## 8. Autolaunched Flow — Production Patterns

### Called from Apex / Process Builder / Another Flow

```
Always define:
□ Input variables — clearly named, correct type
□ Output variables — for caller to check result
□ Error output variable — var_ErrorMessage (Text)

Pattern:
Input: recordId (Text, Required)
Input: actionType (Text, Required)
Output: isSuccess (Boolean)
Output: errorMessage (Text)

Flow logic:
→ Validate inputs (Decision: inputs not null)
→ Process
→ Fault Path: set isSuccess=false, errorMessage={!$Flow.FaultMessage}
→ End: set isSuccess=true
```

### Called from Apex

```apex
Map<String, Object> params = new Map<String, Object>{
    'recordId' => employmentId,
    'actionType' => 'ACTIVATE'
};
Flow.Interview interview = Flow.Interview.createInterview('FL_Employment_Action', params);
interview.start();

Boolean success = (Boolean) interview.getVariableValue('isSuccess');
String errorMsg = (String) interview.getVariableValue('errorMessage');

if (!success) {
    throw new AuraHandledException(errorMsg);
}
```

---

## 9. Flow Naming Conventions

```
Record-Triggered:  RT_[Object]_[Action]        → RT_Employment_SetDefaults
Scheduled:         SCH_[Object]_[Action]        → SCH_Employment_MonthlySync
Screen:            SCR_[Object]_[Action]        → SCR_Employee_OnboardingWizard
Autolaunched:      FL_[Object]_[Action]         → FL_Employment_ActivateRecord
Subflow:           SUB_[Domain]_[Action]        → SUB_HR_NotifyManager
```

---

## 10. Flow Checklist — Before Marking Any Flow "Done"

```
ENTRY CONDITIONS
□ Entry condition defined (not "always" unless intentional)
□ IS_CHANGED used for "created or updated" flows
□ Condition prevents infinite loop

GET RECORDS
□ Every Get Records followed by a null/empty check Decision
□ Filter is specific (not "get all")
□ Correct "first record" vs "all records" selection

DECISIONS
□ All Decision outcomes handled (not just the happy path)
□ Default outcome leads somewhere meaningful (exit or log)
□ Picklist comparisons use API values (English), not Hebrew labels

UPDATE / CREATE
□ Only updating fields this Flow is responsible for
□ Duplicate check before Create
□ Null overwrites prevented

FAULT PATHS
□ Every DML element has a Fault Path
□ Every external callout has a Fault Path
□ Fault Path logs to Case or Error_Log__c
□ Fault Path does NOT re-throw unless intentional rollback needed

BULK SAFETY
□ No SOQL inside Loop
□ No DML inside Loop
□ Collections used for bulk updates

"ALREADY SET" CHECK (for Create triggers)
□ Flow checks if field already has value before setting default
□ Flow checks if related record already exists before creating

GENERAL
□ Flow has a description explaining what it does and when
□ All variables have meaningful names (not var1, var2)
□ Inactive/debug elements removed before activation
□ Flow tested on: single record create, single record update, bulk (use Data Loader)
```

---

## 11. Common Mistakes — The Full List

| Mistake | What Happens | Fix |
|---|---|---|
| No null check after Get Records | NullPointerException or wrong record updated | Add Decision: IS NULL → exit |
| Get Records inside Loop | Governor limit: too many SOQL | Move Get Records outside loop |
| DML inside Loop | Governor limit: too many DML | Collect records, update once outside loop |
| No Fault Path on DML | Uncaught error breaks user's transaction | Add Fault Path → log to Case |
| No entry condition on "created or updated" | Runs every save, causes loops | Add IS_CHANGED condition |
| Overwriting field set on Create | Import data gets overwritten | Check IS NULL before setting |
| Creating duplicate related record | Duplicate data | Check for existing record first |
| Hebrew label in Decision comparison | Condition never matches | Use API value (Active not פעיל) |
| Null variable in Assignment/Formula | Flow error | Null check Decision before use |
| Screen Flow resets input on Back | User loses data | Store in persistent variables |
| Autolaunched Flow no error output | Caller can't handle failure | Add isSuccess + errorMessage output vars |
| Scheduled Flow "first record only" | Processes one record instead of batch | Use "all records" + Loop |
| Update Record updates ALL fields | Unintended field nulling | Choose specific fields only |
| No description on Flow | No one knows what it does | Always add description |

---

## 12. Cross-Domain Interactions (Critical — Often Missed)

### Flow + Validation Rules
- If a Flow DML (Create/Update/Delete Record) is blocked by a Validation Rule on the target object → the Flow **Fault Path catches this error**
- `{!$Flow.FaultMessage}` contains the VR error message text
- Implication: every DML element MUST have a Fault Path to handle VR failures gracefully
- Multiple active VRs on the same object: Salesforce evaluates ALL active VRs simultaneously. If any fail, the entire save is blocked. Evaluation order is not guaranteed — do not rely on ordering.
- VR error text flows up through Fault Path. Log it (e.g., Create Case with fault message) so the failure is traceable.

### Flow Context and FLS
- **Record-Triggered Flows** run in **system context** by default:
  - FLS does NOT prevent Flow from reading or writing any field
  - Flows can read/write encrypted fields, restricted fields, admin-only fields without restriction
  - Exception: if Flow calls an Apex action that uses `WITH SECURITY_ENFORCED` → FLS IS enforced on that Apex query
- **Screen Flows** run in **user context** by default:
  - Get Records respects OWD and Sharing Rules — returns only records the running user can access
  - If user lacks FLS Read on a field and Flow tries to display it → field returns null (no error)
- **Autolaunched Flows called from Apex** (without `System.runAs`) → system context

### Flow + Sharing Rules (OWD)
- Record-Triggered Flow Get Records: returns ALL records from the object regardless of OWD or Sharing Rules (system context)
- Screen Flow Get Records: returns only records the running user has access to (respects OWD + Sharing Rules)
- If you need a Record-Triggered Flow to behave as if it respects sharing → not natively supported; requires workaround via Apex

### Flow + Encrypted Fields
- **Get Records filter on encrypted field**: only `=` and `!=` operators work
- Cannot use: CONTAINS, STARTS WITH, IN, NOT IN, LIKE on encrypted fields in Flow filter → runtime error
- Flow can **write** to encrypted field without restriction
- Formula in Flow that references encrypted field: can use `=` comparison, cannot concatenate for partial match
- If filtering by encrypted field in Get Records → always use exact value match

### Flow + Person Account (`__pc` fields)
- Custom Contact fields on Person Account appear as `FieldName__pc` in Flow (not `__c`)
- Person Accounts must be enabled in the org BEFORE activating any Flow that references `__pc` fields
- In Flow "Get Records" on Account: filter by `ContactField__pc` (not `ContactField__c`)

### Cascading Flows (Flow A calls Flow B)
- If Flow B fails and has no Fault Path → the error propagates up to Flow A's Fault Path
- If Flow A also has no Fault Path → uncaught error, transaction rolls back entirely
- Best practice: each Subflow call should have an output variable `isSuccess` + `errorMessage`; check those in Flow A after the Subflow element
