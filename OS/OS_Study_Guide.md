# Operating Systems — Complete Study Guide for Placement Interviews

*This guide explains every concept in depth with reasoning, examples, and real-world context — designed so you truly understand the "why," not just memorize the "what." Read it once properly, and you won't need to re-read it before interviews.*

---

## 1. What is an Operating System? (Foundations)

An **Operating System (OS)** is the software layer that sits between raw hardware and the applications you run. Your CPU, RAM, disk, and peripherals don't understand "open Chrome" — they understand electrical signals and machine instructions. The OS's job is to translate high-level requests into hardware operations, while also making sure hundreds of programs can share the same limited hardware safely and fairly.

Think of the OS as a hotel manager: guests (processes) each think they have the whole hotel to themselves, but the manager is actually juggling room allocation (memory), scheduling housekeeping (CPU time), handling requests at the front desk (I/O), and preventing guests from walking into each other's rooms (protection/isolation).

**Why this matters for interviews:** A very common opening question is "What does an OS actually do?" The strong answer covers four core responsibilities:
1. **Process management** — deciding which program runs when.
2. **Memory management** — allocating and protecting RAM between programs.
3. **File/I/O management** — handling storage and device communication.
4. **Security/protection** — preventing programs from interfering with each other or the system.

### Program vs Process — get this distinction rock solid
A **program** is just static code sitting on disk — like a recipe written in a cookbook. It does nothing by itself.
A **process** is that recipe actually being *cooked* — it has ingredients in use (memory), a current step being executed (program counter), and its own kitchen space (address space). The same recipe (program) can be cooked (executed) multiple times simultaneously as multiple independent processes — e.g., opening two Chrome windows creates two separate processes from the same program.

### Kernel Mode vs User Mode
The CPU operates in two privilege levels for safety:
- **User Mode** — where regular applications run; they have restricted access and cannot directly touch hardware or other processes' memory.
- **Kernel Mode** — a privileged mode where the OS itself runs; it has full access to hardware and memory.

When a user program needs something only the kernel can do (like reading a file from disk), it makes a **system call** — a controlled, safe request that temporarily switches the CPU into kernel mode, performs the operation, and switches back. This barrier is exactly what prevents a buggy or malicious application from crashing your whole system or reading another program's private memory.

---

## 2. Processes — Life Inside the OS

### The Process Control Block (PCB)
Every process the OS manages is represented internally by a **Process Control Block** — essentially a data structure that acts as the process's "ID card" containing everything the OS needs to pause and resume it later:
- Process ID (PID)
- Current state (Ready, Running, Waiting, etc.)
- Program Counter (which instruction to execute next)
- CPU register values
- Memory management info (page tables, memory limits)
- List of open files and I/O status

Whenever the OS switches from running one process to another, it saves the outgoing process's current situation into its PCB and loads the incoming process's saved situation from its own PCB. This save-and-restore operation is what we call a **context switch**.

### The Process Lifecycle (States)
A process moves through a well-defined set of states during its life:

```
New → Ready → Running → (Waiting ⇄ Running) → Terminated
```

- **New** — the process is being created (OS is setting up its PCB, memory space, etc.).
- **Ready** — the process is fully set up and waiting in a queue for its turn on the CPU. It's not blocked on anything — it's purely waiting for the scheduler to pick it.
- **Running** — the process is actively executing on the CPU right now.
- **Waiting/Blocked** — the process cannot proceed because it's waiting for something external, like disk I/O to complete or user input.
- **Terminated** — the process has finished execution (or was killed), and the OS is cleaning up its resources.

**Why the Ready vs Waiting distinction matters:** A process in "Ready" state is fully capable of running right now — it's only the CPU that's busy elsewhere. A process in "Waiting" state literally cannot run even if given the CPU, because it's stuck waiting on something outside the CPU's control (like a disk read finishing). Interviewers often test whether you understand that a scheduler only picks from the *Ready* queue, never the Waiting queue.

