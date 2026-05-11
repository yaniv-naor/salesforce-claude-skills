---
name: sf-validation-create
description: Creates Salesforce Validation Rules with proper formula syntax, error messages, and CDATA handling. Works across any Salesforce project with automatic prefix detection. Use when user says "create validation", "validation rule", "field validation", "enforce rule", "data quality", "business rule", or mentions validation logic.
license: MIT
metadata:
  version: 1.0.0
  category: salesforce-metadata
  tags: [salesforce, validation, formula, data-quality, generic]
---

# SF Validation Create

Creates Salesforce Validation Rule metadata (`.validationRule-meta.xml`) with proper formula syntax, CDATA handling, and validation. **Works with any Salesforce project.**

## When to Use

- Creating validation rules to enforce data quality
- Adding business logic to prevent invalid data
- Enforcing required field combinations
- Validating date ranges, picklist values, or numeric constraints
- Creating conditional validation rules
- Troubleshooting validation rule deployment errors
- Fixing formula syntax errors

## Instructions

### Step 1: Detect Project Prefix

Use `sf-prefix-detect` skill to determine the project prefix.

**Prefix usage in validation rules:**
- Custom fields on **standard objects**: Use prefix (e.g., `PROJ_Status__c` on `Account`)
- Custom fields on **custom objects**: No prefix (e.g., `Status__c` on `PROJ_Invoice__c`)
- Standard fields: No prefix ever (e.g., `Name`, `CreatedDate`, `OwnerId`)

### Step 2: Gather Validation Requirements

Ask user for (or infer from request):

**Required:**
- **Parent object** - Which object needs this validation? (e.g., `Account`, `PROJ_Invoice__c`)
- **Validation purpose** - What business rule are we enforcing?
- **Condition** - When should the error appear?
- **Error message** - What should users see? (Max 255 characters)

**Optional:**
- **Validation name** - Auto-generated from purpose if not provided
- **Description** - Auto-generated from purpose if not provided
- **Active status** - Default is `true`
- **Error location** - Field-specific or top-of-page (default: top)

**Auto-detected:**
- API Name: Derived from validation purpose
- File path: Based on object name
- CDATA requirement: Based on formula content

### Step 3: Understand Common Validation Patterns

**Pattern 1: Required Field Check**
```
Purpose: Ensure Status field is not blank
Formula: ISBLANK(TEXT(Status__c))
```

**Pattern 2: Date Range Validation**
```
Purpose: End date must be after start date
Formula: AND(NOT(ISBLANK(Start_Date__c)), NOT(ISBLANK(End_Date__c)), End_Date__c < Start_Date__c)
```

**Pattern 3: Conditional Required Field**
```
Purpose: Require rejection reason when status is rejected
Formula: AND(ISPICKVAL(Status__c, "Rejected"), ISBLANK(Rejection_Reason__c))
```

**Pattern 4: Picklist Value Validation**
```
Purpose: Validate specific picklist combinations
Formula: AND(ISPICKVAL(Type__c, "Premium"), ISPICKVAL(Status__c, "Draft"))
```

**Pattern 5: Number Range Validation**
```
Purpose: Quantity must be between 1 and 100
Formula: OR(Quantity__c < 1, Quantity__c > 100)
```

**Pattern 6: Cross-Object Validation (via Lookup)**
```
Purpose: Validate against related object field
Formula: Account__r.Status__c = "Inactive"
```

### Step 4: Build Formula with Proper Functions

**Critical Formula Function Guidelines:**

**TEXT() - Convert non-text to text**
- ❌ WRONG: `TEXT(TextFieldName__c)` - Text fields don't need TEXT()
- ✅ RIGHT: `TEXT(PicklistField__c)` - Use for picklists, dates, numbers
- ✅ RIGHT: Remove TEXT() if field is already text type

**ISPICKVAL() - Check picklist values**
- ✅ REQUIRED: Always use ISPICKVAL() for picklist fields
- ❌ WRONG: `TEXT(Status__c) = "Active"` - Don't use TEXT() with equality
- ✅ RIGHT: `ISPICKVAL(Status__c, "Active")`

