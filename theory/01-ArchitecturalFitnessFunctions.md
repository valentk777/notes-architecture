# Theory: Architectural Fitness Functions

As defined in *Building Evolutionary Architectures* and *Fundamentals of Software Architecture*, an architectural fitness function provides an objective integrity assessment of some architectural characteristic.

They are a mechanism to enforce architectural goals and prevent architectural drift over time.

## Scopes of Fitness Functions

Fitness functions can be categorized by the scope they cover:

### 1. Atomic / Unit Test Level
These are the smallest, most focused fitness functions. They typically run as part of a normal unit test suite.

- **Example:** A static analysis check that fails the build if a developer adds a dependency from a `Domain` layer project to a `UI` layer project.
- **Trade-offs:**
    - **Pros:** Very fast, cheap to run, provide immediate feedback.
    - **Cons:** Very narrow view, cannot test emergent system properties.

### 2. Component / Integration Test Level
These functions test an entire component or service in isolation, but as a deployable unit. This is the category our `WebApplicationFactory` test falls into.

- **Example:** Verifying the resilience (Circuit Breaker), latency, or security (authentication/authorization) of a single microservice by testing its API endpoints.
- **Trade-offs:**
    - **Pros:** More realistic than atomic tests, can verify architectural characteristics of a whole component, still relatively fast.
    - **Cons:** Cannot detect issues that arise from the interaction between multiple components.

### 3. Holistic / System Level
These are the largest and most comprehensive fitness functions. They test the entire system, or a significant portion of it, including the interactions between services.

- **Example:** A chaos engineering experiment (like using Chaos Monkey) that randomly terminates services in a Kubernetes cluster and asserts that the overall system remains available and responsive to user requests.
- **Trade-offs:**
    - **Pros:** Highest fidelity, tests real-world emergent system behavior.
    - **Cons:** Expensive, slow to run, complex to set up and maintain, results can be harder to interpret.

## Summary

A mature architecture uses a combination of all three types of fitness functions to ensure architectural goals are met at every level of the system, from a single line of code to the entire distributed ecosystem.
