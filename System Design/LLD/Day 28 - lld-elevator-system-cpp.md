# LLD: Design an Elevator System

*Phase 3 — Low-Level Design (LLD) / OOD. A zero-to-mastery, interview-style walkthrough, with C++ code.*

---

## Table of Contents
1. [What Are We Actually Building?](#1-what-are-we-actually-building)
2. [Step 1: Clarify Requirements](#2-step-1-clarify-requirements)
3. [Step 2: Identify the Core Entities](#3-step-2-identify-the-core-entities)
4. [Step 3: The Elevator's State Machine](#4-step-3-the-elevators-state-machine)
5. [Step 4: Modeling Requests](#5-step-4-modeling-requests)
6. [Step 5: The Core Challenge — Deciding Which Elevator Responds](#6-step-5-the-core-challenge--deciding-which-elevator-responds)
7. [Step 6: The Core Challenge — How One Elevator Processes Its Requests](#7-step-6-the-core-challenge--how-one-elevator-processes-its-requests)
8. [Step 7: The Elevator Class](#8-step-7-the-elevator-class)
9. [Step 8: The ElevatorController — Tying It Together](#9-step-8-the-elevatorcontroller--tying-it-together)
10. [Step 9: The Full Class Diagram](#10-step-9-the-full-class-diagram)
11. [Step 10: Putting It All Together — a Working Example](#11-step-10-putting-it-all-together--a-working-example)
12. [Step 11: Extending the Design](#12-step-11-extending-the-design)
13. [How to Walk Through This in an Interview](#13-how-to-walk-through-this-in-an-interview)
14. [Quick Recall Cheat Sheet](#14-quick-recall-cheat-sheet)

---

## 1. What Are We Actually Building?

A system controlling **multiple elevators** in a building: people press buttons (inside a car, or on a floor to call one), and the system decides which elevator responds, in what order, and how each elevator moves and stops along the way.

```mermaid
flowchart LR
    A["Person on Floor 5<br/>presses UP button"] --> B["System decides:<br/>WHICH elevator should respond?"] --> C["That elevator moves<br/>to Floor 5, opens doors"]
```

This problem is a notch harder than the Parking Lot — it isn't just "manage a static collection of resources," it involves genuine **stateful behavior over time** (an elevator is always in some state — moving, idle, doors open) and a real **scheduling/optimization decision** (which elevator should take this request, and in what order should one elevator serve multiple requests). This makes it an excellent vehicle for practicing the **State Pattern** and multi-step algorithmic thinking within LLD.

---

## 2. Step 1: Clarify Requirements

### Functional Requirements
- Support **multiple elevators** in a building with **N floors**.
- A person can request an elevator from a floor (specifying **direction**: up or down).
- A person inside an elevator can select a **destination floor**.
- The system must decide **which elevator** should service each external request.
- Each elevator must process its assigned requests in a **sensible order** (not just first-come-first-served, which could cause a lot of unnecessary back-and-forth travel).
- Doors open/close at each stop.

### Non-Functional / Design Requirements
- **Extensible dispatch/scheduling logic** — the algorithm for choosing which elevator responds should be swappable (recall the Strategy pattern from the Design Patterns topic) without changing the rest of the system.
- **Thread-safety** — multiple floor/car button presses can happen simultaneously from different physical locations (echoing the same concurrency theme from the Parking Lot topic).

---

## 3. Step 2: Identify the Core Entities

```mermaid
flowchart TB
    A["Elevator"]
    B["ElevatorController"]
    C["Request"]
    D["Direction (enum)"]
    E["ElevatorState (enum)"]
    F["SchedulingStrategy"]
```

| Entity | Responsibility |
|---|---|
| `Elevator` | Represents one physical elevator car — its current floor, state, and the requests it needs to serve |
| `ElevatorController` | The overall system — receives all requests, decides which elevator handles each |
| `Request` | Represents one pending stop an elevator needs to make |
| `SchedulingStrategy` | Decides which elevator should be assigned a given external request |

---

## 4. Step 3: The Elevator's State Machine

An elevator is always in exactly one of a small number of well-defined states, and it transitions between them based on events — this is a textbook fit for explicitly modeling a **state machine**, which the UML topic's notation can represent directly.

```mermaid
stateDiagram-v2
    [*] --> IDLE
    IDLE --> MOVING_UP: request above current floor
    IDLE --> MOVING_DOWN: request below current floor
    MOVING_UP --> DOORS_OPEN: reached a requested floor
    MOVING_DOWN --> DOORS_OPEN: reached a requested floor
    DOORS_OPEN --> IDLE: doors close, no more requests
    DOORS_OPEN --> MOVING_UP: doors close, more requests above
    DOORS_OPEN --> MOVING_DOWN: doors close, more requests below
```

```mermaid
flowchart TB
    A["Why model this explicitly as states,<br/>rather than just a few boolean flags<br/>like 'isMoving' and 'goingUp'?"] --> B["Prevents INVALID combinations —<br/>e.g. a plain boolean design could<br/>accidentally represent 'moving AND<br/>doors open' at the same time,<br/>which should be IMPOSSIBLE"]
```

```cpp
enum class ElevatorState { IDLE, MOVING_UP, MOVING_DOWN, DOORS_OPEN };
```

**Why an explicit state enum matters:** using distinct, named states (rather than several independent boolean flags) makes **invalid states structurally impossible to represent** — you literally cannot accidentally set the elevator to "moving" and "doors open" simultaneously, because it's always in exactly *one* named state at a time. This is a small but genuinely valuable design habit worth calling out explicitly in an interview.

---

## 5. Step 4: Modeling Requests

Two distinct kinds of requests exist in a real elevator system, and it's worth being precise about the difference:

```mermaid
flowchart TB
    A["External Request<br/>(a floor button, outside the elevator)<br/>— has a DIRECTION (up/down), NOT a specific destination"]
    B["Internal Request<br/>(a button inside the car)<br/>— has a SPECIFIC destination floor,<br/>no direction needed (already implied)"]
```

```cpp
enum class Direction { UP, DOWN, NONE };

class Request {
public:
    int floor;
    Direction direction;  // relevant for EXTERNAL requests; NONE for internal ones

    Request(int f, Direction d = Direction::NONE) : floor(f), direction(d) {}
};
```

---

## 6. Step 5: The Core Challenge — Deciding Which Elevator Responds

This is the first of two genuinely hard problems in this design. When someone on Floor 5 presses "up," which of the building's N elevators should actually respond?

### A naive approach: nearest elevator
Simply pick whichever elevator is physically closest to the requesting floor.

```mermaid
flowchart TB
    A["Problem with pure 'nearest elevator':<br/>doesn't account for DIRECTION or CURRENT STATE"] --> B["Example: the nearest elevator might<br/>be moving DOWN, while the request is for<br/>someone wanting to go UP from a floor<br/>ABOVE the elevator's current position —<br/>sending it there would mean the elevator<br/>has to overshoot, turn around, and come back"]
```

### A better approach: scoring/ranking each elevator
Consider each elevator's current floor, direction, and state, and pick whichever one can serve the request with the **least additional travel/deviation**.

```mermaid
flowchart TB
    A["For each elevator, ask:"] --> B["Is it IDLE? → can go anywhere, low cost"]
    A --> C["Is it moving TOWARD the request,<br/>in a compatible direction? → good candidate,<br/>cost based on distance"]
    A --> D["Is it moving AWAY from the request,<br/>or in the opposite direction? →<br/>high cost (will need to finish its current<br/>trip, then come back)"]
```

This is precisely where the **Strategy Pattern** (Design Patterns topic) applies directly: define a `SchedulingStrategy` interface, and let the specific algorithm (nearest-elevator, load-balanced, zone-based, etc.) be swapped without changing the `ElevatorController`.

```cpp
class SchedulingStrategy {
public:
    virtual Elevator* selectElevator(std::vector<Elevator*>& elevators, const Request& request) = 0;
    virtual ~SchedulingStrategy() {}
};

class NearestElevatorStrategy : public SchedulingStrategy {
public:
    Elevator* selectElevator(std::vector<Elevator*>& elevators, const Request& request) override {
        Elevator* best = nullptr;
        int bestScore = INT_MAX;

        for (Elevator* elevator : elevators) {
            int score = computeCost(*elevator, request);
            if (score < bestScore) {
                bestScore = score;
                best = elevator;
            }
        }
        return best;
    }

private:
    int computeCost(const Elevator& elevator, const Request& request) {
        int distance = std::abs(elevator.getCurrentFloor() - request.floor);

        // IDLE elevators are cheap/flexible — can go anywhere
        if (elevator.getState() == ElevatorState::IDLE) {
            return distance;
        }

        // Moving in a COMPATIBLE direction, toward the request → still reasonable
        bool movingToward =
            (elevator.getState() == ElevatorState::MOVING_UP && request.floor >= elevator.getCurrentFloor()) ||
            (elevator.getState() == ElevatorState::MOVING_DOWN && request.floor <= elevator.getCurrentFloor());

        if (movingToward) {
            return distance;
        }

        // Moving AWAY or in the wrong direction → penalize heavily,
        // since it will have to finish its current trip before backtracking
        return distance + 1000;
    }
};
```

**Why Strategy fits perfectly here:** exactly as with the Parking Lot's pricing logic, `ElevatorController` never needs to know the specific scoring algorithm — it just calls `strategy->selectElevator(...)`. A building could swap in a more sophisticated strategy (e.g., one that also considers current passenger load) with **zero changes** to the controller itself.

---

## 7. Step 6: The Core Challenge — How One Elevator Processes Its Requests

The second hard problem: once an elevator has multiple pending stops (say, floors 3, 7, and 2, requested in that arrival order), in what order should it actually visit them?

### Naive: FIFO (first requested, first served)
```mermaid
flowchart TB
    A["Requests arrive in order: 3, 7, 2"] --> B["FIFO: go to 3, THEN 7, THEN 2"]
    B --> C["❌ Wasteful! If currently on floor 1 and moving up,<br/>going 1→3→7→2 means passing floor 2<br/>on the way to 7, then having to come ALL<br/>the way back down to floor 2 afterward"]
```

### Better: the SCAN algorithm (a.k.a. the "elevator algorithm")
Continue moving in the **current direction**, serving every pending request along the way, until there are no more requests in that direction — **then** reverse direction.

```mermaid
flowchart TB
    A["Currently on floor 1, moving UP<br/>Pending requests: 3, 7, 2"] --> B["Since moving UP: serve requests<br/>IN ORDER along the way UP: 2, then 3, then 7<br/>(NOT the order they were requested in —<br/>the order they're PHYSICALLY encountered)"]
    B --> C["Once no more requests above,<br/>THEN reverse and head down<br/>for any requests below"]
```

```mermaid
sequenceDiagram
    participant Elevator
    Note over Elevator: On floor 1, requests pending: [2, 3, 7]
    Elevator->>Elevator: Move up, reach floor 2 → STOP (was requested)
    Elevator->>Elevator: Move up, reach floor 3 → STOP (was requested)
    Elevator->>Elevator: Move up, reach floor 7 → STOP (was requested)
    Note over Elevator: No more requests above → become IDLE,<br/>or reverse if requests exist below
```

This algorithm — always finishing all requests in the current direction before reversing — is exactly why it's literally named the **elevator algorithm** in computer science; it's also used more broadly for disk scheduling, since the underlying problem (minimize wasted travel while visiting a set of points along a line) is structurally identical.

```cpp
#include <set>

class Elevator {
private:
    int currentFloor;
    ElevatorState state;
    std::set<int> upRequests;    // pending stops while moving up, kept SORTED
    std::set<int> downRequests;  // pending stops while moving down, kept SORTED

public:
    Elevator(int startFloor) : currentFloor(startFloor), state(ElevatorState::IDLE) {}

    void addRequest(int floor) {
        if (floor > currentFloor) {
            upRequests.insert(floor);
        } else if (floor < currentFloor) {
            downRequests.insert(floor);
        }
        if (state == ElevatorState::IDLE) {
            state = (floor > currentFloor) ? ElevatorState::MOVING_UP : ElevatorState::MOVING_DOWN;
        }
    }

    // Called repeatedly (e.g. once per "tick") to advance the elevator by one floor/step
    void step() {
        if (state == ElevatorState::MOVING_UP) {
            if (!upRequests.empty()) {
                currentFloor++;
                if (upRequests.count(currentFloor)) {
                    openDoors();
                    upRequests.erase(currentFloor);
                }
            }
            if (upRequests.empty()) {
                state = downRequests.empty() ? ElevatorState::IDLE : ElevatorState::MOVING_DOWN;
            }
        } else if (state == ElevatorState::MOVING_DOWN) {
            if (!downRequests.empty()) {
                currentFloor--;
                if (downRequests.count(currentFloor)) {
                    openDoors();
                    downRequests.erase(currentFloor);
                }
            }
            if (downRequests.empty()) {
                state = upRequests.empty() ? ElevatorState::IDLE : ElevatorState::MOVING_UP;
            }
        }
    }

    void openDoors() {
        state = ElevatorState::DOORS_OPEN;
        std::cout << "Elevator doors open at floor " << currentFloor << "\n";
        // (doors would close again after a delay, resuming MOVING_UP/DOWN)
    }

    int getCurrentFloor() const { return currentFloor; }
    ElevatorState getState() const { return state; }
};
```

- **Why `std::set` specifically:** it keeps pending requests automatically **sorted**, which is exactly what the SCAN algorithm needs — the elevator always serves the *nearest* pending stop in its current direction next, and a sorted set gives that for free (`*upRequests.begin()` is always the next stop while moving up).

---

## 8. Step 7: The Elevator Class

(Already shown in full above — `Elevator` bundles its state, position, and pending requests, encapsulating exactly how it decides its next move via `step()`, without external code needing to understand the SCAN logic itself.)

---

## 9. Step 8: The ElevatorController — Tying It Together

```mermaid
classDiagram
    class ElevatorController {
        -vector~Elevator~ elevators
        -SchedulingStrategy strategy
        +requestElevator(floor, direction) void
        +tick() void
    }
    ElevatorController "1" *-- "many" Elevator : manages
    ElevatorController --> SchedulingStrategy : uses
```

```cpp
class ElevatorController {
private:
    std::vector<Elevator*> elevators;
    SchedulingStrategy* strategy;

public:
    ElevatorController(SchedulingStrategy* s) : strategy(s) {}

    void addElevator(Elevator* elevator) {
        elevators.push_back(elevator);
    }

    // Called when someone presses an UP/DOWN button on a FLOOR (external request)
    void requestElevator(int floor, Direction direction) {
        Request request(floor, direction);
        Elevator* chosen = strategy->selectElevator(elevators, request);
        if (chosen) {
            chosen->addRequest(floor);
            std::cout << "Assigned floor " << floor << " request to an elevator.\n";
        }
    }

    // Called when someone presses a DESTINATION button INSIDE a specific elevator
    void selectFloor(Elevator* elevator, int floor) {
        elevator->addRequest(floor);
    }

    // Advances every elevator by one step (simulating time passing)
    void tick() {
        for (Elevator* elevator : elevators) {
            elevator->step();
        }
    }
};
```

- `ElevatorController` **composes** many `Elevator`s (they don't exist independently of the specific building's system) and **uses** a `SchedulingStrategy` (a separate, swappable dependency — recall Dependency Inversion from SOLID: the controller depends on the abstract `SchedulingStrategy` interface, not a specific concrete algorithm).

---

## 10. Step 9: The Full Class Diagram

```mermaid
classDiagram
    class ElevatorState {
        <<enumeration>>
        IDLE
        MOVING_UP
        MOVING_DOWN
        DOORS_OPEN
    }
    class Direction {
        <<enumeration>>
        UP
        DOWN
        NONE
    }
    class Request {
        +int floor
        +Direction direction
    }
    class Elevator {
        -int currentFloor
        -ElevatorState state
        -set~int~ upRequests
        -set~int~ downRequests
        +addRequest(floor) void
        +step() void
    }
    class SchedulingStrategy {
        <<interface>>
        +selectElevator(elevators, request) Elevator
    }
    class NearestElevatorStrategy {
        +selectElevator(elevators, request) Elevator
    }
    class ElevatorController {
        +requestElevator(floor, direction) void
        +tick() void
    }

    Elevator --> ElevatorState : has a
    Request --> Direction : has a
    SchedulingStrategy <|.. NearestElevatorStrategy
    ElevatorController "1" *-- "many" Elevator : manages
    ElevatorController --> SchedulingStrategy : uses
```

---

## 11. Step 10: Putting It All Together — a Working Example

```cpp
int main() {
    NearestElevatorStrategy strategy;
    ElevatorController controller(&strategy);

    Elevator elevator1(1);  // starts at floor 1
    Elevator elevator2(10); // starts at floor 10
    controller.addElevator(&elevator1);
    controller.addElevator(&elevator2);

    // Someone on floor 5 wants to go UP
    controller.requestElevator(5, Direction::UP);
    // The NearestElevatorStrategy will likely assign elevator1 (closer, at floor 1)

    // Simulate time passing, one "tick" at a time
    for (int i = 0; i < 10; i++) {
        controller.tick();
    }
}
```

---

## 12. Step 11: Extending the Design

```mermaid
flowchart TB
    A["Add a NEW scheduling algorithm<br/>(e.g. considering passenger load)?"] --> A1["✅ Just add a new SchedulingStrategy subclass —<br/>ZERO changes to ElevatorController"]
    B["Add elevator capacity limits<br/>(max weight/people)?"] --> B1["Extend Elevator with a currentLoad field,<br/>and have SchedulingStrategy factor it<br/>into the cost calculation — the STATE<br/>MACHINE and SCAN logic remain unchanged"]
    C["Support 'express' elevators<br/>that only serve certain floors?"] --> C1["Add a servedFloors set to Elevator,<br/>and filter candidate elevators in the<br/>SchedulingStrategy accordingly"]
    D["Multiple concurrent button presses<br/>from different floors/cars?"] --> D1["Same concurrency principle as the Parking<br/>Lot: addRequest() and the internal request<br/>sets need to be protected (e.g. a mutex per<br/>Elevator), since multiple threads could call<br/>it simultaneously"]
```

**Notice the recurring theme:** nearly every extension is absorbed by adding a new class or modifying one isolated piece (a new `SchedulingStrategy`, a new field on `Elevator`) — not by rewriting the core state machine or SCAN logic. This is the practical, demonstrated payoff of the Open/Closed Principle, exactly as it was for the Parking Lot's pricing extensibility.

---

## 13. How to Walk Through This in an Interview

A strong end-to-end summary sounds like this:

> "I'd model the elevator's behavior as an explicit state machine — Idle, Moving Up, Moving Down, Doors Open — rather than a handful of boolean flags, since that makes invalid combinations, like 'moving' and 'doors open' at once, structurally impossible to represent. There are two genuinely hard problems here: first, which elevator should respond to an external floor request, and second, in what order should one elevator serve its own pending stops. For the first, I'd use the Strategy pattern with a `SchedulingStrategy` interface, scoring each elevator based on distance and whether it's already moving toward the request in a compatible direction — this keeps the assignment algorithm swappable without touching the controller. For the second, I'd use the SCAN algorithm — the elevator keeps moving in its current direction, serving every pending stop it encounters along the way, and only reverses once there's nothing left in that direction — which avoids the wasted back-and-forth travel a naive first-come-first-served approach would cause. I'd store pending stops in a sorted set per direction, so the next stop is always readily available without needing to search. And since button presses can happen concurrently from different floors and cars, I'd protect each elevator's request-handling with a lock to avoid race conditions."

That answer shows: you modeled state explicitly and can justify why, you identified **two separate hard problems** rather than treating this as one simple task, you named and correctly applied a known algorithm (SCAN) rather than inventing an ad-hoc one, and you proactively addressed concurrency.

---

## 14. Quick Recall Cheat Sheet

```mermaid
mindmap
  root((Elevator System LLD))
    State Machine
      IDLE, MOVING_UP, MOVING_DOWN, DOORS_OPEN
      Explicit enum prevents invalid combinations
    Two Hard Problems
      Which elevator responds - dispatch
      What order it serves requests - scheduling
    Dispatch
      Strategy Pattern - SchedulingStrategy
      Score by distance + direction compatibility
      Swappable without touching controller
    Per-Elevator Scheduling
      SCAN algorithm - the "elevator algorithm"
      Finish current direction before reversing
      Sorted sets per direction for fast next-stop lookup
    Extending
      New strategy - just add a subclass
      Concurrency - lock per elevator, same as Parking Lot
```

| If you remember only 5 things |
|---|
| 1. Model the elevator's behavior as an explicit state machine (Idle, Moving Up, Moving Down, Doors Open) rather than boolean flags — this makes invalid states impossible to represent. |
| 2. There are two separate hard problems: which elevator responds to a request (dispatch), and in what order one elevator serves its own stops (scheduling) — solve them independently. |
| 3. Use the Strategy pattern for dispatch, scoring elevators by distance and direction compatibility, so the algorithm is swappable without touching the controller. |
| 4. Use the SCAN algorithm for per-elevator scheduling — keep moving in the current direction, serving every stop along the way, only reversing once nothing remains in that direction. |
| 5. Store pending requests in sorted sets, split by direction, so the next stop is always immediately available without needing to search or sort at request time. |

---

*This file is written in GitHub-flavored Markdown with Mermaid diagrams and C++ code — it will render natively on GitHub, GitLab, and most modern Markdown viewers.*
