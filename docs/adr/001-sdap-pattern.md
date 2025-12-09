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
    "condition": {
      "toDraft": {
        "value": false,
        "type": "Boolean",
        "required": false,
        "uiHint": "checkbox",
        "label": "Send to Draft",
        "description": "Return payment to draft status for additional review"
      },
      "toExecution": {
        "value": true,
        "type": "Boolean", 
        "required": false,
        "uiHint": "checkbox",
        "label": "Approve for Execution",
        "description": "Approve payment for immediate processing"
      },
      "toCancel": {
        "value": false,
        "type": "Boolean",
        "required": false,
        "uiHint": "checkbox", 
        "label": "Cancel Payment",
        "description": "Cancel this payment request entirely"
      }
    },
    "description": {
      "value": "",
      "type": "String",
      "required": true,
      "uiHint": "textarea",
      "label": "Comments",
      "description": "Provide additional details about your decision",
      "placeholder": "Enter your comments here...",
      "maxLength": 1000
    },
    "priority": {
      "value": "",
      "type": "Enum",
      "required": false,
      "uiHint": "select",
      "label": "Priority Level",
      "description": "Select the urgency level for this approval",
      "options": [
        {"value": "low", "label": "Low Priority"},
        {"value": "medium", "label": "Medium Priority"}, 
        {"value": "high", "label": "High Priority"},
        {"value": "critical", "label": "Critical"}
      ]
    },
    "attachments": {
      "value": [],
      "type": "Array",
      "required": false,
      "uiHint": "file-upload",
      "label": "Supporting Documents",
      "description": "Upload supporting documents (max 5 files)",
      "maxItems": 5,
      "allowedTypes": ["pdf", "jpg", "png", "doc", "docx"]
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
