# Salesforce Shield Platform Encryption — SKILL
## Case-Insensitive Deterministic Encryption

## When to use this skill
Read this file at the start of any task involving:
- Encrypting fields on Employee2 / Person Account
- SOQL queries on encrypted fields
- Flow reading/writing encrypted fields
- Bulk data load to encrypted fields
- Unique / ExternalId fields with encryption
- Permission setup for viewing encrypted data
- Troubleshooting "field value masked" or "cannot filter on encrypted field"

---

## 1. Encryption Types — Which to Use and Why

| Scheme | Searchable | Unique | Case-Sensitive | Use Case |
|---|---|---|---|---|
| Probabilistic | ❌ No | ❌ No | N/A | Archives only — never search |
| Deterministic | ✅ Yes | ✅ Yes | ✅ Yes | Exact match, case matters |
| **Deterministic Case-Insensitive** | ✅ Yes | ✅ Yes | ❌ No | **← This org** |

**למה Case-Insensitive Deterministic לפרויקט הזה:**
- תעודת זהות / מספר עובד מוזנים לפעמים עם/בלי אפסים מובילים, אותיות גדולות/קטנות
- SOQL WHERE עובד
- Unique constraint עובד
- ExternalId lookup מ-Bulk Load עובד ללא תלות ב-case של הקובץ

---

## 2. תנאים מוקדמים — לפני שמצפינים

```
1. רישיון Shield Platform Encryption קיים ב-org
2. Permission Set של Shield מוקצה למשתמש Admin
3. מפתח הצפנה נוצר: Setup → Platform Encryption → Key Management
4. סוג השדה תואם (ראה טבלה למטה)
5. השדה אינו Formula — שדות Formula לא ניתן להצפנה
6. השדה אינו Roll-Up Summary — לא תואם
```

### סוגי שדות תואמים להצפנה

| סוג שדה | ניתן להצפנה | הערה |
|---|---|---|
| Text | ✅ כן | הנפוץ ביותר |
| Phone | ✅ כן | |
| Email | ✅ כן | |
| URL | ✅ כן | |
| TextArea | ✅ כן | Probabilistic בלבד |
| LongTextArea | ❌ לא | |
| Formula | ❌ לא | תמיד — יגרום לשגיאה |
| Number / Currency / Date | ❌ לא | |
| Lookup / MasterDetail | ❌ לא | |
| AutoNumber | ❌ לא | |

---

## 3. הגדרת השדה — Field Metadata

> ⚠️ **ה-XML של השדה לא מכיל הגדרות הצפנה.** ההצפנה מופעלת דרך Encryption Policy ב-Setup אחרי ה-deploy.

```xml
<!-- Employee2 / Contact — תעודת זהות -->
<fields>
    <fullName>National_ID__c</fullName>
    <label>National ID</label>
    <type>Text</type>
    <length>20</length>
    <unique>true</unique>
    <externalId>true</externalId>
    <caseSensitive>false</caseSensitive>   <!-- חובה: false עבור CI Deterministic -->
    <required>false</required>
</fields>
```

> ⚠️ `<caseSensitive>false</caseSensitive>` — חובה על השדה. אם `true`, ה-unique constraint יהיה case-sensitive גם עם CI Deterministic.
> ⚠️ שדה Unique + ExternalId = **לא לכלול ב-fieldPermissions של Profile**. הוא אינו optional field.

---

## 4. הפעלת Encryption Policy — אחרי ה-Deploy

```
Setup → Platform Encryption → Encryption Policy → Encrypt Fields
→ בחר Object: Employee2 (או Contact עבור Person Account)
→ בחר Field: National_ID__c
→ בחר Scheme: Deterministic
→ סמן: Case Insensitive
→ שמור
```

> ⚠️ **לא ניתן לפרוס Encryption Policy דרך Metadata API.** זו פעולה ידנית ב-Setup בלבד.
> ⚠️ לאחר הפעלת ההצפנה — נתונים קיימים בשדה אינם מוצפנים אוטומטית. יש להריץ: Setup → Platform Encryption → Encrypt Existing Data.

