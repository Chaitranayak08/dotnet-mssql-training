# Chapter 7:  Parallelism and Concurrency Control

## Part I: Parallelism Use Cases

# 

As applications are required to process more data and perform more complex computations, leveraging modern multi-core processors becomes essential for performance. Parallel programming in.NET, primarily through the Task Parallel Library (TPL), provides the tools to execute work simultaneously, significantly reducing execution time for CPU-intensive tasks.

### Section 1: Parallelism vs. Concurrency: A Critical Distinction

# 

While often used interchangeably, parallelism and concurrency are distinct concepts in software development. Understanding the difference is crucial for applying the correct technique to a given problem.

●       **Concurrency** is about **dealing with** multiple tasks at once. It's a structural concept that allows a program to manage several tasks by making progress on them in overlapping time periods. On a single-core processor, this is achieved through context switching, where the CPU rapidly switches between tasks, creating the illusion of simultaneous execution. Concurrency is primarily aimed at improving application responsiveness, especially for I/O-bound operations (like network requests or file access). Asynchronous programming with async/await is a key tool for achieving concurrency.

●       **Parallelism** is about **doing** multiple tasks at once. It is the simultaneous execution of tasks on multiple processing units (CPU cores). Parallelism is a hardware-dependent concept aimed at improving the speed and efficiency of CPU-bound operations by dividing a large task into smaller sub-tasks and running them in parallel.

In short, concurrency is about structure, while parallelism is about execution. A well-structured concurrent program can be more easily parallelized, but the two are not the same.

### Section 2: The Task Parallel Library (TPL) for CPU-Bound Work

# 

The Task Parallel Library (TPL) is a set of APIs in the System.Threading.Tasks namespace designed to simplify the process of adding parallelism and concurrency to.NET applications. The TPL dynamically scales the degree of concurrency to most efficiently use all available processors, handling the partitioning of work, thread scheduling, and other low-level details.

Two of the most common constructs for data parallelism in the TPL are Parallel.For and Parallel.ForEach.

●       **Parallel.For**: Executes a for loop in which iterations may run in parallel. It is suitable for loop-based work where the number of iterations is known beforehand.

●       **Parallel.ForEach**: Executes a foreach loop in which iterations may run in parallel. This is ideal for processing every item in a collection where the work for each item is independent of the others.

It is important to note that parallelism introduces overhead. For loops with very few iterations or where each iteration is extremely fast, the overhead of partitioning the work and managing threads can make a parallel loop slower than a standard sequential loop.

### Section 3: Common Use Cases for Parallelism

# 

Parallelism is most effective for CPU-bound tasks that can be broken down into independent chunks of work. It is generally not suited for I/O-bound operations, where asynchronous programming is the better choice.

#### Use Case 1: Data Parallelism on Collections

# 

One of the most common scenarios for parallelism is processing a large collection of items where the same computationally expensive operation needs to be performed on each item.

Imagine an application that needs to process tens of thousands of images to generate thumbnails. A sequential foreach loop would process one image at a time. By using Parallel.ForEach, multiple images can be processed simultaneously across different CPU cores, drastically reducing the total processing time.

C#
```csharp
// A large collection of file paths  
List<string> imagePaths = GetImagePaths();  
  
// Process the images in 
parallelParallel.ForEach(imagePaths, imagePath =>  
{    
    // CPU-intensive work: generate a thumbnail for each image    
    GenerateThumbnail(imagePath);  
});
```
Other examples include:

●       Performing complex calculations on a large set of financial data.

●       Migrating a large number of records from one system to another, where each migration involves significant data transformation.

●       Simultaneously hashing a large number of files.

#### Use Case 2: Invoking Multiple Independent Operations

# 

Sometimes an application needs to perform several distinct, long-running, CPU-intensive operations that are not dependent on each other. The Parallel.Invoke method is perfect for this scenario. It takes an array of Action delegates and executes them in parallel.

C#
```csharp
Parallel.Invoke(  
    () => HeavyComputation("A"),  
    () => HeavyComputation("B"),  
    () => HeavyComputation("C"),  
    () => HeavyComputation("D")  
);  
```  

This is useful in scenarios like:

●       Running different complex reports simultaneously.

●       Performing various stages of data analysis that can be executed independently before the results are aggregated.

#### Use Case 3: Parallel LINQ (PLINQ)

# 

PLINQ is a parallel implementation of LINQ to Objects. It allows you to easily introduce parallelism into your LINQ queries by simply adding the .AsParallel() extension method to the data source. PLINQ is particularly useful when you need to perform complex queries (filtering, ordering, transforming) on in-memory collections.

C#
```csharp
var source = Enumerable.Range(1, 10000);  
  
// Use PLINQ to find all numbers where the square root is an integer  
var parallelQuery = from num in source.AsParallel() where Math.Sqrt(num) % 1 == select num;
```
PLINQ is often a good choice when the order of the results is important, as it can preserve the original ordering, whereas Parallel.ForEach does not guarantee any specific order of execution.

## Part II: Understand Concurrency Control

# 

In any multi-user application that interacts with a shared database, a critical challenge is managing **concurrency**. A concurrency conflict occurs when multiple users attempt to read and then modify the same piece of data at the same time, which can lead to data inconsistencies and "lost updates".

### Section 1: Concurrency Control Strategies

