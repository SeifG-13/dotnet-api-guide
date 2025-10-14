### Entité User (Déjà ajouté)
```csharp
public bool TwoFactorEnabled { get; set; } = false;
public string? TwoFactorSecret { get; set; }
```

### Service 2FA
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

## 📊 Utilisation des Audit Logs

### Requêtes Utiles

```csharp
// Voir toutes les actions d'un utilisateur
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

// Voir les tentatives de connexion échouées (Admin)
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

// Voir les connexions par IP suspecte
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

## 🔄 Refresh Token Rotation - Comment ça marche ?

### Principe
1. **Client** envoie Refresh Token
2. **API** vérifie le token
3. **API** révoque l'ancien token
4. **API** génère NOUVEAU Access Token + NOUVEAU Refresh Token
5. **API** lie l'ancien token au nouveau (pour audit)
6. **Client** reçoit les nouveaux tokens

### Avantages
- ✅ Si un Refresh Token est volé, il devient invalide dès la prochaine rotation
- ✅ Traçabilité complète dans la table `RefreshTokens`
- ✅ Détection des tokens compromis (si ancien token utilisé après rotation)

### Détection de Token Compromis
```csharp
// Add to AuthService
private async Task<bool> IsTokenCompromisedAsync(string token)
{
    var refreshToken = await _context.RefreshTokens
        .FirstOrDefaultAsync(rt => rt.Token == token);

    // Si le token a été révoqué ET a un replacedByToken, c'est suspect
    if (refreshToken is not null && 
        refreshToken.IsRevoked && 
        !string.IsNullOrEmpty(refreshToken.ReplacedByToken))
    {
        // Quelqu'un essaie d'utiliser un ancien token
        // Action: Révoquer TOUTE la chaîne de tokens
        await RevokeDescendantRefreshTokensAsync(refreshToken);
        return true;
    }

    return false;
}

