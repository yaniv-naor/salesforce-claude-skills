# SF Validation Create Skill

## Overview

The `sf-validation-create` skill creates Salesforce Validation Rules with proper formula syntax, error messages, and CDATA handling. It works across any Salesforce project with automatic prefix detection.

## Purpose

This skill helps developers:
- Create validation rules with correct formula syntax
- Properly use Salesforce formula functions (TEXT, ISPICKVAL, CASE, etc.)
- Handle XML special characters with CDATA sections
- Generate clear, actionable error messages
- Follow Salesforce best practices for validation rules
- Avoid common formula syntax errors

## When to Use

Use this skill when you need to:
- Create new validation rules
- Enforce business logic at the data layer
- Validate field values, ranges, or combinations
- Prevent invalid data from being saved
- Implement conditional validation logic
- Troubleshoot validation rule formula errors

## Trigger Phrases

The skill activates when users say:
- "create validation"
- "validation rule"
- "field validation"
- "enforce rule"
- "data quality"
- "business rule"
- "prevent invalid data"
- "validate field"

## Key Features

### 1. Formula Function Validation
- Ensures TEXT() is not used on text fields
- Enforces ISPICKVAL() for picklist comparisons
- Validates CASE() parameter count (must be even)
- Checks VALUE() usage with text fields only
- Validates DAY()/MONTH() with Date fields
- Ensures DATEVALUE() with DateTime fields only

### 2. CDATA Handling
- Automatically detects when CDATA is needed
- Properly wraps formulas containing XML special characters:
  - `<` (less than)
  - `>` (greater than)
  - `&` (ampersand)
- Prevents XML parsing errors

### 3. Error Message Generation
- Creates clear, actionable error messages
- Ensures messages are under 255 characters
- Suggests fixes when appropriate
- Uses proper grammar and punctuation

### 4. Project Integration
- Works with any Salesforce project
- Automatic prefix detection via sf-prefix-detect skill
- Proper prefix usage for standard vs custom objects
- Correct file path generation

## Project Structure

```
sf-validation-create/
├── SKILL.md              # Main skill instructions
├── README.md             # This file
└── tests/
    ├── test-1-simple-required-field.md
    ├── test-2-date-range-validation.md
    └── test-3-complex-conditional.md
```

## Test Cases

### Test 1: Simple Required Field
Tests basic validation rule creation with ISBLANK() function and proper TEXT() usage.

**Input:** "Create a validation rule that requires the Status field"

**Expected:** Simple formula with no CDATA needed

### Test 2: Date Range Validation
Tests validation with comparison operators requiring CDATA wrapping.

**Input:** "Ensure End Date is after Start Date"

**Expected:** Formula with CDATA, proper date comparison

### Test 3: Complex Conditional
Tests complex logic with multiple conditions, picklists, and CDATA.

**Input:** "Require Approval Reason when Status is Rejected and Amount > 1000"

**Expected:** Complex formula with ISPICKVAL, ISCHANGED, and CDATA

## Running Tests

To test this skill:

1. Navigate to the skill directory
2. Read each test case in the `tests/` folder
3. Execute the test input
4. Verify the output matches expected behavior
5. Check the validation checklist for each test

## Common Use Cases

### 1. Required Field Validation
```
Formula: ISBLANK(FieldName__c)
CDATA: Not needed
```

### 2. Date Range Validation
```
Formula: AND(NOT(ISBLANK(Start__c)), NOT(ISBLANK(End__c)), End__c < Start__c)
CDATA: Required (contains <)
```

### 3. Conditional Required
```
Formula: AND(ISPICKVAL(Status__c, "Rejected"), ISBLANK(Reason__c))
CDATA: Not needed
```

### 4. Number Range
```
Formula: OR(Amount__c < 0, Amount__c > 10000)
CDATA: Required (contains < and >)
```

### 5. Picklist Combination
```
Formula: AND(ISPICKVAL(Type__c, "Premium"), ISPICKVAL(Status__c, "Draft"))
CDATA: Not needed
```

## Integration with Other Skills

This skill integrates with:
- **sf-prefix-detect**: Detects project prefix for field references
- **sf-object-create**: Validation rules are added to objects
- **sf-field-create**: Validates fields that validation rules reference
- **sf-validate-all**: Validates syntax before deployment

## Troubleshooting Guide

### Issue: TEXT() on text field error
**Solution:** Remove TEXT() wrapper from text fields

### Issue: Missing CDATA error
**Solution:** Wrap formula with `<` or `>` in CDATA section

### Issue: CASE() parameter count error
**Solution:** Ensure even number of parameters (pairs + default)

### Issue: ISPICKVAL error
**Solution:** Use ISPICKVAL() instead of TEXT() for picklist comparisons

### Issue: Formula syntax error
**Solution:** Check parentheses balance and function nesting

## Attribution

**Author:** Claude Code  
**License:** MIT  
**Version:** 1.0.0  
**Category:** Salesforce Metadata Development  
**Last Updated:** 2026-05-05

## Related Documentation

- [Salesforce Validation Rule Documentation](https://help.salesforce.com/s/articleView?id=sf.fields_valid_formulas.htm)
- [Formula Field Reference](https://help.salesforce.com/s/articleView?id=sf.customize_functions.htm)
- [CDATA in XML](https://www.w3.org/TR/xml/#sec-cdata-sect)

## Contributing

To improve this skill:
1. Add new test cases for edge cases
2. Expand troubleshooting section
3. Add more formula pattern examples
4. Update documentation with lessons learned

## Version History

### 1.0.0 (2026-05-05)
- Initial release
- Support for all common formula functions
- CDATA detection and handling
- Comprehensive test suite
- Integration with sf-prefix-detect
- Claude Code attribution
