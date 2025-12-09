# 1. Schema-Driven Action Propagation (SDAP) Pattern

**Date:** 2025-12-08  
**Status:** Proposed  
**Author:** [Igor Soldatenko]  
**Participants:** requiredd for discussion  

## Context

Modern distributed systems face the problem of high coupling when passing actions between business process components. Typical issues:

1. **Rigid API contracts** (OpenAPI, gRPC proto, Avro) required synchronous updates of all system participants
2. **Slow process evolution** - changes in business logic required releasing new versions of UI and microservices
3. **Version incompatibility** - supporting multiple API versions becomes complex and expensive
4. **Validation duplication** - data validation rules are duplicated between frontend and backend

Traditional approaches (DTOs, event buses with fixed schemas) don't solve the problem in systems where:
- Business processes change more frequently than development cycles
- The number of different forms and actions is in the hundreds
- UI must adapt without deployment

## Decision

The **Schema-Driven Action Propagation (SDAP)** pattern for asynchronous interactions in a distributed system.

### Pattern Essence

Pass the data structure (schema) **together with the action itself**, rather than relying on predefined contracts.

### SDAP Event Format

```json
{
  "metadata": {
    "processId": "order-process-123",
    "actionType": "APPROVE_PAYMENT",
    "actionId": "action-456",
    "timestamp": "2024-01-15T10:30:00Z",
    "correlationId": "corr-789",
    "schemaHash": "sha256:abc123def456..."
  },
   "schema": {
    "//": "User action selection. All fields are optional, but at least one must be true.",
    "condition": {
      "toDraft": {
        "value": false,
        "type": "Boolean",
        "required": false
      },
      "toExecution": {
        "value": true,
        "type": "Boolean",
        "required": false
      },
      "toCancel": {
        "value": false,
        "type": "Boolean",
        "required": false
      }
    },    
    "//": "User action description. Always required.",
    "description": {
      "value": "Agreed and ready for implementation",
      "type": "String",
      "required": true
    }
  },
  "context": {
    "processName": "Order Processing",
    "stepName": "Payment Confirmation",
    "userId": "user-123",
    "deadline": "2024-01-16T10:30:00Z"
  }
}
```
***Note:*** schema can be null if the schemaHash is already known by the consumer.

## Consequences

### Positive
- **Loose coupling**: Components communicate through schemas rather than rigid contracts
- **Independent evolution**: Producers and consumers can evolve separately without breaking changes
- **Multi-version support**: Old and new schema versions can coexist in the system
- **Faster deployment**: New business processes can be deployed without UI or service updates
- **Centralized validation**: Single source of truth for data validation rules across all components

### Negative
- **Schema management complexity**: Need to store, version, and validate multiple schema versions
- **Performance overhead**: Additional processing for schema validation and hash comparison
- **Security considerations**: Schemas must be protected from tampering and unauthorized modifications
- **Tooling requirements**: Specialized tools needed for schema management and dynamic UI generation
- **Increased message size**: Events include schema data, increasing payload size

## Compliance Criteria

- [ ] Schemas are versioned using cryptographic hashes (schemaHash)
- [ ] UI dynamically generates forms from received schemas
- [ ] Schema Registry is implemented for field name and component management
- [ ] Security measures for schema integrity (signatures/validation) are in place
- [ ] Consumers cache schemas by hash to avoid repeated transmissions
- [ ] Support for backward/forward compatibility at process instance level

## Related Decisions

- [ADR-002: Schema Registry Implementation](002-schema-registry.md) - For centralized schema management
- [ADR-003: Security Model for SDAP](003-security-model.md) - For schema integrity protection
- [ADR-004: UI Rendering Strategy](004-ui-rendering.md) - For dynamic form generation
