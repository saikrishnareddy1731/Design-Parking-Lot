# Design-Parking-Lot

# Parking Lot System

A low-level design for a multi-level parking lot system in Java. This system supports multiple vehicle types, various spot sizes, fee strategies, and flexible parking allocation strategies. It is designed with extensibility, scalability, and maintainability in mind.

---

## Table of Contents

- [Features](#features)  
- [Requirements](#requirements)  
- [Core Entities](#core-entities)  
- [Design Patterns](#design-patterns)  
- [UML Class Diagram](#uml-class-diagram)  
- [Usage Example](#usage-example)

---

## Features

- Multi-level parking lot support  
- Multiple vehicle types: Bike, Car, Truck  
- Parking spot sizes: Small, Medium, Large  
- Automatic spot assignment with configurable strategies  
- Fee calculation strategies: Flat fee, Vehicle-size-based  
- Tracks entry and exit times using ParkingTicket  
- Singleton implementation for `ParkingLotSystem`  

---

## Requirements

### Functional Requirements

- Support multiple floors and configurable parking spots  
- Classify spots by size and enforce vehicle-spot compatibility  
- Automatic allocation of parking spots  
- Issue parking tickets and track vehicle entry/exit times  
- Calculate parking fees based on duration and strategy  
- Query real-time spot availability  

### Non-Functional Requirements

- **Concurrency:** Handle multiple simultaneous park/unpark operations  
- **Scalability:** Support large facilities with hundreds or thousands of spots  
- **Extensibility:** Easy to add new vehicle types, spot types, or fee strategies  

---

## Core Entities

| Entity | Responsibility |
|--------|----------------|
| `ParkingLotSystem` | Singleton class managing floors, tickets, fee strategies, and parking operations |
| `ParkingFloor` | Manages parking spots on a single floor and handles availability |
| `ParkingSpot` | Represents an individual parking space and its occupancy status |
| `Vehicle` | Base class for all vehicle types with license number and size |
| `Bike`, `Car`, `Truck` | Subclasses of `Vehicle` |
| `VehicleSize` | Enum for SMALL, MEDIUM, LARGE |
| `ParkingTicket` | Stores parking session metadata (vehicle, spot, entry timestamp) |
| `FeeStrategy` | Interface for calculating fees; supports multiple strategies |
| `FlatFeeStrategy` | Calculates fee based on fixed hourly rate |
| `VehicleBasedFeeStrategy` | Calculates fee based on vehicle size |
| `ParkingStrategy` | Interface for spot allocation strategies |
| `NearestFirstStrategy`, `BestFitStrategy`, `FartherFirstStrategy` | Implement different allocation strategies |

---

## Design Patterns

- **Singleton:** `ParkingLotSystem` ensures only one instance exists  
- **Strategy:** Used for `FeeStrategy` and `ParkingStrategy` to allow flexible behavior  
- **Facade:** `ParkingLotSystem` provides a simple interface for external clients  

---

## 🏗️ Architecture Overview
The system is organized around these key concepts:
- **ParkingLotSystem** manages floors, tickets, and the chosen strategies.
- **ParkingFloor** holds multiple **ParkingSpot** instances.
- **ParkingSpot** can park/unpark a **Vehicle** of a certain **VehicleSize**.
- **FeeStrategy** and **ParkingStrategy** are interfaces with interchangeable implementations.
- **Vehicle** is an abstract base with concrete types: **Car**, **Bike**, **Truck**.

---

## UML Class Diagram

```mermaid
classDiagram
    direction LR

    class ParkingLotSystem {
        -ParkingStrategy parkingStrategy
        -FeeStrategy feeStrategy
        -List~ParkingFloor~ floors
        -Map~String, ParkingTicket~ activeTickets
        +addFloor(ParkingFloor): void
        +parkVehicle(Vehicle): Optional~ParkingTicket~
        +unparkVehicle(String): Optional~double~
        +setFeeStrategy(FeeStrategy): void
        +setParkingStrategy(ParkingStrategy): void
        +getInstance(): ParkingLotSystem
    }

    class ParkingFloor {
        -int floorNumber
        -Map~String, ParkingSpot~ spots
        +findAvailableSpot(Vehicle): Optional~ParkingSpot~
        +addSpot(ParkingSpot): void
        +displayAvailability(): void
    }

    class ParkingSpot {
        -VehicleSize spotSize
        -String spotId
        -boolean isOccupied
        +boolean canFitVehicle(Vehicle)
        +void parkVehicle(Vehicle)
        +void unparkVehicle()
        +boolean isAvailable()
    }

    class ParkingTicket {
        -ParkingSpot spot
        -String ticketId
        -long entryTimestamp
        -long exitTimestamp
        -Vehicle vehicle
    }

    class Vehicle {
        <<abstract>>
        -VehicleSize size
        -String licenseNumber
    }
    class Car
    class Bike
    class Truck

    class VehicleSize {
        <<enumeration>>
        SMALL
        MEDIUM
        LARGE
    }

    class FeeStrategy {
        <<interface>>
        +double calculateFee(ParkingTicket)
    }
    class FlatRateFeeStrategy {
        +double RATE_PER_HOUR
        +double calculateFee(ParkingTicket)
    }
    class VehicleBasedFeeStrategy {
        +Map~VehicleSize, Double~ HOURLY_RATES
        +double calculateFee(ParkingTicket)
    }

    class ParkingStrategy {
        <<interface>>
        +Optional~ParkingSpot~ findSpot(List~ParkingFloor~, Vehicle)
    }
    class NearestFirstStrategy
    class FarthestFirstStrategy
    class BestFitStrategy

    class ParkingLotDemo {
        +main(String[])
    }

    %% Relationships
    ParkingLotSystem *-- ParkingFloor
    ParkingLotSystem --> ParkingTicket
    ParkingLotSystem ..> FeeStrategy
    ParkingLotSystem ..> ParkingStrategy

    ParkingFloor *-- ParkingSpot
    ParkingSpot --> Vehicle
    ParkingSpot --> VehicleSize
    ParkingTicket --> ParkingSpot
    ParkingTicket --> Vehicle
    Vehicle --> VehicleSize

    FeeStrategy <|.. FlatRateFeeStrategy
    FeeStrategy <|.. VehicleBasedFeeStrategy
    ParkingStrategy <|.. NearestFirstStrategy
    ParkingStrategy <|.. FarthestFirstStrategy
    ParkingStrategy <|.. BestFitStrategy

    Vehicle <|-- Car
    Vehicle <|-- Bike
    Vehicle <|-- Truck

    ParkingLotDemo ..> ParkingLotSystem

 %% ===== Relationship Lines =====
%% Composition (filled diamond) -----------------
ParkingLotSystem *-- ParkingFloor : composition
ParkingFloor *-- ParkingSpot : composition

%% Aggregation / Association (solid line) -------
ParkingLotSystem --> ParkingTicket : association
ParkingSpot --> Vehicle : association
ParkingSpot --> VehicleSize : uses
ParkingTicket --> ParkingSpot : association
ParkingTicket --> Vehicle : association
Vehicle --> VehicleSize : uses
ParkingLotDemo --> ParkingLotSystem : uses

%% Dependency (dashed line) ---------------------
ParkingLotSystem ..> FeeStrategy : dependency
ParkingLotSystem ..> ParkingStrategy : dependency

%% Interface Realization / Inheritance ----------
FeeStrategy <|.. FlatRateFeeStrategy : implements
FeeStrategy <|.. VehicleBasedFeeStrategy : implements
ParkingStrategy <|.. NearestFirstStrategy : implements
ParkingStrategy <|.. FarthestFirstStrategy : implements
ParkingStrategy <|.. BestFitStrategy : implements

%% Class Inheritance ----------------------------
Vehicle <|-- Car : extends
Vehicle <|-- Bike : extends
Vehicle <|-- Truck : extends
