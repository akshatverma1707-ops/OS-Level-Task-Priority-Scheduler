# Interface Specification & API Contract: OS-Level Task Priority Scheduler

**Document ID:** `DOC-API-001`  
**Phase:** Phase 0 (Requirements Analysis and Implementation Planning)  
**Status:** READY FOR TEAM APPROVAL  
**Authors:** Minarv Tyagi, Akshat Verma, Keshav Kishor  
**Institution:** Jaypee Institute of Information Technology (JIIT)  
**Supervisors:** Dr. Varun Srivastava, Dr. Kirti Jain  

---

## 1. Dual-Interface System Architecture

The simulator architecture defines two interface modes, strictly prioritizing the native terminal environment:

```
+---------------------------------------------------------------------------------------------------+
|                                 C++ SIMULATION ENGINE (BACKEND)                                   |
|   - 100% authoritative for all scheduling, time advancement, metrics, and OS algorithms          |
+---------------------------------------------------------------------------------------------------+
                  ▲                                                                 ▲
                  │                                                                 │
      (In-process API calls)                                            (HTTP / JSON over Loopback)
                  │                                                                 │
                  ▼                                                                 ▼
+-----------------------------------------+                     +-----------------------------------+
|     MODE A: C++ TERMINAL DASHBOARD      |                     |    MODE B: LOCAL REST HTTP API    |
|         [MANDATORY BASELINE]            |                     |       [OPTIONAL EXTENSION]        |
|   - Interactive CLI command prompt      |                     |   - Single-header cpp-httplib     |
|   - ASCII Gantt chart generation        |                     |   - Port: 8080 (localhost only)   |
|   - Formatted ANSI tabular outputs      |                     |   - Consumed by React / Browser   |
|   - Zero external dependencies          |                     |   - Implemented ONLY if approved  |
+-----------------------------------------+                     +-----------------------------------+
```

---

## 2. Mode A: Mandatory Terminal CLI & Text File Contract (Primary Interface)

The interactive ANSI Terminal CLI provides 100% of all required features without requiring network sockets, web servers, or external browser runtimes.

### 2.1. CLI Interactive Shell Commands
```text
SCHEDULER CLI COMMANDS:
  load <filepath>              Load workload configuration from JSON/CSV file
  policy <type>                Set policy (PRIO_NP, PRIO_P, MLFQ, RMS)
  step [n]                     Advance simulation by n ticks (default: 1)
  run                          Run simulation to completion (or up to simulation horizon)
  inspect registry             Dump active AVL Tree nodes (PID, Priority, Memory, State)
  inspect queue                Dump Heap / MLFQ ready queue contents (ordered by precedence)
  inspect memory               Display simulated total, used, free RAM & FIFO admission wait-queue
  inspect history              Dump terminated processes from Hash Table
  inspect tree                 Print ASCII representation of K-ary Process Tree
  inspect rag-single           Run single-unit 3-color DFS cycle detection (SingleUnitRAGDetector)
  inspect rag-multi            Run multi-unit Vector Reduction deadlock detection (VectorReductionDetector)
  banker check                 Run Banker's proactive safety check and print safe sequence (BankerAvoidance)
  metrics                      Display comprehensive scheduling metrics table
  gantt                        Render terminal ASCII execution Gantt chart
  reset                        Reset simulation state to tick 0
  exit                         Terminate application
```

### 2.2. Workload File Format (JSON / CSV)
The input workload allows tasks to be submitted with priority levels, burst profiles, and simulated memory requirements:

```json
{
  "system_config": {
    "total_memory_mb": 1024,
    "context_switch_penalty": 1,
    "starvation_threshold": 30,
    "simulation_horizon_ticks": 1000
  },
  "processes": [
    {
      "pid": 101,
      "parent_pid": 1,
      "arrival_time": 0,
      "burst_time": 8,
      "base_priority": 3,
      "memory_mb": 256
    },
    {
      "pid": 102,
      "parent_pid": 1,
      "arrival_time": 2,
      "burst_time": 4,
      "base_priority": 1,
      "memory_mb": 512
    },
    {
      "pid": 103,
      "parent_pid": 1,
      "arrival_time": 4,
      "burst_time": 2,
      "base_priority": 2,
      "memory_mb": 512
    }
  ]
}
```

