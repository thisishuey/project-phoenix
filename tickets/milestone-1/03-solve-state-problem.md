# Solve State Management Problem

## Description
Investigate and implement a solution for state management in the Dataflow pipeline, evaluating options including XTDB, raw events in BigQuery, Materialize, and Postgres with time extensions.

## Acceptance Criteria
```gherkin
Given the need to maintain state for the Beam pipeline
When I implement the state management solution
Then it should handle out-of-order events correctly
And it should maintain data consistency
And it should support historical data processing
```

## Technical Notes
- Review POC details:
  - Review XTDB details
  - Evaluate raw events storage in BigQuery
  - Investigate Materialize for real-time data integration
  - Research Postgres with time extensions
- Decide and implement the solution we will move forward with