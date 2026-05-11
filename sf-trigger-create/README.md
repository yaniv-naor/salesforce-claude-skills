# sf-trigger-create

Creates Salesforce Triggers with Handler pattern including Handler class and test class. Follows best practices with bulkification and proper trigger contexts.

## Overview

This skill automates the creation of complete, production-ready Salesforce trigger implementations following industry best practices. It generates:

- **Trigger file** (.trigger) - Minimal routing logic
- **Handler class** (.cls) - All business logic with proper bulkification
- **Test class** (_Test.cls) - Comprehensive tests including bulk scenarios (200+ records)
- **Metadata files** (.trigger-meta.xml, .cls-meta.xml) - Proper Salesforce metadata

## When to Use

Use this skill when you need to:
- Create new triggers for Salesforce objects (standard or custom)
- Implement handler pattern for trigger management
- Set up trigger testing framework
- Add trigger contexts to existing implementations

**Trigger phrases:**
- "create trigger"
- "new trigger"
- "trigger for [object name]"
- "trigger handler"
- "add trigger to [object]"

## Features

### Handler Pattern

Implements the recommended handler pattern:
- **One trigger per object** - Ensures predictable execution order
- **Minimal trigger code** - Only routing logic, no business rules
- **Handler class** - All business logic organized by trigger context
- **Easy maintenance** - Add new logic without modifying trigger

### Best Practices

✅ **Bulkification** - Designed for 200+ records  
✅ **Context-aware** - All 7 trigger contexts (or specific ones on request)  
✅ **ApexDoc comments** - Full documentation on all methods  
✅ **Test coverage** - Comprehensive test class with bulk tests  
✅ **Error handling** - Includes error scenario testing  
✅ **Helper methods** - Section for complex logic isolation  

### Flexible Context Support

- **All contexts** (default): before/after insert, update, delete, undelete
- **Specific contexts**: Request only the contexts you need
- **Clean implementation**: No unused or commented-out code

## Quick Start

### Example 1: Custom Object with All Contexts

**You say:**
```
Create a trigger for ABC_Document__c that handles validation and related record creation
```

**Generated:**
- `XYZ_DocumentTrigger.trigger` - All 7 trigger contexts
- `XYZ_DocumentHandler.cls` - Handler with all context methods
- `XYZ_DocumentHandler_Test.cls` - Complete test class
- Metadata files for all components

### Example 2: Standard Object

**You say:**
```
Create trigger for Account to track status changes and update related opportunities
```

**Generated:**
- `XYZ_AccountTrigger.trigger` - Account trigger
- `XYZ_AccountHandler.cls` - Handler accepting Account types
- `XYZ_AccountHandler_Test.cls` - Tests with Account records
- Metadata files

### Example 3: Specific Contexts Only

**You say:**
```
Create a trigger for ABC_CaseFile__c with only beforeInsert and afterUpdate contexts for validation and status tracking
```

**Generated:**
- `XYZ_CaseFileTrigger.trigger` - ONLY before insert and after update
- `XYZ_CaseFileHandler.cls` - ONLY two methods (no unused code)
- `XYZ_CaseFileHandler_Test.cls` - Tests for both contexts
- Metadata files

## File Structure

```
force-app/main/default/
├── triggers/
│   ├── [PREFIX]_[ObjectName]Trigger.trigger
│   └── [PREFIX]_[ObjectName]Trigger.trigger-meta.xml
└── classes/
    ├── [PREFIX]_[ObjectName]Handler.cls
    ├── [PREFIX]_[ObjectName]Handler.cls-meta.xml
    ├── [PREFIX]_[ObjectName]Handler_Test.cls
    └── [PREFIX]_[ObjectName]Handler_Test.cls-meta.xml
```

## What You Get

### Trigger File

```apex
trigger XYZ_DocumentTrigger on ABC_Document__c (
    before insert, before update, before delete,
    after insert, after update, after delete, after undelete
) {
    XYZ_DocumentHandler handler = new XYZ_DocumentHandler();
    
    // Minimal routing logic only
    if (Trigger.isBefore) {
        if (Trigger.isInsert) {
            handler.beforeInsert(Trigger.new);
        } else if (Trigger.isUpdate) {
            handler.beforeUpdate(Trigger.newMap, Trigger.oldMap);
        }
        // ... other contexts
    }
}
```

### Handler Class

```apex
public with sharing class XYZ_DocumentHandler {
    
    /**
     * Before Insert - Set defaults, validate data
     * @param newRecords List of new records (no IDs yet)
     */
    public void beforeInsert(List<ABC_Document__c> newRecords) {
        // TODO: Implement before insert logic
        // - Set default field values
        // - Validate required data
        // - Derive calculated fields
    }
    
    // ... other context methods with proper signatures
    
    // ==================== HELPER METHODS ====================
    // Bulkified helper methods here
}
```

