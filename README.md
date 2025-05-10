# Project Phoenix Documentation

## Overview

Project Phoenix is Phase One of our data migration project, focused on replacing the Stateful Function infrastructure in Ambrosia. This repository contains the technical documentation and requirements for this migration effort.

## Background

Ambrosia is our event streaming solution that:
- Ingests events from various SaaS projects via Kafka
- Processes these events using Flink Statefun
- Projects the results to BigQuery

Project Phoenix aims to replace the Statefun component with a new solution while maintaining the existing data flow and output structure.

## Repository Contents

This repository contains technical documentation to support the development of the Technical Requirements Document (TRD) for Project Phoenix, including:

- BigQuery table schemas and data structures
- System architecture documentation
- Technical requirements templates
- Architecture summaries and feedback

## Purpose

This repository serves as a document repository to feed AI agents for building out the Technical Requirements Document (TRD) for Project Phoenix. It is not a code repository and does not contain any implementation details.

## Related Documentation

- [BigQuery Tables and Schemas](source/bigquery-tables-and-schema-20250508.md)
- [Ambrosia System Architecture Summary](source/ambrosia-system-arcitecture-summary.md)
- [Technical Requirements Document Template](source/trd-template.md) 