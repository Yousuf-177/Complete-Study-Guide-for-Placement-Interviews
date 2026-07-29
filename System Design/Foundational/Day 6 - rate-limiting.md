# Rate Limiting

*A zero-to-mastery guide for system design interviews and real-world architecture.*

---

## Table of Contents
1. [What Is Rate Limiting?](#1-what-is-rate-limiting)
2. [Why It's Needed](#2-why-its-needed)
3. [Where Rate Limiting Sits in a System](#3-where-rate-limiting-sits-in-a-system)
4. [Rate Limiting Algorithms](#4-rate-limiting-algorithms)
5. [How the Client Finds Out It's Been Limited](#5-how-the-client-finds-out-its-been-limited)
6. [Rate Limiting in Distributed Systems](#6-rate-limiting-in-distributed-systems)
7. [What to Limit By](#7-what-to-limit-by)
8. [How to Reason About This in an Interview](#8-how-to-reason-about-this-in-an-interview)
9. [Quick Recall Cheat Sheet](#9-quick-recall-cheat-sheet)

---

## 1. What Is Rate Limiting?

**Rate limiting** is a technique for controlling *how many requests* a client (a user, an IP address, an app) is allowed to make to a system within a given time window — and rejecting the rest.

Think of it like a nightclub bouncer with a strict capacity rule: "only 100 people allowed in per hour." It doesn't matter how many people show up — once the count hits 100, the bouncer turns people away until the next hour's quota opens up.

```mermaid
flowchart LR
    U1[Request 1] --> RL{Rate Limiter}
    U2[Request 2] --> RL
    U3[Request 3 - over limit] --> RL
    RL -->|Allowed| Server[Server]
    RL -->|"❌ Rejected<br/>429 Too Many Requests"| Reject[Blocked]
```

---

## 2. Why It's Needed

Without limits, a system has no defense against a client (malicious or just buggy) sending an overwhelming number of requests.

```mermaid
flowchart TB
    A[No Rate Limiting] --> B[One client can flood the server<br/>with unlimited requests]
    B --> C[Server resources exhausted]
    C --> D[Legitimate users can't get through]
```

### The core reasons you need it
- **Protect against abuse** — a malicious actor could hammer your API to brute-force passwords, scrape all your data, or intentionally overload your servers (a Denial of Service attack).
- **Protect against bugs** — a client-side bug (e.g., a broken retry loop) can accidentally send thousands of requests per second with zero malicious intent.
- **Fair usage** — prevents one heavy user from consuming all the system's capacity and starving everyone else.
- **Cost control** — every request usually costs something (compute, third-party API calls, bandwidth) — limiting requests limits your bill.
- **Protecting downstream systems** — if your API calls a database or a third-party service that has its own limits, your rate limiter protects those systems too.

```mermaid
flowchart LR
    subgraph Without["Without Rate Limiting"]
        direction TB
        W1[Attacker sends 100,000 req/sec] --> W2[🔥 Server crashes]
        W2 --> W3[Real users get errors]
    end
    subgraph With["With Rate Limiting"]
        direction TB
        R1[Attacker sends 100,000 req/sec] --> R2[Rate Limiter blocks excess]
        R2 --> R3[✅ Server stays healthy]
        R3 --> R4[Real users unaffected]
    end
```

---

## 3. Where Rate Limiting Sits in a System

Rate limiting can be enforced at different points, and in real systems it's often applied at more than one layer.

```mermaid
flowchart TB
    Client[Client] --> CDN["1. CDN / Edge<br/>(blocks obvious abuse early, cheaply)"]
    CDN --> Gateway["2. API Gateway<br/>(most common place — before requests even reach app servers)"]
    Gateway --> App["3. Application Code<br/>(fine-grained, per-user/per-feature limits)"]
    App --> DB[(Database)]
```

- **Applying it as early as possible** (e.g., at the edge/gateway) is generally preferred — it's cheaper to reject a request before it consumes app server or database resources.
- **Applying it in application code** allows for more nuanced rules (e.g., "free-tier users get 100 requests/day, paid users get 10,000").

---

## 4. Rate Limiting Algorithms

This is the core engineering decision: *how* do you actually track and enforce the limit? There are several standard algorithms, each with different tradeoffs.

### 4.1 Fixed Window Counter
Divide time into fixed windows (e.g., every 1-minute block), and count requests within each window. Once the count hits the limit, reject further requests until the next window starts.

```mermaid
flowchart LR
    subgraph W1["Window: 12:00:00 - 12:00:59"]
        direction TB
        A1["Requests: 1,2,3...100 ✅"]
        A2["Request 101 ❌ blocked"]
    end
    subgraph W2["Window: 12:01:00 - 12:01:59"]
        direction TB
        B1["Counter resets to 0<br/>Requests allowed again"]
    end
    W1 --> W2
```

- **Simple to implement**, but has a flaw: a burst of traffic right at the boundary between two windows can let through *almost double* the intended limit.

```mermaid
sequenceDiagram
    participant Client
    participant Limiter as Fixed Window Limiter (limit: 100/min)

    Note over Client,Limiter: 12:00:59 — 100 requests sent (fills window 1)
    Note over Client,Limiter: 12:01:00 — window resets
    Note over Client,Limiter: 12:01:01 — another 100 requests sent (fills window 2)
    Note over Limiter: 200 requests went through in ~2 seconds!<br/>Even though the limit was "100 per minute"
```

### 4.2 Sliding Window Log
Instead of fixed blocks, keep a timestamped log of every request. To check if a new request is allowed, count how many requests happened in the *last* 60 seconds, sliding continuously — not tied to a fixed clock boundary.

```mermaid
flowchart LR
    Now["Current time: 12:01:30"] --> Window["Look back exactly 60 seconds:<br/>count requests between 12:00:30 - 12:01:30"]
    Window --> Decision{"Count < limit?"}
    Decision -->|Yes| Allow[✅ Allow]
    Decision -->|No| Block[❌ Block]
```

- **Very accurate** — completely solves the boundary-burst problem from Fixed Window.
- **Downside:** needs to store a timestamp for every single request, which can get memory-intensive at high volume.

### 4.3 Sliding Window Counter
A practical middle ground: combines the simplicity of Fixed Window with better accuracy, by taking a **weighted average** of the current and previous window's counts.

```mermaid
flowchart TB
    A["Previous window: 80 requests"]
    B["Current window so far: 40 requests<br/>(30% of the way through this window)"]
    C["Estimated count = 40 + (80 × 70%) = 96"]
    A --> C
    B --> C
```

- **Good balance** of accuracy and memory efficiency — this is what many real-world systems (like Cloudflare) actually use.

### 4.4 Token Bucket
Imagine a bucket that holds tokens, refilled at a steady rate (e.g., 10 tokens/second) up to some maximum capacity. Every request consumes one token. If the bucket is empty, the request is rejected.

```mermaid
flowchart TB
    Bucket["🪣 Bucket (capacity: 10 tokens)<br/>Refills at 2 tokens/sec"]
    Req1["Request arrives"] --> Check{"Token available?"}
    Check -->|Yes, take 1 token| Allow["✅ Allow"]
    Check -->|"No (bucket empty)"| Block["❌ Block"]
    Bucket -.refill over time.-> Check
```

- **Key advantage:** allows short **bursts** of traffic (as long as tokens have accumulated), while still enforcing a steady average rate over time — this matches real traffic patterns much better than a hard per-second cap.
- **Very widely used** in practice (e.g., AWS API Gateway uses this model).

### 4.5 Leaky Bucket
Similar setup, but instead of tokens being consumed, incoming requests are added to a queue (the "bucket") and processed ("leaked out") at a fixed, constant rate — regardless of how bursty the incoming traffic is.

```mermaid
flowchart TB
    In["Incoming requests<br/>(bursty — arrive unevenly)"] --> Bucket["🪣 Queue"]
    Bucket -->|"Processed at a FIXED steady rate"| Out["Requests sent to server<br/>(smooth, constant rate)"]
    Bucket -->|"If queue overflows"| Drop["❌ Excess requests dropped"]
```

- **Key advantage:** smooths out bursts into a steady, predictable outflow — great when the downstream system (e.g., a fragile third-party API) needs a constant, non-spiky load.
- **Difference from Token Bucket:** Token Bucket allows bursts to pass through immediately if tokens are available; Leaky Bucket always outputs at a fixed rate, queuing anything extra.

### Algorithm comparison

| Algorithm | Handles Bursts? | Accuracy | Memory Cost | Common Use |
|---|---|---|---|---|
| Fixed Window | Poorly (boundary issue) | Low | Very low | Simple, non-critical limits |
| Sliding Window Log | N/A (very precise) | Very high | High | When precision really matters |
| Sliding Window Counter | Good | High | Low-medium | Common production default |
| Token Bucket | Yes, by design | High | Low | APIs expecting bursty traffic |
| Leaky Bucket | No — smooths everything out | High | Low | Protecting fragile downstream systems |

---

## 5. How the Client Finds Out It's Been Limited

When a request is rejected, the server should respond with a clear status code and (ideally) tell the client when it can try again.

```mermaid
sequenceDiagram
    participant Client
    participant RL as Rate Limiter

    Client->>RL: Request #101 (over the limit)
    RL-->>Client: 429 Too Many Requests<br/>Retry-After: 30<br/>X-RateLimit-Remaining: 0
    Note over Client: Client knows to wait<br/>30 seconds before retrying
```

- **429 Too Many Requests** — the standard HTTP status code for this.
- **Helpful response headers:**
  - `Retry-After` — how many seconds to wait before trying again.
  - `X-RateLimit-Limit` — the total allowed requests in the current window.
  - `X-RateLimit-Remaining` — how many requests are left.
  - `X-RateLimit-Reset` — when the limit resets.

This lets well-behaved clients back off gracefully instead of hammering the API blindly.

---

## 6. Rate Limiting in Distributed Systems

Here's a subtlety that trips people up: if your app runs on **multiple servers** behind a load balancer (as covered when discussing horizontal scaling), where does the request *count* actually get stored?

### The naive (broken) approach: count locally on each server

```mermaid
flowchart TB
    U[Client sends 300 requests] --> LB{Load Balancer}
    LB --> S1["Server 1<br/>counts 100 requests locally<br/>thinks: within limit ✅"]
    LB --> S2["Server 2<br/>counts 100 requests locally<br/>thinks: within limit ✅"]
    LB --> S3["Server 3<br/>counts 100 requests locally<br/>thinks: within limit ✅"]
    Note1["Each server only sees its own 100 —<br/>but the client actually got 300 through!<br/>The 'global' limit of 100 was never enforced."]
```

### The fix: a shared, centralized counter

```mermaid
flowchart TB
    U[Client sends requests] --> LB{Load Balancer}
    LB --> S1[Server 1]
    LB --> S2[Server 2]
    LB --> S3[Server 3]
    S1 --> Redis[("Shared Counter<br/>e.g. Redis<br/>tracks the TRUE global count")]
    S2 --> Redis
    S3 --> Redis
    Redis --> Decision{"Global count over limit?"}
    Decision -->|Yes| Block[❌ Block, regardless of which server handled it]
    Decision -->|No| Allow[✅ Allow]
```

- A fast, shared, in-memory store (commonly **Redis**, using atomic increment operations) is used so every app server checks and updates the *same* count — this is the exact same "shared state" pattern used to fix the session problem discussed for horizontal scaling.

---

## 7. What to Limit By

Rate limits can be applied based on different keys, depending on what you're trying to protect against.

```mermaid
flowchart TB
    Key{What to key the limit on?}
    Key --> IP["By IP address<br/>— blocks anonymous abuse,<br/>but breaks down behind shared IPs (e.g. offices)"]
    Key --> User["By User/API Key<br/>— fairer, ties limit to an authenticated identity"]
    Key --> Endpoint["By Endpoint<br/>— e.g. login endpoint gets a stricter limit<br/>than a read-only endpoint, to prevent brute-forcing"]
    Key --> Global["Globally<br/>— an overall system-wide cap,<br/>as a last-resort safety net"]
```

Often, systems combine several of these — e.g., a strict per-IP limit on the login endpoint specifically (to prevent password brute-forcing), plus a broader per-user limit across the whole API.

---

## 8. How to Reason About This in an Interview

If asked *"how would you protect this API from abuse?"*, a strong answer sounds like this:

> "I'd add rate limiting, ideally enforced as early as possible — at the API gateway, before requests even reach the app servers, so we're not wasting compute on requests we're going to reject anyway. For the algorithm, I'd lean toward Token Bucket since real traffic is bursty and it allows short spikes while still enforcing a steady average rate — Fixed Window is simpler but has a boundary problem where nearly double the limit can slip through right at the edge of two windows. Since the app runs on multiple servers behind a load balancer, I can't track the count locally on each server — I'd use a shared store like Redis with atomic increments so the count is consistent globally, no matter which server handles a given request. I'd key the limit by authenticated user for general fairness, but apply a stricter, separate limit by IP specifically on sensitive endpoints like login, to prevent brute-force attempts. And I'd return 429 with a Retry-After header so well-behaved clients know exactly when to try again."

That answer shows: you know rate limiting is a real *system-wide* concern (not just a single line of code), you can justify an *algorithm choice* based on traffic patterns, you catch the *distributed counting problem* (a very common follow-up question), and you think about *what to key limits on* rather than treating it as one-size-fits-all.

---

## 9. Quick Recall Cheat Sheet

```mermaid
mindmap
  root((Rate Limiting))
    Why needed
      Prevent abuse / DoS
      Protect against buggy clients
      Fair usage across users
      Cost control
      Protect downstream systems
    Where enforced
      CDN / Edge
      API Gateway most common
      Application code
    Algorithms
      Fixed Window simple, boundary flaw
      Sliding Window Log precise, memory heavy
      Sliding Window Counter balanced, common default
      Token Bucket allows bursts
      Leaky Bucket smooths to constant rate
    Distributed Systems
      Local counters break silently
      Fix: shared counter e.g. Redis
    Keying
      By IP
      By User / API key
      By Endpoint
      Globally
    Client Signal
      429 Too Many Requests
      Retry-After header
```

| If you remember only 5 things |
|---|
| 1. Rate limiting caps how many requests a client can make in a time window, protecting the system from abuse, bugs, and overload. |
| 2. Token Bucket is the most commonly used algorithm in practice — it allows short bursts while enforcing a steady average rate. |
| 3. Fixed Window is simple but has a boundary flaw — nearly double the limit can slip through right at the edge of two windows. |
| 4. In a multi-server system, counting locally on each server silently breaks the limit — use a shared store like Redis for a true global count. |
| 5. A blocked request should return 429 Too Many Requests with a Retry-After header, so clients know exactly when to try again. |

---

*This file is written in GitHub-flavored Markdown with Mermaid diagrams — it will render natively on GitHub, GitLab, and most modern Markdown viewers.*
