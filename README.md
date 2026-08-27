# carp-coordinator

A modular, high-performance Event System for the [Carp language](https://github.com/carp-lang/Carp).

Refactored into a decoupled architecture, this module provides specialized tools for event routing, time management, and loop orchestration.

## The Conductor Architecture

This library is built on the **Conductor** mental model, separating the system into three independent layers:

1.  **Dispatcher (Routing)**: A standalone switchboard for synchronous event broadcasting and subscription management.
2.  **Scheduler (Clock)**: A high-precision timer heap that tracks *when* events should fire and supports **Task Cancellation**.
3.  **Coordinator (Executive)**: The glue that orchestrates the Scheduler, a Deque-based task queue, and the Dispatcher into a cohesive, throttled run loop with **Urgent Dispatch** support.

## Features

- **Decoupled Modules**: Use only what you need (e.g., just the Dispatcher for synchronous GUI events).
- **Strictly Typed**: Generic over user-defined event types (usually sumtypes).
- **Task Cancellation**: Cancel scheduled or periodic tasks using unique TaskIDs.
- **Result-Aware Handlers**: Handlers return `Result` types, allowing the Coordinator to track and log failures.
- **Urgent Dispatch**: Skip the queue using `dispatch-urgent!` for high-priority system events.
- **Deterministic Iteration**: The Dispatcher snapshots handlers before broadcasting, ensuring safety even if handlers modify subscriptions mid-dispatch.
- **Exhaustive Ticking**: The Coordinator's `tick!` recursively drains the queue until empty, following a microtask-style execution model.
- **High Performance**: Amortized $O(1)$ routing via `Deque` and $O(\log n)$ scheduling via `PriorityQueue`.
- **Intelligent Throttling**: The Coordinator loop automatically sleeps when idle to preserve CPU cycles.

## Installation

Add this to your project by loading `dispatch.carp`.

```clojure
(load "path/to/carp-deque/deque.carp")
(load "path/to/carp-priority-queue/priority_queue.carp")
(load "path/to/carp-coordinator/dispatch.carp")

(use Dispatcher)
(use Scheduler)
(use Coordinator)
```

## Quick Start

### The Full Loop (Coordinator)

```clojure
(use Coordinator)
(use Result)

(deftype AppEvent (Msg [String]) (Stop []))

(let [c (Coordinator.new)]
  (do
    (Coordinator.subscribe! &c &(fn [ev] 
      (match ev 
        (AppEvent.Msg s) (do (IO.println s) (Result.Success ()))
        (AppEvent.Stop) (do (Coordinator.stop! &c) (Result.Success ())))))
    
    (Coordinator.dispatch! &c (AppEvent.Msg @"Hello Coordinator!"))
    (Coordinator.schedule! &c (AppEvent.Stop) 1000l) ; Stop in 1s
    
    (Coordinator.run! &c)))
```

## Running Tests

```bash
carp -x test/engine_test.carp
```

## Examples

See [examples.md](examples.md) for usage examples.

## License

MIT
