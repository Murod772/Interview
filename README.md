# Refuel Event Detection — Requirements



## 1. Input





1. **List of Readings**  


   - Each reading is composed of:  


     - A timestamp (either `time.Time` or a string in ISO-8601 format)  


     - A numeric fuel level (int or float)  


   - **Example (JSON-like)**:


     ```json


     {


       "time": "2025-02-03T18:02:31Z",


       "value": 25


     }


     ```





2. **Ordering**  


   - The readings may not be in chronological order.  


   - The function must sort them by timestamp before processing.





3. **Data Validity**  


   - Assume timestamps are valid and fuel-level values are non-negative.





---





## 2. Core Logic





1. **Refuel Event Definition**  


   - A refuel event occurs when the fuel level exceeds **110%** of the last stable baseline.  


   - Mathematically:


     \[


     \text{current} > \text{baseline} \times 1.10


     \]





2. **Stable Baseline**  


   - Initially set the baseline to the **first** reading’s fuel level.  


   - Process readings in chronological order:  


     - If the new reading **does not** exceed `baseline * 1.10`, keep going.  


     - Once it **does** exceed `baseline * 1.10`, record a refuel event and update the baseline to this new reading’s fuel level.





3. **Incremental Increases as One Event**  


   - Incremental increases (e.g., `25 → 27 → 30 → 33`) should be considered **one** refuel event when the cumulative increase first crosses the 10% threshold from the **initial** baseline (e.g., 25).  


   - After triggering the event, the new reading becomes the new baseline.





4. **Multiple Refuel Events**  


   - If a subsequent reading exceeds **110%** of the updated baseline, record another refuel event.  


   - Continue until all readings are processed.





---





## 3. Output





1. **Refuel Event Details**  


   - The function should return a list of refuel events, each containing:  


     - **Event Timestamp**: when the threshold was **first** exceeded  


     - **Before (Baseline) Fuel Level**: the stable fuel level before the event  


     - **After Fuel Level**: the reading that triggered the event





2. **No Events Case**  


   - If no refuel event is detected, return an empty list (or a designated response).





---





## 4. Edge Cases & Clarifications





1. **Exact 110% vs. More Than 110%**  


   - Refuel event triggers only if `current > baseline * 1.10`.  


   - If `current == baseline * 1.10`, it **does not** count as a refuel.





2. **Short or Empty Data**  


   - If the list is empty or has only one reading, no refuel event can be detected.  


   - Return an empty list or a clear indicator.





3. **Timestamp Gaps**  


   - Large or irregular time gaps between readings do not affect the core logic — only the relative fuel-level change matters.





4. **Sensor Noise**  


   - Minor fluctuations below 10% do not count until they collectively exceed the 10% threshold.



---





## 5. Example Scenario





### Data


```


(1) time=2025-02-03T18:02:31Z, value=25 


(2) time=2025-02-03T18:04:31Z, value=27 


(3) time=2025-02-03T18:06:31Z, value=30 


(4) time=2025-02-03T18:08:31Z, value=33


```





1. **Initial Baseline** = `25`  


2. **Check 27** against `25 * 1.10 = 27.5`  


   - `27 <= 27.5`, no refuel event  


3. **Check 30** against `25 * 1.10 = 27.5`  


   - `30 > 27.5`, **refuel event** triggered  


   - **Event Timestamp** = `2025-02-03T18:06:31Z`  


   - **Before** = `25`, **After** = `30`  


   - **New Baseline** = `30`  


4. **Check 33** against `30 * 1.10 = 33.0`  


   - `33 == 33.0`, not more than 10%, no new event  





**Result**: One refuel event.


![image](https://github.com/user-attachments/assets/7c2c380f-f8e5-41b1-8154-6f1814a26f78)



---





## 6. Additional Implementation Requirements





1. **Write All Code in Go**  


   - Implement the logic and data processing using Go.



2. **Unit Tests**  


   - Write unit tests in Go that cover:


     - No refuel events  


     - Single refuel event  


     - Multiple refuel events  


     - Incremental increase scenario (e.g., 25 → 27 → 30 → 33)  


     - Edge cases (e.g., `current == baseline * 1.10`, empty data)