### Context Switching — the hidden cost of multitasking
Every time the OS decides to stop running Process A and start running Process B, it must:
1. Save Process A's CPU register values, program counter, etc. into A's PCB.
2. Load Process B's previously saved state from B's PCB back into the CPU registers.

**Crucially, no actual "useful work" from either process happens during this switch** — it's pure overhead. This is exactly why excessive context switching (e.g., from too many processes competing for CPU, or a poorly chosen time quantum in Round Robin scheduling) can hurt overall system performance even though each individual process seems to get fair CPU time.

---

## 3. Threads — Lightweight Parallel Execution

A **thread** is the smallest unit of CPU execution — think of it as a single path of execution *within* a process. A process can have one thread (single-threaded) or many threads (multi-threaded), and all threads within the same process **share the same memory address space** (code, global data, heap), but each thread has its **own stack** and its own set of CPU registers/program counter.

**Why threads exist:** Creating a new process is expensive — the OS has to allocate a fresh memory space, copy resources, and set up a new PCB. Threads are much cheaper to create because they simply reuse the existing process's memory rather than getting their own. This makes threads ideal for tasks that need to run concurrently but also need to easily share data — e.g., a web server handling many client connections at once, where each connection is handled by a separate thread but they might share a common cache in memory.

### Process vs Thread — the interview-favorite comparison

| Basis | Process | Thread |
|---|---|---|
| Memory | Own separate address space | Shares address space with sibling threads |
| Creation cost | Heavy/slow (new memory setup) | Light/fast (reuses existing memory) |
| Communication | Needs IPC mechanisms (slower) | Direct shared memory access (faster) |
| Fault isolation | One process crashing doesn't affect others | One thread crashing can bring down the entire process |
| Context switch cost | Expensive (full memory context change) | Cheaper (only registers/stack change) |

**The tradeoff to always mention:** Threads are fast and efficient because they share memory, but that sharing is exactly what makes them dangerous — if two threads modify shared data without proper synchronization, you get bugs like race conditions. Processes are safer (isolated) but more expensive to create and communicate between. This exact tradeoff — speed & sharing vs. safety & isolation — is the recurring theme of concurrent programming.

### Multitasking vs Multiprocessing vs Multithreading — commonly confused trio
- **Multitasking** — the illusion of running multiple processes "at once" on a single CPU by rapidly switching between them (time-sharing). Only one process is truly executing at any given instant.
- **Multiprocessing** — using multiple physical CPUs/cores so that multiple processes genuinely execute *simultaneously*, not just switched rapidly.
- **Multithreading** — running multiple threads within a single process. Can be true parallelism (on a multi-core CPU) or just concurrency (time-sliced on a single core) — this is a subtlety worth mentioning explicitly if asked.

---

## 4. CPU Scheduling — Deciding Who Runs Next

The **scheduler** is the part of the OS that decides which process in the Ready queue gets the CPU next. This decision matters enormously — a bad scheduling policy can make short, quick jobs wait behind long ones, or let some processes starve entirely. Let's build up the important algorithms:

**First Come First Serve (FCFS)** — simplest possible policy: whichever process arrived first, runs first, non-preemptively (it runs to completion once started). The major problem is the **convoy effect** — if a long process happens to arrive first, every short process behind it must wait for the entire long process to finish, even though serving the short ones first would have kept everyone happier on average. This is exactly like being stuck behind someone doing a huge grocery shop at a single-checkout store.

**Shortest Job First (SJF)** — always picks whichever ready process has the smallest total burst (execution) time. This provably minimizes average waiting time among all non-preemptive algorithms — but it requires knowing burst times in advance (often estimated), and long jobs can suffer **starvation** if a constant stream of short jobs keeps arriving and jumping the queue ahead of them.

