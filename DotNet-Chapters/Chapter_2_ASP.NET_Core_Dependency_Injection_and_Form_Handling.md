# Chapter 2:Advanced Concepts in ASP.NET Core Razor Pages

## Part I: Working with Dependency Injection

# 

Dependency Injection (DI) is not merely a feature of ASP.NET Core but a foundational design pattern that is integral to the framework's architecture. It is a technique for achieving Inversion of Control (IoC) between classes and their dependencies, leading to code that is more modular, reusable, and testable

### Section 1: The Core Principles of Dependency Injection

# 

At its heart, Dependency Injection is a software design pattern that allows a class to receive its dependencies from an external source rather than creating them internally.This separation of concerns—constructing objects versus using them—is the key to building loosely coupled applications.

#### The Problem: Tight Coupling

# 

Consider a PageModel that needs to retrieve data from a repository. Without DI, the code might look like this:

C#
```shell
#  This is an example of tight coupling and should be avoided.  
public class IndexModel : PageModel{    
    public void OnGet(){        
# The IndexModel is directly responsible for creating its dependency.        
        CategoryRepository categoryRepository = new CategoryRepository();  
        List<Category> categories = categoryRepository.GetCategories();        #... use the categories    
}  
}
```
This approach introduces several problems:

●       **Lack of Flexibility:** If the implementation of CategoryRepository changes, or if we want to use a different repository, the IndexModel class itself must be modified.

●       **Difficult to Test:** When writing a unit test for IndexModel, it becomes difficult to isolate the class from its dependency. We cannot easily substitute a "mock" or "fake" repository for testing purposes.

●       **Complex Maintenance:** In a large application, the code to create and configure dependencies becomes scattered, making the system harder to maintain.

#### The Solution: Inversion of Control and Abstractions

# 

Dependency Injection solves these issues by inverting control and programming to abstractions (interfaces), not concrete implementations.This is a direct application of the

**Dependency Inversion Principle (DIP)**, which states that high-level modules should not depend on low-level modules; both should depend on abstractions.

The process involves three main steps :

1.     **Define an Abstraction:** Create an interface that defines the contract for the dependency.  
C#  
```shell
public interface ICategoryRepository{    
        List<Category> GetCategories();  
}  
  ```

2.     **Implement the Abstraction:** Create a concrete class that implements the interface.  
C# 
```shell 
public class CategoryRepository : ICategoryRepository{      
     public List<Category> GetCategories()    {        
#... implementation to get categories from a database        
            return new List<Category>();  
    }  
}
```
3.     **Inject the Abstraction:** The consuming class (the "client") requests the dependency via its constructor, depending only on the interface, not the concrete class.  
C# 
```shell 
public class IndexModel : PageModel{    
    private readonly ICategoryRepository repository;     #The dependency is "injected" through the constructor.    
    public IndexModel(ICategoryRepository repository)    {  
        repository = repository;  
    }    
    public void OnGet()    {        
    # The IndexModel uses the dependency without knowing how it was created.        
        List<Category> categories = repository.GetCategories();  
    }  
}
```
By following this pattern, the IndexModel is now decoupled from the CategoryRepository. It can work with any class that implements the ICategoryRepository interface, making the system flexible, maintainable, and easy to test.

### Section 2: The Built-in DI Container in ASP.NET Core

# 

ASP.NET Core is designed from the ground up to support DI and includes a built-in Inversion of Control (IoC) container, represented by the IServiceProvider interface. This container is responsible for creating instances of services and injecting them where they are needed.

#### Registering Services

# 

Services must be registered with the DI container before they can be injected. This is done in the Program.cs file by adding services to the IServiceCollection.

C#
```shell
#  In Program.cs  
    var builder = WebApplication.CreateBuilder(args); #Register the ICategoryRepository service with its concrete implementation.  
    builder.Services.AddScoped<ICategoryRepository, CategoryRepository>();  
  
#  Add services for Razor Pages.  
    builder.Services.AddRazorPages();  
  
    var app = builder.Build(); #... 
  ```

#### Service Lifetimes

# 

When registering a service, you must specify its lifetime, which tells the container how long an instance of the service should live. ASP.NET Core provides three primary service lifetimes :

●       **AddTransient():** A new instance of the service is created every time it is requested from the container. This is suitable for lightweight, stateless services.

●       **AddScoped():** A single instance of the service is created for each client request (within a scope). In a web application, this means one instance per HTTP request. This is the recommended lifetime for services that interact with a database, like an Entity Framework DbContext.

●       **AddSingleton():** A single instance of the service is created the first time it is requested, and this same instance is then used for all subsequent requests throughout the application's lifetime. This is suitable for services that need to maintain a shared state, like a caching service.

### Section 3: Consuming Registered Services

# 

Once a service is registered, the DI container can provide it to other classes. There are several ways to consume a service, with constructor injection being the most common and recommended approach.

●       **Constructor Injection:** This is the most explicit and reliable way to provide dependencies. The class declares its dependencies as parameters in its constructor. The DI container resolves these dependencies and passes them in when the class is instantiated.  
C#  
```shell
public class IndexModel : PageModel{    
    private readonly ICategoryRepository repository;    
    #  The ICategoryRepository is injected here.    
    
    public IndexModel(ICategoryRepository repository)    {  
        repository = repository;  
    }    #...  
}  
  ```

