# Chapter 12 :  Applying Unit Testing in.NET 

## I. The Foundation of Testable Software Architecture

The successful implementation of unit testing in a modern.NET environment is not merely an exercise in debugging; it is an architectural mandate directly contingent upon sound software design principles. Rigorous unit testability demands that components are developed with isolation, modularity, and explicit contracts, largely enabled by adherence to the Object-Oriented Programming (OOP) paradigm and Dependency Injection (DI).

### 1.1 Defining the Unit Test and the Principle of Isolation
#
A unit test is a precisely defined case crafted to validate the specific behavior of a given code unit. A "unit" typically refers to the smallest component of software that can be logically tested, generally an individual class or method. The primary objective of such a test is to verify that, given a set of known input values, the function or method produces the correct, predictable output.  

#### The Isolation Imperative

The core tenet distinguishing unit testing from broader testing strategies (such as integration or end-to-end testing) is the imperative of isolation. The unit under test (SUT) must be rigorously separated from all external dependencies. Dependencies include infrastructure components like databases, file systems, network services, external APIs, and even other complex, stateful services within the application.  

Isolation ensures that when a test fails, the fault lies exclusively within the logic of the SUT itself, eliminating ambiguity. Dependencies must be replaced by controlled substitute objects, often referred to as mocks or stubs, that simulate the required external behavior without engaging the actual infrastructure. This capability to substitute dependencies is crucial for creating a test suite that is fast, reliable, and decoupled from the operational environment.  

#### The Arrange-Act-Assert (AAA) Standard

All high-quality unit tests must adhere to a standardized structure known as the Arrange-Act-Assert (AAA) pattern. This pattern is a functional contract that ensures the test code is immediately readable, maintainable, and verifiable, defining the clear intent behind every test method.  

1.  **Arrange:** This phase involves setting up the environment required for the test execution. This includes initializing the SUT, defining input values, and most critically, configuring the specific behavior of any dependency substitute (mock objects) that the SUT will rely upon during execution.  
    
2.  **Act:** This phase consists solely of executing the specific unit of code being tested. Typically, this is a single method call on the SUT, using the prepared inputs from the Arrange phase.  
    
3.  **Assert:** This final phase compares the actual outcome generated during the Act phase against the expected result, determining whether the test should be reported as successful or failed. Verification checks can include examining return values, checking the SUT’s internal state change, or confirming that the SUT correctly interacted with its mocked dependencies.  
    

### 1.2 Decoupling for Testability: The Role of OOP Pillars

The four pillars of OOP—Encapsulation, Abstraction, Inheritance, and Polymorphism—are not merely theoretical concepts but serve as the foundational principles that enable decoupled and testable architecture.  

#### Encapsulation and the SUT

Encapsulation mandates bundling an object's data and the methods that operate on that data into a single, cohesive unit—the class. In C#, this is enforced by access modifiers, concealing the internal state (private fields) and controlling external interaction through a public interface (public properties and methods).  

This principle is directly beneficial for unit testing because it defines the exact boundaries of the SUT. For instance, the Razor Pages `PageModel` class, which bundles page data (properties) and behavior (handler methods), becomes a perfectly demarcated SUT. Since the internal state is protected, tests are forced to target the public behaviors, making the test focus clear: validating the public contract of the SUT.  

#### Abstraction and Interfaces

Abstraction simplifies complexity by hiding intricate implementation details ("how") and exposing only the essential, high-level functionalities ("what"). In.NET, abstraction is effectively realized through interfaces or abstract base classes.  

Programming against an interface (e.g., `ICustomerService` instead of the concrete `CustomerService` class) is pivotal for testing. This architectural choice decouples the SUT from the specific, complex implementation, ensuring that the SUT depends only on a contract. This complete decoupling allows the test project to seamlessly swap the real, production-ready implementation (which might involve database calls or network traffic) with a controlled, test-specific mock implementation. The interface thus serves as the definition of the external behavior that the mocking framework (like Moq) must simulate.  

#### Inheritance and Shared Testable Logic

Inheritance, which models an "is-a" relationship, allows code reuse by having a derived class acquire the public and protected members of a base class. In large Razor Pages projects, this enables developers to create reusable base classes (e.g., a  
`BasePageModel`) containing shared logic like authorization checks or common data retrieval mechanisms.  

