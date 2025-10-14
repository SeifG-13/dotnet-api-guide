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
- Common Patterns & Best Practices
- Testing Your API
- Troubleshooting
- Quick Reference Card
- Project Structure
- **JWT Authentication & Security ⭐**
- **Advanced Security Features ⭐**
- **Implementation Order 🎯**
- **Authentication Flow Diagram 🔄**
- **Security Best Practices 🛡️**

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
    public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
    {
        // Add your DbSets here
        // ✅ NAMING CONVENTION: Use plural of model name for property
        // Property name = Table name in database (by default)
        public DbSet<YourModel> YourModels { get; set; }
        
        // Examples:
        // public DbSet<Product> Products { get; set; }        // Table: Products
        // public DbSet<Category> Categories { get; set; }    // Table: Categories
        // public DbSet<VideoGame> VideoGames { get; set; } // Table: VideoGames

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

## 🎯 Understanding DbSet Syntax

```csharp
public DbSet<ModelName> PropertyName { get; set; }
         ↑               ↑              ↑
     Type (Model)   Property/Table   Auto-property
```

---

# 4️⃣ Models & Entities

## Simple Model

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

## Common Model Example

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

## One-to-Many Relationship

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

---

# 5️⃣ Controllers & CRUD

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

---

# 6️⃣ NuGet Packages

## Basic Packages

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
|---------|-------------|
| `Add-Migration MigrationName` | Create new migration |
| `Update-Database` | Apply migrations to database |
| `Remove-Migration` | Remove last migration (not applied) |
| `Script-Migration` | View SQL script |
| `Update-Database MigrationName` | Revert to specific migration |
| `Get-Migration` | List all migrations |
| `Drop-Database` | Drop the database |

## .NET CLI (Command Line)

| Command | Description |
|---------|-------------|
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

1. **Create Models** → Define your data structure in `Models/` folder
2. **Add DbSets to DbContext** → Register entities in `Data/AppDbContext.cs`
3. **Update Connection String** → Set your database server in `appsettings.json`
4. **Create Migration** → Run `Add-Migration InitialCreate`
5. **Update Database** → Run `Update-Database`
6. **Create Controller** → Implement CRUD operations
7. **Test API** → Run project and visit `/scalar/v1` for API documentation

---

# 9️⃣ Common Patterns & Best Practices

## Repository Pattern (Optional)

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

---

# 🔟 Testing Your API

## Scalar UI (Recommended)

**URL:** `https://localhost:YOUR_PORT/scalar/v1`

### Features:

- ✅ Interactive API testing
- ✅ Automatic endpoint documentation
- ✅ Request/response examples
- ✅ Schema visualization
- ✅ Try out API calls directly in browser

---

# 1️⃣1️⃣ Troubleshooting Common Issues

## Issue: "A connection was successfully established... but then an error occurred"

**Solution:** Add `TrustServerCertificate=True` to connection string

## Issue: "Cannot create shadow state" or missing properties

**Solution:** Ensure all navigation properties are nullable or properly configured

## Issue: Circular reference error when returning JSON

**Solution:** Add `[JsonIgnore]` to navigation properties or use DTOs

---

# 1️⃣2️⃣ Quick Reference Card

## Command Cheat Sheet

| Task | Command |
|------|---------|
| Create migration | `Add-Migration MigrationName` |
| Apply migration | `Update-Database` |
| Remove migration | `Remove-Migration` |
| Run project | `dotnet run` |
| Add package | `dotnet add package PackageName` |
| List migrations | `Get-Migration` |

## HTTP Methods

| HTTP Method | Purpose | Returns |
|-------------|---------|---------|
| GET | Retrieve data | 200 OK |
| POST | Create new | 201 Created |
| PUT | Update existing | 204 No Content |
| DELETE | Remove data | 204 No Content |

---

# 1️⃣3️⃣ Project Structure Example

```
YourProjectName/
├── Controllers/
│   ├── ProductsController.cs
│   ├── CategoriesController.cs
│   └── AuthController.cs
├── Data/
│   ├── AppDbContext.cs
│   └── UserDbContext.cs
├── Entities/
│   ├── User.cs
│   ├── AuditLog.cs
│   └── RefreshToken.cs
├── Models/
│   ├── Product.cs
│   ├── Category.cs
│   ├── UserDto.cs
│   ├── TokenResponseDto.cs
│   └── RefreshTokenRequestDto.cs
├── Services/
│   ├── IAuthService.cs
│   ├── AuthService.cs
│   ├── IEmailService.cs
│   └── EmailService.cs
├── Validators/
│   └── UserDtoValidator.cs
├── Migrations/
│   ├── 20241013_InitialCreate.cs
│   └── 20241014_AddAuthEntities.cs
├── Program.cs
├── appsettings.json
└── YourProjectName.csproj
```

---

# 1️⃣4️⃣ JWT Authentication & Security

## 🔐 Overview

This section covers complete JWT authentication implementation with modern security features.

## 📦 Required Packages for JWT

```bash
# JWT Authentication
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer

# Identity for password hashing
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore

# Data validation
dotnet add package FluentValidation
dotnet add package FluentValidation.AspNetCore

# Rate Limiting (protection against abuse)
dotnet add package AspNetCoreRateLimit

# Email (for verification and password reset)
dotnet add package MailKit
dotnet add package MimeKit
```

## 📝 appsettings.json Configuration

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER;Database=AuthDb;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "AppSettings": {
    "Token": "YOUR_SUPER_SECRET_KEY_AT_LEAST_32_CHARACTERS_LONG!!!",
    "Issuer": "YourAppName",
    "Audience": "YourAppAudience",
    "AccessTokenExpirationMinutes": 15,
    "RefreshTokenExpirationDays": 7,
    "EmailVerificationTokenExpirationHours": 24,
    "PasswordResetTokenExpirationMinutes": 30
  },
  "EmailSettings": {
    "SmtpServer": "smtp.gmail.com",
    "SmtpPort": 587,
    "SenderEmail": "your-email@gmail.com",
    "SenderName": "Your App",
    "Username": "your-email@gmail.com",
    "Password": "your-app-password"
  },
  "IpRateLimiting": {
    "EnableEndpointRateLimiting": true,
    "StackBlockedRequests": false,
    "RealIpHeader": "X-Real-IP",
    "HttpStatusCode": 429
  }
}
```

## 🗃️ Entities/User.cs

```csharp
namespace YourProjectName.Entities
{
    public class User
    {
        public Guid Id { get; set; }
        public string Username { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string PasswordHash { get; set; } = string.Empty;
        public string Role { get; set; } = "User";
        
        // Email Verification
        public bool EmailVerified { get; set; } = false;
        public string? EmailVerificationToken { get; set; }
        public DateTime? EmailVerificationTokenExpiry { get; set; }
        
        // Password Reset
        public string? PasswordResetToken { get; set; }
        public DateTime? PasswordResetTokenExpiry { get; set; }
        
        // Refresh Token
        public string? RefreshToken { get; set; }
        public DateTime? RefreshTokenExpiryTime { get; set; }
        
        // 2FA (Two-Factor Authentication)
        public bool TwoFactorEnabled { get; set; } = false;
        public string? TwoFactorSecret { get; set; }
        
        // Remember Me
        public string? RememberMeToken { get; set; }
        public DateTime? RememberMeTokenExpiry { get; set; }
        
        // Account Info
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
        public DateTime? LastLoginAt { get; set; }
        public bool IsActive { get; set; } = true;
        public int FailedLoginAttempts { get; set; } = 0;
        public DateTime? LockoutEnd { get; set; }
    }
}
```

## 📊 Entities/AuditLog.cs

```csharp
namespace YourProjectName.Entities
{
    public class AuditLog
    {
        public int Id { get; set; }
        public Guid? UserId { get; set; }
        public string Username { get; set; } = string.Empty;
        public string Action { get; set; } = string.Empty;
        public string IpAddress { get; set; } = string.Empty;
        public string UserAgent { get; set; } = string.Empty;
        public DateTime Timestamp { get; set; } = DateTime.UtcNow;
        public bool Success { get; set; }
        public string? Details { get; set; }
    }
}
```

## 🔄 Entities/RefreshToken.cs

```csharp
namespace YourProjectName.Entities
{
    public class RefreshTokenEntity
    {
        public int Id { get; set; }
        public Guid UserId { get; set; }
        public string Token { get; set; } = string.Empty;
        public DateTime ExpiresAt { get; set; }
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
        public string CreatedByIp { get; set; } = string.Empty;
        public bool IsRevoked { get; set; } = false;
        public DateTime? RevokedAt { get; set; }
        public string? RevokedByIp { get; set; }
        public string? ReplacedByToken { get; set; }
        
