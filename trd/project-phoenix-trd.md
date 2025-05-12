# Technical Requirements Document (TRD) - Project Phoenix

**Document Version:** 1.0  
**Date:** 2025-05-11  
**Status:** Draft  
**Author:** Jeff "Huey" Huelsbeck
**Last Updated:** 2025-05-11

## 1. Executive Summary

Project Phoenix represents Phase 1 of OfficeSpace Software's comprehensive Data Fabric migration strategy. The project focuses on replacing the Apache Flink StateFun components in our current Ambrosia event streaming solution with a more developer-friendly, stable, and efficient alternative based on Google Cloud Dataflow.

The current Ambrosia implementation has presented several challenges:
- Steep learning curve for developers and complex onboarding
- Stability issues with StateFun pods not scaling properly
- Complex infrastructure that is difficult to maintain and debug
- Development bottlenecks when adding new data types or modifying existing ones
- Limited visibility into event processing and outcomes
- Frequent need for production access to debug issues

Project Phoenix will address these challenges by implementing a more accessible and stable solution, enabling faster data migration to BigQuery, and reducing the technical debt associated with the current implementation.

The migration will follow an entity-by-entity approach, starting with simpler entities to validate the architecture before moving to more complex ones. The project will establish the foundation for subsequent phases of the Data Fabric migration strategy:

- **Phase 1: Project Phoenix** - Replace StateFun component with Dataflow-based solution
- **Phase 2: Streamline Data Migration** - Enhance and optimize data fabric pipeline for new data
- **Phase 3: GraphQL API Integration** - Enable client and third-party integrations for event processing
- **Phase 4: Apigee Replacement** - Unify API layers into federated GraphQL API

## 2. Introduction

### 2.1 Purpose and Scope

This Technical Requirements Document (TRD) defines the technical requirements and architecture for Project Phoenix. It serves as the primary technical reference for the project team, stakeholders, and future maintainers of the system.

The scope of Project Phoenix (Phase 1) includes:
- Replacement of Flink StateFun components in the Ambrosia architecture
- Migration of all 9 existing entity types to the new solution
- Development of monitoring and operational tools
- Implementation of proper error handling and reprocessing capabilities
- Documentation and knowledge transfer

Out of scope for Project Phoenix:
- Changes to the upstream Kafka message format
- Modifications to the BigQuery schema beyond what's necessary for the migration
- Implementation of Phase 2, 3, or 4 capabilities (though foundations will be established)
- Refactoring of client applications that consume BigQuery data

### 2.2 Data Fabric Migration Strategy

Project Phoenix is the first phase of a comprehensive Data Fabric migration strategy consisting of four phases:

#### Phase 1: Project Phoenix
- Replace Flink StateFun component with Dataflow-based solution
- Focus on 1:1 migration of existing functionality
- Establish technical foundation for future phases
- Timeline: [Start Date] to [End Date]

#### Phase 2: Streamline Data Migration
- Optimize and enhance data pipeline for new data types
- Improve developer experience for adding new entities
- Build on Project Phoenix architecture
- Tentative Timeline: [Start Date] to [End Date]

#### Phase 3: GraphQL API Integration
- Enable external systems to trigger events
- Integrate presence events with reporting
- Establish GraphQL as primary integration point
- Tentative Timeline: [Start Date] to [End Date]

#### Phase 4: Apigee Replacement
- Unify all API layers into a federated GraphQL API
- Fully replace Apigee functionality
- Complete the Data Fabric vision
- Tentative Timeline: [Start Date] to [End Date]

### 2.3 Stakeholders and References

#### Key Stakeholders

- **Product Management**: Chris, Borna, Jeremy, Kiley
- **Engineering**: [Engineering Lead Names]
- **Operations**: [Operations Team Representative]
- **Quality Engineering**: [QE Representative]
- **Security**: [Security Team Representative]
- **Executive Sponsor**: [Executive Name]

#### Reference Documents

- [BigQuery Tables and Schemas](source/bigquery-tables-and-schema-20250508.md)
- [Ambrosia System Architecture Summary](source/ambrosia-system-arcitecture-summary.md)
- [Luna Ambrosia Feedback](source/luna-ambrosia-feedback.md)
- [Huey Notes](source/huey-notes.md)
- [GoBots Project Phoenix Notes](source/gobots-project-phoenix-notes.md)

