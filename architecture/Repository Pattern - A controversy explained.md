# Repository Pattern - A controversy explained
In this blog post, we'll dive into the ins and outs of the repository pattern and examine both its benefits and its potential drawbacks. We will start from the very basic to some more advanced use cases. So let's dive right into it.

The basics
----------

For a start, let's discuss some basics about the pattern itself, what it should solve and what some basic implementation could look like.

The repository pattern is a design pattern that provides a way to abstract the data access layer in an application. It acts as an intermediary between an application's data access layer and business logic layer, effectively decoupling the two.

A repository is typically implemented as a class containing methods for performing CRUD (create, read, update, and delete) operations on a particular entity type. The methods in a repository typically return domain objects rather than data transfer objects or data access objects, which allows the business logic layer to work with objects specific to the problem domain rather than objects specific to the data access technology being used.

Imagine you have a blog post like the one you are reading right now. You will find all the CRUD operations useful for you. You want to create posts as well as update and delete them. And you, dear reader, also want to read them, so we have a method for retrieving either one item or a whole list. Here is a very basic example in combination with **Entity Framework**:

```null
public class BlogContext : DbContext
{
    public DbSet<BlogPost> BlogPosts { get; set; }
}

public class BlogPost
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    public DateTime CreatedAt { get; set; }
}

public interface IBlogRepository
{
    Task<IEnumerable<BlogPost>> GetAllAsync();
    Task<BlogPost> GetByIdAsync(int id);
    Task AddAsync(BlogPost post);
    Task UpdateAsync(BlogPost post);
    Task DeleteAsync(int id);
}

public class BlogRepository : IBlogRepository
{
    private readonly BlogContext _context;

    public BlogRepository(BlogContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<BlogPost>> GetAllAsync()
    {
        return await _context.BlogPosts.ToListAsync();
    }

    public async Task<BlogPost> GetByIdAsync(int id)
    {
        return await _context.BlogPosts.FindAsync(id);
    }

    public async Task AddAsync(BlogPost post)
    {
        await _context.BlogPosts.AddAsync(post);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateAsync(BlogPost post)
    {
        _context.BlogPosts.Update(post);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var post = await _context.BlogPosts.FindAsync(id);
        _context.BlogPosts.Remove(post);
        await _context.SaveChangesAsync();
    }
}

```

Often times you would use now such a repository in combination with a Dependency Injection Container:

```null
[ApiController]
[Route("api/[controller]")]
public class BlogController : ControllerBase
{
    private readonly IBlogRepository _blogRepository;

    public BlogController(IBlogRepository blogRepository)
    {
        _blogRepository = blogRepository;
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<BlogPost>> GetById(int id)
    {
        var post = await _blogRepository.GetByIdAsync(id);
        if (post == null)
        {
            return NotFound();
        }

        return Ok(post);
    }
}


```

From that approach, we can gain some advantages like:

*   Abstraction: It abstracts the data access layer from the business logic layer, allowing the two to evolve independently of each other. We can also replace the underlying storage or storage technology without touching the rest of our code.
*   Testability: It allows the business logic to be tested without the need for a real database, by providing a mock implementation of the repository.
*   Centralization: It provides a central location for data access logic, making it easier to manage and maintain.

The controversy
---------------

The controversy around the repository pattern centers around the fact that many developers argue that it adds unnecessary complexity and duplicates functionality that is already provided by modern Object-Relational Mapping (ORM) frameworks such as Entity Framework.

ORMs like Entity Framework already abstract the data access layer by providing a higher-level, object-oriented API for working with databases. They automatically handle the mapping of data between the database and the application's domain objects, and they provide a consistent interface for performing CRUD operations on the data. Because of these features, some developers argue that using the repository pattern in conjunction with an ORM like Entity Framework is redundant, as it adds an extra layer of abstraction without providing any additional value.

Let's have a closer look at Entity Framework (but the same holds true for RavenDb, MongoDb , you name it) how it does this abstraction. And it is quite simple: It implements `IQueryable`. `IQueryable<T>` is an interface provided by the .NET framework that represents a queryable collection of objects of type T. It's used to represent a query that can be executed against a data source. The interface defines several methods that allow you to manipulate the query, such as filtering, ordering, and selecting data.

In the context of Entity Framework, `IQueryable<T>` is an abstraction that represents a query against an Entity Framework data source. Entity Framework provides an implementation of the `IQueryable<T>` interface for querying a database. The `DbSet<T>` class, which represents a collection of entities of a specific type, implements this interface and allows you to use the methods defined in `IQueryable<T>` to build and execute a query against the database.

When you use the `DbSet<T>` class to query a database, Entity Framework automatically converts the query expression into the appropriate SQL query for the underlying database and returns the results as a collection of entities of the specified type.

