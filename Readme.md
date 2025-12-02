<img width="8924" height="6863" alt="UML_1" src="https://github.com/user-attachments/assets/1741f9ef-97b1-426a-91e1-bfa0a49f9537" />


---

# Parking Management System — Low Level Design

This repository contains the **Low Level Design (LLD)** for a Parking Management System.
The focus of this design is not implementation but **structure, responsibility, and flow** — how a real system would be organized internally before writing code.

The objective was to model how parking, ticketing, fee calculation, and payment should work together in a clean and extensible system.

---

## Overview

The system is designed around three major service areas:

* Ticket Service
* Fee Calculation Service
* Payment Service

Each service owns its own logic, and no service directly interferes with the internal workings of another. A central manager coordinates the workflow instead of business logic being scattered across the system.

---

## Core Design Concepts

### Parking Structure

The parking system is modeled using a hierarchical structure:

* Parking Lot contains multiple Floors
* Each Floor contains multiple Parking Spots
* Each Spot has a size and availability status
* Vehicles are allocated spots based on compatibility (no hardcoding)

This makes it easy to scale from a single floor to multi-floor parking without redesign.

---

### Ticket Management

Ticket creation and lifecycle are handled by a dedicated service.

Responsibilities include:

* Ticket generation at entry
* Tracking entry time
* Recording allocated parking spot
* Maintaining ticket state from entry to exit

Tickets act as the primary reference throughout the system. All operations (fee calculation and payment) depend on ticket data instead of duplicating state elsewhere.

---

### Fee Calculation Service

Fee calculation is modeled as a standalone service.

The design uses:

* **Factory Pattern** to select the appropriate fee rule
* **Decorator Pattern** to layer pricing rules dynamically

This allows fee logic such as base pricing, weekend rates, surcharge policies, etc., to be added without modifying existing code.

The service is independent from ticket creation and payment processing. It only depends on ticket data and returns a computed amount.

---

### Payment Service

Payment handling is isolated in its own service layer.

The design supports:

* Multiple payment methods via a common interface
* Retry-safe workflows
* Failure handling without breaking ticket flow

A key part of the design is **idempotent payment processing**, which ensures that:

* Duplicate requests do not cause double charges
* Payment retries do not create inconsistent states
* Each transaction is uniquely identifiable

Payment is treated as a state-driven process instead of a single API call.

---

### Central Orchestration (ParkingManager)

Rather than allowing services to call each other directly, the system uses a central manager.

Responsibilities:

* Controls vehicle entry and exit flow
* Coordinates ticket creation
* Triggers fee calculation
* Initiates payment settlement

This prevents tight coupling between services and keeps control flow explicit and traceable.

---

### Persistence Layer (Repositories)

Each major entity is backed by a repository:

* Ticket Repository
* Payment Repository
* Parking Spot Repository
* Floor Repository

The domain logic never directly talks to storage. All persistence operations go through repositories, making the design easier to maintain and test.

---

## Design Goals

The design was built with these goals in mind:

* Clear separation of responsibilities
* Minimal coupling between modules
* Ease of extension without rewriting core logic
* Clean ownership of data
* Predictable system behavior under retries and failures

---

## UML Diagram

The UML diagram in this repository represents:

* Domain entities
* Service boundaries
* Design pattern usage
* Relationships between components
* Responsibility distribution

It is intended as a design artifact, not as a class-by-class code map.

---

## What This Project Focuses On

This repository does not include code.
It is purely focused on:

* Architecture
* Responsibility modeling
* Flow correctness
* Design clarity

---

## Why This Design Exists

The goal of this project was to practice building systems the way they exist in production, not just as academic diagrams.

It helped develop a clearer understanding of:

* How design evolves from failure scenarios
* Why boundaries matter
* How services should interact
* Where complexity truly lives in backend systems

---