**Shortest Remaining Time First (SRTF)** — the preemptive version of SJF: if a new process arrives with a shorter remaining burst time than the currently running process, the CPU switches to the new process immediately. Same starvation risk as SJF, just more responsive.

**Priority Scheduling** — each process is assigned a priority number, and the highest-priority ready process runs next. Can be preemptive or non-preemptive. The classic problem: **starvation** of low-priority processes if high-priority ones keep arriving. The standard fix is **aging** — gradually increasing the priority of processes the longer they wait, guaranteeing they'll eventually get to the top.

**Round Robin (RR)** — each process gets a small fixed **time quantum** on the CPU, then is preempted and moved to the back of the Ready queue, giving the next process a turn. This is fair and responsive (good for interactive systems), but performance is extremely sensitive to the size of the quantum:
- Too large a quantum → behaves almost like FCFS (bad response time for later processes).
- Too small a quantum → excessive context-switching overhead eats into actual useful CPU time.

**Multilevel Queue Scheduling** — processes are split into separate queues based on type or priority (e.g., a "system processes" queue and an "interactive user processes" queue), each queue possibly using a different scheduling algorithm internally. This models real systems better (not all processes are equal) but adds complexity, and lower-priority queues can still starve.

### Key Scheduling Metrics — know these formulas cold
- **Turnaround Time** = Completion Time − Arrival Time (total time from arrival to finishing, including all waiting).
- **Waiting Time** = Turnaround Time − Burst Time (time spent waiting in the ready queue, excluding actual execution time).
- **Response Time** = Time of first CPU response − Arrival Time (important for interactive systems — how quickly does the user get *any* feedback, not necessarily completion).

📖 **Worked example:**
```
Process  Burst Time  Arrival
P1       6           0
P2       8           1
P3       7           2
P4       3           3
```
Under non-preemptive SJF, at time 0 only P1 has arrived, so it runs first (0–6). At time 6, P2, P3, and P4 have all arrived — among their burst times (8, 7, 3), P4 is shortest, so it runs next (6–9). Then P3 (9–16), then P2 (16–24). This demonstrates how SJF reorders execution purely by burst time among currently-arrived processes, not by arrival order.

---

## 5. Process Synchronization — Sharing Safely

When multiple threads or processes access shared data concurrently, unpredictable outcomes can occur if that access isn't carefully controlled. Understanding *why* this happens (not just the fix) is the foundation of this entire topic.

### Race Conditions — the root problem
A **race condition** occurs when the final outcome of a computation depends on the unpredictable timing/interleaving of multiple threads accessing shared data. 

📖 **Concrete example:** Two threads both execute `counter++`. This innocent-looking single line is actually three separate machine operations: (1) read `counter` into a register, (2) increment the register, (3) write it back to `counter`. If Thread A reads `counter = 5`, then before it writes back, Thread B also reads `counter = 5` and writes back `6`, then Thread A finishes and also writes back `6` — the counter should have become `7` (two increments), but it only became `6`. One increment was silently lost because the operations interleaved badly. This is a race condition, and it's why "just incrementing a variable" isn't safe across threads without protection.

### The Critical Section Problem
The section of code where a process accesses shared resources is called the **critical section**. Any valid solution to protecting it must satisfy three requirements:
1. **Mutual Exclusion** — only one process can be inside its critical section at any given time.
2. **Progress** — if no process is currently in its critical section, and some processes want to enter, the decision of who enters next cannot be postponed indefinitely — the system must make progress.
3. **Bounded Waiting** — there must be a limit on how many times *other* processes can enter their critical sections before a specific waiting process is guaranteed its turn (prevents starvation of a particular process even if not deadlock).

### Solutions — Mutex, Semaphore, Monitor
**Mutex (Mutual Exclusion Lock)** — the simplest tool: a lock that can be held by only one thread at a time. Whichever thread locks it is also responsible for unlocking it — this **ownership** concept is what distinguishes a mutex from a semaphore.

