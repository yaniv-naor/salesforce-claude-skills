# SF Field Create Skill

**Version:** 2.0.0  
**Author:** Claude Code  
**License:** MIT

## Overview

Creates Salesforce custom field metadata (`.field-meta.xml`) with proper configuration for all field types. Works across any Salesforce project with automatic prefix detection.

## Features

- Creates all field types (Text, Number, Currency, Picklist, Lookup, Master-Detail, Formula, Rollup Summary, etc.)
- Automatic project prefix detection and application
- Smart prefix handling (standard objects get prefix, custom objects don't)
- Proper relationship configuration (Lookup vs Master-Detail)
- Valid XML generation
- Comprehensive validation

## Installation

1. Copy `SKILL.md` to your project's `.claude/skills/sf-field-create/` directory
2. The skill will be automatically available in Claude Code

## Usage

### Triggering the Skill

Use any of these phrases:
- "create field"
- "new field"
- "add field to [Object]"
- "create lookup"
- "create picklist"
- "add Master-Detail"

### Examples

**Example 1: Text Field on Standard Object**
```
Create a text field called "Notes" on the Account object. Make it 255 characters long.
```

Result: Creates `ENF_Notes__c` on Account (prefix applied for standard object)

**Example 2: Currency Field on Custom Object**
```
Create a currency field called "Amount" on ENF_Invoice__c with 2 decimal places.
```

Result: Creates `Amount__c` on ENF_Invoice__c (no double prefix)

**Example 3: Lookup Relationship**
```
Create a lookup field on ENF_Document__c that relates to Account.
```

Result: Creates `Account__c` lookup field with proper relationship configuration

## Field Types Supported

| Type | Description | Configuration |
|------|-------------|---------------|
| Text | Short text (255 chars) | Length |
| TextArea | Multi-line text | Length |
| LongTextArea | Long text (32K chars) | Length, visible lines |
| RichTextArea | HTML formatted text | Length, visible lines |
| Number | Numeric values | Precision, scale |
| Currency | Money amounts | Precision, scale |
| Percent | Percentage | Precision, scale |
| Date | Date only | None |
| DateTime | Date and time | None |
| Checkbox | Boolean | Default value |
| Picklist | Single-select | Values |
| MultiselectPicklist | Multi-select | Values, visible lines |
| Email | Email address | None |
| Phone | Phone number | None |
| Url | Web address | None |
| Lookup | Object reference | Related object, delete constraint |
| MasterDetail | Parent-child | Related object, relationship order |
| Formula | Calculated | Formula, return type |
| Summary | Rollup aggregation | Summarized field, operation |

## Prefix Rules

### Standard Objects → Prefix Required
Standard objects like Account, Contact, Opportunity always need the project prefix:

```
Account.ENF_CustomField__c     ✅ Correct
Contact.ENF_Email__c           ✅ Correct
Account.CustomField__c         ❌ Wrong
```

### Custom Objects → No Prefix
Custom objects (ending with `__c`) already have the namespace, so fields should NOT have prefix:

```
ENF_Invoice__c.Status__c       ✅ Correct
ENF_Invoice__c.Amount__c       ✅ Correct
ENF_Invoice__c.ENF_Status__c   ❌ Wrong (double prefix)
```

## File Structure

Generated fields are saved in the standard Salesforce structure:

```
force-app/main/default/objects/
├── Account/
│   └── fields/
│       └── ENF_Notes__c.field-meta.xml
├── ENF_Invoice__c/
│   └── fields/
│       ├── Amount__c.field-meta.xml
│       └── Status__c.field-meta.xml
└── ENF_Document__c/
    └── fields/
        └── Account__c.field-meta.xml
```

## Integration

This skill works with:
- `sf-prefix-detect` - Detects project prefix
- `sf-object-create` - Creates custom objects
- `sf-trigger-create` - Creates triggers that use fields

## Testing

See `TEST_RESULTS.md` for comprehensive test results covering:
1. Text field on standard object (with prefix)
2. Currency field on custom object (no prefix)
3. Lookup relationship field

All tests passed ✅

## Key Validations

The skill performs these critical validations:

1. **Prefix Detection:** Automatically determines if prefix is needed
2. **XML Structure:** Ensures valid Salesforce metadata XML
3. **Field Type:** Validates type-specific configuration (length, precision, etc.)
4. **Relationships:** Configures lookup/Master-Detail properly
5. **File Location:** Saves to correct object/fields directory

## Relationship Fields

### Lookup vs Master-Detail

**Lookup:**
- Optional reference to another record
- Delete options: SetNull, Restrict, Cascade
- Up to 40 per object
- Independent security

**Master-Detail:**
- Required parent-child relationship
- Always cascades delete
- Max 2 per object
- Inherits security from parent
- Enables rollup summary fields

## Formula Fields

Supports formula fields with proper return types:
- Text formulas (up to 3,900 chars)
- Number/Currency formulas
- Date/DateTime formulas
- Checkbox (boolean) formulas

Common functions: TEXT(), IF(), AND(), OR(), ISBLANK(), TODAY(), etc.

## Rollup Summary

Only available on Master-Detail parent objects. Supported operations:
- COUNT - Count child records
- SUM - Sum field values
- MIN - Minimum value
- MAX - Maximum value

## Deployment

After creating fields, deploy with:

```bash
# Deploy specific field
sf project deploy start -d force-app/main/default/objects/[Object]/fields/[Field].field-meta.xml

# Deploy all fields for an object
sf project deploy start -d force-app/main/default/objects/[Object]/fields/

# Deploy entire project
sf project deploy start
```

## Troubleshooting

### Error: Master-Detail requires ControlledByParent

**Solution:** Update parent object's sharing model to `ControlledByParent` before deploying Master-Detail field.

### Error: Too many Master-Detail relationships

**Solution:** Salesforce allows max 2 Master-Detail relationships per object. Use Lookup instead.

### Error: Duplicate values for unique field

**Solution:** Ensure all existing records have unique values before deploying a unique field.

### Error: Invalid formula syntax

**Solution:** Test formula in Developer Console or Formula Builder before creating field.

## Best Practices

1. Always use meaningful field names (PascalCase)
2. Add help text for complex fields
3. Set appropriate field-level security
4. Consider validation rules for data quality
5. Add fields to page layouts after deployment
6. Test formulas thoroughly before deployment
7. Use external IDs for integration fields
8. Document relationship fields clearly

## Limitations

- Cannot create polymorphic lookup fields (e.g., Name lookup)
- Cannot modify standard fields
- Formula fields have 3,900 character limit
- Max 2 Master-Detail relationships per object
- Rollup summary only works with Master-Detail

## Version History

### 2.0.0 (2026-05-05)
- Generic/multi-project support
- Automatic prefix detection
- All field types supported
- Comprehensive validation
- Tested on CrmXyz project

### 1.0.0
- Initial version
- Basic field types

## Support

For issues or questions:
1. Check `SKILL.md` for detailed instructions
2. Review `TEST_RESULTS.md` for examples
3. Consult Salesforce metadata API documentation

## License

MIT License - See LICENSE file for details

---

**Generated by Claude Code**  
**Last Updated:** 2026-05-05