**CASE() - Multiple conditions**
- ✅ Parameters must be EVEN number (condition/result pairs + default)
- ✅ Last parameter is always the default value
- Example: `CASE(Priority__c, "High", 1, "Medium", 2, "Low", 3, 0)`

**VALUE() - Convert text to number**
- ✅ Use ONLY with text fields
- ❌ WRONG: `VALUE(123)` - Number doesn't need VALUE()
- ✅ RIGHT: `VALUE(TextFieldWithNumber__c)`

**DAY() / MONTH() / YEAR() - Date parts**
- ✅ Use ONLY with Date fields
- ❌ WRONG: `DAY(DateTimeField__c)` - DateTime needs conversion first
- ✅ RIGHT: `DAY(DATEVALUE(DateTimeField__c))`

**DATEVALUE() - Convert DateTime to Date**
- ✅ Use ONLY with DateTime fields
- ❌ WRONG: `DATEVALUE(DateField__c)` - Date doesn't need DATEVALUE()
- ✅ RIGHT: `DATEVALUE(DateTimeField__c)`

**ISCHANGED() - Detect field changes**
- ✅ Use to check if field value changed in update
- Example: `AND(ISCHANGED(Status__c), ISPICKVAL(Status__c, "Closed"))`

**Common Functions:**
- `ISBLANK()` - Check if field is empty (works with all types)
- `AND()`, `OR()`, `NOT()` - Boolean logic
- `IF()` - Conditional logic: `IF(condition, true_value, false_value)`
- `TODAY()` - Current date
- `NOW()` - Current date/time

### Step 5: Determine CDATA Requirement

**CRITICAL RULE: CDATA for XML Tags**

Validation formulas containing XML special characters MUST be wrapped in CDATA sections.

**When to use CDATA:**
- Formula contains `<` (less than)
- Formula contains `>` (greater than)
- Formula contains `&` (ampersand)
- Formula contains `"` in complex contexts

**Examples requiring CDATA:**

**Example 1: Less than comparison**
```xml
<errorConditionFormula><![CDATA[End_Date__c < Start_Date__c]]></errorConditionFormula>
```

**Example 2: Complex AND/OR logic**
```xml
<errorConditionFormula><![CDATA[
AND(
    NOT(ISBLANK(Start_Date__c)),
    NOT(ISBLANK(End_Date__c)),
    End_Date__c < Start_Date__c
)
]]></errorConditionFormula>
```

**Examples NOT requiring CDATA:**

**Example 1: Simple equality**
```xml
<errorConditionFormula>ISBLANK(Status__c)</errorConditionFormula>
```

**Example 2: Using ISPICKVAL**
```xml
<errorConditionFormula>ISPICKVAL(Status__c, "Active")</errorConditionFormula>
```

### Step 6: Validate Formula Syntax

Before generating XML, validate:

**Syntax Checks:**
- [ ] Parentheses balanced
- [ ] Quotes properly closed
- [ ] Field names valid (with proper prefix rules)
- [ ] Functions used correctly (TEXT, ISPICKVAL, CASE, etc.)
- [ ] Logical operators properly nested (AND, OR, NOT)

**Function Validation:**
- [ ] TEXT() not used on text fields
- [ ] ISPICKVAL() used for all picklist comparisons
- [ ] CASE() has even number of parameters
- [ ] VALUE() only used with text fields
- [ ] DAY()/MONTH() only used with Date fields (not DateTime)
- [ ] DATEVALUE() only used with DateTime fields (not Date)

**Field Reference Checks:**
- [ ] Standard object fields: Correct prefix if custom
- [ ] Custom object fields: No prefix
- [ ] Lookup relationships: Use `__r` notation (e.g., `Account__r.Name`)

### Step 7: Create Error Message

**Error Message Guidelines:**

**Length:** Maximum 255 characters

**Clarity:** Be specific about what's wrong and how to fix it

**Good Examples:**
- ✅ "End Date must be after Start Date."
- ✅ "Rejection Reason is required when Status is Rejected."
- ✅ "Quantity must be between 1 and 100."
- ✅ "Cannot set Status to Closed if Amount is greater than $10,000 without approval."

**Bad Examples:**
- ❌ "Invalid data" - Too vague
- ❌ "This field combination is not allowed" - Not specific enough
- ❌ "Error" - Doesn't explain anything