---

## 5. הרשאות — מי רואה ערכים מוצפנים

> ✅ **החל מ-2022 — `ViewEncryptedData` כבר לא נדרש.** Salesforce הסיר את הדרישה הזו.
> גישה לשדה מוצפן נשלטת דרך **FLS רגיל בלבד** — בדיוק כמו כל שדה אחר.

### הכלל הפשוט

אם למשתמש יש `readable=true` על השדה בפרופיל / Permission Set — הוא רואה את הערך המוצפן. אם `readable=false` — השדה לא נראה כלל. אין צורך בהרשאה מיוחדת.

### דוגמה — FLS על שדה מוצפן בפרופיל

```xml
<!-- Profile XML — FLS על שדה מוצפן, רגיל לחלוטין -->
<fieldPermissions>
    <readable>true</readable>
    <editable>true</editable>
    <field>Account.National_ID__pc</field>
</fieldPermissions>
```

### מי צריך גישה

| פרופיל | readable | editable | סיבה |
|---|---|---|---|
| System Administrator | ✅ | ✅ | ניהול מלא |
| HR_Manager | ✅ | ✅ | רואה וכותב ת"ז |
| Finance | ❌ | ❌ | לא צריך ת"ז |
| Integration User | ✅ | ✅ | כתיבה/קריאה ב-API |
---

## 6. SOQL על שדה מוצפן

### מה עובד עם Deterministic CI

```apex
// ✅ מותר — exact match על שדה מוצפן CI Deterministic
List<PersonAccount> results = [
    SELECT Id, Name, National_ID__pc
    FROM Account
    WHERE National_ID__pc = '123456789'
    AND IsPersonAccount = true
];

// ✅ מותר — case insensitive: 'ABC123' = 'abc123'
List<PersonAccount> results = [
    SELECT Id, Name
    FROM Account
    WHERE National_ID__pc = 'abc123'
];
```

### מה לא עובד

```apex
// ❌ אסור — LIKE על שדה מוצפן
WHERE National_ID__pc LIKE '%123%'

// ❌ אסור — חיפוש חלקי
WHERE National_ID__pc LIKE '12345%'

// ❌ אסור — ORDER BY על שדה מוצפן
ORDER BY National_ID__pc ASC

// ❌ אסור — GROUP BY על שדה מוצפן
GROUP BY National_ID__pc

// ❌ אסור — אופרטורים >, <, >=, <=
WHERE National_ID__pc > '100000000'
```

> ⚠️ **Deterministic CI תומך רק ב-`=` ו-`!=` ב-WHERE.** כל שאר האופרטורים — שגיאה בזמן ריצה.

### Employee2 — שמות שדות מיוחדים

Employee2 הוא Person Account. שדות Custom על Contact מופיעים עם `__pc`:

```apex
// שדה National_ID__c על Contact → מופיע כ-National_ID__pc על Account/Employee2
SELECT Id, FirstName, LastName, National_ID__pc
FROM Account
WHERE IsPersonAccount = true
AND National_ID__pc = :nationalId
```

| אובייקט | API שדה ב-Contact | API שדה ב-Account/Employee2 |
|---|---|---|
| Custom Text field | `National_ID__c` | `National_ID__pc` |
| Standard PersonEmail | `Email` | `PersonEmail` |
| Standard PersonMobilePhone | `MobilePhone` | `PersonMobilePhone` |

---

## 7. Flow — קריאה וכתיבה לשדה מוצפן

### קריאה

Flow קורא שדות מוצפנים רגיל — **בתנאי שלמשתמש יש FLS `readable=true` על השדה.**

```
Get Records → Object: Employee2 (Account)
→ Filter: National_ID__pc Equals {!nationalIdVar}
→ זה עובד עם Deterministic CI — exact match בלבד
```

