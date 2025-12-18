# Architectural Characteristics

An architectural characteristic is a quality attribute of a system that is critical to its success. These characteristics are non-functional requirements that guide design decisions and define the system's operational and developmental properties.

---

### Accessibility
- **Description:** The design of a system that allows it to be used by people with disabilities (e.g., visual, auditory, motor, cognitive).
- **How it's measurable:** Compliance with standards like WCAG (Web Content Accessibility Guidelines) at levels A, AA, or AAA; percentage of user interface elements that are keyboard-navigable and screen-reader compatible.
- **Examples in practice:** Providing alternative text for images, ensuring sufficient color contrast, enabling keyboard navigation, and offering transcripts for audio content.

### Agility
- **Description:** While more of a process-related term, in architecture it refers to the ability to respond quickly to changes in the market, requirements, or technology. It is heavily enabled by other characteristics like Modifiability, Deployability, and Testability.
- **How it's measurable:** Lead time for changes (from idea to deployment); development cycle time; mean time to recovery (MTTR).
- **Examples in practice:** An architecture that allows small, independent teams to deploy features without extensive coordination; a CI/CD pipeline that automates testing and deployment.

### Auditability
- **Description:** The ability of a system to provide a chronological record of activities and events. This is crucial for security, compliance, and debugging.
- **How it's measurable:** Percentage of critical operations that are logged with user context; ability to trace a transaction from start to finish through logs.
- **Examples in practice:** Logging every access to sensitive data in a dedicated, tamper-proof audit log; recording all administrative actions with timestamps and user IDs.

### Availability
- **Description:** The proportion of time that a system is accessible and operational to its users. Often expressed as a percentage.
- **How it's measurable:** Uptime percentage (e.g., 99.9%, 99.99%); Mean Time Between Failures (MTBF); Mean Time To Repair (MTTR).
- **Examples in practice:** A website that is expected to be online 99.99% of the time, allowing for only ~52 minutes of downtime per year. This is often achieved through fault-tolerance.

### Configurability
- **Description:** The ease with which a system's behavior can be changed by its operators or users without changing the code.
- **How it's measurable:** Number of features that can be enabled/disabled via configuration flags; time required to apply a configuration change.
- **Examples in practice:** Using feature flags to turn new functionality on or off; external configuration files (e.g., YAML, .properties) that control database connections or API endpoints.

### Consistency (Data)
- **Description:** Ensures that every read from a data store receives the most recently written value or an error. In distributed systems, this is a spectrum (e.g., strong consistency vs. eventual consistency).
- **How it's measurable:** Time for data to propagate to all nodes (replication lag); number of stale reads observed under load.
- **Examples in practice:** A banking transaction must reflect the latest balance immediately (strong consistency). A social media post's "like" count can take a few seconds to update across all servers (eventual consistency).

### Cost
- **Description:** The total cost associated with the system over its lifecycle, including development, infrastructure, maintenance, and operational costs.
- **How it's measurable:** Total Cost of Ownership (TCO); monthly cloud provider bill; developer-hours required for maintenance.
- **Examples in practice:** Choosing a serverless architecture to reduce idle infrastructure costs; selecting a free open-source library over a licensed commercial one.

### Deployability
- **Description:** The ease and speed with which a new version of the software can be deployed into production.
- **How it's measurable:** Deployment time; deployment frequency; number of manual steps in the deployment process.
- **Examples in practice:** Fully automated CI/CD pipelines; blue-green deployments; canary releases. A highly deployable system might allow for dozens of deployments per day.

### Durability
- **Description:** The guarantee that once data is successfully written and committed, it will not be lost, even in the event of a system failure.
- **How it's measurable:** Data loss percentage over a given period.
- **Examples in practice:** A database that writes data to multiple disks and geographic locations before confirming a transaction; S3's 99.999999999% durability guarantee.

### Elasticity
- **Description:** The ability of a system to automatically and dynamically add or remove resources (e.g., servers, memory) to match the current demand as closely as possible.
- **How it's measurable:** Time to provision new resources; resource utilization percentage (should be high but not saturated).
- **Examples in practice:** An e-commerce site automatically spinning up more web servers during a Black Friday sale and shutting them down afterward to save costs.

### Evolvability
- **Description:** The ability of a system to support future, unforeseen changes and growth easily. It's a long-term strategic view of modifiability.
- **How it's measurable:** Difficulty of adding a new architectural component; time to implement a major new feature that was not part of the original design.
- **Examples in practice:** A microservices architecture where new services can be added without impacting existing ones; designing with clear interfaces and abstractions.

### Extensibility
- **Description:** The ease with which a system can be enhanced with new functionality through well-defined extension points, without modifying the core code.
- **How it's measurable:** Number of available extension points (plugins, APIs); time to develop and integrate a new plugin.
- **Examples in practice:** A web browser that allows users to install extensions; a code editor like VS Code with its marketplace of plugins.