## 3. Current System Analysis

### 3.1 Ambrosia Architecture Overview

Ambrosia is a stateful event processing framework built on Apache Flink StateFun designed to consume events from various sources (primarily Kafka topics), process them through entity-specific functions, maintain state for different entities, and write data to BigQuery for analysis and reporting.

The main components include:

1. **Sources**: Generating events from Huddle application, external system integrations, async imports, and fabrications.

2. **Kafka**: Serving as the message broker for event streaming with topics organized by source.

3. **Flink StateFun Framework**: The underlying Apache Flink StateFun framework that manages stateful function execution.

4. **Entity Functions**: Stateful functions that process events for specific entity types (desk, employee, site, floor, room, neighborhood, visit, etc.).

5. **Middleware Pipeline**: A series of middleware functions applied to each stateful function for cross-cutting concerns like exception handling, logging, and history management.

6. **BigQuery Integration**: Writing processed data to BigQuery tables for analysis.

Events flow through the Ambrosia system as follows:
1. Events are consumed from Kafka topics by the Preparation and Visitor Service Preparation functions
2. These functions enrich the events with metadata and route them to the appropriate entity functions
3. Entity functions process the events, update their state, and write data to BigQuery

### 3.2 StateFun Component Analysis

The StateFun component currently serves as the core processing engine for Ambrosia:

- **State Management**: Uses a history-based approach where each entity function maintains a history of events that have affected it, and for some entities, periodic snapshots of state.

- **Entity Functions**: Processes events for specific entity types:
  - **Desks**: Manages desk-related events and state
  - **Employees**: Manages employee-related events and state
  - **Sites**: Manages site-related events and state
  - **Floors**: Manages floor-related events and state
  - **Rooms**: Manages room-related events and state
  - **Neighborhoods**: Manages neighborhood-related events and state
  - **Visits**: Manages visit-related events and state
  - **Greetly Locations**: Manages location-related events and state

- **Middleware**: Provides cross-cutting concerns like exception handling, logging, and history management.

### 3.3 Pain Points and Limitations

Based on feedback and analysis, the current StateFun implementation has several significant pain points:

- **Scalability Issues**: StateFun pods are not scaling properly, leading to performance bottlenecks.

- **Technological Complexity**: The technology is not popular, poorly documented, and requires specialized knowledge.

- **Development Overhead**: Adding new events requires a verbose approach, slowing down development.

- **Limited Visibility**: There is no clear visibility into what events are coming in and what was the result of processing.

- **Operational Challenges**: Debugging requires production access, and there's no built-in replay functionality.

- **Data Management Issues**: The fear of "Fabrications" - if things go wrong, production needs to be re-fabricated.

- **Technical Limitations**: StateFun actor state exceeds the message size limit for some entities, requiring complex workarounds.

- **Knowledge Gap**: Limited expertise in the technology and lack of proper education for new engineers.

### 3.4 Data Flow and Processing Model

The current data flow in Ambrosia involves:

1. **Event Ingestion**: Events are published to Kafka topics from various sources.

2. **Event Processing**: StateFun functions consume these events, maintain state, and process them according to business logic.

3. **State Management**: Entity state is maintained using either:
   - History without snapshots (for Desk, Neighborhood, Visit, Greetly Location)
   - History with snapshots (for Employee, Floor, Room)
   - Distribution middleware (for Site)

4. **BigQuery Projection**: Processed data is written to BigQuery tables corresponding to each entity type.

5. **Event Time Processing**: The system needs to handle events that arrive out of order, potentially by days, weeks, or months.

### 3.5 Current Operational Model

The current operational approach for Ambrosia has several challenges:

- **Debugging**: Requires production access to view logs and diagnose issues.

- **Monitoring**: Limited visibility into event processing status and outcomes.

- **Scaling**: Manual intervention often required for scaling issues.

- **Error Handling**: Lacks robust error handling and recovery mechanisms.

- **Disaster Recovery**: No built-in capabilities for event replay or disaster recovery.

