# Implement Greetly Site Configuration

## Description
Implement Greetly Site Configuration functionality for managing site-level privacy and configuration settings in the visitor management system.

## Acceptance Criteria

### Configuration Management
```gherkin
Given a siteConfigUpdated event
When it is processed by the event pipeline
Then the configuration should be properly stored
And the state should be updated
And the event should be processed in order
```

### State Management
```gherkin
Given multiple configuration events for the same site
When they are processed by the event pipeline
Then later events should override earlier ones
And the state should reflect the most recent configuration
And out-of-order events should be handled correctly
```

### Visit Integration
```gherkin
Given a visitor check-in or preregistration event
When it is processed by the event pipeline
Then the current site configuration should be forwarded to the Visit function
And the message should have typename com.officespacesoftware/visit
```

### Error Handling
```gherkin
Given an invalid configuration event
When it is processed by the event pipeline
Then the error should be logged
And the pipeline should continue processing other events
```

## Technical Notes
- Configure event pipeline to:
  - Process siteConfigUpdated events
  - Handle state management integration
  - Manage Visit function integration
  - Support error handling and logging
  - Enable monitoring and observability 