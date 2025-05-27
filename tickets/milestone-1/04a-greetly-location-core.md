# Implement Core Greetly Location Entity

## Description
Implement the basic Greetly Location entity with core state management and event processing capabilities.

## Acceptance Criteria
```gherkin
Given a greetlyLocationCreated event
When it is processed
Then a new location state should be created
And the greetlyLocationId should be set
And ossSiteId should be null
```

## Technical Notes
- Implement basic state structure (greetlyLocationId, ossSiteId)
- Set up event sourcing pattern
- Implement chronological event processing
- Add basic logging and error handling 