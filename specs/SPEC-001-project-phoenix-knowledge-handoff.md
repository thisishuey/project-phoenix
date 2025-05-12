# SPEC-001: Project Phoenix Knowledge Handoff for Agentic Resources

## Purpose of This Document

This document serves as a comprehensive knowledge transfer resource for agentic tools and AI assistants that may be involved in maintaining, updating, or extending the Project Phoenix documentation. It summarizes key information, context, technical details, and important nuances gathered during the creation of the Technical Requirements Document (TRD) and supporting materials.

## Project Overview

### Core Project Information

**Project Name**: Project Phoenix  
**Project Purpose**: Replace Flink StateFun components in the Ambrosia event streaming solution with Google Cloud Dataflow  
**Project Timeframe**: 4 months with a lean team (2 developers + agentic tools)  
**Project Context**: Phase 1 of a larger Data Fabric migration strategy (4 phases total)

### Project Goals

1. Replace Ambrosia's Flink StateFun components with something more developer-friendly and stable
2. Reduce development time to get data into BigQuery
3. Maintain simple architecture while improving operational stability
4. Establish foundation for subsequent phases of the Data Fabric migration

### Pain Points Being Addressed

1. StateFun pods not scaling properly
2. Limited expertise in Flink StateFun and poor documentation
3. Verbose approach when adding new events
4. Limited visibility into event processing
5. Production access required for debugging
6. Fear of "Fabrications" when things go wrong
7. Complex infrastructure that's difficult to maintain and debug
8. StateFun actor state exceeding message size limits

## Technical Architecture

### Current System (Ambrosia)

Ambrosia is a stateful event processing framework built on Apache Flink StateFun that:
- Ingests events from various sources via Kafka
- Processes them through entity-specific functions
- Maintains state for different entities
- Projects results to BigQuery

Key components include:
- StateFun Framework (core processing engine)
- Entity Functions (desk, employee, site, floor, etc.)
- Middleware Pipeline (for cross-cutting concerns)
- BigQuery Integration (data projection)

### Proposed Solutions (Three Paths)

**Path 1: Direct Streaming Pipeline**
- Flow: Kafka → Dataflow → BigQuery
- Simplest architecture, fully managed by GCP
- Limitations in handling complex temporal data

**Path 2: Enrichment-Based Pipeline**
- Flow: Kafka → Dataflow → XTDB → BigQuery
- Leverages XTDB for temporal processing
- More complex but better for temporal data

**Path 3: Dual-Pipeline Approach**
- Split flow for real-time vs. historical data
- Most complex architecture
- Best for clearly separated processing needs

The current preference is to start with Path 1 unless technical constraints are identified during prototyping.

### Key Technology Choices

1. **Google Cloud Dataflow**: Primary processing engine, replacing StateFun
2. **Apache Kafka**: Continued use for event streaming
3. **Google BigQuery**: Continued use for data storage
4. **XTDB**: Optional component for temporal data handling (decision pending)

## Entity Information

### BigQuery Tables and Entity Types

The system processes and maintains 9 primary entity types with corresponding BigQuery tables:

1. **Desk_Bookings_Table**: Tracks desk reservations with check-in/out times
2. **Desk_Moves_Table**: Records desk relocations and assignments
3. **Desk_Shifts_Table**: Manages desk availability patterns
4. **Desks_Table_v2**: Main desk entity information (complex with nested structures)
5. **Employees_Table**: Personnel data including assignments
6. **Greetly_Visits_Table**: Visitor management information
7. **Neighborhoods_Table**: Logical groupings of desks (simpler structure)
8. **Presence_Table**: Employee location tracking (complex temporal aspects)
9. **Sites_Table**: Physical location information

### Entity Migration Order

Current prioritization for migration (simplest to most complex):
1. Neighborhoods (simpler schema, fewer dependencies)
2. Sites (foundation for other entities)
3. Employees (moderate complexity)
4. Floors (depends on Sites)
5. Greetly Visits (external integration)
6. Desk Shifts (specialized temporal aspects)
7. Desk Moves (depends on Desks)
8. Presence (complex temporal requirements)
9. Desks (complex with nested structures)

### Entity Relationships

- Sites contain Floors
- Floors contain Neighborhoods and Desks
- Neighborhoods group Desks
- Employees are assigned to Sites and Desks
- Desk Bookings, Moves, and Shifts all reference Desks
- Presence records reference Employees and Sites
- Greetly Visits reference Employees and Sites

## Project Strategy and Approach

### Migration Strategy

- Entity-by-entity migration (not a complete replacement at once)
- Starting with simpler entities to establish patterns
- Parallel processing during transition periods
- Comprehensive validation for each entity
- Template-based approach to accelerate implementation

### Implementation Phases (Full Data Fabric Strategy)

1. **Phase 1: Project Phoenix** - Replace StateFun (current phase)
2. **Phase 2: Streamline Data Migration** - Enhance pipeline for new data
3. **Phase 3: GraphQL API Integration** - Enable external integrations
4. **Phase 4: Apigee Replacement** - Unify API layers

### Development Methodology

- Lean team (2 developers)
- Heavy use of agentic tools for acceleration
- Template-based development with reusable patterns
- Automated testing and validation
- Continuous integration and deployment

## Temporal Processing Requirements

### Key Temporal Challenges

- Processing out-of-order events (up to a year old)
- Maintaining historical views of entity state
- Handling corrections to previous data
- Ensuring time-based queries reflect correct entity relationships
- Supporting both retroactive (past) and proactive (future) events

### Use Cases with Temporal Requirements

