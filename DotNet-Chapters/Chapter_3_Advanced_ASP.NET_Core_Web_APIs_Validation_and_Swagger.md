# Chapter 3:Advanced ASP.NET Core: Web APIs, Validation, and Swagger

## Part I: Understand and Demonstrate Web APIs

# 

This part transitions from server-rendered pages to data-centric services. We will explore the architecture of Web API controllers in ASP.NET Core, which are fundamental for building modern, decoupled applications that serve data to a variety of clients, such as single-page applications (SPAs), mobile apps, or other backend services.

### Section 1: Introduction to Web API Controllers

# 

While Razor Pages are designed to build page-focused, server-rendered web applications, Web APIs serve a different but complementary purpose. A Web API is a framework for building HTTP-based services that typically return data, most commonly in JSON format, rather than HTML. This makes them ideal for scenarios where the user interface is managed on the client-side (e.g., with frameworks like React or Angular) or for providing data to other services.

#### The Role of the API Controller

# 

In ASP.NET Core, a Web API controller is a C# class responsible for handling incoming HTTP requests and returning formatted data as a response. Unlike Razor Pages, which are page-centric, or MVC controllers, which are designed to return views, API controllers are data-centric.

Key characteristics of a Web API controller include:

●       **Inheritance from ControllerBase:** While it's possible to inherit from the Controller class, the recommended practice for Web APIs is to inherit from ControllerBase. The ControllerBase class provides the essential functionality for handling HTTP requests without the extra overhead of view support, which is unnecessary for an API.

●       **The \[ApiController\] Attribute:** Applying this attribute to a controller class enables several "opinionated," API-specific behaviors that streamline development:

○       **Attribute Routing Requirement:** It enforces the use of attribute-based routing, making the connection between URLs and controller actions explicit and clear.

○       **Automatic HTTP 400 Responses:** If model validation fails, the framework automatically returns an HTTP 400 Bad Request response with details of the validation errors, saving you from writing boilerplate if (!ModelState.IsValid) checks in every action.

○       **Binding Source Inference:** It intelligently infers where to get data for action parameters (e.g., from the request body, route, or query string), reducing the need for explicit attributes like \`\` or \[FromQuery\].

Here is a basic example of a Web API controller:

C#
```csharp
[ApiController]  
public class ProductsController : ControllerBase  
{  
#      GET: api/products  
    [HttpGet]  
    public IEnumerable<string> Get()  
    {  
        return new string { "Product A", "Product B" };  
    }  
}  
  
```
In this example, the ")\] attribute uses a template where \[controller\] is replaced by the controller's name ("Products"), resulting in a base route of /api/products. The \[HttpGet\] attribute specifies that the Get method should handle HTTP GET requests to this route.

### Section 2: Building RESTful Services with Controllers

# 

Web APIs are typically designed to follow the principles of REpresentational State Transfer (REST), an architectural style that uses standard HTTP verbs (GET, POST, PUT, DELETE) to perform operations on resources.

#### Action Methods and HTTP Verbs

# 

Action methods are the public methods on a controller that handle requests. You map these methods to HTTP verbs using attributes:

●       **\[HttpGet\]:** Retrieves a resource or a list of resources.

●       **\[HttpPost\]:** Creates a new resource.

●       **\[HttpPut\]:** Updates an existing resource completely.

●       **\[HttpDelete\]:** Deletes a resource.

#### Returning Data with IActionResult

# 

While you can return data directly from an action method, it is a best practice to return an IActionResult or ActionResult<T>. This allows you to return standard HTTP status codes and responses, providing more meaningful feedback to the client.

Common IActionResult types include:

●       **Ok(data):** Returns an HTTP 200 OK status with the requested data.

●       **NotFound():** Returns an HTTP 404 Not Found status.

●       **BadRequest(errors):** Returns an HTTP 400 Bad Request status, often with validation errors.

●       **CreatedAtAction(actionName, routeValues, data):** Returns an HTTP 201 Created status, typically used after a POST request. It includes a Location header with the URL to the newly created resource.

#### Example: A Full CRUD Controller

# 

The following example demonstrates a controller for managing Car objects, implementing all four CRUD (Create, Read, Update, Delete) operations.

C#
```csharp
[ApiController]
[Route("api/[controller]")]
public class CarsController : ControllerBase
{
    private static List<Car> cars = new List<Car>
    {
        new Car { Id = 1, Make = "Toyota", Model = "Camry", Year = 2022, Price = 25000 }
    };

