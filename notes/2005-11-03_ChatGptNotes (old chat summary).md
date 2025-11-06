# Architectural Learning Summary

This document summarizes key architectural concepts and principles discussed.

---

## 1. The Role and Mindset of a Modern Architect

- **Architect as a Rule-Maker:** The architect's primary role is to create principles, guidelines, and frameworks that empower development teams to make consistent decisions, rather than dictating every choice.
- **Continuous Improvement:** Architecture is not static. It requires constant observation of the running system to identify areas for improvement as business needs and technologies evolve.
- **Technology Stewardship:** An architect must track technology trends, not for the sake of novelty, but to re-evaluate if existing solutions are still optimal. The focus should be on understanding the *differences* and trade-offs between tools (e.g., Kafka vs. RabbitMQ).
- **Ensuring Architectural Integrity:** Avoid being a "drive-by architect." Architectural intent must be sustained through automated checks (fitness functions), clear documentation, and team alignment.
- **Business & Organizational Acumen:** A great architect understands the business domain, its drivers, and the organizational structure. Decisions must be aligned with business value and navigated through the company's political landscape.
- **Architecture vs. Implementation:** The line is blurry. Architecture defines intent, but it's a continuous feedback loop with development, where implementation validates and refines architectural decisions.

---

## 2. Core Architectural Principles

- **Everything is a Trade-off:** There are no universal "best practices." Every decision depends on context.
- **Quality Attributes as Drivers:** Non-functional requirements (the "-ilities") are the primary drivers of architectural structure.
- **ADRs (Architecture Decision Records):** Document every significant decision, including context, options considered, and consequences.
- **Architectural Fitness Functions:** Automate the verification of architectural principles (e.g., modularity, coupling) with tests that run as part of the CI/CD pipeline.

---

## 3. Documenting Architecture

### ARC42 Framework
A structured template for documenting software architecture.
- **1. Introduction and Goals:** The "why." Business drivers and top quality goals.
- **2. Constraints:** The "rules." Technical, organizational, and legal limitations.
- **3. Context and Scope:** The "boundaries." System inputs, outputs, and external dependencies.
- **4. Solution Strategy:** The "big picture." Fundamental technology and pattern choices.
- **5. Building Block View:** The "what." Static decomposition of the system. This is where C4 Level 2 (Container) and C4 Level 3 (Component) diagrams fit.
- **6. Runtime View:** The "how." Scenarios showing how building blocks interact.
- **7. Deployment View:** The "where." Mapping of software to infrastructure.
- **8. Crosscutting Concepts:** The "house rules." Consistent solutions for logging, security, etc.
- **9. Architectural Decisions:** The "why" behind choices (ADRs).
- **10. Quality Requirements:** The "how well." Measurable quality scenarios (QAS).
- **11. Risks and Technical Debt:** Known issues and potential problems.
- **12. Glossary:** Definitions of domain and technical terms.

### C4 Modeling
A visual language for communicating architecture at different levels of detail.
- **Level 1: Context:** Shows the system as a black box in its environment.
- **Level 2: Container:** Zooms into the system to show its deployable units (services, databases, etc.).
- **Level 3: Component:** Zooms into a container to show its internal components.
- **Diagramming Messy Systems:** For complex dependencies (e.g., many services using a message broker), do not duplicate icons. Use a single icon and bundle relationships, using labels to specify details (e.g., queue names). Move detailed topology to a separate, more focused diagram.

---

## 4. Architectural Patterns

| Pattern | Agility | Deploy | Test | Perf | Scale | Simplicity | Cost |
|---|---|---|---|---|---|---|---|
| Layered | ⚙️ | ✅ | ✅ | ⚙️ | ⚙️ | ✅ | 💵 |
| Monolith | ⚙️ | ⚙️ | ✅ | ⚙️ | ⚙️ | ✅ | 💵 |
| Microkernel | ✅ | ✅ | ✅ | ⚙️ | ⚙️ | ✅ | 💵💵 |
| Event-Driven | ✅ | ✅ | ✅ | ⚙️ | ✅ | ⚙️ | 💵💵💵 |
| Pipeline | ✅ | ⚙️ | ✅ | ⚙️ | ⚙️ | ✅ | 💵 |
| Space-Based | ✅ | ✅ | ⚙️ | ✅ | ✅ | ⚙️ | 💵💵 |
| Microservices | ✅ | ✅ | ⚙️ | ⚙️ | ✅ | ⚙️ | 💵💵 |

- **Microkernel:** A core system with functionality extended through plug-ins. (e.g., IDEs like VS Code, CI/CD servers like Jenkins).
- **Event-Driven Architecture (EDA):**
    - **Broker Topology:** Decoupled publishers and subscribers. Fast and parallel but makes error handling and workflow coordination difficult.
    - **Mediator Topology:** A central orchestrator manages workflows. Simplifies coordination but the mediator can become a bottleneck, both technically and organizationally (Conway's Law).
- **Pipeline (Pipe-and-Filter):** Used for sequential data processing (e.g., ETL jobs, CI/CD pipelines, middleware chains). Simple and linear, but can be a scaling bottleneck.
- **Space-Based Architecture:** Solves high-scalability issues by removing the central database bottleneck. Uses a replicated in-memory data grid. Complex and expensive but offers near-infinite scalability.
- **Microservices:** System of small, autonomous, independently deployable services. Offers high agility and scalability but introduces distributed system complexity.
    - **Service Templates:** A "starter kit" or boilerplate project for creating new services. It defines structure and conventions, and often references shared libraries (e.g., NuGet packages) for common functionality like logging or auth.

---

## 5. Distributed Systems Concepts

- **Distributed Transactions:** Traditional two-phase commits (2PC) are brittle and rarely used in modern systems.
- **Saga Pattern:** Manages consistency across services through a sequence of local transactions. If a step fails, the saga executes compensating actions to semantically "undo" previous steps. There is no true rollback.
- **Outbox Pattern:** Ensures an event is published reliably after a database transaction commits. It achieves this by saving the event to an "outbox" table within the same local DB transaction. A separate process then sends the event from the table to the message broker.
- **CQRS (Command Query Responsibility Segregation):** A pattern, not an architecture. It separates the write model (Commands) from the read model (Queries). This allows for highly optimized read paths but introduces eventual consistency.
- **API Composition:** An aggregator service that calls multiple downstream services and joins their data to create a single response for a client.

---

## 6. Measuring Architecture (DevOps & Metrics)

- **DORA Metrics:** A set of four key metrics to measure the performance of a software development team.
    1.  **Deployment Frequency:** How often you deploy. (Measures Throughput)
    2.  **Lead Time for Changes:** Time from commit to production. (Measures Throughput)
    3.  **Change Failure Rate:** Percentage of deployments causing a failure. (Measures Stability)
    4.  **Time to Restore Service (MTTR):** Time to recover from a failure. (Measures Stability)
- **ATAM (Architecture Tradeoff Analysis Method):** A structured, workshop-based method to evaluate architectural decisions against quality attributes and business goals. It helps identify risks, trade-offs, and sensitivity points.

---

## 7. Cautionary Tales in Engineering

- **Vasa Ship (1628):** Scope creep (too many cannons) led to instability and failure.
- **F-16 Fighter Jet:** A clear, constrained vision (lightweight, agile) succeeded where a bloated, multi-requirement design failed.
- **Mars Climate Orbiter (1999):** Interface mismatch (imperial vs. metric units) caused system loss.
- **Knight Capital (2012):** A deployment error (re-activating old code) led to catastrophic financial loss in minutes.