# CDN Basics + Concurrency Control (Locking, Isolation Levels, MVCC)

*A zero-to-mastery guide for system design interviews and real-world architecture.*

---

## Table of Contents
**Part 1: CDN Basics**
1. [What Is a CDN?](#1-what-is-a-cdn)
2. [Why It's Needed](#2-why-its-needed)
3. [How a CDN Actually Works](#3-how-a-cdn-actually-works)
4. [Push vs Pull CDNs](#4-push-vs-pull-cdns)
5. [What CDNs Are (and Aren't) Good For](#5-what-cdns-are-and-arent-good-for)

**Part 2: Concurrency Control**
6. [What Problem Is Concurrency Control Solving?](#6-what-problem-is-concurrency-control-solving)
7. [The Classic Problem: Lost Updates](#7-the-classic-problem-lost-updates)
8. [Locking](#8-locking)
9. [Isolation Levels](#9-isolation-levels)
10. [MVCC (Multi-Version Concurrency Control)](#10-mvcc-multi-version-concurrency-control)
11. [Optimistic vs Pessimistic Concurrency Control](#11-optimistic-vs-pessimistic-concurrency-control)

**Wrap-up**
12. [How to Reason About This in an Interview](#12-how-to-reason-about-this-in-an-interview)
13. [Quick Recall Cheat Sheet](#13-quick-recall-cheat-sheet)

---

# Part 1: CDN Basics

## 1. What Is a CDN?

A **CDN (Content Delivery Network)** is a network of servers, physically spread out across many geographic locations, that store cached copies of your content — so users can fetch it from a server that's physically **close to them**, instead of always reaching back to your one, possibly-far-away origin server.

Think of it like a chain of local bakeries carrying the same signature cake, versus one single bakery in one city trying to ship that cake to customers all over the world — obviously the local branch gets it to the customer faster and fresher.

```mermaid
flowchart TB
    Origin[("Origin Server<br/>e.g. in Mumbai")]
    Origin --> Edge1["CDN Edge Server<br/>New York"]
    Origin --> Edge2["CDN Edge Server<br/>London"]
    Origin --> Edge3["CDN Edge Server<br/>Singapore"]
    UserNY[User in New York] --> Edge1
    UserLon[User in London] --> Edge2
    UserSG[User in Singapore] --> Edge3
```

---

## 2. Why It's Needed

### Without a CDN

```mermaid
sequenceDiagram
    participant User as User in Tokyo
    participant Origin as Origin Server in Mumbai

    User->>Origin: Request image.jpg (long physical distance)
    Note over User,Origin: High latency — data physically<br/>has to travel across continents
    Origin-->>User: Response (slow)
```

### With a CDN

```mermaid
sequenceDiagram
    participant User as User in Tokyo
    participant Edge as Nearby CDN Edge Server (Tokyo)
    participant Origin as Origin Server in Mumbai

    User->>Edge: Request image.jpg
    alt Edge already has it cached
        Edge-->>User: Response ⚡ (very fast — short distance)
    else Edge doesn't have it yet (cache miss)
        Edge->>Origin: Fetch it once
        Origin-->>Edge: image.jpg
        Edge-->>User: Response (slightly slower, but only the FIRST time)
        Note over Edge: Now cached at this edge for<br/>every future nearby user
    end
```

### The core reasons you need one
- **Lower latency** — physical distance is a real, hard cost (recall from the Performance Metrics topic — network travel time is a direct component of total latency), and a CDN minimizes it by serving from a nearby location.
- **Reduced origin server load** — every request served by an edge server never even reaches your origin server, freeing it up (this is the exact same load-reduction principle as caching, covered earlier, just applied geographically).
- **Better handling of traffic spikes** — a CDN's distributed capacity can absorb sudden bursts of traffic (e.g., a viral post) far better than a single origin server could alone.
- **Improved reliability** — if the origin server briefly struggles, cached content at the edges can often still be served to users.

---

## 3. How a CDN Actually Works

```mermaid
flowchart TB
    A["User requests a resource"] --> B{"Is it cached at the<br/>NEAREST edge server?"}
    B -->|"Yes — Cache HIT"| C["⚡ Served instantly from the edge"]
    B -->|"No — Cache MISS"| D["Edge server fetches from<br/>the origin server (or a nearby edge)"]
    D --> E["Edge server caches it<br/>for next time"]
    E --> F["Served to the user"]
```

### DNS-based routing
When a user requests your domain, the CDN's DNS system automatically directs them to the **nearest available edge server**, based on the user's geographic location — this routing decision happens transparently, before the actual content request is even made.

```mermaid
flowchart LR
    U1[User in Brazil] -->|DNS resolves to| E1[Nearest Edge:<br/>São Paulo]
    U2[User in Germany] -->|DNS resolves to| E2[Nearest Edge:<br/>Frankfurt]
```

---

## 4. Push vs Pull CDNs

### Pull CDN (the common default)
The CDN only fetches content from the origin **the first time** it's requested at a given edge location (a cache miss) — exactly as shown in Section 2's diagram. Content is "pulled" on demand.

```mermaid
flowchart LR
    A["First user requests file"] --> B["Cache miss — edge pulls from origin"]
    B --> C["Now cached — all future nearby users get it instantly"]
```
- **Good for:** most typical websites — simple to set up, and only caches what's actually being requested.

### Push CDN
Content is proactively **uploaded** to all CDN edge servers ahead of time, before any user ever requests it — the origin "pushes" the content out.

```mermaid
flowchart LR
    Origin["Origin Server"] -->|"Proactively uploads content<br/>to ALL edges upfront"| E1[Edge 1]
    Origin --> E2[Edge 2]
    Origin --> E3[Edge 3]
```
- **Good for:** sites with less frequently-changing content, or where you want to guarantee content is available at every edge immediately (e.g., a major software release going live at a precise moment everywhere at once).

---

## 5. What CDNs Are (and Aren't) Good For

```mermaid
flowchart TB
    Good["✅ CDNs excel at:<br/>Static content — images, videos, CSS, JS,<br/>downloadable files, static HTML pages"]
    Bad["⚠️ CDNs are less useful for:<br/>Highly dynamic, personalized, or<br/>rapidly-changing per-user data<br/>(though modern CDNs increasingly cache<br/>some dynamic content too, with short TTLs)"]
```

- CDNs are extremely effective for **static assets** that are the same for every user (an image, a JS bundle) — cache once, serve to millions.
- They're less naturally suited for **highly personalized, real-time data** (like a user's private, constantly-updating notification feed) — though modern CDNs do increasingly support caching some dynamic content with short expiration times.

---

# Part 2: Concurrency Control

## 6. What Problem Is Concurrency Control Solving?

**Concurrency control** is the set of techniques a database uses to make sure that when **multiple operations happen at the same time**, the data stays correct — instead of the operations stepping on each other and corrupting the result.

```mermaid
flowchart TB
    A["Two operations happen<br/>AT THE SAME TIME<br/>on the SAME data"] --> B{"Without concurrency control:<br/>results can be WRONG,<br/>depending on exact timing"}
    B --> C["Concurrency Control:<br/>rules that guarantee<br/>correct results regardless of timing"]
```

This becomes unavoidable the moment a system has more than one user, or more than one server (recall horizontal scaling from Day 1) — multiple requests genuinely can, and will, try to read and write the same data simultaneously.

---

## 7. The Classic Problem: Lost Updates

This single example makes the whole topic concrete.

**Scenario:** A bank account has ₹1000. Two withdrawal requests of ₹100 each arrive at nearly the same instant.

```mermaid
sequenceDiagram
    participant T1 as Transaction 1 (withdraw ₹100)
    participant DB as Database
    participant T2 as Transaction 2 (withdraw ₹100)

    T1->>DB: Read balance = ₹1000
    T2->>DB: Read balance = ₹1000
    Note over T1,T2: Both read the SAME starting value!
    T1->>DB: Write balance = ₹900 (1000 - 100)
    T2->>DB: Write balance = ₹900 (1000 - 100, using its OWN stale read)
    Note over DB: Final balance: ₹900<br/>❌ SHOULD have been ₹800 —<br/>one withdrawal was silently "lost"!
```

Both transactions read the same starting balance before either one had written its update, so the second write **overwrote** the first one's result, instead of building on top of it. This is called a **lost update**, and it's exactly the kind of bug concurrency control exists to prevent.

---

## 8. Locking

### The idea
A **lock** prevents other transactions from reading and/or modifying a piece of data while one transaction is currently working with it — forcing operations to happen one at a time on that specific piece of data, rather than simultaneously.

```mermaid
sequenceDiagram
    participant T1 as Transaction 1
    participant DB as Database
    participant T2 as Transaction 2

    T1->>DB: LOCK row + Read balance = ₹1000
    T2->>DB: Try to read/write same row
    Note over T2: BLOCKED — waits for T1's lock to release
    T1->>DB: Write balance = ₹900, UNLOCK
    Note over T2: Now proceeds
    T2->>DB: LOCK row + Read balance = ₹900 (correct, up-to-date value!)
    T2->>DB: Write balance = ₹800, UNLOCK
    Note over DB: Final balance: ₹800 ✅ Correct!
```

### Types of locks

```mermaid
flowchart TB
    Locks["Lock Types"] --> Shared["Shared Lock (Read Lock)<br/>Multiple transactions CAN read simultaneously,<br/>but none can write while any shared lock is held"]
    Locks --> Exclusive["Exclusive Lock (Write Lock)<br/>ONLY one transaction can hold this,<br/>blocking ALL other reads and writes"]
```

- **Shared lock:** many readers can proceed at once (reading doesn't conflict with other reading).
- **Exclusive lock:** a writer needs sole access — no one else can read or write until it's done.

### The downside of locking: deadlocks
If Transaction 1 locks Row A and waits for Row B, while Transaction 2 locks Row B and waits for Row A, neither can ever proceed — this is a **deadlock**.

```mermaid
flowchart LR
    T1["Transaction 1<br/>holds lock on Row A<br/>waiting for Row B"] -.waiting.-> T2
    T2["Transaction 2<br/>holds lock on Row B<br/>waiting for Row A"] -.waiting.-> T1
    Note1["Both stuck forever —<br/>database typically detects this<br/>and forcibly aborts one transaction"]
```

Databases typically detect deadlocks automatically and resolve them by forcibly rolling back one of the transactions, letting the other proceed.

---

## 9. Isolation Levels

**Isolation** is one of the four ACID properties (recall the SQL vs NoSQL topic) — it defines exactly how much concurrent transactions are allowed to "see" of each other's in-progress, uncommitted changes. Databases let you choose **how strict** this isolation should be, because stricter isolation is safer but slower.

```mermaid
flowchart LR
    A["Read Uncommitted<br/>(weakest, fastest)"] --> B["Read Committed"] --> C["Repeatable Read"] --> D["Serializable<br/>(strongest, slowest)"]
```

### The problems each level protects against

```mermaid
flowchart TB
    P1["Dirty Read:<br/>reading another transaction's<br/>UNCOMMITTED (possibly-to-be-rolled-back) changes"]
    P2["Non-Repeatable Read:<br/>re-reading the same row TWICE in one transaction<br/>gives DIFFERENT results, because another<br/>transaction committed a change in between"]
    P3["Phantom Read:<br/>re-running the same QUERY twice gives a<br/>DIFFERENT SET of rows, because another<br/>transaction inserted/deleted matching rows in between"]
```

### Level by level

**Read Uncommitted** — transactions can see other transactions' uncommitted changes.
```mermaid
sequenceDiagram
    participant T1
    participant T2
    T1->>T1: Update balance to ₹500 (not yet committed)
    T2->>T1: Read balance
    T1-->>T2: ₹500 (a "dirty read" — this might get rolled back!)
    T1->>T1: ROLLBACK
    Note over T2: T2 acted on data that never actually existed
```

**Read Committed** — transactions only ever see committed data (no dirty reads), but re-reading the same row twice within one transaction can still return different values if another transaction commits in between.

**Repeatable Read** — guarantees that if you read the same row twice within one transaction, you'll get the same value both times — but new rows matching a query's filter (inserted by someone else) can still appear on a re-run (a "phantom").

**Serializable** — the strictest level: transactions behave as if they ran one at a time, completely sequentially, with zero overlap — eliminates all three problems above, at the cost of the most reduced concurrency (and therefore the slowest performance under heavy simultaneous load).

### Comparison table

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|---|---|---|---|---|
| Read Uncommitted | Possible | Possible | Possible | Fastest |
| Read Committed | Prevented | Possible | Possible | Fast |
| Repeatable Read | Prevented | Prevented | Possible | Moderate |
| Serializable | Prevented | Prevented | Prevented | Slowest |

**Practical takeaway:** most applications default to **Read Committed** (PostgreSQL, Oracle) or **Repeatable Read** (MySQL/InnoDB) — Serializable is reserved for situations where correctness is absolutely critical and the performance cost is acceptable, like certain financial operations.

---

## 10. MVCC (Multi-Version Concurrency Control)

### The problem MVCC solves
Locking (Section 8) works, but it can be a performance bottleneck — readers and writers blocking each other means less concurrency overall. **MVCC** is a clever alternative approach used by most modern databases (PostgreSQL, MySQL/InnoDB) that avoids much of this blocking.

### The idea
Instead of locking a row so only one transaction can touch it, the database keeps **multiple versions** of each row. When a transaction starts, it gets a consistent "snapshot" view of the data as it existed at that moment — writers create a *new version* of a row rather than overwriting the old one in place, so readers never have to wait for writers, and writers don't have to wait for readers.

```mermaid
flowchart TB
    Row["Row: balance"] --> V1["Version 1: ₹1000<br/>(created at t=0, valid until t=5)"]
    Row --> V2["Version 2: ₹900<br/>(created at t=5, valid until t=9)"]
    Row --> V3["Version 3: ₹800<br/>(created at t=9, current)"]
```

```mermaid
sequenceDiagram
    participant Reader as Reader (started at t=6)
    participant DB as Database (MVCC)
    participant Writer as Writer (updates at t=9)

    Note over Reader: Reader's snapshot = data as of t=6
    Writer->>DB: Create NEW version (₹800) at t=9
    Reader->>DB: Read balance
    DB-->>Reader: ₹900 (the version valid at t=6,<br/>NOT blocked by the writer's newer version)
    Note over Reader,Writer: Reader and Writer never blocked each other!
```

### Why this is powerful
- **Readers never block writers, and writers never block readers** — a huge concurrency win compared to pure locking, since read and write operations can genuinely happen in parallel on the same row.
- **This is exactly how databases achieve isolation levels like Repeatable Read efficiently** — each transaction just keeps reading from its own consistent snapshot, rather than needing to hold locks the entire time.
- Old row versions are eventually cleaned up by a background process once no active transaction still needs them (in PostgreSQL, this process is called "vacuuming").

---

## 11. Optimistic vs Pessimistic Concurrency Control

This is a useful higher-level framing that ties locking and MVCC together.

```mermaid
flowchart TB
    A{"Concurrency Control Philosophy"}
    A --> Pess["Pessimistic:<br/>ASSUME conflicts will happen —<br/>lock data upfront to prevent them"]
    A --> Opt["Optimistic:<br/>ASSUME conflicts are rare —<br/>proceed without locking,<br/>check for conflicts only at the end"]
```

### Pessimistic (locking-based)
Lock the data *before* working with it, blocking anyone else from touching it in the meantime — as covered in Section 8. Safe by default, but reduces concurrency, since other transactions must wait even if a conflict would never have actually occurred.

### Optimistic (check-at-the-end)
Proceed without locking anything, but before committing the final write, check whether the data changed since you first read it (often via a version number or timestamp column). If it changed, reject the write and let the application retry.

```mermaid
sequenceDiagram
    participant T1 as Transaction
    participant DB as Database

    T1->>DB: Read balance = ₹1000, version = 5
    Note over T1: ...does some work, no lock held...
    T1->>DB: Update balance to ₹900, WHERE version = 5
    alt version is still 5 (no one else changed it)
        DB-->>T1: ✅ Success, version becomes 6
    else version changed (someone else already updated it)
        DB-->>T1: ❌ Rejected — retry with fresh data
    end
```

- **Good for:** systems with mostly non-conflicting operations (e.g., most users editing *different* rows) — avoids the overhead of locking when conflicts are genuinely rare.
- **Bad for:** high-contention scenarios (many transactions frequently fighting over the exact same row) — leads to a lot of wasted work and retries.

---

## 12. How to Reason About This in an Interview

If asked *"how would you handle two users updating the same record at the same time?"*, a strong answer sounds like this:

> "First I'd figure out how contested this data actually is — if conflicts on the same row are rare, like most users editing their own separate profile, I'd use optimistic concurrency control: read the data along with a version number, and only commit the write if the version hasn't changed, retrying otherwise. This avoids locking overhead for the common case where there's no real conflict. But if this is high-contention data — like a limited-inventory item where many users are genuinely racing for the same row — I'd use pessimistic locking instead, taking an exclusive lock before reading, so the second transaction simply waits and works from the up-to-date value rather than risking a lost update. For the isolation level, I'd default to Read Committed or Repeatable Read for general use, and only reach for Serializable if this specific operation is critical enough that even phantom reads could cause real problems, accepting the performance cost. I'd also mention that most modern databases use MVCC under the hood, which lets readers and writers avoid blocking each other in the common case, so a lot of this concurrency safety comes 'for free' without needing to hand-roll locking everywhere."

That answer shows: you can choose between optimistic and pessimistic based on the *actual contention pattern* rather than picking one by default, you know the *specific problems* each isolation level prevents (not just their names), and you understand *why* MVCC exists as the practical mechanism behind a lot of this.

---

## 13. Quick Recall Cheat Sheet

```mermaid
mindmap
  root((CDN + Concurrency Control))
    CDN
      Caches content at geographically close edge servers
      Reduces latency and origin load
      Pull CDN - cache on first request
      Push CDN - proactively uploaded
      Best for static content
    Concurrency Control
      Solves: multiple operations on same data at once
      Lost Update - the classic failure case
    Locking
      Shared read vs Exclusive write locks
      Risk: deadlocks
    Isolation Levels
      Read Uncommitted - dirty reads possible
      Read Committed - common default
      Repeatable Read - common default
      Serializable - strictest, slowest
    MVCC
      Multiple versions per row
      Readers don't block writers, vice versa
      Used by PostgreSQL, MySQL InnoDB
    Optimistic vs Pessimistic
      Optimistic - check version at commit time
      Pessimistic - lock upfront
      Choose based on contention level
```

| If you remember only 5 things |
|---|
| 1. A CDN caches content at edge servers close to users, cutting latency and reducing load on the origin server — best for static content. |
| 2. Concurrency control prevents bugs like "lost updates" when multiple operations touch the same data at the same time. |
| 3. Locking (shared for reads, exclusive for writes) prevents conflicts but can cause blocking and deadlocks. |
| 4. Isolation levels (Read Uncommitted → Read Committed → Repeatable Read → Serializable) trade correctness guarantees for performance — stricter is safer but slower. |
| 5. MVCC (used by PostgreSQL, MySQL) keeps multiple row versions so readers and writers don't block each other — it's how most modern databases achieve good concurrency without constant locking. |

---

*This file is written in GitHub-flavored Markdown with Mermaid diagrams — it will render natively on GitHub, GitLab, and most modern Markdown viewers.*