**Tips:**
- Use active voice
- Be concise but specific
- Suggest the fix when possible
- Use proper punctuation

### Step 8: Generate Validation Rule XML

Create `.validationRule-meta.xml` file with proper structure:

**Basic Structure (No CDATA needed):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationRule xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Generated by Claude Code on [YYYY-MM-DD] -->
    
    <fullName>Validation_Rule_Name</fullName>
    <active>true</active>
    <description>[Description] - Generated by Claude Code</description>
    <errorConditionFormula>ISBLANK(Status__c)</errorConditionFormula>
    <errorMessage>Status is required.</errorMessage>
</ValidationRule>
```

**With CDATA (for XML special characters):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationRule xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Generated by Claude Code on [YYYY-MM-DD] -->
    
    <fullName>Validation_Rule_Name</fullName>
    <active>true</active>
    <description>[Description] - Generated by Claude Code</description>
    <errorConditionFormula><![CDATA[
        AND(
            NOT(ISBLANK(Start_Date__c)),
            NOT(ISBLANK(End_Date__c)),
            End_Date__c < Start_Date__c
        )
    ]]></errorConditionFormula>
    <errorMessage>End Date must be after Start Date.</errorMessage>
</ValidationRule>
```

**Optional: Field-Level Error Display:**
```xml
<errorDisplayField>FieldName__c</errorDisplayField>
```

**Key principles:**
- ✅ Use descriptive validation rule names
- ✅ Include attribution in description
- ✅ Wrap formulas with `<`, `>`, `&` in CDATA
- ✅ Keep error messages under 255 characters
- ✅ Set `<active>true</active>` to enable immediately

### Step 9: Complete Examples

**Example 1: Simple Required Field**

**Business Rule:** Status field must not be blank

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationRule xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Generated by Claude Code on 2026-05-05 -->
    
    <fullName>Require_Status</fullName>
    <active>true</active>
    <description>Ensures Status field is not blank - Generated by Claude Code</description>
    <errorConditionFormula>ISBLANK(TEXT(Status__c))</errorConditionFormula>
    <errorMessage>Status is required.</errorMessage>
</ValidationRule>
```

**File name:** `Require_Status.validationRule-meta.xml`

**Example 2: Date Range Validation (with CDATA)**

**Business Rule:** End date must be after start date

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationRule xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Generated by Claude Code on 2026-05-05 -->
    
    <fullName>Valid_Date_Range</fullName>
    <active>true</active>
    <description>Ensures End Date is after Start Date - Generated by Claude Code</description>
    <errorConditionFormula><![CDATA[
        AND(
            NOT(ISBLANK(Start_Date__c)),
            NOT(ISBLANK(End_Date__c)),
            End_Date__c < Start_Date__c
        )
    ]]></errorConditionFormula>
    <errorMessage>End Date must be after Start Date.</errorMessage>
    <errorDisplayField>End_Date__c</errorDisplayField>
</ValidationRule>
```

**File name:** `Valid_Date_Range.validationRule-meta.xml`

**Example 3: Conditional Required Field**

**Business Rule:** Rejection reason required when status is rejected

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationRule xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Generated by Claude Code on 2026-05-05 -->
    
    <fullName>Require_Reason_When_Rejected</fullName>
    <active>true</active>
    <description>Requires rejection reason when status is Rejected - Generated by Claude Code</description>
    <errorConditionFormula>
        AND(
            ISPICKVAL(Status__c, "Rejected"),
            ISBLANK(Rejection_Reason__c)
        )
    </errorConditionFormula>
    <errorMessage>Rejection Reason is required when Status is Rejected.</errorMessage>
    <errorDisplayField>Rejection_Reason__c</errorDisplayField>
</ValidationRule>
```

**File name:** `Require_Reason_When_Rejected.validationRule-meta.xml`

**Example 4: Picklist Combination Validation**

**Business Rule:** Cannot set Type to Premium if Status is Draft

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationRule xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Generated by Claude Code on 2026-05-05 -->
    
    <fullName>Prevent_Premium_Draft</fullName>
    <active>true</active>
    <description>Prevents Premium type when status is Draft - Generated by Claude Code</description>
    <errorConditionFormula>
        AND(
            ISPICKVAL(Type__c, "Premium"),
            ISPICKVAL(Status__c, "Draft")
        )
    </errorConditionFormula>
    <errorMessage>Type cannot be Premium when Status is Draft.</errorMessage>
</ValidationRule>
```