## 4. Requirements

### 4.1 Functional Requirements

#### 4.1.1 Event Streaming Capabilities

- **F1**: The system must consume events from Kafka topics, maintaining compatibility with existing event formats.
- **F2**: The system must support both real-time event processing and asynchronous processing for historical data.
- **F3**: The system must maintain the ability to process events for all 9 current entity types.
- **F4**: The system must support event replay capabilities for recovery and reprocessing scenarios.
- **F5**: The system must provide mechanisms to handle both individual events and bulk events.

#### 4.1.2 Temporal Data Processing

- **F6**: The system must accurately handle out-of-order events based on their valid time rather than processing time.
- **F7**: The system must support events with effective dates in the past (retroactive) and future (proactive).
- **F8**: The system must correctly process historical data loads that may span weeks, months, or up to a year.
- **F9**: The system must maintain a consistent view of entity state as it was at any point in time.
- **F10**: The system must handle corrections to previously processed events while maintaining data integrity.

#### 4.1.3 Entity-Specific Processing Requirements

- **F11**: The system must process and maintain state for Desk entities, including bookings, availability, and assignments.
- **F12**: The system must process and maintain state for Employee entities, including department and location assignments.
- **F13**: The system must process and maintain state for Site entities, handling the complexity that previously required distribution middleware.
- **F14**: The system must process and maintain state for Floor entities, including online status and reference data.
- **F15**: The system must process and maintain state for Room entities, including attributes and availability.
- **F16**: The system must process and maintain state for Neighborhood entities, including grouping and assignment.
- **F17**: The system must process and maintain state for Visit entities, including check-in and check-out events.
- **F18**: The system must process and maintain state for Greetly Location entities, including mapping and configuration.
- **F19**: The system must handle entity relationships and references between different entity types.

#### 4.1.4 Data Enrichment and Transformation

- **F20**: The system must support enrichment of events with reference data from other entity types.
- **F21**: The system must transform raw events into the format required for BigQuery storage.
- **F22**: The system must handle data type conversions and formatting appropriate for each field.
- **F23**: The system must validate event data against expected schemas and handle validation failures gracefully.
- **F24**: The system must support custom transformation logic for each entity type.

#### 4.1.5 BigQuery Data Projection

- **F25**: The system must write processed data to the existing BigQuery tables without schema changes.
- **F26**: The system must maintain all required fields and relationships defined in the current schema.
- **F27**: The system must ensure data consistency and integrity when writing to BigQuery.
- **F28**: The system must handle BigQuery write failures gracefully with appropriate retry mechanisms.
- **F29**: The system must optimize writes to BigQuery to minimize cost and maximize performance.

### 4.2 Non-Functional Requirements

#### 4.2.1 Performance Metrics and SLAs

- **NF1**: The system must process real-time events with end-to-end latency of no more than 5 seconds under normal conditions.
- **NF2**: The system must process asynchronous events at a rate of at least 1000 events per second.
- **NF3**: The system must achieve 99.9% uptime for the event processing pipeline.
- **NF4**: The system must handle peak loads of at least 5x the average throughput without degradation.
- **NF5**: The system must maintain performance SLAs even during regular maintenance operations.

#### 4.2.2 Scalability Requirements

- **NF6**: The system must automatically scale resources based on throughput requirements.
- **NF7**: The system must support processing for approximately 1000 distinct client organizations.
- **NF8**: The system must handle growth in data volume and throughput of 20% year-over-year.
- **NF9**: The system must support entity scaling without manual intervention, addressing current limitations with large entities like Sites.
- **NF10**: The system must scale efficiently for both continuous processing and burst loads (such as large imports).

#### 4.2.3 Reliability and Availability Targets

- **NF11**: The system must achieve 99.9% reliability for event processing (successful completion).
- **NF12**: The system must implement automatic recovery mechanisms for common failure scenarios.
- **NF13**: The system must provide redundancy for critical components to prevent single points of failure.
- **NF14**: The system must gracefully handle dependent service outages (Kafka, BigQuery) with appropriate retry mechanisms.
- **NF15**: The system must implement a dead-letter queue for events that cannot be processed after multiple attempts.

