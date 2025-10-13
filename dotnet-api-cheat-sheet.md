# ASP.NET Core Web API - Complete Setup Guide

## 1. Program.cs

```csharp
using Microsoft.EntityFrameworkCore;
using Scalar.AspNetCore;
using YourProjectName.Data;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddOpenApi();

// Add DbContext with SQL Server
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.MapScalarApiReference();  // Scalar API documentation
    app.MapOpenApi();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

---

## 2. appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER_NAME;Database=YOUR_DATABASE_NAME;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

### Common Connection String Examples

```json
// Local SQL Server Express
"Server=.\\SQLEXPRESS;Database=MyDb;Trusted_Connection=True;TrustServerCertificate=True"

// SQL Server with username/password
"Server=localhost;Database=MyDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True"

// Azure SQL Database
"Server=tcp:yourserver.database.windows.net,1433;Database=MyDb;User ID=username;Password=password;Encrypt=True;"
```

---

## 3. Data/AppDbContext.cs

```csharp
using Microsoft.EntityFrameworkCore;
using YourProjectName.Models;

namespace YourProjectName.Data
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
        {
        }

        // Add your DbSets here
        // ✅ NAMING CONVENTION: Use plural of model name for property
        // Property name = Table name in database (by default)
        public DbSet<YourModel> YourModels => Set<YourModel>();
        
        // Examples:
        // public DbSet<Product> Products => Set<Product>();        // Table: Products
        // public DbSet<Category> Categories => Set<Category>();    // Table: Categories
        // public DbSet<VideoGame> VideoGames => Set<VideoGame>(); // Table: VideoGames

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // Configure relationships, constraints, seed data here
            
            // Example: Seed data
            // modelBuilder.Entity<Product>().HasData(
            //     new Product { Id = 1, Name = "Product 1", Price = 9.99m },
            //     new Product { Id = 2, Name = "Product 2", Price = 19.99m }
            // );

            // Example: Override table name
            // modelBuilder.Entity<User>().ToTable("tbl_Users");
            
            // Example: Configure relationships
            // modelBuilder.Entity<Product>()
            //     .HasOne(p => p.Category)
            //     .WithMany(c => c.Products)
            //     .HasForeignKey(p => p.CategoryId);
        }
    }
}
```

### Understanding DbSet Syntax

```csharp
public DbSet<ModelName> PropertyName => Set<ModelName>();
         ↑               ↑                    ↑
     Type (Model)   Property/Table      EF Core method
```

**Breakdown:**
- `DbSet<ModelName>`: The type - tells C# this is a collection of ModelName entities
- `PropertyName`: What you use in code AND becomes the table name in database
- `Set<ModelName>()`: EF Core method that returns the DbSet for that entity

**Examples:**

```csharp
// ✅ GOOD: Standard convention - Plural property name
public DbSet<Product> Products => Set<Product>();
// In code: _context.Products.ToListAsync()
// Database table name: Products

// ✅ GOOD: Custom table name (when needed)
public DbSet<User> AppUsers => Set<User>();
// In code: _context.AppUsers.ToListAsync()
// Database table name: AppUsers

// ❌ WRONG: Don't use plural in Set<>
public DbSet<Product> Products => Set<Products>();  // ❌ Products is not a type!

// ⚠️ WORKS but breaks convention
public DbSet<Product> ProductTable => Set<Product>();
// Confusing and non-standard naming
```

**Key Rule:** The name inside `Set<HERE>` must be your Model class name (singular), not the property name!

---

## 4. Models/YourModel.cs

```csharp
namespace YourProjectName.Models
{
    public class YourModel
    {
        public int Id { get; set; }
        public string? Name { get; set; }
        
