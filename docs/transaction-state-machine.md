# Transaction State Machine

## 1. Overview

The transaction state machine defines the possible states of a transaction, the conditions for moving between states, and the transitions permitted during its lifecycle.

It ensures that transaction processing follows controlled and predictable paths, preserving financial accuracy, consistency and traceability.

## 2. Transaction States

| State | Description |
|---|---|
| **Initiated** | The customer has created and submitted a transaction instruction. |
| **Pending** | The system has received the instruction, but processing has not yet started. |
| **Processing** | Required checks have passed, and the system is executing the financial operations. |
| **Successful** | The required financial changes have been committed successfully. |
| **Rejected** | The instruction was declined before processing because it failed a required check. |
| **Failed** | Processing was attempted but could not be completed successfully. |

## 3. State Transitions

The expected transaction flow is:

```text
Initiated
    |
    v
Pending
    |
    +-----------> Rejected
    |
    v
Processing
    |
    +-----------> Successful
    |
    +-----------> Failed
```

### Allowed Transitions

| Current State | Next State | Condition |
|---|---|---|
| Initiated | Pending | Transaction instruction is received and acknowledged. |
| Pending | Processing | Required authentication, authorisation and business validation checks pass. |
| Pending | Rejected | A required check fails. |
| Processing | Successful | Financial updates are committed successfully. |
| Processing | Failed | Processing encounters an execution failure. |

## 4. Terminal States

The following states are terminal:

- **Successful:** The transaction has completed.
- **Rejected:** The instruction was declined before processing.
- **Failed:** The processing attempt did not complete successfully.

A terminal transaction does not automatically return to an earlier state.

Any correction to a successful transaction must follow a separate process, such as a reversal. A retry must also be handled without duplicating the original financial operation.

## 5. Failure and Retry Handling

Transaction failures must be handled according to their actual processing outcome.

- Rejected transactions may be resubmitted after the underlying issue is corrected.
- Failed transactions may be retried only after confirming that the previous attempt did not commit.
- Repeated requests must be handled using idempotency controls to prevent duplicate financial execution.
- Transactions with uncertain outcomes must be investigated before another financial execution is attempted.

Retries must preserve the original transaction history and maintain a clear relationship between the original instruction and subsequent attempts.

## 6. State Management Principles

- **Controlled transitions:** Transactions must only move through permitted state transitions.
- **Consistency:** Transaction states must accurately reflect the underlying financial outcome.
- **Immutability:** Completed transaction history must be preserved.
- **Idempotency:** Repeated requests must not result in duplicate financial execution.
- **Traceability:** State changes must be recorded for audit and investigation.
- **Atomicity:** Related financial updates must succeed together or be rolled back together.

## 7. Scope Considerations

This state machine defines the initial transaction processing model for internal account-to-account transfers.

Future enhancements may introduce additional states or workflows for reversals, manual reviews, reconciliation and transaction recovery.