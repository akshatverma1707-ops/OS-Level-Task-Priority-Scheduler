# Architectural Decision Register (ADR): OS-Level Task Priority Scheduler

**Document ID:** `DOC-DEC-001`  
**Phase:** Phase 0 (Requirements Analysis and Implementation Planning)  
**Status:** READY FOR TEAM APPROVAL  
**Authors:** Minarv Tyagi, Akshat Verma, Keshav Kishor  
**Institution:** Jaypee Institute of Information Technology (JIIT)  
**Supervisors:** Dr. Varun Srivastava, Dr. Kirti Jain  

---

## Decision Register Overview

| ADR # | Decision Title | Proposed Recommendation | Approval Status |
|:---:|:---|:---|:---:|
| **ADR-01** | Frontend Architecture Selection | Mandatory C++ Terminal CLI Core; Optional React Web UI via HTTP (implemented ONLY if approved) | **READY FOR TEAM APPROVAL** |
| **ADR-02** | Scope of STL Restriction | Core Custom Data Structures Only (Permit standard I/O streams and strings) | **READY FOR TEAM APPROVAL** |
| **ADR-03** | Priority Numerical Convention | Custom Priority Convention: 0 = highest priority, 99 = lowest priority (smaller numerical value = higher priority) | **READY FOR TEAM APPROVAL** |
| **ADR-04** | Ready Queue: Binary Heap vs. Fibonacci Heap | Custom Dynamic Array Binary Min-Heap with PCB Index Map (`heap_index`) | **READY FOR TEAM APPROVAL** |
| **ADR-05** | Simulation Clock Progression Model | Hybrid Tick-Based Simulation ($\Delta t=1$) with Event Jump Optimization | **READY FOR TEAM APPROVAL** |
| **ADR-06** | K-ary Process Tree Node Representation | Left-Child Right-Sibling (LCRS) Binary Pointer Structure | **READY FOR TEAM APPROVAL** |
| **ADR-07** | History Hash Table Collision Resolution | Separate Chaining with Custom Singly-Linked Buckets and Prime Modulo | **READY FOR TEAM APPROVAL** |
| **ADR-08** | Deadlock Subsystem: Three Distinct Algorithms | Separate Interfaces: `SingleUnitRAGDetector`, `VectorReductionDetector`, `BankerAvoidance` | **READY FOR TEAM APPROVAL** |
| **ADR-09** | Simulated Memory Management Model | Total RAM Capacity with Strict FIFO Admission Queue & Documented HoL Blocking | **READY FOR TEAM APPROVAL** |
| **ADR-10** | RMS Schedulability, Horizon & Event Ordering | Liu-Layland Sufficiency, Configurable Horizon ($H_{sim}$), Boundary Completion First | **READY FOR TEAM APPROVAL** |
| **ADR-11** | Priority Inheritance Baseline Scope & Deferral | Multiple Mutexes, Multiple Waiters & Held Locks; Defer Nested/Transitive Chains | **READY FOR TEAM APPROVAL** |

---

## ADR-01: Frontend Architecture Selection

### Context
The original JIIT project synopsis states under Technology Used:
> *"Environment: Local Terminal / Console"*  
and lists a Graphical User Interface (GUI) under Section 9:
> *"Future Scope: Additional features might involve a graphical user interface (GUI)..."*

### Recommendation
**Adopt Dual-Mode Architecture (Mandatory C++ Terminal Baseline + Conditional React UI):**
1. Implement the **C++ Terminal Dashboard** as the mandatory core in Phase 9A.
2. The embedded HTTP server and React frontend are placed on an optional bonus track (Phase 9B, +30 hours).
3. **The embedded HTTP server will ONLY be implemented if the team explicitly approves the React frontend.** The core simulator will never have a runtime dependency on network sockets or browsers.

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-02: Scope of the Standard Template Library (STL) Restriction

### Recommendation
**Adopt Core Custom Data Structures Restriction Only:**  
- Strictly prohibit STL containers (`std::vector`, `std::map`, `std::queue`, `std::list`, `std::set`) in the implementation of AVL Tree, Binary Heap, Hash Table, K-ary Tree, FIFO Queues, and Graphs.
- Permit non-algorithmic standard utilities: `std::string` for parsing workload files, `std::ifstream` / `std::ostream` for file I/O and terminal formatting.

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-03: Priority Numbering Convention

### Context
Operating systems and simulation models use different numerical conventions for task priorities. The team must define an unambiguous project-wide convention.

### Recommendation
**Adopt Custom Priority Convention (0 = Highest Priority, 99 = Lowest Priority):**  
- `0` = Highest Scheduling Priority (Real-time / Critical).
- `99` = Lowest Scheduling Priority (Idle / Background).
- A smaller numerical value represents higher scheduling priority.
- Ready queue implemented as a Min-Heap.
- Tie-breaking: Identical priorities resolved deterministically by earlier arrival time, then lowest PID.

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-04: Ready Queue Selection (Binary Heap vs. Fibonacci Heap)

