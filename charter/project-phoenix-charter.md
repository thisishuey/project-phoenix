# Project Phoenix Charter

## Project Overview

Project Phoenix is Phase One of OfficeSpace Software's comprehensive Data Fabric migration initiative. The project focuses on replacing the Flink StateFun components in the current Ambrosia event streaming solution with a more developer-friendly, stable, and efficient alternative based on Google Cloud Dataflow.

## Business Case

The current Ambrosia implementation using Flink StateFun has presented several challenges:

- Steep learning curve for developers leading to slow onboarding
- Stability issues with StateFun pods not scaling properly
- Complex infrastructure that is difficult to maintain and debug
- Development bottlenecks when adding new data types or modifying existing ones
- Limited visibility into event processing and outcomes
- Operational burden requiring production access for debugging

Project Phoenix addresses these challenges by implementing a more accessible and stable solution, enabling faster data migration to BigQuery and reducing the technical debt associated with the current implementation.

## Project Objectives

1. Replace Flink StateFun components with Google Cloud Dataflow while maintaining existing data flows
2. Reduce development time and complexity for moving data to BigQuery
3. Improve system stability and operational visibility
4. Create a solution that supports ~1000 clients with appropriate multi-tenancy
5. Establish the foundation for subsequent phases of the Data Fabric migration

## Success Criteria

1. **Development Simplicity**: Significantly reduced developer onboarding time and complexity
2. **Operational Stability**: Improved uptime and reduced operational incidents
3. **Migration Timeline**: Complete migration of all entity types within the established timeframe
4. **Performance**: Meet or exceed the current system's performance characteristics
5. **Forward Compatibility**: Successfully establish technical foundations for Phase 2 of the Data Fabric project

## Scope

### In Scope

- Replacement of Flink StateFun components in the Ambrosia architecture
- Migration of all 9 existing entity types to the new solution
- Development of monitoring and operational tools
- Implementation of proper error handling and reprocessing capabilities
- Documentation and knowledge transfer

### Out of Scope

- Changes to the upstream Kafka message format
- Modifications to the BigQuery schema beyond what's necessary for the migration
- Implementation of Phase 2, 3, or 4 capabilities (though foundations will be established)
- Refactoring of client applications that consume BigQuery data

## Project Approach

Project Phoenix will follow an entity-by-entity migration strategy, starting with simpler entities to validate the approach before moving to more complex ones. The project will include:

1. **Analysis & Design**: Evaluate technical options and define the target architecture
2. **Prototyping**: Build proofs of concept to validate key capabilities
3. **Development**: Implement the solution starting with a single entity type
4. **Incremental Migration**: Gradually move entity types from the old to the new system
5. **Validation**: Ensure data consistency and performance throughout the migration
6. **Complete Transition**: Decommission the old StateFun components

## Key Stakeholders

- **Product Management**: Chris, Borna, Jeremy, Kiley
- **Engineering**: [Engineering Lead Names]
- **Operations**: [Operations Team Representative]
- **Quality Engineering**: [QE Representative]
- **Security**: [Security Team Representative]
- **Executive Sponsor**: [Executive Name]

## Timeline Milestones

1. **Sprint 10**: Feature Planning and TRD Completion
2. **Sprint 11-12**: Technical Prototyping
3. **Sprint 13-14**: First Entity Implementation
4. **Sprint 15-20**: Incremental Entity Migration
5. **Sprint 21**: Complete Transition and Validation
6. **Sprint 22**: Project Close and Handover to Operations

## Constraints and Assumptions

### Constraints

- Must maintain compatibility with existing upstream and downstream systems
- Must support ~1000 clients with appropriate multi-tenancy
- Must meet industry-standard SLAs for data freshness and processing latency

### Assumptions

- Google Cloud Dataflow will provide adequate performance and cost profile
- The entity-by-entity migration approach will allow for controlled risk management
- Existing technical documentation is sufficient for understanding current entity processing

## Risks and Mitigation Strategies

| Risk | Impact | Probability | Mitigation |
|------|--------|------------|------------|
| Dataflow cost exceeds expectations | High | Medium | Perform early cost modeling; prepare fallback to alternative Beam runners if needed |
| Complex temporal logic proves difficult to implement | High | Medium | Early prototyping of most complex use cases; consider XTDB for supplementary temporal processing |
| Migration impacts production stability | High | Low | Robust testing, gradual migration, monitoring, and rollback procedures |
| Resource constraints delay project | Medium | Medium | Clear prioritization; potential adjustment of scope or timeline |
| Unforeseen technical complexities | Medium | Medium | Technical spikes early in the project; contingency buffer in timeline |

## Project Team

- **Project Manager**: [Name]
- **Technical Lead**: [Name]
- **Senior Developers**: [Names]
- **QA Engineers**: [Names]
- **Operations Support**: [Names]

## Approval

| Name | Role | Signature | Date |
|------|------|-----------|------|
|      |      |           |      |
|      |      |           |      |
|      |      |           |      |