> ⚠️ בFlow — **אין תמיכה ב-LIKE או חיפוש חלקי** על שדה מוצפן. exact match בלבד.
> ⚠️ אם ל-Running User אין `readable=true` על השדה — ה-Filter לא יוכל לסנן לפיו.

### כתיבה

```
Update Records → Set Field Values
→ National_ID__pc = {!nationalIdVar}
→ Salesforce מצפין אוטומטית לפני השמירה
```

> ✅ כתיבה לשדה מוצפן עובדת רגיל — Salesforce מטפל בהצפנה שקוף.

### מה לא לעשות ב-Flow

```
❌ אל תשתמש ב-Contains / Starts With על שדה מוצפן — לא יעבוד
❌ אל תשתמש ב-Sort על שדה מוצפן ב-Get Records
❌ אל תציג שדה מוצפן ב-Screen Flow אם ל-Running User אין FLS readable=true על השדה
```

---

## 8. LWC — תצוגה של שדה מוצפן

### הצגה בסיסית

LWC מציג את הערך המוצפן רגיל — **בתנאי שלמשתמש יש FLS `readable=true` על השדה.**

```javascript
// wireGetRecord — עובד רגיל
import { LightningElement, api, wire } from 'lwc';
import { getRecord, getFieldValue } from 'lightning/uiRecordApi';
import NATIONAL_ID from '@salesforce/schema/Account.National_ID__pc';

export default class EmployeeDetails extends LightningElement {
    @api recordId;

    @wire(getRecord, { recordId: '$recordId', fields: [NATIONAL_ID] })
    account;

    get nationalId() {
        return getFieldValue(this.account.data, NATIONAL_ID);
        // אם למשתמש אין FLS readable=true → השדה לא יוחזר כלל
    }
}
```

### בדיקת מסיכה

```javascript
get isMasked() {
    const val = getFieldValue(this.account.data, NATIONAL_ID);
    return val && val.includes('*');
}
```

```html
<template>
    <template if:true={isMasked}>
        <span>🔒 שדה מוצפן — אין הרשאת צפייה</span>
    </template>
    <template if:false={isMasked}>
        <lightning-output-field field-name="National_ID__pc" record-id={recordId}>
        </lightning-output-field>
    </template>
</template>
```

### Apex ב-LWC על שדה מוצפן

```apex
@AuraEnabled(cacheable=true)
public static Account getEmployeeByNationalId(String nationalId) {
    // WITH SECURITY_ENFORCED מכבד FLS — שדות ללא readable=true ישמטו
    return [
        SELECT Id, Name, National_ID__pc
        FROM Account
        WHERE National_ID__pc = :nationalId
        AND IsPersonAccount = true
        WITH SECURITY_ENFORCED
        LIMIT 1
    ];
}
```

> ⚠️ `WITH SECURITY_ENFORCED` — מכבד FLS. אם למשתמש אין `readable=true` על השדה — השדה ישמט מהתוצאה.
> ⚠️ **`cacheable=true` + שדה מוצפן** — הערך נשמר ב-cache. שקול `cacheable=false` לשדות רגישים אם הרשאות FLS עשויות להשתנות.

---

## 9. Bulk Data Load — טעינת נתונים לשדה מוצפן

### עקרונות

- **Salesforce מצפין אוטומטית** כשמעלים ערכים דרך Data Loader / Bulk API
- אין צורך להצפין את הקובץ CSV לפני ההעלאה
- ה-Integration User שמבצע ה-load צריך FLS `readable=true` על שדות מוצפנים שנקראים

### Upsert על שדה ExternalId מוצפן

```
Operation: Upsert
External ID Field: National_ID__pc
File: employees.csv
```

```csv
National_ID__pc,FirstName,LastName,PersonEmail
123456789,ישראל,ישראלי,israel@example.com
987654321,שרה,כהן,sarah@example.com
```

> ✅ **CI Deterministic מאפשר Upsert על שדה מוצפן** — Salesforce מצפין את הערך מהקובץ ומחפש התאמה.
> ✅ `123456789` ו-`123456789` (עם/בלי רווחים, אותיות שונות) — יתאמו לאותה רשומה בגלל Case-Insensitive.