#### 4.2.4 Data Consistency Requirements

- **NF16**: The system must ensure that the state of entities in BigQuery accurately reflects the processed events.
- **NF17**: The system must handle concurrent updates to the same entity in a consistent manner.
- **NF18**: The system must maintain referential integrity between related entities.
- **NF19**: The system must provide mechanisms to detect and reconcile data inconsistencies.
- **NF20**: The system must ensure that data in BigQuery is consistent with source systems, allowing for the expected processing delay.

#### 4.2.5 Security and Compliance Requirements

- **NF21**: The system must implement appropriate authentication and authorization for all components.
- **NF22**: The system must encrypt data in transit and at rest.
- **NF23**: The system must maintain audit logs for all data processing activities.
- **NF24**: The system must comply with GDPR requirements for personal data handling.
- **NF25**: The system must implement appropriate data access controls based on client organization boundaries.

### 4.3 Operational Requirements

#### 4.3.1 Monitoring and Observability

- **O1**: The system must provide dashboards showing the overall health and performance of the pipeline.
- **O2**: The system must emit structured logs that can be used for troubleshooting and analysis.
- **O3**: The system must track and expose metrics for throughput, latency, error rates, and resource utilization.
- **O4**: The system must support tracing of individual messages through the entire pipeline using unique identifiers.
- **O5**: The system must provide visibility into the state of processing for each entity type and client.

#### 4.3.2 Alerting Mechanisms

- **O6**: The system must generate alerts for abnormal conditions such as high error rates, degraded performance, or component failures.
- **O7**: The system must provide configurable alerting thresholds to minimize false positives.
- **O8**: The system must include context information in alerts to facilitate troubleshooting.
- **O9**: The system must support different notification channels for alerts (email, SMS, Slack, etc.).
- **O10**: The system must implement alert correlation to reduce noise during widespread issues.

#### 4.3.3 Disaster Recovery

- **O11**: The system must support complete recovery from a catastrophic failure with minimal data loss.
- **O12**: The system must provide mechanisms to replay events from Kafka in case of processing failures.
- **O13**: The system must maintain backups of critical configuration and state information.
- **O14**: The system must support regional failover for critical components.
- **O15**: The system must include documented procedures for disaster recovery scenarios.

#### 4.3.4 Multi-Tenancy Support

- **O16**: The system must properly isolate data and processing for approximately 1000 client organizations.
- **O17**: The system must ensure that events from one client do not impact processing for other clients.
- **O18**: The system must provide tenant-specific monitoring and metrics.
- **O19**: The system must enable tenant-specific configurations where appropriate.
- **O20**: The system must scale resources efficiently across all tenants.

#### 4.3.5 Administrative Interfaces and Tools

- **O21**: The system must provide interfaces for monitoring event processing status.
- **O22**: The system must support tools for diagnosing and troubleshooting issues.
- **O23**: The system must include mechanisms for manual intervention when necessary.
- **O24**: The system must support configuration management for pipeline components.
- **O25**: The system must provide access controls for administrative interfaces based on roles.

### 4.4 Forward Compatibility Requirements

#### 4.4.1 Considerations for Phase 2 (Streamlined Data Migration)

- **FC1**: The system architecture must support the addition of new entity types without significant rework.
- **FC2**: The system must implement modular transformation logic that can be extended for new data types.
- **FC3**: The system must support schema evolution patterns for BigQuery tables.
- **FC4**: The system must provide developer-friendly interfaces for defining new entity processing.
- **FC5**: The system must include comprehensive documentation to facilitate Phase 2 development.

#### 4.4.2 Architecture Decisions for Phase 3 (GraphQL API Integration)

- **FC6**: The system must adopt event formats and structures that will be compatible with GraphQL integration.
- **FC7**: The system must support bidirectional event flow to allow future external event ingestion.
- **FC8**: The system must implement authentication and authorization patterns that can extend to external systems.
- **FC9**: The system must maintain clear separation of concerns to facilitate API integration.
- **FC10**: The system must support event subscription patterns for real-time data access.

#### 4.4.3 Foundation Elements for Phase 4 (Apigee Replacement)

