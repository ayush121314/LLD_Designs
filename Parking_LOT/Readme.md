<img width="8924" height="6863" alt="UML_1" src="https://github.com/user-attachments/assets/1741f9ef-97b1-426a-91e1-bfa0a49f9537" />







# Parking Management System — Low Level Design

This repository contains the Low Level Design (LLD) for a **scalable Parking Management System**.
The design focuses on **clear separation of responsibilities, extensibility, and real-world modeling** rather than code implementation.

The goal of this project is to demonstrate:

* Design thinking
* Pattern application
* System decomposition
* Object collaboration

This is a design-first project aimed at interview readiness and architectural clarity.

Link to preview -> https://lnkd.in/gXDgEnHY

Detailed Sequence diagram-: 

Parking ->https://lnkd.in/dz4a3fh4

Unpark-> https://lnkd.in/dz4a3fh4

---

## System Scope

This system models the lifecycle of a vehicle in a parking facility:

* Vehicle entry and ticket creation
* Parking slot selection using strategy
* Ticket persistence
* Fee calculation using factory + decorator
* Payment processing with routing
* Parking exit and ticket closure
* Repository-based persistence
* Display update mechanism

---

## High-Level Flow

### Entry

1. Client requests parking.
2. ParkingManager selects a spot using ParkingStrategy.
3. ParkingSpot parks the vehicle.
4. TicketManager creates and saves a ticket.
5. Ticket is returned to the client.

### Exit

1. Client provides ticket ID.
2. TicketManager retrieves ticket.
3. Fee calculator is selected using factory.
4. Fee decorators apply dynamic pricing.
5. Payment routed to correct service.
6. Payment persisted.
7. Spot is freed.
8. Ticket is marked complete.

---

## Architecture Overview

### Core Orchestrator

* **ParkingManager**
  Central controller for entry and exit flows.

---

### Ticket System

**Entities**

* Ticket

**Manager**

* TicketManager

**Persistence**

* TicketRepository

**Responsibilities**

* Ticket creation
* Status updates
* Ticket lookup
* Payment reference tracking

---

### Parking System

**Core Models**

* ParkingLot
* ParkingFloor
* ParkingSpot
* SpotInfo

**Repositories**

* FloorRepository
* SpotInfoRepository

**Responsibilities**

* Floor management
* Spot metadata management
* Occupancy control
* Real-time availability tracking
* Display notifications

---

### Strategy System (Parking Spot Selection)

**Interface**

* ParkingStrategy

**Implementations**

* FirstAvailable
* BestFit

**Responsibility**

* Dynamically choose optimal parking spot without changing core logic.

---

### Fee Calculation System

Uses a combination of **Factory + Strategy + Decorator**.

**Entry Point**

* CalculateFee

**Factory**

* FeeCalcFactory (Singleton)

**Interface**

* FeeCalculator

**Base Calculators**

* LiveTicketFee
* ExpiredFee

**Decorator**

* TicketDecorator

**Decorators**

* WeekendFee
* RainFee
* SurchargeFee

**Responsibility**

* Construct pricing logic dynamically.
* Avoid condition-heavy fee calculation logic.
* Enable extension without modification.

---

### Payment System

**Entity**

* Payment

**Manager**

* PaymentManager

**Router (Factory)**

* PaymentRouter

**Services**

* UPIService
* CardService
* CashService

**Persistence**

* PaymentRepository

**Responsibilities**

* Idempotent payment handling
* Retry tracking
* Provider integration abstraction
* Payment state transitions

---

## Design Patterns Used

| Area                     | Pattern    |
| ------------------------ | ---------- |
| Parking strategy         | Strategy   |
| Payment routing          | Factory    |
| Fee calculator selection | Factory    |
| Fee logic extension      | Decorator  |
| Display updates          | Observer   |
| Persistence layer        | Repository |
| System coordination      | Facade     |
| FeeCalcFactory           | Singleton  |

---

## Directory-Style Flow Visualization

### Entry

```
Client
 └── ParkingManager
       ├── ParkingStrategy
       │     └── ParkingSpot (park)
       └── TicketManager
             └── TicketRepository (save)
```

### Exit

```
Client
 └── ParkingManager
       ├── TicketManager (getTicket)
       ├── FeeCalcFactory
       │     └── FeeCalculator (Decorators)
       ├── PaymentManager
       │     ├── PaymentRouter
       │     │     └── PaymentService
       │     └── PaymentRepository (save)
       └── ParkingSpot (unpark)
```

---

## Key Design Decisions

### Strong Separation of Responsibilities

Each system handles:

* One core responsibility
* One form of orchestration
* One persistence role

---

### Explicit Dependency Flow

* No cross-layer coupling
* No circular dependencies
* Managers use repositories, not vice-versa
* Strategies have no persistence

---

### Extensibility

* New pricing rules can be added as decorators
* New payment methods can be introduced without changing clients
* New parking allocation strategies can be added with zero core change

---

### Thread Safety Consideration

* ParkingSpot owns its locking mechanism.
* Concurrency contention is isolated per spot.
* No lock escalation at system level.

---

## What Is Intentionally Not Included

This design does not include:

* REST APIs
* Databases
* UI logic
* Authentication
* Infra setup

This is **pure LLD**, not an implementation project.

---