However, the use of inheritance in components intended for testing introduces a crucial constraint: any base class must adhere strictly to the Explicit Dependency Principle. If a component is testable, all its dependencies—including those required by the base class—must be provided via constructor injection. This prevents the derived classes from accidentally inheriting untestable, hard-coded dependencies, maintaining testability across the entire class hierarchy.  

### 1.3 Dependency Inversion and Injection: Enabling Mocking

Dependency Injection (DI) is the practical realization of the Dependency Inversion Principle, which mandates that high-level modules should not depend on low-level modules; both should depend on abstractions (interfaces).

#### The Causal Chain of DI

ASP.NET Core utilizes a sophisticated, built-in DI container, configured during application startup in `Program.cs`. DI acts as the enabling mechanism that allows a production application to manage the lifecycles of concrete service implementations. More significantly for testing, DI allows those concrete implementations, registered at runtime, to be swapped for controlled mock objects at test time.  

Without DI and Abstraction, a service that directly instantiates its dependency (e.g., `var writer = new MessageWriter();`) creates a hard coupling. To test this service, one would have to modify the production code to inject a fake  
`MessageWriter`, which violates the integrity of the unit test and breaks isolation.

By contrast, using constructor injection—where dependencies are required as interface parameters in the SUT's constructor—the SUT becomes completely agnostic to the concrete type of the dependency. This allows the test harness to inject a specific, controlled mock object into the constructor during the Arrange phase. This flexibility is the critical bridge: it is the utilization of OOP abstraction that enables the swapping necessary for isolation via DI, ensuring the code is inherently testable.  

## II. The.NET Testing Landscape: Frameworks and Tooling

Successful unit testing relies not only on good architectural design but also on the selection of robust, effective tooling. In the modern.NET ecosystem, the standard test framework pairing is xUnit.net for execution and Moq for dependency substitution.

### 2.1 Selecting the Right Test Framework

While.NET supports several established frameworks—xUnit, NUnit, and MSTest—the selection often dictates the flexibility, performance, and structure of the resulting test suite.  

#### xUnit.net (The Recommended Standard)

xUnit.net is widely favored for contemporary.NET development due to its clean design and performance characteristics. It utilizes specific attributes for test identification: `[Fact]` is used for basic, singular tests, and `is used in conjunction with` or other data sources for parameterized, data-driven tests.  

A key architectural advantage of xUnit is that it runs tests in parallel by default, providing superior execution speed for large suites. Furthermore, xUnit maintains a minimalist approach to built-in assertions, directing developers toward external, richer assertion libraries, such as FluentAssertions. This design preference reinforces the principle that the test framework should primarily manage execution flow, delegating detailed failure description and result verification to specialized, dedicated tools.  

#### Comparison of Leading.NET Unit Testing Frameworks

The following table provides a concise comparison of the primary features of the available.NET testing frameworks, illustrating why xUnit is often selected for projects prioritizing speed and modern conventions.

Comparison of Leading.NET Unit Testing Frameworks

| Feature | xUnit.net | NUnit | MSTest |
| --- | --- | --- | --- |
| Test Attribute | [Fact] (Simple), `` (Parameterized) | (Simple), (Parameterized) | `` |
| Parallel Execution | Default and built-in | Supported, configuration required | Limited support (improved recently) |
| Assertion Library | Minimal built-in, encourages external (e.g., FluentAssertions) | Rich set of built-in assertions | Standard built-in assertions |
| Data-Driven Testing | and | `` | `` |



### 2.2 The Necessity of Mocking: Simulating Dependencies with Moq

Unit tests require isolation, but components rarely operate without dependencies. Mocking provides the necessary solution by creating controlled, artificial implementations of interfaces or abstract classes that the SUT depends upon.  

#### Moq as the Industry Standard

Moq is the most popular and widely adopted mocking framework in the.NET community. It offers intuitive mechanisms for simulating complex dependency behaviors, ensuring that tests focus strictly on the core functionality of the SUT without relying on the real implementation.  

Moq allows developers to perform two fundamental actions essential for rigorous testing:

1.  **Setup Behavior:** A test can define exactly what the mock object should return when a specific method is called with specific arguments. This is achieved using the  
    
    `mock.Setup(...)` syntax. This ensures the SUT receives predictable data, allowing test results to be deterministic.  
    
2.  **Verify Interaction:** When the SUT’s function is to coordinate or delegate work (e.g., calling a repository to save data), the test must confirm that the dependent method was called correctly. Moq facilitates this through the `mock.Verify(...)` method, allowing developers to check _if_, _how many times_, and _with what arguments_ a dependency method was invoked.  
    

