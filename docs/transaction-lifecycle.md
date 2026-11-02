# Transactly
## Transaction Lifecycle

### 1. Overview

The transaction lifecycle describes the stages an internal account transfer goes through, from customer initiation to completion or failure.

### 2. Transaction Stages

| Stage | Description |
|---|---|
| Initiation | Customer submits a transfer instruction. |
| Receipt | System receives and acknowledges the request. |
| Authentication | System verifies the customer's identity. |
| Authorisation | System checks whether the customer has permission to access and operate on the account. |
| Validation | System checks transaction details and business rules, including account status, transfer limits and available funds. |
| Reservation | Required funds are temporarily held to prevent them from being used by other transactions. |
| Processing | System coordinates the financial operations required to execute the transfer. |
| Posting | Debit and credit entries are recorded in the financial ledger. |
| Commit | Related financial changes are successfully committed to the database. |
| Confirmation | Customer receives the transaction outcome. |
| Reconciliation | Financial records are compared to identify and investigate discrepancies. |

### 3. Transaction Outcomes

A transaction may result in one of the following outcomes:

- **Successful:** All required financial operations have been completed and committed.
- **Failed:** The transaction could not be completed.
- **Pending:** The request has been received but is awaiting processing or another required action.
- **Processing:** The system is actively executing the transaction.

### 4. Failure Handling

Transactions may fail at different stages of their lifecycle.

- Authentication or authorisation failures prevent unauthorised operations.
- Validation failures prevent invalid instructions from being processed.
- Processing failures must be handled without compromising financial integrity.
- If a database transaction fails before commit, related financial changes must be rolled back.
- If the outcome of a transaction is uncertain, the system must establish its actual state before retrying.
- If a financial operation has already been committed and needs correction, an appropriate reversal or compensating operation may be required.

### 5. Key Processing Principles

**Atomicity:** The sender's debit and recipient's credit must succeed together or be rolled back together within the same database transaction.

**Idempotency:** Repeated submissions of the same instruction must not result in duplicate financial execution.

**Concurrency Control:** Simultaneous transactions must not cause incorrect balances or conflicting financial updates.

**Traceability:** Every transaction must have a unique reference and an identifiable processing history.

**Reconciliation:** Financial records must be independently compared to identify discrepancies.


### 6. Scope Considerations

The initial Transactly implementation focuses on internal transfers within one financial institution.

External clearing, interbank settlement and payment network processing are outside the initial scope.