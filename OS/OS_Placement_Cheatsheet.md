```
╔══════════════════════════════════════════════════════╗
║   OPERATING SYSTEMS — Placement Interview Cheat Sheet ║
║   Subject: OS  |  Level: Campus Placements/SDE        ║
╚══════════════════════════════════════════════════════╝
```

---

## ⚡ 1. CORE CONCEPTS (30-second recall)

- **Operating System** → software that manages hardware & acts as interface between user and computer.
- **Process** → a program in execution; has its own memory space (Code, Data, Stack, Heap).
- **Thread** → lightweight process; unit of CPU execution; shares memory with other threads of same process.
- **Program vs Process** → Program = passive code on disk; Process = active instance in execution.
- **Multitasking** → executing multiple processes seemingly at once (time-sharing on 1 CPU).
- **Multiprocessing** → using multiple CPUs to run processes truly in parallel.
- **Multithreading** → multiple threads within a single process, sharing memory.
- **Context Switching** → CPU saves state of one process/thread and loads another's state (overhead, no useful work done during switch).
- **Kernel** → core of OS; manages hardware resources directly; runs in privileged (kernel) mode.
- **System Call** → interface for user programs to request kernel services (e.g., `read()`, `fork()`).
- **Interrupt** → signal that halts CPU's current task to handle an urgent event (hardware/software).
- **Virtual Memory** → abstraction giving each process illusion of large, contiguous private memory using disk as extension of RAM.
- **Thrashing** → excessive paging/swapping causes CPU to spend more time swapping than executing (system near-freeze).
- **Spooling** → buffering data (e.g., print jobs) so a slow I/O device doesn't block the CPU.
- **Deadlock** → set of processes stuck forever, each waiting for a resource held by another in the set.
- **Starvation** → a process waits indefinitely because other higher-priority processes keep getting resources first.
- **Race Condition** → outcome depends on unpredictable timing/order of concurrent processes accessing shared data.
- **Critical Section** → code segment where shared resource is accessed; must be protected from concurrent access.

---

## 📐 2. PROCESS STATES & LIFECYCLE

```
New → Ready → Running → (Waiting ⇄ Running) → Terminated
              ↑    ↓
           (Ready ⇄ Running via scheduler)
```

| State | Meaning |
|---|---|
| New | Process being created |
| Ready | Waiting for CPU allocation (in ready queue) |
| Running | Currently executing on CPU |
| Waiting/Blocked | Waiting for I/O or event |
| Terminated | Finished execution |

**Process Control Block (PCB)** → data structure holding all info about a process: PID, state, program counter, CPU registers, memory limits, open files, scheduling info. Loaded/saved during every context switch.

---

## 🧩 3. PROCESS vs THREAD

| Basis | Process | Thread |
|---|---|---|
| Memory | Separate address space | Shares address space with sibling threads |
| Creation cost | Heavy (slow) | Light (fast) |
| Communication | IPC needed (slow) | Shared memory (fast) |
| Crash impact | One process crash doesn't affect others | One thread crash can bring down whole process |
| Context switch | Expensive | Cheaper |

💡 **Mnemonic** → "Threads are Twins sharing a house (memory); Processes are Strangers in separate houses."

---

## 🔄 4. CPU SCHEDULING ALGORITHMS

| Algorithm | Type | Description | Drawback |
|---|---|---|---|
| **FCFS** | Non-preemptive | First Come First Serve, run in arrival order | Convoy effect (short jobs wait behind long ones) |
| **SJF** | Non-preemptive | Shortest Job First — least burst time runs first | Starvation for long jobs |
| **SRTF** | Preemptive | Preemptive version of SJF | Starvation for long jobs |
| **Priority Scheduling** | Both | Highest priority runs first | Starvation for low priority (fix: aging) |
| **Round Robin** | Preemptive | Each process gets fixed time quantum in rotation | Performance depends heavily on quantum size |
| **Multilevel Queue** | Both | Multiple queues by process type/priority | Complex, can still starve lower queues |

**Key metrics:**
- **Waiting Time** = Turnaround Time − Burst Time
- **Turnaround Time** = Completion Time − Arrival Time
- **Response Time** = Time of first CPU response − Arrival Time

💡 **Mnemonic** → "**F**irst **C**ome **F**irst **S**erved = Queue at a shop; **S**hortest **J**ob **F**irst = Express checkout for few items"

---

## 🔐 5. PROCESS SYNCHRONIZATION

**Critical Section Problem** → ensure that when one process is executing in its critical section, no other process is allowed to execute in its own critical section (shared resource).

**Requirements for a solution:**
1. **Mutual Exclusion** → only one process in critical section at a time.
2. **Progress** → if no process is in CS, one of the waiting processes must be allowed in without indefinite delay.
3. **Bounded Waiting** → limit on how many times other processes enter CS before a waiting process gets its turn.

