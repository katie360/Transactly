# Business Rules

## 1. Overview

This document defines the business rules governing how Transactly processes internal account-to-account transfers.

These rules ensure that transfers are valid, authorised, accurate and traceable. They also define how the system handles failures, duplicate requests and concurrent transactions.

## 2. Transfer Validation Rules

Every transfer must satisfy the following conditions before processing:

### 2.1 Sufficient Available Funds
- The sender must have sufficient available funds to complete the transfer.
- Available balance is calculated using the ledger balance less active fund reservations.
- Transfers exceeding the available balance must be rejected.

### 2.2 Account Eligibility
- Both sender and recipient accounts must exist and be active.
- Frozen, closed or dormant accounts cannot participate in transfers in V1.

### 2.3 Valid Recipient
- The recipient account must exist and be active before processing begins.

### 2.4 Transfer Limits
- The minimum transfer amount is KSh 100.
- The maximum transfer amount is KSh 100,000 per transaction.
- These limits must be configurable rather than hardcoded.

### 2.5 Valid Transfer Amount
- The transfer amount must be greater than zero.
- The amount must not exceed the decimal precision supported by the currency.
- Invalid amounts must be rejected before processing.

### 2.6 Account Ownership and Authorisation
- Customers may only initiate transfers from accounts they own or are explicitly authorised to operate.
- The system must verify the customer's access rights before processing.

### 2.7 Currency Matching
- Sender and recipient accounts must use the same currency.
- Cross-currency transfers must be rejected in V1.

### 2.8 Self-Transfers
- Sender and recipient accounts must be different.
- Transfers between the same account must be rejected.

## 3. Transaction Processing Rules

### 3.1 Duplicate Transaction Prevention
- Every transfer request must have a unique idempotency key.
- Repeated requests using the same key and transaction details must return the original transaction result without executing the transfer again.
- Reusing an idempotency key with different transaction details must be rejected.

### 3.2 Atomic Transfer Processing
- Debiting the sender and crediting the recipient must happen as one atomic operation.
- Both financial updates must succeed together.
- If either operation fails before the transaction commits, all related financial changes must be rolled back.

### 3.3 Negative Balance Prevention
- An account's available balance must never become negative.
- The system must prevent customers from spending the same funds more than once.

### 3.4 Concurrency Control
- Simultaneous transfer requests must be handled safely.
- Database-level concurrency controls must ensure that competing transactions cannot overspend an account.

### 3.5 Ledger Integrity
- Every successful transfer must produce corresponding debit and credit ledger entries.
- The total debit amount must equal the total credit amount for the transfer.
- Ledger entries must not be silently modified or deleted.

## 4. Transaction Status and Failure Handling

### 4.1 Transaction Status
Transactions must follow the defined state transitions in the transaction state machine.

The system must distinguish between:
- Successful: Financial changes have been committed.
- Rejected: The request failed a required check before processing.
- Failed: Processing was attempted but could not be completed.
- Pending and Processing: The transaction has not reached a final outcome.

### 4.2 Failed Transactions
When processing fails:
- The transaction status must be updated to Failed.
- The customer must be informed that the transfer was unsuccessful.
- Any active fund reservation must be released.
- No incomplete debit or credit must remain.
- Failure details must be retained for investigation.

### 4.3 Transaction Reversals
- Completed transactions must not be deleted or edited to undo their financial impact.
- Reversals must be recorded as separate transactions linked to the original transaction.

## 5. Transaction History, Audit and Notifications

### 5.1 Transaction History
- Customers must be able to view their own transaction history.
- Authorised operations officers must be able to access relevant transaction records for monitoring and investigation.

### 5.2 Audit Trail
- Important transaction events must be recorded, including initiation, validation, rejection, processing, success and failure.
- Audit records must not be silently altered or deleted.
- Corrections must be recorded as additional events.

### 5.3 Transaction Confirmation
- Successful transfers must generate a confirmation containing the transaction reference, amount, currency, sender, recipient, date and time, and final status.
- Account details must be masked where appropriate.
- A transfer must only be confirmed as successful after its financial changes have been committed.

### 5.4 Transaction Notifications
- Customers must receive notifications for successful, rejected and failed transfers.
- Notifications must include relevant transaction details without exposing sensitive account information.
- In-app notifications are the initial notification channel.

## 6. V1 Scope Limitations and Future Enhancements

The following functionality is intentionally excluded from the initial implementation:

| Feature | Future Enhancement |
|---|---|
| Cross-currency transfers | Currency conversion, exchange-rate management, fees and FX ledger entries |
| Transaction fees | Fee calculation, deduction and recording |
| Daily transfer limits | Customer-specific and daily transaction limits |
| Transaction reversals | Controlled reversal workflows and approval processes |
| External notifications | SMS and email notifications |
| Joint accounts | Multiple account owners and authorised representatives |
| Advanced account rules | Configurable eligibility rules based on account type and customer profile |

## 7. Business Rule Enforcement

Business rules must be enforced on the backend rather than relying solely on frontend validation.

The system must ensure that all financial operations follow these rules regardless of how a transfer request is submitted.

These rules form the foundation for Transactly's transaction processing logic, database constraints, validation mechanisms and automated tests.