**File name:** `Prevent_Premium_Draft.validationRule-meta.xml`

**Example 5: Number Range Validation (with CDATA)**

**Business Rule:** Quantity must be between 1 and 100

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationRule xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Generated by Claude Code on 2026-05-05 -->
    
    <fullName>Quantity_Valid_Range</fullName>
    <active>true</active>
    <description>Ensures Quantity is between 1 and 100 - Generated by Claude Code</description>
    <errorConditionFormula><![CDATA[
        OR(
            Quantity__c < 1,
            Quantity__c > 100
        )
    ]]></errorConditionFormula>
    <errorMessage>Quantity must be between 1 and 100.</errorMessage>
    <errorDisplayField>Quantity__c</errorDisplayField>
</ValidationRule>
```

**File name:** `Quantity_Valid_Range.validationRule-meta.xml`

**Example 6: Cross-Field Formula (with CDATA)**

**Business Rule:** Amount cannot exceed Budget

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationRule xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Generated by Claude Code on 2026-05-05 -->
    
    <fullName>Amount_Within_Budget</fullName>
    <active>true</active>
    <description>Ensures Amount does not exceed Budget - Generated by Claude Code</description>
    <errorConditionFormula><![CDATA[
        AND(
            NOT(ISBLANK(Amount__c)),
            NOT(ISBLANK(Budget__c)),
            Amount__c > Budget__c
        )
    ]]></errorConditionFormula>
    <errorMessage>Amount cannot exceed Budget.</errorMessage>
    <errorDisplayField>Amount__c</errorDisplayField>
</ValidationRule>
```

**File name:** `Amount_Within_Budget.validationRule-meta.xml`

### Step 10: Save File

Save to standard Salesforce structure:

```
force-app/main/default/objects/
└── [ObjectAPIName]/
    └── validationRules/
        └── [ValidationRuleName].validationRule-meta.xml
```

**Example paths:**
- `force-app/main/default/objects/Account/validationRules/Require_Status.validationRule-meta.xml`
- `force-app/main/default/objects/PROJ_Invoice__c/validationRules/Valid_Date_Range.validationRule-meta.xml`

### Step 11: Provide Summary

```
✅ Created Validation Rule: [ValidationRuleName]

📋 Validation Details:
- Parent Object: [ObjectAPIName]
- Rule Name: [ValidationRuleName]
- Active: [Yes/No]
- Error Message: [Error Message]

🔍 Formula:
[Display formula for user review]

📝 Next Steps:
1. Review formula logic and error message
2. Test validation with sample data in Salesforce
3. Deploy: sf project deploy start -d force-app/main/default/objects
4. Verify validation fires correctly in UI
5. Update help text on affected fields if needed

💡 Tips:
- Test validation with both valid and invalid data
- Consider deactivating during bulk data loads
- Update page layouts to show error messages clearly
- [If CDATA:] Formula contains XML characters, properly wrapped in CDATA

📍 File Created:
- [file_path]
```

## Critical Rules & Constraints

### Naming Conventions

**Validation Rule Names:**
- ✅ Alphanumeric characters and underscores only
- ✅ Must start with a letter
- ✅ Cannot end with underscore
- ✅ Cannot contain consecutive underscores
- ✅ Maximum 40 characters
- ❌ NO `__c` suffix (unlike fields)

**Good Names:**
- ✅ `Require_Status`
- ✅ `Valid_Date_Range`
- ✅ `Prevent_Premium_Draft`

**Bad Names:**
- ❌ `Require Status` (spaces not allowed)
- ❌ `Require_Status__c` (no __c suffix)
- ❌ `_Require_Status` (cannot start with underscore)
- ❌ `Require_Status_` (cannot end with underscore)
- ❌ `Require__Status` (no consecutive underscores)

### Formula Function Rules

