# Chapter 5: Entity Framework,Transactions and Error Handling in ASP.NET Core

## Part I: Apply Entity Framework

# 

This part introduces Entity Framework Core (EF Core), the modern object-relational mapper (ORM) for.NET. We will cover its setup and application in a Razor Pages project, demonstrating how it abstracts away raw database code and streamlines data access.

### Section 1: Introduction to Entity Framework Core

# 

Entity Framework (EF) Core is a lightweight, extensible, open-source, and cross-platform version of the popular Entity Framework data access technology. It functions as an Object-Relational Mapper (ORM), which enables.NET developers to work with a database using.NET objects, eliminating the need for most of the data-access code they usually need to write.

Instead of writing raw SQL and using ADO.NET objects like NpgsqlCommand and NpgsqlDataReader, developers can use EF Core to perform database operations through high-level, object-oriented code. EF Core handles the "translation" between your C# objects and the relational database tables, a process often referred to as mapping.

Key benefits of using EF Core include:

●       **Increased Productivity**: EF Core generates the necessary SQL commands for create, read, update, and delete (CRUD) operations, significantly reducing the amount of boilerplate data access code.

●       **Language-Integrated Query (LINQ)**: You can use LINQ to write strongly-typed queries against your database, which provides compile-time checking and IntelliSense.

●       **Database Provider Model**: EF Core supports various database engines (like SQL Server, PostgreSQL, SQLite, and MySQL) through a provider model, allowing you to switch databases with minimal code changes.

●       **Migrations**: EF Core includes a powerful migration feature that allows you to evolve your database schema over time as your model classes change, keeping the database in sync with your application's code.

### Section 2: Setting Up EF Core in a Razor Pages Project

# 

Integrating EF Core into an ASP.NET Core Razor Pages application is a straightforward process involving a few key steps.

1.     **Install NuGet Packages:**  
You need to install the EF Core provider for the database you intend to use, along with the tools package for migration commands. For example, to use PostgreSQL, you would install:

○       Npgsql.EntityFrameworkCore.PostgreSQL

○       Microsoft.EntityFrameworkCore.Tools

2.     **Create Model Classes:**  
Define plain C# classes (often called POCOs - Plain Old CLR Objects) that represent the entities in your application. These classes will be mapped to tables in the database. By convention, a property named Id or ClassNameId is treated as the primary key.1  
C#
```csharp  
// Example Model for a Movie  
public class Movie{    
    public int Id { get; set; }    
    public string? Title { get; set; }    
    public DateTime ReleaseDate { get; set; }    
    public string? Genre { get; set; }    
    public decimal Price { get; set; }  
}  
 ``` 

3.     **Create the Database Context (DbContext):**  
The DbContext class is the heart of EF Core's functionality. It represents a session with the database and is used to query and save data. You create a class that inherits from DbContext and contains DbSet<T> properties for each entity you want to manage.1  
C#
```csharp  
using Microsoft.EntityFrameworkCore;
public class SchoolContext : DbContext{    
    public SchoolContext(DbContextOptions<SchoolContext> options) : base(options)    {  
    }    
public DbSet<Movie> Movie { get; set; }  
}  
 ``` 

4.     **Register the DbContext:**  
The DbContext must be registered with the dependency injection (DI) container. This is done in Program.cs. You also provide the database connection string, which is typically stored in appsettings.json.1  
C# 
```csharp  
// In Program.cs  
var builder = WebApplication.CreateBuilder(args);// Add services to the container.  
builder.Services.AddRazorPages();  
builder.Services.AddDbContext<SchoolContext>(options =>  
    options.UseNpgsql(builder.Configuration.GetConnectionString("SchoolContext")));  
  
var app = builder.Build();//...  
```  

### Section 3: Performing CRUD Operations

# 

Once EF Core is configured, you can inject your DbContext into any PageModel and use it to perform CRUD operations.

C#
```csharp 
public class CreateModel : PageModel
{    
    private readonly ContosoUniversity.Data.SchoolContext.context;    
    public CreateModel(ContosoUniversity.Data.SchoolContext context)    
    {  
        context = context;  
    }      
    public Movie Movie { get; set; }    
    public async Task<IActionResult> OnPostAsync()    {        
        if (!ModelState.IsValid)  
            {   
                 return Page();  
            }        
        // CREATE operation        
            context.Movie.Add(Movie);        
            await context.SaveChangesAsync();        
            return RedirectToPage("./Index");  
    }  
}  
```  

●       **Read (All Records)**: To retrieve all records from a table, you can use the ToListAsync() method on the DbSet.  
C#  
```csharp 
public IList<Movie> Movies { get;set; }  
  
public async Task OnGetAsync(){  
    Movies = await context.Movie.ToListAsync();  
}
```
●       **Read (Single Record)**: To find a single record by its primary key, use FindAsync().  
C# 
```csharp  
public Movie Movie { get; set; }  
  
public async Task<IActionResult> OnGetAsync(int? id)        
{           
    
    if (id == null) return NotFound();  
    Movie = await context.Movie.FindAsync(id);    
    if (Movie == null) return NotFound();       
    return Page();  
}
```
●       **Update**: To update a record, you typically fetch it, modify its properties, and then call SaveChangesAsync().  
C#  
```csharp
public async Task<IActionResult> OnPostAsync(){  
    context.Attach(Movie).State = EntityState.Modified;    
    await context.SaveChangesAsync();    
    return RedirectToPage("./Index");  
}  
```  

