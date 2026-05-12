---
name: sf-translation-create
description: Creates Salesforce Translation files (global and object-level) for Hebrew and Arabic. Translates all metadata (objects, fields, picklists, validation rules, layouts, tabs, apps) automatically with high accuracy. Use when user asks to "translate", "add translation", "create Hebrew/Arabic translations", "תרגום", or mentions translating Salesforce metadata to Hebrew/Arabic. Works across any Salesforce project. ALWAYS use this skill when translation is mentioned, even if user doesn't explicitly say "create translation file".
license: MIT
metadata:
  author: Israeli MOF Development Team
  version: 1.0.0
  category: salesforce-development
  tags: [salesforce, translation, metadata, hebrew, arabic, i18n, localization]
---

# SF Translation Create

Creates Salesforce translation files for Hebrew (iw) and Arabic (ar) with automatic translation of all metadata components. Supports both **global translation files** and **object-level translations**.

## When to Use

- Translating Salesforce metadata to Hebrew or Arabic
- Creating translation files for new or existing projects
- Adding translations for objects, fields, picklists, validation rules
- Translating UI elements (tabs, apps, buttons, labels)
- Localizing Salesforce applications for Hebrew/Arabic users
- After completing project build when all English metadata is ready
- **IMPORTANT:** After adding ANY new component (field, object, validation rule, etc.) - run this skill to update translations
- **INCREMENTAL UPDATES:** When user adds new components to existing project, automatically check if translations need updating

## Translation Architecture

Salesforce supports two translation approaches, and this skill uses BOTH:

### 1. Global Translation File
**Path:** `force-app/main/default/translations/[language].translation-meta.xml`

Translates:
- Custom Applications (Lightning Apps)
- Custom Labels
- Custom Tabs
- Flows (button labels, screen text)
- Quick Actions
- Global UI elements

### 2. Object-Level Translation Files
**Path:** `force-app/main/default/objectTranslations/[ObjectName]-[language]/[ObjectName]-[language].objectTranslation-meta.xml`

Translates per object:
- Object label and plural label (with Hebrew grammatical cases)
- Field labels and help text
- Picklist values (each value individually)
- Validation rule error messages
- Record type labels
- Page layout section labels
- Quick action labels
- List view labels
- Compact layout labels

## Instructions

### Step 0: Detect Project Prefix

Use `sf-prefix-detect` skill to determine the project prefix if not already known.

### Step 1: Gather Translation Requirements

**Ask user:**

1. **Target languages:**
   - Default: Hebrew (iw)
   - **ALWAYS ask:** "Do you also want to translate to Arabic (ar)?"
   - If yes, create translations for both languages

2. **Translation source:**
   - "Do you have a translation dictionary/glossary file?"
   - If yes: "Please attach it in any format (Excel, CSV, JSON, Word, etc.)"
   - If no: Proceed with automatic translation

3. **Scope:**
   - "Should I translate the entire project, or specific objects?"
   - Default: Entire project (recommended)

### Step 2: Check for Existing Translations

**CRITICAL - Check Translation Status:**

Before creating ANY translation files, check if translations already exist:

1. **Check for existing object translations:**
   ```bash
   ls force-app/main/default/objectTranslations/*/
   ```
   
2. **Check for existing global translation:**
   ```bash
   ls force-app/main/default/translations/
   ```

**Translation Update Strategy:**

For each component to translate:

- ✅ **No existing translation** → Create new translation
- ✅ **Partial translation** (some fields/values missing) → **Add ONLY missing translations**
- ❌ **Full translation exists** → **Skip, do not modify**

**How to detect partial translation:**
1. Read existing translation file
2. Compare with current English metadata
3. Identify missing translations (new fields, new picklist values, new objects)
4. Add ONLY the missing parts

### Step 3: Scan Metadata for Translatable Components

**IMPORTANT:** Only translate English metadata that does NOT have existing translations.

**Scan for:**

#### A. Global Components
- [ ] Custom Applications (Lightning Apps)
  - Path: `force-app/main/default/applications/*.app-meta.xml`
  - Extract: `<label>` tags
- [ ] Custom Labels
  - Path: `force-app/main/default/labels/CustomLabels.labels-meta.xml`
  - Extract: all `<fullName>` and `<value>` pairs
- [ ] Custom Tabs
  - Path: `force-app/main/default/tabs/*.tab-meta.xml`
  - Extract: `<label>` tags