### הגדרת Integration User

```apex
// ב-Sandbox — וודא שה-Integration User מוגדר עם:
// 1. Profile עם objectPermissions על Account (Create, Read, Edit)
// 2. FLS readable+editable על שדות מוצפנים בפרופיל / Permission Set
// 3. Permission Set Licenses: Salesforce + Industries + Shield

User integrationUser = [
    SELECT Id, Name, Profile.Name
    FROM User
    WHERE Username = 'integration@moh-sandbox.com'
    LIMIT 1
];
```

### שגיאות נפוצות ב-Bulk Load

| שגיאה | סיבה | פתרון |
|---|---|---|
| `FIELD_INTEGRITY_EXCEPTION: encrypted field` | ניסיון לעשות LIKE/חיפוש חלקי | השתמש ב-exact match בלבד |
| `INVALID_FIELD: National_ID__pc` | שגיאת שם שדה — `__c` במקום `__pc` | וודא סיומת `__pc` ל-Person Account fields |
| `ערכים מוצפנים לא מוחזרים` | Integration User אין FLS readable על השדה | הוסף fieldPermissions לפרופיל |
| `Duplicate detected` | CI encryption תפס כפילות | צפוי — זה עובד נכון |
| `CANNOT_ENCRYPT_THIS_FIELD_TYPE` | ניסיון להצפין Number/Formula/Lookup | שנה סוג שדה לפני הצפנה |

---

## 10. Deploy Order — Shield Encryption

```
1. Deploy שדה (Text, caseSensitive=false, unique=true, externalId=true)
2. Deploy Profile / Permission Set (FLS — לא לכלול שדה unique כ-fieldPermission)
3. Setup → Platform Encryption → Key Management → Generate Key (אם לא קיים)
4. Setup → Platform Encryption → Encryption Policy → הצפן את השדה
5. Setup → Platform Encryption → Encrypt Existing Data (לנתונים קיימים)
6. בדוק: SOQL exact match, Bulk Upsert, Flow Get Records
```

> ⚠️ **שלבים 5-7 הם ידניים ב-Setup** — לא Metadata API.
> ⚠️ **הצפנת נתונים קיימים (שלב 7) לוקחת זמן** בהתאם לכמות הרשומות. ניתן לעקוב ב-Setup → Platform Encryption → Encryption Statistics.

---

## 11. בדיקת תקינות אחרי הצפנה

```apex
// הרץ ב-Developer Console → Execute Anonymous
// בדיקה 1: SOQL exact match עובד
List<Account> res = [
    SELECT Id, Name, National_ID__pc
    FROM Account
    WHERE National_ID__pc = '123456789'
    AND IsPersonAccount = true
    LIMIT 1
];
System.debug('Found: ' + res.size() + ' records');
System.debug('Value: ' + (res.isEmpty() ? 'not found' : res[0].National_ID__pc));

// בדיקה 2: Case insensitive עובד
List<Account> res2 = [
    SELECT Id FROM Account
    WHERE National_ID__pc = '123456789'  // אותו ערך, case שונה
    AND IsPersonAccount = true LIMIT 1
];
System.debug('CI match: ' + res2.size());

// בדיקה 3: ערך השדה מוחזר תקין (לא ריק)
if (!res.isEmpty()) {
    System.debug('Encrypted value visible: ' + !res[0].National_ID__pc.contains('*'));
}
```

---

## 12. Common Mistakes

