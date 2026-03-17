# Chapter 8: Retry & Timeout,Logging & Monitoring Background Services 

#   
  

## Part I: Demonstrating Logging and Monitoring

#   

Modern applications, especially those in distributed environments, must be resilient to transient failures and observable to allow for effective diagnostics. This part covers the essential practices of implementing resilience patterns, structured logging, and health monitoring in ASP.NET Core.

  

### Section 1: Implementing Resilience with Retry and Timeout Policies

#   

When your application communicates with external services over a network (such as calling a database or a remote API), it will inevitably encounter transient faults—temporary errors that resolve themselves quickly. Building a resilient application means anticipating these faults and handling them gracefully, rather than allowing them to cause a catastrophic failure.

The recommended approach in ASP.NET Core is to use IHttpClientFactory in combination with resilience libraries like Polly. As of.NET 8, these capabilities are integrated directly into the framework, making them easier to implement.

  

#### The Standard Resilience Handler

#   

ASP.NET Core provides a standard resilience handler that combines several key strategies into a single, easy-to-configure pipeline :

1.  Retry: Automatically retries a failed request a set number of times. This is effective for transient network errors.
    
2.  Attempt Timeout: Applies a timeout to each individual request attempt, preventing a single call from hanging indefinitely.
    
3.  Circuit Breaker: Monitors for repeated failures. If the failure rate exceeds a certain threshold, the circuit "opens," and subsequent requests fail immediately without even attempting to contact the failing service. This prevents the application from overwhelming a struggling dependency.
    
4.  Total Request Timeout: Applies an overall timeout to the entire operation, including all retry attempts, ensuring the process doesn't run longer than a specified limit.
    

  

#### Configuration Example

#   

You can configure these policies when you register an HttpClient in your Program.cs file. This is typically done using the AddStandardResilienceHandler extension method.

  

C#
```csharp
// In Program.cs  
var builder = WebApplication.CreateBuilder(args);  
  
builder.Services.AddHttpClient<MyApiService>(client =>  
{  
    client.BaseAddress = new Uri("https://api.example.com/");  
})  
.AddStandardResilienceHandler(options =>  
{  
    // Configure the retry policy  
    options.Retry.MaxRetryAttempts = 5;  
    options.Retry.Delay = TimeSpan.FromSeconds(2);  
  
    // Configure the total request timeout  
    options.TotalRequestTimeout.Timeout = TimeSpan.FromSeconds(60);  
  
    // Configure the timeout for each individual attempt  
    options.AttemptTimeout.Timeout = TimeSpan.FromSeconds(10);  
});  
  
builder.Services.AddRazorPages();  
var app = builder.Build();  
//...  
```  

In this example, any HttpClient injected for MyApiService will automatically be resilient. If a request fails, it will be retried up to 5 times with a 2-second delay between attempts. Each attempt is limited to 10 seconds, and the entire operation (including all retries) cannot exceed 60 seconds.

  

### Section 2: Logging for Diagnostics and Insight

#   

Effective logging is the cornerstone of an observable application. It provides the necessary insight to diagnose problems, understand application behavior, and monitor performance. ASP.NET Core has a flexible, built-in logging framework centered around the ILogger interface.

  

#### Using ILogger

#   

The recommended way to create logs is to request an ILogger<T> instance from the dependency injection (DI) container, where T is the class that will be writing the logs. The framework automatically associates the logger with a category, which by default is the fully qualified name of the class (MyWebApp.Pages.IndexModel, for example).

  

C#
```csharp
// In a PageModel class  
public class IndexModel : PageModel  
{  
    private readonly ILogger<IndexModel> logger;  
  
    public IndexModel(ILogger<IndexModel> logger)  
    {  
        logger = logger;  
    }  
  
    public void OnGet()  
    {  
        logger.LogInformation("Index page visited at {Time}", DateTime.UtcNow);  
    }  
  
    public async Task<IActionResult> OnPostAsync()  
    {  
        try  
        {  
            //... perform some operation...  
        }  
        catch (Exception ex)  
        {  
            logger.LogError(ex, "An error occurred while processing the form post.");  
            // Handle the error  
        }  
        return Page();  
    }  
}  
```  


#### Log Levels and Configuration

#   

The logging framework supports several log levels to indicate the severity of an event. When you configure a minimum log level, messages at that level and higher are recorded. The levels, in order of increasing severity, are:

*   Trace
    
*   Debug
    
*   Information
    
*   Warning
    
*   Error
    
*   Critical
    

You can configure the minimum log level for different categories and providers in your appsettings.json file.

JSON
```JSON  

{  
  "Logging": {  
    "LogLevel": {  
      "Default": "Information",  
      "Microsoft.AspNetCore": "Warning"  
    }  
  }  
}  
 ``` 

This configuration sets the default minimum log level for all categories to Information. However, for any category starting with Microsoft.AspNetCore, it raises the minimum level to Warning, filtering out the more verbose informational logs from the framework itself.

  

### Section 3: Monitoring Application Health with Health Checks

#   

Health checks provide a simple way for an external monitoring service (like a container orchestrator or load balancer) to determine the status of your application. ASP.NET Core provides a built-in framework for creating health check endpoints.A health check endpoint returns one of three statuses:

*   Healthy: The application and its critical dependencies are working correctly.
    
*   Degraded: The application is running but is in a degraded state (e.g., a non-critical dependency is slow or unavailable).
    
