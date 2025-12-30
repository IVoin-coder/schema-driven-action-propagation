# Schema-Driven Action Propagation (SDAP)
 schema-driven-action-propagation

**Schema-Driven Action Propagation (SDAP)**  is an architectural template in which actions are transmitted and interpreted based on schemas rather than hard-coded logic or fixed contracts.

> **Core Idea:** Instead of relying on pre-shared, static data contracts (DTOs, APIs), SDAP lets the **producer send the exact data schema it needs right now**. The **consumer's job is to understand this schema and provide data that matches it.** This enables systems to evolve independently while maintaining compatibility without synchronized releases.

> ⚠️ **Status**: Under Discussion  
> 📢 **Seeking**: Architectural criticism and community review

## Table of Contents

- [Problem Statement](#problem-statement)
- [Terminology](#terminology)
- [SDAP Specification](#sdap-specification)
- [Core Concepts](#core-concepts)
- [SDAP Event Format](#sdap-event-format)
- [Schema Evolution](#schema-evolution)
- [How SDAP Works](#how-sdap-works)
- [SDAP Integration Scenarios](#sdap-integration-scenarios)
  - [Use Case: Schema-Driven Action Propagation (SDAP) for Dynamic UI in Business Processes](#use-case-schema-driven-action-propagation-sdap-for-dynamic-ui-in-business-processes)
  - [Use Case: Backend Integration (Bidirectional Contracts) (Factory ↔ FRC)](#use-case-backend-integration-bidirectional-contracts-factory--frc)
- [Security Considerations](#security-considerations)
- [Schema Evolution Strategy](#schema-evolution-strategy)
- [Potential Risks & Mitigations](#potential-risks--mitigations)
- [When SDAP is Particularly Useful](#when-sdap-is-particularly-useful)
- [Discussion](#discussion)
- [Documentation](#documentation)
- [Related Resources](#related-resources)
- [Security Version Scheme Distribution (SVSD)](#security-version-scheme-distribution-svsd)
- [License](#license)

## Problem Statement

Modern distributed systems often propagate actions between components connected by business processes. Problems arise when data transfer contracts change faster than UI or microservice development cycles.

Rigid DTOs, OpenAPI contracts, and manual synchronization lead to:
- High component coupling
- Need for synchronous releases
- Difficulty supporting multiple API versions
- Slow adaptation to business process changes
SDAP provides an alternative by shifting the focus from API contracts to structural descriptions of actions.

## Terminology

| Term | Definition |
|------|------------|
| **Action** | A process step requiring input, confirmation, or processing |
| **Schema** | A formal description of data structure required for action execution |
| **schemaHash** | A schema version identifier (typically a hash) |
| **Producer** | Event initiator |
| **Consumer** | Event handler |
| **UI-Renderer** | Component generating forms from schemas |
| **Schema Registry** | An optional auxiliary component for namespace management and field reuse |

## SDAP Specification

**Schema-Driven Action Propagation** is an architectural template that standardizes the way actions and data structures are described between services. The approach is technology-agnostic, does not require a dedicated infrastructure layer, and does not impose any orchestration or routing mechanisms. This pattern defines only the **form**, but not the **execution** of the logic.

### What SDAP provides (Is)
- A formal way to describe the structure of actions and data
- Loose coupling through structural descriptions instead of rigid contracts
- Schema versioning based on content hashing
- A model where multiple schema versions coexist without migration
- A way to unify data and actions independent of business logic
- Predictable contract evolution without a centralized registry

### What does not provide (Is Not)
SDAP deliberately does not cover the following responsibilities:
- Execution and orchestration - SDAP does not define execution order, action lifecycle, or business process rules
- Stable contracts - SDAP is not intended for long-lived, static APIs
- Security and Trust - SDAP does not introduce trust or security models, leaving these decisions to the Producer and Consumer
- User interface - SDAP describes structure, not visualization or UX
- Transport and serialization - SDAP is transport-agnostic
- Schema storage and analytics - SDAP is not a schema repository or historical registry
  
### What does not require:
- Centralized schema registry
- Migration mechanisms
- Dedicated message brokers
- Specialized routing layers

### SDAP does not define:
- business logic for actions
- execution routes
- workflow mechanics
- process orchestration

## Core Concepts
### Versioning Model
A schema version is defined by a hash of its contents. Any structural change produces a new version. Older versions remain immutable, enabling:
- No migrations
- Forward and backward compatibility
- Parallel processing of multiple schema versions

### Actions
An Action is an intent identifier whose required data is described by a schema.

**A schema:**  While schemas can drive UI form generation, their primary role is to **formally describe data structures for any consumer** — whether it's a UI renderer, another microservice, an external system configuration, or a data pipeline. The schema defines "what" data is needed, not "who" will use it or "how".
- Describes what data is allowed
- Describes affordances (interaction capabilities), not business logic
- Does not specify how or by whom the action is processed

Important: An action contains no business logic and does not define execution mechanics. Interpretation is entirely the responsibility of the consuming system.

Business logic, the response to an Action, and the order of execution of steps are entirely the responsibility
of a specific system or service.

### Propagation
In SDAP, propagation is understood semantically, not as an execution mechanism. SDAP does not perform orchestration and does not define:
- Routing
- Call chains
- Execution order.
  
**Each system independently decides:**
- How to process actions
- Where to forward them
- Which components participate in execution
  
SDAP defines only the format of actions and their data.
Propagation does not imply delivery guarantees, execution semantics, or responsibility transfer.

### Goals and Objectives

SDAP addresses the following challenges::
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

**Important Note:** The schema field may be null if the consumer has already cached the schema using schemaHash. This optimizes message size for repeated interactions.

**Key `metadata` fields:**
- **metadata** - technical context
- **schema** - declarative structure description
- **context** - business context
- **schemaHash** - a hash fingerprint of the schema content

schemaHash uniquely identifies the schema version and is used for caching and integrity verification.

The schema may be omitted if the consumer already has the version identified by schemaHash.
schemaHash is not a trust or security mechanism.

## Schema Evolution
When schemas change, all additions or modifications must preserve backward compatibility.

## How SDAP Works
1. The producer creates an SDAP event with actionType and schemaHash
2. The schema is sent only if the consumer does not already have it
3. **The consumer:**
- Verifies schemaHash
- Validates data against the schema
- Interprets the action within its own context
4. All subsequent behavior is defined by the consuming system

SDAP does not prescribe where the final execution decision is made.

## Benefits
- **Loose coupling** - systems depend on schemas and hashes, not rigid DTOs
- **Flexibility and scalability** - new steps and forms without redeployment
- **Independent evolution** - producers and consumers evolve independently
- **Multi-version support** - old processes keep their schemas, new ones use updated versions
- **Development speed** - schemas serve UI generation, documentation, and validation

## SDAP Integration Scenarios 

### Use Case: Schema-Driven Action Propagation (SDAP) for Dynamic UI in Business Processes
See how [Use Case: Dynamic UI with SDAP](use-cases/01-dynamic-ui-integration.md) - How SDAP enables dynamic form generation without redeployment...

### Use Case: Backend Integration (Bidirectional Contracts) (Factory ↔ FRC)
Explore [factory-FRC integration with SDAP](use-cases/02-bidirectional-backend.md) for evolving contracts...

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
- **Schema-level Authorization**: verify user has permission to execute specific action types
- **Context Validation**: schemaHash is used only to verify immutability, not authentication

## Schema Evolution Strategy

### Versioning Approach
1. **Hash-Based Identification**: every unique schema produces a new hash
2. **Field-Based Composition**: Schemas are built from registered field names in Schema Registry, allowing controlled evolution
3. **Forward Compatibility**: new fields must be optional
   
### Migration Techniques
- **Lazy Migration**: Running processes complete with original schema version 
- **Dual Validation**: Old and new schemas coexist during transitions

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
| **[Examples](use-cases/)** | ✅ Complete | **Use cases and implementations** - Real-world examples and integration scenarios |
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

### Security Version Scheme Distribution (SVSD)

**[SVSD](https://github.com/IVoin-coder/security-version-scheme-distribution)** is an extension of the SDAP pattern that ensures deterministic, secure, and versioned delivery of action-schemas between producers and consumers in long-running processes. It preserves SDAP flexibility while eliminating risks of schema tampering, desynchronization, and unpredictable changes. 

### 🏗️ Related Architectural Patterns
Repository: [https://github.com/IVoin-coder/security-version-scheme-distribution](https://github.com/IVoin-coder/security-version-scheme-distribution)
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
