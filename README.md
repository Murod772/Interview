## Refuel Event Detection 

### 1. Goal

Implement a Go function to detect “refuel” events in a sequence of fuel‐level readings, plus accompanying unit tests.

---

### 2. Input

* A slice of readings, each with:

  * `Timestamp` (`time.Time` or ISO-8601 string)
  * `Level` (`float64`)
* Readings may arrive out of order.

---

### 3. Processing

1. **Sort** readings by timestamp.
2. **Baseline** = first reading’s level.
3. **Traverse** in order:

   * If `current > baseline * 1.10`:

     * Record an event:

       * `When`: timestamp of this reading
       * `Before`: baseline
       * `After`: current
     * Set `baseline = current`
   * Otherwise, continue.

---

### 4. Output

Return a slice of events:

```go
type Event struct {
  When        time.Time
  BeforeLevel float64
  AfterLevel  float64
}
```

If no events detected, return an empty slice.

---

### 5. Edge Cases

* **Empty** or single‐reading input → no events.
* **Exact 110%** (`current == baseline*1.10`) → **no** event.
* **Incremental rises** count as one event the moment cumulative rise >10%.

---

### 6. What to Deliver

1. **Go code** implementing:

   * A function `DetectRefuels(readings []Reading) []Event`
2. **Unit tests** (using Go’s `testing` package) covering:

   * No events (empty, single)
   * Single event
   * Multiple events
   * Incremental scenario (e.g. 25 → 27 → 30 → 33)
   * Exact 110% boundary


### 6. Visiual Example

Fuel Level
   │
60 ┤
   │
50 ┤                        ● (50)
   │
40 ┤
   │
30 ┤
   │
22 ┼────────────── Threshold = 20 × 1.10 = 22.0
   │
20 ┼──● (20)─────────────────────────────────── Time ➔
     t₁   t₂   t₃       t₄

Readings:
- t₁ = 2025-02-03T18:00:00Z, level 20  
- t₂ = 2025-02-03T18:02:00Z, level 21  
- t₃ = 2025-02-03T18:04:00Z, level 22  
- t₄ = 2025-02-03T18:06:00Z, level 50  ← big jump  

Detected Refuel Event:

| Event Time               | Before Level | After Level |
|--------------------------|--------------|-------------|
| 2025-02-03T18:06:00 UTC  | 20           | 50          |
