# SF Apex Async Skill

Creates production-grade asynchronous Apex classes (Queueable, Batch, Schedulable) with proper error handling, chaining patterns, and comprehensive test coverage.

## Purpose

This skill automates the creation of async Apex classes following Salesforce best practices and Claude Code standards. It handles:

- **Queueable Classes**: With Finalizer pattern, chaining support, and callout capability
- **Batch Classes**: With start/execute/finish pattern, partial success handling, and stateful counters
- **Schedulable Classes**: With cron expressions and delegation to Batch/Queueable

## When to Use

Trigger this skill when the user mentions:
- "create queueable"
- "batch class" or "batch job"
- "schedulable" or "scheduled job"
- "async apex" or "asynchronous"
- "background job" or "background processing"
- "scheduled task" or "recurring job"

## Key Features

### Queueable
- ✅ Chain depth limiting (prevents infinite loops)
- ✅ Finalizer pattern for cleanup
- ✅ Optional Database.AllowsCallouts
- ✅ Job tracking with AsyncApexJob
- ✅ Error handling in Finalizer

### Batch
- ✅ Three-phase pattern (start/execute/finish)
- ✅ Partial success with Database.update(records, false)
- ✅ Optional Database.Stateful for counters
- ✅ Custom query support
- ✅ Job statistics in finish()

### Schedulable
- ✅ Delegates to Batch or Queueable
- ✅ Cron expression examples
- ✅ scheduleJob() helper method
- ✅ Error handling and logging

## Architecture

```
sf-apex-async/
├── skill.md                          # Main skill implementation
├── README.md                         # This file
└── tests/                            # Test cases
    ├── test-1-queueable-with-finalizer.md
    ├── test-2-batch-class.md
    └── test-3-schedulable-with-batch.md
```

## Integration

This skill integrates with:
- **sf-prefix-detect**: For project prefix detection
- **sf-apex-create**: Shares base Apex patterns
- **sf-validate-all**: Can validate async classes

## Test Cases

### Test 1: Queueable with Finalizer
- **Input**: "Create a queueable class to process Document records for status updates"
- **Validates**: Queueable interface, Finalizer pattern, chaining logic, test coverage
- **Files**: Main class + Test class + Metadata files

### Test 2: Batch Class
- **Input**: "Create a batch job to update all Case records older than 30 days with status Closed"
- **Validates**: Batch interface, start/execute/finish, partial success, bulk testing (250+ records)
- **Files**: Main class + Test class + Metadata files

### Test 3: Schedulable with Batch
- **Input**: "Create a scheduled job to run document cleanup daily at 2 AM"
- **Validates**: Schedulable interface, cron expressions, delegation, CronTrigger tests
- **Files**: Main class + Test class + Metadata files

## Usage Examples

### Example 1: Create Queueable
```
User: "Create a queueable to sync documents with external API"

Skill generates:
- XYZ_DocumentSyncQueueable.cls
  - Implements Queueable, Database.AllowsCallouts
  - Includes Finalizer for error handling
  - Chain depth limiting (MAX_CHAIN_DEPTH = 5)
  - Proper error logging
- XYZ_DocumentSyncQueueable_Test.cls
  - testQueueableExecution()
  - testQueueableWithEmptyList()
  - testQueueableChaining()
  - testFinalizerOnError()
```

### Example 2: Create Batch
```
User: "Need a batch to update 50,000 Account records"

Skill generates:
- XYZ_AccountUpdateBatch.cls
  - Implements Database.Batchable<SObject>
  - start() with QueryLocator
  - execute() with partial success
  - finish() with job statistics
  - Database.Stateful for counters
- XYZ_AccountUpdateBatch_Test.cls
  - testBatchExecution()
  - testBatchWithCustomQuery()
  - testBatchWithSmallBatchSize()
  - testBatchPartialSuccess()
  - testBatchWithNoRecords()
```

### Example 3: Create Schedulable
```
User: "Schedule a daily job at 2 AM to cleanup old records"

Skill generates:
- XYZ_DailyCleanupSchedulable.cls
  - Implements Schedulable
  - Delegates to Batch job
  - Static scheduleJob() helper
  - Cron expression examples
- XYZ_DailyCleanupSchedulable_Test.cls
  - testSchedulableExecution()
  - testSchedulableManualExecution()
  - testScheduleJobWithDifferentCronExpressions()
```

## Async Patterns

