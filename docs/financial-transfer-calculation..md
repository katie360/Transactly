# Financial Transfer Calculations

## 1. Overview

This document explains how Transactly calculates account balances and records financial movements during internal account-to-account transfers.

It covers ledger balances, available balances, fund reservations, debit and credit entries, concurrent transfers and failed transaction handling.

## 2. Understanding Account Balances

Transactly distinguishes between ledger balance and available balance.

### 2.1 Ledger Balance

The ledger balance represents the actual financial balance recorded in an account.

It changes when financial transactions are successfully posted.

### 2.2 Available Balance

The available balance represents the funds currently available for new transactions.

It accounts for funds already reserved for pending transactions.

**Calculation:**

Available Balance = Ledger Balance - Active Fund Reservations

### Example

Account A has:
- Ledger balance: KSh 20,000
- Active reservations: KSh 2,000

Available balance:

KSh 20,000 - KSh 2,000 = KSh 18,000

The KSh 2,000 reservation reduces the available balance but does not change the ledger balance.

## 3. Successful Transfer Scenario

A customer transfers KSh 5,000 from Account A to Account B.

### Initial Account Balances

| Account | Ledger Balance | Active Reservations | Available Balance |
|---|---:|---:|---:|
| Account A | KSh 20,000 | KSh 2,000 | KSh 18,000 |
| Account B | KSh 8,000 | KSh 0 | KSh 8,000 |

### Transfer Processing

Transactly validates the transfer and confirms that Account A has sufficient available funds.

The system then:
1. Debits KSh 5,000 from Account A.
2. Credits KSh 5,000 to Account B.
3. Records the corresponding ledger entries.
4. Commits the financial changes atomically.

### Final Account Balances

| Account | Ledger Balance | Active Reservations | Available Balance |
|---|---:|---:|---:|
| Account A | KSh 15,000 | KSh 2,000 | KSh 13,000 |
| Account B | KSh 13,000 | KSh 0 | KSh 13,000 |

The existing KSh 2,000 reservation remains active because it belongs to a separate pending transaction.

## 4. Ledger Entries

Every successful transfer must produce corresponding debit and credit entries.

For the KSh 5,000 transfer:

| Account | Entry Type | Amount |
|---|---|---:|
| Account A | Debit | KSh 5,000 |
| Account B | Credit | KSh 5,000 |

The total debit must equal the total credit.

This maintains financial consistency and supports transaction traceability.

## 5. Concurrent Transfer Scenario

Consider Account A with the following balances:

- Ledger balance: KSh 20,000
- Active reservations: KSh 2,000
- Available balance: KSh 18,000

Two transfer requests arrive simultaneously:

| Transfer | Requested Amount |
|---|---:|
| Transfer 1 | KSh 12,000 |
| Transfer 2 | KSh 8,000 |

Both requests cannot succeed because their combined amount exceeds the available balance.

### Expected Outcome

Assuming Transfer 1 is processed first:

1. Transfer 1 succeeds and debits KSh 12,000.
2. Account A's ledger balance becomes KSh 8,000.
3. The KSh 2,000 reservation remains active.
4. Account A's available balance becomes KSh 6,000.
5. Transfer 2 fails because KSh 8,000 exceeds the remaining available balance.

The system must use database-level concurrency controls to prevent both requests from using the same initial available balance.

## 6. Fund Reservation Handling

Fund reservations ensure that money committed to pending transactions cannot be spent again.

A reservation:
- Reduces the available balance.
- Does not immediately change the ledger balance.
- Must be linked to its originating transaction.

### Reservation Lifecycle

**Active:** Funds are reserved while the transaction is pending.

**Consumed:** The reservation is applied as part of the transaction's financial processing.

**Released:** Funds become available again when the associated transaction is cancelled or fails.

A reservation belonging to one transaction must not be consumed or released by an unrelated transaction.

## 7. Failed Transaction Scenario

Suppose a transaction has reserved KSh 2,000 but eventually fails.

Transactly must:
1. Update the transaction status to Failed.
2. Release the associated KSh 2,000 reservation.
3. Restore the amount to the available balance.
4. Notify the customer.
5. Retain the original transaction and its failure details.

The original failed transaction must not be deleted or reused.

### Retry Handling

If the customer chooses to retry:

- Transactly creates a new transaction with a new transaction ID.
- A new idempotency key is generated.
- The new request goes through the normal validation process.
- The retry must satisfy all business rules, including sufficient available funds.

A new transaction ID does not bypass the original business validation requirements.

## 8. Financial Integrity Principles

Transactly must maintain the following principles:

1. **Balance Accuracy:** Ledger and available balances must be calculated correctly.
2. **Atomicity:** Related financial updates must succeed or fail together.
3. **Double-Entry Recording:** Every successful transfer must have corresponding debit and credit entries.
4. **Reservation Integrity:** Reserved funds must remain associated with their originating transactions.
5. **Concurrency Control:** Simultaneous requests must not result in overspending.
6. **Idempotency:** Repeated delivery of the same request must not execute the transfer twice.
7. **Traceability:** Financial records and transaction history must be retained.

## 9. Scope Considerations

The initial implementation will support internal account-to-account transfers using a shared SQL Server database.

Advanced financial operations such as transaction reversals, external settlement, transaction fees and cross-currency processing will be considered as future enhancements.