The ability to both control the input _to_ the SUT (via setup) and observe the interaction _from_ the SUT (via verification) ensures that testing is comprehensive, covering both data transformation and coordination behavior.

### 2.3 Setting Up the Testing Project Structure

Structural discipline is mandatory for maintaining a scalable and efficient test suite. The physical separation of test code from production code reinforces the architectural mandate of isolation.

#### Project Naming Convention and Separation

Unit tests should always reside in separate projects specifically created from a test framework template. This test project should adhere to a strict naming convention, typically following the pattern:  

`[ProductionProjectName].UnitTests`. For example, a business logic layer project named `MyProject.Services` should correspond to a test project named `MyProject.Services.UnitTests`.  

Furthermore, unit tests and integration tests must be organized into separate, distinct test projects. Unit tests must possess _no_ external dependencies on infrastructure layers (such as database projects or external file systems). This allows unit tests to run rapidly and reliably as part of continuous integration pipelines. Integration tests, which intentionally involve supporting infrastructure (e.g., databases or networks) to test component interaction, are fundamentally different and must be run later in the DevOps pipeline. This structural separation is vital for maintaining the speed and reliability of the developer feedback loop.  

## III. Unit Testing Core C# Business Logic

The foundation of any testable.NET application is a well-structured service layer composed of pure C# classes. These classes embody the core business logic and are decoupled from the ASP.NET Core framework, making them ideal subjects for unit testing.

### 3.1 Testing Pure Functions and Logic Classes

#### The Simple Case (Stateless Logic)

Pure functions or stateless classes—methods that only rely on input parameters and return a predictable output without causing external side effects—represent the easiest units to test. In this scenario, the AAA pattern focuses primarily on state verification.  

For example, testing a simple `CalculatorService` method requires minimal setup:

C#
```csharp
    // CalculatorServiceTest.cs
    public class CalculatorControllerTest
    {
        private readonly CalculatorService _unitTesting = new CalculatorService();
        
        [Fact]
        public void Add_ValidInputs_ReturnsCorrectSum()
        {
            // Arrange
            double a = 5;
            double b = 3;
            double expected = 8;
            
            // Act
            var actual = _unitTesting.Add(a, b);
            
            // Assert
            Assert.Equal(expected, actual, 0); 
        }
    }
```
The test is encapsulated within the `[Fact]` attribute and clearly demonstrates the three logical parts of the AAA standard.

#### Data-Driven Testing

To ensure robustness against multiple scenarios and edge cases, tests should utilize data-driven methods, avoiding code duplication. xUnit's `attribute, combined with`, allows the same test logic to be executed efficiently across various input combinations.  

C#
```csharp
    public class MathUtilsTests
    {
    
        public void Multiply_VariousInputs_ReturnMultipliedResult(int value1, int value2, int expectedResult)
        {
            // Act
            int actualResult = MathUtils.Multiply(value1, value2);
            
            // Assert
            Assert.Equal(expectedResult, actualResult);
        }
    }
```
### 3.2 Implementing Moq for Service Layer Isolation (Deep Dive)

When testing a Service Under Test (SUT) that relies on external interfaces—which are typically resolved via Dependency Injection—Moq is indispensable for providing controlled, faked behavior.  

#### Setting up Return Values

Consider testing an `OrderProcessor` that depends on `IInventoryService`. The test must ensure the `OrderProcessor` behaves correctly whether inventory is available or not, without actually querying a live stock system.

C#
```csharp
    // Arrange
    var mockInventory = new Mock<IInventoryService>();
    
    // Configure the mock to return 'true' when CheckStock is called, 
    // regardless of the product ID passed (It.IsAny<int>()).
    mockInventory.Setup(i => i.CheckStock(It.IsAny<int>()))
                .Returns(true);
    
    var orderProcessor = new OrderProcessor(mockInventory.Object);
    
    // Act
    bool result = orderProcessor.PlaceOrder(123, 1);
    
    // Assert...
```
This `Setup` phase ensures that the SUT receives predictable data, stabilizing the testing environment.

#### Testing Asynchronous Code

Given that most modern.NET service layers use asynchronous operations (e.g., `async Task`), the unit tests must accommodate this. This requires defining the mock setup to return an asynchronous task, often achieved using `Task.FromResult(T)`, and using asynchronous assertion methods from the framework, such as `Assert.ThrowsAsync<T>()` for exceptions.  