### Queueable Chaining
```apex
public void execute(QueueableContext context) {
    if (chainDepth >= MAX_CHAIN_DEPTH) {
        return; // Prevent infinite chain
    }
    
    // Process records...
    
    // Chain next job
    if (hasMoreWork()) {
        System.enqueueJob(new MyQueueable(nextBatch, chainDepth + 1));
    }
}
```

### Batch Partial Success
```apex
Database.SaveResult[] results = Database.update(records, false, AccessLevel.USER_MODE);

for (Integer i = 0; i < results.size(); i++) {
    if (!results[i].isSuccess()) {
        Database.Error error = results[i].getErrors()[0];
        System.debug(LoggingLevel.ERROR, 'Error: ' + error.getMessage());
    }
}
```

### Schedulable Delegation
```apex
public void execute(SchedulableContext sc) {
    try {
        MyBatch batch = new MyBatch();
        Id batchId = Database.executeBatch(batch, 200);
        System.debug('Batch job enqueued: ' + batchId);
    } catch (Exception e) {
        System.debug(LoggingLevel.ERROR, 'Error: ' + e.getMessage());
    }
}
```

## Cron Expressions

Common patterns included in Schedulable classes:

| Schedule | Cron Expression |
|----------|----------------|
| Daily at 2 AM | `'0 0 2 * * ?'` |
| Every Monday at 9 AM | `'0 0 9 ? * MON'` |
| First day of month | `'0 0 0 1 * ?'` |
| Every 15 minutes | `'0 0/15 * * * ?'` |
| Weekdays at 8 AM | `'0 0 8 ? * MON-FRI'` |
| Last day of month | `'0 0 23 L * ?'` |

Format: `Seconds Minutes Hours Day_of_month Month Day_of_week [Optional_year]`

## Governor Limits

### Queueable
- Max 50 per transaction
- 1 enqueue per execute (chaining allowed)
- Shares limits with synchronous code

### Batch
- Max 5 concurrent batch jobs per org
- Max 100 batch jobs queued/in-progress
- Each batch gets fresh governor limits
- Max 50M records processable

### Schedulable
- Max 100 scheduled jobs per org
- Lightweight wrapper (delegates to Batch/Queueable)

## Best Practices

### Always Include
1. ✅ Chain depth limiting (Queueable)
2. ✅ Finalizer pattern (Queueable)
3. ✅ Partial success DML (Batch)
4. ✅ Job statistics logging (Batch finish)
5. ✅ Error handling and logging
6. ✅ USER_MODE or AccessLevel.USER_MODE
7. ✅ Test.startTest()/stopTest() in tests
8. ✅ AsyncApexJob queries in tests
9. ✅ Bulk test data (250+ records for Batch)
10. ✅ Claude Code attribution

### Never Include
1. ❌ SOQL/DML in loops
2. ❌ Infinite chaining without depth check
3. ❌ Hardcoded IDs
4. ❌ @future methods (use Queueable instead)
5. ❌ Missing Test.stopTest()
6. ❌ Tests with insufficient data volume

## Troubleshooting

### Issue: "Too many queueable jobs added to the queue"
- **Solution**: Use Batch instead, or increase chunk size per Queueable

### Issue: "Maximum stack depth reached"
- **Solution**: Implement MAX_CHAIN_DEPTH limiting

### Issue: Batch running too slowly
- **Solution**: Increase batch size, optimize SOQL, remove unnecessary operations

### Issue: "Invalid cron expression"
- **Solution**: Use provided cron examples, validate format

### Issue: Tests not completing
- **Solution**: Ensure Test.stopTest() is present

## Attribution

All generated classes include:
- **Header**: "CREATED BY: Claude Code"
- **Comment**: "Generated by Claude Code on [YYYY-MM-DD]"

## Version History

- **v1.0.0** (2026-05-05): Initial release
  - Queueable with Finalizer
  - Batch with partial success
  - Schedulable with cron examples
  - Comprehensive test coverage
  - Claude Code attribution

## Related Skills

- **sf-apex-create**: Base Apex class creation (includes basic Batch/Queueable templates)
- **sf-prefix-detect**: Project prefix detection
- **sf-trigger-create**: Trigger creation (often calls async jobs)
- **sf-validate-all**: Validates all Salesforce components

## License

MIT License - Claude Code

---

**Skill**: sf-apex-async  
**Version**: 1.0.0  
**Author**: Claude Code  
**Last Updated**: 2026-05-05