- **FC11**: The system must adopt consistent API design patterns that can be extended to a unified GraphQL API.
- **FC12**: The system must implement service discovery mechanisms that will support API federation.
- **FC13**: The system must standardize error handling and response formats.
- **FC14**: The system must support traffic management patterns that can be applied to API requests.
- **FC15**: The system must implement metrics collection that can be extended to API performance monitoring.

## 5. Solution Architecture

### 5.1 Architecture Overview

The Project Phoenix architecture will replace the current Flink StateFun component with a Google Cloud Dataflow based solution while maintaining the existing Kafka integration and BigQuery output. The high-level architecture is illustrated below:

```
┌───────────┐     ┌───────────┐     ┌─────────────────────┐     ┌───────────┐
│           │     │           │     │                     │     │           │
│  Sources  │────▶│   Kafka   │────▶│  Google Dataflow    │────▶│ BigQuery  │
│           │     │           │     │                     │     │           │
└───────────┘     └───────────┘     └─────────────────────┘     └───────────┘
     ▲                                        │
     │                                        │
     │                                        ▼
┌────┴────────┐                        ┌─────────────┐
│             │                        │             │
│  Huddle     │                        │  Enhanced   │
│             │                        │  Monitoring │
└─────────────┘                        └─────────────┘
```

#### Component Descriptions

1. **Sources**:
   - Huddle application events
   - External system integrations
   - Async imports and fabrications
   - No changes required to existing event producers

2. **Kafka**:
   - Continues to serve as the message broker for event streaming
   - Maintains existing topic structure and event formats
   - Provides buffering and replay capabilities

3. **Google Dataflow**:
   - Replaces Flink StateFun for event processing
   - Processes events through entity-specific pipelines
   - Manages state and performs transformations
   - Implements business logic for each entity type
   - Autoscales based on throughput requirements

4. **BigQuery**:
   - Continues to serve as the data warehouse for processed data
   - Maintains existing table structure and schema
   - Provides data for analysis and reporting

5. **Enhanced Monitoring**:
   - Provides visibility into pipeline performance and health
   - Tracks events through the system using unique identifiers
   - Generates alerts for abnormal conditions
   - Supports troubleshooting and diagnostics

#### Data Flow Patterns

1. **Event Ingestion**:
   - Events generated from sources and published to Kafka topics
   - No changes to existing event formats or topic structure

2. **Event Processing**:
   - Dataflow pipelines consume events from Kafka topics
   - Events are routed to entity-specific processing logic
   - Processing includes validation, transformation, and state management
   - Processed data is written to BigQuery

3. **Error Handling**:
   - Invalid or unprocessable events are routed to dead-letter queues
   - Errors are logged with context information
   - Alerts are generated for abnormal error rates

4. **Monitoring and Observability**:
   - Metrics are collected for all components
   - Logs are generated with correlation IDs for tracing
   - Dashboards provide visibility into pipeline health and performance

#### Integration Points

1. **Upstream Integration**:
   - Kafka serves as the integration point for event producers
   - No changes to existing producer integration

2. **Downstream Integration**:
   - BigQuery serves as the integration point for data consumers
   - No changes to existing consumer integration

3. **Operational Integration**:
   - Monitoring systems integrate with GCP monitoring services
   - Alerting integrates with existing notification channels

### 5.2 Technology Stack

#### Core Technologies

1. **Google Cloud Dataflow**:
   - Fully managed service for stream and asynchronous processing
   - Built on Apache Beam for unified programming model
   - Autoscaling capabilities for efficient resource utilization
   - Comprehensive monitoring and diagnostics
   - Integration with other GCP services

2. **Apache Kafka**:
   - Distributed event streaming platform
   - Provides durable storage and replay capabilities
   - Maintains existing integration with event producers

3. **Google BigQuery**:
   - Fully managed, serverless data warehouse
   - Provides storage for processed entity data
   - Supports SQL queries for analysis and reporting

4. **Apache Beam**:
   - Unified programming model for stream and asynchronous processing
   - Used by Dataflow for pipeline implementation
   - Supports Java and Python SDKs

#### Supporting Technologies

1. **Google Cloud Monitoring**:
   - Provides visibility into system health and performance
   - Collects metrics from Dataflow and other components
   - Supports custom dashboards and alerts

