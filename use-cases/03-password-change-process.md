# Use Case: Password Change Process

This example demonstrates how SDAP governs action admissibility compared to a traditional hardcoded API approach.

## Traditional API Approach (Hardcoded Contract)
In a conventional implementation:
- The frontend statically defines a "Change Password" button.
- The request format is predefined in code (e.g., POST /api/v1/change-password).
- Required fields are hardcoded (e.g., login, new_password).

If the business later introduces additional requirements (e.g., old_password or sms_code):
- The frontend must be modified.
- The application must be rebuilt and redeployed.
- Version synchronization between client and server becomes mandatory.

In this model, action admissibility is implicitly defined by the interface contract.

## SDAP-Based Approach
Under SDAP, admissibility is not predefined in the client.
It is computed and propagated as an Action Schema.

### Step A - State Resolution

The Follower requests the current state of the "Security" process for a user.

The Master responds with:
- ProcessVersion
- State
- optional contextual information

Example:

```
State: Standard_Security
ProcessVersion: 2.1
```

### Step B - Contract Computation and Propagation

The Master computes admissible actions:

```
A = F(ProcessVersion, State, Context, Capabilities)
```

The resulting Action Schema may look like:

```
{
  "action": "change_password",
  "method": "POST",
  "fields": [
    { "name": "old_password", "type": "string", "required": true },
    { "name": "new_password", "type": "password_strength_v2", "required": true },
    { "name": "sms_code", "type": "digits", "length": 4 }
  ]
}
```

This schema is:

- computed deterministically

- bound to the current process instance

- fixed as an immutable artifact

- persisted for validation

### Step C - Interpretation and Execution

The Follower:
- interprets the Action Schema
- renders input fields dynamically
- validates user input according to schema rules
- submits the selected action to the Master

The Follower does not:
- define new fields
- remove required fields
- alter admissibility rules

It executes within the contract defined by the Master.

### Step D - Strict Validation

Upon receiving the action, the Master:
- validates it against the stored Action Schema
- verifies ProcessVersion binding
- rejects any request that does not match the computed contract

If a client attempts to send a request based on an outdated schema, it is rejected due to contract mismatch.

## Context-Based Variability
Because admissibility is computed, different users may receive different schemas under the same application:
- Standard users -> SMS confirmation
- High-security users -> biometric confirmation
- Internal users -> hardware token validation

From the Follower’s perspective, this is simply a different schema instance.

No conditional UI logic is required.

## Architectural Implications

Under SDAP:

- Clients implement schema interpretation, not predefined action paths.
- Contract evolution is centralized.
- Action admissibility changes without redeploying Followers.
- Desynchronization risks are eliminated at the contract level.

The system does not hardcode procedural paths.
It implements a deterministic mechanism for interpreting computed admissibility contracts.

* [SDAP readme](README.md) 
