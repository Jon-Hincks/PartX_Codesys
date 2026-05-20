# HMI-Customizable Sequence Controller

This project is a flexible, table-driven process sequencer built in **CODESYS V3**. It allows an operator to build and execute a custom multi-step automation sequence directly from the HMI without needing to rewrite any PLC backend code.

---

## 💡 How It Works

Instead of hardcoding fixed steps (like a permanent Forward $\rightarrow$ Reverse loop), this system uses a **Finite State Machine (FSM)** paired with a structured data **Array**.

### 1. The HMI Data Table

The operator configures a sequence table on the screen. Each row represents a single step and contains:

* **ON/OFF Toggle:** Enables or completely bypasses that specific step.
* **Direction Dropdown:** Sets the action for that step (e.g., UP, DOWN, IDLE).
* **Duration Box:** Sets how long that step runs in milliseconds.

### 2. The Logic Engine

When the master schedule command is triggered from the HMI, the PLC uses an **Index Pointer** (a row counter) to step through the table:

1. It looks at the active row index.
2. If the row is disabled, it skips it instantly.
3. If the row is enabled, it reads the chosen direction, activates the corresponding physical outputs, and starts a timer scaled dynamically to the user's input time.
4. Once the timer finishes, it cleanly turns off the outputs, increments the row counter by 1, and repeats the process for the next row.
5. After checking the final row, it turns off the master command and safely resets back to the initial idle state.

---

## 🏗️ Program Structure

To ensure maximum performance and avoid common PLC programming pitfalls, the project is split cleanly across languages:

* **Parent Program (`PRG` - Ladder Logic):** Holds the persistent memory allocation. All internal timers, triggers, and index tracking variables live here so their data survives consistently across every millisecond PLC scan.
* **Child Sequence (`Action` - Structured Text):** Handles the mathematical data loop. It uses a `CASE` statement to process the active rows and evaluate the HMI variables dynamically.

---

## 📈 Key Advantages

* **Infinite Scalability:** You can expand the table from 4 rows to 8, 20, or more by simply updating the array boundaries in the data structure and the logic guard condition.
* **One-Shot Safety:** The master trigger uses a rising-edge trigger. Even if an operator leaves the "Run Sequence" button pressed indefinitely on the HMI, the sequence executes **exactly once** and waits for a fresh cycle.
* **Bypass Capability:** Unchecked steps are bypassed in less than a millisecond, preventing the machine from pausing or stuttering on unconfigured rows.