2. **Google Cloud Logging**:
   - Centralizes logs from all components
   - Supports structured logging for better analysis
   - Integrates with monitoring for comprehensive observability

3. **XTDB (Optional)**:
   - Immutable database with temporal query capabilities
   - Could be used for complex temporal data enrichment
   - Provides "bitemporal" modeling for handling out-of-order events

#### Rationale for Technology Choices

1. **Google Cloud Dataflow**:
   - Provides a managed service to reduce operational overhead
   - Built on Apache Beam, which is more widely used and better documented than Flink StateFun
   - Offers better developer experience and simpler learning curve
   - Provides native integration with other GCP services
   - Offers comprehensive monitoring and observability
   - Supports autoscaling to handle varying workloads efficiently

2. **Apache Kafka (continued use)**:
   - Already integrated with event producers
   - Provides necessary durability and replay capabilities
   - Well-established in the current architecture

3. **Google BigQuery (continued use)**:
   - Already stores processed entity data
   - Provides necessary query capabilities for reporting
   - No need to change this component

4. **XTDB (optional consideration)**:
   - Specialized database designed for temporal data processing
   - Could provide significant advantages for handling out-of-order events
   - Would add complexity to the architecture
   - Decision pending based on prototype evaluation

### 5.3 Technical Path Options

Based on the requirements and technology evaluation, three potential architectural paths have been identified:

#### Path 1: Direct Streaming Pipeline

```
┌───────────┐     ┌───────────┐     ┌─────────────────────┐     ┌───────────┐
│           │     │           │     │                     │     │           │
│  Sources  │────▶│   Kafka   │────▶│  Google Dataflow    │────▶│ BigQuery  │
│           │     │           │     │                     │     │           │
└───────────┘     └───────────┘     └─────────────────────┘     └───────────┘
```

**Description**: This is the most straightforward approach, replacing Flink StateFun with Dataflow while maintaining the same overall architecture. Dataflow would handle all transformations, enrichments, and business logic directly.

**Advantages**:
- Simplest architecture with fewest components
- Fully managed by GCP with minimal operational overhead
- Straightforward migration path
- Lower complexity for development and operations

**Challenges**:
- Handling out-of-order events would rely solely on Dataflow's windowing capabilities
- Complex temporal queries might be difficult to implement
- Might require custom state management for some entity types

**Best suited for**:
- Data with minimal temporal complexity
- Relatively narrow time windows for out-of-order events
- Emphasis on simplicity and operational efficiency

#### Path 2: Enrichment-Based Pipeline

```
┌───────────┐     ┌───────────┐     ┌─────────────────┐     ┌───────┐     ┌───────────┐
│           │     │           │     │                 │     │       │     │           │
│  Sources  │────▶│   Kafka   │────▶│  Dataflow       │────▶│ XTDB  │────▶│ BigQuery  │
│           │     │           │     │  (Processing)   │     │       │     │           │
└───────────┘     └───────────┘     └─────────────────┘     └───────┘     └───────────┘
```

**Description**: Dataflow processes events in real-time but leverages XTDB for temporal data enrichment and complex time-based joins before writing to BigQuery.

**Advantages**:
- Combines Dataflow's scalable processing with XTDB's sophisticated temporal capabilities
- Better handling of complex out-of-order events
- More powerful temporal query capabilities
- Clean separation of concerns

**Challenges**:
- More complex architecture with additional components
- Requires managing XTDB as a separate system
- Potential performance bottlenecks if not properly sized
- Additional operational overhead

**Best suited for**:
- Complex temporal requirements including frequent out-of-order processing
- Need for sophisticated historical data corrections
- Applications requiring complex temporal queries

#### Path 3: Dual-Pipeline Approach

