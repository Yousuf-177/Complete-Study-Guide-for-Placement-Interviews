# Software Engineering Practices — Placement Interview Cheat Sheet

> Dense, high-signal revision notes. Built for fast recall under interview pressure, not casual reading.

---

# 1. Topic Overview

- **Definition**: Software Engineering (SE) = the systematic, disciplined, quantifiable approach to designing, developing, testing, deploying, and maintaining software.
- **Purpose**: Convert unpredictable, ad-hoc "coding" into a repeatable engineering discipline — controlling cost, quality, schedule, and risk.
- **Why it exists**: The "software crisis" (1960s-70s) — projects were late, over-budget, buggy, unmaintainable. SE emerged to bring engineering rigor (process, metrics, standards) to software the way civil engineering brought rigor to construction.
- **Real-world usage**: Every company shipping software at scale — SDLC models, Agile ceremonies, CI/CD pipelines, code review, testing pyramids, design principles (SOLID) are used daily at FAANG-scale and startups alike.
- **Industry relevance**: Distinguishes a "coder" from an "engineer." Interviewers use SE-practice questions to gauge whether you can operate in a team, write maintainable code, and reason about tradeoffs — not just solve LeetCode.
- **Interview importance**: Common in HR + technical rounds, especially for freshers — tests process awareness (Agile/Scrum), fundamentals (SDLC models), and code-quality thinking (SOLID, testing, design patterns).

---

# 2. Core Concepts

## 2.1 SDLC (Software Development Life Cycle)
- **Definition**: The structured sequence of phases software passes through: Requirements → Design → Implementation → Testing → Deployment → Maintenance.
- **Internal working**: Each phase has defined inputs/outputs (e.g., Requirements phase outputs an SRS document consumed by Design).
- **Why it matters**: Without a defined process, requirements get missed, testing is rushed, and maintenance costs explode.
- **Advantages**: Predictability, clear milestones, easier resource planning.
- **Disadvantages**: Rigid models (Waterfall) don't handle changing requirements well.
- **Analogy**: Like constructing a building — you don't start electrical wiring before the architectural blueprint is approved.
- **Common mistake**: Treating SDLC as strictly linear even in Agile — in reality, phases overlap/iterate.
- **Interview trap**: "Which SDLC model is best?" → There's no universal best; answer must be context-dependent (stable requirements → Waterfall; evolving requirements → Agile).

## 2.2 Software Requirements Specification (SRS)
- **Definition**: A formal document describing what the system should do (functional) and how well it should do it (non-functional).
- **Internal working**: Gathered via stakeholder interviews, use cases, user stories → structured into functional/non-functional requirement sections.
- **Why it matters**: Ambiguous requirements are the #1 cause of project failure/rework.
- **Advantages/Disadvantages**: Clear contract vs. overhead/rigidity if requirements evolve (common in Agile, where SRS is replaced by evolving backlogs).
- **Analogy**: A recipe — vague recipes ("add some sugar") cause inconsistent results; precise SRS = precise dish.
- **Interview trap**: Confusing functional (what the system does) vs non-functional (performance, security, scalability, usability) requirements.

## 2.3 Modularity, Cohesion & Coupling
- **Definition**: 
  - **Modularity**: breaking a system into independent, manageable units (modules).
  - **Cohesion**: how closely related/focused the responsibilities *within* a module are (want **HIGH**).
  - **Coupling**: how dependent modules are *on each other* (want **LOW**).
- **Internal working**: High cohesion = a module does one well-defined thing. Low coupling = changing one module doesn't ripple-break others.
- **Why it matters**: Directly determines maintainability, testability, and reusability.
- **Advantages**: Low coupling + high cohesion → easier to test in isolation, easier to change/extend.
- **Common mistake**: Believing "modular" automatically means "good" — a module can be modular but still have low cohesion (doing unrelated things) or high coupling (leaking internals).
- **Analogy**: A well-organized toolbox — screwdrivers together (high cohesion), not spread randomly across drawers, and each drawer works independently of the others (low coupling).
- **Interview trap**: "Give an example of tight coupling" → e.g., Class A directly instantiates and manipulates Class B's internal fields, instead of going through an interface.

