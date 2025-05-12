# Project Phoenix: Data Flow Diagrams

## Overview

This document provides visual representations of the current Ambrosia architecture and the proposed Project Phoenix architecture alternatives. These diagrams illustrate the data flows, component interactions, and key integration points.

## Current Ambrosia Architecture

```
┌───────────┐     ┌───────────┐     ┌─────────────────────┐     ┌───────────┐
│           │     │           │     │                     │     │           │
│  Sources  │────▶│   Kafka   │────▶│  Flink StateFun     │────▶│ BigQuery  │
│           │     │           │     │                     │     │           │
└───────────┘     └───────────┘     └─────────────────────┘     └───────────┘
     ▲                                        │
     │                                        │
     │                                        ▼
┌────┴────────┐                        ┌─────────────┐
│             │                        │             │
│  Huddle     │                        │  Monitoring │
│             │                        │             │
└─────────────┘                        └─────────────┘
```

### Component Description

1. **Sources**:
   - Huddle application events
   - External system integrations
   - Batch imports and fabrications

2. **Kafka**:
   - Message broker for event streaming
   - Topics organized by source

3. **Flink StateFun**:
   - Core processing component being replaced
   - Maintains state for different entity types
   - Processes events through entity-specific functions
   - Applies transformations and enrichments

4. **BigQuery**:
   - Data warehouse for analysis and reporting
   - Contains 9 primary tables per entity type

5. **Monitoring**:
   - Limited visibility into processing
   - Requires production access for detailed debugging

## Proposed Architecture: Path 1 (Direct Streaming)

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

### Key Differences

1. **Google Dataflow**:
   - Replaces Flink StateFun for event processing
   - Managed service with auto-scaling
   - Unified batch and streaming model

2. **Enhanced Monitoring**:
   - Improved visibility through Dataflow monitoring
   - Structured logging by tenant ID
   - Ability to trace messages through the system

## Proposed Architecture: Path 2 (Enrichment-Based)

```
┌───────────┐     ┌───────────┐     ┌─────────────────┐     ┌───────┐     ┌───────────┐
│           │     │           │     │                 │     │       │     │           │
│  Sources  │────▶│   Kafka   │────▶│  Dataflow       │────▶│ XTDB  │────▶│ BigQuery  │
│           │     │           │     │  (Processing)   │     │       │     │           │
└───────────┘     └───────────┘     └─────────────────┘     └───────┘     └───────────┘
     ▲                                      │                   │
     │                                      │                   │
     │                                      ▼                   ▼
┌────┴────────┐                      ┌─────────────┐     ┌─────────────┐
│             │                      │             │     │             │
│  Huddle     │                      │  Enhanced   │     │ Temporal    │
│             │                      │  Monitoring │     │ Query API   │
└─────────────┘                      └─────────────┘     └─────────────┘
```

### Key Differences

1. **XTDB**:
   - Provides temporal data enrichment
   - Handles complex time-based queries
   - Manages out-of-order events

2. **Temporal Query API**:
   - Allows complex temporal queries
   - Supports audit and historical data access

## Proposed Architecture: Path 3 (Dual-Pipeline)

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
     ▲                  │       │  XTDB               │───┘
     │                  │       │                     │
     │                  ▼       └─────────────────────┘
┌────┴────────┐    ┌─────────┐          │
│             │    │         │          │
│  Huddle     │    │ Router  │          ▼
│             │    │         │    ┌─────────────┐
└─────────────┘    └─────────┘    │             │
                        │         │  Dataflow   │
                        │         │  (Batch)    │
                        ▼         │             │
                   ┌─────────────┐└─────────────┘
                   │             │
                   │  Enhanced   │
                   │  Monitoring │
                   │             │
                   └─────────────┘