*   Unhealthy: The application is not functioning correctly and should be taken out of service.
    

  

#### Implementing Health Checks

#   

1.  Register and Map Health Checks:  
    In Program.cs, you need to register the health check services and map an endpoint for them.  
    C# 
    ```csharp 
    // In Program.cs  
    var builder = WebApplication.CreateBuilder(args);  
      
    // Register the health check services  
    builder.Services.AddHealthChecks();  
      
    var app = builder.Build();  
      
    // Map the endpoint  
    app.MapHealthChecks("/health");  
      
    app.Run();  
     ``` 
      
    
2.  Create a Custom Health Check:  
    To check a specific dependency, you create a class that implements the IHealthCheck interface. This interface has a single method, CheckHealthAsync, which contains the logic to verify the health of a component. 

    C# 
    ```csharp 
    // Example of a custom health check for an external API  
    public class ExternalApiHealthCheck : IHealthCheck  
    {  
        private readonly IHttpClientFactory httpClientFactory;  
      
        public ExternalApiHealthCheck(IHttpClientFactory httpClientFactory)  
        {  
            httpClientFactory = httpClientFactory;  
        }  
      
        public async Task<HealthCheckResult> CheckHealthAsync(  
            HealthCheckContext context,  
            CancellationToken cancellationToken = default)  
        {  
            using (var client = httpClientFactory.CreateClient())  
            {  
                try  
                {  
                    var response = await client.GetAsync("https://api.example.com/ping", cancellationToken);  
                    if (response.IsSuccessStatusCode)  
                    {  
                        return HealthCheckResult.Healthy("External API is available.");  
                    }  
                    return HealthCheckResult.Unhealthy("External API is unavailable.");  
                }  
                catch (Exception ex)  
                {  
                    return HealthCheckResult.Unhealthy("Failed to check external API.", ex);  
                }  
            }  
        }  
    }  
    ```    
      
    
    3. Register the Custom Health Check:  
    Finally, add your custom check to the health check services in Program.cs.  
    C#  
    ```csharp
    // In Program.cs  
    builder.Services.AddHealthChecks()  
      .AddCheck<ExternalApiHealthCheck>("external\_api\_check");  
      
     ``` 
    

Now, when a request is made to the /health endpoint, your custom health check will be executed, and its status will be included in the overall health report of the application.

  

## Part II: Apply Background Services

#   

Many applications need to perform tasks that run independently of the user-facing request-response cycle. These include sending email notifications, processing items from a message queue, or performing periodic data cleanup. In ASP.NET Core, these long-running tasks are implemented as background services.

  

### Section 1: IHostedService and the BackgroundService Class

#   

The core of background tasks in ASP.NET Core is the IHostedService interface. It defines two methods:

*   StartAsync(CancellationToken): Called when the application host starts. This is where you begin your background work.
    
*   StopAsync(CancellationToken): Called when the application host is performing a graceful shutdown. This is your opportunity to clean up resources and stop your work gracefully.
    

While you can implement IHostedService directly, the framework provides a more convenient abstract base class called BackgroundService. This class handles the boilerplate logic and provides a single abstract method for you to implement :

*   ExecuteAsync(CancellationToken stoppingToken): This method is called by StartAsync. You place your long-running logic inside this method, typically within a loop that continues as long as a cancellation has not been requested via the stoppingToken.
    

  

### Section 2: Implementing a Timed Background Service

#   

The following example shows a simple background service that logs a message every 10 seconds.

  

C#
```csharp
public class TimedHostedService : BackgroundService  
{  
    private readonly ILogger<TimedHostedService> logger;  
  
    public TimedHostedService(ILogger<TimedHostedService> logger)  
    {  
        logger = logger;  
    }  
  
   protected override async Task ExecuteAsync(CancellationToken stoppingToken)  
    {  
        logger.LogInformation("Timed Hosted Service running.");  
  
        while (!stoppingToken.IsCancellationRequested)  
        {  
            logger.LogInformation("Timed Hosted Service is working at: {time}", DateTimeOffset.Now);  
             
            // Wait for 10 seconds, or until a cancellation is requested  
            await Task.Delay(10000, stoppingToken);  
        }  
    }  
  
    public override async Task StopAsync(CancellationToken stoppingToken)  
    {  
        logger.LogInformation("Timed Hosted Service is stopping.");  
        await base.StopAsync(stoppingToken);  
    }  
}  
```  

  

In this implementation:

*   We inject ILogger to write log messages.
    
*   The ExecuteAsync method contains a while loop that runs as long as the application is not shutting down (!stoppingToken.IsCancellationRequested).
    
*   Task.Delay is used to pause execution without blocking a thread. Crucially, it respects the stoppingToken, so if a shutdown is initiated, the delay will be canceled immediately, allowing the service to stop promptly.
    

  

### Section 3: Registering the Background Service

#   

To enable your background service, you must register it with the dependency injection container in Program.cs using the AddHostedService extension method.

  

C#
```csharp  

// In Program.cs  
var builder = WebApplication.CreateBuilder(args);  
  
// Register other services...  
builder.Services.AddRazorPages();  
  
// Register the background service  
builder.Services.AddHostedService<TimedHostedService>();  
  
var app = builder.Build();  
//...  
```  

Once registered, the ASP.NET Core host will automatically manage the lifetime of your service. It will call StartAsync when the application starts and StopAsync when it shuts down, ensuring your background tasks are a fully integrated and managed part of your application's lifecycle.