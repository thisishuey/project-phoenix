# Architecture Decision Record: StateFun Replacement Technology

## ADR-001: Selection of Google Cloud Dataflow as StateFun Replacement

**Date**: 2025-05-11  
**Status**: Proposed  
**Supersedes**: N/A  
**Superseded by**: N/A

## Context

OfficeSpace Software's Ambrosia event streaming solution currently uses Apache Flink StateFun for processing event streams from Kafka and projecting data to BigQuery. This component has presented several challenges:

1. **Developer Experience**: High learning curve, limited documentation, and specialized knowledge requirements make it difficult to onboard new developers.

2. **Operational Stability**: StateFun pods experience scaling issues, leading to system instability and performance degradation.

3. **Complexity**: The current implementation is difficult to maintain, debug, and extend, making it time-consuming to add new data types or modify existing ones.

4. **Visibility**: There is limited visibility into event processing, making it challenging to trace messages through the system and troubleshoot issues.

5. **Operational Access**: Debugging often requires production access, creating operational and security concerns.

The Project Phoenix initiative aims to replace StateFun with a more developer-friendly, stable, and efficient solution while maintaining existing data flows and output structures.

## Decision

We will replace Apache Flink StateFun with Google Cloud Dataflow as the primary stream processing technology for Project Phoenix.

The implementation will follow one of three potential paths:

1. **Direct Streaming Pipeline**: Kafka → Dataflow → BigQuery
2. **Enrichment-Based Pipeline**: Kafka → Dataflow → XTDB → BigQuery
3. **Dual-Pipeline Approach**: 
   - Real-time: Kafka → Dataflow → BigQuery
   - Historical: Kafka → XTDB → Dataflow → BigQuery

Initial prototyping will evaluate these approaches, with a current preference for Path 1 (Direct Streaming) unless technical constraints are identified.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **Google Cloud Dataflow** | • Fully managed service<br>• Built on Apache Beam with unified batch/streaming model<br>• Strong integration with GCP ecosystem<br>• Auto-scaling capabilities<br>• Comprehensive monitoring<br>• Potential for templates to accelerate development | • Potential cost concerns<br>• Vendor lock-in<br>• Limited control over infrastructure<br>• Handling of extended time-windows may require additional solutions |
| **Apache Beam (self-hosted)** | • Same programming model as Dataflow<br>• Greater flexibility in deployment<br>• Potentially lower cost<br>• No vendor lock-in | • Operational overhead of self-hosting<br>• Requires infrastructure management<br>• No managed auto-scaling | 
| **Apache Kafka Streams** | • Tightly integrated with Kafka<br>• Simpler programming model<br>• Lighter weight than Flink<br>• Event-time processing support | • More limited feature set<br>• Less scalable for complex processing<br>• Would require additional operational knowledge<br>• More limited ecosystem |
| **Alternative Flink Implementation** | • Reuse existing knowledge<br>• Proven capability for our use cases<br>• Would require less relearning | • Wouldn't address fundamental issues with Flink<br>• Complexity would remain<br>• Learning curve for new developers still high |

## Consequences

### Positive

- Simplified development experience with more mainstream technology
- Reduced operational burden through managed service
- Improved monitoring and visibility
- Better scalability and performance stability
- Faster development cycles for new features
- Better documentation and community support

### Negative

- Cost may be higher for Dataflow vs. self-hosted alternatives
- Migration effort required for all entity types
- New technology learning curve (though expected to be lower than StateFun)
- Potential vendor lock-in to Google Cloud Platform

### Neutral

- Will require updating operational processes and monitoring
- Development teams will need training on Dataflow/Beam
- New architectural patterns will emerge that differ from the current system

## Compliance Requirements

- Must maintain GDPR compliance for personal data processing
- Must preserve audit trail capabilities for regulated clients
- Must maintain existing security controls and data access patterns

## Implementation

The implementation will follow an entity-by-entity migration approach:

1. Select a pilot entity for initial implementation
2. Develop prototypes to validate the approach
3. Implement the full solution for the pilot entity
4. Gradually migrate remaining entities
5. Run parallel processing during transition
6. Decommission StateFun components

## Related Decisions

- Selection of specific Dataflow architecture pattern
- Decision on XTDB usage for temporal data handling
- Entity migration prioritization
- Monitoring and observability approach

## References

- [Google Cloud Dataflow Documentation](https://cloud.google.com/dataflow/docs)
- Ambrosia System Architecture Summary
- Luna Ambrosia Feedback documentation
- Current BigQuery schema documentation
