# Salesforce LWC — Lightning Web Components SKILL

## When to use this skill
Read this file at the start of any task involving:
- Creating or editing Lightning Web Components
- LWC in Record Pages (FlexiPage)
- LWC calling Apex
- LWC with Hebrew / RTL
- LWC deployment and permissions

---

## 1. File Structure — Every LWC

```
force-app/main/default/lwc/
  myComponent/
    myComponent.html          ← template
    myComponent.js            ← controller
    myComponent.css           ← styles (optional)
    myComponent.js-meta.xml   ← metadata (targets, visibility)
```

> ⚠️ **Folder name = component name = must be camelCase.** All 4 files must have the exact same base name.
> ⚠️ **No Hebrew in component names, file names, or API names.**

---

## 2. Metadata File — js-meta.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>66.0</apiVersion>
    <isExposed>true</isExposed>    <!-- true = visible in App Builder -->
    <targets>
        <target>lightning__RecordPage</target>       <!-- Record Page -->
        <target>lightning__AppPage</target>           <!-- App Page -->
        <target>lightning__HomePage</target>          <!-- Home Page -->
        <target>lightning__FlowScreen</target>        <!-- Flow Screen -->
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__RecordPage">
            <property name="recordId" type="String" />   <!-- auto-populated by platform -->
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

| Target | Where it appears |
|---|---|
| `lightning__RecordPage` | Record Page in App Builder |
| `lightning__AppPage` | Custom App Page |
| `lightning__HomePage` | Home Page |
| `lightning__FlowScreen` | Inside a Screen Flow |
| `lightning__UtilityBar` | Utility bar at bottom |

> ⚠️ `<isExposed>false</isExposed>` = component is NOT visible in App Builder. Use for child components only.

---

## 3. HTML Template

```html
<!-- myComponent.html -->
<template>
    <!-- Conditional rendering -->
    <template if:true={isLoading}>
        <lightning-spinner alternative-text="Loading" size="small"></lightning-spinner>
    </template>

    <template if:false={isLoading}>
        <!-- Card wrapper -->
        <lightning-card title="פרטי עובד" icon-name="standard:employee">
            <div class="slds-p-around_medium" dir="rtl">   <!-- RTL for Hebrew -->

                <!-- Display field -->
                <lightning-output-field field-name="Name" record-id={recordId}>
                </lightning-output-field>

                <!-- Input -->
                <lightning-input
                    label="הערה"
                    value={noteValue}
                    onchange={handleNoteChange}>
                </lightning-input>

                <!-- Button -->
                <lightning-button
                    label="שמור"
                    variant="brand"
                    onclick={handleSave}>
                </lightning-button>

            </div>
        </lightning-card>
    </template>
</template>
```

### RTL — Hebrew Display

> ⚠️ **LWC does NOT automatically flip to RTL.** Must add `dir="rtl"` explicitly.

```html
<!-- Option 1: div wrapper -->
<div dir="rtl">...</div>

<!-- Option 2: root template (affects everything) -->
<template>
    <div dir="rtl" class="slds-p-around_medium">
        ...
    </div>
</template>
```

```css
/* myComponent.css — alternative */
:host {
    direction: rtl;
    text-align: right;
}
```

---

## 4. JavaScript Controller

```javascript
// myComponent.js
import { LightningElement, api, track, wire } from 'lwc';
import { getRecord, getFieldValue } from 'lightning/uiRecordApi';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import { NavigationMixin } from 'lightning/navigation';
import saveNote from '@salesforce/apex/EmploymentController.saveNote';
import EMPLOYMENT_STATUS from '@salesforce/schema/Employment__c.Status__c';
import EMPLOYMENT_NAME from '@salesforce/schema/Employment__c.Name';

export default class MyComponent extends NavigationMixin(LightningElement) {

    // Props passed from parent / App Builder / FlexiPage
    @api recordId;           // auto-populated on Record Pages
    @api customProp;         // configurable in App Builder (must be in js-meta.xml targetConfigs)

    // Reactive private properties
    @track noteValue = '';
    @track isLoading = false;
    error;

    // Wire — auto-fetch record data
    @wire(getRecord, { recordId: '$recordId', fields: [EMPLOYMENT_STATUS, EMPLOYMENT_NAME] })
    employment;

    // Getter — clean access to wired field
    get employmentStatus() {
        return getFieldValue(this.employment.data, EMPLOYMENT_STATUS);
    }

    get employmentName() {
        return getFieldValue(this.employment.data, EMPLOYMENT_NAME);
    }

    // Event handler
    handleNoteChange(event) {
        this.noteValue = event.target.value;
    }

    // Apex call
    handleSave() {
        this.isLoading = true;
        saveNote({ recordId: this.recordId, note: this.noteValue })
            .then(() => {
                this.showToast('הצלחה', 'הרשומה נשמרה בהצלחה', 'success');
                this.noteValue = '';
            })
            .catch(error => {
                this.showToast('שגיאה', error.body.message, 'error');
            })
            .finally(() => {
                this.isLoading = false;
            });
    }

    // Toast helper
    showToast(title, message, variant) {
        this.dispatchEvent(new ShowToastEvent({ title, message, variant }));
    }

    // Navigation
    navigateToRecord(recordId) {
        this[NavigationMixin.Navigate]({
            type: 'standard__recordPage',
            attributes: {
                recordId: recordId,
                actionName: 'view'
            }
        });
    }
}
```