### Recommendation
**Adopt Custom Dynamic Array Binary Min-Heap with PCB Index Map:**  
- Storing `size_t heap_index` inside the PCB allows $O(1)$ node location and $O(\log n)$ priority updates (`siftUp` / `siftDown`) when effective priorities change during Priority Inheritance.
- Avoids the extreme pointer overhead and cache penalties of Fibonacci heaps.

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-05: Simulation Clock Progression Model

### Recommendation
**Adopt Hybrid Tick-Event Progression:**  
- Advances tick-by-tick ($\Delta t = 1$) while the CPU is executing tasks or undergoing context switches.
- Jumps forward to the next scheduled arrival event when the CPU is completely idle.

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-06: K-ary Process Tree Node Representation

### Recommendation
**Adopt Left-Child Right-Sibling (LCRS) Representation:**  
- Maps arbitrary $K$-ary process hierarchies onto binary node pointers (`first_child`, `next_sibling`, `parent`) with fixed memory ($24$ bytes per node).

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-07: History Hash Table Collision Resolution

### Recommendation
**Adopt Separate Chaining with Prime Modulo:**  
- Custom singly-linked bucket lists handle collisions cleanly without primary or secondary clustering.
- Uses Knuth's multiplicative hash and dynamic table doubling at load factor $\alpha > 0.75$.

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-08: Deadlock Subsystem: Three Distinct Algorithms

### Context
The team must model both single-instance and multiple-instance resources, as well as proactive avoidance and reactive detection.

### Recommendation
**Implement Three Independently Testable Algorithms with Dedicated Interfaces:**
1. **`IRAGCycleDetector` (`SingleUnitRAGDetector`):** Single-instance RAG cycle detection using 3-color DFS in $O(V+E)$. (Cycle is necessary and sufficient).
2. **`IMultiUnitDeadlockDetector` (`VectorReductionDetector`):** Multiple-instance Vector Reduction Algorithm (Shoshani-Coffman) in $O(m \times n^2)$. (Cycle is necessary, but not sufficient).
3. **`IBankerAvoidance` (`BankerAvoidance`):** Dijkstra's Banker's Algorithm for proactive deadlock avoidance and safe-sequence calculation in $O(m \times n^2)$.

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-09: Simulated Memory Management Model & Head-of-Line Blocking

### Context
The synopsis requires submitting tasks with memory footprints and scheduling based on memory.

### Recommendation
**Adopt Total RAM Capacity with Strict FIFO Admission Queue & Documented HoL Blocking:**
- Total System RAM is simulated (e.g., 1024 MB).
- Tasks are admitted to `READY` if memory fits, or placed in `WAITING_FOR_MEMORY` (a strict `CustomFIFOQueue`).
- When running tasks complete, memory is reclaimed, and waiting tasks are evaluated.
- **Architectural Trade-off:** Strict FIFO ordering causes **Head-of-Line (HoL) Blocking** when a large task at the queue head blocks smaller runnable tasks behind it. This limitation is retained to guarantee arrival-order fairness and prevent starvation of large processes.

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-10: Rate Monotonic Scheduling (RMS) Model Assumptions, Horizon & Boundary Events

### Context
RMS schedulability bounds require precise mathematical assumptions and simulation boundary handling.

### Recommendation
**Document Formal Model, Simulation Horizon, and Boundary Event Ordering:**
1. **Liu-Layland Model Assumptions:** Periodic tasks, $D_i = T_i$, independent tasks, constant WCET $C_i \le T_i$, zero overheads, preemptible, static rate-monotonic priorities ($T_i < T_j \implies \text{Priority}_i < \text{Priority}_j$, where smaller numerical value means higher scheduling priority under the project custom convention).
2. **Sufficiency vs. Feasibility:** $U \le n(2^{1/n}-1)$ is sufficient, not necessary (harmonic sets with $U > U_{bound}$ up to 1.0 are schedulable). $U \le 1.0$ is the necessary feasibility condition.
3. **Simulation Horizon ($H_{sim}$) & Overflow Safety:** Simulates over $\text{LCM}(T_1, \dots, T_n)$ capped at a configurable simulation horizon $H_{sim}$ to prevent 64-bit arithmetic overflow during $\text{LCM}$ calculation with coprime periods.
4. **Deterministic Boundary Ordering:** At tick $t = d$, task completion evaluation executes **before** deadline evaluation; finishing exactly at $t=d$ is recorded as **MET / SUCCESSFUL**, not a deadline miss.

### Status
**READY FOR TEAM APPROVAL**

---

## ADR-11: Priority Inheritance Protocol (PIP) Baseline Scope & Deferral

### Context
Priority inheritance can range from simple single-lock elevation to complex nested multi-lock transitive propagation with deadlock risks.

### Recommendation
**Support Multi-Lock Baseline; Explicitly Defer Nested/Transitive Chains:**
1. **Mandatory Baseline Scope:** Support multiple independent mutexes, multiple waiting tasks queued per mutex, multiple locks held simultaneously by a single process ($HeldLocks(P_i)$), dynamic elevation to highest waiting task, and dynamic effective-priority recalculation across remaining held locks upon release.
2. **Explicit Deferral:** Nested/transitive priority inheritance chains ($P_1 \to M_1 \to P_2 \to M_2 \to P_3$) are deferred to an optional extension track.

### Status
**READY FOR TEAM APPROVAL**
