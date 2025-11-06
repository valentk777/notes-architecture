# Architectural Fitness Function: Component-Level

**Definition:** A component-level fitness function is an automated test that verifies an architectural characteristic (e.g., resilience, security, scalability) for a single service or component, treating it as a black box.

**Context:** In ASP.NET Core, the primary tool for this is `WebApplicationFactory<T>`.

**Why `WebApplicationFactory`?**
- It hosts the application in-memory, including the full request pipeline, configuration, and dependency injection.
- It allows testing via a real `HttpClient`, sending actual HTTP requests to the service.
- This tests the *observable external behavior* of the service, not its internal implementation details. It is a true black-box test of the component's architecture.

## Example: Testing a Circuit Breaker's Half-Open State

This fitness function verifies that a circuit breaker correctly transitions from Half-Open to Closed after a successful trial request.

```csharp
public class ResilienceFitnessFunction : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;

    public ResilienceFitnessFunction(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
    }

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
        // Wait for the configured "DurationOfBreak" to elapse.
        await Task.Delay(TimeSpan.FromSeconds(30)); // Assuming a 30s break duration

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
}
```