### Fault-tolerance
- **Description:** The ability of a system to continue operating, possibly at a reduced level, even after one or more of its components fail.
- **How it's measurable:** Number of component failures the system can withstand before failing entirely; successful request percentage during a failure simulation.
- **Examples in practice:** A web application that stays online even if one of its database replicas goes down; a streaming service that continues to play video if a content delivery node fails.

### Interoperability
- **Description:** The ability of a system or its components to communicate, exchange information, and function cooperatively with other disparate systems.
- **How it's measurable:** Time to integrate with a new third-party system; number of supported data formats (e.g., JSON, XML, Protobuf).
- **Examples in practice:** Using standard REST APIs to connect a mobile app with a backend built on a completely different technology stack.

### Maintainability
- **Description:** The ease and speed with which a system can be changed, corrected, or adapted. It's about the efficiency of the development team over the system's lifetime.
- **How it's measurable:** Cyclomatic complexity of code; code coverage; time to fix a bug; time for a new developer to become productive.
- **Examples in practice:** Well-documented, clean code with a consistent style; a modular architecture where changes are isolated.

### Manageability / Operability
- **Description:** The ease with which a system can be monitored, operated, and controlled by an operations team.
- **How it's measurable:** Time to diagnose a problem; number of manual steps to perform routine operational tasks.
- **Examples in practice:** A system that provides a comprehensive dashboard with health checks and key metrics; automated alerts for critical issues.

### Modifiability
- **Description:** The ease with which a system can be changed, reflecting how quickly new features can be added or bugs can be fixed without introducing new problems.
- **How it's measurable:** Cost and time to make a specific change; number of components affected by a single change (coupling).
- **Examples in practice:** Changing a business rule in a single, isolated module instead of across ten different files. This is often achieved through high modularity.

### Modularity
- **Description:** A design principle where a system is decomposed into a set of distinct, independent, and cohesive modules with well-defined interfaces. This is a key enabler for modifiability and maintainability.
- **How it's measurable:** Coupling between modules; cohesion within modules.
- **Examples in practice:** An application structured into a "user management" module, a "payment processing" module, and an "order" module, each handling its own distinct responsibilities.

### Observability
- **Description:** The ability to understand the internal state of a system from its external outputs (logs, metrics, traces). It's about being able to ask new questions about the system without deploying new code.
- **How it's measurable:** Time to answer a question about system behavior; coverage of distributed tracing.
- **Examples in practice:** Using tools like Prometheus, Grafana, and Jaeger to collect and visualize metrics, logs, and traces from all components of a distributed system.

### Performance
- **Description:** An umbrella term for how effectively a system uses its resources to perform its functions within a given time frame. It encompasses latency, throughput, and efficiency.
- **How it's measurable:** Response time (latency) in milliseconds; transactions per second (throughput); resource utilization (CPU/memory percentage).
- **Examples in practice:** An API that responds to requests in under 100ms; a data processing pipeline that can handle 1 million events per second.

### Portability
- **Description:** The ease with which a system can be moved from one computing environment to another (e.g., from one cloud provider to another) with minimal changes.
- **How it's measurable:** Time and cost to migrate the system to a new environment; number of platform-specific dependencies.
- **Examples in practice:** A containerized application using Docker, which can run on any host that has a container runtime, be it on-premises or in any cloud.

### Reliability
- **Description:** The ability of a system to perform its required functions correctly and consistently over a specified period. It's a combination of availability and data integrity.
- **How it's measurable:** Mean Time Between Failures (MTBF); error rate.
- **Examples in practice:** A system that processes financial transactions without errors and remains available during business hours.

### Resilience
- **Description:** The ability of a system to handle and recover from failures gracefully. It's a broader concept than fault-tolerance, including the ability to heal and adapt to new failure modes.
- **How it's measurable:** Mean Time To Recovery (MTTR); success rate of operations during a chaos engineering experiment.
- **Examples in practice:** Implementing circuit breakers that prevent cascading failures; services that automatically restart and self-heal when they crash.

### Reusability
- **Description:** The degree to which parts of a software system can be used in other systems with minimal modification.
- **How it's measurable:** Number of times a component or library is used across different projects.
- **Examples in practice:** Creating a shared library for authentication that is used by multiple microservices within a company.

### Scalability
- **Description:** The ability of a system to handle a growing amount of work by adding resources, either vertically (adding more power to an existing server) or horizontally (adding more servers).
- **How it's measurable:** Performance degradation as load increases; cost per user/request at different scales.
- **Examples in practice:** A web application that can handle 100 users or 10 million users by adding more servers to its cluster.