Now here a bit salt from my side to the topic itself. Where `IQueryable` is an abstraction, it is a leaky one. When you use the `IQueryable<T>` methods to build a query, the query is not executed immediately but is translated into an expression tree representing the query. This expression tree can then be executed by the underlying data access technology to retrieve the data from the data source. The problem with this is that the expression tree can contain methods and properties that are specific to the underlying data access technology, such as `ToString()` or `Provider` property, which can make the code that uses the `IQueryable<T>` interface dependent on the specific implementation of the data access technology. This can make it harder to change the data access technology later on or to test the code that uses the `IQueryable<T>` interface. You might have seen calls like `ToListAsync()`, which sits on top of `IQueryable`. Often those functions are native to a specific provider (in that case Entity Framework).

So, in summary, we have an abstraction layer that might or might not abstract what is already abstracted.

Another point of controversy is that the repository pattern can make it harder to use advanced features of the ORM, such as lazy loading, caching, and query optimization, which are not exposed by the repository interface. In some cases, it's also argued that using the repository pattern can make the codebase more difficult to understand, as it can add unnecessary complexity and duplication of functionality.

Also you might think: Hey, don't have to write a lot of repositories if I have a lot domain objects? Well yes and no. I will show you the **generic repository pattern** approach. But also here we might find a problem: The more we generalize the harder it gets to fine-tune certain scenarios without gaining a lot of unwanted complexity.

Generic repository pattern
--------------------------

The generic version looks similiar to the one above. Instead of having a specific entity like `BlogPost` we have just some type:

```null
public interface IBlogRepository<T> where T : Entity
{
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> GetByIdAsync(int id);
    Task AddAsync(T post);
    Task UpdateAsync(T post);
    Task DeleteAsync(int id);
}

public class BlogRepository<T> : IBlogRepository<T> where T : Entity
{
    private readonly BlogContext _context;

    public BlogRepository(BlogContext context)
    {
        _context = context;
    }

    public async Task<T> GetByIdAsync(int id)
    {
        return await _context.DbSet<T>.FirstOrDefaultAsync(t => t.Id == id);
    }

```

Okay, that is more convenient. Now in combination with dependency injection, we can use the full power:

```null

services.AddScoped(typeof(IRepository<>), typeof(Repository<>));


public BlogController(IRepository<BlogPost> blogRepository)

```

Before I show you another neat thing to improve your repository further, if the situation allows it, let's pick up one point of the controversy I showed earlier. The generic version, as its name suggests, is generic. So how do we do filtering, ordering and maybe even pagination? Exactly this part becomes tricky.

Here just the interface to cover all three concerns at once:

```null
Task<IEnumerable<TProjection>> GetAllByProjectionAsync<TProjection>(  
    Expression<Func<TEntity, TProjection>> selector,  
    Expression<Func<TEntity, bool>> filter = null,  
    Expression<Func<TEntity, object>> orderBy = null,  
    bool descending = true,  
    int page = 1,  
    int pageSize = int.MaxValue);

```

Uff! Just reading that feels like a lot! The implementation is even trickier. [https://github.com/linkdotnet/Blog/blob/master/src/LinkDotNet.Blog.Infrastructure/Persistence/Sql/Repository.cs#L44](https://github.com/linkdotnet/Blog/blob/master/src/LinkDotNet.Blog.Infrastructure/Persistence/Sql/Repository.cs#L44) if you want to see an implementation of that.

So with all the information, we have so far, there is still one open and important question: When do we use the repository pattern?

When to use
-----------

1.  Working with legacy databases: If you are working with a legacy database that does not have a modern ORM, the repository pattern can help to abstract the data access logic and make it more manageable. Once you have migrated to a state where you feel comfortable, you can reevaluate whether or not you need this central abstraction.
2.  Supporting different data sources: If you want to support different data sources, such as relational databases, non-relational databases, XML, and Web Services. Let's say you have software that also offers plugins for various database engines. Sure then the repository pattern will help you out!
3.  Caching.If you want to cache data from the data source, the repository pattern can help to provide a consistent interface for interacting with the cache. Bonus points: You can use other design patterns like the [decorator pattern](https://steven-giesel.com/blogPost/35df92c5-d5dd-488b-9b6b-25ed12c0420e) to make this work.
4.  Multi-tenancy: If you want to support multiple tenants, the repository pattern can help to provide a consistent interface for interacting with the data source that can be optimized for multi-tenancy.

Conclusion
----------

In conclusion, the repository pattern is a design pattern that abstracts the data access layer of an application, providing a consistent interface for interacting with the data source. It can be useful in certain scenarios such as working with legacy databases, decoupling the data access logic, unit testing, supporting different data sources, centralizing data access logic, and supporting different architectural patterns. However, it's also considered a controversial pattern as many developers argue that it adds unnecessary complexity and duplicates functionality that is already provided by modern ORM frameworks like Entity Framework. It's important to weigh its pros and cons against the current architecture and toolset of the application before deciding to use it. It's worth noting that the repository pattern can be useful in certain cases, but it's not always the best solution for every scenario.