### 3.3 Verification of State vs. Verification of Behavior

Unit testing strategies fundamentally divide into two related areas based on the SUT's purpose: testing state and testing behavior.

#### Testing State

State testing focuses on checking the direct output or observable state change of the SUT. For example, verifying that a `CalculateTax` method returns $5.00 is a state verification.

#### Testing Behavior (Crucial for Coordination)

When a method's primary responsibility is to orchestrate calls between various services (acting as a "coordinator"), behavior verification becomes critical. The test must confirm that the SUT correctly initiated the necessary actions on its dependencies. This is often the case when testing logic that involves side effects, such as saving data, logging, or sending notifications.  

For instance, if an `OrderProcessor.PlaceOrder` method is expected to call `IInventoryService.DecreaseStock` and then `IEmailService.SendConfirmation`, the test must confirm that both dependent methods were called.

C#
```csharp
    // Arrange
    var mockEmailService = new Mock<IEmailService>();
    //... setup for successful placement...
    
    // Act
    orderProcessor.PlaceOrder(101);
    
    // Assert: Verify that the notification was sent exactly once
    mockEmailService.Verify(e => e.SendConfirmation(It.IsAny<string>()), Times.Once); 
```
This type of verification is essential for ensuring that the coordinator class correctly delegates work and adheres to the business process flow. The effectiveness of the mocking framework, in this context, lies equally in its ability to observe the SUT's output (calls made to dependencies) as in its ability to control the input to the SUT (setting up returns).

The complexity of setting up and retrieving mocked dependencies, particularly when a class has multiple dependencies, often necessitates the creation of utility classes, such as a generic `TestingObject<T>` helper. These utilities are designed to streamline the dependency injection process within the Arrange phase, abstracting away the boilerplate code required to manage mocks, thus increasing test developer efficiency and readability.  

## IV. Advanced Unit Testing in ASP.NET Core Razor Pages

While core business logic resides in service layers, the Razor Pages `PageModel` contains the critical logic that handles HTTP requests, processes form submissions, manages validation state, and coordinates the service layer. Unit testing these PageModels presents unique challenges due to their reliance on the ASP.NET Core execution pipeline and web context.  

### 4.1 The PageModel as the Unit Under Test (SUT)

The `PageModel` class, which inherits from `Microsoft.AspNetCore.Mvc.RazorPages.PageModel`, represents the logical unit for a specific web page. Its public constructor is where service dependencies are typically injected, adhering to the Dependency Inversion Principle. Its public handler methods (  
`OnGetAsync`, `OnPostAsync`, etc.) are the SUTs that must be individually validated.

It is crucial to define the scope of PageModel unit testing: the goal is to verify the PageModel’s C# handler logic (data retrieval, validation checks, service coordination, and result type). The unit test does  
_not_ attempt to verify the rendering of the Razor view (`.cshtml` file).

### 4.2 Isolating Page Handlers (OnGet, OnPost)

Page handlers must be tested independently to verify their distinct responses to different HTTP verbs and contexts.

#### OnGetAsync() Testing

This handler typically executes upon initial page load (HTTP GET) and is responsible for data initialization. The test objective is to verify that the handler correctly utilizes its mocked dependencies (e.g.,  

`IProductService`) to fetch required data and correctly assign that data to the public properties of the PageModel before returning a rendering result.

#### OnPostAsync() Testing

The `OnPostAsync()` handler processes HTTP POST requests, usually from form submissions, involving critical state-changing logic (creation, updating, deletion).  

1.  **Success Path:** The test must simulate a valid form submission by manually populating the PageModel's bound properties (the \`\` fields). It then verifies that the SUT correctly calls its mocked dependencies (e.g., confirming `_db.Products.Add` or `_db.SaveChangesAsync` was called via Moq verification) and returns the correct success result, often a `RedirectToPageResult`.  
    
2.  **Failure Path (Validation):** Unit tests must rigorously check validation logic. This involves manually injecting errors into the `ModelStateDictionary` _before_ calling the handler. The assertion must confirm that the handler detects the invalid state (`!ModelState.IsValid`) and immediately bypasses the business logic, returning a `PageResult` to re-render the view, rather than attempting a state-changing operation like a database update.  
    

#### Named Handlers and Polymorphism