### Test Class

```apex
@isTest
private class XYZ_DocumentHandler_Test {
    
    @testSetup
    static void setupTestData() {
        // Shared test data (200 records)
    }
    
    @isTest
    static void testBeforeInsert() {
        // Test before insert logic
    }
    
    @isTest
    static void testBulkInsert() {
        // Test with 200+ records for bulkification
    }
    
    // ... 7-9 test methods covering all scenarios
}
```

## How It Works

### 1. Prefix Detection

Automatically detects your project prefix:
- Reads working directory name
- Transforms to uppercase with underscore (e.g., `CrmXyz` → `XYZ_`)
- Can also use sf-prefix-detect skill

### 2. Smart Object Naming

Extracts clean object names:
- `ABC_Document__c` → `Document` (for DocumentTrigger, DocumentHandler)
- `Account` → `Account` (for AccountTrigger, AccountHandler)

### 3. API Version Detection

Reads from `sfdx-project.json`:
```json
{
  "sourceApiVersion": "64.0"
}
```

Falls back to `66.0` if not found.

### 4. Code Generation

Generates all files following the templates in SKILL.md:
- Trigger with proper delegation
- Handler with all requested context methods
- Test class with bulk tests (200+ records)
- Metadata files with correct API version

## Trigger Context Reference

| Context | When | Use For |
|---------|------|---------|
| **before insert** | Before records saved | Validation, defaults, calculated fields |
| **before update** | Before updates saved | Validation, prevent invalid changes |
| **before delete** | Before records deleted | Prevention logic, permission checks |
| **after insert** | After records saved with IDs | Create related records, notifications |
| **after update** | After updates saved | Update related records, workflows |
| **after delete** | After records deleted | Cleanup, archiving |
| **after undelete** | After records restored | Restore related data |

## Best Practices Implemented

### One Trigger Per Object

❌ **Bad:** Multiple triggers on same object (unpredictable order)  
✅ **Good:** One trigger delegates to handler (consistent execution)

### Bulkification

❌ **Bad:** SOQL/DML inside loops  
✅ **Good:** Queries outside loops, use Maps for lookups

Example from generated handler:
```apex
// Good pattern - SOQL outside loop
Set<Id> parentIds = new Set<Id>();
for (Object record : newRecords) {
    parentIds.add(record.Id);
}
List<Related__c> allRelated = [
    SELECT Id, Parent__c FROM Related__c
    WHERE Parent__c IN :parentIds
];
```

### Test Coverage

Generated test classes include:
- ✅ Tests for each trigger context
- ✅ Bulk tests with 200+ records
- ✅ Positive and negative scenarios
- ✅ Error handling tests
- ✅ Proper test patterns (Test.startTest/stopTest)
- ✅ Assert statements for verification

**Target:** >75% coverage (aim for 85%+)

## Testing the Generated Code

### 1. Run Tests

```bash
sf apex run test --tests XYZ_DocumentHandler_Test
```

### 2. Check Coverage

```bash
sf apex get test --test-run-id <ID> --code-coverage
```

### 3. Deploy

```bash
sf project deploy start -d force-app/main/default
```

## Customization

After generation, implement the TODOs in the handler class:

1. **Handler Class** - Add business logic to context methods
2. **Test Class** - Complete test data setup with required fields
3. **Helper Methods** - Add bulkified helper methods as needed

## Requirements

- Salesforce DX project structure
- `force-app/main/default/triggers/` directory
- `force-app/main/default/classes/` directory
- `sfdx-project.json` (optional, for API version)

## Limitations

- Generates skeleton code with TODOs (you implement the business logic)
- Does not query object metadata for required fields
- Assumes standard Salesforce project structure
- One trigger per skill invocation (run multiple times for multiple triggers)

## Version History

**v2.0.0** (2026-05-05)
- Generic multi-project support
- Updated frontmatter and metadata
- Claude Code authorship
- Comprehensive test coverage

**v1.0.0**
- Initial release
- Handler pattern implementation
- Bulk testing support

## Testing

See [TEST_CASES.md](TEST_CASES.md) for detailed test scenarios and [TEST_RESULTS.md](TEST_RESULTS.md) for validation results.

**Test Status:** ✅ All tests passing (3/3)

## License

MIT License

## Author

Claude Code

## Support

For issues or questions:
1. Review [SKILL.md](SKILL.md) for detailed instructions
2. Check [TEST_CASES.md](TEST_CASES.md) for examples
3. Review generated code for inline comments and TODOs

---

**Generated by Claude Code**  
**Category:** Salesforce Development  
**Tags:** salesforce, trigger, handler, testing, bulkification, generic
