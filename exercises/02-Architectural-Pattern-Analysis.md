# Exercise 02: Architectural Pattern Analysis

This exercise analyzes a real-world system to understand its underlying architectural characteristics, moving beyond simple pattern labels.

## 1. System Description

The system is described as follows:

- **Composition:** 5 services, each with its own database.
- **Frontend Access:** A BFF (Backend-for-Frontend) gateway sits in front of the services.
- **Service Communication (Synchronous):** Services make direct, synchronous REST calls to each other when they need data immediately.
- **Service Communication (Asynchronous):** Data is duplicated across service databases. This data is kept in sync via RabbitMQ messages.
- **Shared Dependency:** A shared Redis cache is used by most/all APIs to access a common subset of domain data.
- **Service Granularity:** The services are not considered "micro" (i.e., they are large), with one service encompassing an entire management platform.

## 2. The Central Question

Is this a microservices architecture? A service-oriented architecture? Something else?

The label is irrelevant. The critical task is to analyze the trade-offs and consequences of the existing design.

## 3. Architectural Analysis Questions

Answer the following questions to dissect the architecture based on its observable characteristics.

1.  **On Synchronous Coupling:** Services make direct REST calls to each other. What happens to the availability and response time of Service A if Service B, which it calls, becomes slow or fails? How does this synchronous coupling impact the goal of service autonomy?

2.  **On Data Strategy:** The system uses two different strategies for inter-service data: asynchronous data duplication via RabbitMQ and a shared Redis cache. 
    - Why do you think both patterns exist? What different needs might they be solving?
    - What are the trade-offs of having duplicated data that is eventually consistent versus a single, centralized, consistent source in Redis?

3.  **On Service Granularity & Cohesion:** You noted one API covers an entire management platform. The "micro" in microservices is less important than cohesion. Is this large service highly cohesive (focused on a single business purpose), or is it a "mini-monolith" that bundles many unrelated functions together? What is the impact of this large service on deployment frequency and team autonomy?

4.  **Putting It Together:** Based on your answers, what are the defining architectural characteristics of this system? Forget the labels. Is it defined by its synchronous coupling? Its complex data management? Its mix of service sizes? What are the primary pain points or benefits your team experiences as a result of this architecture?