●       **Delete**: To delete a record, you fetch it, remove it from the DbSet, and then call SaveChangesAsync().  
C# 
```csharp 
public async Task<IActionResult> OnPostAsync(int? id)
{    
    var movie = await context.Movie.FindAsync(id);    
    if(movie!= null)  
    {  
        context.Movie.Remove(movie);        
        await context.SaveChangesAsync();  
    }    
        return RedirectToPage("./Index");  
}  
```  

## Part II: Understand Transactions and Error Handling in Razor Pages

 This part delves into crucial data integrity and robustness concepts. We will explore how EF Core simplifies the use of parameterized queries to prevent security vulnerabilities, and how to manage transactions and handle errors gracefully.

### Section 1: Parameterized Queries and SQL Injection

# 
A common security vulnerability in database applications is **SQL injection**. This attack occurs when malicious user input is concatenated directly into a SQL query string, potentially allowing an attacker to alter the query's logic to view or manipulate sensitive data.

The primary defense against SQL injection is to **always use parameterized queries**. A parameterized query separates the SQL command from the data. The data values are sent to the database separately and are never treated as executable code.

**The ADO.NET Way (Manual Parameterization)**

With raw ADO.NET, you must manually create parameters and add them to the command object.

C#
```csharp
// Manually adding parameters with Npgsql  
await using var cmd = new NpgsqlCommand("INSERT INTO employee (first_name, last_name) VALUES (@fname, @lname)", connection);  
cmd.Parameters.AddWithValue("@fname", "John");  
cmd.Parameters.AddWithValue("@lname", "Doe");  
await cmd.ExecuteNonQueryAsync();
```
**The Entity Framework Core Way (Automatic Parameterization)**

A major advantage of EF Core is that it automatically parameterizes all queries generated from LINQ expressions. When you write a LINQ query, EF Core translates it into a safe, parameterized SQL query behind the scenes.

C#
```csharp
// LINQ query in a PageModel  
var movie = await context.Movie.FirstOrDefaultAsync(m =>m.Id == id);
```
The above LINQ query is automatically converted by EF Core into a parameterized SQL query similar to this:

SQL
```sql
SELECT * FROM Movie WHERE Id = @p0LIMIT 1;  
```  
By using LINQ, you are inherently protected from SQL injection attacks without needing to manually manage query parameters.

### Section 2: Transactions

# 

A database transaction is a single, atomic unit of work. It ensures that a group of database operations either all succeed or none of them do. This is governed by the ACID properties (Atomicity, Consistency, Isolation, Durability).

**Implicit Transactions in EF Core**

By default, EF Core uses implicit transactions. When you call SaveChangesAsync(), EF Core automatically wraps all the detected changes (inserts, updates, deletes) into a single transaction. If any part of the operation fails, the entire transaction is automatically rolled back, ensuring your data remains in a consistent state. For most standard CRUD operations, this default behavior is sufficient.

**Explicit Transactions**

For more complex scenarios, such as when you need to coordinate multiple SaveChangesAsync() calls or include other operations within a single unit of work, you can manage transactions explicitly.

C#
```csharp
public async Task<IActionResult> OnPostAsync()
{    // Begin a transaction    
  await using var transaction = await context.Database.BeginTransactionAsync();    
  try    
  {        
    // Operation 1        
    context.SomeTable.Add(newEntity1);        
    await context.SaveChangesAsync();        
    // Operation 2: Maybe call an external service or perform another save        
    context.AnotherTable.Add(newEntity2);        
    await context.SaveChangesAsync();        
    // If all operations succeed, commit the transaction 
    await transaction.CommitAsync();  
    }  
    catch (Exception ex)  
    {        
    // If any operation fails, roll back the entire transaction        
    await transaction.RollbackAsync();        
    // Log the error and handle it        
    ModelState.AddModelError(string.Empty, "An error occurred. The operation was rolled back.");        return Page();  
    }    return RedirectToPage("./Index");  
}   
```
### Section 3: Error Handling

# 

Robust applications must anticipate and gracefully handle potential errors, especially during database interactions. A common approach is to use try-catch blocks to manage exceptions.

When working with EF Core, a DbUpdateException is often thrown when SaveChangesAsync() fails due to issues like a foreign key violation or a concurrency conflict.

The following example shows how to implement error handling in a PageModel handler.

C#
```csharp
public async Task<IActionResult> OnPostAsync(){    
    if (!ModelState.IsValid)  
    {        
            return Page();  
    }    
    try    
    {  
        context.Movie.Add(Movie);        
        await context.SaveChangesAsync();        
        return RedirectToPage("./Index");  
    }  
    catch (DbUpdateException ex)  
    {        
    // Log the detailed exception for debugging purposes        
    logger.LogError(ex, "An error occurred while creating the movie.");        
    // Provide a user-friendly error message        
    ModelState.AddModelError(string.Empty, "Unable to save changes. " + "Try again, and if the problem persists, see your system administrator.");        return Page();  
    }  
    catch (Exception ex)  
    {        // Catch other potential exceptions        
        logger.LogError(ex, "An unexpected error occurred");  
        ModelState.AddModelError(string.Empty, "An unexpected error occurred.");        
return Page();  
    }  
}
```
By wrapping database calls in try-catch blocks, you can prevent the application from crashing, log valuable diagnostic information, and provide clear, user-friendly feedback, creating a more resilient and professional application.