**Semaphore** — an integer variable manipulated only through two atomic operations:
- `wait()` (also called P, from Dutch "proberen" = to test) — decrements the semaphore; if it goes below 0, the calling process blocks.
- `signal()` (also called V, "verhogen" = to increment) — increments the semaphore, potentially waking a blocked process.

There are two flavors:
- **Binary Semaphore** — restricted to values 0 or 1, functionally similar to a mutex, but *without* ownership — any thread can call `signal()`, not just the one that called `wait()`. This is a subtle but real difference interviewers sometimes probe.
- **Counting Semaphore** — can hold any non-negative integer value, used to manage access to a *pool* of identical resources (e.g., a semaphore initialized to 5 lets up to 5 threads proceed simultaneously, perfect for something like a fixed-size connection pool).

**Monitor** — a higher-level synchronization construct, usually built into the programming language itself, that bundles shared data, the procedures that operate on it, and the necessary locking together — automatically ensuring mutual exclusion so the programmer doesn't have to manually call `wait()`/`signal()` everywhere. Easier to use correctly than raw semaphores because the compiler/runtime enforces the locking discipline.

📖 **Worked example — Producer-Consumer problem (a classic interview scenario):**
```
semaphore empty = N, full = 0, mutex = 1;

Producer:                       Consumer:
wait(empty); wait(mutex);       wait(full); wait(mutex);
  produce_item();                 consume_item();
signal(mutex); signal(full);    signal(mutex); signal(empty);
```
Here, `empty` tracks how many empty slots remain in a shared buffer of size N, and `full` tracks how many filled slots exist. A producer must wait for an empty slot before producing, and a consumer must wait for a full slot before consuming — this prevents the producer from overflowing a full buffer and the consumer from reading an empty one, while `mutex` ensures only one thread touches the buffer's internal pointers at a time.

---

## 6. Deadlock — When Everything Grinds to a Halt

A **deadlock** is a state where a group of processes are each waiting for a resource that another process in the same group is holding — forming a circular chain of waiting that never breaks on its own. No process can proceed, and without outside intervention, the system stays frozen forever.

### The Four Necessary Conditions (Coffman Conditions)
For a deadlock to occur, **all four** of these must hold simultaneously — understanding this is the key to understanding every deadlock-handling strategy, because each strategy works by breaking one of these four conditions:

1. **Mutual Exclusion** — at least one resource must be non-shareable (only one process can use it at a time).
2. **Hold and Wait** — a process is currently holding at least one resource while also waiting to acquire additional resources held by others.
3. **No Preemption** — resources cannot be forcibly taken away from a process; they can only be released voluntarily.
4. **Circular Wait** — there exists a cycle of processes, P1 waiting for a resource held by P2, P2 waiting for one held by P3, ..., and the last one waiting for a resource held by P1.

📖 **Concrete example:** Thread A locks Mutex 1, then tries to lock Mutex 2. Meanwhile, Thread B has already locked Mutex 2, and now tries to lock Mutex 1. Neither can proceed — A is waiting for Mutex 2 (held by B), and B is waiting for Mutex 1 (held by A). This is a textbook circular wait, and it's an extremely common real-world bug pattern in multithreaded code — the standard fix is to **always acquire locks in a consistent global order** (e.g., always lock Mutex 1 before Mutex 2, everywhere in your code), which directly breaks the circular wait condition.

### Handling Strategies

**Prevention** — design the system so at least one of the four conditions can *never* occur. For example, requiring processes to request *all* resources they'll ever need upfront (breaks Hold and Wait), or enforcing a strict global resource-acquisition order (breaks Circular Wait).

**Avoidance** — allow the four conditions to potentially exist, but carefully decide at runtime whether granting a particular resource request could lead to a deadlock, and refuse it if so. The classic algorithm here is the **Banker's Algorithm**.

