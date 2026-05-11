# SF Apex Invocable Skill

Creates Salesforce Invocable Methods (callable from Flow) with proper structure including Request/Response classes, error handling, and bulkification.

## Overview

This skill generates production-ready Apex invocable methods that can be called from Salesforce Flow Builder. It follows best practices for:

- @InvocableMethod annotation with proper metadata
- Request/Response wrapper classes with @InvocableVariable
- Bulkification (List input/output)
- Comprehensive error handling
- Test class generation with 75%+ coverage

## When to Use

Use this skill when you need to:

- Create Flow actions (callable from Flow Builder)
- Build reusable logic for Screen Flows or Record-Triggered Flows
- Enable Flow to call complex Apex logic
- Integrate external systems via Flow (with HTTP callouts)
- Perform calculations or validations from Flow

## Trigger Phrases

- "create invocable"
- "invocable method"
- "Flow action"
- "call from Flow"
- "@InvocableMethod"
- "create action for Flow Builder"

## Key Features

### 1. Proper Annotation Structure

```apex
@InvocableMethod(
    label='Action Name in Flow Builder'
    description='Detailed description for Flow Builder'
    category='Category'
    callout=true  // Only if HTTP callouts needed
)
```

### 2. Request/Response Pattern

**Request class (inputs from Flow):**
```apex
public class Request {
    @InvocableVariable(label='Field Name' required=true)
    public String fieldName;
}
```

**Response class (outputs to Flow):**
```apex
public class Response {
    @InvocableVariable(label='Success')
    public Boolean isSuccess;
    
    @InvocableVariable(label='Error Message')
    public String errorMessage;
}
```

### 3. Bulkification

Always processes List<Request> and returns List<Response>:

```apex
public static List<Response> execute(List<Request> requests) {
    // Collect IDs for bulk query
    Set<Id> recordIds = new Set<Id>();
    for (Request req : requests) {
        recordIds.add(req.recordId);
    }
    
    // Query once outside loop
    Map<Id, SObject> recordsById = new Map<Id, SObject>([...]);
    
    // Process each request
    for (Request req : requests) {
        // Process using queried data
    }
}
```

### 4. Error Handling

Graceful error handling that returns errors in Response (recommended):

```apex
try {
    // Business logic
    res.isSuccess = true;
} catch (Exception e) {
    res.isSuccess = false;
    res.errorMessage = e.getMessage();
    res.errorType = e.getTypeName();
}
```

### 5. Comprehensive Test Coverage

Generated test classes include:
- Success scenarios
- Bulk processing (200+ records)
- Error handling (missing required fields, invalid data)
- Edge cases specific to the business logic

## Test Cases

### Test Case 1: Send Email Invocable

**Prompt:** Create an invocable method to send emails from Flow. The action should allow sending custom emails with recipient, subject, body, and optional CC.

**Generated Files:**
- `ABC_SendEmailInvocable.cls` - Main invocable class
- `ABC_SendEmailInvocable_Test.cls` - Test class with 6 test methods

**Features:**
- Bulkified email sending using Messaging.sendEmail()
- Required fields: recipient email, subject, body
- Optional fields: CC email
- Validates all required inputs
- Returns success/error for each email

**Test Coverage:**
- Success with single email
- Success with CC
- Bulk scenario (10 emails)
- Missing recipient error
- Missing subject error
- Missing body error

### Test Case 2: Calculate Discount Invocable

**Prompt:** Create an invocable method to calculate discount amounts. It should take an original amount and discount percentage as inputs, and return the discounted amount, discount value, and success status.

**Generated Files:**
- `ABC_CalculateDiscountInvocable.cls` - Main invocable class
- `ABC_CalculateDiscountInvocable_Test.cls` - Test class with 9 test methods

**Features:**
- Calculates discount value and final discounted amount
- Validates percentage range (0-100)
- Validates non-negative amounts
- Returns all calculated values to Flow

**Test Coverage:**
- 10% discount calculation
- 50% discount calculation
- 0% discount (no discount)
- Bulk scenario (200 calculations)
- Missing original amount error
- Missing discount percentage error
- Negative amount error
- Invalid percentage (>100) error
- Negative percentage error

### Test Case 3: Call External API Invocable

**Prompt:** Create an invocable method to call external APIs from Flow. It should support GET and POST methods, accept endpoint URL and request body, and return status code and response body.

**Generated Files:**
- `ABC_CallExternalAPIInvocable.cls` - Main invocable class
- `ABC_CallExternalAPIInvocable_Test.cls` - Test class with 9 test methods

**Features:**
- Supports GET, POST, PUT, DELETE methods
- Defaults to GET when method not specified
- Accepts JSON request body
- Returns HTTP status code and response body
- Proper callout annotation (@InvocableMethod callout=true)
- 2-minute timeout

**Test Coverage:**
- GET request success
- POST request success
- Default GET method when not specified
- 404 error handling
- 500 error handling
- Missing endpoint error
- Bulk scenario (5 API calls)
- PUT method
- DELETE method

