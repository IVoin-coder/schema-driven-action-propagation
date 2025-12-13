# Schema-Driven Action Propagation (SDAP)
 schema-driven-action-propagation

**SDAP** - Architectural pattern for loosely coupled action propagation through schemas in distributed systems.

> **Core Idea:** Instead of relying on pre-shared, static data contracts (DTOs, APIs), SDAP lets the **producer send the exact data schema it needs right now**. The **consumer's job is to understand this schema and provide data that matches it.** This enables systems to evolve independently while maintaining perfect compatibility.

> ⚠️ **Status**: Draft / Under Discussion  
> 📢 **Seeking**: Architectural criticism and community review

## Table of Contents

- [Problem Statement](#problem-statement)
- [Terminology](#terminology)
- [SDAP Specification](#sdap-specification)
- [Basic Concepts](#basic-concepts)
- [SDAP Event Format](#sdap-event-format)
- [SDAP Integration Scenarios](#sdap-integration-scenarios)
  - [Use Case: Schema-Driven Action Propagation (SDAP) for Dynamic UI in Business Processes](#use-case-schema-driven-action-propagation-sdap-for-dynamic-ui-in-business-processes)
  - [Use Case: Backend Integration (Bidirectional Contracts) (Factory ↔ FRC)](#use-case-backend-integration-bidirectional-contracts-factory--frc)
- [How SDAP Works](#how-sdap-works)
- [Advantages](#advantages)
- [Typical Use Case](#typical-use-case)
- [Security Considerations](#security-considerations)
- [Schema Evolution Strategy](#schema-evolution-strategy)
- [Potential Risks & Mitigations](#potential-risks--mitigations)
- [When SDAP is Particularly Useful](#when-sdap-is-particularly-useful)
- [Discussion](#discussion)
- [Documentation](#documentation)
- [Related Resources](#related-resources)
- [License](#license)

## Problem Statement

Modern distributed systems often propagate actions between components connected by business processes. The problem arises when data transfer contracts change faster than UI or microservice delivery cycles.

Rigid DTOs, OpenAPI contracts, and manual synchronization lead to:
- High component coupling
- Need for synchronous releases
- Difficulty supporting multiple API versions
- Slow adaptation to business process changes

## Terminology

| Term | Definition |
|------|------------|
| **Action** | Process step requiring input, confirmation, or processing |
| **Schema** | Formal description of data structure required for action execution |
| **schemaHash** | Schema version identifier (typically a hash) |
| **Producer** | Event initiator |
| **Consumer** | Event handler |
| **UI-Renderer** | Component generating forms from schemas |
| **Schema Registry** | Centralized service storing allowed field names and reusable atomic components for schema composition |

## SDAP Specification

**Schema-Driven Action Propagation (SDAP)** is an architectural approach that standardizes the way actions and data structures are described between services. The approach is technology-agnostic, does not require a dedicated infrastructure layer, and does not impose any orchestration or routing mechanisms. SDAP defines only the **form**, but not the **execution** of the logic.

### What SDAP provides (Is)
- A formal way to describe the structure of actions and data
- A mechanism for loosely coupled interactions between services
- An approach in which the schema version is determined by the hash of its contents
- A model where different schema versions can coexist without migrations
- A way to unify data and actions independent of business logic
- A tool for predictable contract evolution without a centralized registry

### What SDAP does not provide (Is Not)
SDAP is not a universal solution. It is a specialized pattern focused on describing and propagating the shape of actions and their associated data schemas between loosely coupled components. It does not aim to replace or handle the following:

- **Not a process orchestration engine** – SDAP does not enforce workflow rules, schedule tasks, or implement business logic. It only ensures that participants understand the expected data structure for each action.
- **Not a replacement for stable APIs** – If processes and data structures are stable over long periods, SDAP may add unnecessary abstraction. It is intended for scenarios where flexibility and evolution are required.
- **Not a dynamic rules engine** – Conditions in schemas are used for validation and structuring data (for generating UI forms, configuring external services, defining data pipelines, etc.), not for triggering actions or evaluating business logic.
- **Not a UI framework** – While SDAP provides schemas that can be used for dynamic form generation, it does not mandate how forms are rendered, styled, or validated visually. Implementation of a UI renderer is optional.
- **Not limited to JSON or frontend interactions** – The JSON example is illustrative. SDAP is serialization-agnostic and can work with Protobuf, Avro, XML, or any other format. It is equally applicable to backend-to-backend communication, IoT, or any distributed system component.
- **Not dependent on a specific messaging system** – SDAP is messaging-agnostic and can work with Kafka, RabbitMQ, or other brokers. It does not require a central schema registry.
- **Not a schema storage solution** – Schema versions exist for validation and backward compatibility. SDAP is not a database for historical analysis of schemas.
- **Not a performance optimization tool** – SDAP focuses on decoupling and schema-driven propagation. It is not designed to improve throughput, latency, or memory usage.
- **Not a handler of large binary payloads** – SDAP is suitable for lightweight action/event payloads. Large files or documents should be handled via dedicated storage mechanisms.
  
### SDAP does not require:
- a schema registry (optional; useful for namespace management and field reuse)
- a migration engine
- an action propagation engine
- specialized route monitoring
- a message broker or middleware
### SDAP does not define:
- business logic for actions
- execution routes
- workflow mechanics
- process orchestration

## Basic Concepts

### Versioning Model
A schema version is determined by the hash of its contents. Any change to the data structure or action results in a new version. Old versions remain unchanged, which allows:
- to avoid migrations
- to ensure forward and backward compatibility
- to process events from different versions in parallel

### Actions
Action is an **operation type identifier** described in the schema.

**Schema as a universal descriptor:** While schemas can drive UI form generation, their primary role is to **formally describe data structures for any consumer** — whether it's a UI renderer, another microservice, an external system configuration, or a data pipeline. The schema defines "what" data is needed, not "who" will use it or "how".

Important: An Action does not contain business logic* and does not define how it is executed.

Business logic, the response to an Action, and the order of execution of steps are entirely the responsibility
of a specific system or service.

SDAP defines the structure of Action input and output data, as well as the permissible
state transitions, but does not describe the execution process.

### Propagation
Propagation in SDAP should be understood semantically, not as a runtime mechanic and does not handle orchestration or define a routing mechanism.
The system itself decides:
- how the action chain is constructed,
- which services events are sent to,
- which components are responsible for execution.

SDAP only defines the format for describing steps and their data, ensuring
loosely coupled integration.

### Goals and Objectives

SDAP solves the following problems:
- Eliminate rigid API contracts between system participants
- Ensure compatibility during process evolution
- Reduce coupling between event producers and consumers
- Accelerate new process rollout without UI or service recompilation
- Centralize validation and data description through schemas

### SDAP Event Format

An SDAP event is a **self-contained message** that carries both the intent of an action and the formal description (`schema`) of the data required to execute it. This allows consumers to process the action independently, without needing prior knowledge of a fixed API contract.

The format consists of three main parts:
- **`metadata`**: Technical information about the event and the process.
- **`schema`**: A declarative description of the data structure and user interface required to perform the action.
- **`context`**: The business context in which the action takes place.

**Important Note:** The `schema` field can be `null` if the recipient has already cached the schema using its `schemaHash`. This optimizes the event size during repeated communication.

**Key `metadata` fields:**
- **`actionType`**: The type of action (domain event) semantically describing a step in the business process (e.g., `APPROVE_PAYMENT`).
- **`actionId`**: **A unique global identifier for this specific event instance**. Critical for duplicate detection (deduplication) by consumers.
- **`correlationId`**: An identifier for tracing the entire business process (e.g., a single order), linking all related events.
- **`schemaHash`**: **A hash fingerprint of the `schema` object's content**. Used for unambiguous schema version identification, caching, and data integrity verification.

**Schema Evolution:** When modifying a schema, any additions or changes must maintain **backward compatibility**. It is recommended to add new fields as optional (`"required": false`) and never remove or fundamentally change the semantics of existing fields.

## SDAP Integration Scenarios

SDAP's flexibility allows it to be applied in different contexts, from user-facing interfaces to backend system integration.

### Use Case: Schema-Driven Action Propagation (SDAP) for Dynamic UI in Business Processes
See how [Use Case: Dynamic UI with SDAP](use-cases/01-dynamic-ui-integration.md) - How SDAP enables dynamic form generation without redeployment...

### Use Case: Backend Integration (Bidirectional Contracts) (Factory ↔ FRC)
Explore [factory-FRC integration with SDAP](use-cases/02-bidirectional-backend.md) for evolving contracts...

## How SDAP Works

1. **Initiator component** (e.g., BPMN engine, microservice) creates an event containing:
   - Process identifier
   - Action type (stage, status, etc.)
   - Schema hash (schemaHash)
   - Schema (schema) - if it's new or updated

2. **Event is published** to an event broker (e.g., Kafka, RabbitMQ)

3. **Consumer component** of the event (e.g., business logic handler):
   - Checks for schema existence by hash
   - Validates data according to the schema
   - Stores the event with schema version binding
   - Delivers it to UI or other systems for further interaction

4. **UI receives the schema** upon request and dynamically generates a form for data input

5. **User fills out the form**, and the result is sent back

6. **Component receiving the response** validates data against the same schema version and continues business process execution

## Advantages

- **Loose coupling**: Each participant works only with hash and schema, not rigid DTOs or contracts
- **Flexibility and scalability**: New steps and forms can be added without recompilation or new interface releases
- **Independent evolution**: Event producer can evolve without breaking consumers; if schema changes, it simply gets a new version
- **Multi-version schema support**: Old steps continue working with stored schema, new ones use updated version
- **Development speed**: Schemas can be used for form generation, documentation, and validation simultaneously

## Typical Use Case

1. Business process is launched in the system
2. At each task stage, the orchestrator emits SDAP event with JSON format
3. Task service receives event, validates and caches schema
4. UI requests pending tasks, receives schemas, renders forms
5. User submits form data, which is validated against schema
6. Validated action propagates to the current process step

## Security Considerations

### Schema Integrity Protection
- **Schema Registry Validation**: All schemas must be composed using field names and components registered in the Schema Registry
- **Digital Signatures**: Schemas can be signed by trusted authorities (e.g., process orchestrator) to prevent tampering
- **Hash Validation**: Consumers should verify that `schemaHash` matches the actual schema content

### Data Validation
- **Input Sanitization**: UI-Renderer must sanitize inputs based on schema types to prevent injection attacks
- **Context-Aware Encoding**: Render form fields with proper encoding for output context
- **Size Limits**: Enforce reasonable limits on string lengths and array sizes

### Access Control
- **Schema-level Authorization**: Verify user has permission to execute specific action types
- **Context Validation**: Ensure action context (processId, userId) matches current session

## Schema Evolution Strategy

### Versioning Approach
1. **Hash-Based Identification**: Each unique schema generates a new hash; running processes continue using original versions
2. **Field-Based Composition**: Schemas are built from registered field names in Schema Registry, allowing controlled evolution
3. **Forward Compatibility**: New fields should be optional to avoid breaking existing consumers

### Migration Techniques
- **Lazy Migration**: Running processes complete with original schema version
- **Registry-Controlled Changes**: Only field names registered in Schema Registry can be used in new schemas
- **Dual Validation**: Support both old and new schemas during transition periods

### Cache Management
- **Schema Caching**: Consumers cache schemas by hash to avoid repeated downloads
- **TTL Policies**: Set time-to-live for cached schemas to ensure periodic refresh
- **Registry Synchronization**: Regular updates from Schema Registry to get new allowed field names

## Potential Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| **Incorrect UI behavior with unaccounted schema** | UI always receives the current schema from the handler, not relying on caches |
| **Database growth due to storing multiple schema versions** | Schemas are saved only with real changes, identified by hash |
| **UX difficulties and form customization** | Use mature form generation libraries (e.g., react-jsonschema-form) |
| **Schema management and field reuse** | Schema can be assembled from atomic, reusable components (checkbox, input, select, etc.) |
| **Unpredictable changes on the producer's side** | Each schema is built based on the structure of fields stored in the Schema Registry. |

## When SDAP is Particularly Useful

✅ **Systems with rapidly changing business processes**  
✅ **Large number of similar steps with similar data structure**  
✅ **Microservice architecture** where reducing coupling between services is important  
✅ **UI that must dynamically adapt** to process changes without redeployment  
✅ **Need for independent evolution** of frontend and backend components  

## Discussion

**This pattern is under active discussion!** I welcome architectural criticism and real-world implementation examples.

### 🔑 Key Questions for the Community

1. **Security**: How to protect schemas from forgery?
2. **Performance**: Maximum practical schema size?
3. **UX**: How to customize dynamically generated forms?
4. **Evolution**: Migrating running processes to new schemas?

### 📝 How to Provide Feedback

| Method | Purpose |
|--------|---------|
| **[Create an Issue](https://github.com/IVoin-coder/schema-driven-action-propagation/issues/new)** | Questions, suggestions, criticism |
| **[Start a Discussion](https://github.com/IVoin-coder/schema-driven-action-propagation/discussions)** | Architectural debates |
| **[Submit a Pull Request](https://github.com/IVoin-coder/schema-driven-action-propagation/pulls)** | Documentation improvements |

### 🎯 Specifically Looking For

- ❌ Potential vulnerabilities in the approach
- ❌ Scaling limitations in production
- ❌ Similar patterns in other systems
- ❌ Blind spots or overlooked edge cases

## Documentation

This section tracks the documentation status for the SDAP pattern.

| Document | Status | Description |
|----------|--------|-------------|
| **[ADR-001: SDAP Pattern](docs/adr/001-sdap-pattern.md)** | ✅ Complete | **Architectural Decision Record** - Core pattern documentation with context, decision, and consequences |
| **[Examples](docs/examples.md)** | 🔄 Planned | **Use cases and implementations** - Real-world examples and integration scenarios |
| **[Security Guide](docs/security.md)** | 🔄 Planned | **Best practices** for secure SDAP implementation |

### Contribution Opportunities
- Help draft the **Specification** document
- Provide **real-world examples** for the Examples document
- Contribute to the **comparative analysis** with other patterns
- Suggest **additional documentation** needs via Issues
## Related Resources

### 📚 Foundational Concepts
| Concept | Description | Relevance to SDAP |
|---------|-------------|-------------------|
| **Self-Describing Messages** | Messages that include metadata about their structure | Core principle of SDAP - schema travels with action |
| **Schema Evolution** | Techniques for evolving data structures without breaking compatibility | SDAP supports multiple schema versions simultaneously |
| **Dynamic UI Generation** | Rendering user interfaces from declarative schemas | SDAP enables forms to be generated from action schemas |

### 🏗️ Related Architectural Patterns
- **[Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)** — Foundation for asynchronous action propagation in SDAP
- **[CQRS (Command Query Responsibility Segregation)](https://martinfowler.com/bliki/CQRS.html)** — Can use SDAP for command validation and UI generation
- **[Process Orchestration](https://camunda.com/blog/2023/02/orchestration-vs-choreography/)** — BPMN/workflow engines can emit SDAP events for user tasks  
- **[Schema Registry Pattern]()** — Centralized schema management that can complement SDAP implementations
- **[Self-Describing Messages](https://dev.to/cadienvan/event-carried-state-transfer-a-pattern-for-distributed-data-management-in-event-driven-systems-165h)** — Core principle from DDD and Event-Driven architectures
- **[Backward/Forward Compatibility](https://en.wikipedia.org/wiki/Forward_compatibility)** — Schema evolution techniques at the process instance level
    
## License

**Distributed under the MIT License.** See the `LICENSE` file in the repository for details.

---

**👤 Author**: [Igor Soldatenko]  
**Repository**: https://github.com/IVoin-coder/schema-driven-action-propagation  
**Goal**: Create a community-reviewed architectural pattern for modern distributed systems  
**Discussion**: [GitHub Discussions](https://github.com/IVoin-coder/schema-driven-action-propagation/discussions)  
**Issues**: [GitHub Issues](https://github.com/IVoin-coder/schema-driven-action-propagation/issues)
