# Patel-HW3-RTS26Summer-
HW 3 — Task design
[Homework PDF](https://github.com/parth1163/Patel-HW3-RTS26Summer-/blob/2effbb0aea4e6746d4cfd8b9f2c7c162f4bc28d7/HW%203%20%E2%80%94%20Task%20design.pdf)
---

## Part A — Task Table Analysis
The system utilization and scheduling details are analyzed below based on standard rate monotonic principles.

### CPU Utilization
* **Total Utilization:** $\frac{1}{10} + \frac{3}{20} + \frac{5}{50} + \frac{10}{100} = 10\% + 15\% + 10\% + 10\% = 45\%$
* **Idle Fraction Leftover:** $100\% - 45\% = 55\%$

### Priority Ranking
**(Highest)** $T1 \rightarrow T2 \rightarrow T3 \rightarrow T4$ **(Lowest)**

* **Justification:** Tasks with shorter operational periods receive higher execution priority than tasks with longer periods because they run at a higher frequency.

| Task | Priority Band | Justification |
| :--- | :--- | :--- |
| **T1** | 10-14 Sensor/IPC | It handles sensor data and has the shortest period, requiring immediate data processing. |
| **T2** | 15-18 Control | It manages core control algorithms and loop logic. While this band is numerically higher, T1 retains top execution priority due to Rate Monotonic Scheduling. |
| **T3** | 5-9 User | Handled as a default user task because its long period indicates it is for general communication transmission rather than raw, live telemetry data. |
| **T4** | 1-4 Housekeeping | Assigned to the background tier because its long execution cycle is ideal for miscellaneous system logging. |

### Yield Semantics
The task loop should yield using `vTaskDelayUntil` because it eliminates timing drift by utilizing a fixed tick count. This prevents processing lag and corrects any phase errors by ensuring tasks remain synchronized after being preempted.

### State-Machine Trace
![State-Machine Trace](https://raw.githubusercontent.com/parth1163/Patel-HW3-RTS26Summer-/5b08007a0f9b74a18b8d3f2a4505c4df04550274/StateMachineTrace.png)

---

## Part B — xTaskCreate Defended

    xTaskCreate(
        vControlLoop,       // Function pointer
        "ControlLoop",      // Task name string
        512,                // Stack size in WORDS (512 words = 2048 bytes)
        NULL,               // Pointer to Parameters
        16,                 // Priority number
        NULL                // Task Handler out-parameter
    );

### Parameter Defenses
* **`vControlLoop`:** The arbitrary name of the loop function defined in the system task table.
* **`"ControlLoop"`:** A human-readable text name assigned to the task for debugging purposes.
* **`512`:** I estimated the worst-case stack depth as 512. The slides state the maximum stack is 8KB; I quartered that value to allocate 2048 bytes (2KB stack), which equates to exactly 512 words on a 4-bytes-per-word architecture.
* **`NULL`:** Passed because there are no initialization pointers, arguments, or variables handed to this specific task at startup.
* **`16`:** The chosen numerical priority sitting inside the 15–18 "Control" band designated for time-critical control loop algorithms.
* **`NULL`:** Passed because the firmware design does not require an active task handle reference for runtime lifecycle modifications, suspension, or deletion.

---

## Part C — Theme Park Ride Control Design
**Theme:** Industrial / Theme-Park Automation

### System Task Table

| Name | Purpose | Period | WCET | Deadline | Priority | Justification |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **T1 - SeatbeltLock** | Detects if the rider is in the cart and that the harness or lap bar is locked before dispatch. | 20ms | 1ms | 20ms | **17** | Critical hard deadline requiring the highest priority because it directly governs initial rider constraint safety. |
| **T2 - EmergencyShutdown** | Monitors tracks and vehicle positions, tripping emergency brakes if a hazard is detected. | 20ms | 2ms | 20ms | **16** | Hard safety deadline set high to ensure active emergency interventions can immediately preempt non-safety systems. |
| **T3 - OperatorLogs** | Updates the operator station panel screen with live ride telemetry and system statuses. | 100ms | 10ms | 100ms | **9** | Lower priority soft/firm deadline; delay causes lag on the monitor but does not trigger a catastrophic critical failure. |

### Concurrency Diagram
![Theme Park System Concurrency Diagram](https://raw.githubusercontent.com/parth1163/Patel-HW3-RTS26Summer-/5b08007a0f9b74a18b8d3f2a4505c4df04550274/ConcurencyDiagram.png)

### Shared Resource and Guarding Mechanism
* **Shared Resource:** A global memory telemetry structure containing live sensor state readings from the seatbelt locks, shared across the SeatbeltLock, EmergencyShutdown, and `OperatorLogs tasks. This resource provides concurrent status updates across all three tasks.
* **Guarding:** **FreeRTOS Mutex** — Since multiple tasks can concurrently access this shared memory segment, a mutex semaphore is implemented to enforce mutual exclusion and guarantee that the telemetry data does not become corrupted during a preemption cycle.

---

## Part D — Industry Anchor: AUTOSAR Classic

AUTOSAR Classic uses a fixed-priority preemptive scheduling model to run its tasks. It is used in the **automotive industry** to run Electronic Control Units (ECUs), and it is specifically designed for safety systems with hard real-time requirements. It manages things like your engine, steering, and anti-lock brakes when you drive. We use this technology every day without even realizing it, which is why I chose to write about it.

A major difference from FreeRTOS is how memory and tasks are allocated. FreeRTOS allows you to dynamically create and destroy tasks and queues at runtime while the system is running. However, AUTOSAR Classic forbids runtime allocation for safety reasons, meaning every task, stack size, and memory boundary must be hardcoded and remain fixed at compile time before the car is ready to sell or drive, just as listed in the JPL Power of 10 rules.

### Sources Used
* [AUTOSAR Classic Platform Standard](https://www.autosar.org/standards/classic-platform)
* [UL Solutions — AUTOSAR Classic Platform Overview](https://www.ul.com/sis/blog/autosar-part-1-classic-platform)
* 
### AI Disclosure
Accessed June 7,2026:
I used Gemini to help me clean up the spelling, format the data tables, and convert my PDF into a ReadMe for GitHub. All the actual 
calculations, designs, and answers are my own work from the assignment.