Razor Pages supports named handlers (e.g., `OnPostDeleteAsync`, `OnPostArchiveAsync`), which allow a single PageModel object to manage multiple distinct actions in response to the same HTTP POST verb. This mechanism is an application of run-time polymorphism. Each named handler must be tested independently to ensure the SUT executes the specific, intended business logic for that action.  

Unit Testing Objectives for Razor PageModel Handlers

| Page Handler | HTTP Verb | Primary Test Objective | Key Mocking Focus |
| --- | --- | --- | --- |
| OnGetAsync() | GET | Verify data retrieval and correct initialization of page properties before rendering. | Mocking data services (DAL/Repository) to return predictable data. |
| OnPostAsync() | POST | Verify execution of state-changing logic (creation/update) and handling of redirects or result types. | Mocking data services, testing ModelState validation, and verifying SaveChanges calls. |
| OnPost<Name>Async() | POST (Named) | Verify specific actions (e.g., Delete, Archive) are triggered with correct parameters. | Mocking business services and ensuring only the intended action method is verified. |



### 4.3 Mocking Infrastructure Dependencies: Context, User Identity, and ModelState

The greatest challenge in testing PageModels is overcoming the implicit dependencies on the ASP.NET Core runtime, which normally provides objects like `HttpContext`, `User`, and `PageContext` automatically. For true unit isolation, these must be manually constructed or mocked during the Arrange phase.

#### Manual Setup of Web Context

To test critical functionality, the PageModel requires a complex setup of framework objects:

1.  **PageContext and ModelState:** To test validation and result handling, the test must manually create an `ActionContext` and a `PageContext`. The `ModelStateDictionary` is instantiated and populated with specific errors to simulate an invalid incoming request.  
    
2.  **Mocking User Identity:** If a PageModel’s logic depends on the authenticated user (e.g., retrieving `PageModel.User.Claims`), the test must construct a fake identity. This involves creating a `ClaimsPrincipal` object with the required claims and roles and injecting it into a `DefaultHttpContext`. This `HttpContext` is then assigned to the PageModel's `PageContext` property.  
    

The setup process is extensive and complex, but this overhead is a necessary trade-off for achieving isolation. It allows developers to test crucial validation and coordination logic reliably without the performance penalty or dependency issues associated with spinning up a full web server environment.  

#### Handling Data Flow and Model Binding in Tests

