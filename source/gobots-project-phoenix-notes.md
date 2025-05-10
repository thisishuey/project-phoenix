# GoBots: Project Phoenix Notes

## Resources
Claude Conversation: https://claude.ai/share/4127b09d-c7e2-468c-9927-0e4c2a1a212b

## Use Cases

### Integrations:

#### Example 1: Employee Presence Data (UAPI → Huddle)

- Employee badges in, 3rd party sensor detects employee presence, CSV with badge events is uploaded by client
- Presence detected events are published that contains some ids (employee id, location id, etc)
- Presence data enriched with employee/location details or transformed into other entities (e.g. presence detected on a floor is rolled up to the site as well).
- Data projected to BQ for reporting, Vitess/Huddle for real time presence data
  
#### Example 2: Greetly Visits (Greetly → Huddle)

- Visitor checks in to visit a host (email) at a location
- Greetly publishes a visitor check in event for a specified host email at the specified location
- Visit event is enriched from other data streams:
  - OSS employee data based on host email (employee stream)
  - OSS site data based on location → site mapping (site stream, greetly location stream, greetly site config stream)
- Enriched visit projected to BQ for reporting, Huddle for visitor desk booking

### Example 3: AssetSpace → Request Manager

- TBD

### Raw/Historical data capturing (Data lake/warehouse)

For consumption by reports, analytics, AI/ML

### Event time processing

This means that when an event comes in and is effective in the past or in the future, that any reference data is also correct with respect to that effective date. e.g. “I need to know the state of entity ‘X' as of datetime 'Y’”

If a site has a network interruption for a few hours, when it comes back up there might be a lot of delayed sensor data that arrives at once. We want to make sure the floor plan still only reflects occupancy based on events that are recent, not old.

Each scenario below has a theoretical list of information that was sent to OfficeSpace. The first date will represent when OfficeSpace received the information. The second date will represent when the activity actually happened.

### Old presence events

Wednesday: An employee is transferred from the Finance to the Accounting department. This event happens at the same time the information it is sent to OfficeSpace, that is to say, it is sent in real-time.

 

Friday: A facility manager uploads the week's presence events. Our transferred employee came in every day of the week so has presence events in the upload for Monday, Tuesday, Wednesday, Thursday and Friday.

 

Even though the presence event upload is happening on Friday, any solution we choose needs to be able to correctly associate the employee to the Finance department for the events from Monday and Tuesday, and the Accounting department for the events from Wednesday, Thursday, and Friday.

 

Future Desk Bookings

Last week: There is a bookable desk in Room A.


Monday: An employee schedules a desk booking for Wednesday for that desk.

 

Tuesday: The scheduled desk is moved to Room B. This event happens at the same time the information is sent to OfficeSpace, that is to say, it is sent in real-time.

Thursday: The scheduled desk is moved again, now to Room C. This event happens at the same time the information is sent to OfficeSpace, that is to say, it is sent in real-time.

Friday: A facility manager runs a report to see the desk bookings for the last week and grouped by room. The manager expects to see our booking associated with Room B, since that is the room the desk was in when the booking started.

 

The main caveat for this scenario is that OfficeSpace is not guaranteed to get any new information on Wednesday when the booking starts, but we do need to make sure we have captured related information that is valid for that day. We need to be able to keep track of the booking coming up in the future to be sure that when the related information, like their room, changes, what we record for reporting gets changed too. It also isn't enough to look at a desk's room at the time a report is run. As we can see, if we took that approach it would seem like the booking was for Room C.

 

Desk Booking Series

Monday: An employee schedules a repeating desk booking for every day this week.

 

Wednesday: The employee legally changes their name. This event happens at the same time the information is sent to OfficeSpace, that is to say, it is sent in real-time.

 

Friday: A facility manager runs a report and groups it by employee. Only the new name should be shown in the grouped output and that group should include the events from before the name was changed as well as after.


The main caveat for this scenario is that the report has to show the latest name regardless of what the persons name was at the time the desk booking started.

 

Personal Identifiable Information

Monday: An employee leaves the company and requests to be forgotten. This event happens at the same time the information is sent to OfficeSpace, that is to say, it is sent in real-time.

 

Tuesday: Presence events are uploaded that include events from that employee before they left.

 

Friday: A facility manager runs a report about presence for the week. The report still show aggregate presence events for that employee, but when they drill down, any personally identifiable information has been anonymized.

Automatic updating of later state when a late event comes in. E.g., 

Pre-computing values that rely on data from other entities. E.g., What is the calendar day that a presence event was detected on? The event includes its effective timestamp, but many systems just send those in UTC. It’s actually the site that knows the timezone, so to be able to calculate that the presence event happened on April 2nd, for example, we need to know that timezone from site. 

Should feed the query part of the Universal API

Should consume the mutation part of the Universal API