### Security
- **Description:** The ability of a system to protect itself and its data from unauthorized access, use, disclosure, disruption, modification, or destruction.
- **How it's measurable:** Number of vulnerabilities found by a penetration test; time to detect and respond to a breach; compliance with security standards (e.g., ISO 27001, SOC 2).
-  **Examples in practice:** Encrypting all data at rest and in transit; using multi-factor authentication; implementing strict authorization rules.

### Simplicity
- **Description:** A design principle that aims to reduce complexity, making the system easier to understand, maintain, and reason about.
- **How it's measurable:** Code complexity metrics; cognitive load on developers (qualitative); number of components.
- **Examples in practice:** Choosing a simpler, well-understood technology over a more complex, cutting-edge one if it meets the requirements (KISS principle).

### Supportability
- **Description:** The ease with which a system can be supported by technical support staff, including diagnosing and resolving customer issues.
- **How it's measurable:** Mean time to resolve a support ticket; number of escalations to the engineering team.
- **Examples in practice:** Providing clear error messages with unique codes; building diagnostic tools for support staff to use.

### Testability
- **Description:** The ease with which a system or its components can be verified to behave correctly through tests.
- **How it's measurable:** Percentage of code covered by automated tests (code coverage); time to run the full test suite; effort required to write a new test.
- **Examples in practice:** Designing components with clear interfaces and using dependency injection to allow for easy mocking and testing in isolation.

### Usability
- **Description:** The ease with which end-users can learn, operate, and be effective with the system. It is a measure of user experience and satisfaction.
- **How it's measurable:** Time for a new user to complete a task; number of user errors; user satisfaction scores (e.g., NPS).
- **Examples in practice:** An intuitive and self-explanatory user interface; clear and helpful error messages; a logical workflow for completing tasks.

---

## Common Architectural Trade-offs

Architectural characteristics are not independent. Optimizing for one often comes at the expense of others. Understanding these trade-offs is a core skill of a software architect.

### Security
- **Negatively impacts:**
  - `*` **Performance:** Encryption, decryption, and security-related checks all consume CPU cycles, which can increase latency.
  - `*` **Usability:** Stricter password policies, multi-factor authentication, and frequent re-authentication can add friction for the user.
  - `*` **Simplicity & Maintainability:** Adding security measures introduces complexity into the system, making it harder to understand and maintain.

### High Availability & Fault-Tolerance
- **Negatively impacts:**
  - `*` **Cost:** Achieved through redundancy (e.g., multiple servers, data centers), which significantly increases infrastructure and operational costs.
  - `*` **Simplicity:** Managing redundant components, failover logic, and data replication adds significant complexity to the system.
  - `*` **Consistency:** In distributed systems, high availability can sometimes be at odds with strong consistency (see CAP theorem).

### Strong Consistency
- **Negatively impacts:**
  - `*` **Performance (Latency):** Ensuring all nodes in a distributed system have the same data requires coordination protocols that slow down write operations.
  - `*` **Availability:** In some failure scenarios (network partitions), a system might choose to become unavailable rather than serve potentially stale or inconsistent data.

### Modularity & Modifiability
- **Negatively impacts:**
  - `*` **Performance:** Introducing layers of abstraction, such as APIs between modules or microservices, can add network or processing overhead compared to a monolithic call.
  - `*` **Simplicity (in the small):** While it simplifies the overall system (in the large), the introduction of interfaces, dependency injection, and module boundaries can make a small part of the codebase seem more complex.

### Elasticity & Scalability
- **Negatively impacts:**
  - `*` **Simplicity:** Designing a system to be stateless and horizontally scalable is significantly more complex than building for a single server.
  - `*` **Consistency:** Maintaining data consistency across multiple, dynamically changing nodes is a major challenge in distributed systems.
  - `*` **Cost:** While elasticity can save money by matching demand, the initial development and infrastructure cost to enable it can be high.

### Reusability
- **Negatively impacts:**
  - `*` **Simplicity:** Building a generic, reusable component is often more complex and time-consuming than building a specific solution for the immediate problem.
  - `*` **Performance:** A generic component might include features that are not needed for a specific use case, adding unnecessary overhead.

### Observability
- **Negatively impacts:**
  - `*` **Performance:** Instrumenting code with extensive logging, tracing, and metrics collection consumes CPU, memory, and network bandwidth, which can impact application performance.
  - `*` **Cost:** Storing and analyzing vast amounts of telemetry data (logs, traces, metrics) can be very expensive.

### Usability
- **Negately impacts:**
  - `*` **Performance:** A highly interactive and rich user interface might require fetching and processing large amounts of data, which can lead to slower response times.
  - `*` **Simplicity:** Building a polished and intuitive user experience often requires significant development effort and can add complexity to the frontend codebase.