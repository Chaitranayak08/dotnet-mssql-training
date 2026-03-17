# Chapter 9: HttpClient And REST,SSO and TokenAuth

  
  

## Part I: Consuming and Building Resilient REST APIs

  

Modern applications are often built as a collection of services that communicate over HTTP. This section provides a recap of the REST architectural style, how to consume APIs using HttpClient in.NET, and how to build resilience into these communications to handle the transient failures inherent in distributed systems.

  

### Section 1: Recap of REST APIs and HttpClient

  

REpresentational State Transfer (REST) is an architectural style for designing networked applications. It relies on a stateless, client-server communication protocol—almost always HTTP. A Web API that adheres to the principles of REST is known as a RESTful API.

  

#### Core Principles of REST

  

RESTful APIs are designed around resources, which are any kind of object, data, or service that can be accessed by the client. A resource is identified by a URI (Uniform Resource Identifier). Clients interact with resources by sending requests to these URIs using standard HTTP methods:

*   GET: Retrieves a representation of a resource.
    
*   POST: Creates a new resource.
    
*   PUT: Updates an existing resource completely.
    
*   DELETE: Deletes a resource.
    

These APIs are data-centric, typically returning data in a machine-readable format like JSON, rather than the server-rendered HTML of a traditional web page. In ASP.NET Core, RESTful APIs are built using API controllers, which are classes that derive from ControllerBase and are marked with the \[ApiController\] attribute to enable API-specific behaviors.

  

#### Consuming APIs with HttpClient

  

In.NET, the primary tool for consuming REST APIs is the HttpClient class. It provides a simple way to send HTTP requests and receive HTTP responses from a resource identified by a URI.

While you can instantiate HttpClient directly, the recommended practice in ASP.NET Core applications is to use IHttpClientFactory. This factory provides several benefits, including:

*   Centralized Configuration: You can configure all your HttpClient instances in one place (Program.cs).
    
*   Efficient Connection Management: IHttpClientFactory pools HttpClient handlers to avoid the socket exhaustion and DNS issues that can occur when creating many HttpClient instances.
    
*   Middleware Integration: It allows you to plug in middleware for outgoing HTTP requests, which is perfect for implementing logging, authentication, and resilience patterns.
    

You register and configure an HttpClient for your application in Program.cs:

  

C#
```csharp
var builder = WebApplication.CreateBuilder(args);  
  
builder.Services.AddHttpClient("MyApiClient", client =>  
{  
    client.BaseAddress = new Uri("https://api.example.com/");  
});  
  
//...  
```  


You can then inject IHttpClientFactory into your services or PageModel classes to create a client and make API calls.

  

### Section 2: Building Resilient API Clients with Retry and Timeout

  

Network communications are inherently unreliable. Transient faults—such as temporary network glitches, service unavailability, or request timeouts—are expected to occur. A resilient application is designed to anticipate and handle these faults gracefully.

The two most fundamental resilience patterns are Retry and Timeout.

*   Retry Pattern: Automatically retries a failed operation a configured number of times, often with a delay between attempts. This can overcome temporary issues without any user-facing error.
    
*   Timeout Pattern: Ensures that an operation doesn't hang indefinitely by setting a maximum time limit for it to complete.
    

As of.NET 8, ASP.NET Core has integrated these patterns directly into IHttpClientFactory through a standard resilience handler. This handler combines several strategies into a robust pipeline:

1.  Total Request Timeout: An overall timeout for the entire operation, including all retries.
    
2.  Retry Pipeline: A retry policy for transient errors.
    
3.  Circuit Breaker: A pattern that stops requests to a failing service for a period of time to allow it to recover.
    
4.  Attempt Timeout: A timeout for each individual retry attempt.
    

You can add this resilience pipeline to an HttpClient during registration in Program.cs:

  

C#
```csharp
builder.Services.AddHttpClient("MyApiClient", client =>  
{  
    client.BaseAddress = new Uri("https://api.example.com/");  
})  
.AddStandardResilienceHandler(options =>  
{  
    // Configure the retry policy  
    options.Retry.MaxRetryAttempts = 4;  
    options.Retry.Delay = TimeSpan.FromSeconds(1);  
  
    // Configure the timeout for each attempt  
    options.AttemptTimeout.Timeout = TimeSpan.FromSeconds(5);  
});  
```  

This configuration creates a client that will retry a failed request up to 4 times, with a 1-second delay between each attempt. Each attempt is given a 5-second timeout. It's important to note that the default HttpClient.Timeout is 100 seconds. The total time taken by all retries and delays must not exceed this value, or you must configure a longer timeout on the HttpClient itself.

  

## Part II: Different Security Approaches in Action

  

Securing web applications and APIs is a critical requirement. This section explores two of the most common and powerful security models used in modern applications: token-based authentication with JWT for securing APIs, and Single Sign-On (SSO) with OpenID Connect for federated user authentication.

  

### Section 3: Token-Based Authentication with JWT

  

JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. Because it is digitally signed, the information can be verified and trusted.1 JWTs are the most common way to secure RESTful APIs.

A JWT consists of three parts separated by dots (.):

*   Header: Contains metadata about the token, such as the token type (JWT) and the signing algorithm used (e.g., HMAC SHA256).
    
*   Payload: Contains the "claims," which are statements about an entity (typically the user) and additional data. Claims include information like the user's ID, roles, and the token's expiration time.
    
