# Implement Greetly Location Visitor Events

## Description
Implement visitor event handling for Greetly Locations, including check-in, preregistration, and check-out events.

## Acceptance Criteria

### Visitor Check-in
```gherkin
Given a greetlyVisitorCheckedIn event
When it is processed
Then the Visit function should be subscribed
And location history should be forwarded
```

### Visitor Check-out
```gherkin
Given a greetlyVisitorCheckedOut event
When it is processed
Then the Visit function should be unsubscribed
```

## Technical Notes
- Implement visitor event processing
- Set up Visit function subscription logic
- Add location history forwarding
- Implement unsubscription on check-out 