    # GET: api/Cars
    [HttpGet]
    public ActionResult<IEnumerable<Car>> GetCars()
    {
        return Ok(cars);
    }

    #  GET: api/Cars/1
    [HttpGet("{id}")]
    public ActionResult<Car> GetCar(int id)
    {
        var car = cars.FirstOrDefault(c => c.Id == id);
        if (car == null)
        {
            return NotFound();
        }
        return Ok(car);
    }

    #  POST: api/Cars
    [HttpPost]
    public ActionResult<Car> PostCar(Car car)
    {
        car.Id = cars.Max(c => c.Id) + 1; // Simple ID generation
        cars.Add(car);
        return CreatedAtAction(nameof(GetCar), new { id = car.Id }, car);
    }

    #  PUT: api/Cars/1
    [HttpPut("{id}")]
    public IActionResult PutCar(int id, Car car)
    {
        if (id != car.Id)
        {
            return BadRequest();
        }
        var existingCar = cars.FirstOrDefault(c => c.Id == id);
        if (existingCar == null)
        {
            return NotFound();
        }
        #  Update logic
        existingCar.Make = car.Make;
        existingCar.Model = car.Model;
        existingCar.Year = car.Year;
        existingCar.Price = car.Price;

        return NoContent();
    }

    #  DELETE: api/Cars/1
    [HttpDelete("{id}")]
    public IActionResult DeleteCar(int id)
    {
        var car = cars.FirstOrDefault(c => c.Id == id);
        if (car == null)
        {
            return NotFound();
        }
        cars.Remove(car);
        return NoContent();
    }
}
  
  
```
## Part II: Working with Model Validation and API Documentation

# 

Ensuring the integrity of incoming data and providing clear documentation are crucial for building robust and usable applications. This part covers how ASP.NET Core handles model validation and how to automatically generate interactive API documentation using Swagger.

### Section 3: Working with Model Validation

# 

Model validation is the process of ensuring that data submitted by a user conforms to the application's business rules before it is processed. ASP.NET Core provides a powerful and declarative way to handle validation using data annotation attributes.

#### Using Data Annotations

# 

Data annotations are attributes from the System.ComponentModel.DataAnnotations namespace that you apply directly to the properties of your model classes to define validation rules. This approach keeps the validation logic clean and closely tied to the data it protects.

Common built-in validation attributes include:

●       **\[Required\]**: Specifies that the property must have a value.

●      **\[StringLength\]`**: Specifies the minimum and maximum length for a string.

●       **\[Range\]**: Specifies the minimum and maximum value for a numeric type.

●       **\[EmailAddress\]**: Validates that the property has a valid email format.

●       **\[RegularExpression\]**: Validates the property against a specified regular expression pattern.

Here is an example of a model with data annotations applied:

C#
```csharp
public class ProductInputModel{        
    public string Name { get;set; }        
    public decimal Price { get; set; }  
}  
  ```

#### Checking ModelState.IsValid

# 

In both Razor Pages and controllers, the framework runs validation after model binding and populates a ModelState dictionary with any errors. You can check the ModelState.IsValid property to determine if the submitted data is valid.

**In a Razor Page PageModel:**

For Razor Pages, you must explicitly check ModelState.IsValid in your handler method. If it's invalid, you typically return the page to display the errors.

C#
```csharp
public class CreateModel : PageModel{      
    public ProductInputModel Product { get; set; }    
    public async Task<IActionResult> OnPostAsync()   {        
        if (!ModelState.IsValid)  
        {            
            return Page(); # Re-render the page with   validation errors.        
        }        
        # ... process valid data...     
   return RedirectToPage("./Index");  
    }  
}  
```  

