# Implement Greetly Location Mapping

## Description
Implement the mapping functionality for Greetly Locations to OSS sites, including mapping and unmapping events.

## Acceptance Criteria

### Location Mapping
```gherkin
Given a greetlyLocationMapped event
When it is processed
Then the ossSiteId should be updated
And the greetlyLocationId should be preserved
```

### Location Unmapping
```gherkin
Given a greetlyLocationUnmapped event
When it is processed
Then the ossSiteId should be cleared
And the greetlyLocationId should be preserved
```

## Technical Notes
- Implement mapping state transitions
- Add validation for mapping events
- Ensure proper state preservation during mapping changes
- Add logging for mapping operations 