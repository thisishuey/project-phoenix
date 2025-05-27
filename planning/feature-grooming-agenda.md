# Feature Grooming Agenda

## Goals for Feature Planning

- Define Milestones
- Outline first Sprint(+)
  - Finalize any POC work
  - Greetly Location to be the first entity to tackle
- Outline the Path 3 solution as a worst case scenario
- Determine MVP for Presence events
- Medtronic by October
- We will need to run the solutions in parallel and then cutover once everything in migrated
- July is when Luna is going to start working on new Statefun
- One Entity, then two, then many!

## Questions

- Are we going to be ready for Feature Planning on Wednesday? Or could we do a TRD walk through with Luna Tuesday?
  - For Luna
    - What problems are we trying to solve
    - Just high level architecture decision, e.g. Beam & Dataflow
    - Review Pain Points and how they are solved
    - Allow for Q&A from Luna

## Concerns

- Dataflow has transient state, we will need to store this somewhere (currently testing BQ & XTDB)
  - If we use BQ, what does that mean?
  - Instead of raw events do we store Materialized Views?
  - Is there value in storing raw events in BQ instead of using Kafka as our Data Lake?

## TRD Changes

- XTDB Performance → Change this to “Enrichment Store” to separate the technology from the solution
  - XTDB v2 doesn’t seem to be able to handle the performance needed
  - XTDB v2 still in beta
  - XTDB v1 uses datalog instead of postgres
- Phase 2:
  - Pain Point: Schema management
  - Dev tooling
  - New Dev Onboarding
    - ArgoCD vCluster Setup
    - run Strimzi setup
- Beam tech: Java or Python (TBD, Leaning towards Java)

## Planning

- Tickets for Milestone 1
  - Establish Development Environment
    - Claude Code - Huey needs to talk to Kiley about Max
    - Setup test pipeline for shorter feedback loop
  - Dataflow development pipeline
    - Java Setup - publish repo
  - Solve the State Problem
    - Raw events to BQ? Create a ticket, Joe’s current work might make this moot
    - Spike on Materialize (Materialize: Real-time Data Integration & Transformation )
    - POC on using Beam to build “Interval Effective Model” (think materialized view) and push to BQ?
    - Postgres with time extensions?
  - greetlyLocation (try to break down into stories)
- Tickets for Milestone 2 - Greetly end-to-end
  - push to huddle (GraphQL)
  - topic to topic router (Beam)
  - greetlySiteConfig (try to break down into stories)
  - employee (try to break down into stories)
  - visit (try to break down into stories)
  - site (try to break down into stories)
- Cards for Milestone 3 - Location Data
  - floor
  - neighborhood
  - room
  - desk
- Cards for Milestone 4 - Supporting Data
  - designation
  - deskAvailability
  - deskBooking
  - move
  - user
- Cards for Milestone 5 - Finalizing
  - Documentation
  - Performance Testing
  - Dashboards
  - Deployment
  - Prep for Phase 2 with Luna
- Other Cards
  - Until we get a new mechanism in place, Beam will have to push to GraphQL mutation
    - We need to add a card to push visits to Huddle through mutation
    - We will need to evaluate this against pushing to a Kafka topic & setting huddle up to consume the topic
  - Multiple topics? Multiple Partitions? (Add a card to investigate)
    - Need to put pieces in place to manage this vs. Luna work (LD Flag? Topic-to-topic router?)
    - Card to investigate creating a topic to topic router
  - Dedupe
    - Can we use the existing?
    - Should we replace with Beam

## Parking Lot

- Concerns about milestone approach, concern was that we aren’t planning all the mileststones, but in fact will will plan all the milestones
- Don’t paint ourselves into a corner