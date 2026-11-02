# Domain Model

## 1. Overview

The domain model defines the core entities, their attributes and relationships within Transactly.

It represents the business concepts required to manage customers, accounts, internal transfers, financial records and transaction investigations.

## 2. Core Entities

### Customer
Represents a bank customer who owns one or more accounts.

Key attributes:
- CustomerId
- FullName
- Email
- PhoneNumber
- CreatedAt

### User
Represents an individual who can authenticate and access Transactly.

Key attributes:
- UserId
- CustomerId (nullable for non-customer users)
- Username
- PasswordHash
- IsActive
- CreatedAt

### Account
Represents a bank account belonging to a customer.

Key attributes:
- AccountId
- CustomerId
- AccountNumber
- AccountType
- Currency
- LedgerBalance
- AccountStatus
- CreatedAt

Available balance is determined using the ledger balance and active fund reservations.

### Transaction
Represents a transfer instruction and its processing outcome.

Key attributes:
- TransactionId
- ReferenceNumber
- SenderAccountId
- RecipientAccountId
- Amount
- Currency
- Description
- IdempotencyKey
- Status
- CreatedAt
- UpdatedAt

### Ledger Entry
Records the financial impact of a transaction.

Key attributes:
- LedgerEntryId
- TransactionId
- AccountId
- EntryType (Debit or Credit)
- Amount
- CreatedAt

Each internal transfer produces corresponding debit and credit entries.

### Transaction Event
Records significant transaction activities and state changes.

Key attributes:
- EventId
- TransactionId
- EventType
- PreviousState
- NewState
- Description
- CreatedAt

### Fund Reservation
Records funds temporarily reserved for a transaction.

Key attributes:
- ReservationId
- TransactionId
- AccountId
- Amount
- Status
- CreatedAt
- ReleasedAt

### Investigation Note
Stores notes added by operations officers while investigating transactions.

Key attributes:
- NoteId
- TransactionId
- OfficerUserId
- Note
- CreatedAt

Investigation notes are immutable once saved. Corrections are recorded as additional notes.

### Role
Defines the responsibilities assigned to a system user.

Key attributes:
- RoleId
- RoleName
- Description

### Permission
Defines the actions a user is allowed to perform.

Key attributes:
- PermissionId
- PermissionName
- Description

Users receive permissions through their assigned roles.

## 3. Entity Relationships

| Relationship | Description |
|---|---|
| Customer → Account | One customer can own multiple accounts. |
| Customer → User | A registered customer can have a linked user account. |
| User → Role | A user can be assigned one or more roles. |
| Role → Permission | A role can have multiple permissions. |
| Account → Transaction | An account can participate in multiple transactions as sender or recipient. |
| Transaction → Ledger Entry | One transaction can have multiple ledger entries. |
| Transaction → Transaction Event | One transaction can have multiple events. |
| Transaction → Fund Reservation | One transaction can have one or more reservations. |
| Transaction → Investigation Note | One transaction can have multiple investigation notes. |
| User → Investigation Note | One operations officer can create multiple investigation notes. |

## 4. Financial Data Principles

- Ledger entries provide a traceable record of financial movements.
- Each internal transfer must have corresponding debit and credit entries.
- Available balance accounts for active fund reservations.
- Financial updates must maintain atomicity and consistency.
- Posted ledger entries must not be silently modified or deleted.
- Reservations must be linked to their originating transactions.

## 5. Initial Implementation Scope

The first version will use a simplified banking model:

- Each account belongs to one customer.
- A customer can own multiple accounts.
- Registered customers have linked user accounts.
- Operations officers access the system through separate user accounts.
- Role-based permissions control access to system functions.
- Joint accounts and authorised representatives are excluded initially.

## 6. Future Enhancements

The domain model may later support:

- Joint account ownership.
- Authorised account representatives.
- Multiple account currencies.
- Transaction fees and additional ledger entries.
- Reversals and refunds.
- Advanced operational workflows.