**TEXT() Function:**
| Field Type | Use TEXT()? | Example |
|------------|-------------|---------|
| Text | ❌ NO | `ISBLANK(TextFieldName__c)` |
| Picklist | ✅ YES (or use ISPICKVAL) | `TEXT(PicklistField__c) = "Value"` |
| Number | ✅ YES | `TEXT(NumberField__c) = "100"` |
| Date | ✅ YES | `TEXT(DateField__c)` |
| Checkbox | ✅ YES | `TEXT(CheckboxField__c) = "true"` |

**ISPICKVAL() Requirement:**
- ✅ ALWAYS use for picklist comparisons
- ❌ NEVER use TEXT() + equality for picklists
- Example: `ISPICKVAL(Status__c, "Active")` not `TEXT(Status__c) = "Active"`

**CASE() Function:**
- ✅ Must have EVEN number of parameters
- ✅ Last parameter is default value
- Example: `CASE(Priority__c, "High", 1, "Medium", 2, 0)` - 5 params (2 pairs + default)

### CDATA Rules

**MUST use CDATA when formula contains:**
- `<` (less than)
- `>` (greater than)
- `&` (ampersand)
- Multiple `"` in complex expressions

**CDATA syntax:**
```xml
<errorConditionFormula><![CDATA[
    Your formula here with < or > or &
]]></errorConditionFormula>
```

**DO NOT use CDATA when:**
- Formula has simple equality checks
- Using only ISBLANK(), ISPICKVAL(), AND(), OR() without comparisons
- No XML special characters present

### Error Message Constraints

- Maximum 255 characters
- Must be clear and actionable
- Should explain what's wrong and how to fix
- Use proper grammar and punctuation

### Validation Rule Behavior

**When Triggered:**
- On record create
- On record update
- On record save (before actual save)

**Prevents:**
- Saving invalid data
- API updates with invalid data
- Bulk imports with invalid data