1. **Old Presence Events**: Events uploaded days/weeks later must associate with correct reference data as of that time
2. **Future Desk Bookings**: Bookings made in advance must reference correct desk location at booking time
3. **Desk Booking Series**: Repeating bookings must handle entity changes over time
4. **PII Handling**: Anonymization requirements for employee data while maintaining history

## Operational Considerations

### Monitoring Requirements

- Tracing messages through the system by UUID
- Component status dashboards with alerting
- Metric counts with instance/tenant ID tracking
- Prometheus integration for metrics
- Enhanced visibility compared to current system

### Multi-tenancy Requirements

- Support for approximately 1000 client organizations
- Data isolation in BigQuery (though not during processing)
- Tenant-specific monitoring capabilities
- Performance isolation between tenants

### Disaster Recovery Needs

- Event replay capabilities from Kafka
- Proper error handling and dead-letter queues
- Data consistency validation mechanisms
- Automated recovery procedures

## Documentation Structure

The Project Phoenix documentation repository includes:

1. **Technical Requirements Document (TRD)**: Primary reference document
2. **Project Charter**: High-level overview and business case
3. **Architecture Decision Records**: Documentation of key technical decisions
4. **Technical Proof of Concept Plan**: Specifications for validating key capabilities
5. **Entity Migration Template**: Standardized approach for entity migration
6. **Risk Register**: Assessment of project risks and mitigations
7. **Data Flow Diagrams**: Visual representations of architecture
8. **Operational Runbook Template**: Framework for operational procedures
9. **Phase Integration Plan**: How Project Phoenix connects to later phases

## Important Nuances and Context

### Technical Nuances

1. **State Management Complexities**: Different entity types have different state management needs; some use snapshots while others maintain full history.

2. **Temporal Processing Subtleties**: Handling of event time vs. processing time is crucial for data accuracy, especially for presence data.

3. **Scale Variation**: Entity volumes vary widely - some entities like Sites are fewer but complex, while others like Presence events are high-volume.

4. **Fabrications Context**: "Fabrications" refer to recreating data from scratch, a problematic but sometimes necessary process in the current system.

### Organizational Context

1. **Team Composition Reality**: Limited to just two developers with agentic tools, not a large team.

2. **Operational Concerns**: Operations team has specific requirements for visibility, monitoring, and troubleshooting capabilities.

3. **Client Base Scale**: System supports approximately 1000 client organizations with varying needs and data volumes.

4. **Past Challenges**: Previous issues with StateFun have created organizational caution about complex architectural choices.

### Documentation Maintenance Guidelines

1. **TRD Updates**: As the project progresses, the TRD should be updated to reflect actual implementation decisions, especially after the prototyping phase.

2. **Terminology Consistency**: Maintain consistent terminology across documents, particularly around entity names and technical component references.

3. **Architecture Decision Documentation**: Any significant deviation from the planned architecture should be captured in a new Architecture Decision Record.

4. **Timeline Adjustments**: The project timeline is aggressive for a two-person team; timeline sections may need adjustment as the project progresses.

5. **Component Documentation Prioritization**: Focus documentation efforts on custom-built components rather than managed services like Dataflow that have extensive external documentation.

## References and Resources

### Key Documentation Sources

1. BigQuery Tables and Schema Documentation (source/bigquery-tables-and-schema-20250508.md)
2. Ambrosia System Architecture Summary (source/ambrosia-system-arcitecture-summary.md)
3. Luna Ambrosia Feedback (source/luna-ambrosia-feedback.md)
4. Huey Notes (source/huey-notes.md)
5. GoBots Project Phoenix Notes (source/gobots-project-phoenix-notes.md)

### Technology Documentation

1. [Google Cloud Dataflow Documentation](https://cloud.google.com/dataflow/docs)
2. [Apache Beam Documentation](https://beam.apache.org/documentation/)
3. [XTDB Documentation](https://docs.xtdb.com/) (if used)
4. [BigQuery Documentation](https://cloud.google.com/bigquery/docs)

### Repository Structure

```
project-phoenix-docs/
├── README.md
├── specs/
│   └── SPEC-001-project-phoenix-knowledge-handoff.md
├── trd/
│   └── project-phoenix-trd.md
├── charter/
│   └── project-phoenix-charter.md
├── architecture/
│   ├── adr-template.md
│   ├── adr-001-statefun-replacement.md
│   └── data-flow-diagrams.md
├── planning/
│   ├── technical-poc-plan.md
│   ├── entity-migration-template.md
│   ├── risk-register.md
│   └── phase-integration-plan.md
├── operations/
│   └── operational-runbook-template.md
└── source/
    ├── bigquery-tables-and-schema-20250508.md
    ├── ambrosia-system-arcitecture-summary.md
    ├── luna-ambrosia-feedback.md
    ├── huey-notes.md
    └── gobots-project-phoenix-notes.md
```

## Conclusion

This knowledge handoff document captures the essential context, information, and nuances gathered during the creation of the Project Phoenix documentation. It should serve as a foundational resource for any agentic tool or AI assistant that needs to maintain, update, or extend this documentation in the future. When working with these documents, it's important to understand not just the technical details but also the broader context of Project Phoenix as Phase 1 of a larger Data Fabric migration strategy.

## Document Information

| Attribute | Value |
|-----------|-------|
| Document Number | SPEC-001 |
| Title | Project Phoenix Knowledge Handoff for Agentic Resources |
| Author | [Author Name] |
| Created Date | 2025-05-11 |
| Last Updated | 2025-05-11 |
| Status | Draft |
| Revision | 1.0 |