        // Navigation
        public User? User { get; set; }
        
        public bool IsExpired => DateTime.UtcNow >= ExpiresAt;
        public bool IsActive => !IsRevoked && !IsExpired;
    }
}
```

## 🗄️ Data/UserDbContext.cs

```csharp
using Microsoft.EntityFrameworkCore;
using YourProjectName.Entities;

namespace YourProjectName.Data
{
    public class UserDbContext : DbContext
    {
        public UserDbContext(DbContextOptions<UserDbContext> options) : base(options)
        {
        }

        public DbSet<User> Users { get; set; }
        public DbSet<AuditLog> AuditLogs { get; set; }
        public DbSet<RefreshTokenEntity> RefreshTokens { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // User configuration
            modelBuilder.Entity<User>()
                .HasIndex(u => u.Email)
                .IsUnique();

            modelBuilder.Entity<User>()
                .HasIndex(u => u.Username)
                .IsUnique();

            // RefreshToken configuration
            modelBuilder.Entity<RefreshTokenEntity>()
                .HasOne(rt => rt.User)
                .WithMany()
                .HasForeignKey(rt => rt.UserId)
                .OnDelete(DeleteBehavior.Cascade);
        }
    }
}
```

## 📋 Models/DTOs

### UserDto.cs

```csharp
namespace YourProjectName.Models
{
    public class UserDto
    {
        public string Username { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string Password { get; set; } = string.Empty;
    }
}
```

### TokenResponseDto.cs

```csharp
namespace YourProjectName.Models
{
    public class TokenResponseDto
    {
        public required string AccessToken { get; set; }
        public required string RefreshToken { get; set; }
    }
}
```

### RefreshTokenRequestDto.cs

```csharp
namespace YourProjectName.Models
{
    public class RefreshTokenRequestDto
    {
        public Guid UserId { get; set; }
        public required string RefreshToken { get; set; }
    }
}
```

### LoginDto.cs

```csharp
namespace YourProjectName.Models
{
    public class LoginDto
    {
        public string Username { get; set; } = string.Empty;
        public string Password { get; set; } = string.Empty;
        public bool RememberMe { get; set; } = false;
    }
}
```

### PasswordResetDto.cs

```csharp
namespace YourProjectName.Models
{
    public class PasswordResetRequestDto
    {
        public string Email { get; set; } = string.Empty;
    }