## Supported Input/Output Types

| Type | Supported | Notes |
|------|-----------|-------|
| String | ✅ | Most common |
| Integer | ✅ | Whole numbers |
| Boolean | ✅ | True/false |
| Decimal | ✅ | Decimal numbers |
| Id | ✅ | Salesforce record IDs |
| Date | ✅ | Date only |
| DateTime | ✅ | Date and time |
| SObject | ✅ | Generic or specific types |
| List<T> | ✅ | Collections of supported types |
| Map | ❌ | Not supported |
| Set | ❌ | Not supported |

## Usage in Flow Builder

1. Deploy the invocable class to your Salesforce org
2. Open Flow Builder
3. Add an "Action" element
4. Search for the action by its label (e.g., "Send Custom Email")
5. Configure input variables
6. Use output variables in subsequent Flow elements

## Best Practices Implemented

1. **Naming Convention**: `[PREFIX]_[ActionName]Invocable`
2. **Sharing Model**: `with sharing` for security
3. **API Version**: 62.0
4. **Attribution**: CREATED BY: Claude Code header
5. **Documentation**: Comprehensive JavaDoc comments
6. **Bulkification**: No SOQL/DML in loops
7. **Error Handling**: Try-catch with meaningful error messages
8. **Test Coverage**: Multiple test methods covering success and error paths
9. **Assertions**: Modern Assert.* syntax
10. **Meta.xml**: Proper metadata files for all classes

## Integration with Other Skills

This skill works seamlessly with:

- **sf-prefix-detect**: Automatically detects project prefix (FRC, ENF, etc.)
- **sf-apex-create**: Can be used as an alternative for non-invocable Apex classes

## File Structure

```
iteration-1/
├── send-email-invocable/
│   ├── inputs/
│   │   └── prompt.txt
│   ├── with_skill/
│   │   └── outputs/
│   │       ├── ABC_SendEmailInvocable.cls
│   │       ├── ABC_SendEmailInvocable.cls-meta.xml
│   │       ├── ABC_SendEmailInvocable_Test.cls
│   │       └── ABC_SendEmailInvocable_Test.cls-meta.xml
│   └── eval_metadata.json
├── calculate-discount-invocable/
│   ├── inputs/
│   │   └── prompt.txt
│   ├── with_skill/
│   │   └── outputs/
│   │       ├── ABC_CalculateDiscountInvocable.cls
│   │       ├── ABC_CalculateDiscountInvocable.cls-meta.xml
│   │       ├── ABC_CalculateDiscountInvocable_Test.cls
│   │       └── ABC_CalculateDiscountInvocable_Test.cls-meta.xml
│   └── eval_metadata.json
└── call-api-invocable/
    ├── inputs/
    │   └── prompt.txt
    ├── with_skill/
    │   └── outputs/
    │       ├── ABC_CallExternalAPIInvocable.cls
    │       ├── ABC_CallExternalAPIInvocable.cls-meta.xml
    │       ├── ABC_CallExternalAPIInvocable_Test.cls
    │       └── ABC_CallExternalAPIInvocable_Test.cls-meta.xml
    └── eval_metadata.json
```

## Common Patterns

### Pattern 1: Data Processing (No Callout)

Use for calculations, validations, data transformations:
- Category: "Calculation" or "Validation"
- No `callout=true` needed
- Fast execution
- Can be called from before-save flows

### Pattern 2: Email/Messaging

Use for sending emails or SMS:
- Category: "Communication"
- Uses Messaging.sendEmail() or similar
- Bulkified sending
- Returns success/failure per message

### Pattern 3: External Integration (With Callout)

Use for API calls:
- Category: "Integration"
- Requires `callout=true`
- Cannot be called from before-save flows
- Needs Remote Site Settings configured

## Troubleshooting

### Error: Method must be static

**Cause:** Missing `static` keyword

**Solution:** Add `static` to method signature:
```apex
public static List<Response> execute(List<Request> requests)
```

### Error: Callout not allowed

**Cause:** Missing `callout=true` in annotation

**Solution:** Add to @InvocableMethod:
```apex
@InvocableMethod(callout=true)
```

### Error: Cannot find action in Flow

**Cause:** Class not deployed or label not matching

**Solution:**
1. Deploy class to org
2. Refresh Flow Builder
3. Search by exact label

## Version

**Skill Version:** 2.0.0
**Created:** May 2026
**Author:** Claude Code
**License:** MIT

## Summary

The sf-apex-invocable skill successfully generates production-ready invocable methods that:

- Follow Salesforce best practices
- Include comprehensive test coverage
- Handle errors gracefully
- Support bulkification
- Work seamlessly in Flow Builder
- Are ready for deployment

All three test cases demonstrate different use cases:
1. Email sending (Messaging API)
2. Data calculation (pure computation)
3. External API integration (HTTP callouts)

The skill is ready for use in any Salesforce project.
