# Schema-Driven Action Propagation (SDAP)

**SDAP** is an architectural discipline a contract-based interaction model for Master–Follower systems, in which the set of admissible actions is deterministically computed from the process version and state, fixed as an immutable artifact, and strictly validated upon execution.

> **Core Idea:** Coordination of interacting systems where the controlling system (Master) generates a schema of admissible actions and their requirements for the current process context, and the dependent system (Follower) interprets that schema and executes actions strictly within its boundaries.

> **Core Principle:** 
The admissibility of an action is determined not by the interface,
but by a computed contract derived from the process version and state.

## Scope of the Discipline

SDAP applies to systems where:
- processes evolve under version control
- a controlling side (Master) exists
- an executing side (Follower) exists
- behavior is defined by process logic
- admissible actions depend on the current state
- strict reproducibility and evolution control are required

SDAP is independent of:
- technology stack
- transport protocol
- data storage mechanism
- Follower implementation details

## Table of Contents

- [Motivation ](#motivation)
- [Core Concepts](#core-concepts)
- [Discipline Rules](#discipline-rules)
- [Example](#example)
- [Benefits](#benefits) 
- [Related Architectural Patterns](#related-architectural-patterns) 
- [License](#license)

 
## Motivation

- Separation of process ownership and system implementation (Master–Follower)
- Dynamic yet strictly computable admissibility of actions
- Centralized contract eliminating inconsistencies
- Controlled process evolution through versioning
- Absence of soft/parallel compatibility between versions

---

## Core Concepts

### Master-Follower

##### ***Master*** 
- owns the process
- computes admissible actions
- produces the Action Schema
- validates action execution  

##### ***Follower***
- receives the schema
- interprets it
- executes actions within its constraints
- implements its own internal logic
- does not modify the contract

### Action Schema
- A formalized set of actions and rules
- ProcessVersion defines the function for computing the schema, while a concrete schema instance is fixed for a specific process state
- An immutable artifact
- Used for idempotency, auditing, reproducibility, and desynchronization protection

### Version Model
Each ProcessVersion defines:
- the process model
- the function for computing admissible actions
- validation rules
- the structure of the Action Schema

Any change to these elements requires a new ProcessVersion.

### Contract Computation

```text
A = F(ProcessVersion, State, Context, Capabilities)
```
#### where
- F — deterministic and immutable within a ProcessVersion. Any change to F requires a new ProcessVersion
- A — the set of admissible actions
- ProcessVersion — process version
- State — current process state
- Context — execution context (e.g., user permissions)
- Capabilities — technical capabilities of a specific type of Follower known to the Master

---

## Discipline Rules

1. **Version Binding** - The process version defines the computation model of the schema. Any change to the process model or function F requires a new ProcessVersion.

2. **Immutable Instance Binding** - The schema for a specific process instance is immutable once fixed.

3. **Strict Validation** - Every action is validated by the Master against the computed schema.

4. **Contract Persistence** - The schema is stored as a durable artifact.

5. **No Cross-Version Compatibility** - Soft или parallel compatibility не допускаются.

6. **Separation of Responsibility** - The Master governs the contract; the Follower interprets it.

---

### Non-Goals
SDAP does not:

- define the internal business logic of the Follower
- handle UI rendering
- define transport interaction mechanisms
- describe scaling strategies

SDAP governs action admissibility, not action implementation.

 ---

## Example 

#### UI-oriented example
```
{
  "processInstanceId": "12345",
  "processVersion": "v2",
  "schemaHash": "abcd1234",
  "actions": {
    "approve": { "type": "Boolean", "required": true },
    "reject": {
      "type": "Boolean",
      "required": true,
      "rules": {
        "master": { "detailsTask": { "type": "String", "required": true } },
        "follower": { "uploadFile": true }
      }
    }
  }
}
```
- The Master generates the Action Schema based on the known capabilities of a specific Follower type.
- The Follower receives the schema.
- It interprets and applies the schema rules, without adding new rules or modifying action admissibility, and executes the contract within its implementation boundaries.
- It performs application logic or renders the interface.
- It sends the selected action back to the Master.

The Master validates the action based on the version and the schema.

---
## Benefits
- Determinism and reproducibility
- Centralized contract control
- Implementation flexibility for the Follower without violating rules
- Controlled process evolution
- Elimination of cross-system desynchronization

---
 
## Related Architectural Patterns

### Related Concepts

SDAP is not derived from other patterns and is not an extension of them.
However, it intersects with several architectural directions at the conceptual level.

- **[Security Version Scheme Distribution (SVSD)](https://github.com/IVoin-coder/security-version-scheme-distribution)** - Deterministic, secure, and versioned distribution of action schemas between producer and consumer systems. Preserves SDAP flexibility while reducing risks of unauthorized access, desynchronization, and unpredictable changes.

- **[Process orchestration](https://camunda.com/process-orchestration/)**  - An orchestrator manages process flow and state transitions, while SDAP formalizes action admissibility as a computed contract.
Difference - orchestration governs transitions; SDAP governs admissibility.

- **[CQRS (Command Query Responsibility Segregation)](https://martinfowler.com/bliki/CQRS.html)** — Separates reads and writes. SDAP may be used to strictly validate commands.
Difference - SDAP formalizes the admissible command set as a function of version and state.

- **[Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)** — EDA distributes and reacts to events as the primary integration mechanism.
SDAP governs the computed contract of admissible actions derived from process version and state, while events may serve as triggers for recalculating that contract.

- **[Self-Describing Messages](https://dev.to/cadienvan/event-carried-state-transfer-a-pattern-for-distributed-data-management-in-event-driven-systems-165h)** — Messages carry metadata for interpretation. In SDAP, the Action Schema acts as a self-contained admissibility contract.

---    
## License

**Distributed under the MIT License.** See the `LICENSE` file in the repository for details.

---

**👤 Author**: [Igor Soldatenko]  
**Repository**: https://github.com/IVoin-coder/schema-driven-action-propagation  
