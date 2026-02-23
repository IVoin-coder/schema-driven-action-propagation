# Use Case: Cross-Service Action Propagation
## Billing (Master) -> Notifier (Follower) 

## Situation Without SDAP (Classical Approach)
There are two services:
- **Billing** - confirms payments
- **Notifier** - sends notifications

In the classical model, Notifier contains hardcoded logic:
- on payment_confirmed
- if amount <= 100k -> send email
- if amount > 100k -> call the manager

When business rules change (for example, the threshold changes or a new channel is introduced), it becomes necessary to:
- modify the Notifier code
- introduce additional branching logic
- deploy a new version

Business logic becomes distributed across multiple services.

## Situation with SDAP

Under the SDAP model:
- Billing acts as the Master
- Notifier acts as the Follower
- admissible actions are computed exclusively by the Master

Billing does not send a simple event.
It sends a package containing:
- state context (SVSD)
- data
- an action schema (propagation)

Example:
```
{
  "context": {
    "state": "payment_confirmed",
    "version": "2.0"
  },
  "data": {
    "amount": 150000,
    "user_id": "u_99"
  },
  "action_propagation": {
    "execute_notification": {
      "type": "external_call",
      "target": "http://telephony-service/make-call",
      "payload": {
        "to": "{{manager_phone}}",
        "script": "High value payment received: 150000"
      }
    }
  }
}
```

## What Happens on the Notifier Side

Notifier:
- does not analyze amount
- does not decide which notification channel to use
- does not contain business conditions

Billing:
1. Receives the package
2. Validates the structure
3. Interprets action_propagation
4. Executes the described action

All admissibility logic resides in Billing.

## What This Approach Enables

### Rule Changes
If notification rules change, only the schema generation logic in Billing is modified.
Notifier’s code remains unchanged.

### Canary Deployment
Billing can send a new schema version to a subset of requests.
All services continue operating on the same codebase.

### No Policy Drift
The Follower does not infer admissibility from data.
It executes the propagated contract.

## Core Difference

In the classical model:
> The Follower interprets the event and makes a decision.

In SDAP:
> The Master computes the admissible action and propagates its execution contract.

## Capability Awareness Requirement

For SDAP to function correctly, the Master must be aware of the Follower’s declared capabilities.

The Master cannot propagate arbitrary actions.
It may only generate admissibility contracts that fall within the capability domain of the Follower.

In this example:
- Notifier declares supported action types (e.g., email, external_call, etc.)
- Billing computes admissibility only within that declared capability set

This ensures:
- no hidden execution assumptions
- no implicit coupling
- deterministic behavior

The Master computes what is admissible.
The Follower guarantees how it is executed, within its declared capability boundary.

* [SDAP readme](README.md) 
