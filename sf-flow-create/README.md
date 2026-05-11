# SF Flow Create Skill

A Salesforce skill that creates Flow metadata files (`.flow-meta.xml`) with proper configuration for Screen Flows, Autolaunched Flows, and Record-Triggered Flows.

## Overview

This skill helps create Salesforce Flow metadata with the correct structure, naming conventions, and configuration. It works across any Salesforce project by automatically detecting the project prefix and API version.

## Features

- **Multiple Flow Types**: Supports Screen, Autolaunched, Record-Triggered, and Scheduled flows
- **Automatic Prefix Detection**: Integrates with `sf-prefix-detect` to use correct project prefix
- **API Version Detection**: Reads API version from `sfdx-project.json`
- **Validation**: Includes checks for flow type, XML structure, DML in loops, and trigger configuration
- **Best Practices**: Follows Salesforce Flow best practices for bulkification and error handling

## Supported Flow Types

| Flow Type | Use Case | Process Type | Trigger Config |
|-----------|----------|--------------|----------------|
| **Screen Flow** | Interactive user processes | `Flow` | None |
| **Autolaunched Flow** | Called from Apex/other flows | `AutoLaunchedFlow` | None |
| **Record-Triggered Flow** | Automation on record changes | `AutoLaunchedFlow` | Required |
| **Scheduled Flow** | Recurring automation | `AutoLaunchedFlow` | Schedule config |

## Usage Examples

### Screen Flow
```
Create a screen flow called "DocumentApproval" for document approval process.
```

**Result**: Creates `ABC_DocumentApproval.flow-meta.xml` with:
- `processType`: `Flow`
- Status: `Draft`
- No trigger configuration

### Record-Triggered Flow (Before Save)
```
Create a record-triggered flow called "DocumentBeforeInsert" that runs before a Document__c record is inserted to validate required fields.
```

**Result**: Creates `ABC_DocumentBeforeInsert.flow-meta.xml` with:
- `processType`: `AutoLaunchedFlow`
- `triggerType`: `RecordBeforeSave`
- `recordTriggerType`: `Create`
- `object`: `Document__c`

### Autolaunched Flow
```
Create an autolaunched flow called "ProcessDocuments" that can be called from Apex to process multiple documents.
```

**Result**: Creates `ABC_ProcessDocuments.flow-meta.xml` with:
- `processType`: `AutoLaunchedFlow`
- No trigger or schedule configuration
- Description mentions invocable nature

## File Structure

```
force-app/main/default/flows/
└── [PREFIX]_[FlowName].flow-meta.xml
```

Example:
- `ABC_DocumentApproval.flow-meta.xml`
- `ABC_DocumentBeforeInsert.flow-meta.xml`
- `ABC_ProcessDocuments.flow-meta.xml`

## Validation Checks

The skill performs these validations:

1. **Flow Type Validation**
   - Screen Flow: `processType` = `Flow`
   - Record-Triggered/Autolaunched: `processType` = `AutoLaunchedFlow`

2. **Trigger Configuration Validation** (Record-Triggered only)
   - Must have `object` element
   - Must have `recordTriggerType` (Create, Update, Delete)
   - Must have `triggerType` (RecordBeforeSave, RecordAfterSave)

3. **XML Structure Validation**
   - Valid XML syntax
   - Required elements: `apiVersion`, `label`, `processType`, `start`, `status`
   - Proper namespace: `http://soap.sforce.com/2006/04/metadata`

4. **DML in Loops Detection**
   - Warns if DML operations found inside loop elements
   - Suggests bulkification patterns

## Test Results

### Iteration 1 - All Tests Pass ✅

**Test Coverage:**
- ✅ Screen Flow (8 assertions)
- ✅ Record-Triggered Flow Before Save (10 assertions)
- ✅ Autolaunched Flow (10 assertions)

**Pass Rate:** 100% (28/28 assertions)

**Key Validations:**
- Correct process type for each flow type
- Proper trigger configuration for record-triggered flows
- No trigger config for screen/autolaunched flows
- Correct prefix usage (FRC)
- Correct API version (64.0)
- Valid XML structure
- Draft status for new flows

## Next Steps After Creating a Flow

1. **Open Flow Builder**: Setup → Flows → [Flow Name]
2. **Build Flow Logic**:
   - Add elements (Get Records, Create Records, Update Records, Decisions, etc.)
   - Configure entry conditions (if Record-Triggered)
   - Add fault handling for errors
   - Test with Debug mode
3. **Activate Flow**: Change status from Draft to Active
4. **Deploy**: `sf project deploy start -d force-app/main/default/flows`
5. **Monitor**: Setup → Flows → [Flow Name] → Run History

## Best Practices

### Bulkification
- ✅ Use collection variables and process in bulk
- ✅ Perform DML operations outside of loops
- ❌ Never put Create/Update Records inside a Loop

### Error Handling
- Add fault paths to all DML elements
- Display error messages to users (Screen Flows)
- Log errors for debugging (Record-Triggered)

### Performance
- Use specific filter conditions
- Limit record retrieval when possible
- Index frequently queried fields

### Security
- Use "Respect field-level security" checkbox
- Check user permissions before DML
- Don't expose sensitive data in Screen Flows

## Dependencies

- **sf-prefix-detect**: Used to detect project prefix
- **sfdx-project.json**: Used to detect API version
- **force-app/main/default/flows**: Standard Salesforce directory structure

## License

MIT License - Created by Claude Code

## Version

2.0.0 - Generic/Multi-Project Support