Should ensure that any any piece of information only has a single source of truth and is only published by one entity. e.g., floor_online should only be published in floor events, not in desk events.

Should ensure that producers require zero knowledge about how consumers will use their events (decoupled)

Not choke on “bulk tasks” ← Pending a clearer use case of what was meant by bulk tasks by Juan.

Ambrosia Pain Points:
Problem: Schema management

Our "schema" definition is enforced by .proto descriptors (which is a serialization mechanism)

No real validation across producers/consumers, outside of structure

Requires inclusion of submodules in any repo that leverages them. 

Making changes requires updating multiple repos

Potential Solution: Schema registry (Kafka schema registry, external service, etc)

An external service that standardizes event format across applications

Ex: Kafka schema registry, Karapace  (open source schema management)

 

Problem: Difficult for devs to work with/Need to be a Flink expert

Steep learning curve for developers. Huge barrier of entry.

Need to understand the inner workings of Flink architecture to efficiently use the system

Wrong abstractions

Potential Solutions:

Standardized SDKs to emit events from any app

Ruby/JS/Python/??

Publishes events to Kafka

Validates schema (ideally from a schema registry)

Doesn’t have to be an SDK, could be a graphql service/endpoint instead so we dont have to maintain sdks for different languages.

This comes with the con that developers would have to add mutations for each event

Can mitigate this by having code to autocreate the mutations from a schema

Is it possible to have a single generic mutation for pushing an event JSON blob along with an event type and validating it against the schema?

Important thing is it decouples the “what” (I want to publish an event) from the “how” (kafka producer, etc)

Declarative Abstractions:

Focus developer concerns to what they want, not how to implement it

High level example of what this could look like:



Phoenix.emit_event(
  type: "vistor.checked-in",
  payload: {
    checkInRecordId: visit.checkInRecordId,
    visitorId: visit.visitorId,
    firstName: visit.firstName,
    lastName: visit.lastName,
    email: visit.email,
    eventStart: visit.checkInTime ?? visit.preregistrationEventStart,
    locationId: visit.locationId
  },
  # Optional transformation directives
  transformations: {
    enrich_with: ["employees", "location_site_mappings", "sites"],
    project_to: ["bigquery", "huddle"]
  }
)
Leverages prebuilt "processing code" (Flink jobs, cloud fn, w/e) that do the work

Still requires a small team to maintain the underlying processing engine, but hopefully not everyone has to be Flink experts

SQL Interface over our data streams

Provide an interactive query mode to iterate/test against data streams

Could potentially be published as Flink jobs from there

Neflix implemented something like this. See Streaming SQL in Data Mesh  for more detail

 

Problem: Difficult to get data for other entities/streams

Ex: Visits require data about employees, sites/site mappings/, greetly locations, etc

Statefun requires us to jump through hoops to get temporal history for an entity (or multiple entities in this case)

How do you handle when the data you need isn't there? We handle this with subscriptions today, but that has its own issues

 

Problem: Performance

What are the real bottlenecks. Memory? Throughput?

Is this a matter of tuning? Is it a Flink/Kafka a problem? Is our implementation just wrong?

 

Open Questions
What are the pain points with fabrications, will other options entirely remove them or just shift them to other parts of the solution?

Aren’t we just trying to dump data into BigQuery from MariaDB?

If we published to entity based kafka topics, like “employee” instead of source based topics like “raw-messages.huddle” do we even need a stream processing layer?

Who from product should be part of these discussions and from when?

Would a data lake architecture be a smart “phase 1” approach? e.g. capture as much raw, unstructured data as we can and decide the ETL pipeline looks like in later phases?

Can we use cloud functions for simple event processing/transformations? Do we need a full fledged event system like Flink/Dataflow?

Could we go live with a solution that doesn’t push to BQ? A solution that would replace BigQuery, the info that is currently in BQ will be available in the new solution.

Out of Scope
Tools to Consider
Google Dataflow

Tableflow

API that sits on top of Kafka and represents them as open table formats (Apache Iceberg, etc)

Should be “readable” from BigQuery with either BigQuery tables for Apache Iceberg  |  Google Cloud or Create BigLake external tables for Apache Iceberg  |  BigQuery  |  Google Cloud 

XTDB

Shifts the enrichment concern to query time rather than processing time

Supports event-time as a first-class citizen

Has “continuous queries” that can trigger writes to BQ when their results change

Nevermind, this was an AI hallucination. There were open feature requests for this but they were closed.

Akka

Doesn’t handle “event time” processing out-of-the-box, we would have to use it like legos to build our own

EventStoreDB

Supports event-replay out-of-the-box but doesn’t have an “event-time” concept

Axon Framework

Supports event-replay out-of-the-box but doesn’t have an “event-time” concept

Confluence Platform (including Kafka Streams and/or KSQLDB)

Flink Table/SQL APIs

Would require a catalog that supports time travel to use it’s temporal joins

