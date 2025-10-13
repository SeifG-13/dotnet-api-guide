# ASP.NET Core Web API - Complete Setup Guide

> 💡 **Framework:** .NET 9.0 | **EF Core:** 9.0.9 | **Last Updated:** October 2024

---

# 📋 Table of Contents

- Program.cs Setup
- appsettings.json Configuration  
- DbContext Setup
- Models & Entities
- Controllers & CRUD
- NuGet Packages
- EF Core Commands
- Quick Start Workflow
- Common Patterns
- Testing Your API
- Troubleshooting
- Quick Reference

---

# 1️⃣ Program.cs Setup

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
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")
    )
);

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

### 🔑 Key Points

- `AddControllers()` - Enables API controllers
- `AddDbContext<>()` - Registers database context
- `MapScalarApiReference()` - Adds API documentation UI
- `MapControllers()` - Maps controller endpoints

---

# 2️⃣ appsettings.json Configuration

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

## 📝 Connection String Examples

### Local SQL Server Express

```
Server=.\\SQLEXPRESS;Database=MyDb;Trusted_Connection=True;TrustServerCertificate=True
```

### SQL Server with Credentials

```
Server=localhost;Database=MyDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True
```

### Azure SQL Database

```
Server=tcp:yourserver.database.windows.net,1433;Database=MyDb;User ID=username;Password=password;Encrypt=True;
```

---

# 3️⃣ DbContext Setup

## Data/AppDbContext.cs

```csharp
using Microsoft.EntityFrameworkCore;
using YourProjectName.Models;

namespace YourProjectName.Data
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options) 
            : base(options)
        {
        }

        // Add your DbSets here
        public DbSet<YourModel> YourModels => Set<YourModel>();
        public DbSet<Product> Products => Set<Product>();
        public DbSet<Category> Categories => Set<Category>();

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // Seed data
            modelBuilder.Entity<Product>().HasData(
                new Product 
                { 
                    Id = 1, 
                    Name = "Product 1", 
                    Price = 9.99m 
                },
                new Product 
                { 
                    Id = 2, 
                    Name = "Product 2", 
                    Price = 19.99m 
                }
            );

            // Override table name (optional)
            modelBuilder.Entity<User>()
                .ToTable("tbl_Users");
            
            // Configure relationships
            modelBuilder.Entity<Product>()
                .HasOne(p => p.Category)
                .WithMany(c => c.Products)
                .HasForeignKey(p => p.CategoryId);
        }
    }
}
```

## 🎯 Understanding DbSet Syntax

```csharp
public DbSet<ModelName> PropertyName => Set<ModelName>();
```

**Breakdown:**

- `DbSet<ModelName>` → The type (collection of entities)
- `PropertyName` → What you use in code + becomes table name
- `Set<ModelName>()` → EF Core method

### ✅ Correct Examples

```csharp
// Standard convention - Plural property name
public DbSet<Product> Products => Set<Product>();
// Code: _context.Products.ToListAsync()
// Table: Products

// Custom table name
public DbSet<User> AppUsers => Set<User>();
// Code: _context.AppUsers.ToListAsync()
// Table: AppUsers
```

### ❌ Wrong Examples

```csharp
// Don't use plural in Set<>
public DbSet<Product> Products => Set<Products>();  
// ❌ Products is not a type!
```

### 💡 Key Rule

The name inside `Set<HERE>` must be your **Model class name** (singular), not the property name!

---

# 4️⃣ Models & Entities

## Simple Model

```csharp
namespace YourProjectName.Models
{
    public class Product
    {
        public int Id { get; set; }
        public required string Name { get; set; }
        public decimal Price { get; set; }
        public string? Description { get; set; }
        public DateTime CreatedAt { get; set; }
    }
}
```

## One-to-Many Relationship

```csharp
// Parent - One category has many products
public class Category
{
    public int Id { get; set; }
    public required string Name { get; set; }
    
    // Navigation property
    public List<Product>? Products { get; set; }
}

// Child - Many products belong to one category
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

## One-to-One Relationship

```csharp
public class VideoGame
{
    public int Id { get; set; }
    public string? Title { get; set; }
    
