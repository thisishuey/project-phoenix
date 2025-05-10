# Ambrosia System Architecture Summary

## Overview

Ambrosia is a stateful event processing framework built on Apache Flink StateFun (version 3.3.0) designed to consume events from various sources (primarily Kafka topics), process them through entity-specific functions, maintain state for different entities, and write data to BigQuery for analysis and reporting.

## Core Architecture

The system is built around the concept of stateful functions that process events and maintain state for different entity types (desk, employee, floor, room, site, etc.). These functions are orchestrated by the Apache Flink StateFun framework, which provides scalable, fault-tolerant event processing with consistent state management.

The main components include:

1. **StateFun Framework:** The underlying Apache Flink StateFun framework that manages stateful function execution

2. **Function Definitions:** Definitions of stateful functions for different entity types

3. **Entity Functions:** Stateful functions that process events for specific entity types (desk, employee, site, floor, room, neighborhood, visit, etc.)

4. **Middleware Pipeline:** A series of middleware functions applied to each stateful function for cross-cutting concerns

5. **Event Processing Flow:** The flow of events from Kafka through preparation functions to entity functions

6. **BigQuery Integration:** Writing processed data to BigQuery tables for analysis

## Event Processing Flow

Events flow through the Ambrosia system as follows:

1. Events are consumed from Kafka topics by the Preparation and Visitor Service Preparation functions

2. These functions enrich the events with metadata and route them to the appropriate entity functions

3. Entity functions process the events, update their state, and write data to BigQuery

The system uses a sequence of steps to process events, as illustrated in the desk booking flow

## State Management

Ambrosia uses a history-based approach to state management. Each entity function maintains a history of events that have affected it, and for some entities, periodic snapshots of state. The current state is calculated by replaying these events.

The system uses two main approaches to state management:

1. **History without snapshots:** For entities like Desk, Neighborhood, Visit, and Greetly Location

2. **History with snapshots:** For entities like Employee, Floor, Room, and Site

The `maxHistoryLimiter` middleware enforces maximum history sizes for entities that don't use snapshots, dropping messages when an entity's history exceeds its maximum size.

### Managing State Without Snapshot

- **Desk:** Manages desk-related events and state
- **Neighborhood:** Manages neighborhood-related events and state
- **Visit:** Manages visit-related events and state
- **Greetly Location:** Manages location-related events and state

### Managing State with Snapshot
- **Employee:** Manages employee-related events and state
- **Floor:** Manages floor-related events and state
- **Room:** Manages room-related events and state

### Manages State with Distribution Middleware
- **Site:** Manages site-related events and state

Each entity function processes events specific to its entity type, updates its state accordingly, and writes data to BigQuery.

## Deployment Architecture

1. **Google Kubernetes Engine:** Hosts the Ambrosia application containers
2. **Docker Containers:** Package the Ambrosia application and its dependencies
3. **Apache Flink:** Provides the runtime environment for StateFun
4. **Kafka:** Provides event streaming and topic management
5. **BigQuery:** Stores processed data for analysis and reporting

The deployment is organized into different tiers (Development, Staging, Production) and regions (US, Europe, Asia) as shown in the deployment diagrams

## Error Handling and Monitoring

Ambrosia uses Bugsnag for error tracking and monitoring. The application is configured to send error reports to Bugsnag when errors occur in production.

The system also includes custom error types for specific scenarios.

## Integration Points

Ambrosia integrates with several systems:

1. **Kafka:** For event ingestion and messaging
2. **BigQuery:** For data storage and analysis
3. **Looker:** For data visualization and reporting
4. **Huddle:** The main application that generates events and consumes processed data

## Summary

Ambrosia is a robust, event-driven system built on Apache Flink StateFun that processes events, maintains entity state, and writes data to BigQuery. Its architecture is centered around stateful functions for different entity types, with a middleware pipeline that provides cross-cutting concerns like exception handling, logging, and history management.

The system is designed to be scalable, fault-tolerant, and maintainable, with clear separation of concerns between different components and functions. It provides a comprehensive framework for processing events from various sources, maintaining consistent state, and providing data for analysis and reporting.

Notes:

- This summary focuses on the core architecture and functionality of Ambrosia as a stateful event processing system built on Apache Flink StateFun.
- The system appears to be handling various entity types (desk, employee, floor, room, etc.) and maintaining their state through a history-based approach.
- The deployment architecture spans multiple environments and regions, suggesting a globally distributed system.