| טעות | מה קורה | תיקון |
|---|---|---|
| `caseSensitive=true` על שדה מוצפן CI | Unique constraint case-sensitive | שנה ל-`caseSensitive=false` |
| LIKE על שדה מוצפן ב-SOQL | Runtime error | exact match בלבד (`=` / `!=`) |
| ORDER BY על שדה מוצפן | Runtime error | הסר ORDER BY |
| שדה מוצפן ב-fieldPermissions של Profile | Deploy error | הסר — שדה Unique/ExternalId אינו optional |
| Integration User ללא FLS readable על השדה | שדה מוצפן לא מוחזר ב-query | הוסף fieldPermissions לפרופיל |
| `cacheable=true` ב-Apex על שדה מוצפן | Cache מחזיר ערך ישן | שקול `cacheable=false` |
| הצפנה על Formula field | Deploy error | Formula fields לא ניתן להצפנה |
| שכחת Encrypt Existing Data | נתונים ישנים לא מוצפנים | הרץ מ-Setup → Platform Encryption |
| `__c` במקום `__pc` ב-Bulk Load | שגיאת INVALID_FIELD | Person Account custom fields = `__pc` |
| ניסיון Metadata API לשנות Encryption Policy | לא אפשרי | רק ידנית ב-Setup |

---

## 13. Cross-Domain Interactions (Critical — Often Missed)

### Encrypted Field + Flow
- **Get Records filter on encrypted field**: only `=` and `!=` operators are valid
  ```
  ✅ National_ID__pc Equals {!inputId}
  ❌ National_ID__pc Contains {!inputId}    → runtime error
  ❌ National_ID__pc Starts With {!inputId} → runtime error
  ```
- Flow can **write** to encrypted field without restriction
- Flow can **read** encrypted field value (system context) and assign to variable
- Cannot use encrypted field in Flow **Decision** comparisons other than equals/not equals

### Encrypted Field + Validation Rules
- **Cannot use an encrypted field as the first parameter of `VLOOKUP`**
  ```
  ❌ VLOOKUP($ObjectType.JobCatalog__c.Fields.National_ID__c, National_ID__c, Id)
     → first param must be Record Name field ($Name) — not encrypted or numeric
  ✅ VLOOKUP($ObjectType.JobCatalog__c.Fields.Name, Name, IsActive__c)
  ```
- Can use encrypted field in other formula positions: `IF(National_ID__c = "123", ...)` → valid
- Cannot use encrypted field in `INCLUDES()`, `ISCHANGED()` across partial values — exact only

### Encrypted Field + Reports
- Encrypted field values display normally in Reports for users with FLS `readable=true`
- **Report filter**: only `=` (equals) supported — not CONTAINS, STARTS WITH, NOT EQUAL (string partial)
- Cannot group by (GROUP BY) or sort by (ORDER BY) an encrypted field in summary/grouped reports
- Workaround for "show me all employees with ID starting with 123": not possible with encrypted field — must use non-encrypted partial match field

### Encrypted Field + Bulk Load (Data Loader / SFDC CLI)
- Can use encrypted field as **ExternalId** for upsert operations
- Matching is exact, case-insensitive (Deterministic CI)
- **Encryption Policy must be activated BEFORE the load** — if encryption is not yet active, upsert matching fails silently or inserts duplicates
- After bulk load: verify via SOQL `WHERE National_ID__pc = 'test_value'` that records match
- Error symptom: upsert inserts new records instead of updating → encryption policy not active or field not CI

### Encrypted Field + Criteria-Based Sharing Rules
- **Encrypted fields cannot be used as criteria** in criteria-based Sharing Rules
- The encrypted field does not appear in the Sharing Rule field picker
- Workaround: use a separate non-encrypted status/category field as the sharing criterion

### Encrypted Field Activation Sequence (Critical Order)
```
1. Deploy field metadata (type=Text/Phone/Email, externalId=true, caseSensitive=false if needed)
2. In Setup → Platform Encryption → Encryption Policy: activate encryption on the field (MANUAL — not deployable)
3. If existing data: Setup → Platform Encryption → Encrypt Existing Data → run job
4. Verify: SOQL query on field = value returns expected records
5. Only after step 2: Bulk Load upserts using this field as ExternalId will work correctly
```

**If step 2 is skipped**: field stores data as plaintext → encryption policy applied later → existing data not encrypted → queries fail to match encrypted vs unencrypted values