**Detection and Recovery** — let deadlocks happen, but periodically check for them (using a **wait-for graph**, where a cycle indicates deadlock), and when detected, recover by forcibly terminating or rolling back one of the involved processes (the "victim") to break the cycle.

**Ignorance (the "Ostrich Algorithm")** — many general-purpose operating systems (like Linux and Windows) simply don't actively try to prevent, avoid, or detect deadlocks at the OS level, betting that they're rare enough in practice that the overhead of constant checking isn't worth it. This is actually the real-world default in most systems — a genuinely interesting fact to mention in interviews.

### The Banker's Algorithm — worked intuition
The Banker's Algorithm simulates: "if I grant this resource request, and every process eventually asks for its declared maximum need, is there *still* some order in which all processes can finish?" It works by checking if a **safe sequence** exists — an order of running processes to completion such that, at each step, the currently available resources are enough to satisfy that process's maximum remaining need. If such a sequence exists, the state is "safe," and the request can be granted; otherwise, the request is denied (or postponed) to avoid entering an unsafe state that could lead to deadlock. The name comes from the analogy of a banker who won't lend more money than they can guarantee to eventually recover from all customers.

---

## 7. Deadlock vs Starvation — Don't Confuse These

| Basis | Deadlock | Starvation |
|---|---|---|
| Definition | A group of processes are permanently stuck, each waiting circularly on another | A single process waits indefinitely because other processes keep "cutting in line" |
| Resolves itself? | Never — requires external intervention | Might eventually resolve, just very unfair |
| Root cause | All 4 Coffman conditions hold at once | Poor scheduling/priority policy consistently favoring others |
| Typical fix | Prevention, avoidance, or detection+recovery | **Aging** — gradually boost the waiting process's priority until it wins |

**The clean way to explain the difference in an interview:** In a deadlock, literally nobody in the stuck group can make any progress — it's a true standstill. In starvation, the *system as a whole* keeps making progress (other processes are running fine), it's just that one specific process is unlucky enough to keep losing out. This distinction is exactly why the fixes are different: deadlock needs structural changes to how resources are requested/granted, while starvation just needs a fairness mechanism like aging.

---

## 8. Memory Management — Fitting Programs into RAM

### Why memory management is hard
RAM is limited and must be shared safely across many processes, while each process needs to believe it has its own large, private, contiguous chunk of memory (even though physically it might be scattered, or even partially on disk). This illusion is what memory management techniques are built to provide.

### Paging
Physical memory is divided into fixed-size chunks called **frames**, and each process's logical memory is divided into equal-sized chunks called **pages**. The OS maintains a **page table** for every process, mapping each of that process's logical page numbers to the actual physical frame numbers where that data currently lives.

Because pages are fixed-size and a process's last page might not be fully used, paging causes **internal fragmentation** — wasted space *within* an allocated block. But because any free frame can be given to any process (no need for contiguous physical memory), paging completely eliminates **external fragmentation**.

### Segmentation
Memory is instead divided into variable-sized **segments** that correspond to logical divisions in a program — like a code segment, a data segment, a stack segment. This aligns nicely with how programmers actually think about their program's structure, but because segments have varying sizes, over time free memory gets broken into small, unusable gaps scattered around — this is **external fragmentation**.

### Paging vs Segmentation — side by side

| Basis | Paging | Segmentation |
|---|---|---|
| Block size | Fixed | Variable |
| Fragmentation type | Internal | External |
| Matches programmer's view? | No — purely a physical memory trick | Yes — reflects logical program structure |
| Address form | Single number (page number + offset) | Two numbers (segment number + offset) |

### Virtual Memory and Demand Paging
**Virtual memory** is the technique that gives every process the illusion of a large, private address space, even if physical RAM is much smaller — achieved by using disk storage (swap space) as an extension of RAM. 