Unit tests execute outside the ASP.NET Core request pipeline; therefore, the model binder, which normally populates properties marked with the \`\` attribute, is bypassed.

When testing an `OnPost` handler, the test must simulate a successful model binding operation by manually populating the PageModel's input properties during the Arrange phase. For instance, if the PageModel has a `public Product Product { get; set; }` property, the test must execute `pageModel.Product = new Product { /*... test data... */ };` before calling `OnPostAsync`.

Finally, asserting the handler’s return type is mandatory. PageModel handler methods return `IActionResult` objects, which must be verified using framework assertions, such as `Assert.IsType<PageResult>(result)` for validation failure or `Assert.IsType<RedirectToPageResult>(result)` for success.  

Given the significant volume of setup code required to mock or instantiate the chain of web context dependencies (`HttpContext` → `ActionContext` → `PageContext` → `PageModel`), expert teams routinely create specialized `PageModelTestHelper` utility classes. These helpers encapsulate the complex, multi-step boilerplate instantiation, making the resulting unit tests cleaner and more maintainable.  

## V. Quality Assurance Best Practices and Maintainability

A unit test suite's long-term value is directly proportional to its readability, organization, and adherence to maintenance standards. These practices transform tests into reliable, executable documentation for the system's behavior.

### 5.1 Naming Conventions for Clarity and Searchability

Adopting a strict, descriptive naming convention is mandatory for team-wide quality assurance, ensuring tests serve as immediate documentation of system behavior. The established professional standard is a three-part naming convention, separated by underscores:  

`[Method/Class]_[Condition]_`.  

1.  **System Under Test (SUT):** The method or class being examined (e.g., `OrderService`, `OnPostAddMessageAsync`).
    
2.  **Condition/Scenario:** The state or input parameters that trigger the behavior (e.g., `WithInsufficientFunds`, `WhenModelStateIsInvalid`, `ValidCredentials`).
    
3.  **Expected Result:** The verifiable outcome (e.g., `ShouldThrowArgumentException`, `ReturnsPageResult`, `ShouldCallRepositoryOnce`).
    

**Example Enforcement:**

*   A test for a subtraction method becomes: `Subtract_NegativeResult_ReturnsCorrectValue`.
    
*   A test for a PageModel handler: `OnPostAddMessageAsync_WhenModelStateIsInvalid_ReturnsPageResult`.  
    

This naming scheme makes the purpose and outcome of every test instantly clear, improving collaboration and speeding up debugging by allowing developers to immediately understand the function's contract and anticipated behavior under specific conditions.

### 5.2 Test Project Organization and Separation of Concerns

Effective organization ensures that tests are easily locatable and that architectural principles are upheld structurally.

#### Mapping Production Code to Test Code

The test project structure should mirror the production code structure. For every production component (e.g.,  

`OrderService.cs`), there should be a corresponding test class (e.g., `OrderServiceTests.cs`). This tight correlation enables quick navigation within the IDE (e.g., using F12 to jump directly from a test body to the method under inspection).  

Crucially, the unit test project must only reference the production code project and the necessary test framework packages (xUnit, Moq, etc.). It must not reference any project containing infrastructure concerns (e.g., Entity Framework Core contexts, external storage APIs). This structural constraint enforces the fundamental architectural rule that unit tests must remain isolated from complex dependencies.  

### 5.3 Strategies for Refactoring Untestable Legacy Code

In legacy systems, it is common to encounter stateful, untestable Razor PageModels where complex business logic is embedded directly within handler methods (e.g., database queries or complex calculations inside `OnPostAsync`). To bring this code under test coverage, architectural refactoring is necessary.

#### De-scoping the PageModel

The primary strategy is to uphold the **Separation of Concerns** principle. All complex business logic must be migrated out of the PageModel and into a new, dedicated service class layer (e.g.,  

`IOrderService`). This service class is designed to be purely testable—stateless and reliant only on abstractions (interfaces) for its own dependencies.

The PageModel handler is then refactored to become a simple coordinator. Its only responsibilities are request handling, validation, coordinating the service call via DI, and determining the appropriate `IActionResult` (e.g., redirect or re-render).

#### The Interface Extraction Principle

If a legacy class or dependency cannot be immediately refactored, the **Interface Extraction Principle** is applied. An interface is extracted from the legacy class, and the PageModel is modified to depend on this new interface instead of the concrete class. This allows the legacy class to be brought under DI control. By depending on the interface, the PageModel's new coordinator logic can be unit tested immediately by injecting a mock of the interface, even while the legacy implementation remains untouched and untestable in the production environment. This architectural evolution enforces a gradual shift of functional responsibility away from stateful, infrastructure-dependent components (like the PageModel) into stateless, isolated service layers, ensuring that all new logic adheres to testability standards.

## VI. Conclusions and Recommendations

The successful application of unit testing in.NET is inherently linked to architectural excellence, particularly in decoupled frameworks like ASP.NET Core Razor Pages.

The analysis confirms that the testability of an application is a direct measure of its adherence to core OOP principles. Encapsulation defines the testable unit, while Abstraction, realized through interfaces, provides the mandatory decoupling. Dependency Injection is the practical mechanism that leverages this abstraction, allowing controlled substitution of dependencies via mocking frameworks like Moq.

For high-assurance software development in.NET, the following directives are essential:

1.  **Mandate Decoupling via Interfaces:** All interactions between the PageModel, business services, and data access layers must occur through interfaces. This ensures the capability to swap real services for mocks at test time.
    
2.  **Enforce the Coordinator Pattern:** PageModels must be limited to serving as coordinators, handling only request processing, model binding, and delegation. All database interaction, complex calculations, and business rules must reside in isolated, injectable service classes.
    
3.  **Standardize Tooling:** Adopt xUnit.net for test execution due to its parallelism and modern design, and mandate Moq for all dependency substitution, utilizing both `Setup()` for input control and `Verify()` for behavior observation.
    
4.  **Accept the Overhead of Isolation:** Recognize that testing Razor PageModels requires significant boilerplate code to manually construct the necessary web context (e.g., `PageContext`, `ModelState`, `ClaimsPrincipal`). This overhead is necessary to maintain complete isolation from the ASP.NET Core runtime environment, thereby ensuring test integrity.
    
5.  **Maintain Strict Conventions:** Implement the three-part `[Method]_[Condition]_` naming convention and enforce the structural separation of unit test projects from infrastructure concerns. These standards ensure the test suite is reliable, maintainable, and acts as living documentation for the system’s intended behavior.