private async Task RevokeDescendantRefreshTokensAsync(RefreshTokenEntity refreshToken)
{
    // Révoquer récursivement tous les tokens descendants
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

## 🧪 Tests API - Exemples Complets

### test-auth.http
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

### 7.# ASP.NET Core Web API - Complete Setup Guide

## Table of Contents
1. [Program.cs](#1-programcs)
2. [appsettings.json](#2-appsettingsjson)
3. [Data/AppDbContext.cs](#3-dataappdbcontextcs)
4. [Models](#4-modelsyourmodelcs)
5. [Controllers](#5-controllersyourcontrollercs)
6. [NuGet Packages](#6-required-nuget-packages)
7. [EF Core Commands](#7-essential-ef-core-commands)
8. [Quick Start Workflow](#8-quick-start-workflow)
9. [Common Patterns](#9-common-patterns--best-practices)
10. [Testing](#10-testing-your-api)
11. [Troubleshooting](#11-troubleshooting-common-issues)
12. [Quick Reference](#12-quick-reference-card)
13. [Project Structure](#13-project-structure-example)
14. **[JWT Authentication Setup](#14-jwt-authentication--security) ⭐ NEW**
15. **[Advanced Security Features](#15-advanced-security-features) ⭐ NEW**

---

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

### Understanding DbSet Syntax

```csharp
public DbSet<ModelName> PropertyName { get; set; }
         ↑               ↑              ↑
     Type (Model)   Property/Table   Auto-property
```

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

---

## 6. Required NuGet Packages

### Basic Packages

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

---

## 11. Troubleshooting Common Issues

### Issue: "A connection was successfully established... but then an error occurred"
**Solution:** Add `TrustServerCertificate=True` to connection string

### Issue: "Cannot create shadow state" or missing properties
**Solution:** Ensure all navigation properties are nullable or properly configured

### Issue: Circular reference error when returning JSON
**Solution:** Add `[JsonIgnore]` to navigation properties or use DTOs

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

# 14. JWT Authentication & Security

## 🔐 Overview

Cette section couvre l'implémentation complète d'un système d'authentification JWT avec toutes les fonctionnalités de sécurité modernes.

## 📦 Packages Requis pour JWT

```bash
# Authentification JWT
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer

# Identity pour le hashing de mots de passe
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore

# Validation des données
dotnet add package FluentValidation
dotnet add package FluentValidation.AspNetCore

# Rate Limiting (protection contre les abus)
dotnet add package AspNetCoreRateLimit

# Email (pour vérification et reset password)
dotnet add package MailKit
dotnet add package MimeKit
```

## 🏗️ Structure de Projet avec JWT

```
YourProjectName/
├── Controllers/
│   └── AuthController.cs
├── Data/
│   └── UserDbContext.cs
├── Entities/
│   ├── User.cs
│   ├── AuditLog.cs
│   └── RefreshToken.cs
├── Models/
│   ├── UserDto.cs
│   ├── TokenResponseDto.cs
│   ├── RefreshTokenRequestDto.cs
│   ├── EmailVerificationDto.cs
│   └── PasswordResetDto.cs
├── Services/
│   ├── IAuthService.cs
│   ├── AuthService.cs
│   ├── IEmailService.cs
│   └── EmailService.cs
├── Validators/
│   ├── UserDtoValidator.cs
│   └── PasswordResetValidator.cs
├── Program.cs
└── appsettings.json
```

## 📝 Configuration appsettings.json

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
        public string Action { get; set; } = string.Empty; // Login, Logout, Register, etc.
        public string IpAddress { get; set; } = string.Empty;
        public string UserAgent { get; set; } = string.Empty;
        public DateTime Timestamp { get; set; } = DateTime.UtcNow;
        public bool Success { get; set; }
        public string? Details { get; set; }
    }
}
```

## 🔄 Entities/RefreshToken.cs (pour Token Rotation)

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
        public string? ReplacedByToken { get; set; } // Pour la rotation
        
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

## 📧 Services/IEmailService.cs

```csharp
namespace YourProjectName.Services
{
    public interface IEmailService
    {
        Task SendEmailVerificationAsync(string email, string username, string token);
        Task SendPasswordResetAsync(string email, string username, string token);
        Task SendLoginNotificationAsync(string email, string username, string ipAddress);
    }
}
```

## 📧 Services/EmailService.cs

```csharp
using MailKit.Net.Smtp;
using MimeKit;

namespace YourProjectName.Services
{
    public class EmailService : IEmailService
    {
        private readonly IConfiguration _configuration;

        public EmailService(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        public async Task SendEmailVerificationAsync(string email, string username, string token)
        {
            var verificationLink = $"https://yourdomain.com/verify-email?token={token}";
            var subject = "Verify Your Email";
            var body = $@"
                <h2>Hello {username},</h2>
                <p>Please verify your email by clicking the link below:</p>
                <a href='{verificationLink}'>Verify Email</a>
                <p>This link will expire in 24 hours.</p>
            ";

            await SendEmailAsync(email, subject, body);
        }

        public async Task SendPasswordResetAsync(string email, string username, string token)
        {
            var resetLink = $"https://yourdomain.com/reset-password?token={token}";
            var subject = "Password Reset Request";
            var body = $@"
                <h2>Hello {username},</h2>
                <p>You requested a password reset. Click the link below:</p>
                <a href='{resetLink}'>Reset Password</a>
                <p>This link will expire in 30 minutes.</p>
            ";

            await SendEmailAsync(email, subject, body);
        }

        public async Task SendLoginNotificationAsync(string email, string username, string ipAddress)
        {
            var subject = "New Login Detected";
            var body = $@"
                <h2>Hello {username},</h2>
                <p>A new login was detected on your account.</p>
                <p>IP Address: {ipAddress}</p>
                <p>Time: {DateTime.UtcNow:yyyy-MM-dd HH:mm:ss} UTC</p>
                <p>If this wasn't you, please reset your password immediately.</p>
            ";

            await SendEmailAsync(email, subject, body);
        }

        private async Task SendEmailAsync(string toEmail, string subject, string body)
        {
            var message = new MimeMessage();
            message.From.Add(new MailboxAddress(
                _configuration["EmailSettings:SenderName"],
                _configuration["EmailSettings:SenderEmail"]
            ));
            message.To.Add(new MailboxAddress("", toEmail));
            message.Subject = subject;

            var bodyBuilder = new BodyBuilder { HtmlBody = body };
            message.Body = bodyBuilder.ToMessageBody();

            using var client = new SmtpClient();
            await client.ConnectAsync(
                _configuration["EmailSettings:SmtpServer"],
                int.Parse(_configuration["EmailSettings:SmtpPort"]!),
                MailKit.Security.SecureSocketOptions.StartTls
            );

            await client.AuthenticateAsync(
                _configuration["EmailSettings:Username"],
                _configuration["EmailSettings:Password"]
            );

            await client.SendAsync(message);
            await client.DisconnectAsync(true);
        }
    }
}
```

## 🔒 Services/IAuthService.cs

```csharp
using YourProjectName.Entities;
using YourProjectName.Models;

namespace YourProjectName.Services
{
    public interface IAuthService
    {
        Task<User?> RegisterAsync(UserDto request, string ipAddress);
        Task<TokenResponseDto?> LoginAsync(UserDto request, string ipAddress, bool rememberMe);
        Task<TokenResponseDto?> RefreshTokensAsync(RefreshTokenRequestDto request, string ipAddress);
        Task<bool> VerifyEmailAsync(string token);
        Task<bool> RequestPasswordResetAsync(string email);
        Task<bool> ResetPasswordAsync(PasswordResetDto request);
        Task<bool> LogoutAsync(Guid userId, string ipAddress);
        Task<bool> RevokeTokenAsync(string token, string ipAddress);
    }
}
```

## 🔒 Services/AuthService.cs (Extrait principal)

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Security.Cryptography;
using System.Text;
using YourProjectName.Data;
using YourProjectName.Entities;
using YourProjectName.Models;

namespace YourProjectName.Services
{
    public class AuthService : IAuthService
    {
        private readonly UserDbContext _context;
        private readonly IConfiguration _configuration;
        private readonly IEmailService _emailService;
        private readonly IHttpContextAccessor _httpContextAccessor;

        public AuthService(
            UserDbContext context,
            IConfiguration configuration,
            IEmailService emailService,
            IHttpContextAccessor httpContextAccessor)
        {
            _context = context;
            _configuration = configuration;
            _emailService = emailService;
            _httpContextAccessor = httpContextAccessor;
        }

        // ==================== REGISTER ====================
        public async Task<User?> RegisterAsync(UserDto request, string ipAddress)
        {
            if (await _context.Users.AnyAsync(u => u.Username == request.Username))
            {
                await LogAuditAsync(null, request.Username, "Register", ipAddress, false, "Username already exists");
                return null;
            }

            if (await _context.Users.AnyAsync(u => u.Email == request.Email))
            {
                await LogAuditAsync(null, request.Username, "Register", ipAddress, false, "Email already exists");
                return null;
            }

            var user = new User
            {
                Username = request.Username,
                Email = request.Email,
                Role = "User"
            };

            var hashedPassword = new PasswordHasher<User>().HashPassword(user, request.Password);
            user.PasswordHash = hashedPassword;

            // Generate email verification token
            user.EmailVerificationToken = GenerateSecureToken();
            user.EmailVerificationTokenExpiry = DateTime.UtcNow.AddHours(24);

            _context.Users.Add(user);
            await _context.SaveChangesAsync();

            // Send verification email
            await _emailService.SendEmailVerificationAsync(user.Email, user.Username, user.EmailVerificationToken);

            await LogAuditAsync(user.Id, user.Username, "Register", ipAddress, true, "User registered successfully");

            return user;
        }

        // ==================== LOGIN ====================
        public async Task<TokenResponseDto?> LoginAsync(UserDto request, string ipAddress, bool rememberMe)
        {
            var user = await _context.Users.FirstOrDefaultAsync(u => u.Username == request.Username);
            
            if (user is null)
            {
                await LogAuditAsync(null, request.Username, "Login", ipAddress, false, "User not found");
                return null;
            }

            // Check lockout
            if (user.LockoutEnd.HasValue && user.LockoutEnd > DateTime.UtcNow)
            {
                await LogAuditAsync(user.Id, user.Username, "Login", ipAddress, false, "Account locked");
                return null;
            }

            // Verify password
            var result = new PasswordHasher<User>().VerifyHashedPassword(user, user.PasswordHash, request.Password);
            
            if (result == PasswordVerificationResult.Failed)
            {
                user.FailedLoginAttempts++;
                
                // Lock account after 5 failed attempts
                if (user.FailedLoginAttempts >= 5)
                {
                    user.LockoutEnd = DateTime.UtcNow.AddMinutes(30);
                }
                
                await _context.SaveChangesAsync();
                await LogAuditAsync(user.Id, user.Username, "Login", ipAddress, false, "Invalid password");
                return null;
            }

            // Check email verification
            if (!user.EmailVerified)
            {
                await LogAuditAsync(user.Id, user.Username, "Login", ipAddress, false, "Email not verified");
                return null;
            }

            // Reset failed attempts
            user.FailedLoginAttempts = 0;
            user.LockoutEnd = null;
            user.LastLoginAt = DateTime.UtcNow;

            // Generate tokens
            var accessToken = CreateAccessToken(user);
            var refreshToken = await GenerateAndSaveRefreshTokenAsync(user, ipAddress);

            // Handle Remember Me
            if (rememberMe)
            {
                user.RememberMeToken = GenerateSecureToken();
                user.RememberMeTokenExpiry = DateTime.UtcNow.AddDays(30);
            }

            await _context.SaveChangesAsync();

            // Send login notification
            await _emailService.SendLoginNotificationAsync(user.Email, user.Username, ipAddress);

            await LogAuditAsync(user.Id, user.Username, "Login", ipAddress, true, "Login successful");

            return new TokenResponseDto
            {
                AccessToken = accessToken,
                RefreshToken = refreshToken
            };
        }

        // ==================== REFRESH TOKENS WITH ROTATION ====================
        public async Task<TokenResponseDto?> RefreshTokensAsync(RefreshTokenRequestDto request, string ipAddress)
        {
            var user = await _context.Users.FindAsync(request.UserId);
            if (user is null)
            {
                return null;
            }

            var refreshToken = await _context.RefreshTokens
                .FirstOrDefaultAsync(rt => rt.Token == request.RefreshToken && rt.UserId == request.UserId);

            if (refreshToken is null || !refreshToken.IsActive)
            {
                await LogAuditAsync(user.Id, user.Username, "RefreshToken", ipAddress, false, "Invalid or expired refresh token");
                return null;
            }

            // Revoke old refresh token
            refreshToken.IsRevoked = true;
            refreshToken.RevokedAt = DateTime.UtcNow;
            refreshToken.RevokedByIp = ipAddress;

            // Generate new refresh token (rotation)
            var newAccessToken = CreateAccessToken(user);
            var newRefreshToken = await GenerateAndSaveRefreshTokenAsync(user, ipAddress);

            // Link old token to new token
            refreshToken.ReplacedByToken = newRefreshToken;

            await _context.SaveChangesAsync();

            await LogAuditAsync(user.Id, user.Username, "RefreshToken", ipAddress, true, "Token refreshed successfully");

            return new TokenResponseDto
            {
                AccessToken = newAccessToken,
                RefreshToken = newRefreshToken
            };
        }

        // ==================== EMAIL VERIFICATION ====================
        public async Task<bool> VerifyEmailAsync(string token)
        {
            var user = await _context.Users
                .FirstOrDefaultAsync(u => u.EmailVerificationToken == token);

            if (user is null || user.EmailVerificationTokenExpiry < DateTime.UtcNow)
            {
                return false;
            }

            user.EmailVerified = true;
            user.EmailVerificationToken = null;
            user.EmailVerificationTokenExpiry = null;

            await _context.SaveChangesAsync();

            var ipAddress = GetIpAddress();
            await LogAuditAsync(user.Id, user.Username, "VerifyEmail", ipAddress, true, "Email verified");

            return true;
        }

        // ==================== PASSWORD RESET REQUEST ====================
        public async Task<bool> RequestPasswordResetAsync(string email)
        {
            var user = await _context.Users.FirstOrDefaultAsync(u => u.Email == email);
            
            if (user is null)
            {
                // Don't reveal if email exists
                return true;
            }

            user.PasswordResetToken = GenerateSecureToken();
            user.PasswordResetTokenExpiry = DateTime.UtcNow.AddMinutes(30);

            await _context.SaveChangesAsync();

            await _emailService.SendPasswordResetAsync(user.Email, user.Username, user.PasswordResetToken);

            var ipAddress = GetIpAddress();
            await LogAuditAsync(user.Id, user.Username, "PasswordResetRequest", ipAddress, true, "Password reset requested");

            return true;
        }

        // ==================== PASSWORD RESET ====================
        public async Task<bool> ResetPasswordAsync(PasswordResetDto request)
        {
            var user = await _context.Users
                .FirstOrDefaultAsync(u => u.PasswordResetToken == request.Token);

            if (user is null || user.PasswordResetTokenExpiry < DateTime.UtcNow)
            {
                return false;
            }

            var hashedPassword = new PasswordHasher<User>().HashPassword(user, request.NewPassword);
            user.PasswordHash = hashedPassword;
            user.PasswordResetToken = null;
            user.PasswordResetTokenExpiry = null;

            // Revoke all refresh tokens (force re-login)
            var refreshTokens = await _context.RefreshTokens
                .Where(rt => rt.UserId == user.Id && rt.IsActive)
                .ToListAsync();

            foreach (var token in refreshTokens)
            {
                token.IsRevoked = true;
                token.RevokedAt = DateTime.UtcNow;
            }

            await _context.SaveChangesAsync();

            var ipAddress = GetIpAddress();
            await LogAuditAsync(user.Id, user.Username, "PasswordReset", ipAddress, true, "Password reset successfully");

            return true;
        }

        // ==================== LOGOUT ====================
        public async Task<bool> LogoutAsync(Guid userId, string ipAddress)
        {
            var user = await _context.Users.FindAsync(userId);
            if (user is null)
            {
                return false;
            }

            // Revoke all active refresh tokens
            var refreshTokens = await _context.RefreshTokens
                .Where(rt => rt.UserId == userId && rt.IsActive)
                .ToListAsync();

            foreach (var token in refreshTokens)
            {
                token.IsRevoked = true;
                token.RevokedAt = DateTime.UtcNow;
                token.RevokedByIp = ipAddress;
            }

            await _context.SaveChangesAsync();

            await LogAuditAsync(userId, user.Username, "Logout", ipAddress, true, "User logged out");

            return true;
        }

        // ==================== REVOKE TOKEN ====================
        public async Task<bool> RevokeTokenAsync(string token, string ipAddress)
        {
            var refreshToken = await _context.RefreshTokens
                .Include(rt => rt.User)
                .FirstOrDefaultAsync(rt => rt.Token == token);

            if (refreshToken is null || !refreshToken.IsActive)
            {
                return false;
            }

            refreshToken.IsRevoked = true;
            refreshToken.RevokedAt = DateTime.UtcNow;
            refreshToken.RevokedByIp = ipAddress;

            await _context.SaveChangesAsync();

            await LogAuditAsync(
                refreshToken.UserId,
                refreshToken.User?.Username ?? "Unknown",
                "RevokeToken",
                ipAddress,
                true,
                "Token revoked"
            );

            return true;
        }

        // ==================== PRIVATE HELPER METHODS ====================

        private string CreateAccessToken(User user)
        {
            var claims = new List<Claim>
            {
                new Claim(ClaimTypes.Name, user.Username),
                new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
                new Claim(ClaimTypes.Email, user.Email),
                new Claim(ClaimTypes.Role, user.Role)
            };

            var key = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(_configuration["AppSettings:Token"]!)
            );

            var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha512Signature);

            var tokenDescriptor = new JwtSecurityToken(
                issuer: _configuration["AppSettings:Issuer"],
                audience: _configuration["AppSettings:Audience"],
                claims: claims,
                expires: DateTime.UtcNow.AddMinutes(
                    int.Parse(_configuration["AppSettings:AccessTokenExpirationMinutes"]!)
                ),
                signingCredentials: creds
            );

            return new JwtSecurityTokenHandler().WriteToken(tokenDescriptor);
        }

        private async Task<string> GenerateAndSaveRefreshTokenAsync(User user, string ipAddress)
        {
            var refreshToken = new RefreshTokenEntity
            {
                UserId = user.Id,
                Token = GenerateSecureToken(),
                ExpiresAt = DateTime.UtcNow.AddDays(
                    int.Parse(_configuration["AppSettings:RefreshTokenExpirationDays"]!)
                ),
                CreatedByIp = ipAddress
            };

            _context.RefreshTokens.Add(refreshToken);
            await _context.SaveChangesAsync();

            return refreshToken.Token;
        }

        private string GenerateSecureToken()
        {
            var randomNumber = new byte[64];
            using var rng = RandomNumberGenerator.Create();
            rng.GetBytes(randomNumber);
            return Convert.ToBase64String(randomNumber);
        }

        private async Task LogAuditAsync(Guid? userId, string username, string action, string ipAddress, bool success, string? details)
        {
            var userAgent = _httpContextAccessor.HttpContext?.Request.Headers["User-Agent"].ToString() ?? "Unknown";

            var auditLog = new AuditLog
            {
                UserId = userId,
                Username = username,
                Action = action,
                IpAddress = ipAddress,
                UserAgent = userAgent,
                Success = success,
                Details = details
            };

            _context.AuditLogs.Add(auditLog);
            await _context.SaveChangesAsync();
        }

        private string GetIpAddress()
        {
            return _httpContextAccessor.HttpContext?.Connection.RemoteIpAddress?.ToString() ?? "Unknown";
        }
    }
}
```

---

## 🎮 Controllers/AuthController.cs (Complet)

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using YourProjectName.Entities;
using YourProjectName.Models;
using YourProjectName.Services;

namespace YourProjectName.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class AuthController : ControllerBase
    {
        private readonly IAuthService _authService;

        public AuthController(IAuthService authService)
        {
            _authService = authService;
        }

        // POST: api/auth/register
        [HttpPost("register")]
        public async Task<ActionResult<User>> Register(UserDto request)
        {
            var ipAddress = GetIpAddress();
            var user = await _authService.RegisterAsync(request, ipAddress);
            
            if (user is null)
            {
                return BadRequest(new { message = "Username or email already exists." });
            }

            return Ok(new
            {
                message = "Registration successful. Please check your email to verify your account.",
                userId = user.Id
            });
        }

        // POST: api/auth/login
        [HttpPost("login")]
        public async Task<ActionResult<TokenResponseDto>> Login(LoginDto request)
        {
            var ipAddress = GetIpAddress();
            var result = await _authService.LoginAsync(
                new UserDto { Username = request.Username, Password = request.Password, Email = string.Empty },
                ipAddress,
                request.RememberMe
            );

            if (result is null)
            {
                return BadRequest(new { message = "Invalid credentials or email not verified." });
            }

            return Ok(result);
        }

        // POST: api/auth/refresh-tokens
        [HttpPost("refresh-tokens")]
        public async Task<ActionResult<TokenResponseDto>> RefreshTokens(RefreshTokenRequestDto request)
        {
            var ipAddress = GetIpAddress();
            var result = await _authService.RefreshTokensAsync(request, ipAddress);

            if (result is null)
            {
                return BadRequest(new { message = "Invalid or expired refresh token." });
            }

            return Ok(result);
        }

        // GET: api/auth/verify-email?token=xxx
        [HttpGet("verify-email")]
        public async Task<IActionResult> VerifyEmail([FromQuery] string token)
        {
            var result = await _authService.VerifyEmailAsync(token);

            if (!result)
            {
                return BadRequest(new { message = "Invalid or expired verification token." });
            }

            return Ok(new { message = "Email verified successfully. You can now login." });
        }

        // POST: api/auth/request-password-reset
        [HttpPost("request-password-reset")]
        public async Task<IActionResult> RequestPasswordReset(PasswordResetRequestDto request)
        {
            await _authService.RequestPasswordResetAsync(request.Email);

            return Ok(new { message = "If the email exists, a password reset link has been sent." });
        }

        // POST: api/auth/reset-password
        [HttpPost("reset-password")]
        public async Task<IActionResult> ResetPassword(PasswordResetDto request)
        {
            var result = await _authService.ResetPasswordAsync(request);

            if (!result)
            {
                return BadRequest(new { message = "Invalid or expired reset token." });
            }

            return Ok(new { message = "Password reset successfully. Please login with your new password." });
        }

        // POST: api/auth/logout
        [Authorize]
        [HttpPost("logout")]
        public async Task<IActionResult> Logout()
        {
            var userId = Guid.Parse(User.FindFirst(System.Security.Claims.ClaimTypes.NameIdentifier)?.Value!);
            var ipAddress = GetIpAddress();

            var result = await _authService.LogoutAsync(userId, ipAddress);

            if (!result)
            {
                return BadRequest(new { message = "Logout failed." });
            }

            return Ok(new { message = "Logged out successfully." });
        }

        // POST: api/auth/revoke-token
        [Authorize]
        [HttpPost("revoke-token")]
        public async Task<IActionResult> RevokeToken(RevokeTokenDto request)
        {
            var ipAddress = GetIpAddress();
            var result = await _authService.RevokeTokenAsync(request.Token, ipAddress);

            if (!result)
            {
                return BadRequest(new { message = "Token revocation failed." });
            }

            return Ok(new { message = "Token revoked successfully." });
        }

        // GET: api/auth (Protected endpoint example)
        [Authorize]
        [HttpGet]
        public IActionResult AuthenticatedOnlyEndpoint()
        {
            var username = User?.Identity?.Name;
            return Ok(new { message = $"Hello, {username}. You are authenticated." });
        }

        // GET: api/auth/admin-only (Admin-only endpoint)
        [Authorize(Roles = "Admin")]
        [HttpGet("admin-only")]
        public IActionResult AdminOnlyEndpoint()
        {
            var username = User?.Identity?.Name;
            return Ok(new { message = $"Hello, {username}. You are an admin." });
        }

        // Helper method
        private string GetIpAddress()
        {
            return HttpContext.Connection.RemoteIpAddress?.ToString() ?? "Unknown";
        }
    }
}
```

---

## 📋 Models/LoginDto.cs

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

## 📋 Models/PasswordResetDto.cs

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

## 📋 Models/RevokeTokenDto.cs

```csharp
namespace YourProjectName.Models
{
    public class RevokeTokenDto
    {
        public string Token { get; set; } = string.Empty;
    }
}
```

---

## 🔧 Program.cs (Configuration Complète avec JWT)

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

// ⚠️ ORDER IS CRITICAL ⚠️
app.UseIpRateLimiting();
app.UseCors("AllowFrontend");
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
app.MapHealthChecks("/health");

app.Run();
```

---

# 15. Advanced Security Features

## 📝 Checklist des Fonctionnalités Implémentées

### ✅ Fonctionnalités de Base
- [x] **Registration** - Inscription avec validation
- [x] **Login** - Connexion avec JWT
- [x] **Access Token** (15 minutes d'expiration)
- [x] **Refresh Token** (7 jours d'expiration)
- [x] **Password Hashing** (avec PasswordHasher)

### ✅ Fonctionnalités Avancées
- [x] **Email Verification** - Vérification email obligatoire
- [x] **Password Reset** - Réinitialisation mot de passe par email
- [x] **Remember Me** - Token longue durée (30 jours)
- [x] **Logout** - Révocation des tokens actifs
- [x] **Refresh Token Rotation** - Rotation automatique des refresh tokens
- [x] **Audit Logs** - Logs de toutes les actions (qui, quand, quoi, IP)
- [x] **Rate Limiting** - Protection contre brute force
- [x] **Account Lockout** - Verrouillage après 5 tentatives échouées
- [x] **CORS** - Configuration pour frontend
- [x] **Health Checks** - Vérification état de l'API

### 🔜 Fonctionnalités Optionnelles (À Implémenter)
- [ ] **2FA (Two-Factor Authentication)** - Authentification à deux facteurs
- [ ] **Social Login** (Google, Facebook, etc.)
- [ ] **IP Whitelist/Blacklist**
- [ ] **Device Management** - Gestion des appareils connectés

---

## 🔐 2FA (Two-Factor Authentication) - Guide d'Implémentation

### Packages Requis
```bash
dotnet add package OtpNet
dotnet add package QRCoder
```

### Entité User (Déjà ajouté)
```csharp
public bool TwoFactorEnabled { get; set; } = false;
public string? TwoFactorSecret { get; set; }
```

### Service 2FA
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

## 📊 Utilisation des Audit Logs

### Requêtes Utiles

```csharp
// Voir toutes les actions d'un utilisateur
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

// Voir les tentatives de connexion échouées (Admin)
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

// Voir les connexions par IP suspecte
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

## 🔄 Refresh Token Rotation - Comment ça marche ?

### Principe
1. **Client** envoie Refresh Token
2. **API** vérifie le token
3. **API** révoque l'ancien token
4. **API** génère NOUVEAU Access Token + NOUVEAU Refresh Token
5. **API** lie l'ancien token au nouveau (pour audit)
6. **Client** reçoit les nouveaux tokens

### Avantages
- ✅ Si un Refresh Token est volé, il devient invalide dès la prochaine rotation
- ✅ Traçabilité complète dans la table `RefreshTokens`
- ✅ Détection des tokens compromis (si ancien token utilisé après rotation)

### Détection de Token Compromis
```csharp
// Add to AuthService
private async Task<bool> IsTokenCompromisedAsync(string token)
{
    var refreshToken = await _context.RefreshTokens
        .FirstOrDefaultAsync(rt => rt.Token == token);

    // Si le token a été révoqué ET a un replacedByToken, c'est suspect
    if (refreshToken is not null && 
        refreshToken.IsRevoked && 
        !string.IsNullOrEmpty(refreshToken.ReplacedByToken))
    {
        // Quelqu'un essaie d'utiliser un ancien token
        // Action: Révoquer TOUTE la chaîne de tokens
        await RevokeDescendantRefreshTokensAsync(refreshToken);
        return true;
    }

    return false;
}

private async Task RevokeDescendantRefreshTokensAsync(RefreshTokenEntity refreshToken)
{
    // Révoquer récursivement tous les tokens descendants
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

## 🧪 Tests API - Exemples Complets

### test-auth.http
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

## 🎯 Migration Commands

```bash
# Create migration for User, AuditLog, RefreshToken
dotnet ef migrations add AddAuthEntities

# Apply migration
dotnet ef database update

# If you need to add 2FA later
dotnet ef migrations add Add2FASupport
dotnet ef database update
```

---

## 📋 Checklist Finale d'Implémentation

### Étapes à Suivre

- [ ] 1. Installer tous les packages NuGet requis
- [ ] 2. Créer les entités (User, AuditLog, RefreshTokenEntity)
- [ ] 3. Créer le DbContext avec les DbSets
- [ ] 4. Configurer appsettings.json
- [ ] 5. Créer les Models/DTOs
- [ ] 6. Créer les Validators (FluentValidation)
- [ ] 7. Implémenter IEmailService et EmailService
- [ ] 8. Implémenter IAuthService et AuthService
- [ ] 9. Créer AuthController
- [ ] 10. Configurer Program.cs (JWT, CORS, Rate Limiting, etc.)
- [ ] 11. Créer et appliquer les migrations
- [ ] 12. Tester avec le fichier .http
- [ ] 13. Vérifier les logs d'audit dans la BD
- [ ] 14. Tester le rate limiting
- [ ] 15. Configurer le service email (Gmail App Password)

---

## 🔐 Configuration Gmail pour l'Email

### Étapes pour obtenir un App Password Gmail

1. Aller sur [Google Account](https://myaccount.google.com/)
2. Sécurité → Validation en deux étapes (activer si pas déjà fait)
3. Mots de passe des applications
4. Sélectionner "Application" → Autre → "YourAppName"
5. Copier le mot de passe généré (16 caractères)
6. Mettre dans appsettings.json :

```json
"EmailSettings": {
  "SmtpServer": "smtp.gmail.com",
  "SmtpPort": 587,
  "SenderEmail": "your-email@gmail.com",
  "SenderName": "Your App",
  "Username": "your-email@gmail.com",
  "Password": "xxxx xxxx xxxx xxxx"  // App password
}
```

---

## 🚀 Prochaines Étapes (Fonctionnalités Avancées)

### 1. Implémenter 2FA (Two-Factor Authentication)
- Package : `OtpNet`, `QRCoder`
- Endpoints : `/enable-2fa`, `/verify-2fa`, `/disable-2fa`

### 2. Social Login (Google, Facebook)
- Package : `Microsoft.AspNetCore.Authentication.Google`
- Configuration OAuth

### 3. Device Management
- Table `UserDevices` avec fingerprint du navigateur
- Notification lors de connexion depuis nouvel appareil

### 4. IP Whitelist/Blacklist
- Table `AllowedIPs` et `BlockedIPs`
- Middleware de vérification IP

### 5. Session Management
- Voir tous les appareils connectés
- Déconnecter des appareils spécifiques

---

## 📚 Ressources Utiles

- [JWT.io](https://jwt.io/) - Décoder et vérifier les JWT
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Microsoft Identity Documentation](https://learn.microsoft.com/en-us/aspnet/core/security/)
- [FluentValidation Docs](https://docs.fluentvalidation.net/)

---

**Last Updated:** October 2024 | **Framework:** .NET 9.0 | **EF Core:** 9.0.9

**✨ Félicitations ! Tu as maintenant une API ASP.NET Core complète avec authentification JWT et toutes les fonctionnalités de sécurité modernes !** 🎉