# 

There are two primary strategies for managing concurrency conflicts in a database environment:

1.     **Pessimistic Concurrency**: This strategy assumes that conflicts are likely to happen. It works by **locking** data when it is read, preventing any other user from modifying it until the first user releases the lock. This is analogous to checking out a file from a source control system. While it guarantees that conflicts cannot occur, it is not practical for web applications. The disconnected nature of HTTP means that locks could be held for an indefinite amount of time, severely impacting application scalability and performance. Entity Framework Core provides no built-in support for pessimistic concurrency.

2.     **Optimistic Concurrency**: This strategy assumes that conflicts are rare. It does **not** lock data. Instead, when an update is attempted, the application checks to see if the data has been changed by another user since it was originally read. If a change is detected, a concurrency conflict is flagged, and the application must resolve it. This is the default and recommended approach for web applications and is fully supported by EF Core.

### Section 2: Implementing Optimistic Concurrency in EF Core

# 

EF Core implements optimistic concurrency by using a **concurrency token**. A concurrency token is a property that EF Core checks during an UPDATE or DELETE operation to verify that the data has not changed.

There are two common ways to configure a concurrency token:

#### Method 1: Using a Timestamp or RowVersion

# 

Many database systems have a special data type that is automatically updated every time a row is modified. In SQL Server, this is the rowversion (or timestamp) type. This is the most robust and common way to implement a concurrency token.

You can configure a property as a row version token by adding the [Timestamp] data annotation to a byte property in your entity class.

C#
```csharp
public class Department{    
    public int DepartmentID { get; set; }    
    public string Name { get; set; }    
    public decimal Budget { get; set; } 
     
    [Timestamp]  // Concurrency token
    public byte RowVersion { get; set; }  
}
```  
  

When EF Core reads a Department entity, it stores the RowVersion value. When it later generates an UPDATE statement, it includes the original RowVersion value in the WHERE clause.

SQL
```sql
UPDATE [Departments]
SET [Name] = @p0, [Budget] = @p1
WHERE [DepartmentID] = @p2 AND [RowVersion] = @p3;

```  

If another user has modified the row in the meantime, the RowVersion in the database will be different, the WHERE clause will not match any rows, and the UPDATE statement will report that zero rows were affected. EF Core interprets this as a concurrency conflict and throws a DbUpdateConcurrencyException.

#### Method 2: Using the \[ConcurrencyCheck\] Attribute

# 

If your table doesn't have a rowversion column, you can configure EF Core to use one or more existing properties as the concurrency token by applying the \[ConcurrencyCheck\] attribute.

C#
```csharp
public class Department{    
    public int DepartmentID { get; set; }  
  
    [ConcurrencyCheck]    
    public string Name { get; set; }    
    public decimal Budget { get; set; }  
}  
```  

With this configuration, EF Core will include the original value of the Name property in the WHERE clause of UPDATE and DELETE statements. This approach is generally less robust than using a dedicated rowversion column, as it's possible for multiple properties to be involved, leading to larger and potentially less performant WHERE clauses.

### Section 3: Handling Concurrency Conflicts

# 

When a concurrency conflict occurs, EF Core throws a DbUpdateConcurrencyException. Your application code is responsible for catching this exception and deciding how to resolve the conflict.

A common pattern in a Razor Pages OnPostAsync handler is to wrap the call to SaveChangesAsync() in a try-catch block.

C#
```csharp
public async Task<IActionResult> OnPostAsync(){    
    if (!ModelState.IsValid)  
    {        
        return Page();  
    }  
  
    context.Attach(Department).State = EntityState.Modified;    
    try {        
        await context.SaveChangesAsync();        
        return RedirectToPage("./Index");  
    }  
    catch (DbUpdateConcurrencyException ex)  
    {        
    // A concurrency conflict occurred.       
     // The exception exposes the conflicting entities.        
     var exceptionEntry = ex.Entries.Single();        
     var clientValues = (Department)exceptionEntry.Entity;        
     var databaseEntry = exceptionEntry.GetDatabaseValues();        
     if (databaseEntry == null)  
        {  
            ModelState.AddModelError(string.Empty,"Unable to save. The department was deleted by another user.");            
        return Page();  
        }        
    var dbValues = (Department)databaseEntry.ToObject();        
    // You can now compare clientValues and dbValues and decide how to resolve.        
    // For example, you could show an error message to the user, allowing them to see the new values and decide whether to overwrite them.        
    ModelState.AddModelError(string.Empty, "The record you attempted to edit " + "was modified by another user after you got the original value. Your " + "edit operation was canceled.");        
    // Update the RowVersion to the new value from the database        
    // so that if the user saves again, it will succeed (if no new conflict).        
    Department.RowVersion = (byte)dbValues.RowVersion;    
    ModelState.Remove("Department.RowVersion");        
    return Page();  
    }  
}
```

When a DbUpdateConcurrencyException is caught, you have three sets of values to work with to resolve the conflict :

1.     **Current Values**: The values your application was trying to write to the database.

2.     **Original Values**: The values that were originally read from the database before any edits were made.

3.     **Database Values**: The values currently stored in the database, which you can get from the exception object.

Based on these values, you can implement a resolution strategy, such as asking the user to confirm if they want to overwrite the changes, or attempting to merge the changes programmatically.