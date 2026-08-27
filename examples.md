# Coordinator System Examples

## Standalone Dispatcher

Perfect for simple message passing or GUI event routing where you don't need a loop or timers.

```clojure
(use Dispatcher)
(use Result)

(deftype UIEvent (Click [Int Int]) (Hover [String]))

(defn main []
  (let [d (Dispatcher.new)
        id (Dispatcher.subscribe! &d &(fn [ev] 
             (match ev
               (UIEvent.Click x y) (do (IO.println "Clicked!") (Result.Success ()))
               _ (Result.Success ()) )))]
    (do
      (ignore (Dispatcher.broadcast! &d &(UIEvent.Click 10 20)))
      (Dispatcher.unsubscribe! &d id))))
```

## Standalone Scheduler with Cancellation

Use this to track time-based tasks and drain them into your own custom processing logic.

```clojure
(use Scheduler)

(defn main []
  (let [s (Scheduler.new)]
    (do
      (let [id (Scheduler.schedule! &s @"One-shot" 100l)]
        (ignore (Scheduler.cancel! &s id)))
        
      (Scheduler.schedule-periodic! &s @"Heartbeat" 500l)
      
      (while true
        (let [due (Scheduler.drain-due! &s (System.nanotime))]
          (doall &(fn [ev] (IO.println ev)) &due))))))
```

## Full Loop with Urgent Dispatch

```clojure
(use Coordinator)
(use Result)

(deftype AppEvent (Critical [String]) (Normal [String]) (Stop []))

(defn main []
  (let [c (Coordinator.new)]
    (do
      (Coordinator.subscribe! &c &(fn [ev]
        (match ev
          (AppEvent.Critical s) (do (IO.println &(str* "URGENT: " s)) (Result.Success ()))
          (AppEvent.Normal s) (do (IO.println s) (Result.Success ()))
          (AppEvent.Stop) (do (Coordinator.stop! &c) (Result.Success ())))))
      
      (Coordinator.dispatch! &c (AppEvent.Normal @"I was here first"))
      (Coordinator.dispatch-urgent! &c (AppEvent.Critical @"I jumped the line"))
      (Coordinator.schedule! &c (AppEvent.Stop) 100l)
      
      (Coordinator.run! &c))))
```