```

### Key Differences

1. **Router**:
   - Directs events to appropriate processing pipeline
   - Real-time events go directly to Dataflow
   - Historical/out-of-order events go to XTDB

2. **Dual Processing**:
   - Real-time pipeline for current events
   - Batch pipeline for historical processing
   - Merged results in BigQuery

## Entity-Specific Data Flow Example: Desk Bookings

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────────┐
│              │     │              │     │                      │
│  Desk        │────▶│  Kafka       │────▶│  Dataflow Pipeline   │
│  Booking     │     │  Topic       │     │                      │
│  Events      │     │              │     │  ┌──────────────┐    │
└──────────────┘     └──────────────┘     │  │              │    │
                                           │  │ Parse Event │    │
                                           │  │              │    │
                                           │  └──────┬───────┘    │
                                           │         │            │
                                           │         ▼            │
                                           │  ┌──────────────┐    │     ┌───────────────┐
                                           │  │              │    │     │               │
                                           │  │ Enrich with  │    │     │  BigQuery     │
                                           │  │ Reference    │────┼────▶│  Desk_Bookings│
                                           │  │ Data         │    │     │  Table        │
                                           │  │              │    │     │               │
                                           │  └──────────────┘    │     └───────────────┘
                                           │                      │
                                           └──────────────────────┘
```

## Phase Integration Architecture

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  Phase 1: Project Phoenix                                                  │
│                                                                            │
│  ┌───────────┐     ┌───────────┐     ┌─────────────┐     ┌───────────┐    │
│  │           │     │           │     │             │     │           │    │
│  │  Sources  │────▶│   Kafka   │────▶│  Dataflow   │────▶│ BigQuery  │    │
│  │           │     │           │     │             │     │           │    │
│  └───────────┘     └───────────┘     └─────────────┘     └───────────┘    │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  Phase 2: Streamline Data Migration                                        │
│                                                                            │
│  ┌───────────┐     ┌───────────┐     ┌─────────────┐     ┌───────────┐    │
│  │  Enhanced │     │           │     │ Optimized   │     │           │    │
│  │  Sources  │────▶│   Kafka   │────▶│ Dataflow    │────▶│ BigQuery  │    │
│  │           │     │           │     │ Pipelines   │     │           │    │
│  └───────────┘     └───────────┘     └─────────────┘     └───────────┘    │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  Phase 3: GraphQL API Integration                                          │
│                                                                            │
│  ┌───────────┐     ┌───────────┐     ┌─────────────┐     ┌───────────┐    │
│  │           │     │           │     │             │     │           │    │
│  │  Sources  │────▶│   Kafka   │────▶│  Dataflow   │────▶│ BigQuery  │    │
│  │           │     │           │     │             │     │           │    │
│  └───────────┘     └───────────┘     └─────────────┘     └───────────┘    │
│       ▲                                                        │          │
│       │                                                        │          │
│       │            ┌─────────────────────────┐                │          │
│       └────────────│                         │◀───────────────┘          │
│                    │    GraphQL Federation   │                           │
│                    │                         │                           │
│                    └─────────────────────────┘                           │
│                                 ▲                                        │
│                                 │                                        │
│                    ┌────────────┴────────────┐                           │
│                    │                         │                           │
│                    │   External Clients      │                           │
│                    │                         │                           │
│                    └─────────────────────────┘                           │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  Phase 4: Apigee Replacement                                              │
│                                                                            │
│  ┌───────────┐     ┌───────────┐     ┌─────────────┐     ┌───────────┐    │
│  │           │     │           │     │             │     │           │    │
│  │  Sources  │────▶│   Kafka   │────▶│  Dataflow   │────▶│ BigQuery  │    │
│  │           │     │           │     │             │     │           │    │
│  └───────────┘     └───────────┘     └─────────────┘     └───────────┘    │
│       ▲                                                        │          │
│       │                                                        │          │
│       │            ┌─────────────────────────┐                │          │
│       └────────────│                         │◀───────────────┘          │
│                    │  Unified GraphQL API    │                           │
│                    │  (Apigee Replacement)   │                           │
│                    └─────────────────────────┘                           │
│                                 ▲                                        │
│                                 │                                        │
│                    ┌────────────┴────────────┐                           │
│                    │                         │                           │
│                    │   All API Consumers     │                           │
│                    │                         │                           │
│                    └─────────────────────────┘                           │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

## Notes

1. These diagrams represent high-level architectural concepts and will be refined as the project progresses.
2. Actual implementation details will be determined during the technical prototyping phase.
3. Monitoring components and additional integration points are simplified for clarity.
