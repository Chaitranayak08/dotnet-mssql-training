# Chapter 6 : Repository Pattern and Async Programming

## Part I: Understand the Repository Pattern

# 

As applications grow in complexity, maintaining a clean and organized architecture becomes crucial. The Repository Pattern is a widely adopted design pattern that helps achieve this by creating a clear separation between an application's business logic and its data access code.

### Section 1: What is the Repository Pattern?

# 

The Repository Pattern is a design pattern that acts as an abstraction layer, or a mediator, between the domain (business logic) layer and the data mapping (data access) layer of an application. Instead of having your

PageModel classes directly interact with a data access technology like Entity Framework Core, they communicate with a repository. The repository exposes a simple, collection-like interface for performing data operations, hiding the underlying details of how data is fetched, stored, or updated.

This separation provides several key architectural advantages.

●       **Decoupling**: Your business logic is no longer tightly coupled to a specific data access technology. The PageModel only knows about the repository's interface, not whether the data is coming from EF Core, Dapper, or another source. This makes it easier to switch data access technologies in the future.

●       **Testability**: By programming against an interface, you can easily create "mock" or "fake" implementations of the repository for unit testing. This allows you to test your business logic in isolation, without needing a live database connection.

●       **Centralized Data Access Logic**: All the logic for querying and updating data is centralized within the repository classes. This prevents data access code from being scattered throughout the application, making it easier to maintain, reuse, and debug.

### Section 2: Implementing the Repository Pattern in ASP.NET Core

# 

Implementing the Repository Pattern in an ASP.NET Core application typically involves creating a generic interface and class for common operations, and then registering them with the built-in dependency injection (DI) container.

#### Step 1: Define a Generic Repository Interface

# 

First, define a generic interface that outlines the standard CRUD (Create, Read, Update, Delete) operations. This interface will work for any entity type in your application.

C#
```csharp
// In a Repositories folder or a separate project  
public interface IRepository<T> where T : class{    
    Task<T> GetByIdAsync(Guid id);  
    Task<IEnumerable<T>> GetAllAsync();  
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);    
    Task AddAsync(T entity);    
    Task UpdateAsync(T entity);    
    Task DeleteAsync(Guid id);  
}  
```  


#### Step 2: Implement the Generic Repository

# 

Next, create a concrete implementation of the generic interface. This class will take the application's DbContext as a dependency and use it to perform the actual database operations.

C#
```csharp
// In a Repositories folder or a separate project  
public class Repository<T> : IRepository<T> where T : class{    
    protected readonly ApplicationDbContext dbContext;    
    public Repository(ApplicationDbContext dbContext)    {  
          dbContext = dbContext;  
    }    
    public async Task<IEnumerable<T>> GetAllAsync()  
    {        
    return await dbContext.Set<T>().ToListAsync();  
    }    
    public async Task<T> GetByIdAsync(Guid id)    {        
        return await dbContext.Set<T>().FindAsync(id);  
    }    
    public async Task AddAsync(T entity)    
    {        await dbContext.Set<T>().AddAsync(entity);       
            await dbContext.SaveChangesAsync();  
    }       
//... other method implementations...  
}    
```


#### Step 3: Register the Repository for Dependency Injection

# 

In your Program.cs file, register the generic repository with the DI container. This tells the framework to provide an instance of Repository<T> whenever an IRepository<T> is requested.

C#
```csharp
// In Program.cs  
builder.Services.AddScoped(typeof(IRepository<>),typeof(Repository<>));  
```

#### Step 4: Use the Repository in a PageModel

# 

Finally, you can inject the repository interface into your PageModel's constructor and use it to access data, completely abstracting away the DbContext.

C#
```csharp
public class IndexModel : PageModel{    
    private readonly IRepository<Employee>employeeRepository;    
    public IndexModel(IRepository<Employee> employeeRepository)    
    {  
        employeeRepository = employeeRepository;  
    }    
public IEnumerable<Employee> Employees { get; set; }    
public async Task OnGetAsync(){  
        Employees = await employeeRepository.GetAllAsync();  
    }  
}
```
By following this pattern, the IndexModel has no direct knowledge of Entity Framework Core. Its only dependency is the IRepository<Employee> interface, resulting in a cleaner, more maintainable, and highly testable application architecture.

## Part II: Apply Async Programming

# 

Modern web applications must be responsive and scalable. Asynchronous programming is a key technique in.NET for achieving these goals, especially in a web server environment like ASP.NET Core where efficiently managing resources is critical.

### Section 1: The Importance of Asynchronous Programming

# 

In a traditional **synchronous** program, when a long-running operation occurs—such as querying a database, calling an external API, or reading a large file—the executing thread is blocked. It sits idle, waiting for the operation to complete before it can move on to the next line of code.

In a web application, this is highly inefficient. Web servers have a limited pool of threads to handle incoming requests. If all threads are blocked waiting for I/O operations, the server cannot process new requests, leading to poor performance and scalability issues. The application becomes unresponsive.

**Asynchronous** programming solves this problem. When an async operation is started, it does not block the thread. Instead, the thread is released back to the thread pool to handle other work. When the long-running operation completes, a thread from the pool is used to resume the method's execution from where it left off. This allows a web server to handle a much larger number of concurrent requests with fewer threads, dramatically improving scalability.

### Section 2: The async and await Keywords

# 

C# provides two keywords, async and await, that simplify the writing of asynchronous code. They allow you to write async code that reads with the same logical flow as synchronous code.

●       **async**: This keyword is used to modify a method declaration. It signals to the compiler that the method is asynchronous and may contain one or more await expressions. An async method typically returns a Task (if it returns no value) or a Task<T> (if it returns a value of type T).

●       **await**: This operator is applied to a Task inside an async method. It tells the application to suspend the execution of the current method until the awaited task is complete. While suspended, control is returned to the method's caller, and the thread is freed to do other work.

### Section 3: Applying async/await in a Razor Pages Application

# 

Applying asynchronous programming in Razor Pages is straightforward and is considered a best practice for any I/O-bound operation.

Consider a **synchronous** handler method that fetches data from a database:

C#
```csharp
// Synchronous - BLOCKS the thread  
public class IndexModel : PageModel{    
    private readonly SchoolContext context;    
    //... constructor...    
    public IList<Student> Students { get;set; }    
    public void OnGet()    {        
        //This line blocks the thread until the database returns all students.        
        Students = context.Student.ToList();  
    }  
}
```
Now, let's convert this to be **asynchronous** using async and await:

C#
```csharp
// Asynchronous - DOES NOT block the thread  
public class IndexModel : PageModel{    
    private readonly SchoolContext context;    
    //... constructor...    
    public IList<Student> Students { get;set; }    
    public async Task OnGetAsync()    {        
        // The 'await' keyword frees the thread to do other work        
        // while the database query executes.        
        Students = await context.Student.ToListAsync();  
    }  
}
```

Key changes to make a handler method asynchronous:

1.     **Change the return type**: void becomes async Task. If the method returns a value, like IActionResult, it becomes async Task<IActionResult>.

2.     **Add the async keyword**: The method signature is marked with async.

3.     **Add the Async suffix**: By convention, asynchronous methods are named with an Async suffix (e.g., OnGetAsync).

4.     **Use await**: Call the asynchronous version of the I/O-bound method (e.g., ToListAsync() instead of ToList()) and precede it with the await keyword.

By making these simple changes, you significantly improve your application's ability to handle more users and remain responsive under load. A crucial best practice is to be "async all the way"—meaning if you call an async method, you should await it, and the calling method should in turn be async. Avoid blocking on async code with methods like .Wait() or .Result, as this can lead to deadlocks.