- [ ] Flows
  - Path: `force-app/main/default/flows/*.flow-meta.xml`
  - Extract: screen labels, button labels, field labels, help text
  - NOTE: Flow translations are complex - extract `<label>` and `<helpText>` from flow screens

#### B. Object-Level Components

For each object (both standard and custom):

- [ ] **Object metadata**
  - Path: `force-app/main/default/objects/[ObjectName]/[ObjectName].object-meta.xml`
  - Extract: `<label>`, `<pluralLabel>`, `<description>` (if exists)

- [ ] **Fields**
  - Path: `force-app/main/default/objects/[ObjectName]/fields/*.field-meta.xml`
  - Extract: `<label>`, `<description>`, `<inlineHelpText>`
  - For picklists: extract ALL `<fullName>` values from `<valueSet>` → `<valueSetDefinition>` → `<value>`

- [ ] **Validation Rules**
  - Path: `force-app/main/default/objects/[ObjectName]/validationRules/*.validationRule-meta.xml`
  - Extract: `<errorMessage>`

- [ ] **Record Types**
  - Path: `force-app/main/default/objects/[ObjectName]/recordTypes/*.recordType-meta.xml`
  - Extract: `<label>`, `<description>`

- [ ] **Page Layouts**
  - Path: `force-app/main/default/objects/[ObjectName]/layouts/*.layout-meta.xml`
  - Extract: section labels from `<layoutSections>` → `<label>`

- [ ] **Compact Layouts**
  - Path: `force-app/main/default/objects/[ObjectName]/compactLayouts/*.compactLayout-meta.xml`
  - Extract: `<label>`

- [ ] **List Views**
  - Path: `force-app/main/default/objects/[ObjectName]/listViews/*.listView-meta.xml`
  - Extract: `<label>`

- [ ] **Quick Actions**
  - Path: `force-app/main/default/objects/[ObjectName]/quickActions/*.quickAction-meta.xml`
  - Extract: `<label>`

### Step 3: Process Translation Input (if provided)

If user provided translation glossary:

1. **Parse the file** (Excel/CSV/JSON/Word/etc.)
2. **Extract mappings:** English → Hebrew (and Arabic if applicable)
3. **Build translation dictionary:**
   ```
   {
     "Account": "חשבון",
     "Contact": "איש קשר",
     "Status": "סטטוס",
     "Active": "פעיל",
     ...
   }
   ```
4. **Flag missing translations:** Any English text not in glossary

### Step 4: Automatic Translation

For all text NOT in user glossary:

**Translation approach:**
- Use Claude's bilingual capabilities for high-accuracy Hebrew/Arabic translation
- Context-aware: Field labels vs UI buttons vs error messages
- Technical terminology: Preserve when appropriate (e.g., "Email", "ID")
- Abbreviations: Expand when needed (e.g., "No." → "מספר")

**Translation principles:**
1. **Preserve technical terms** when commonly used in Hebrew/Arabic
   - Example: "Email" → "Email" (not "דוא״ל" unless specified)
   - Example: "ID" → "מזהה" (translate)
2. **Formal language** for business applications
3. **Concise labels** - Hebrew/Arabic can be longer than English
4. **Gender-aware** - Hebrew has masculine/feminine forms
5. **Right-to-left** - No special handling needed (Salesforce handles RTL)

**Track uncertain translations:**
- Ambiguous terms
- Domain-specific jargon
- Abbreviations without context
- Very long phrases

### Step 5: Generate Hebrew Object Translation Files

For each object that has translatable components:

