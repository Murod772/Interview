# Refuel Event Detection — Coding Challenge

## Objective
Implement a Go program that detects “refuel” events from a sequence of fuel-level readings and returns a list of detected events. You are free to choose the detection approach; explain your assumptions and trade-offs.

---

## 1) Input

1. **List of Readings**  
   Each reading has:
   - `timestamp` (Go `time.Time` or ISO-8601 string)
   - `level` (float64, non-negative)

   **Example (JSON-like):**
   ```json
   { "time": "2025-02-03T18:02:31Z", "value": 25 }
   ```

2. **Ordering**  
   Readings may be out of order; sort by timestamp before processing.

3. **Data Validity**  
   Assume timestamps are valid; levels are non-negative.

---

## 2) Functional Requirements (You choose the algorithm)

1. **Refuel Event Definition (Threshold)**  
   A refuel event reflects a **material increase** in fuel level of **more than 10%** relative to a **reference level**.  
   - You must define and justify your **reference level** (e.g., last stable level, a smoothed/filtered value, moving median, etc.).
   - The comparison must use a strict inequality: increases of exactly 10.0% do **not** count.

2. **Event Grouping**  
   Increments that are part of the **same real-world refuel** should be treated as **one** event (e.g., 25 → 27 → 30 → 33 may be one event if that reflects a single refuel).  
   - Propose a reasonable rule for grouping (e.g., hysteresis, time windows, slope/derivative checks, or stability criteria) and document it.

3. **Multiple Events**  
   If subsequent data shows another >10% increase relative to your (possibly updated) reference, record another event. Clearly document when/how your reference updates.

4. **Noise Handling**  
   Minor fluctuations (sensor noise, slosh) below the threshold should **not** trigger events.  
   - You may use smoothing, debouncing, or other techniques; briefly justify your choice.

---

## 3) Output

Return a list (slice) of events with the following fields:

```go
type Event struct {
    When        time.Time // timestamp at which the increase is deemed to have crossed your 10% threshold
    BeforeLevel float64   // the reference/baseline level immediately before the event (per your definition)
    AfterLevel  float64   // the reading/value that triggered the event
}
```

If no events are detected, return an empty slice.

---

## 4) Edge Cases & Clarifications

1. **Exactly 10% vs. >10%**  
   Trigger only when `increase > 10%` by your definition; exactly 10% does not count.

2. **Short or Empty Data**  
   Empty list or a single reading → no events.

3. **Irregular Timestamps**  
   Large gaps don’t affect correctness by themselves; only the change pattern matters.

4. **Out-of-Order Input**  
   Must sort by timestamp before detection.

---

## 5) Example (for expectations only; not prescriptive of method)

**Data:**
```
(1) 2025-02-03T18:02:31Z → 25
(2) 2025-02-03T18:04:31Z → 27
(3) 2025-02-03T18:06:31Z → 30
(4) 2025-02-03T18:08:31Z → 33
```

A reasonable detector would produce **one** refuel event at `(3)` with:
- `BeforeLevel ≈ 25`
- `AfterLevel = 30`
- `When = 2025-02-03T18:06:31Z`  
…and **no** event at `(4)` since it’s exactly 10% relative to `30` (strictly “> 10%” required).  
Your approach may differ internally, but it should reach this outcome and be consistent with your documented rules.

---

## 6) Implementation Requirements

1. **Language**  
   Write all code in **Go**.

2. **Tests**  
   Provide Go unit tests that cover at least:
   - No refuel events  
   - Single refuel event  
   - Multiple refuel events  
   - Incremental increase case (e.g., 25 → 27 → 30 → 33 as one event)  
   - Edge cases (exactly 10%, empty input, out-of-order input)

3. **Documentation**  
   In `README.md` (or package doc), briefly describe:
   - Your definition of the **reference level**  
   - How/when you **update** it  
   - How you **group** increments into one event  
   - Any **noise-handling** or smoothing used

---

## 7) What We Evaluate

- **Correctness:** Outputs match expectations on provided/hidden tests  
- **Clarity:** Code readability and clear documentation of assumptions  
- **Robustness:** Handling of noise, ordering, and edge cases  
- **Testing Quality:** Coverage and meaningful assertions  
- **Design Justification:** Sound reasoning for your chosen approach (not the specific algorithm itself)

---

## 8) Optional Enhancements (not required)

- Configurable threshold (e.g., via flag/env)  
- CLI tool that reads JSON/CSV of readings and prints detected events  
- Microservice endpoint (`POST /detect`) that accepts readings and returns events (with OpenAPI spec)

---

**Deliverables:**  
- Go source code (detector + tests)  
- Short documentation of your approach and assumptions


![image](https://github.com/user-attachments/assets/7c2c380f-f8e5-41b1-8154-6f1814a26f78)