**Demand paging** is the specific strategy of only loading a page into physical RAM *when it's actually needed* (i.e., when the process tries to access it and triggers a **page fault**), rather than loading the entire process into memory upfront. This is why you can run a program that's technically larger than your available RAM — not all of it needs to be resident in memory simultaneously.

### Page Replacement Algorithms — what happens when RAM is full
When a page fault occurs and there's no free frame available, the OS must choose an existing page to evict to make room. Different algorithms make this choice differently:

- **FIFO (First-In-First-Out)** — evicts whichever page has been in memory the longest, regardless of how often it's used. Simple but can perform poorly, and suffers from a strange phenomenon called **Belady's Anomaly**, where *increasing* the number of available frames can actually *increase* the number of page faults — counter-intuitive, but a real, well-documented behavior of FIFO specifically.
- **LRU (Least Recently Used)** — evicts the page that hasn't been accessed for the longest time, based on the reasonable assumption that pages used recently are likely to be used again soon (temporal locality). Generally performs well but requires extra bookkeeping (tracking access history) which adds overhead.
- **Optimal** — evicts whichever page will not be needed for the longest time *in the future*. This provably minimizes page faults, but it requires knowing the future access pattern in advance, which is impossible in a real running system — it exists purely as a theoretical benchmark to measure other algorithms against.
- **LFU (Least Frequently Used)** — evicts the page that has been accessed the fewest number of times. The weakness: a page that was just loaded and is genuinely about to be used a lot hasn't had a chance to build up a high access count yet, so LFU can wrongly evict useful new pages.

### Thrashing — when the system chokes on its own memory management
**Thrashing** occurs when a system is so overloaded with processes competing for limited RAM that it spends almost all of its time swapping pages in and out of disk, and almost no time actually executing useful instructions. CPU utilization paradoxically *drops* even as more processes are added, because the CPU keeps sitting idle waiting for slow disk swaps. The fix is to reduce the **degree of multiprogramming** (fewer processes competing for memory) or use working-set-based memory allocation, ensuring each process actually gets enough frames to hold its actively-used pages.

---

## 9. Inter-Process Communication (IPC)

Since processes have separate, isolated memory spaces by default, they need explicit mechanisms to communicate or share data:

- **Pipes** — a unidirectional communication channel typically used between a parent process and its child (related processes).
- **Named Pipes (FIFO)** — like a regular pipe, but has a name in the filesystem, so even unrelated processes can use it to communicate.
- **Message Queues** — the kernel maintains a queue of discrete messages; processes can send and receive from it without needing to be directly connected or running at the same time.
- **Shared Memory** — the fastest IPC mechanism, since it lets multiple processes directly read/write to a common region of memory — no copying through the kernel needed. The tradeoff is that the processes themselves must handle synchronization (via semaphores/mutexes) to avoid race conditions, since the OS doesn't automatically protect shared memory access.
- **Sockets** — a general communication endpoint that works both within the same machine and across a network, forming the basis of most client-server communication.
- **Signals** — a simple, lightweight way for the OS or another process to notify a process that a specific event happened (e.g., `SIGKILL` to forcibly terminate, `SIGTERM` to request graceful termination).

**How to decide which to mention in an interview:** If asked "how would two processes on the same machine share data fast," the strong answer is **shared memory** (fastest, but needs explicit synchronization). If asked about processes communicating reliably without worrying about timing, **message queues** are a solid answer. If asked about network communication, **sockets** is the expected answer.

---

## 10. Common Misconceptions to Correct Before Your Interview