    // One-to-one
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

## Many-to-Many Relationship

```csharp
public class Student
{
    public int Id { get; set; }
    public required string Name { get; set; }
    
    // Many students can have many courses
    public List<Course>? Courses { get; set; }
}

public class Course
{
    public int Id { get; set; }
    public required string Title { get; set; }
    
    // Many courses can have many students
    public List<Student>? Students { get; set; }
}
```

> 💡 **Note:** EF Core automatically creates join table `CourseStudent`

## Preventing Circular References

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

# 5️⃣ Controllers & CRUD

## Basic Controller Template

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using YourProjectName.Data;
using YourProjectName.Models;

namespace YourProjectName.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ProductsController : ControllerBase
    {
        private readonly AppDbContext _context;

        public ProductsController(AppDbContext context)
        {
            _context = context;
        }

        // GET: api/Products
        [HttpGet]
        public async Task<ActionResult<List<Product>>> GetAll()
        {
            return Ok(await _context.Products.ToListAsync());
        }

        // GET: api/Products/5
        [HttpGet("{id}")]
        public async Task<ActionResult<Product>> GetById(int id)
        {
            var product = await _context.Products.FindAsync(id);
            
            if (product == null)
            {
                return NotFound();
            }
            
            return Ok(product);
        }

        // POST: api/Products
        [HttpPost]
        public async Task<ActionResult<Product>> Create(Product newProduct)
        {
            if (string.IsNullOrEmpty(newProduct.Name))
            {
                return BadRequest("Product name is required.");
            }

            _context.Products.Add(newProduct);
            await _context.SaveChangesAsync();

            return CreatedAtAction(
                nameof(GetById), 
                new { id = newProduct.Id }, 
                newProduct
            );
        }

        // PUT: api/Products/5
        [HttpPut("{id}")]
        public async Task<IActionResult> Update(int id, Product updatedProduct)
        {
            var product = await _context.Products.FindAsync(id);
            
            if (product == null)
            {
                return NotFound();
            }

            product.Name = updatedProduct.Name;
            product.Price = updatedProduct.Price;

            await _context.SaveChangesAsync();

            return NoContent();
        }