## 2.4 Software Quality Attributes (the "-ilities")
- **Definition**: Non-functional properties: **Reliability, Maintainability, Usability, Efficiency, Portability, Scalability, Security, Testability**.
- **Why it matters**: Functional correctness alone doesn't make software "good" — a correct-but-unmaintainable system is a liability.
- **Interview trap**: Being asked to trade off two attributes (e.g., "security vs usability," "performance vs maintainability") — always answer with context-driven reasoning, not an absolute stance.

---

# 3. Technical Deep Dive

## 3.1 SDLC Models — Execution Flow

| Model | Flow | Best for | Worst for |
|---|---|---|---|
| **Waterfall** | Strictly sequential: Req→Design→Code→Test→Deploy→Maintain (no going back) | Stable, well-understood requirements (e.g., embedded/safety-critical systems) | Projects with evolving requirements |
| **V-Model** | Waterfall + a testing phase mapped to each dev phase (Unit test ↔ Design, System test ↔ Requirements) | Safety-critical systems (avionics, medical devices) | Fast-changing requirements |
| **Iterative** | Repeat build-test-refine cycles, expanding scope each time | Medium-complexity projects with partial clarity | Very tight deadlines with no iteration budget |
| **Spiral** | Iterative + explicit **risk analysis** at each loop (Boehm's model) | Large, high-risk projects | Small/simple projects (overhead not justified) |
| **Agile** | Short iterations ("sprints"), continuous feedback, evolving backlog | Rapidly changing requirements, need for fast delivery | Projects needing heavy upfront documentation/compliance |
| **RAD (Rapid Application Development)** | Heavy prototyping + user feedback loops, minimal planning | UI-heavy apps needing fast prototypes | Large, complex systems with many dependencies |

## 3.2 Agile Methodology — Deep Dive

- **4 Values (Agile Manifesto)**:
  1. Individuals and interactions **over** processes and tools
  2. Working software **over** comprehensive documentation
  3. Customer collaboration **over** contract negotiation
  4. Responding to change **over** following a plan
- **12 Principles** (know a few, not all): early/continuous delivery, welcome changing requirements even late, deliver working software frequently, business + devs work together daily, sustainable pace, technical excellence, simplicity, self-organizing teams, regular reflection/tuning.

### Scrum (most common Agile framework asked about)
- **Roles**: Product Owner (owns backlog/priorities), Scrum Master (removes blockers, facilitates process), Development Team (builds the product).
- **Artifacts**: Product Backlog (all desired work), Sprint Backlog (subset committed for this sprint), Increment (working product slice at sprint end).
- **Ceremonies/Events**:
  - **Sprint Planning** — decide what goes into the sprint.
  - **Daily Standup** — 15 min, "what I did / will do / blockers."
  - **Sprint Review** — demo increment to stakeholders.
  - **Sprint Retrospective** — team reflects on process improvements.
- **Sprint length**: typically 1–4 weeks (2 weeks most common).

### Kanban (alternative Agile framework)
- Visual board (To Do / In Progress / Done), **continuous flow** (no fixed sprints), **WIP limits** (cap on how many items can be "in progress" at once to prevent overload/bottlenecks).
- **Scrum vs Kanban**: Scrum = fixed-length sprints + defined roles; Kanban = continuous flow + WIP limits, no mandatory roles/ceremonies.

## 3.3 Version Control (Git) — Execution Flow
- **Working directory → Staging area (index) → Local repo (commit) → Remote repo (push)**.
- **Branching model**: `main`/`master` (stable), `develop` (integration), `feature/*` (isolated work), `hotfix/*` (urgent prod fixes).
- **Merge vs Rebase**:
  - `merge`: combines histories, creates a merge commit, preserves true history (non-linear).
  - `rebase`: replays your commits on top of another branch, creates linear history, **rewrites commit hashes** (dangerous on shared/public branches).
- **Merge conflict**: occurs when the same lines are changed differently on both branches being merged — resolved manually by choosing/combining changes, then committing.

## 3.4 Testing — Execution Flow & Levels
```
Unit Test → Integration Test → System Test → Acceptance Test
(smallest, fastest, most numerous)   →   (largest, slowest, fewest — "Testing Pyramid")
```
- **Unit Testing**: tests a single function/method/class in isolation (mocks/stubs for dependencies).
- **Integration Testing**: tests interaction between multiple modules/components (e.g., service + database).
- **System Testing**: tests the entire integrated system against requirements (black-box).
- **Acceptance Testing (UAT)**: validates the system meets business needs — often done by/with the client.
- **Regression Testing**: re-running existing tests after a code change to ensure nothing broke.
- **Smoke Testing**: quick, shallow test to check the build isn't fundamentally broken before deeper testing.

## 3.5 Design Patterns — Categories & Internal Mechanism
- **Creational** (object creation): Singleton, Factory, Abstract Factory, Builder, Prototype.
- **Structural** (composing objects/classes): Adapter, Decorator, Facade, Proxy, Composite, Bridge.
- **Behavioral** (object interaction/responsibility): Observer, Strategy, Command, State, Iterator, Template Method, Chain of Responsibility.

## 3.6 CI/CD — Execution Flow
```
Code Commit → CI Server triggers Build → Run Automated Tests → 
   (if pass) → Package Artifact → Deploy to Staging → 
   (manual/automated gate) → Deploy to Production (CD)
```
- **Continuous Integration (CI)**: merge code frequently (multiple times/day), auto-build + auto-test on every commit — catches integration bugs early.
- **Continuous Delivery**: code is always in a deployable state; deployment to production is a manual trigger.
- **Continuous Deployment**: every passing change is automatically deployed to production — no manual gate.

---

# 4. Implementations

## 4.1 SOLID in Code (Basic → Production-grade thinking)

**S — Single Responsibility Principle**: a class should have only one reason to change.
```python
# Bad: one class does two jobs (data + persistence)
class Report:
    def generate(self): ...
    def save_to_db(self): ...

# Good: split responsibilities
class Report:
    def generate(self): ...

class ReportRepository:
    def save(self, report): ...
```

**O — Open/Closed Principle**: open for extension, closed for modification.
```python
# Bad: must edit this function every time a new shape is added
def area(shape):
    if shape.type == "circle": return 3.14 * shape.r ** 2
    elif shape.type == "square": return shape.side ** 2

# Good: extend via new subclasses, no existing code touched
class Shape:
    def area(self): raise NotImplementedError

class Circle(Shape):
    def area(self): return 3.14 * self.r ** 2

class Square(Shape):
    def area(self): return self.side ** 2
```

**L — Liskov Substitution Principle**: subtypes must be substitutable for their base type without breaking correctness.
```python
# Classic violation: Square "is-a" Rectangle mathematically,
# but forcing equal sides breaks Rectangle's expected behavior
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h

class Square(Rectangle):
    def set_width(self, w): self.width = self.height = w   # breaks caller expectations
```

**I — Interface Segregation Principle**: don't force classes to implement methods they don't use — prefer many small, specific interfaces over one bloated one.

**D — Dependency Inversion Principle**: depend on abstractions, not concrete implementations.
```python
# Bad: tightly coupled to a specific implementation
class Notification:
    def __init__(self):
        self.email = EmailService()   # hard dependency

# Good: depend on abstraction, inject the concrete implementation
class Notification:
    def __init__(self, service: MessageService):   # interface/abstract type
        self.service = service
```

## 4.2 Best Practices
- Write tests **before or alongside** code (TDD: Red → Green → Refactor).
- Small, focused commits with meaningful messages (imperative mood: "Fix null pointer in parser", not "fixed bug").
- Code reviews mandatory before merge — catch bugs, share knowledge, enforce standards.
- Keep functions short, single-purpose; avoid deep nesting (>3 levels is a smell).

## 4.3 Anti-Patterns
- **God Object**: one class doing everything (violates SRP).
- **Spaghetti Code**: tangled control flow, no clear structure.
- **Copy-Paste Programming**: duplicated logic instead of abstraction (violates DRY).
- **Golden Hammer**: using one familiar tool/pattern for every problem regardless of fit.
- **Premature Optimization**: optimizing before profiling shows it's needed — wastes time, adds complexity.

## 4.4 Debugging Strategies
- Reproduce reliably → isolate (binary search through code/commits, e.g. `git bisect`) → inspect state (logs, debugger breakpoints) → form hypothesis → fix → add a regression test.

---

# 5. Important Features

| Feature/Practice | Purpose | Benefit | Tradeoff | When to use / avoid |
|---|---|---|---|---|
| **TDD (Test-Driven Development)** | Write failing test → minimal code to pass → refactor | Forces testable design, high coverage by construction | Slower initial velocity, discipline-heavy | Use for core business logic; skip for throwaway prototypes |
| **Pair Programming** | Two devs, one keyboard | Fewer bugs, knowledge sharing | 2x person-hours for a task | Use for complex/critical code; avoid for simple boilerplate |
| **Code Review** | Peer check before merge | Catch bugs early, spread ownership, enforce standards | Slows merge velocity | Always use in team settings |
| **Refactoring** | Improve internal structure without changing external behavior | Reduces technical debt, improves maintainability | Risk of introducing bugs if untested | Do continuously, backed by tests |
| **Design Patterns** | Reusable solution templates to common design problems | Shared vocabulary, proven solutions | Overuse adds unnecessary abstraction | Use when the *problem* matches the pattern's intent, not by default |
| **Microservices** | Split system into independently deployable services | Independent scaling/deployment, fault isolation | Operational complexity, network latency, distributed debugging | Use for large teams/systems with clear domain boundaries; avoid for small apps ("premature microservices") |
| **Monolith** | Single deployable unit | Simpler deployment/debugging initially | Scaling and team-ownership harder as it grows | Good starting point for most new products |

---

# 6. Theoretical Concepts

- **Boehm's Spiral Model**: risk-driven process model — each cycle = Determine objectives → Identify/resolve risks → Develop/test → Plan next iteration.
- **Conway's Law**: "Organizations design systems that mirror their own communication structure." — explains why microservices often map to team boundaries.
- **Brooks's Law**: "Adding manpower to a late software project makes it later" (from *The Mythical Man-Month*) — onboarding cost + communication overhead outweighs added capacity short-term.
- **The Cone of Uncertainty**: estimation accuracy improves as a project progresses — early estimates can be off by 4x, late-stage estimates converge near actual effort.
- **Technical Debt**: metaphor (Ward Cunningham) — shortcuts taken now accrue "interest" (extra future work) just like financial debt.
- **DRY (Don't Repeat Yourself)**: every piece of knowledge should have a single, unambiguous representation in a system.
- **KISS (Keep It Simple, Stupid)**: prefer the simplest design that solves the problem.
- **YAGNI (You Aren't Gonna Need It)**: don't build functionality until it's actually required — avoids speculative over-engineering.

---

# 7. Formulas / Estimation Models

### 7.1 COCOMO (Constructive Cost Model) — Basic Model
**Formula**: `Effort = a × (KLOC)^b` person-months
- `KLOC` = thousands of lines of code (estimated).
- `a, b` = constants depending on project type:

| Project Type | a | b |
|---|---|---|
| Organic (small, simple, experienced team) | 2.4 | 1.05 |
| Semi-detached (medium complexity/team) | 3.0 | 1.12 |
| Embedded (complex, tight constraints) | 3.6 | 1.20 |

**Interpretation**: Larger/more complex projects don't just scale effort linearly — the exponent `b > 1` captures the fact that complexity/coordination overhead grows faster than size.
**Usage example**: A 50 KLOC organic project → `Effort = 2.4 × 50^1.05 ≈ 145` person-months.
**Shortcut understanding**: More lines of code ≠ proportionally more effort — coordination overhead compounds.

### 7.2 Function Point Analysis (FPA)
Estimates size/complexity based on **functional user requirements** (inputs, outputs, inquiries, files, interfaces) rather than lines of code — useful because it's **language-independent** (unlike KLOC-based COCOMO).

### 7.3 Cyclomatic Complexity (McCabe's Metric)
**Formula**: `V(G) = E - N + 2P`
- `E` = edges in control flow graph, `N` = nodes, `P` = connected components (usually 1 for a single program).
- Simplified: `V(G) = number of decision points (if/while/for/case) + 1`.
**Interpretation**: Measures the number of **independent linear paths** through code — directly indicates minimum number of test cases needed for full branch coverage, and correlates with maintainability (higher = harder to test/understand; >10 is a common "refactor this" threshold).

---

# 8. Concept Map / Flow

```
Software Engineering
 ├── Process Models (SDLC)
 │    ├── Waterfall / V-Model   (plan-driven)
 │    ├── Spiral                (risk-driven)
 │    └── Agile                 (change-driven)
 │         ├── Scrum (roles, artifacts, ceremonies)
 │         └── Kanban (WIP limits, continuous flow)
 ├── Requirements Engineering
 │    ├── Functional requirements
 │    └── Non-functional requirements (the "-ilities")
 ├── Design
 │    ├── Principles (SOLID, DRY, KISS, YAGNI)
 │    ├── Design Patterns (Creational / Structural / Behavioral)
 │    └── Architecture (Monolith vs Microservices, Layered, Event-driven)
 ├── Implementation
 │    ├── Coding standards
 │    └── Version Control (Git: branch/merge/rebase)
 ├── Testing
 │    ├── Unit → Integration → System → Acceptance  (Testing Pyramid)
 │    └── TDD / BDD
 ├── Deployment
 │    └── CI/CD (Continuous Integration/Delivery/Deployment)
 └── Maintenance
      ├── Corrective / Adaptive / Perfective / Preventive
      └── Technical Debt management
```

**Learning/Execution Order**: Requirements → Design → Implementation → Testing → Deployment → Maintenance (mirrors SDLC itself — this IS the pipeline).

---

# 9. Mnemonics & Memory Tricks

- **SOLID** — Single, Open-closed, Liskov, Interface segregation, Dependency inversion.
- **DRY, KISS, YAGNI** — "Don't repeat, keep simple, you won't need it" — the three guiding restraint principles.
- **Testing Pyramid** — "**U**nicorns **I**nvest **S**lowly, **A**lways" → **U**nit, **I**ntegration, **S**ystem, **A**cceptance (bottom = many/fast, top = few/slow).
- **Scrum Artifacts** — "**P**eople **S**hip **I**ncrements" → **P**roduct backlog, **S**print backlog, **I**ncrement.
- **4 Maintenance Types** — "**CAPP**" → **C**orrective (fix bugs), **A**daptive (adapt to new environment), **P**erfective (improve performance/features), **P**reventive (refactor to avoid future issues).
- **Design Pattern Categories** — "**C**reate **S**tructures **B**ehaviorally" → **C**reational, **S**tructural, **B**ehavioral.
- **COCOMO exponent intuition**: "bigger project, bigger-than-linear pain" — remember `b > 1` always.

---

# 10. Comparisons

### Waterfall vs Agile

| Aspect | Waterfall | Agile |
|---|---|---|
| Requirements | Fixed upfront | Evolve continuously |
| Delivery | One final delivery | Frequent incremental delivery |
| Customer involvement | Mostly at start/end | Continuous |
| Change tolerance | Low (expensive late changes) | High (built for change) |
| Documentation | Heavy | Lightweight, "just enough" |
| Best fit | Regulatory/safety-critical, well-understood domains | Startups, evolving products |

### Unit Test vs Integration Test

| Aspect | Unit Test | Integration Test |
|---|---|---|
| Scope | Single function/class | Multiple components together |
| Speed | Very fast | Slower (real dependencies: DB, network) |
| Dependencies | Mocked/stubbed | Real (or realistic test doubles) |
| Failure signal | Pinpoints exact broken unit | Signals a broken *interaction* |

### Verification vs Validation
- **Verification**: "Are we building the product **right**?" (conforms to spec — reviews, static analysis, unit tests).
- **Validation**: "Are we building the **right** product?" (meets user needs — UAT, beta testing).

### Merge vs Rebase (Git)

| Aspect | Merge | Rebase |
|---|---|---|
| History | Preserves true (non-linear) history | Creates clean linear history |
| Commit hashes | Unchanged | Rewritten |
| Safety on shared branches | Safe | Dangerous (never rebase public/shared history) |

### Monolith vs Microservices

| Aspect | Monolith | Microservices |
|---|---|---|
| Deployment | Single unit | Independent per service |
| Scaling | Scale whole app | Scale individual services |
| Complexity | Simpler ops, harder to scale teams | Complex ops (networking, monitoring), easier team autonomy |
| Failure impact | One bug can crash everything | Isolated failure domains (with proper design) |

---

# 11. Interview Section

**Q1: What's the difference between Verification and Validation?**
A: Verification checks if the software is built according to spec ("right way"); Validation checks if the software meets actual user needs ("right product"). Verification uses reviews/static checks; Validation uses UAT/beta testing.

**Q2: Why is Waterfall considered risky for modern software projects?**
A: It assumes requirements are fully known upfront and doesn't accommodate change — by the time you reach Testing, requirements may already be outdated, and defects found late are expensive to fix (cost-to-fix grows exponentially the later a defect is found).

**Q3: Explain SOLID with one real example each.**
A: (see Section 4.1 code examples — be ready to explain each letter with a concrete before/after snippet.)

**Q4: What is technical debt, and is it always bad?**
A: Shortcuts taken to ship faster that must be "paid back" later via refactoring. Not inherently bad — *deliberate, tracked* technical debt (e.g., "we'll hardcode this for the MVP, ticket #123 to fix") is a valid strategic tradeoff; *reckless, untracked* debt is the dangerous kind.

**Q5 (Scenario): Your team's Scrum sprint keeps failing to complete committed stories. What would you investigate?**
A: Check if: stories are properly estimated (planning poker/story points), scope creep is happening mid-sprint, the team is being interrupted by unplanned work, or story sizes are too large (should be broken into smaller vertical slices).

**Q6 (Debugging): A regression test suite passes locally but fails in CI. What do you check?**
A: Environment differences (OS, dependency versions, timezone/locale), flaky tests (order-dependence, shared state, race conditions), missing environment variables/config, or a difference in test data setup between local and CI.

**Q7 (Conceptual trap): "Agile means no documentation and no planning."**
A: False — Agile values working software *over comprehensive* documentation, not *zero* documentation; it still plans, just incrementally (sprint-by-sprint) rather than all upfront.

**Q8: What is the difference between a Design Pattern and a Design Principle?**
A: A principle (e.g., SOLID) is a general guideline for *why* to design a certain way; a pattern (e.g., Observer, Factory) is a concrete, reusable *solution template* to a specific recurring design problem — patterns are often derived from applying principles.

**Q9 (Output prediction): In Git, you rebase a feature branch onto main, then push. Teammates who already pulled the old feature branch — what happens?**
A: Their local history diverges from the rewritten remote history — they'll get conflicts/duplicate commits on next pull; they must either force-pull or rebase their own work on the new history. This is exactly why rebasing shared/public branches is discouraged.

**Q10 (Follow-up bait): "Which testing type would you prioritize with limited time?"**
A: Unit tests — cheapest, fastest, catch the most bugs per dollar of effort (this is the entire rationale behind the Testing Pyramid shape).

---

# 12. Real-World Engineering Perspective

- **Industry usage**: Nearly every tech company runs some Agile variant (Scrum/Kanban hybrid), enforces mandatory PR/code review, and runs CI/CD pipelines (GitHub Actions, Jenkins, GitLab CI) gating merges on automated test suites.
- **Scaling challenges**: As teams grow, Conway's Law kicks in — communication overhead grows quadratically with team size (Brooks's Law), pushing large orgs toward microservices + autonomous teams.
- **Production issues**: Poor test coverage → regressions slipping to prod; tight coupling → a small change requires touching dozens of files ("shotgun surgery" code smell); skipped code reviews → security/logic bugs reaching customers.
- **Security concerns**: Skipping input validation/design review stages is a top root cause of vulnerabilities (SQL injection, XSS) — security should be a first-class NFR, not an afterthought ("shift-left security").
- **Reliability concerns**: Lack of proper testing/rollback strategy in CD pipelines → bad deploys directly hit users; mitigated via canary releases, feature flags, blue-green deployments.
- **Monitoring & Maintainability**: Well-designed systems (SOLID, low coupling) are easier to instrument, log, and safely modify; poorly designed systems accumulate technical debt that eventually cripples velocity ("high debt = every change breaks something else").
- **Performance bottlenecks**: Premature optimization (violating YAGNI/KISS) often *adds* complexity without measurable benefit — the standard advice is "measure first, optimize the actual bottleneck" (profiling before optimizing).

---

# 13. Revision Zone

### 1-Minute Revision
- SDLC = Requirements → Design → Code → Test → Deploy → Maintain.
- Agile > Waterfall for changing requirements; Waterfall > Agile for fixed, well-understood, regulated projects.
- SOLID = 5 design principles for maintainable OOP code.
- Testing Pyramid: many Unit tests, fewer Integration, fewest System/Acceptance (fast & cheap at the bottom).
- Verification = "right way" (spec-conformance); Validation = "right product" (user needs).
- DRY/KISS/YAGNI = don't repeat, keep simple, don't over-build.
- CI = auto build/test on every commit; CD = auto-deployable / auto-deployed.

### 5-Minute Revision (Key Concepts + Traps)
- Know all 4 Agile Manifesto values and be able to state them in your own words.
- Scrum roles (PO, SM, Dev Team) + artifacts (Product Backlog, Sprint Backlog, Increment) + ceremonies (Planning, Standup, Review, Retro).
- SOLID — be ready to write a 3-4 line before/after code snippet for any letter on demand.
- Merge vs Rebase — know rebase rewrites history and is unsafe on shared branches.
- COCOMO formula `Effort = a·(KLOC)^b` — know it's non-linear (b > 1).
- Cyclomatic complexity ≈ number of decision points + 1 — higher = harder to test/maintain.
- Common trap: Agile ≠ "no planning/no docs" — it's *iterative* planning/docs, not zero.
- Common trap: Fractional-knapsack-style "greedy always works" thinking doesn't apply to design tradeoffs — always justify context (e.g., microservices aren't universally "better" than monoliths).

### Last-Minute Interview Bullets
- Always frame answers with **tradeoffs**, not absolutes — interviewers reward "it depends, here's why."
- Have one **concrete example** ready for: a SOLID violation you fixed, a merge conflict you resolved, a bug you debugged systematically, and a time you used a design pattern.
- Be ready to explain **why** a practice exists (root cause), not just define it — e.g., don't just say "TDD is red-green-refactor," explain *why* writing the test first forces better design.

---

# 14. Hidden Insights

- **Lesser-known fact**: The Waterfall model as originally described by Winston Royce (1970) *explicitly recommended iteration/feedback loops* between adjacent phases — the "strictly one-way" version most people criticize is actually a common misreading of the original paper.
- **Internal optimization insight**: Git rebase's "danger" isn't really about rebase itself — it's about rewriting *shared/pushed* history; rebasing your own *unpushed* local commits is completely safe and often preferred for a clean history.
- **Historical design reason**: Microservices exist largely because of *organizational* scaling limits (Conway's Law), not purely technical ones — many companies successfully run enormous monoliths (e.g., early-stage Shopify, Basecamp) far longer than commonly assumed necessary.
- **Common misconception**: "100% code coverage" does not mean "bug-free" — coverage measures *lines executed* by tests, not *correctness of assertions*; you can have 100% coverage with weak/meaningless assertions.
- **Interview gold nugget**: When discussing Agile vs Waterfall, mentioning that **hybrid models (Water-Scrum-Fall)** are extremely common in real enterprises — upfront planning/compliance (Waterfall-like) combined with Agile execution — shows real-world exposure beyond textbook definitions.
- **Hidden insight on technical debt**: Not all debt is equal — "reckless & unintentional" debt (bad code from not knowing better) is the most dangerous quadrant in Martin Fowler's Technical Debt Quadrant, worse than "deliberate & intentional" debt taken knowingly as a tradeoff.

---

*End of cheat sheet. Revisit the Concept Map (Section 8) and Interview Section (Section 11) the night before your interview for maximum recall efficiency.*