**Does NOT Affect:**
- Existing records (until they're edited)
- Admin/system processes (if configured)

## Verification Checklist

Before generating validation rule XML, verify:

### Required Elements
- [ ] `<fullName>` present and follows naming rules
- [ ] `<active>` set to true or false
- [ ] `<errorConditionFormula>` present and syntactically correct
- [ ] `<errorMessage>` present and under 255 characters
- [ ] `<description>` present and meaningful

### Formula Validation
- [ ] TEXT() not used on text fields
- [ ] ISPICKVAL() used for all picklist comparisons
- [ ] CASE() has even number of parameters
- [ ] VALUE() only used with text fields
- [ ] DAY()/MONTH() only used with Date fields
- [ ] DATEVALUE() only used with DateTime fields
- [ ] Parentheses balanced
- [ ] Field references correct (with proper prefix rules)

### CDATA Check
- [ ] If formula contains `<`, `>`, or `&`: Wrapped in CDATA
- [ ] If no special characters: No CDATA needed

### Best Practices
- [ ] Error message is clear and actionable
- [ ] Description explains business rule
- [ ] errorDisplayField set if specific field error
- [ ] Attribution included in description

## Examples

### Example 1: Simple Required Field

**User:** "Make Status field required on Invoice object"

**Result:**
- Formula: `ISBLANK(TEXT(Status__c))`
- Error: "Status is required."
- CDATA: Not needed

### Example 2: Date Range

**User:** "Ensure end date is after start date"

**Result:**
- Formula: `AND(NOT(ISBLANK(Start_Date__c)), NOT(ISBLANK(End_Date__c)), End_Date__c < Start_Date__c)`
- Error: "End Date must be after Start Date."
- CDATA: Required (contains `<`)

### Example 3: Conditional Required

**User:** "Require approval reason when status is rejected"

**Result:**
- Formula: `AND(ISPICKVAL(Status__c, "Rejected"), ISBLANK(Approval_Reason__c))`
- Error: "Approval Reason is required when Status is Rejected."
- CDATA: Not needed

## Troubleshooting

### Error: TEXT() on text field

**Error Message:**
```
Incorrect parameter type for function 'TEXT()'. Expected Number, Date, DateTime, Picklist, received Text
```

**Solution:**
1. Identify text fields in formula
2. Remove TEXT() wrapper
3. Change `ISBLANK(TEXT(TextField__c))` to `ISBLANK(TextField__c)`

### Error: CASE() parameter count

**Error Message:**
```
Incorrect number of parameters for function 'CASE()'. Expected even number, received odd
```

**Solution:**
1. Count CASE() parameters
2. Add default value as last parameter
3. Example: `CASE(Field, "A", 1, "B", 2, 0)` - 5 params (2 pairs + default = even parameters after field name)

### Error: Missing CDATA

**Error Message:**
```
Invalid XML character in formula
```

**Solution:**
1. Check formula for `<`, `>`, or `&`
2. Wrap formula in CDATA section:
```xml
<errorConditionFormula><![CDATA[
    End_Date__c < Start_Date__c
]]></errorConditionFormula>
```

### Error: Invalid picklist comparison

**Error Message:**
```
Field is a picklist, so you must use ISPICKVAL function
```

**Solution:**
1. Replace `TEXT(PicklistField__c) = "Value"` with `ISPICKVAL(PicklistField__c, "Value")`
2. Always use ISPICKVAL() for picklist fields

### Error: Invalid field reference

**Error Message:**
```
Field does not exist or insufficient permissions
```

**Solution:**
1. Verify field name and API name
2. Check prefix rules:
   - Standard object custom fields: WITH prefix
   - Custom object fields: NO prefix
3. Verify field exists on object

### Error: Formula syntax error

**Error Message:**
```
Syntax error. Missing ')'
```

**Solution:**
1. Count opening and closing parentheses
2. Use formula editor to validate syntax
3. Check for proper nesting of functions

### Error: Error message too long

**Error Message:**
```
Error message exceeds maximum length of 255 characters
```

**Solution:**
1. Shorten error message
2. Keep it concise but clear
3. Focus on what's wrong and how to fix

## Reference

### Common Formula Patterns

**Required Field:**
```
ISBLANK(FieldName__c)
```

**Required Picklist:**
```
ISPICKVAL(PicklistField__c, "")
```

**Date Range:**
```
AND(NOT(ISBLANK(StartDate__c)), NOT(ISBLANK(EndDate__c)), EndDate__c < StartDate__c)
```

**Conditional Required:**
```
AND(ISPICKVAL(Status__c, "Rejected"), ISBLANK(Reason__c))
```

**Number Range:**
```
OR(Amount__c < 0, Amount__c > 10000)
```

**Cross-Field Comparison:**
```
AND(NOT(ISBLANK(Amount__c)), NOT(ISBLANK(Budget__c)), Amount__c > Budget__c)
```

**Lookup Validation:**
```
AND(NOT(ISBLANK(Account__c)), Account__r.Status__c = "Inactive")
```

**Changed Field Check:**
```
AND(ISCHANGED(Status__c), ISPICKVAL(Status__c, "Closed"))
```

### Function Quick Reference

| Function | Purpose | Example |
|----------|---------|---------|
| ISBLANK() | Check if empty | `ISBLANK(Field__c)` |
| ISPICKVAL() | Check picklist value | `ISPICKVAL(Status__c, "Active")` |
| TEXT() | Convert to text | `TEXT(DateField__c)` |
| VALUE() | Convert to number | `VALUE(TextField__c)` |
| AND() | All conditions true | `AND(A, B, C)` |
| OR() | Any condition true | `OR(A, B, C)` |
| NOT() | Negate condition | `NOT(ISBLANK(Field__c))` |
| IF() | Conditional | `IF(Amount__c > 100, true, false)` |
| CASE() | Multiple conditions | `CASE(Field, "A", 1, "B", 2, 0)` |
| TODAY() | Current date | `TODAY()` |
| NOW() | Current datetime | `NOW()` |
| DATEVALUE() | DateTime to Date | `DATEVALUE(DateTimeField__c)` |
| DAY() | Day of month | `DAY(DateField__c)` |
| MONTH() | Month number | `MONTH(DateField__c)` |
| YEAR() | Year | `YEAR(DateField__c)` |
| ISCHANGED() | Field changed | `ISCHANGED(Field__c)` |

---

**Generated by Claude Code**
**Last Updated:** 2026-05-05
**Version:** 1.0.0 (Generic/Multi-Project Support)