**Solutions:**
- **Locks/Mutex** → binary flag; only one process/thread can hold it at a time.
- **Semaphore** → integer variable, controlled via `wait()`(P) and `signal()`(V) operations.
  - **Binary Semaphore** → value 0 or 1, works like a mutex.
  - **Counting Semaphore** → value can range over a domain, controls access to a pool of resources.
- **Monitors** → high-level synchronization construct combining mutual exclusion + condition variables, managed by the language/compiler (easier than raw semaphores).

📝 Example: Semaphore-based solution for Producer-Consumer:
```
semaphore empty = N, full = 0, mutex = 1;

Producer:                       Consumer:
wait(empty); wait(mutex);       wait(full); wait(mutex);
  produce_item();                 consume_item();
signal(mutex); signal(full);    signal(mutex); signal(empty);
```

---

## ⚠️ 6. DEADLOCK — Conditions, Prevention, Avoidance

### 4 Necessary Conditions (Coffman Conditions) — ALL must hold for deadlock:
```
1. Mutual Exclusion   → resource can't be shared, only 1 process at a time
2. Hold and Wait      → process holds a resource while waiting for another
3. No Preemption      → resource can't be forcibly taken away
4. Circular Wait      → chain of processes each waiting on the next
```
💡 **Mnemonic** → "**M**ad **H**atters **N**ever **C**irculate" (Mutual Exclusion, Hold & Wait, No Preemption, Circular Wait)

### Handling strategies:
| Strategy | Approach |
|---|---|
| **Prevention** | Break at least one of the 4 conditions (e.g., request all resources at once) |
| **Avoidance** | Use algorithms like **Banker's Algorithm** to only grant requests that keep system in "safe state" |
| **Detection & Recovery** | Allow deadlock, detect via wait-for graph / resource allocation graph, then kill/rollback a process |
| **Ignorance (Ostrich Algorithm)** | Assume deadlock rarely happens (used by most general-purpose OS like Linux/Windows) |

**Banker's Algorithm** → simulates resource allocation in advance; grants a request only if the resulting state is still "safe" (i.e., some sequence exists where all processes can finish).

---

## 🗺️ 7. MEMORY MANAGEMENT

```
Contiguous Allocation → Paging → Segmentation → Virtual Memory (Demand Paging)
```

### Paging
- Physical memory divided into fixed-size **frames**; logical memory divided into same-size **pages**.
- Avoids external fragmentation (but can cause **internal fragmentation** — last page not fully used).
- **Page Table** maps logical page number → physical frame number, per process.

### Segmentation
- Divides memory into variable-sized **segments** based on logical units (code, stack, heap).
- Matches how programmers think, but causes **external fragmentation**.

### Virtual Memory & Demand Paging
- Only loads pages into RAM **when needed** (on a page fault), not the entire process upfront.
- Allows running processes larger than physical RAM using disk (swap space) as backing store.

### Page Replacement Algorithms (which page to evict when RAM is full)
| Algorithm | Rule | Note |
|---|---|---|
| **FIFO** | Evict oldest loaded page | Suffers from Belady's Anomaly |
| **LRU** | Evict Least Recently Used page | Good performance, higher overhead to track |
| **Optimal** | Evict page not needed for longest future time | Theoretical benchmark, not implementable in practice |
| **LFU** | Evict Least Frequently Used page | Can wrongly evict newly loaded but useful pages |

**Belady's Anomaly** → increasing number of frames can sometimes INCREASE page faults (happens with FIFO).

**Thrashing** → too many page faults, CPU spends more time swapping than executing → fix by reducing degree of multiprogramming or giving more frames (working set model).

---

## 📊 8. PAGING vs SEGMENTATION

