# Exercise 01: Verifying Resilience with a Component Fitness Function

This exercise captures the process of designing and implementing a component-level fitness function to verify the Circuit Breaker resilience pattern.

## 1. The Core Concepts

### What is Resilience?
Resilience is the ability of a system to withstand failure and continue to function. It's not about preventing failures—which is impossible in a distributed system—but about gracefully handling them to avoid catastrophic system-wide outages. A resilient system might respond with degraded functionality, cached data, or default responses, but it does not crash.

### Code implementation
```
services.AddHttpClient("ResilientClient")
      .AddResilienceHandler("standard", builder =>
      {
          builder.AddRetry(new RetryStrategyOptions
          {
              MaxRetryAttempts = 3,
              Delay = TimeSpan.FromSeconds(2),
              BackoffType = DelayBackoffType.Exponential
          });

          builder.AddCircuitBreaker(new CircuitBreakerStrategyOptions
          {
              FailureRatio = 0.5,
              MinimumThroughput = 10,
              SamplingDuration = TimeSpan.FromSeconds(30),
              BreakDuration = TimeSpan.FromSeconds(20)
          });

          builder.AddTimeout(TimeSpan.FromSeconds(5));
      });
```

### Resilience Patterns: Retry vs. Circuit Breaker
You mentioned a full Polly setup with retries. This is a key point.
- **Retry Pattern:** This pattern is for handling *transient* (temporary and self-correcting) faults. For example, a brief network blip or a server that's momentarily busy. You retry the operation a few times with a short delay, hoping it will succeed.
- **Circuit Breaker Pattern:** This pattern is for handling *systemic* or longer-lasting faults. If a downstream service is completely down, retrying dozens of times is harmful. It wastes resources on both the client and server and can lead to cascading failures. The Circuit Breaker detects this total failure, "opens" the circuit, and immediately fails any further calls for a set period, giving the downstream system time to recover.

A robust implementation uses both: a retry policy wrapped inside a circuit breaker.

## 2. The Challenge

**Goal:** Design a high-level, black-box architectural fitness function to test the resilience of a service that depends on a potentially failing downstream API.

**Architectural Characteristic:** Resilience
**Pattern to Verify:** Circuit Breaker

## 3. The Thought Process: From White-Box to Black-Box

### Initial Idea (Developer-Centric, White-Box)
Your initial thinking was to test the gateway and API configurations, perhaps with unit tests. This is a classic developer approach: "I wrote code to configure Polly, so I'll write a test to check that the configuration is correct."

- **Value:** Proves the code was written as intended.
- **Weakness:** It tests the *implementation*, not the *outcome*. It doesn't prove the service is resilient. If another developer bypasses the configured `HttpClient`, the test still passes while the resilience is gone.

### The Architectural Approach (Architect-Centric, Black-Box)
The goal is to verify the *observable behavior* of the service. We treat the service as a black box and test its response to failure, exactly as a real client would experience it. This is what a fitness function does.

## 4. The Fitness Function Implementation

We use `WebApplicationFactory` to test the observable behavior of the service under failure conditions.

### Other Test Cases to Consider
The two tests below are the most complex, but a complete suite would also verify:
- **`Closed -> Open`:** Assert that after N consecutive failures, the N+1 request fails *immediately* with a 503 (or similar) without hitting the downstream service.
- **`Open -> Open`:** Assert that while the circuit is open, all calls continue to fail immediately.
- **`Open -> Half-Open`:** Assert that after the break duration, the *next* call is allowed through to the downstream service (the trial call).

### Test 1: Verify `Half-Open -> Closed` Transition (Happy Path)

This test ensures that after the break duration, a successful trial call will close the circuit.

```csharp
[Fact]
public async Task CircuitBreaker_Should_Transition_ToClosed_FromHalfOpen_OnSuccessfulTrial()
{
    // ARRANGE
    var mockDownstreamService = new Mock<IDownstreamService>();
    var client = _factory.WithWebHostBuilder(builder => {
        builder.ConfigureServices(services => services.AddSingleton(mockDownstreamService.Object));
    }).CreateClient();

    // Phase 1: Open the Circuit
    mockDownstreamService.Setup(s => s.GetData()).ThrowsAsync(new HttpRequestException());
    for (int i = 0; i < 5; i++) { await client.GetAsync("/data"); }
    
    // Phase 2: Wait for Half-Open
    await Task.Delay(TimeSpan.FromSeconds(30));

    // Phase 3: Test the Half-Open Trial Request
    mockDownstreamService.Setup(s => s.GetData()).ReturnsAsync("Successful Data");
    var trialResponse = await client.GetAsync("/data");
    Assert.Equal(HttpStatusCode.OK, trialResponse.StatusCode);

    // Phase 4: Verify the Circuit is Now Closed
    var subsequentResponse = await client.GetAsync("/data");
    Assert.Equal(HttpStatusCode.OK, subsequentResponse.StatusCode);
}
```

### Test 2: Verify `Half-Open -> Open` Transition (Failure Path)

This test ensures that after the break duration, a *failed* trial call will re-open the circuit immediately.

```csharp
[Fact]
public async Task CircuitBreaker_Should_Transition_ToOpen_FromHalfOpen_OnFailedTrial()
{
    // ARRANGE
    var mockDownstreamService = new Mock<IDownstreamService>();
    var client = _factory.WithWebHostBuilder(builder => {
        builder.ConfigureServices(services => services.AddSingleton(mockDownstreamService.Object));
    }).CreateClient();

    // Phase 1: Open the Circuit
    mockDownstreamService.Setup(s => s.GetData()).ThrowsAsync(new HttpRequestException("Initial Failure"));
    for (int i = 0; i < 5; i++) { await client.GetAsync("/data"); }
    
    // Phase 2: Wait for Half-Open
    await Task.Delay(TimeSpan.FromSeconds(30));

    // Phase 3: Test the Half-Open Trial Request (and fail it)
    mockDownstreamService.Setup(s => s.GetData()).ThrowsAsync(new HttpRequestException("Trial Failure"));
    var trialResponse = await client.GetAsync("/data");
    
    // Assert that the service correctly propagated the failure from the trial call.
    Assert.Equal(HttpStatusCode.InternalServerError, trialResponse.StatusCode); // Or whatever status the service returns on downstream failure

    // Phase 4: Verify the Circuit is Immediately Re-Opened
    // The mock is still configured to fail, but it shouldn't even be called.
    // The circuit should be open and fast-fail immediately.
    var subsequentResponse = await client.GetAsync("/data");
    Assert.Equal(HttpStatusCode.ServiceUnavailable, subsequentResponse.StatusCode); // The breaker's "open" response
}
```
