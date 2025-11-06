# Exercise 01: Verifying Resilience with a Component Fitness Function

This exercise captures the process of designing and implementing a component-level fitness function to verify the Circuit Breaker resilience pattern.

## 1. The Challenge

**Goal:** Design a high-level, black-box architectural fitness function to test the resilience of a service that depends on a potentially failing downstream API.

**Architectural Characteristic:** Resilience
**Pattern:** Circuit Breaker

## 2. Initial Implementation (Conceptual)

A service, `MyService`, depends on `IDownstreamService`. To protect the system, a Polly Circuit Breaker policy is applied to the `HttpClient` used to call the downstream service.

```csharp
// In Startup.cs or Program.cs

services.AddHttpClient<IDownstreamService, DownstreamService>()
    .AddPolicyHandler(
        HttpPolicyExtensions
            .HandleTransientHttpError()
            .CircuitBreakerAsync(5, TimeSpan.FromSeconds(30))
    );
```

## 3. Initial Testing Idea (White-box)

The first instinct is to unit test the Polly configuration. This is a white-box test that verifies the implementation, not the architectural outcome.

- **Problem:** This confirms the library is configured, but not that the service *behaves* resiliently. If a developer changes the implementation (e.g., uses a different client), the test might still pass while the architecture breaks.

## 4. Architectural Fitness Function (Black-box)

We use `WebApplicationFactory` to test the observable behavior of the service under failure conditions.

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
