# Team Workflow & Engineering Collaboration: OS-Level Task Priority Scheduler

**Document ID:** `DOC-TEAM-001`  
**Phase:** Phase 0 (Requirements Analysis and Implementation Planning)  
**Status:** READY FOR TEAM APPROVAL  
**Authors:** Minarv Tyagi, Akshat Verma, Keshav Kishor  
**Institution:** Jaypee Institute of Information Technology (JIIT)  
**Supervisors:** Dr. Varun Srivastava, Dr. Kirti Jain  
**Repository:** [OS-Level-Task-Priority-Scheduler](https://github.com/akshatverma1707-ops/OS-Level-Task-Priority-Scheduler)

---

## 1. Team Composition and Balanced Workload Distribution

To ensure equitable effort distribution across three undergraduate engineers while eliminating single-point-of-failure bottlenecks, the project responsibilities are divided across three primary engineering roles:

```
+-------------------------------------------------------------------------------------------------------------+
|                                           THREE-PERSON TEAM SPLIT                                           |
+------------------------------------+------------------------------------+-----------------------------------+
|      MEMBER 1: MINARV TYAGI        |      MEMBER 2: AKSHAT VERMA        |     MEMBER 3: KESHAV KISHOR       |
|    Enrollment: 2501200090          |    Enrollment: 2501200074          |    Enrollment: 2501200087         |
|  Data Structures & Core Engine     |  Algorithms & OS Architect         |  Resources, Analytics & Interface |
+------------------------------------+------------------------------------+-----------------------------------+
| - AVL Tree Process Registry        | - IScheduler Architecture          | - Hash Table Process History      |
| - Custom Binary Min-Heap & Index   | - Non-Preemptive Priority Sched    | - Resource & Mutex Manager        |
| - K-ary Process Tree (LCRS)        | - Preemptive Priority Sched        | - Single-Unit RAG DFS Cycle Det   |
| - Custom FIFO Queue                | - Multilevel Feedback Queue (MLFQ) | - Multi-Unit Vector Reduction Det |
| - Process Model (PCB) Lifecycle    | - Priority Inversion & Multi-PIP   | - Banker's Deadlock Avoidance     |
| - Simulation Clock & CPU Dispatcher| - Rate Monotonic Scheduling (RMS)  | - Analytics & Metrics Engine      |
| - Simulated Memory Manager & HoL   | - Workload Trace Verification      | - Mandatory Terminal Dashboard    |
| - Admission Wait-Queue Management  | - Liu-Layland & Boundary Analysis  | - Workload JSON/CSV Parser        |
+------------------------------------+------------------------------------+-----------------------------------+
```

---

## 2. Milestone 1.5: Shared Interfaces Formal Sign-Off Gate (MANDATORY)

To prevent integration breakage when members develop modules in parallel, the team establishes **Milestone 1.5**:

```
[Phase 1: Toolchain Setup]
            │
            ▼
[Milestone 1.5: Shared Interface Formal Sign-Off Gate]
  - backend/include/core/Types.hpp (Enums: ProcessState, priorities, error codes)
  - backend/include/core/Process.hpp (PCB fields, memory footprint, heap index)
  - backend/include/engine/IScheduler.hpp (Pure virtual scheduler contract)
  - backend/include/deadlock/ResourceTypes.hpp (Resource descriptors and matrices)
  - backend/include/engine/SimulationClock.hpp (Clock ticks & discrete events)
  - backend/include/engine/MemoryManager.hpp (Memory admission & capacity interface)
            │
            ▼ (Unanimous sign-off by Minarv, Akshat, and Keshav required)
[Phase 2: Custom Data Structures]
            │
            ▼
[Checkpoint: Progress & Burn Rate Review] (Assess actual hours vs 255h target)
```

### Protocol for Milestone 1.5:
1. Minarv, Akshat, and Keshav will draft and review the shared header contracts together in a dedicated branch (`feat/shared-interfaces`).
2. Every shared header must be approved by **all three members** before merging to `main`.
3. Once merged, these interfaces are **frozen**. No member may alter a method signature or struct field in these headers without opening an explicit micro-PR and obtaining unanimous approval from the other two members.

---

## 3. Checkpoint: First Implementation Milestone Progress & Burn Rate Review

Upon completion of Phase 2 (Custom Data Structures), the entire team will conduct a formal checkpoint review:
1. **Compare Actual vs. Estimated Hours:** Evaluate actual engineer-hours spent across Phase 1 and Phase 2 against preliminary targets.
2. **Assess Tooling & Sanitizer Velocity:** Verify whether AddressSanitizer and unit tests are catching defects early without slowing down development.
3. **Pacing Recalibration:** If burn rate exceeds estimates, the team will streamline subsequent phases or adjust task assignments before proceeding to the simulation engine.

---

## 4. Detailed Responsibility Matrices

### 4.1. Member 1: Minarv Tyagi (Data Structures & Engine Core)
- **Primary Ownership:**
  - `backend/include/datastructures/AVLTree.hpp` & `src/datastructures/AVLTree.cpp`
  - `backend/include/datastructures/BinaryHeap.hpp` & `src/datastructures/BinaryHeap.cpp` (with `heap_index` tracking)
  - `backend/include/datastructures/KaryTree.hpp` & `src/datastructures/KaryTree.cpp` (LCRS)
  - `backend/include/datastructures/CustomQueue.hpp` & `src/datastructures/CustomQueue.cpp`
  - `backend/include/core/Process.hpp` & `src/core/Process.cpp`
  - `backend/include/engine/SimulationClock.hpp` & `src/engine/SimulationClock.cpp`
  - `backend/include/engine/CPUDispatcher.hpp` & `src/engine/CPUDispatcher.cpp`
  - `backend/include/engine/MemoryManager.hpp` & `src/engine/MemoryManager.cpp` (with Head-of-Line blocking reporting)
- **Secondary / Review Responsibilities:**
  - Review all heap rebalancing and pointer synchronization calls in Schedulers.
  - Review memory allocation/deallocation to guarantee zero AddressSanitizer leaks.
  - Co-author Phase 1 (`CMakeLists.txt` and toolchain setup).

### 4.2. Member 2: Akshat Verma (Algorithms & OS Architect)
- **Primary Ownership:**
  - `backend/include/engine/IScheduler.hpp`
  - `backend/include/schedulers/NonPreemptivePriorityScheduler.hpp` & `.cpp`
  - `backend/include/schedulers/PreemptivePriorityScheduler.hpp` & `.cpp`
  - `backend/include/schedulers/MLFQScheduler.hpp` & `.cpp`
  - `backend/include/sync/PriorityInheritance.hpp` & `src/sync/PriorityInheritance.cpp` (multi-mutex holding & release)
  - `backend/include/schedulers/RMSScheduler.hpp` & `.cpp` (Liu-Layland bounds, horizon $H_{sim}$, boundary events)
- **Secondary / Review Responsibilities:**
  - Review engine dispatch hooks and context-switch latency calculations.
  - Review deadlock avoidance safe-sequence logic.
  - Maintain algorithm-level verification test suites in `backend/tests/`.

### 4.3. Member 3: Keshav Kishor (Resources, Analytics & Interface Lead)
- **Primary Ownership:**
  - `backend/include/datastructures/HashTable.hpp` & `src/datastructures/HashTable.cpp`
  - `backend/include/datastructures/CustomGraph.hpp` & `src/datastructures/CustomGraph.cpp`
  - `backend/include/deadlock/ResourceManager.hpp` & `src/deadlock/ResourceManager.cpp`
  - `backend/include/deadlock/SingleUnitRAGDetector.hpp` & `.cpp` (IRAGCycleDetector)
  - `backend/include/deadlock/VectorReductionDetector.hpp` & `.cpp` (IMultiUnitDeadlockDetector)
  - `backend/include/deadlock/BankerAvoidance.hpp` & `src/deadlock/BankerAvoidance.cpp` (IBankerAvoidance)
  - `backend/include/analytics/MetricsEngine.hpp` & `src/analytics/MetricsEngine.cpp`
  - `backend/include/interface/CLIDashboard.hpp` & `backend/src/interface/`
  - `backend/src/utils/WorkloadParser.cpp`
- **Secondary / Review Responsibilities:**
  - Review process termination history archival hooks.
  - Oversee workload test files (`examples/workloads/*.json`).
  - Coordinate end-to-end integration and defense presentation rehearsal.

---

## 5. Git Branching Model and Version Control Policy

The project uses a structured **Feature-Branch Workflow** anchored to GitHub:
- **Repository Remote:** `https://github.com/akshatverma1707-ops/OS-Level-Task-Priority-Scheduler`

### 5.1. Branch Classification
- `main`: Production-ready, stable codebase. Direct commits to `main` are strictly forbidden. All code enters `main` via merged Pull Requests.
- `feat/<module-name>`: Feature development branches.
- `fix/<bug-description>`: Bugfix branches for resolving failing tests or memory leaks.
- `test/<suite-name>`: Test suite expansion branches.

### 5.2. Pull Request (PR) Workflow and Gate Conditions
1. **Branch Creation:** The developer branches off the latest `main`.
2. **Local Verification:** Before opening a PR, the author must locally run:
   - Zero compilation warnings under `-Wall -Wextra -Wpedantic`.
   - All unit tests passing (`ctest`).
   - Clean AddressSanitizer run (`ASan`).
3. **PR Submission:** The author opens a PR using `.github/pull_request_template.md`.
4. **Mandatory Peer Review:** Every PR requires at least **one approving review** from a peer team member before merging.
5. **Merge Strategy:** Squash and merge (or rebase merge) to maintain a clean, linear Git history on `main`.

---

## 6. Workload Budget and Semester Pacing

To ensure the project remains completely achievable for undergraduate students balancing other coursework:
- **Preliminary Planning Target:** **255 engineer-hours** across 12 phases (**preliminary target, not a guaranteed commitment**).
- **Per-Student Allocation:** **85 hours per student** (~6 hours/week over 14 weeks).
- **Optional React Extension (Phase 9B):** Completely decoupled and capped at +30 hours. It will only be attempted if all core phases pass ahead of schedule.

---

## 7. Definition of Done (DoD)

A task or phase is considered complete ("Done") only when all following criteria are satisfied:
1. **Implementation:** Fully implemented in C++ conforming to modern C++ standards.
2. **No STL Violation:** Verified that core scheduling queues, registries, graphs, and trees use custom node structures.
3. **Unit Tests:** Dedicated unit tests implemented and passing in `backend/tests/`.
4. **Memory Safety:** Passes AddressSanitizer (`ASan`) with zero memory leaks and zero invalid reads/writes.
5. **Code Style & Comments:** Public functions, structs, and invariants documented with clear comments.
6. **Peer Review:** Code reviewed and approved by at least one other team member.
7. **Traceability:** Corresponds to an explicit requirement in `docs/PROJECT_PLAN.md`.
