
# 🏧 ATM System — Low Level Design (LLD)

<img width="14743" height="12397" alt="UML_ATM" src="https://github.com/user-attachments/assets/b5388755-eeae-435f-855d-22fc489246eb" />

This repository contains the **Low Level Design (LLD)** for a production-grade **ATM Machine System**.

The design emphasizes:

* Clear separation of orchestration and business logic
* Transaction-safe processing and rollback handling
* Extensible transaction flows
* Clean hardware abstraction
* Pattern-oriented design
* Context-driven execution

This project is **design-focused**, not an implementation project.
It serves as a reference architecture for **interview preparation** and understanding **real-world system modelling**.

Live Here -: https://lnkd.in/gcXv6Vdx

Detailed Sequence diagram
Withdraw-: https://lnkd.in/g_qaVHEr
Balance Check-> https://lnkd.in/g9TvhxDD
Deposit-> https://lnkd.in/g6aMG8Mf


---

## 🎯 System Scope

This system models the **complete life cycle of an ATM transaction**:

* Card validation and PIN verification
* Transaction routing (Withdraw, Deposit, Balance Enquiry)
* Chain-based handler execution
* Bank system interaction
* Cash planning and denomination selection
* Hardware integration
* Receipt generation
* Logging and journaling
* Safe transaction completion and rollback

The design is extensible and flexible enough to support:

* New transaction types
* Hardware replacement
* Additional verification steps
* Fee or rule injection

---

## 🧩 High Level Flow

### ▶ Withdraw / Deposit

1. User interacts with ATM
2. ATM forwards request to `HandleManager`
3. `TransactionFactory` selects correct Strategy
4. Strategy builds corresponding handler chain
5. Chain validates and prepares data
6. Database transaction begins
7. Bank balance is updated
8. Receipt is prepared
9. Commit succeeds
10. Hardware is triggered to dispense / accept cash
11. Receipt is printed
12. Result returned to user

---

### ▶ Balance Enquiry

1. User requests balance
2. Card and PIN validated
3. Bank queried (read-only)
4. Receipt prepared
5. Result returned
6. No rollback needed

---

## 🏗 Architecture Overview

---

### 🎛 Central Orchestrator

#### ATM

Entry point from the user interface.

Responsibilities:

* Capture request
* Forward it to HandleManager
* Display result

---

#### HandleManager (Singleton)

Coordinates transaction execution.

Responsibilities:

* Extract transaction type
* Delegate to `TransactionFactory`
* Execute selected `Strategy`

---

### 🧠 Strategy System

Each transaction type is modeled as a **Strategy**.

#### Interface

* `TransactionStrategy`

#### Implementations

* `WithdrawStrategy`
* `DepositStrategy`
* `BalanceEnquiryStrategy`

Responsibilities:

* Build the correct handler chain
* Start transaction
* Commit or rollback
* Trigger hardware after commit
* Build final ATM result

---

### 🔗 Chain of Responsibility (Handlers)

Each transaction runs inside a **Handler Chain**.

Examples:

```
Withdraw:
CardCheck → PinCheck → BalanceCheck → CashPlanner → BalanceUpdater → ReceiptBuilder → Logger

Deposit:
CardCheck → PinCheck → AddCash → BalanceUpdater → ReceiptBuilder → Logger

Balance Enquiry:
CardCheck → PinCheck → BalanceCheck → ReceiptBuilder → Logger
```

Each handler:

* Receives context
* Performs one small operation
* Returns success / failure
* Does not call hardware directly

---

### 📦 Context Object

All request data flows through:

## `AtmContext`

```txt
cardNo
pinNo
queryType
amount
accountBalance
cashBlock
receiptData
success
message
```

Context acts as:

* Execution state carrier
* Message bus
* Transaction snapshot

Handlers update the context and Strategy interprets it.

---

### 🏦 Bank System Integration

Interactions with the bank are abstracted via:

* `BankService`
* `AccountRepository`

Responsibilities:

* Read balance
* Update balance
* Perform banking transaction with rollback

All updates happen inside **bank-side transactions**.

---

### 💸 Cash Dispensing System

The ATM uses a **Denomination Chain**:

```
DispenserManager
 └── DispenseHandler
       ├── ThousandHandler
       ├── HundredHandler
       └── FiftyHandler
```

Responsibilities:

* Compute optimal note combination
* Populate `cashBlock`
* Validate availability

No physical hardware is triggered inside the chain.

---

### 📄 Receipt System

Handlers generate receipt content **before commit**.

Structures:

#### ReceiptData

```
cardNoMasked
transactionType
amount
balance
timestamp
status
```

Receipts are printed **only after successful commit**.

---

### ⚙ Hardware Abstraction

All physical interactions are wrapped by:

## `AtmHardwareRepo`

Controls:

* Cash dispenser
* Receipt printer
* Display unit

Hardware is triggered **only after commit** to avoid inconsistencies.

---

### 💾 Repository Layer

Handles persistence:

| Repository            | Purpose  |
| --------------------- | -------- |
| AccountRepository     | Banking  |
| AtmJournalRepository  | Logging  |
| TransactionRepository | Auditing |

Repositories:

* Never decide flow
* Never talk to UI
* Only persist

---

## 🧱 Design Patterns Used

| Area                  | Pattern                 |
| --------------------- | ----------------------- |
| Transaction switching | Strategy                |
| Handler chaining      | Chain of Responsibility |
| Transaction creation  | Factory                 |
| Dispenser chain       | Chain of Responsibility |
| Hardware layer        | Facade                  |
| Logging & persistence | Repository              |
| Central manager       | Singleton               |
| Transaction control   | Unit of Work            |
| Result transport      | Context Object          |

---

## 🧠 Rollback Strategy

Rollback happens in two layers:

### ATM Layer

Controls:

* Local journal
* Receipt preparation
* Transaction life cycle

### Bank Layer

Controls:

* Account state
* Transaction commit
* Reversal logic

If `BalanceUpdater` fails:

* Bank rolls back
* ATM rolls back
* No cash dispensed

If ATM fails after bank success:

* Reconciliation logic handles it (out of scope)

---

## 🌱 Extensibility

You can easily add:

* A new transaction type → new Strategy
* New validation logic → new Handler
* New hardware → new Adapter
* New denomination → add a handler
* Multi-currency support
* Rules engine
* Biometric validation

With **zero changes to core flow**.

---

## 🚫 Out of Scope (Intentionally)

This design does NOT include:

* REST API
* Networking
* UI logic
* Encryption
* Hardware drivers
* Hosting
* Database schemas

This is a **pure Low Level Design exercise**.

---