*   Signature: A cryptographic signature created using the encoded header, the encoded payload, and a secret key known only to the server. The signature is used to verify that the token has not been tampered with.
    

  

#### Bearer Token Authentication Flow

  

The most common way to use JWTs is as Bearer Tokens. The flow is as follows:

1.  A user authenticates with a server using their credentials (e.g., username and password).
    
2.  If the credentials are valid, the server generates a signed JWT and sends it back to the client.
    
3.  The client stores this token and includes it in the Authorization header of every subsequent request to a protected API endpoint, using the Bearer scheme: Authorization: Bearer <token>.
    
4.  The API server receives the request, extracts the token, and validates its signature, expiration, issuer, and audience. If the token is valid, the server processes the request.
    

This model is stateless, meaning the server does not need to store any session information. Each request is authenticated solely based on the token it carries, which makes this approach highly scalable and ideal for microservices architectures.

  

#### Implementation in ASP.NET Core

  

1.  Install Packages: Add the Microsoft.AspNetCore.Authentication.JwtBearer NuGet package to your Web API project.
    
2.  Configure appsettings.json: Store your JWT configuration, including the secret key, issuer, and audience.  
    JSON  
    ```JSON
    "Jwt":{  
      "Key": "This is a sample secret key for signing the token",  
      "Issuer": "https://myapi.com",  
      "Audience": "https://myapi.com"  
    }  
    ```     
    
3.  Register Services in Program.cs: Configure the authentication middleware to validate incoming JWTs.  
    C#  
    ```csharp
    builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)  
      .AddJwtBearer(options =>  
        {  
            options.TokenValidationParameters = new TokenValidationParameters  
            {  
                ValidateIssuer = true,  
                ValidateAudience = true,  
                ValidateLifetime = true,  
                ValidateIssuerSigningKey = true,  
                ValidIssuer = builder.Configuration["Jwt:Issuer"],  
                ValidAudience = builder.Configuration["Jwt:Audience"],  
                IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))  
            };  
        }); 

    ```
      
    
4.  Enable Middleware: Add the authentication and authorization middleware to the request pipeline in Program.cs.  
    C#  
    ```csharp
    app.UseAuthentication();  
    app.UseAuthorization();  
    ```  
      
    
5.  Protect Endpoints: Use the \[Authorize\] attribute to secure your API controllers or specific action methods.  
    C#
    ```csharp  
    [ApiController]  
    [Authorize] // This entire controller is now protected

    public class ProductsController : ControllerBase

    {

    [HttpGet]

    public IActionResult Get()

    {

    return Ok(new { "Product A", "Product B" });

    }

    }

    ```

### Section 4: Single Sign-On (SSO) with OpenID Connect

  

Single Sign-On (SSO) is an authentication scheme that allows a user to log in with a single set of credentials to any of several related, yet independent, software systems.9 This improves user experience and centralizes identity management.

OpenID Connect (OIDC) is the leading standard for implementing SSO. It is an identity layer built on top of the OAuth 2.0 authorization framework. While OAuth 2.0 is designed for authorization (granting access to resources), OIDC is designed for authentication (verifying a user's identity).

  

#### OIDC Authentication Flow (Authorization Code Flow)

  

1.  A user attempts to access a protected resource in your web application (the Client).
    
2.  The application redirects the user to an external Identity Provider (IdP), such as Azure AD, Google, or Auth0.
    
3.  The user authenticates with the IdP using their credentials.
    
4.  After successful authentication, the IdP redirects the user back to the client application with a temporary authorization code.
    
5.  The client application's backend securely exchanges this authorization code with the IdP for an ID Token and an Access Token.
    
6.  The client validates the ID Token (which is a JWT) to confirm the user's identity and extracts claims about the user.
    
7.  The client then typically creates a local session for the user, usually by issuing an encrypted cookie, so the user doesn't have to re-authenticate on subsequent requests.
    

  

#### Implementation in ASP.NET Core

  

1.  **Install Packages:** Add the Microsoft.AspNetCore.Authentication.OpenIdConnect and Microsoft.AspNetCore.Authentication.Cookies NuGet packages.
    
2.  **Configure appsettings.json: Store your OIDC provider's details.**  
    JSON  
    ```json
    "Oidc": {  
      "Authority": "https://login.microsoftonline.com/{tenant-id}",  
      "ClientId": "your-client-id",  
      "ClientSecret": "your-client-secret"  
    }  
    ``` 
      
    
3.  **Register Services in Program.cs**: Configure both Cookie and OIDC authentication handlers.  
    C#  
    ```csharp
    builder.Services.AddAuthentication(options =>  
    {  
        options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;  
        options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;  
    })  
      
    

    .AddCookie()

    .AddOpenIdConnect(options =>

    {

    options.Authority = builder.Configuration\["Oidc:Authority"\];

    options.ClientId = builder.Configuration\["Oidc:ClientId"\];

    options.ClientSecret = builder.Configuration;

    options.ResponseType = "code";

    options.SaveTokens = true;

    });

    ```
    4.**Enable Middleware**: Add the authentication and authorization middleware to the request pipeline.csharp
    ```csharp
    app.UseAuthentication();

    app.UseAuthorization();
    ```
    
5. **Protect Pages**: Use the \[Authorize\] attribute on your Razor Pages' PageModel classes or on specific folders to trigger the authentication flow. When an unauthenticated user tries to access a protected page, the middleware will automatically redirect them to the Identity Provider to log in.