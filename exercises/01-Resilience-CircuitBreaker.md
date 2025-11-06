# Exercise 01: Verifying Resilience with a Component Fitness Function

This exercise captures the process of designing and implementing a component-level fitness function to verify the Circuit Breaker resilience pattern.

## 1. Core Concepts

### What is Resilience?
Resilience is the ability of a system to withstand failure and continue to function. It is not about preventing failures—which is impossible in a distributed system—but about gracefully handling them to avoid catastrophic system-wide outages. A resilient system might respond with degraded functionality, cached data, or default responses, but it does not crash.

### Resilience Patterns & Implementation

Two common patterns for resilience are Retry and Circuit Breaker.

- **Retry Pattern:** This pattern is for handling *transient* (temporary and self-correcting) faults, like a brief network blip. The operation is retried a few times, hoping it will succeed.
- **Circuit Breaker Pattern:** This pattern is for handling *systemic* or longer-lasting faults. If a downstream service is down, retrying is wasteful. The Circuit Breaker detects this, "opens" the circuit, and immediately fails subsequent calls for a set period, giving the downstream system time to recover.

A robust implementation uses both, often by wrapping a retry policy inside a circuit breaker. The following is a sample implementation using Polly's `AddResilienceHandler`:

```csharp
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

## 2. The Verification Challenge

**Goal:** Design a high-level, black-box architectural fitness function to test the resilience of a service that depends on a potentially failing downstream API.

**Architectural Characteristic:** Resilience
**Pattern to Verify:** Circuit Breaker

## 3. Evolution of Testing Approach

### White-Box Testing (Implementation-focused)
A common initial approach is to unit test the resilience policy configuration. This is a white-box test that verifies the implementation, not the architectural outcome.

- **Value:** Proves the code was written as intended.
- **Weakness:** It tests the *implementation*, not the *outcome*. It doesn't prove the service behaves resiliently. If a developer later bypasses the configured `HttpClient`, the test might still pass while the architectural guarantee of resilience is broken.

### Black-Box Testing (Behavior-focused)
The architectural approach is to verify the *observable behavior* of the service. The service is treated as a black box, and its response to failure is tested exactly as a real client would experience it. This is the role of a fitness function.

## 4. The Fitness Function Solution

`WebApplicationFactory` is used to host the service in-memory and test its observable behavior via HTTP requests.

### Comprehensive Test Cases
A complete test suite for a circuit breaker should verify all state transitions:
- **`Closed -> Open`:** Assert that after N consecutive failures, the N+1 request fails *immediately* without hitting the downstream service.
- **`Open -> Open`:** Assert that while the circuit is open, all calls continue to fail immediately.
- **`Open -> Half-Open`:** Assert that after the break duration, the *next* call is allowed through to the downstream service (the trial call).
- **`Half-Open -> Closed`:** The "happy path" test; ensures a successful trial call closes the circuit.
- **`Half-Open -> Open`:** The "unhappy path" test; ensures a failed trial call re-opens the circuit.

Below are the implementations for the two most complex transitions.

### Test 1: Verify `Half-Open -> Closed` Transition (Happy Path)

This test ensures that after the break duration, a successful trial call will close the circuit.

```csharp
[Fact]
public async Task CircuitBreaker_Should_Transition_ToClosed_FromHalfOpen_OnSuccessfulTrial()
{
    // ARRANGE
    var mockDownstreamService = new Mock<IDownstreamService>();
    var client = _factory.WithWebHostBuilder(builder =>
    {
        builder.ConfigureServices(services =>
        {
            // Replace the real dependency with our mock for this test run
            services.AddSingleton(mockDownstreamService.Object);
        });
    }).CreateClient();

    // --- Phase 1: Force the Circuit to Open ---
    // Configure the dependency to fail consistently.
    mockDownstreamService.Setup(s => s.GetData()).ThrowsAsync(new HttpRequestException());

    // Send enough failing requests to trip the breaker.
    for (int i = 0; i < 5; i++) // Assuming breaker opens after 5 failures
    {
        await client.GetAsync("/data");
    }
    
    // Verify it's open by checking for the fast-fail response.
    var responseWhenOpen = await client.GetAsync("/data");
    Assert.Equal(HttpStatusCode.ServiceUnavailable, responseWhenOpen.StatusCode); // Or whatever the policy dictates

    // --- Phase 2: Wait for the Half-Open State ---
    // Wait for the configured "BreakDuration" to elapse.
    await Task.Delay(TimeSpan.FromSeconds(20)); // Assuming a 20s break duration from the Polly setup

    // --- Phase 3: Test the Half-Open Trial Request ---
    // Configure the dependency to succeed for the trial call.
    mockDownstreamService.Setup(s => s.GetData()).ReturnsAsync("Successful Data");

    // Send the single trial request.
    var trialResponse = await client.GetAsync("/data");

    // Assert the trial call was successful.
    Assert.Equal(HttpStatusCode.OK, trialResponse.StatusCode);

    // --- Phase 4: Verify the Circuit is Now Closed ---
    // The mock is still configured to succeed. Send another request.
    var subsequentResponse = await client.GetAsync("/data");
    Assert.Equal(HttpStatusCode.OK, subsequentResponse.StatusCode);

    // Verify the downstream service was called twice after the break,
    // proving the circuit closed and is not in the half-open state anymore.
    mockDownstreamService.Verify(s => s.GetData(), Times.Exactly(2));
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
    var client = _factory.WithWebHostBuilder(builder =>
    {
        builder.ConfigureServices(services =>
        {
            // Replace the real dependency with our mock for this test run
            services.AddSingleton(mockDownstreamService.Object);
        });
    }).CreateClient();

    // --- Phase 1: Force the Circuit to Open ---
    // Configure the dependency to fail consistently.
    mockDownstreamService.Setup(s => s.GetData()).ThrowsAsync(new HttpRequestException("Initial Failure"));

    // Send enough failing requests to trip the breaker.
    for (int i = 0; i < 5; i++) // Assuming breaker opens after 5 failures
    {
        await client.GetAsync("/data");
    }
    
    // Verify it's open by checking for the fast-fail response.
    var responseWhenOpen = await client.GetAsync("/data");
    Assert.Equal(HttpStatusCode.ServiceUnavailable, responseWhenOpen.StatusCode); // Or whatever the policy dictates

    // --- Phase 2: Wait for the Half-Open State ---
    // Wait for the configured "BreakDuration" to elapse.
    await Task.Delay(TimeSpan.FromSeconds(20)); // Assuming a 20s break duration from the Polly setup

    // --- Phase 3: Test the Half-Open Trial Request (and fail it) ---
    // Configure the dependency to fail for the trial call.
    mockDownstreamService.Setup(s => s.GetData()).ThrowsAsync(new HttpRequestException("Trial Failure"));

    // Send the single trial request.
    var trialResponse = await client.GetAsync("/data");
    
    // Assert that the service correctly propagated the failure from the trial call.
    Assert.Equal(HttpStatusCode.InternalServerError, trialResponse.StatusCode); // Or whatever status the service returns on downstream failure

    // --- Phase 4: Verify the Circuit is Immediately Re-Opened ---
    // The mock is still configured to fail, but it shouldn't even be called.
    // The circuit should be open and fast-fail immediately.
    var subsequentResponse = await client.GetAsync("/data");
    Assert.Equal(HttpStatusCode.ServiceUnavailable, subsequentResponse.StatusCode); // The breaker's "open" response

    // Verify the downstream service was called only once after the break (the trial call),
    // proving the circuit re-opened and subsequent calls were fast-failed.
    mockDownstreamService.Verify(s => s.GetData(), Times.Once());
}
```