    public class PasswordResetDto
    {
        public string Token { get; set; } = string.Empty;
        public string NewPassword { get; set; } = string.Empty;
    }
}
```

## ✅ Validators/UserDtoValidator.cs

```csharp
using FluentValidation;
using YourProjectName.Models;

namespace YourProjectName.Validators
{
    public class UserDtoValidator : AbstractValidator<UserDto>
    {
        public UserDtoValidator()
        {
            RuleFor(x => x.Username)
                .NotEmpty().WithMessage("Username required")
                .Length(3, 50).WithMessage("Username 3-50 characters")
                .Matches("^[a-zA-Z0-9_-]+$").WithMessage("Letters, numbers, - and _ only");

            RuleFor(x => x.Email)
                .NotEmpty().WithMessage("Email required")
                .EmailAddress().WithMessage("Valid email required");

            RuleFor(x => x.Password)
                .NotEmpty().WithMessage("Password required")
                .MinimumLength(8).WithMessage("Min 8 characters")
                .Matches("[A-Z]").WithMessage("One uppercase required")
                .Matches("[a-z]").WithMessage("One lowercase required")
                .Matches("[0-9]").WithMessage("One digit required")
                .Matches("[^a-zA-Z0-9]").WithMessage("One special character required");
        }
    }
}
```

## 🔧 Program.cs (Complete JWT Configuration)

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using System.Text;
using AspNetCoreRateLimit;
using FluentValidation;
using FluentValidation.AspNetCore;
using Scalar.AspNetCore;
using YourProjectName.Data;
using YourProjectName.Services;

var builder = WebApplication.CreateBuilder(args);

// ==================== CONTROLLERS ====================
builder.Services.AddControllers();

// ==================== VALIDATION ====================
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssemblyContaining<Program>();

// ==================== OPENAPI ====================
builder.Services.AddOpenApi();

// ==================== DATABASE ====================
builder.Services.AddDbContext<UserDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"));
});

// ==================== HTTP CONTEXT ACCESSOR ====================
builder.Services.AddHttpContextAccessor();

// ==================== RATE LIMITING ====================
builder.Services.AddMemoryCache();
builder.Services.Configure<IpRateLimitOptions>(options =>
{
    options.GeneralRules = new List<RateLimitRule>
    {
        new RateLimitRule
        {
            Endpoint = "*",
            Period = "1m",
            Limit = 60
        },
        new RateLimitRule
        {
            Endpoint = "*/auth/login",
            Period = "1m",
            Limit = 5
        },
        new RateLimitRule
        {
            Endpoint = "*/auth/register",
            Period = "1h",
            Limit = 3
        }
    };

    options.QuotaExceededResponse = new QuotaExceededResponse
    {
        Content = "{{ \"message\": \"Too many requests. Please try again later.\", \"retryAfter\": \"{0}\" }}",
        ContentType = "application/json",
        StatusCode = 429
    };

    options.EnableEndpointRateLimiting = true;
    options.StackBlockedRequests = false;
});

builder.Services.AddSingleton<IIpPolicyStore, MemoryCacheIpPolicyStore>();
builder.Services.AddSingleton<IRateLimitCounterStore, MemoryCacheRateLimitCounterStore>();
builder.Services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();
builder.Services.AddSingleton<IProcessingStrategy, AsyncKeyLockProcessingStrategy>();
builder.Services.AddInMemoryRateLimiting();

// ==================== CORS ====================
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins(
                "http://localhost:3000",
                "http://localhost:4200",
                "http://localhost:5173"
            )
            .AllowAnyHeader()
            .AllowAnyMethod()
            .AllowCredentials();
    });
});

// ==================== JWT AUTHENTICATION ====================
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["AppSettings:Issuer"],
            ValidAudience = builder.Configuration["AppSettings:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["AppSettings:Token"]!)
            ),
            ClockSkew = TimeSpan.Zero
        };
    });

// ==================== HEALTH CHECKS ====================
builder.Services.AddHealthChecks()
    .AddDbContextCheck<UserDbContext>("database");

// ==================== SERVICES ====================
builder.Services.AddScoped<IAuthService, AuthService>();
builder.Services.AddScoped<IEmailService, EmailService>();

var app = builder.Build();

// ==================== MIDDLEWARE PIPELINE ====================
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference();
}

app.UseHttpsRedirection();

// ⚠️ ORDER IS CRITICAL - DO NOT CHANGE ⚠️
app.UseIpRateLimiting();        // 1. Rate limiting FIRST
app.UseCors("AllowFrontend");   // 2. CORS
app.UseAuthentication();         // 3. Authentication (BEFORE Authorization!)
app.UseAuthorization();          // 4. Authorization

app.MapControllers();
app.MapHealthChecks("/health");

app.Run();
```

---

# 1️⃣5️⃣ Advanced Security Features

## 📝 Implemented Features Checklist

### ✅ Basic Features

- [x] **Registration** - Sign up with validation
- [x] **Login** - Authentication with JWT
- [x] **Access Token** (15 minutes expiration)
- [x] **Refresh Token** (7 days expiration)
- [x] **Password Hashing** (with PasswordHasher)

### ✅ Advanced Features

- [x] **Email Verification** - Mandatory email verification
- [x] **Password Reset** - Reset password via email
- [x] **Remember Me** - Long-duration token (30 days)
- [x] **Logout** - Revoke active tokens
- [x] **Refresh Token Rotation** - Automatic token rotation
- [x] **Audit Logs** - Log all actions (who, when, what, IP)
- [x] **Rate Limiting** - Protection against brute force
- [x] **Account Lockout** - Lock after 5 failed attempts
- [x] **CORS** - Configuration for frontend
- [x] **Health Checks** - API status verification

---

# 1️⃣6️⃣ Implementation Order (Recommended)

## 🎯 Phase 1: Basic Setup (30 min)

1. Install NuGet packages
2. Create `appsettings.json` with configuration
3. Create 3 Entities (User, AuditLog, RefreshToken)
4. Create `UserDbContext`
5. Migration: `Add-Migration AddAuthEntities`
6. Apply: `Update-Database`

## 🎯 Phase 2: Models & Validators (15 min)

7. Create all DTOs (UserDto, TokenResponseDto, etc.)
8. Create Validators (UserDtoValidator)

## 🎯 Phase 3: Services (45 min)

9. Create `IEmailService` and `EmailService`
10. Create `IAuthService` and `AuthService`

## 🎯 Phase 4: Controller & Config (30 min)

11. Create `AuthController`
12. Configure `Program.cs` (JWT, CORS, Rate Limiting)

## 🎯 Phase 5: Testing (30 min)

13. Configure Gmail App Password
14. Test with `.http` file
15. Verify audit logs in database

**⏱️ Total estimated time: 2h30**

---

# 1️⃣7️⃣ Complete Authentication Flow

## 🔄 Flow Diagram

```
┌─────────────┐
│   CLIENT    │
└──────┬──────┘
       │
       │ 1. POST /register {username, email, password}
       ▼
┌─────────────────────────────────────────────────┐
│  API - Register                                 │
│  • Hash password                                │
│  • Generate email verification token            │
│  • Save user (EmailVerified = false)            │
│  • Send verification email                      │
└──────┬──────────────────────────────────────────┘
       │
       │ 2. User clicks link in email
       │    GET /verify-email?token=xxx
       ▼
┌─────────────────────────────────────────────────┐
│  API - Verify Email                             │
│  • Check token validity                         │
│  • Set EmailVerified = true                     │
└──────┬──────────────────────────────────────────┘
       │
       │ 3. POST /login {username, password}
       ▼
┌─────────────────────────────────────────────────┐
│  API - Login                                    │
│  • Verify credentials                           │
│  • Check EmailVerified = true                   │
│  • Generate Access Token (15 min)               │
│  • Generate Refresh Token (7 days)              │
│  • Save Refresh Token in DB                     │
│  • Log audit (IP, UserAgent)                    │
└──────┬──────────────────────────────────────────┘
       │
       │ Returns: {accessToken, refreshToken}
       ▼
┌─────────────┐
│   CLIENT    │  Stores tokens
│   Saves:    │  • accessToken in memory
│   • access  │  • refreshToken in httpOnly cookie
│   • refresh │    or secure storage
└──────┬──────┘
       │
       │ 4. GET /api/protected
       │    Authorization: Bearer {accessToken}
       ▼
┌─────────────────────────────────────────────────┐
│  API - Protected Endpoint                       │
│  • Validate JWT signature                       │
│  • Check expiration                             │
│  • Extract user claims                          │
└──────┬──────────────────────────────────────────┘
       │
       │ Returns: Protected data
       ▼
┌─────────────┐
│   CLIENT    │
└──────┬──────┘
       │
       │ 5. After 15 min, Access Token expires
       │    POST /refresh-tokens {userId, refreshToken}
       ▼
┌─────────────────────────────────────────────────┐
│  API - Refresh Tokens (with Rotation)          │
│  • Find Refresh Token in DB                     │
│  • Validate: not expired, not revoked           │
│  • Revoke old Refresh Token                     │
│  • Generate NEW Access Token                    │
│  • Generate NEW Refresh Token                   │
│  • Link old → new (ReplacedByToken)             │
│  • Save new Refresh Token in DB                 │
└──────┬──────────────────────────────────────────┘
       │
       │ Returns: {newAccessToken, newRefreshToken}
       ▼
┌─────────────┐
│   CLIENT    │  Updates tokens
└─────────────┘
```

---

# 1️⃣8️⃣ Security Best Practices

## 🛡️ DO's

✅ **HTTPS only** in production  
✅ **Short tokens** - Access Token max 30 min  
✅ **Token rotation** - Always implement  
✅ **Rate limiting** - Essential against brute force  
✅ **Audit logs** - Track ALL sensitive actions  
✅ **Secure secrets** - Use Azure Key Vault / AWS Secrets Manager in prod  
✅ **Email verification** - Mandatory for security  
✅ **Strong passwords** - Min 8 chars, uppercase, lowercase, digit, special

## ❌ DON'Ts

❌ **Token in URL** - Never expose token in query params  
❌ **No HTTPS** - Tokens stolen in transit  
❌ **Token too long** - Access Token > 1h = risk  
❌ **Hardcoded secrets** - Always use appsettings/env variables  
❌ **localStorage for tokens** - Use httpOnly cookies or memory  
❌ **No validation** - Always validate user inputs  
❌ **Ignore rate limiting** - Makes attacks easier

## 🔒 Production Checklist

- [ ] JWT key > 256 bits (32+ characters)
- [ ] HTTPS configured with valid certificate
- [ ] Secrets in Azure Key Vault / AWS
- [ ] Rate limiting enabled
- [ ] Audit logs in place
- [ ] Email verification mandatory
- [ ] CORS configured strictly (not "*")
- [ ] Health checks configured
- [ ] Active monitoring (Application Insights, Sentry)

---

# 1️⃣9️⃣ Troubleshooting JWT

## Issue: "401 Unauthorized" even with valid token

**Solution:**
- Verify that `UseAuthentication()` is BEFORE `UseAuthorization()`
- Check that secret key in appsettings matches the one used to generate token
- Verify token expiration with [jwt.io](https://jwt.io)

## Issue: "Bearer token not found"

**Solution:**
- Header must be: `Authorization: Bearer YOUR_TOKEN`
- No space after "Bearer" except before the token

## Issue: Email not sent

**Solution:**
- Verify Gmail App Password is correct (16 characters)
- Verify 2-step verification is enabled on Gmail
- Check ports: 587 for TLS, 465 for SSL

## Issue: Rate limiting not working

**Solution:**
- Verify that `UseIpRateLimiting()` is called BEFORE `UseAuthorization()`
- Check rules are configured in appsettings.json
- Test with multiple rapid requests

## Issue: Refresh token rotation fails

**Solution:**
- Verify token is not already revoked in database
- Check token is not expired
- Verify userId matches

---

# 2️⃣0️⃣ Gmail Configuration for Email

## 📧 Steps to Get Gmail App Password

1. Go to [Google Account](https://myaccount.google.com/)
2. Security → 2-Step Verification (enable if not already)
3. App passwords
4. Select "App" → Other → "YourAppName"
5. Copy the generated password (16 characters)
6. Put in appsettings.json:

```json
"EmailSettings": {
  "SmtpServer": "smtp.gmail.com",
  "SmtpPort": 587,
  "SenderEmail": "your-email@gmail.com",
  "SenderName": "Your App",
  "Username": "your-email@gmail.com",
  "Password": "xxxx xxxx xxxx xxxx"
}
```

---

# 2️⃣1️⃣ API Testing - Complete Examples

## 🧪 test-auth.http

```http
@baseUrl = https://localhost:7020
@accessToken = YOUR_ACCESS_TOKEN
@refreshToken = YOUR_REFRESH_TOKEN
@userId = YOUR_USER_ID
@emailVerificationToken = YOUR_EMAIL_TOKEN
@passwordResetToken = YOUR_RESET_TOKEN

### 1. Health Check
GET {{baseUrl}}/health

### 2. Register
POST {{baseUrl}}/api/auth/register
Content-Type: application/json

{
  "username": "testuser",
  "email": "test@example.com",
  "password": "Test@1234"
}

### 3. Verify Email
GET {{baseUrl}}/api/auth/verify-email?token={{emailVerificationToken}}

### 4. Login
POST {{baseUrl}}/api/auth/login
Content-Type: application/json

{
  "username": "testuser",
  "password": "Test@1234",
  "rememberMe": false
}

### 5. Login with Remember Me
POST {{baseUrl}}/api/auth/login
Content-Type: application/json

{
  "username": "testuser",
  "password": "Test@1234",
  "rememberMe": true
}

### 6. Access Protected Endpoint
GET {{baseUrl}}/api/auth
Authorization: Bearer {{accessToken}}

### 7. Refresh Tokens
POST {{baseUrl}}/api/auth/refresh-tokens
Content-Type: application/json

{
  "userId": "{{userId}}",
  "refreshToken": "{{refreshToken}}"
}

### 8. Request Password Reset
POST {{baseUrl}}/api/auth/request-password-reset
Content-Type: application/json

{
  "email": "test@example.com"
}

### 9. Reset Password
POST {{baseUrl}}/api/auth/reset-password
Content-Type: application/json

{
  "token": "{{passwordResetToken}}",
  "newPassword": "NewPassword@123"
}

### 10. Logout
POST {{baseUrl}}/api/auth/logout
Authorization: Bearer {{accessToken}}

### 11. Revoke Token
POST {{baseUrl}}/api/auth/revoke-token
Authorization: Bearer {{accessToken}}
Content-Type: application/json

{
  "token": "{{refreshToken}}"
}

### 12. Admin Only Endpoint
GET {{baseUrl}}/api/auth/admin-only
Authorization: Bearer {{accessToken}}

### 13. Test Rate Limiting (Try 6 times quickly)
POST {{baseUrl}}/api/auth/login
Content-Type: application/json

{
  "username": "testuser",
  "password": "WrongPassword123!"
}
```

---

# 2️⃣2️⃣ Using Audit Logs

## 📊 Useful Queries

### View all user actions

```csharp
[Authorize]
[HttpGet("audit-logs/me")]
public async Task<ActionResult<List<AuditLog>>> GetMyAuditLogs()
{
    var userId = Guid.Parse(User.FindFirst(ClaimTypes.NameIdentifier)?.Value!);
    
    var logs = await _context.AuditLogs
        .Where(a => a.UserId == userId)
        .OrderByDescending(a => a.Timestamp)
        .Take(50)
        .ToListAsync();
    
    return Ok(logs);
}
```

### View failed login attempts (Admin)

```csharp
[Authorize(Roles = "Admin")]
[HttpGet("audit-logs/failed-logins")]
public async Task<ActionResult<List<AuditLog>>> GetFailedLogins()
{
    var logs = await _context.AuditLogs
        .Where(a => a.Action == "Login" && !a.Success)
        .OrderByDescending(a => a.Timestamp)
        .Take(100)
        .ToListAsync();
    
    return Ok(logs);
}
```

### View suspicious IP connections

```csharp
[Authorize(Roles = "Admin")]
[HttpGet("audit-logs/suspicious-ips")]
public async Task<ActionResult> GetSuspiciousIPs()
{
    var suspiciousIPs = await _context.AuditLogs
        .Where(a => !a.Success)
        .GroupBy(a => a.IpAddress)
        .Where(g => g.Count() > 10)
        .Select(g => new
        {
            IpAddress = g.Key,
            FailedAttempts = g.Count(),
            LastAttempt = g.Max(a => a.Timestamp)
        })
        .ToListAsync();
    
    return Ok(suspiciousIPs);
}
```

---

# 2️⃣3️⃣ Refresh Token Rotation - How It Works

## 🔄 Principle

1. **Client** sends Refresh Token
2. **API** verifies the token
3. **API** revokes the old token
4. **API** generates NEW Access Token + NEW Refresh Token
5. **API** links old token to new (for audit)
6. **Client** receives new tokens

## ✅ Advantages

- If a Refresh Token is stolen, it becomes invalid at next rotation
- Complete traceability in `RefreshTokens` table
- Detection of compromised tokens (if old token used after rotation)

## 🔍 Compromised Token Detection

```csharp
// Add to AuthService
private async Task<bool> IsTokenCompromisedAsync(string token)
{
    var refreshToken = await _context.RefreshTokens
        .FirstOrDefaultAsync(rt => rt.Token == token);

    // If token was revoked AND has replacedByToken, it's suspicious
    if (refreshToken is not null && 
        refreshToken.IsRevoked && 
        !string.IsNullOrEmpty(refreshToken.ReplacedByToken))
    {
        // Someone is trying to use an old token
        // Action: Revoke ENTIRE token chain
        await RevokeDescendantRefreshTokensAsync(refreshToken);
        return true;
    }

    return false;
}

private async Task RevokeDescendantRefreshTokensAsync(RefreshTokenEntity refreshToken)
{
    // Recursively revoke all descendant tokens
    if (!string.IsNullOrEmpty(refreshToken.ReplacedByToken))
    {
        var childToken = await _context.RefreshTokens
            .FirstOrDefaultAsync(rt => rt.Token == refreshToken.ReplacedByToken);
        
        if (childToken is not null && childToken.IsActive)
        {
            childToken.IsRevoked = true;
            childToken.RevokedAt = DateTime.UtcNow;
            await RevokeDescendantRefreshTokensAsync(childToken);
        }
    }
    
    await _context.SaveChangesAsync();
}
```

---

# 2️⃣4️⃣ 2FA (Two-Factor Authentication) - Implementation Guide

## 📦 Required Packages

```bash
dotnet add package OtpNet
dotnet add package QRCoder
```

## 🔐 User Entity (Already added)

```csharp
public bool TwoFactorEnabled { get; set; } = false;
public string? TwoFactorSecret { get; set; }
```

## 🛠️ 2FA Service

```csharp
// Add to IAuthService
Task<string> Enable2FAAsync(Guid userId);
Task<bool> Verify2FAAsync(Guid userId, string code);
Task<bool> Disable2FAAsync(Guid userId, string code);

// Implementation in AuthService
public async Task<string> Enable2FAAsync(Guid userId)
{
    var user = await _context.Users.FindAsync(userId);
    if (user is null) throw new Exception("User not found");

    var secret = OtpNet.KeyGeneration.GenerateRandomKey(20);
    var base32Secret = OtpNet.Base32Encoding.ToString(secret);
    
    user.TwoFactorSecret = base32Secret;
    user.TwoFactorEnabled = false; // Will be enabled after verification
    
    await _context.SaveChangesAsync();

    // Generate QR code URL
    var appName = "YourAppName";
    var qrCodeUrl = $"otpauth://totp/{appName}:{user.Email}?secret={base32Secret}&issuer={appName}";
    
    return qrCodeUrl;
}

public async Task<bool> Verify2FAAsync(Guid userId, string code)
{
    var user = await _context.Users.FindAsync(userId);
    if (user is null || string.IsNullOrEmpty(user.TwoFactorSecret))
    {
        return false;
    }

    var secretBytes = OtpNet.Base32Encoding.ToBytes(user.TwoFactorSecret);
    var totp = new OtpNet.Totp(secretBytes);
    
    if (totp.VerifyTotp(code, out _, new VerificationWindow(2, 2)))
    {
        user.TwoFactorEnabled = true;
        await _context.SaveChangesAsync();
        return true;
    }

    return false;
}
```

---

# 2️⃣5️⃣ Migration Commands

## 🗄️ Essential Commands

```bash
# Create migration for User, AuditLog, RefreshToken
dotnet ef migrations add AddAuthEntities

# Apply migration
dotnet ef database update

# If you need to add 2FA later
dotnet ef migrations add Add2FASupport
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# List all migrations
dotnet ef migrations list

# Generate SQL script
dotnet ef migrations script
```

---

# 2️⃣6️⃣ Next Steps (Optional Features)

## 🔜 Features to Implement

### 1. Implement 2FA (Two-Factor Authentication)

- Package: `OtpNet`, `QRCoder`
- Endpoints: `/enable-2fa`, `/verify-2fa`, `/disable-2fa`

### 2. Social Login (Google, Facebook)

- Package: `Microsoft.AspNetCore.Authentication.Google`
- OAuth configuration

### 3. Device Management

- `UserDevices` table with browser fingerprint
- Notification on new device login

### 4. IP Whitelist/Blacklist

- `AllowedIPs` and `BlockedIPs` tables
- IP verification middleware

### 5. Session Management

- View all connected devices
- Disconnect specific devices

---

# 2️⃣7️⃣ Useful Resources

## 📚 Documentation & Tools

- [JWT.io](https://jwt.io/) - Decode and verify JWT
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Microsoft Identity Documentation](https://learn.microsoft.com/en-us/aspnet/core/security/)
- [FluentValidation Docs](https://docs.fluentvalidation.net/)
- [ASP.NET Core Security Best Practices](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/)
- [Entity Framework Core Docs](https://learn.microsoft.com/en-us/ef/core/)

---

# 2️⃣8️⃣ Final Implementation Checklist

## ✅ Complete Steps

### Initial Setup

- [ ] Install all required NuGet packages
- [ ] Create entities (User, AuditLog, RefreshTokenEntity)
- [ ] Create DbContext with DbSets
- [ ] Configure appsettings.json

### Models & Validators

- [ ] Create Models/DTOs
- [ ] Create Validators (FluentValidation)

### Services

- [ ] Implement IEmailService and EmailService
- [ ] Implement IAuthService and AuthService

### Controllers & Configuration

- [ ] Create AuthController
- [ ] Configure Program.cs (JWT, CORS, Rate Limiting, etc.)

### Database

- [ ] Create and apply migrations
- [ ] Verify tables are created

### Testing

- [ ] Configure email service (Gmail App Password)
- [ ] Test with .http file
- [ ] Verify audit logs in database
- [ ] Test rate limiting
- [ ] Verify all endpoints work

### Production

- [ ] Change JWT secret key (32+ characters)
- [ ] Configure HTTPS
- [ ] Move secrets to Azure Key Vault / AWS
- [ ] Configure monitoring
- [ ] Test security (penetration testing)

---

**Last Updated:** October 2024 | **Framework:** .NET 9.0 | **EF Core:** 9.0.9

**✨ Congratulations! You now have a complete guide for creating an ASP.NET Core API with JWT authentication and all modern security features!** 🎉