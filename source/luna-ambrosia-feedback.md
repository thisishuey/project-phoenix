# Luna Ambrosia Feedback

## Pain Points

- Statefun pods are not scaling
- Using a technology that is not popular and is not well documented
- Not using the events to write to the main data store (MariaDB) makes it easy to fail to send events to Kafka.
- The Fear of Fabrications. If things go wrong we need to re-fabricate Production.
- Not expertise in the technology, nor proper education to new engineers
- No fully understand the whole scope of what Ambrosia is doing.
- Verbose approach when adding new events.
- There is no visibility for what events are coming in and what was the result of those.
- Requires production access to be able to debug or see what is doing.
- "Replay from kafka" feature was never implemented and makes this whole system pointless.
- I don't think we are realizing the goal of a persistent event data store, given how many times we refabricate.
- Statefun actor state exceeds the message size limit. For sites this was fixed with a difficult-to-understand and difficult-to-debug restructuring of data across multiple actors.
- Complex infraestructure hard to maintain and debug.
- Slows down the development. Eg. by requiring a lot of changes just to write to a new column in BigQuert.

## Use Cases

- Get all the data from huddle to BigQuery efficiently.
- Data Needs
  - Real Time
    - Individual Events
    - Bulk Events
  - Async
    - Individual Events
    - Bulk Events
      - Fabrications
      - CSV Uploads
  - Disaster Recovery

## Development Experience

- Logs can be difficult to parse.
- A lot of up-front work adding new data and data types.
- Too fragile. bad data from a single client can bring down entire pipeline.
- Domain Knowledge: Typically Devs struggle with Ambrosia. It just seems hard to learn and conquer. And the deep architectural know-how is limited with just few folks. This is in contrast to Huddle where we have lots of Experts.
- Difficult to examine data in StateFun.
- Visibility
  - Infra (Hardware, memory, pods, VMs, etc)
  - Transportation (messages in, out, percentiles, throughput, success ratio, errors)
  - Business logic: types of events received, action taken, logics
- Is UAPI the evolution of Ambrosia? Is Phoenix the evolution of UAPI ? Do we really need something new?
- Can we just have clients (i.e. huddle) consuming from Kafka?
- Maybe we don't need  Ambrosia-Flink for real time events.
- Maybe we need a async job out of the real time events solution for async data.
- Maybe we need a async job to syncronize MariaDB with BQ directly.