**File structure:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObjectTranslation xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Hebrew grammatical cases for object name -->
    <caseValues>
        <article>Definite</article>
        <plural>false</plural>
        <value>[Object name with ה' prefix, singular]</value>
    </caseValues>
    <caseValues>
        <article>None</article>
        <plural>false</plural>
        <value>[Object name without ה', singular]</value>
    </caseValues>
    <caseValues>
        <article>Definite</article>
        <plural>true</plural>
        <value>[Object name with ה' prefix, plural]</value>
    </caseValues>
    <caseValues>
        <article>None</article>
        <plural>true</plural>
        <value>[Object name without ה', plural]</value>
    </caseValues>
    <gender>[Masculine|Feminine]</gender>
    
    <!-- Field translations -->
    <fields>
        <label>[Hebrew label]</label>
        <name>[FieldAPIName__c]</name>
        <!-- If picklist: -->
        <picklistValues>
            <masterLabel>[English value]</masterLabel>
            <translation>[Hebrew translation]</translation>
        </picklistValues>
        <!-- If has help text: -->
        <help>[Hebrew help text]</help>
    </fields>
    
    <!-- Layout translations -->
    <layouts>
        <layout>[LayoutAPIName]</layout>
        <sections>
            <label>[Hebrew section label]</label>
            <section>[English section name]</section>
        </sections>
    </layouts>
    
    <!-- Record type translations -->
    <recordTypes>
        <label>[Hebrew label]</label>
        <name>[RecordTypeAPIName]</name>
    </recordTypes>
    
    <!-- Validation rule error messages -->
    <validationRules>
        <errorMessage>[Hebrew error message]</errorMessage>
        <name>[ValidationRuleAPIName]</name>
    </validationRules>
    
    <!-- Quick actions -->
    <quickActions>
        <label>[Hebrew label]</label>
        <name>[QuickActionAPIName]</name>
    </quickActions>
    
    <!-- Name field label -->
    <nameFieldLabel>[Hebrew name field label]</nameFieldLabel>
</CustomObjectTranslation>
```

**Path:** `force-app/main/default/objectTranslations/[ObjectName]-iw/[ObjectName]-iw.objectTranslation-meta.xml`

**Hebrew grammatical cases explained:**
- **Definite/None** - With/without ה' prefix (the)
- **Singular/Plural** - יחיד/רבים
- **Gender** - Masculine or Feminine (affects verb conjugation in UI)

**Examples:**
```xml
<!-- Feminine example: "Request" -->
<caseValues>
    <article>Definite</article>
    <plural>false</plural>
    <value>הבקשה</value> <!-- the request -->
</caseValues>
<caseValues>
    <article>None</article>
    <plural>false</plural>
    <value>בקשה</value> <!-- request -->
</caseValues>
<caseValues>
    <article>Definite</article>
    <plural>true</plural>
    <value>הבקשות</value> <!-- the requests -->
</caseValues>
<caseValues>
    <article>None</article>
    <plural>true</plural>
    <value>בקשות</value> <!-- requests -->
</caseValues>
<gender>Feminine</gender>
```

### Step 6: Generate Global Translation File (Hebrew)

**Path:** `force-app/main/default/translations/iw.translation-meta.xml`

**Structure:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Translations xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Custom Applications -->
    <customApplications>
        <label>[Hebrew app label]</label>
        <name>[AppAPIName]</name>
    </customApplications>
    
    <!-- Custom Labels -->
    <customLabels>
        <label>[Hebrew label]</label>
        <name>[CustomLabelAPIName]</name>
    </customLabels>
    
    <!-- Custom Tabs -->
    <customTabs>
        <label>[Hebrew label]</label>
        <name>[TabAPIName]</name>
    </customTabs>
    
    <!-- Flow Definitions (if applicable) -->
    <flowDefinitions>
        <flows>
            <screens>
                <fields>
                    <fieldText>[Hebrew text]</fieldText>
                    <name>[FieldAPIName]</name>
                </fields>
                <name>[ScreenAPIName]</name>
                <nextOrFinishButtonLabel>[Hebrew button label]</nextOrFinishButtonLabel>
            </screens>
        </flows>
        <fullName>[FlowAPIName]</fullName>
    </flowDefinitions>
</Translations>
```

### Step 7: Generate Arabic Translations (if requested)

Repeat Steps 5 and 6 for Arabic:
- Language code: `ar`
- File paths: `objectTranslations/[ObjectName]-ar/` and `translations/ar.translation-meta.xml`
- Arabic grammatical rules: Similar structure to Hebrew but different cases
- Gender: Arabic also has masculine/feminine

### Step 8: Components That CANNOT Be Translated

**Do NOT attempt to translate:**
- ❌ **Queues** - No translation support in metadata
- ❌ **API Names** - Always remain in English
- ❌ **Formula fields** - Formula syntax must stay English
- ❌ **Workflow rules** - Criteria stay English
- ❌ **Apex code** - Code stays English
- ❌ **Custom metadata type records** - Usually technical
- ❌ **Permission sets/profiles** - Internal names stay English

**When encountering non-translatable components:**
- List them clearly
- Explain why they can't be translated
- Confirm they're correctly excluded

### Step 9: Validation

**CRITICAL:** Run dry-run validation for EACH translation file immediately after creation.

```bash
# Validate object translation
sf project deploy start --source-dir force-app/main/default/objectTranslations/[ObjectName]-iw --target-org [OrgAlias] --dry-run

# Validate global translation
sf project deploy start --source-dir force-app/main/default/translations --target-org [OrgAlias] --dry-run
```

**Common validation errors:**
1. **Field name doesn't exist** - Typo in API name
2. **Invalid picklist value** - Typo in masterLabel
3. **Layout doesn't exist** - Wrong layout API name
4. **XML syntax error** - Malformed tags, missing closing tags
5. **Encoding issue** - File must be UTF-8

**Fix ALL errors immediately before proceeding.**

### Step 10: Summary Report

Provide comprehensive summary:

**Translation Summary:**
```
🔄 Translation Mode: [New Translation | Update Existing]

✅ Translations Created/Updated:
   - Language(s): Hebrew (iw) [+ Arabic (ar)]
   - Objects translated: X (Y new, Z updated)
   - Fields translated: A (B new, C updated)
   - Picklist values translated: D (E new, F updated)
   - Validation rules translated: G
   - Custom labels translated: H
   - Tabs translated: I
   - Apps translated: J

📁 Files Created/Modified:
   - Global translation: translations/iw.translation-meta.xml [CREATED|UPDATED]
   - Object translations: X files (Y new, Z updated)
   
   New files:
   - objectTranslations/NewObject__c-iw/NewObject__c-iw.objectTranslation-meta.xml
   
   Updated files:
   - objectTranslations/ExistingObject__c-iw/ExistingObject__c-iw.objectTranslation-meta.xml
     → Added 3 new fields, 5 new picklist values

✅ Validation Status: All files passed dry-run validation

⚠️ Uncertain Translations (please review):
   - "[English term]" → "[Hebrew translation]" (reason: ambiguous context)
   - "[English term]" → "[Hebrew translation]" (reason: technical jargon)

✅ Skipped (existing translations preserved):
   - ObjectName__c: All fields already translated
   - AnotherObject__c: Partial translation exists, added only missing items
   
❌ Non-Translatable Components (correctly excluded):
   - Queue: [QueueName] (queues cannot be translated)
   - Formula field: [FieldName] (formulas stay in English)

📋 Next Steps:
   1. Review uncertain translations above
   2. Test in scratch org: sf org open
   3. Switch to Hebrew: Setup → My Settings → Language → עברית
   4. Verify all labels appear correctly
   5. Deploy to target org when satisfied
```

### Step 11: User Review Guidance

**Tell the user:**

"Please review the uncertain translations I listed. You can:
1. Open the translation files directly and edit
2. Provide corrections and I'll update
3. Test in the org by switching to Hebrew (Setup → Language)

To test translations:
1. Open scratch org: `sf org open`
2. Go to Setup → My Settings → Language & Time Zone
3. Select 'עברית' (Hebrew) or 'العربية' (Arabic)
4. Navigate through your app and verify labels
5. Check picklist values, error messages, page layouts"

## Advanced: Handling Special Cases

### Case 1: Standard Objects (Account, Contact, etc.)

Standard objects already have Salesforce-provided translations. You should ONLY create object translation files for custom fields added to standard objects.

**Example - Contact with custom fields:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomObjectTranslation xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Do NOT translate standard fields (Email, Phone, etc.) -->
    <!-- ONLY translate custom fields -->
    <fields>
        <label>תפקיד</label>
        <name>PROJ_Role__c</name>
    </fields>
</CustomObjectTranslation>
```

### Case 2: Picklist Value Dependencies

If picklist has controlling/dependent relationships, ALL values must be translated consistently.

### Case 3: Record Type Picklist Values

Record types restrict picklist values. Ensure translated picklist values match the record type configuration.

### Case 4: Multi-Language Projects

If project already has Hebrew translations and you're adding Arabic:
- Do NOT regenerate Hebrew files
- Only create new Arabic files
- Use existing Hebrew translations as reference

### Case 5: Updating Existing Translations (Most Common Scenario)

**This is a CRITICAL feature** - when new components are added to the project:

**Workflow for updating existing translations:**

1. **Detect what changed in English metadata:**
   - New fields added to existing objects?
   - New picklist values added to existing fields?
   - New objects created?
   - New validation rules, record types, layouts?

2. **Read existing translation file:**
   ```bash
   # For object translations
   cat force-app/main/default/objectTranslations/ObjectName-iw/ObjectName-iw.objectTranslation-meta.xml
   
   # For global translation
   cat force-app/main/default/translations/iw.translation-meta.xml
   ```

3. **Compare and identify gaps:**
   - English field exists but NOT in translation file → **Add translation**
   - Picklist has 5 values but translation only has 3 → **Add missing 2**
   - Object exists but no translation file → **Create new file**

4. **Add ONLY missing translations:**
   - Preserve ALL existing translations exactly as-is
   - Insert new translations in correct XML structure
   - Maintain alphabetical order if applicable
   - Keep existing formatting

5. **Validate the updated file:**
   - Ensure XML is well-formed
   - Run dry-run validation
   - Verify no existing translations were changed

**Example - Adding new field to existing object:**

**Before** (`objectTranslations/Account-iw/Account-iw.objectTranslation-meta.xml`):
```xml
<CustomObjectTranslation>
    <fields>
        <label>תיאור</label>
        <name>Description</name>
    </fields>
</CustomObjectTranslation>
```

**After** (new field `Priority__c` added):
```xml
<CustomObjectTranslation>
    <fields>
        <label>תיאור</label>
        <name>Description</name>
    </fields>
    <!-- NEW: Added translation for new field -->
    <fields>
        <label>עדיפות</label>
        <name>Priority__c</name>
        <picklistValues>
            <masterLabel>High</masterLabel>
            <translation>גבוה</translation>
        </picklistValues>
        <picklistValues>
            <masterLabel>Medium</masterLabel>
            <translation>בינוני</translation>
        </picklistValues>
        <picklistValues>
            <masterLabel>Low</masterLabel>
            <translation>נמוך</translation>
        </picklistValues>
    </fields>
</CustomObjectTranslation>
```

**NEVER:**
- ❌ Delete existing translations
- ❌ Modify existing translations (unless user explicitly requests)
- ❌ Reorder existing translations (preserve original structure)
- ❌ Change translation wording without user approval

## Troubleshooting

### Error: "Field does not exist"
- **Cause:** Typo in field API name
- **Fix:** Verify field exists in object metadata, check spelling

### Error: "Invalid picklist value"
- **Cause:** masterLabel doesn't match exact picklist value
- **Fix:** Copy exact value from field metadata (case-sensitive)

### Error: "Invalid XML"
- **Cause:** Syntax error, special characters not escaped
- **Fix:** Escape special characters: `&` → `&amp;`, `<` → `&lt;`, `"` → `&quot;`, `'` → `&apos;`

### Hebrew/Arabic Text Appears as Gibberish
- **Cause:** File encoding is not UTF-8
- **Fix:** Ensure all `.objectTranslation-meta.xml` files are UTF-8 encoded

### Translations Don't Appear in Org
- **Cause:** User's language setting not changed
- **Fix:** Setup → My Settings → Language → Select Hebrew/Arabic

## Tips for High-Quality Translations

1. **Context matters:** "Status" might be "סטטוס" (technical) or "מצב" (state) depending on context
2. **Consistency:** Use same translation for same term across all objects
3. **Brevity:** Hebrew labels can be longer than English - keep them concise
4. **User-facing vs technical:** Error messages should be friendly, field labels can be technical
5. **Test thoroughly:** Always test in actual org with Hebrew/Arabic enabled

## Common Terms Reference

**Business Objects:**
- Account → חשבון
- Contact → איש קשר
- Opportunity → הזדמנות
- Lead → לקוח פוטנציאלי
- Case → פנייה

**Common Fields:**
- Name → שם
- Status → סטטוס
- Date → תאריך
- Description → תיאור
- Owner → בעלים
- Created By → נוצר על ידי
- Last Modified → שונה לאחרונה

**UI Elements:**
- Save → שמור
- Cancel → ביטול
- Edit → עריכה
- Delete → מחיקה
- New → חדש
- Search → חיפוש

## Definition of Done

Translation is complete when:
- ✅ All translatable components scanned
- ✅ All object translation files created
- ✅ Global translation file created
- ✅ All files pass dry-run validation (0 errors)
- ✅ Uncertain translations listed for user review
- ✅ Non-translatable components documented
- ✅ Summary report provided
- ✅ Testing guidance given to user

---

**Generated by Claude Code - SF Translation Create Skill v1.0.0**