| Basis | Paging | Segmentation |
|---|---|---|
| Division | Fixed-size blocks | Variable-size logical units |
| Fragmentation | Internal | External |
| Programmer visibility | Invisible to programmer | Reflects logical program structure |
| Address | Single number (page#, offset) | Two numbers (segment#, offset) |

---

## 📊 9. DEADLOCK vs STARVATION

| Basis | Deadlock | Starvation |
|---|---|---|
| Definition | Processes stuck forever, circular wait | Process waits indefinitely due to unfair scheduling |
| Resolvable? | Never resolves on its own | May eventually get resource |
| Cause | 4 Coffman conditions hold simultaneously | Poor priority/scheduling policy |
| Fix | Prevention/avoidance/detection | **Aging** → gradually increase priority of waiting process |

---

## 📝 10. QUICK EXAMPLES

📝 **Example (SJF Scheduling):**
```
Process  Burst Time  Arrival
P1       6           0
P2       8           1
P3       7           2
P4       3           3

Execution order (non-preemptive SJF): P1(0-6) → P4(6-9) → P3(9-16) → P2(16-24)
```

📝 **Example (Banker's Algorithm safe state check):**
Given Available, Max, and Allocation matrices, find if a sequence of processes exists such that each process's (Max − Allocation) ≤ Available at some point, updating Available after each process "finishes" and releases its resources.

📝 **Example (Race condition):**
Two threads increment a shared counter (`counter++`) without a lock — this isn't atomic (read → increment → write), so both threads can read the same old value, leading to a lost update.

---

## 🃏 11. MNEMONICS & MEMORY TRICKS

- 💡 **Coffman Conditions** → "Mad Hatters Never Circulate" (Mutual Exclusion, Hold & Wait, No Preemption, Circular Wait)
- 💡 **Process states** → "New Recruits Report Weekly, Then Retire" (New → Ready → Running → Waiting → Terminated)
- 💡 **Page Replacement** → "First In First Ousted" (FIFO), "Least Recently Used = Least Recently Loved" (LRU)
- 💡 **Semaphore ops** → P = Proceed with caution (wait/decrement), V = Vacate/release (signal/increment)
- 💡 **Scheduling metrics** → "Turnaround = Total time in system, Waiting = Turnaround minus actual work"

---

## ⚠️ 12. COMMON EXAM/INTERVIEW TRAPS

- ❌ Wrong: Thinking multithreading always means true parallelism → ✅ Right: On a single-core CPU, threads are time-sliced (concurrency), not truly parallel.
- ❌ Wrong: Believing deadlock and starvation are the same → ✅ Right: Deadlock = never resolves; Starvation = may resolve eventually, fixed via aging.
- ❌ Wrong: Paging has no fragmentation → ✅ Right: Paging avoids EXTERNAL fragmentation but still has INTERNAL fragmentation.
- ❌ Wrong: More RAM frames always reduce page faults → ✅ Right: FIFO can suffer Belady's Anomaly — more frames can increase faults.
- ❌ Wrong: Mutex and Binary Semaphore are identical → ✅ Right: Mutex is ownership-based (only the locking thread can unlock); binary semaphore has no ownership concept, any thread can signal it.
- ❌ Wrong: System calls run in user mode → ✅ Right: System calls switch CPU to kernel mode to safely access hardware/resources.
- ❌ Wrong: Context switching does useful computation → ✅ Right: It's pure overhead — no user process instructions execute during the switch itself.

---

## 📊 13. IPC (Inter-Process Communication) METHODS

| Method | Description |
|---|---|
| **Pipes** | Unidirectional communication between related processes |
| **Named Pipes (FIFO)** | Like pipes, but works between unrelated processes too |
| **Message Queues** | Processes exchange messages via a queue maintained by kernel |
| **Shared Memory** | Fastest — processes access common memory region directly (needs synchronization) |
| **Sockets** | Communication over network or same machine (client-server) |
| **Signals** | Simple notification mechanism for events (e.g., SIGKILL, SIGTERM) |

---

## 🎯 14. LAST-MINUTE INTERVIEW TIPS

1. Always connect **deadlock/synchronization answers to a real code example** (Producer-Consumer, Dining Philosophers) — interviewers love seeing applied understanding.
2. For scheduling questions, be ready to **draw a Gantt chart** and calculate waiting/turnaround time by hand.
3. Know the **difference between process and thread cold** — this is asked in almost every SDE interview.
4. Practice explaining **virtual memory and paging** with a simple diagram — visualization proves real understanding.
5. Be ready to explain a **real deadlock scenario** (e.g., two threads locking two mutexes in opposite order) and how to avoid it (consistent lock ordering).
6. Mention **real OS you've used/administered** (Linux) to sound practical — e.g., explain `fork()`, `exec()`, or process states via `ps`/`top`.
7. If asked "OS you're familiar with," be ready to discuss scheduling policy or memory management specifics of Linux at a high level.

---

## 🔑 15. ONE-GLANCE SUMMARY BOX

```
┌───────────────────────────────────────────────────────────┐
│  🔑 MUST-KNOW ESSENTIALS — OS PLACEMENT                    │
│  1. Process = program in execution; Thread = lightweight   │
│     unit sharing process memory                            │
│  2. States: New→Ready→Running→Waiting→Terminated           │
│  3. Scheduling: FCFS, SJF/SRTF, Priority, Round Robin       │
│  4. Deadlock needs ALL 4: Mutual Excl, Hold&Wait,           │
│     No Preemption, Circular Wait                            │
│  5. Sync tools: Mutex, Semaphore (P/V), Monitor             │
│  6. Paging = fixed blocks, internal fragmentation           │
│     Segmentation = variable blocks, external fragmentation │
│  7. Page replacement: FIFO, LRU, Optimal, LFU               │
│  8. Thrashing = too much swapping, too little real work    │
│  9. Deadlock ≠ Starvation (starvation fixed via aging)      │
│  10. IPC: Pipes, Message Queues, Shared Memory, Sockets     │
└───────────────────────────────────────────────────────────┘
```

---

*Want me to go deeper on any section (e.g., detailed Banker's Algorithm walkthrough, Dining Philosophers problem, or a mock OS interview Q&A round), or create a practice quiz to test yourself?*