        // Add your properties here
    }
}
```

### Common Model Examples

#### Simple Model
```csharp
public class Product
{
    public int Id { get; set; }
    public required string Name { get; set; }
    public decimal Price { get; set; }
    public string? Description { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

#### Model with One-to-Many Relationship
```csharp
// Parent
public class Category
{
    public int Id { get; set; }
    public required string Name { get; set; }
    
    // Navigation property - one category has many products
    public List<Product>? Products { get; set; }
}

// Child
public class Product
{
    public int Id { get; set; }
    public required string Name { get; set; }
    public decimal Price { get; set; }
    
    // Foreign key
    public int CategoryId { get; set; }
    
    // Navigation property
    public Category? Category { get; set; }
}
```

#### Model with One-to-One Relationship
```csharp
public class VideoGame
{
    public int Id { get; set; }
    public string? Title { get; set; }
    
    // One-to-one relationship
    public VideoGameDetails? VideoGameDetails { get; set; }
}

public class VideoGameDetails
{
    public int Id { get; set; }
    public string? Description { get; set; }
    public DateTime ReleaseDate { get; set; }
    
    // Foreign key
    public int VideoGameId { get; set; }
}
```

#### Model with Many-to-Many Relationship
```csharp
public class Student
{
    public int Id { get; set; }
    public required string Name { get; set; }
    
    // Navigation property - many students can have many courses
    public List<Course>? Courses { get; set; }
}

public class Course
{
    public int Id { get; set; }
    public required string Title { get; set; }
    
    // Navigation property - many courses can have many students
    public List<Student>? Students { get; set; }
}

// EF Core automatically creates join table: CourseStudent
```

#### Using JSON Ignore for Circular References
```csharp
using System.Text.Json.Serialization;

public class Genre
{
    public int Id { get; set; }
    public required string Name { get; set; }
    
    [JsonIgnore]  // Prevents infinite loop when serializing
    public List<VideoGame>? VideoGames { get; set; }
}
```

---

## 5. Controllers/YourController.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using YourProjectName.Data;
using YourProjectName.Models;

namespace YourProjectName.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class YourController : ControllerBase
    {
        private readonly AppDbContext _context;

        public YourController(AppDbContext context)
        {
            _context = context;
        }

        // GET: api/Your
        [HttpGet]
        public async Task<ActionResult<List<YourModel>>> GetAll()
        {
            return Ok(await _context.YourModels.ToListAsync());
        }

        // GET: api/Your/5
        [HttpGet("{id}")]
        public async Task<ActionResult<YourModel>> GetById(int id)
        {
            var item = await _context.YourModels.FindAsync(id);
            
            if (item == null)
            {
                return NotFound();
            }
            
            return Ok(item);
        }

        // POST: api/Your
        [HttpPost]
        public async Task<ActionResult<YourModel>> Create(YourModel newItem)
        {
            if (newItem == null)
            {
                return BadRequest("Invalid data.");
            }

            _context.YourModels.Add(newItem);
            await _context.SaveChangesAsync();

            return CreatedAtAction(nameof(GetById), new { id = newItem.Id }, newItem);
        }

        // PUT: api/Your/5
        [HttpPut("{id}")]
        public async Task<IActionResult> Update(int id, YourModel updatedItem)
        {
            var item = await _context.YourModels.FindAsync(id);
            
            if (item == null)
            {
                return NotFound();
            }

            // Update properties
            item.Name = updatedItem.Name;
            // Update other properties...

            await _context.SaveChangesAsync();

            return NoContent();
        }

        // DELETE: api/Your/5
        [HttpDelete("{id}")]
        public async Task<IActionResult> Delete(int id)
        {
            var item = await _context.YourModels.FindAsync(id);
            
            if (item == null)
            {
                return NotFound();
            }

            _context.YourModels.Remove(item);
            await _context.SaveChangesAsync();

            return NoContent();
        }
    }
}
```

### Controller with Related Data (Includes)

```csharp
[HttpGet]
public async Task<ActionResult<List<Product>>> GetAll()
{
    return Ok(await _context.Products
        .Include(p => p.Category)         // Load related category
        .Include(p => p.Reviews)          // Load related reviews
        .ToListAsync());
}

[HttpGet("{id}")]
public async Task<ActionResult<VideoGame>> GetById(int id)
{
    var game = await _context.VideoGames
        .Include(vg => vg.VideoGameDetails)
        .Include(vg => vg.Developer)
        .Include(vg => vg.Publisher)
        .Include(vg => vg.Genres)
        .FirstOrDefaultAsync(vg => vg.Id == id);
    
    if (game == null)
    {
        return NotFound();
    }
    
    return Ok(game);
}
```

### Controller with Better Validation

```csharp
[HttpPost]
public async Task<ActionResult<Product>> Create(Product newProduct)
{
    // Validate required fields
    if (string.IsNullOrEmpty(newProduct.Name))
    {
        return BadRequest("Product name is required.");
    }

    if (newProduct.Price <= 0)
    {
        return BadRequest("Price must be greater than zero.");
    }

    _context.Products.Add(newProduct);
    await _context.SaveChangesAsync();

    return CreatedAtAction(nameof(GetById), new { id = newProduct.Id }, newProduct);
}
```

---

## 6. Required NuGet Packages

Add these to your `.csproj` file:

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="9.0.9" />
  <PackageReference Include="Microsoft.EntityFrameworkCore" Version="9.0.9" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="9.0.9" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="9.0.9">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
  </PackageReference>
  <PackageReference Include="Scalar.AspNetCore" Version="2.8.11" />
</ItemGroup>
```

### Install via Command Line

```bash
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.AspNetCore.OpenApi
dotnet add package Scalar.AspNetCore
```

---

## 7. Essential EF Core Commands

### Package Manager Console (Visual Studio)

```powershell
# Create new migration
Add-Migration MigrationName

# Apply migrations to database
Update-Database

# Remove last migration (only if not applied to database)
Remove-Migration

# View SQL that will be executed
Script-Migration

# Revert database to specific migration
Update-Database MigrationName

# List all migrations
Get-Migration

# Drop database
Drop-Database
```

### .NET CLI (Command Line)

```bash
# Create new migration
dotnet ef migrations add MigrationName

# Apply migrations to database
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# View SQL script
dotnet ef migrations script

# Revert to specific migration
dotnet ef database update MigrationName

# List all migrations
dotnet ef migrations list

# Drop database
dotnet ef database drop
```

---

## 8. Quick Start Workflow

### Step-by-Step Process

1. **Create Models** → Define your data structure in `Models/` folder
2. **Add DbSets to DbContext** → Register entities in `Data/AppDbContext.cs`
3. **Update Connection String** → Set your database server in `appsettings.json`
4. **Create Migration** → Run `Add-Migration InitialCreate`
5. **Update Database** → Run `Update-Database`
6. **Create Controller** → Implement CRUD operations
7. **Test API** → Run project and visit `/scalar/v1` for API documentation

### Example Workflow

```bash
# 1. Create your models (Product.cs, Category.cs)

# 2. Add to DbContext
# public DbSet<Product> Products => Set<Product>();
# public DbSet<Category> Categories => Set<Category>();

# 3. Create migration
Add-Migration AddProductAndCategory

# 4. Check the migration file (optional)
# Review Migrations/XXXXXX_AddProductAndCategory.cs

# 5. Apply to database
Update-Database

# 6. Create controller (ProductsController.cs)

# 7. Run and test
dotnet run
# Visit: https://localhost:7193/scalar/v1
```

---

## 9. Common Patterns & Best Practices

### Repository Pattern (Optional)

```csharp
// Interface
public interface IRepository<T> where T : class
{
    Task<List<T>> GetAllAsync();
    Task<T?> GetByIdAsync(int id);
    Task<T> CreateAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

// Implementation
public class Repository<T> : IRepository<T> where T : class
{
    private readonly AppDbContext _context;
    private readonly DbSet<T> _dbSet;

    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public async Task<List<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync();
    }

    public async Task<T?> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }

    public async Task<T> CreateAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
        await _context.SaveChangesAsync();
        return entity;
    }

    public async Task UpdateAsync(T entity)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var entity = await _dbSet.FindAsync(id);
        if (entity != null)
        {
            _dbSet.Remove(entity);
            await _context.SaveChangesAsync();
        }
    }
}
```

### DTO Pattern (Data Transfer Objects)

```csharp
// Model (Database)
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public DateTime CreatedAt { get; set; }
    public bool IsDeleted { get; set; }
}

// DTO (API Response) - Only expose what's needed
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    // Exclude: CreatedAt, IsDeleted
}

// Usage in Controller
[HttpGet]
public async Task<ActionResult<List<ProductDto>>> GetAll()
{
    var products = await _context.Products
        .Where(p => !p.IsDeleted)
        .Select(p => new ProductDto
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price
        })
        .ToListAsync();
    
    return Ok(products);
}
```

### Global Error Handling

```csharp
// Add to Program.cs
app.UseExceptionHandler("/error");

// Create ErrorController.cs
[ApiController]
[Route("[controller]")]
public class ErrorController : ControllerBase
{
    [HttpGet]
    [HttpPost]
    [HttpPut]
    [HttpDelete]
    public IActionResult HandleError()
    {
        return Problem();
    }
}
```

### CORS Configuration (for frontend apps)

```csharp
// In Program.cs - Add before builder.Build()
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll",
        policy =>
        {
            policy.AllowAnyOrigin()
                  .AllowAnyMethod()
                  .AllowAnyHeader();
        });
});

// After app.Build()
app.UseCors("AllowAll");
```

---

## 10. Testing Your API

### Access API Documentation

**Scalar UI (Recommended):** `https://localhost:YOUR_PORT/scalar/v1`

Features:
- Interactive API testing
- Automatic endpoint documentation
- Request/response examples
- Schema visualization
- Try out API calls directly in browser

### Using Postman/Thunder Client

**GET Request:**
```
GET https://localhost:7193/api/products
```

**POST Request:**
```
POST https://localhost:7193/api/products
Content-Type: application/json

{
  "name": "New Product",
  "price": 29.99
}
```

**PUT Request:**
```
PUT https://localhost:7193/api/products/1
Content-Type: application/json

{
  "name": "Updated Product",
  "price": 39.99
}
```

**DELETE Request:**
```
DELETE https://localhost:7193/api/products/1
```

---

## 11. Troubleshooting Common Issues

### Issue: "A connection was successfully established... but then an error occurred"
**Solution:** Add `TrustServerCertificate=True` to connection string

### Issue: "Cannot create shadow state" or missing properties
**Solution:** Ensure all navigation properties are nullable or properly configured

### Issue: Circular reference error when returning JSON
**Solution:** Add `[JsonIgnore]` to navigation properties or use DTOs

### Issue: Migration not applying changes
**Solution:** 
```bash
Remove-Migration
Add-Migration NewMigrationName
Update-Database
```

### Issue: DbSet property not recognized
**Solution:** Ensure you're using `Set<ModelName>()` not `Set<PropertyName>()`

---

## 12. Quick Reference Card

| Task | Command |
|------|---------|
| Create migration | `Add-Migration MigrationName` |
| Apply migration | `Update-Database` |
| Remove migration | `Remove-Migration` |
| Run project | `dotnet run` |
| Add package | `dotnet add package PackageName` |
| List migrations | `Get-Migration` |

| HTTP Method | Purpose | Returns |
|-------------|---------|---------|
| GET | Retrieve data | 200 OK |
| POST | Create new | 201 Created |
| PUT | Update existing | 204 No Content |
| DELETE | Remove data | 204 No Content |

---

## 13. Project Structure Example

```
YourProjectName/
├── Controllers/
│   ├── ProductsController.cs
│   └── CategoriesController.cs
├── Data/
│   └── AppDbContext.cs
├── Models/
│   ├── Product.cs
│   ├── Category.cs
│   └── VideoGame.cs
├── Migrations/
│   ├── 20241013_InitialCreate.cs
│   └── 20241014_AddProducts.cs
├── Program.cs
├── appsettings.json
└── YourProjectName.csproj
```

---

**Last Updated:** October 2024 | **Framework:** .NET 9.0 | **EF Core:** 9.0.9