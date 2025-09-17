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


## UML Class Diagram

```mermaid
classDiagram
    class ParkingLotSystem {
        -parkingStrategy: ParkingStrategy
        -floors: List~ParkingFloor~
        -liveTickets: List~ParkingTicket~
        +addFloor(ParkingFloor): void
        +findSpot(Vehicle): Optional~ParkingSpot~
        +parkVehicle(Vehicle, FeeStrategy): Optional~ParkingSpot~
        +setFeeStrategy(FeeStrategy): void
        +unparkVehicle(ParkingSpot): String
        +getInstance(): ParkingLotSystem
    }

    class ParkingLotDemo {
        +main(String[]): void
    }

    class ParkingFloor {
        -floorNumber: int
        -spots: Map~VehicleSize, ParkingSpot~
        +findSpot(Vehicle): Optional~ParkingSpot~
        +addSpot(ParkingSpot): void
        +displayAvailability(): void
    }

    class ParkingTicket {
        -spot: ParkingSpot
        -entryTimestamp: long
        -vehicle: Vehicle
    }

    class ParkingSpot {
        -size: VehicleSize
        -occupied: boolean
        -vehicle: Vehicle
        +canFitVehicle(Vehicle): boolean
        +isAvailable(): boolean
    }

    class Vehicle {
        -licenseNumber: String
    }

    class Bike {
    }

    class Car {
    }

    class Truck {
    }

    class VehicleSize {
        <<enumeration>>
        SMALL
        MEDIUM
        LARGE
    }

    class FeeStrategy {
        <<interface>>
        +calculateFee(ParkingTicket): double
    }

    class FlatFeeStrategy {
        -RATE_PER_HOUR: double
        +calculateFee(ParkingTicket): double
    }

    class VehicleBasedFeeStrategy {
        -HOURLY_RATES: Map~VehicleSize, Double~
        +calculateFee(ParkingTicket): double
    }

    class ParkingStrategy {
        <<interface>>
        +findSpotList(ParkingFloor, Vehicle): Optional~ParkingSpot~
    }

    class NearestFirstStrategy {
    }

    class BestFitStrategy {
    }

    class FartherFirstStrategy {
    }

    ParkingLotDemo --> ParkingLotSystem
    ParkingLotSystem --> ParkingStrategy
    ParkingLotSystem --> ParkingFloor
    ParkingLotSystem --> ParkingTicket
    ParkingLotSystem ..> FeeStrategy
    ParkingFloor --> ParkingSpot
    ParkingSpot --> Vehicle
    ParkingSpot --> VehicleSize
    ParkingTicket --> ParkingSpot
    ParkingTicket --> Vehicle
    ParkingTicket --> ParkingStrategy
    FeeStrategy <|-- FlatFeeStrategy
    FeeStrategy <|-- VehicleBasedFeeStrategy
    ParkingStrategy <|-- NearestFirstStrategy
    ParkingStrategy <|-- BestFitStrategy
    ParkingStrategy <|-- FartherFirstStrategy
    Vehicle <|-- Bike
    Vehicle <|-- Car
    Vehicle <|-- Truck
    Vehicle --> VehicleSize