### @api vs @track vs plain property

| Decorator | Meaning |
|---|---|
| `@api` | Public — settable from parent or App Builder |
| `@track` | Reactive — changes trigger re-render (objects/arrays need this) |
| plain | Reactive for primitives (string, number, boolean) automatically |

> ⚠️ In modern LWC (API 41+), primitive properties are reactive by default. `@track` is only needed for objects/arrays where you mutate nested properties.

---

## 5. Apex Controller for LWC

```apex
// force-app/main/default/classes/EmploymentController.cls
public with sharing class EmploymentController {

    @AuraEnabled(cacheable=true)      // cacheable=true required for @wire
    public static Employment__c getEmployment(Id recordId) {
        return [
            SELECT Id, Name, Status__c, Start_Date__c, Employee__c
            FROM Employment__c
            WHERE Id = :recordId
            WITH SECURITY_ENFORCED        // enforce FLS
            LIMIT 1
        ];
    }

    @AuraEnabled                       // no cacheable — for DML operations
    public static void saveNote(Id recordId, String note) {
        Employment__c emp = new Employment__c(
            Id = recordId,
            Notes__c = note
        );
        update emp;
    }

    @AuraEnabled(cacheable=true)
    public static List<Employment__c> getActiveEmployments() {
        return [
            SELECT Id, Name, Status__c, Employee__r.Name
            FROM Employment__c
            WHERE Status__c = 'Active'
            WITH SECURITY_ENFORCED
            ORDER BY Name
        ];
    }
}
```

> ⚠️ **`cacheable=true` is required for `@wire`.** Without it: error "cacheable must be true".
> ⚠️ **DML methods must NOT have `cacheable=true`.** Insert/Update/Delete → no cacheable.
> ⚠️ **`with sharing` is required** unless there's a specific reason. Never use `without sharing` unless necessary and documented.

---

## 6. Wire vs. Imperative Calls

| Pattern | When to use |
|---|---|
| `@wire` | Auto-fetch on load, reactive to property changes, read-only |
| Imperative (`.then/.catch`) | On user action (button click), DML, conditional fetching |

```javascript
// Wire — automatic
@wire(getRecord, { recordId: '$recordId', fields: [STATUS_FIELD] })
wiredRecord;

// Imperative — on demand
handleClick() {
    getActiveEmployments()
        .then(result => { this.records = result; })
        .catch(error => { this.error = error; });
}
```

---

## 7. Component Communication

### Parent → Child (property binding)

```html
<!-- parent.html -->
<c-child-component record-id={currentRecordId} status={employmentStatus}></c-child-component>
```

```javascript
// childComponent.js
@api recordId;
@api status;
```

> ⚠️ **camelCase in JS = kebab-case in HTML.** `recordId` → `record-id`, `myProp` → `my-prop`.

### Child → Parent (custom event)

```javascript
// child — fire event
this.dispatchEvent(new CustomEvent('statuschange', {
    detail: { newStatus: 'Active' }
}));
```

```html
<!-- parent — listen -->
<c-child-component onstatuschange={handleStatusChange}></c-child-component>
```

```javascript
// parent — handle
handleStatusChange(event) {
    const newStatus = event.detail.newStatus;
}
```

### Unrelated Components (Lightning Message Service)

```javascript
// Publisher
import { publish, MessageContext } from 'lightning/messageService';
import EMPLOYMENT_CHANNEL from '@salesforce/messageChannel/EmploymentChannel__c';

@wire(MessageContext) messageContext;

publishMessage() {
    publish(this.messageContext, EMPLOYMENT_CHANNEL, { recordId: this.recordId });
}

// Subscriber
import { subscribe, MessageContext } from 'lightning/messageService';
@wire(MessageContext) messageContext;
subscription = null;

connectedCallback() {
    this.subscription = subscribe(this.messageContext, EMPLOYMENT_CHANNEL, (message) => {
        this.handleMessage(message);
    });
}
```

---

## 8. Adding LWC to FlexiPage

```xml
<!-- In FlexiPage XML -->
<componentInstance>
    <componentInstanceProperties>
        <n>recordId</n>
        <value>{!recordId}</value>    <!-- platform variable -->
    </componentInstanceProperties>
    <componentName>c:myComponent</componentName>
    <identifier>c_myComponent</identifier>
</componentInstance>
```