```
                                ┌─────────────────────┐
                                │                     │
                                │  Dataflow           │
                                │  (Real-time)        │───┐
┌───────────┐     ┌───────────┐ │                     │   │
│           │     │           │ └─────────────────────┘   │     ┌───────────┐
│  Sources  │────▶│   Kafka   │                           │────▶│ BigQuery  │
│           │     │           │ ┌─────────────────────┐   │     │           │
└───────────┘     └───────────┘ │                     │   │     └───────────┘
                     │  │       │  XTDB               │───┘
                     │  │       │                     │
                     │  │       └─────────────────────┘
                     │  │                 │
                     │  │                 │
                     │  │                 ▼
                     │  │           ┌─────────────┐
                     │  └──────────▶│  Dataflow   │
                     │              │  (Async)    │
                     │              │             │
                     ▼              └─────────────┘
               ┌─────────────┐
               │             │
               │  Router     │
               │             │
               └─────────────┘
```

**Description**: Separate pipelines for different types of data flows - real-time events go directly through Dataflow, while historical or out-of-order events are processed through XTDB first.

**Advantages**:
- Optimized paths for different data timing needs
- Better isolation of concerns
- Can be more performant for mixed workloads
- More granular scaling based on workload type

**Challenges**:
- Most complex architecture to implement and maintain
- Potential data consistency issues between pipelines
- Requires sophisticated routing logic
- Highest operational overhead

**Best suited for**:
- Systems with clearly separable real-time vs. asynchronous processing requirements
- High volume of both real-time and historical data
- Need for optimized performance for different workload types

#### Comparative Analysis and Recommendation

Based on the analysis of requirements and technical options, the following comparative assessment can be made:

| Criteria | Path 1: Direct | Path 2: Enrichment | Path 3: Dual |
|----------|---------------|-------------------|-------------|
| Architectural Complexity | Low | Medium | High |
| Operational Overhead | Low | Medium | High |
| Development Complexity | Low | Medium | High |
| Temporal Processing Capability | Basic | Advanced | Advanced |
| Scalability | Good | Good | Excellent |
| Maintainability | Excellent | Good | Fair |
| Migration Complexity | Low | Medium | High |

**Preliminary Recommendation**:
- Start with prototyping Path 1 (Direct Streaming) as it offers the simplest migration path
- Evaluate its capabilities for handling temporal requirements through early prototyping
- If temporal requirements cannot be met, consider Path 2 (Enrichment-Based)
- Only consider Path 3 if there are clear performance benefits that justify the added complexity

The final decision will be informed by the results of technical prototyping as outlined in the Technical Evaluation Framework.

### 5.4 Phase Integration Architecture

#### How Project Phoenix Architecture Supports Future Phases

The Project Phoenix architecture is designed to establish a foundation that supports future phases of the Data Fabric migration:

1. **Modular Design**:
   - Entity-specific processing is isolated to facilitate adding new entities in Phase 2
   - Clear separation of concerns enables integration with GraphQL in Phase 3
   - Consistent API patterns prepare for unified API in Phase 4

2. **Scalable Infrastructure**:
   - Cloud-native approach with managed services provides scalability for future growth
   - Autoscaling capabilities handle increasing workloads across phases
   - Multi-tenant design supports expansion of client base

3. **Developer Experience**:
   - Improved tooling and documentation reduces onboarding time for future phases
   - Standardized patterns make it easier to add new functionality
   - Better monitoring and observability facilitates troubleshooting

#### Integration Touchpoints for Phase 2

The following architectural elements provide integration points for Phase 2 (Streamlined Data Migration):

1. **Pipeline Templates**:
   - Reusable pipeline patterns for new entity types
   - Standardized transformation logic
   - Configurable components

2. **Schema Management**:
   - Support for schema evolution in BigQuery tables
   - Versioning of event formats
   - Compatibility with existing data

3. **Development Tooling**:
   - CI/CD pipelines for automated deployment
   - Testing frameworks for validation
   - Documentation for developer onboarding

#### Design Considerations for Phases 3 and 4

The architecture incorporates several design elements that prepare for Phases 3 and 4:

1. **Event-Driven Architecture**:
   - Establishes patterns for event-based integration
   - Supports bidirectional event flow needed for GraphQL subscriptions
   - Provides foundation for event-sourced APIs

2. **Authentication and Authorization**:
   - Implements security patterns that can extend to external systems
   - Supports multi-tenant isolation required for API federation
   - Establishes identity management approach

3. **API Design Patterns**:
   - Consistent entity representations across the system
   - Standardized error handling and response formats