●       **Action Method Injection:** In some cases, a dependency might only be needed for a single action method. The \`\` attribute can be used to inject a service directly into a Razor Page handler method or an MVC controller action.  
C# 
```shell 
public class IndexModel : PageModel{    
    public IActionResult OnGet( ICategoryRepository repository)    {        
        var categories = repository.GetCategories();        
            #...        
        return Page();  
    }  
}
```
●       **Property Injection:** This involves injecting a dependency through a public property. The built-in ASP.NET Core DI container does not support property injection out of the box, and it is generally discouraged because it can hide a class's dependencies, making the code harder to understand and test.

## Part II: Working with Razor Pages Forms and Model Binding

# 

Handling user input through HTML forms is a fundamental aspect of web development. Razor Pages simplifies this process with a powerful set of features, including Tag Helpers and a robust model binding system, which work together to seamlessly connect your user interface with your server-side C# code.

### Section 1: Building and Handling Forms

# 

A typical workflow for handling forms in Razor Pages involves creating the form markup, defining a data model, and implementing a handler method to process the submission.

#### Form Markup and Tag Helpers

# 

In the .cshtml file, a standard HTML <form> element is used with the method attribute set to "post". ASP.NET Core enhances this with **Tag Helpers**, which are server-side components that process HTML elements to generate the final output. Key tag helpers for forms include

●       `<label asp-for="PropertyName">`: Generates a <label> element and its for attribute to match the specified model property.

●       `<input asp-for="PropertyName">`: Generates an `<input>` element with the id and name attributes set to match the model property. It also sets the type attribute based on the property's data type or data annotations.

●       `<span asp-validation-for="PropertyName">`: A placeholder to display validation error messages for the specified property.

HTML
```shell
@page  
@model CreateModel  
  
<form method\="post"\>    <div\>        <label asp-for\="Product.Name"\></label\>        <input asp-for\="Product.Name" />        <span asp-validation-for\="Product.Name"\></span\>    </div\>    <button type\="submit"\>Create</button\>  
</form\>  
  
```
Razor Pages are automatically protected against Cross-Site Request Forgery (XSRF/CSRF) attacks. The <form> tag helper injects a hidden anti-forgery token, which is validated on the server when the form is submitted.

#### Handling Form Submissions with OnPostAsync

# 

In the PageModel (.cshtml.cs file), you handle the form submission by implementing a page handler method that corresponds to the HTTP verb. For a form posted with method="post", the framework will invoke the OnPost() or OnPostAsync() handler.

This handler is responsible for:

1.     Checking if the submitted data is valid.

2.     Performing the required business logic (e.g., saving the data to a database).

3.     Redirecting the user to another page (a best practice known as the Post-Redirect-Get pattern) or re-displaying the form with validation errors.

### Section 2: The Power of Model Binding

# 

Model binding is the ASP.NET Core feature that automates the process of taking data from an HTTP request and mapping it to the parameters of a handler method or the public properties of a PageModel. It eliminates the need for manually parsing request data, converting it to.NET types, and assigning it to variables, which is a tedious and error-prone process.

#### Data Sources and Precedence

# 

The model binder can retrieve data from several sources in a specific order of precedence

1.     **Form values:** Data submitted in the body of a POST request.

2.     **Route data:** Values captured from the URL path (e.g., an id in /Products/123).

3.     **Query string parameters:** Values from the URL's query string (e.g., an id in /Products?id=123).

Understanding this precedence is crucial for debugging, as a value from a higher-priority source (like a form field) will override a value from a lower-priority source (like the query string) for the same parameter name.

#### Binding to PageModel Properties with 

# 

While you can bind incoming data directly to handler method parameters, the most common and powerful approach for forms is to bind the data to properties on the PageModel class. To enable this, you must decorate the property with the \`\` attribute. This attribute instructs the model binding system to populate the property with data from the incoming request when a non-GET request occurs.

C#
```shell
# In a PageModel class  
public class CreateModel : PageModel{    
#  The attribute enables model binding for this property on POST requests.
      public ProductInputModel Product { get; set;}       

      public async Task<IActionResult> OnPostAsync()      { 
           if (!ModelState.IsValid)  
        {           
    #   If validation fails, re-display the page.           
                return Page();  
        }        
#  At this point, the 'Product' property has been automatically        
#  populated with the submitted form data.        
# ... save the product to the database...        
        return RedirectToPage("./Index");  
    }  
}  
  
```
By default, only applies to POST requests for security reasons. To enable model binding for a property on GET requests, you must explicitly opt-in by setting the \`SupportsGet\` property to \`true\`

### Section 3: Validation

# 

Data validation is essential for ensuring the integrity of user-submitted data. Razor Pages integrates seamlessly with the data annotation validation attributes from the System.ComponentModel.DataAnnotations namespace.

You can apply validation rules directly to the properties of your input model:

C#
```shell
public class ProductInputModel{        
    public string Name { get; set; }      
    public decimal Price { get; set; }  
}  
  
```
In the OnPostAsync handler, you check the ModelState.IsValid property. This property will be false if any of the submitted data violates the validation rules defined by the data annotations. If the model state is invalid, you should return Page(), which re-renders the page. The <span asp-validation-for="..."> tag helpers will then automatically display the relevant error messages next to the form fields.