**In a Web API Controller:**

When using the \[ApiController\] attribute, this check is performed automatically. If ModelState.IsValid is false, the framework intercepts the request and returns an HTTP 400 Bad Request response containing a JSON object with the validation errors. This eliminates the need for the if (!ModelState.IsValid) block in your action methods.

C#
```csharp
[ApiController]   
public class ProductsController : ControllerBase  
{  
    [HttpPost]  
    public IActionResult Create(ProductInputModel product)  
    {  
#          The \[ApiController\] attribute handles the ModelState check.  
#          If validation fails, a 400 response is returned automatically.  
#          This code is only reached if the model is valid.  
  
#         .. process valid data...  
        return Ok(product);  
    }  
}  
```  

### Section 4: Documenting and Testing APIs with Swagger (OpenAPI)

# 

When you create a Web API, you are creating a contract for other developers or services to use. Clear documentation is essential for making your API understandable and easy to consume. The **OpenAPI Specification** (formerly Swagger Specification) is a language-agnostic standard for describing REST APIs.

**Swagger UI** is a tool that takes an OpenAPI specification and generates a rich, interactive web-based UI. This UI allows users to visualize the API's resources, understand its operations, and test the endpoints directly from the browser.

#### Integrating Swagger with Swashbuckle

# 

In ASP.NET Core, the easiest way to integrate Swagger is by using the **Swashbuckle.AspNetCore** NuGet package. It automatically generates an OpenAPI specification from your controllers, routes, and models.

The integration process involves three main steps:

1.     Install the NuGet Package:  
In the Package Manager Console, run:  
Install-Package Swashbuckle.AspNetCore

2.     Register Swagger Services in Program.cs:  
Add the Swagger generator to the service collection. This service is responsible for inspecting your API and generating the OpenAPI document (openapi.json).  
C# 
```csharp
#  In Program.cs  
var builder = WebApplication.CreateBuilder(args);  
  
builder.Services.AddControllers();// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbucklebuilder.Services.AddEndpointsApiExplorer();  
builder.Services.AddSwaggerGen(); // Add this line  
  
```
3.     Enable the Swagger Middleware in Program.cs:  
Add the middleware that serves the generated JSON document and the Swagger UI itself. This is typically done only in the development environment.  
C#  
```csharp
var app = builder.Build();# Configure the HTTP request pipeline.  
if (app.Environment.IsDevelopment())  
{  
    app.UseSwagger(); # Serves the generated JSON file
    app.UseSwaggerUI(); # Serves the Swagger UI  
}  
//...

```
  

After completing these steps, running your application and navigating to /swagger in your browser will display the interactive Swagger UI, where you can explore and test all of your API endpoints.

#### Enhancing Documentation with XML Comments

# 

To make the generated documentation even more useful, Swashbuckle can read XML documentation comments from your code.

1.     Enable XML Documentation:  
In your project's .csproj file, add the following property to generate an XML documentation file at compile time:  
XML  
<PropertyGroup\>  <GenerateDocumentationFile\>true</GenerateDocumentationFile\>  
</PropertyGroup\>  
  

2.     Configure Swashbuckle to Use the XML File:  
Modify the AddSwaggerGen configuration in Program.cs to include the path to the generated XML file.  
C#
```csharp 
builder.Services.AddSwaggerGen(options =>  
{    var xmlFile = $"{System.Reflection.Assembly.GetExecutingAssembly().GetName().Name}.xml";    
    var xmlPath = System.IO.Path.Combine(AppContext.BaseDirectory, xmlFile);  
    options.IncludeXmlComments(xmlPath);  
});
```
3.     Add Comments to Your Code:  
Now you can add standard C# XML documentation comments (///) to your controller actions and models. These comments will appear in the Swagger UI, providing detailed descriptions for your API endpoints and schemas.  
C#
```csharp  
#  <summary>  
#  Creates a new Car.  
#  </summary>  
#  <param name="car">The car to create.</param>  
#  <returns>A newly created car.</returns>  
[HttpPost]  
public ActionResult<Car> PostCar(Car car){  
  //...  
}