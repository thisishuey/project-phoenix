# Implement Greetly Location Lifecycle

## Description
Implement core lifecycle management for Greetly kiosk locations in the visitor management system.

## Acceptance Criteria

### Location Creation
```gherkin
Given a location creation event
When it is processed by the event pipeline
Then the location record should be properly stored
And the state should be updated
And the event should be processed in order
```

### Location Import
```gherkin
Given a location import event
When it is processed by the event pipeline
Then the location record should be properly stored
And the state should be updated
And the event should be processed in order
```

### Location Deletion
```gherkin
Given a location deletion event
When it is processed by the event pipeline
Then the location record should be marked as deleted
And the state should be updated
And the event should be processed in order
```

### Error Handling
```gherkin
Given an invalid lifecycle event
When it is processed by the event pipeline
Then the error should be logged
And the pipeline should continue processing other events
```

## Technical Notes
- Configure event pipeline to:
  - Process location lifecycle events
  - Support location creation and import
  - Handle location deletion
  - Enable monitoring and observability
  - Maintain data quality and validation 