> ⚠️ Custom LWC namespace is always `c:` (e.g. `c:employmentDetails`).
> ⚠️ `identifier` must be unique within the FlexiPage.

---

## 9. Deployment

```bash
# Deploy single component
sf project deploy start --metadata "LightningComponentBundle:myComponent" --target-org moh-sandbox

# Deploy component + Apex controller together
sf project deploy start \
  --metadata "LightningComponentBundle:myComponent" \
              "ApexClass:EmploymentController" \
  --target-org moh-sandbox
```

### Deploy Checklist — Every New LWC

- [ ] All 4 files present with identical base name
- [ ] `js-meta.xml` has correct `<targets>` for where it will be used
- [ ] `isExposed=true` if it needs to appear in App Builder
- [ ] Apex controller deployed alongside
- [ ] `@AuraEnabled` on all methods called from LWC
- [ ] `cacheable=true` on read-only methods used with `@wire`
- [ ] If added to FlexiPage — FlexiPage redeployed
- [ ] Profile has FLS on all fields the component reads/writes

---

## 10. Common Mistakes

| Mistake | What Happens | Fix |
|---|---|---|
| File name ≠ folder name | Deploy error | All 4 files must share exact base name |
| `@AuraEnabled` without `cacheable=true` on wired method | Runtime error | Add `cacheable=true` |
| `cacheable=true` on DML method | Runtime error | Remove `cacheable` from insert/update/delete methods |
| No `dir="rtl"` | Hebrew displays left-to-right | Add `dir="rtl"` to container div |
| `@api` property mutated directly | LWC warning / unexpected behavior | Never mutate `@api` props — fire event to parent |
| camelCase in HTML template | Property not passed | Use kebab-case: `recordId` → `record-id` |
| `isExposed=false` | Component missing in App Builder | Set `isExposed=true` |
| Missing target in js-meta.xml | Component not available in that context | Add correct `<target>` |
| Apex without `with sharing` | FLS/sharing bypassed | Always use `with sharing` unless documented |
| Component deployed but FlexiPage not redeployed | Old version shown | Redeploy FlexiPage after adding component |

---

## 11. Cross-Domain Interactions (Critical — Often Missed)

### FLS Enforcement in Apex — NOT Automatic
FLS is **not enforced automatically** in Apex called from LWC. You must enforce it explicitly.

**Option 1 — `WITH SECURITY_ENFORCED` in SOQL:**
```apex
@AuraEnabled(cacheable=true)
public static List<Employee__c> getEmployees() {
    return [SELECT Id, Name, National_ID__c FROM Employee__c WITH SECURITY_ENFORCED];
}
```
- If user lacks Read on `National_ID__c` → query throws `System.QueryException`
- Handle this in the catch block and return an appropriate error

**Option 2 — `Security.stripInaccessible()` (preferred for partial access):**
```apex
@AuraEnabled(cacheable=true)
public static List<Employee__c> getEmployees() {
    List<Employee__c> records = [SELECT Id, Name, National_ID__c FROM Employee__c];
    SObjectAccessDecision decision = Security.stripInaccessible(
        AccessType.READABLE, records
    );
    return decision.getRecords(); // returns records with inaccessible fields stripped (null)
}
```
- Fields the user cannot read are returned as null (no exception)
- Use this when partial data display is acceptable

**If neither is used:** user without FLS Read CAN see the field value via LWC → **security hole**.

### LWC + Sharing Rules (OWD)
- Apex with `with sharing` respects OWD + Sharing Rules: user only sees records they can access
- Apex with `without sharing` bypasses all Sharing Rules: user can query any record regardless of OWD
- Use `without sharing` only for specific system operations (integration, background processes) — always document why
- If component displays data that could be private → default to `with sharing`

### LWC + Encrypted Fields
- Encrypted field values are readable normally if user has FLS `readable=true` on the Profile
- No special permission needed (as of API v48+ / Summer '22)
- If LWC query filters on encrypted field → Apex SOQL must use only `=` or `!=` (not LIKE or IN with partial match)
- If component allows search by encrypted field (e.g., ID lookup) → only exact match supported

### LWC + FlexiPage Access
- Deploying a component is not enough: the FlexiPage must also be updated and re-deployed
- Profile must have access to the FlexiPage (not just the component itself)
- If FlexiPage is not assigned to a record type / profile → Salesforce uses the default Record Page
- After adding component to FlexiPage in App Builder → retrieve the FlexiPage metadata → redeploy

### LWC Error Handling Pattern
```javascript
// Always handle both wire errors and imperative call errors
import { LightningElement, wire } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

// For imperative calls:
handleAction() {
    myApexMethod({ param: this.value })
        .then(result => { /* handle success */ })
        .catch(error => {
            this.dispatchEvent(new ShowToastEvent({
                title: 'שגיאה',
                message: error.body?.message || 'אירעה שגיאה',
                variant: 'error'
            }));
        });
}
```