- **"Multithreading always means true parallel execution."** Not necessarily — on a single-core CPU, multiple threads are still just time-sliced (concurrent, not parallel); true simultaneous execution needs multiple physical cores.
- **"Deadlock and starvation are basically the same problem."** They're not — deadlock is a permanent standstill for a whole group of processes; starvation is one process being unfairly delayed while the rest of the system keeps functioning normally.
- **"Paging has no fragmentation at all."** Paging eliminates *external* fragmentation, but still has *internal* fragmentation, since the last page allocated to a process is rarely used completely.
- **"Adding more RAM frames always reduces page faults."** Usually true, but FIFO specifically can violate this via Belady's Anomaly — a genuinely surprising fact worth knowing.
- **"A mutex and a binary semaphore are exactly the same thing."** They behave similarly but differ in ownership — only the thread that locked a mutex can unlock it, while any thread can call `signal()` on a binary semaphore regardless of who called `wait()`.
- **"System calls execute in user mode."** They don't — a system call is precisely the mechanism that switches the CPU into kernel mode temporarily so it can safely perform privileged operations, then switches back.
- **"Context switching does useful work for the processes involved."** It doesn't — it's pure bookkeeping overhead (saving/loading state); no actual process instructions execute during the switch itself.
- **"Round Robin is always fair and good."** Its performance is extremely sensitive to the time quantum size — too large behaves like FCFS, too small wastes time on constant context switching.

---

## 11. How to Talk About OS in an Interview (Strategy)

1. **Always tie abstract concepts to a concrete scenario.** Explaining deadlock through "two threads locking two mutexes in opposite order" is far more convincing than reciting the Coffman conditions from memory alone.
2. **Be ready to draw a Gantt chart** for any scheduling question and manually compute waiting time, turnaround time, and response time — this is one of the most commonly asked hands-on exercises.
3. **Nail the process vs thread distinction cold** — this is asked in nearly every SDE interview in some form, often as an opener.
4. **Practice explaining virtual memory and paging with a simple diagram** (logical pages mapping to physical frames via a page table) — visualizing it signals genuine understanding, not memorized definitions.
5. **When discussing synchronization, mention a specific real bug pattern** you understand — like the lost-update race condition on a shared counter, or the classic Producer-Consumer buffer problem.
6. **Connect answers to a real OS you've used**, typically Linux — mentioning `fork()`, `exec()`, process states visible via `ps`/`top`, or how Linux handles deadlocks (mostly via the Ostrich Algorithm) shows practical familiarity beyond textbook theory.
7. **Expect tradeoff-style follow-ups.** "Would you use Round Robin or Priority Scheduling here?" doesn't have one right answer — the strong response discusses the tradeoff (fairness/responsiveness vs. optimizing for specific important tasks) rather than picking one blindly.

---

## 12. Quick Reference Summary (for post-study revision)

```
┌────────────────────────────────────────────────────────────────┐
│  KEY TAKEAWAYS TO INTERNALIZE                                   │
│                                                                  │
│  • OS = manages CPU, memory, I/O, and protects processes         │
│  • Process = isolated execution unit; Thread = lightweight,     │
│    shares memory with sibling threads                           │
│  • States: New → Ready → Running → Waiting → Terminated         │
│  • Scheduling: FCFS(convoy effect), SJF/SRTF(starvation risk),  │
│    Priority(fix via aging), Round Robin(quantum-sensitive)      │
│  • Deadlock needs ALL 4: Mutual Excl, Hold&Wait,                │
│    No Preemption, Circular Wait                                  │
│  • Sync tools: Mutex(ownership), Semaphore(P/V, no ownership),  │
│    Monitor(language-level, easiest to use correctly)            │
│  • Paging = fixed blocks, internal fragmentation                │
│    Segmentation = variable blocks, external fragmentation       │
│  • Page replacement: FIFO(Belady's Anomaly), LRU, Optimal, LFU  │
│  • Thrashing = too much swapping, too little real execution     │
│  • Deadlock ≠ Starvation — starvation fixed via aging            │
│  • IPC: Pipes, Message Queues, Shared Memory(fastest), Sockets  │
└────────────────────────────────────────────────────────────────┘
```

---

*Once you've read through this and feel comfortable, revisit the quick-recall cheat sheet for a 10-minute brush-up right before your interview. Want a practice mock-interview Q&A round to test how well this has actually stuck?*