        // DELETE: api/Products/5
        [HttpDelete("{id}")]
        public async Task<IActionResult> Delete(int id)
        {
            var product = await _context.Products.FindAsync(id);
            
            if (product == null)
            {
                return NotFound();
            }

            _context.Products.Remove(product);
            await _context.SaveChangesAsync();

            return NoContent();
        }
    }
}
```

## Controller with Related Data

```csharp
[HttpGet]
public async Task<ActionResult<List<Product>>> GetAll()
{
    return Ok(await _context.Products
        .Include(p => p.Category)         // Load category
        .Include(p => p.Reviews)          // Load reviews
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

---

# 6️⃣ NuGet Packages

## Required Packages

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

## Install via CLI

```bash
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.AspNetCore.OpenApi
dotnet add package Scalar.AspNetCore
```

---

# 7️⃣ EF Core Commands

## Package Manager Console (Visual Studio)

| Command | Description |
| --- | --- |
| `Add-Migration MigrationName` | Create new migration |
| `Update-Database` | Apply migrations to database |
| `Remove-Migration` | Remove last migration (not applied) |
| `Script-Migration` | View SQL script |
| `Update-Database MigrationName` | Revert to specific migration |
| `Get-Migration` | List all migrations |
| `Drop-Database` | Drop the database |

## .NET CLI (Command Line)

| Command | Description |
| --- | --- |
| `dotnet ef migrations add MigrationName` | Create new migration |
| `dotnet ef database update` | Apply migrations |
| `dotnet ef migrations remove` | Remove last migration |
| `dotnet ef migrations script` | View SQL script |
| `dotnet ef database update MigrationName` | Revert to specific migration |
| `dotnet ef migrations list` | List all migrations |
| `dotnet ef database drop` | Drop the database |

---

# 8️⃣ Quick Start Workflow

## Step-by-Step Process

### Step 1: Create Models

Define your data structure in `Models/` folder

```csharp
// Models/Product.cs
public class Product
{
    public int Id { get; set; }
    public required string Name { get; set; }
    public decimal Price { get; set; }
}
```

### Step 2: Add DbSets to DbContext

Register entities in `Data/AppDbContext.cs`

```csharp
public DbSet<Product> Products => Set<Product>();
public DbSet<Category> Categories => Set<Category>();
```

### Step 3: Update Connection String

Set your database server in `appsettings.json`

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=.\\SQLEXPRESS;Database=MyDb;..."
}
```

### Step 4: Create Migration

```bash
Add-Migration InitialCreate
```

### Step 5: Update Database

```bash
Update-Database
```

### Step 6: Create Controller

Implement CRUD operations

### Step 7: Test API

Run project and visit `/scalar/v1` for documentation

---

## Example Workflow

```bash
# 1. Create models (Product.cs, Category.cs)

# 2. Add DbSets to DbContext
# public DbSet<Product> Products => Set<Product>();

# 3. Create migration
Add-Migration AddProductAndCategory

# 4. Apply to database
Update-Database

# 5. Create controller (ProductsController.cs)

# 6. Run and test
dotnet run
# Visit: https://localhost:7193/scalar/v1
```

---

# 9️⃣ Common Patterns

## Repository Pattern

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
    
    // ... other methods
}
```

## DTO Pattern (Data Transfer Objects)

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

// DTO (API Response)
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Usage
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

## CORS Configuration

```csharp
// In Program.cs
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

# 🔟 Testing Your API

## Scalar API Documentation

**URL:** `https://localhost:YOUR_PORT/scalar/v1`

### Features:

- ✅ Interactive API testing
- ✅ Automatic endpoint documentation
- ✅ Request/response examples
- ✅ Schema visualization
- ✅ Try API calls in browser

## HTTP Request Examples

### GET Request

```
GET https://localhost:7193/api/products
```

### POST Request

```
POST https://localhost:7193/api/products
Content-Type: application/json

{
  "name": "New Product",
  "price": 29.99
}
```

### PUT Request

```
PUT https://localhost:7193/api/products/1
Content-Type: application/json

{
  "name": "Updated Product",
  "price": 39.99
}
```

### DELETE Request

```
DELETE https://localhost:7193/api/products/1
```

---

# 1️⃣1️⃣ Troubleshooting

## Common Issues & Solutions

### ❌ Connection Error

**Problem:** "A connection was successfully established... but then an error occurred"

**Solution:** Add `TrustServerCertificate=True` to connection string

---

### ❌ Shadow State Error

**Problem:** "Cannot create shadow state" or missing properties

**Solution:** Ensure all navigation properties are nullable or properly configured

---

### ❌ Circular Reference JSON Error

**Problem:** Infinite loop when returning JSON

**Solution:** 

- Add `[JsonIgnore]` to navigation properties
- OR use DTOs instead of returning entities directly

---

### ❌ Migration Not Applying

**Problem:** Changes not reflected in database

**Solution:**

```bash
Remove-Migration
Add-Migration NewMigrationName
Update-Database
```

---

### ❌ DbSet Not Recognized

**Problem:** Property not available in controller

**Solution:** Ensure you're using `Set<ModelName>()` not `Set<PropertyName>()`

---

# 1️⃣2️⃣ Quick Reference

## Command Cheat Sheet

| Task | Command |
| --- | --- |
| Create migration | `Add-Migration MigrationName` |
| Apply migration | `Update-Database` |
| Remove migration | `Remove-Migration` |
| Run project | `dotnet run` |
| Add package | `dotnet add package PackageName` |
| List migrations | `Get-Migration` |

## HTTP Methods

| Method | Purpose | Returns |
| --- | --- | --- |
| GET | Retrieve data | 200 OK |
| POST | Create new | 201 Created |
| PUT | Update existing | 204 No Content |
| DELETE | Remove data | 204 No Content |

## Project Structure

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

# 📌 Key Takeaways

> ✅ **DbSet Syntax:** `public DbSet<Model> PropertyName => Set<Model>();`

> ✅ **Migration Flow:** Create Model → Add DbSet → Add-Migration → Update-Database

> ✅ **Controller Pattern:** Inject DbContext → Use async methods → Return ActionResult

> ✅ **Always use Include()** when loading related data

> ✅ **Test with Scalar** at `/scalar/v1` endpoint

---

**Framework:** .NET 9.0 | **EF Core:** 9.0.9 | **Last Updated:** October 2024