### 2.3. ASCII Gantt Chart Terminal Output Format
```text
=== CPU EXECUTION TIMELINE (GANTT CHART) ===
Tick: 0    2 3       7 8   10            16
      [P101][C][ P102 ][C][P103][  P101   ]
Legend: [C] = Context Switch (1 tick penalty)

=== SIMULATED MEMORY STATUS ===
Total RAM: 1024 MB | Used RAM: 768 MB | Free RAM: 256 MB
Admission Queue: [PID 104 (512 MB)] <-- Head-of-Line Blocking active (Free 256 MB unutilized)
                 [PID 105 (64 MB)]  <-- Waiting behind Head-of-Line

=== SCHEDULING METRICS SUMMARY ===
+-----+---------+-------+-------+------+------+------+---------+--------+
| PID | Arrival | Burst | Start | Comp | Wait | TAT  | Memory  | Starve |
+-----+---------+-------+-------+------+------+------+---------+--------+
| 101 | 0       | 8     | 0     | 16   | 8    | 16   | 256 MB  | NO     |
| 102 | 2       | 4     | 3     | 7    | 1    | 5    | 512 MB  | NO     |
| 103 | 4       | 2     | 8     | 10   | 2    | 6    | 512 MB  | NO     |
+-----+---------+-------+-------+------+------+------+---------+--------+
Average Waiting Time:    3.67 ticks
Average Turnaround Time: 8.33 ticks
CPU Utilization:         87.50%
Total Context Switches:  2
```

---

## 3. Mode B: Local REST HTTP API Specification (Optional Extension)

> [!NOTE]
> **Conditional Implementation:**  
> This HTTP API and the embedded `cpp-httplib` server will **ONLY** be implemented if the team explicitly approves the React + TypeScript frontend in ADR-01. It is completely decoupled from the core C++ simulation engine.

### 3.1. Server Characteristics
- **Transport:** HTTP/1.1 over TCP loopback (`http://127.0.0.1:8080`).
- **Data Interchange:** Standard UTF-8 JSON.
- **Embedded Implementation:** Single-header C++ library (`cpp-httplib`).

### 3.2. Error Model (RFC 7807 Problem Details)
```json
{
  "error": {
    "code": "INSUFFICIENT_SYSTEM_MEMORY",
    "message": "Process memory requirement exceeds total system physical capacity.",
    "field": "processes[0].memory_mb",
    "invalid_value": 2048,
    "system_capacity_mb": 1024
  }
}
```

### 3.3. Endpoints Overview

#### `POST /api/v1/simulation/init`
Initializes a new simulation session with a selected scheduling policy, system memory capacity, simulation horizon, and parameters.

#### `POST /api/v1/simulation/workload`
Submits a batch workload of processes including memory footprint attributes.

#### `POST /api/v1/simulation/step`
Steps simulation forward by $N$ ticks, returning current active PID, remaining burst, memory utilization, and events.

#### `GET /api/v1/simulation/timeline`
Retrieves the complete execution Gantt timeline, context switches, and process state histories.

#### `GET /api/v1/simulation/metrics`
Retrieves aggregated metrics (waiting, turnaround, response times, throughput, CPU utilization, and memory usage).

#### `POST /api/v1/deadlock/single-instance`
Executes single-unit 3-color DFS cycle detection via `SingleUnitRAGDetector`.

#### `POST /api/v1/deadlock/multi-instance`
Executes multiple-instance Vector Reduction deadlock detection via `VectorReductionDetector`.

#### `POST /api/v1/deadlock/banker/check`
Evaluates a resource request or tests state safety via `BankerAvoidance`, returning the safe sequence.

#### `GET /api/v1/rms/analysis`
Reports Rate Monotonic schedulability analysis: computes the Liu-Layland sufficiency bound, compares actual utilization, handles hyperperiod simulation with overflow safety, and reports simulated deadline misses.
