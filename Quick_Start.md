# 🚀 ASP.NET Core Web API - Quick Start Guide

## 📋 Table of Contents
1. [Prerequisites](#prerequisites)
2. [Project Setup](#project-setup)
3. [NuGet Packages Installation](#nuget-packages-installation)
4. [Project Structure](#project-structure)
5. [Step-by-Step Implementation](#step-by-step-implementation)
6. [Authentication & Authorization](#authentication--authorization)
7. [Testing Your API](#testing-your-api)
8. [Common Commands](#common-commands)

---

## Prerequisites

✅ **Required Tools:**
- Visual Studio 2022 (or VS Code)
- .NET 9.0 SDK
- SQL Server (Express or full version)
- SQL Server Management Studio (SSMS) - Optional

---

## Project Setup

### 🎯 Option 1: Visual Studio 2022

1. Open Visual Studio 2022
2. Click **"Create a new project"**
3. Search for **"ASP.NET Core Web API"**
4. Click **Next**
5. Project name: `YourProjectName`
6. Location: Choose your folder
7. Click **Next**
8. Framework: **.NET 9.0**
9. Authentication type: **None** (we'll add JWT manually)
10. Configure for HTTPS: ✅ **Checked**
11. Enable OpenAPI support: ✅ **Checked**
12. Click **Create**

### 🎯 Option 2: Visual Studio Code / Terminal

```bash
# Create new Web API project
dotnet new webapi -n YourProjectName --framework net9.0

# Navigate to project folder
cd YourProjectName

# Open in VS Code
code .
```

---

## NuGet Packages Installation

### 📦 Option 1: Visual Studio 2022 (GUI)

1. Right-click on your project in **Solution Explorer**
2. Click **"Manage NuGet Packages..."**
3. Click **"Browse"** tab
4. Search and install each package below:

**Essential Packages:**
```
✅ Microsoft.EntityFrameworkCore (9.0.10)
✅ Microsoft.EntityFrameworkCore.SqlServer (9.0.10)
✅ Microsoft.EntityFrameworkCore.Tools (9.0.10)
✅ Microsoft.AspNetCore.OpenApi (9.0.0)
✅ Scalar.AspNetCore (Latest)
```

**JWT Authentication Packages:**
```
✅ Microsoft.AspNetCore.Authentication.JwtBearer (9.0.0)
✅ Microsoft.AspNetCore.Identity.EntityFrameworkCore (9.0.0)
✅ System.IdentityModel.Tokens.Jwt (Latest)
```

**Validation & Security:**
```
✅ FluentValidation (Latest)
✅ FluentValidation.AspNetCore (Latest)
✅ AspNetCoreRateLimit (Latest)
```

**Optional - Email Support:**
```
✅ MailKit (Latest)
✅ MimeKit (Latest)
```

### 📦 Option 2: Package Manager Console (Visual Studio)

```powershell
# Essential Packages
Install-Package Microsoft.EntityFrameworkCore -Version 9.0.10
Install-Package Microsoft.EntityFrameworkCore.SqlServer -Version 9.0.10
Install-Package Microsoft.EntityFrameworkCore.Tools -Version 9.0.10
Install-Package Microsoft.AspNetCore.OpenApi -Version 9.0.0
Install-Package Scalar.AspNetCore

# JWT Authentication
Install-Package Microsoft.AspNetCore.Authentication.JwtBearer -Version 9.0.0
Install-Package Microsoft.AspNetCore.Identity.EntityFrameworkCore -Version 9.0.0
Install-Package System.IdentityModel.Tokens.Jwt

# Validation & Security
Install-Package FluentValidation
Install-Package FluentValidation.AspNetCore
Install-Package AspNetCoreRateLimit

# Optional - Email
Install-Package MailKit
Install-Package MimeKit
```

### 📦 Option 3: Terminal / VS Code

```bash
# Essential Packages
dotnet add package Microsoft.EntityFrameworkCore --version 9.0.10
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 9.0.10
dotnet add package Microsoft.EntityFrameworkCore.Tools --version 9.0.10
dotnet add package Microsoft.AspNetCore.OpenApi --version 9.0.0
dotnet add package Scalar.AspNetCore

# JWT Authentication
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer --version 9.0.0
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore --version 9.0.0
dotnet add package System.IdentityModel.Tokens.Jwt

# Validation & Security
dotnet add package FluentValidation
dotnet add package FluentValidation.AspNetCore
dotnet add package AspNetCoreRateLimit

# Optional - Email
dotnet add package MailKit
dotnet add package MimeKit
```

---

## Project Structure

Create this folder structure in your project:

```
YourProjectName/
├── Controllers/
│   ├── AuthController.cs
│   ├── ProductController.cs (example)
│   └── VideoGameController.cs (example)
├── Data/
│   ├── AppDbContext.cs
│   └── UserDbContext.cs
├── Entities/
│   ├── User.cs
│   ├── AuditLog.cs
│   └── RefreshToken.cs
├── Models/
│   ├── Product.cs (example)
│   ├── VideoGame.cs (example)
│   ├── UserDto.cs
│   ├── LoginDto.cs
│   ├── TokenResponseDto.cs
│   └── RefreshTokenRequestDto.cs
├── Services/
│   ├── IAuthService.cs
│   ├── AuthService.cs
│   ├── IEmailService.cs (optional)
│   └── EmailService.cs (optional)
├── Validators/
│   └── UserDtoValidator.cs
├── Migrations/
│   └── (auto-generated)
├── Program.cs
├── appsettings.json
└── YourProjectName.csproj
```

---

## Step-by-Step Implementation

### 🔥 PHASE 1: Database Configuration (5 minutes)

#### 1️⃣ Configure `appsettings.json`

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER_NAME;Database=YourDatabaseName;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "AppSettings": {
    "Token": "YOUR_SUPER_SECRET_KEY_AT_LEAST_32_CHARACTERS_LONG_FOR_SECURITY!!!",
    "Issuer": "YourAppName",
    "Audience": "YourAppAudience",
    "AccessTokenExpirationMinutes": 15,
    "RefreshTokenExpirationDays": 7
  },
  "IpRateLimiting": {
    "EnableEndpointRateLimiting": true,
    "StackBlockedRequests": false,
    "HttpStatusCode": 429
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

**🔍 Find Your SQL Server Name:**

**Option A: SQL Server Management Studio (SSMS)**
- Open SSMS
- The server name appears in the connection dialog
- Common formats:
  - `localhost` or `.`
  - `.\SQLEXPRESS`
  - `YOUR_COMPUTER_NAME\SQLEXPRESS`

**Option B: Visual Studio**
- View → SQL Server Object Explorer
- Server name appears at the top

**Option C: Command Line**
```bash
# PowerShell
sqlcmd -L

# This lists all SQL Server instances
```

---

### 🔥 PHASE 2: Create Entities (10 minutes)

#### 2️⃣ Create `Entities/User.cs`

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
        
        // Account Info
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
        public DateTime? LastLoginAt { get; set; }
        public bool IsActive { get; set; } = true;
        public int FailedLoginAttempts { get; set; } = 0;
        public DateTime? LockoutEnd { get; set; }
    }
}
```

#### 3️⃣ Create `Entities/AuditLog.cs`

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

#### 4️⃣ Create `Entities/RefreshToken.cs`

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
        
        public User? User { get; set; }
        
        public bool IsExpired => DateTime.UtcNow >= ExpiresAt;
        public bool IsActive => !IsRevoked && !IsExpired;
    }
}
```

---

### 🔥 PHASE 3: Create Models (DTOs) (5 minutes)

#### 5️⃣ Create `Models/UserDto.cs`

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

#### 6️⃣ Create `Models/LoginDto.cs`

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

#### 7️⃣ Create `Models/TokenResponseDto.cs`

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

#### 8️⃣ Create `Models/RefreshTokenRequestDto.cs`

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

---

### 🔥 PHASE 4: Create DbContext (5 minutes)

#### 9️⃣ Create `Data/UserDbContext.cs`

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

---

### 🔥 PHASE 5: Create Validators (5 minutes)

#### 🔟 Create `Validators/UserDtoValidator.cs`

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

---

### 🔥 PHASE 6: Create Services (20 minutes)

#### 1️⃣1️⃣ Create `Services/IAuthService.cs`

```csharp
using YourProjectName.Entities;
using YourProjectName.Models;

namespace YourProjectName.Services
{
    public interface IAuthService
    {
        Task<User?> RegisterAsync(UserDto request, string ipAddress);
        Task<TokenResponseDto?> LoginAsync(LoginDto request, string ipAddress);
        Task<TokenResponseDto?> RefreshTokensAsync(RefreshTokenRequestDto request, string ipAddress);
        Task<bool> LogoutAsync(Guid userId, string ipAddress);
    }
}
```

#### 1️⃣2️⃣ Create `Services/AuthService.cs`

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
        private readonly IHttpContextAccessor _httpContextAccessor;

        public AuthService(
            UserDbContext context,
            IConfiguration configuration,
            IHttpContextAccessor httpContextAccessor)
        {
            _context = context;
            _configuration = configuration;
            _httpContextAccessor = httpContextAccessor;
        }

        public async Task<User?> RegisterAsync(UserDto request, string ipAddress)
        {
            if (await _context.Users.AnyAsync(u => u.Username == request.Username))
            {
                await LogAuditAsync(null, request.Username, "Register", ipAddress, false, "Username exists");
                return null;
            }

            if (await _context.Users.AnyAsync(u => u.Email == request.Email))
            {
                await LogAuditAsync(null, request.Username, "Register", ipAddress, false, "Email exists");
                return null;
            }

            var user = new User
            {
                Username = request.Username,
                Email = request.Email,
                Role = "User",
                EmailVerified = true // Set to false if you want email verification
            };

            var hashedPassword = new PasswordHasher<User>().HashPassword(user, request.Password);
            user.PasswordHash = hashedPassword;

            _context.Users.Add(user);
            await _context.SaveChangesAsync();

            await LogAuditAsync(user.Id, user.Username, "Register", ipAddress, true, "User registered");

            return user;
        }

        public async Task<TokenResponseDto?> LoginAsync(LoginDto request, string ipAddress)
        {
            var user = await _context.Users.FirstOrDefaultAsync(u => u.Username == request.Username);
            
            if (user is null)
            {
                await LogAuditAsync(null, request.Username, "Login", ipAddress, false, "User not found");
                return null;
            }

            if (user.LockoutEnd.HasValue && user.LockoutEnd > DateTime.UtcNow)
            {
                await LogAuditAsync(user.Id, user.Username, "Login", ipAddress, false, "Account locked");
                return null;
            }

            var result = new PasswordHasher<User>().VerifyHashedPassword(user, user.PasswordHash, request.Password);
            
            if (result == PasswordVerificationResult.Failed)
            {
                user.FailedLoginAttempts++;
                
                if (user.FailedLoginAttempts >= 5)
                {
                    user.LockoutEnd = DateTime.UtcNow.AddMinutes(30);
                }
                
                await _context.SaveChangesAsync();
                await LogAuditAsync(user.Id, user.Username, "Login", ipAddress, false, "Invalid password");
                return null;
            }

            user.FailedLoginAttempts = 0;
            user.LockoutEnd = null;
            user.LastLoginAt = DateTime.UtcNow;

            var accessToken = CreateAccessToken(user);
            var refreshToken = await GenerateAndSaveRefreshTokenAsync(user, ipAddress);

            await _context.SaveChangesAsync();
            await LogAuditAsync(user.Id, user.Username, "Login", ipAddress, true, "Login successful");

            return new TokenResponseDto
            {
                AccessToken = accessToken,
                RefreshToken = refreshToken
            };
        }

        public async Task<TokenResponseDto?> RefreshTokensAsync(RefreshTokenRequestDto request, string ipAddress)
        {
            var user = await _context.Users.FindAsync(request.UserId);
            if (user is null) return null;

            var refreshToken = await _context.RefreshTokens
                .FirstOrDefaultAsync(rt => rt.Token == request.RefreshToken && rt.UserId == request.UserId);

            if (refreshToken is null || !refreshToken.IsActive)
            {
                await LogAuditAsync(user.Id, user.Username, "RefreshToken", ipAddress, false, "Invalid token");
                return null;
            }

            refreshToken.IsRevoked = true;
            refreshToken.RevokedAt = DateTime.UtcNow;
            refreshToken.RevokedByIp = ipAddress;

            var newAccessToken = CreateAccessToken(user);
            var newRefreshToken = await GenerateAndSaveRefreshTokenAsync(user, ipAddress);

            refreshToken.ReplacedByToken = newRefreshToken;

            await _context.SaveChangesAsync();
            await LogAuditAsync(user.Id, user.Username, "RefreshToken", ipAddress, true, "Token refreshed");

            return new TokenResponseDto
            {
                AccessToken = newAccessToken,
                RefreshToken = newRefreshToken
            };
        }

        public async Task<bool> LogoutAsync(Guid userId, string ipAddress)
        {
            var user = await _context.Users.FindAsync(userId);
            if (user is null) return false;

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
    }
}
```

---

### 🔥 PHASE 7: Create Controller (10 minutes)

#### 1️⃣3️⃣ Create `Controllers/AuthController.cs`

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
                message = "Registration successful.",
                userId = user.Id
            });
        }

        [HttpPost("login")]
        public async Task<ActionResult<TokenResponseDto>> Login(LoginDto request)
        {
            var ipAddress = GetIpAddress();
            var result = await _authService.LoginAsync(request, ipAddress);

            if (result is null)
            {
                return BadRequest(new { message = "Invalid credentials." });
            }

            return Ok(result);
        }

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

        [Authorize]
        [HttpGet("me")]
        public IActionResult GetCurrentUser()
        {
            var username = User?.Identity?.Name;
            var role = User?.FindFirst(System.Security.Claims.ClaimTypes.Role)?.Value;
            return Ok(new { username, role });
        }

        [Authorize(Roles = "Admin")]
        [HttpGet("admin-only")]
        public IActionResult AdminOnlyEndpoint()
        {
            var username = User?.Identity?.Name;
            return Ok(new { message = $"Hello, {username}. You are an admin." });
        }

        private string GetIpAddress()
        {
            return HttpContext.Connection.RemoteIpAddress?.ToString() ?? "Unknown";
        }
    }
}
```

---

### 🔥 PHASE 8: Configure Program.cs (10 minutes)

#### 1️⃣4️⃣ Update `Program.cs`

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

// Controllers
builder.Services.AddControllers();

// Validation
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssemblyContaining<Program>();

// OpenAPI
builder.Services.AddOpenApi();

// Database
builder.Services.AddDbContext<UserDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"));
});

// HTTP Context Accessor
builder.Services.AddHttpContextAccessor();

// Rate Limiting
builder.Services.AddMemoryCache();
builder.Services.Configure<IpRateLimitOptions>(options =>
{
    options.GeneralRules = new List<RateLimitRule>
    {
        new RateLimitRule { Endpoint = "*", Period = "1m", Limit = 60 },
        new RateLimitRule { Endpoint = "*/auth/login", Period = "1m", Limit = 5 },
        new RateLimitRule { Endpoint = "*/auth/register", Period = "1h", Limit = 3 }
    };
    
    options.QuotaExceededResponse = new QuotaExceededResponse
    {
        Content = "{{ \"message\": \"Too many requests. Please try again later.\" }}",
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

// CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:3000", "http://localhost:5173")
            .AllowAnyHeader()
            .AllowAnyMethod()
            .AllowCredentials();
    });
});

// JWT Authentication
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

// Health Checks
builder.Services.AddHealthChecks()
    .AddDbContextCheck<UserDbContext>("database");

// Services
builder.Services.AddScoped<IAuthService, AuthService>();

var app = builder.Build();

// Middleware Pipeline
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference();
}

```csharp
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

### 🔥 PHASE 9: Database Migration (5 minutes)

#### 1️⃣5️⃣ Create and Apply Migration

**Visual Studio 2022 - Package Manager Console:**

```powershell
# Open: Tools → NuGet Package Manager → Package Manager Console

# Create migration
Add-Migration InitialAuthSetup

# Apply to database
Update-Database
```

**VS Code / Terminal:**

```bash
# Create migration
dotnet ef migrations add InitialAuthSetup

# Apply to database
dotnet ef database update
```

**✅ Success Indicators:**
- You should see output: "Done. To undo this action, use Remove-Migration"
- In SQL Server, you'll see new tables: `Users`, `AuditLogs`, `RefreshTokens`

---

## Authentication & Authorization

### 🔐 How JWT Works in Your API

```
┌─────────────┐
│   CLIENT    │
└──────┬──────┘
       │
       │ 1. POST /api/auth/register
       │    { username, email, password }
       ▼
┌──────────────────────────────┐
│  API - Register              │
│  • Hash password             │
│  • Save user to database     │
│  • Log audit                 │
└──────┬───────────────────────┘
       │
       │ 2. POST /api/auth/login
       │    { username, password }
       ▼
┌──────────────────────────────┐
│  API - Login                 │
│  • Verify credentials        │
│  • Generate Access Token     │
│  • Generate Refresh Token    │
│  • Return both tokens        │
└──────┬───────────────────────┘
       │
       │ Returns: 
       │ { accessToken: "...", refreshToken: "..." }
       ▼
┌─────────────┐
│   CLIENT    │  Stores tokens
└──────┬──────┘
       │
       │ 3. GET /api/auth/me
       │    Authorization: Bearer {accessToken}
       ▼
┌──────────────────────────────┐
│  API - Protected Endpoint    │
│  • Validate JWT              │
│  • Check expiration          │
│  • Extract user claims       │
└──────┬───────────────────────┘
       │
       │ Returns user data
       ▼
┌─────────────┐
│   CLIENT    │
└─────────────┘
```

### 🎯 Using Authorization Attributes

```csharp
// Anyone can access (no attribute)
[HttpGet("public")]
public IActionResult PublicEndpoint()
{
    return Ok("This is public");
}

// Requires authentication (any logged-in user)
[Authorize]
[HttpGet("protected")]
public IActionResult ProtectedEndpoint()
{
    return Ok("You are authenticated");
}

// Requires specific role
[Authorize(Roles = "Admin")]
[HttpGet("admin")]
public IActionResult AdminOnlyEndpoint()
{
    return Ok("You are an admin");
}

// Requires multiple roles (OR logic)
[Authorize(Roles = "Admin,Moderator")]
[HttpGet("staff")]
public IActionResult StaffEndpoint()
{
    return Ok("You are staff");
}
```

---

## Testing Your API

### 🧪 Create Test File

Create `test-auth.http` in your project root:

```http
@baseUrl = https://localhost:7020
@accessToken = 
@refreshToken = 
@userId = 

### 1. Health Check
GET {{baseUrl}}/health

### 2. Register User
POST {{baseUrl}}/api/auth/register
Content-Type: application/json

{
  "username": "testuser",
  "email": "test@example.com",
  "password": "Test@1234"
}

### 3. Login
POST {{baseUrl}}/api/auth/login
Content-Type: application/json

{
  "username": "testuser",
  "password": "Test@1234",
  "rememberMe": false
}

### 4. Get Current User (Protected)
GET {{baseUrl}}/api/auth/me
Authorization: Bearer {{accessToken}}

### 5. Refresh Tokens
POST {{baseUrl}}/api/auth/refresh-tokens
Content-Type: application/json

{
  "userId": "{{userId}}",
  "refreshToken": "{{refreshToken}}"
}

### 6. Logout
POST {{baseUrl}}/api/auth/logout
Authorization: Bearer {{accessToken}}

### 7. Admin Only Endpoint (will fail if not admin)
GET {{baseUrl}}/api/auth/admin-only
Authorization: Bearer {{accessToken}}
```

### 🚀 Testing Steps

#### Step 1: Run Your API

**Visual Studio 2022:**
- Press `F5` or click "Run"
- API starts at `https://localhost:7020`

**VS Code / Terminal:**
```bash
dotnet run
```

#### Step 2: Access Scalar API Documentation

Open browser: `https://localhost:7020/scalar/v1`

You'll see interactive API documentation with all endpoints.

#### Step 3: Test Register

**Using Scalar UI:**
1. Find `POST /api/auth/register`
2. Click "Try it out"
3. Enter test data:
```json
{
  "username": "testuser",
  "email": "test@example.com",
  "password": "Test@1234"
}
```
4. Click "Execute"

**Expected Response (200 OK):**
```json
{
  "message": "Registration successful.",
  "userId": "guid-here"
}
```

#### Step 4: Test Login

**Using Scalar UI:**
1. Find `POST /api/auth/login`
2. Click "Try it out"
3. Enter credentials:
```json
{
  "username": "testuser",
  "password": "Test@1234",
  "rememberMe": false
}
```
4. Click "Execute"

**Expected Response (200 OK):**
```json
{
  "accessToken": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "base64-encoded-string..."
}
```

**⭐ COPY THE ACCESS TOKEN!** You'll need it for protected endpoints.

#### Step 5: Test Protected Endpoint

1. Find `GET /api/auth/me`
2. Click "Try it out"
3. Click the 🔒 lock icon or "Authorize" button
4. Enter: `Bearer YOUR_ACCESS_TOKEN_HERE`
5. Click "Authorize"
6. Click "Execute"

**Expected Response (200 OK):**
```json
{
  "username": "testuser",
  "role": "User"
}
```

---

## Common Commands

### 📦 Entity Framework Commands

#### Visual Studio - Package Manager Console

```powershell
# Create migration
Add-Migration MigrationName

# Apply migrations
Update-Database

# Remove last migration (if not applied)
Remove-Migration

# View SQL script
Script-Migration

# Drop database
Drop-Database

# List all migrations
Get-Migration

# Update to specific migration
Update-Database MigrationName
```

#### VS Code / Terminal

```bash
# Create migration
dotnet ef migrations add MigrationName

# Apply migrations
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# View SQL script
dotnet ef migrations script

# Drop database
dotnet ef database drop

# List migrations
dotnet ef migrations list

# Update to specific migration
dotnet ef database update MigrationName
```

### 🚀 Run & Build Commands

```bash
# Run the project
dotnet run

# Run with watch (auto-restart on changes)
dotnet watch run

# Build the project
dotnet build

# Clean build artifacts
dotnet clean

# Restore NuGet packages
dotnet restore

# Publish for production
dotnet publish -c Release
```

### 🔍 Useful Commands

```bash
# Check .NET version
dotnet --version

# List installed SDKs
dotnet --list-sdks

# Create new project
dotnet new webapi -n ProjectName

# Add package
dotnet add package PackageName

# Remove package
dotnet remove package PackageName

# List packages
dotnet list package
```

---

## 🎯 Quick Verification Checklist

After completing all steps, verify:

### ✅ Database
- [ ] SQL Server is running
- [ ] Database is created
- [ ] Tables exist: `Users`, `AuditLogs`, `RefreshTokens`
- [ ] `__EFMigrationsHistory` table exists

**Check in SSMS:**
```sql
-- Check if database exists
SELECT name FROM sys.databases WHERE name = 'YourDatabaseName';

-- Check tables
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES;

-- Check users table
SELECT * FROM Users;
```

### ✅ API
- [ ] API runs without errors
- [ ] Scalar UI opens at `/scalar/v1`
- [ ] Health check returns healthy: `/health`

### ✅ Authentication
- [ ] Can register new user
- [ ] Can login with credentials
- [ ] Receive access token and refresh token
- [ ] Can access protected endpoint with token
- [ ] Cannot access protected endpoint without token
- [ ] Admin endpoint blocks non-admin users

### ✅ Audit Logs
- [ ] Registration logged
- [ ] Login logged
- [ ] Failed login attempts logged

**Check in SSMS:**
```sql
-- View audit logs
SELECT * FROM AuditLogs ORDER BY Timestamp DESC;

-- View refresh tokens
SELECT * FROM RefreshTokens;
```

---

## 🐛 Troubleshooting

### Issue: "Cannot connect to SQL Server"

**Solution:**
1. Check SQL Server is running:
   - Open "Services" (Windows + R → `services.msc`)
   - Find "SQL Server (SQLEXPRESS)"
   - Ensure it's "Running"
   - If stopped, right-click → Start

2. Verify connection string in `appsettings.json`:
```json
"Server=.\\SQLEXPRESS;Database=YourDb;Trusted_Connection=True;TrustServerCertificate=True"
```

### Issue: "401 Unauthorized" with valid token

**Solution:**
1. Check middleware order in `Program.cs`:
   - `UseAuthentication()` MUST come BEFORE `UseAuthorization()`

2. Verify token format:
   - Must be: `Bearer YOUR_TOKEN`
   - Space after "Bearer"
   - No quotes around token

### Issue: Migration fails

**Solution:**
```bash
# Remove last migration
dotnet ef migrations remove

# Delete Migrations folder
# Fix your models/entities
# Create migration again
dotnet ef migrations add NewMigration
dotnet ef database update
```

### Issue: "FluentValidation not working"

**Solution:**
Ensure in `Program.cs`:
```csharp
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssemblyContaining<Program>();
```

### Issue: Rate limiting not working

**Solution:**
Check `Program.cs` middleware order:
```csharp
app.UseIpRateLimiting();  // MUST be first
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();
```

---

## 🎓 Next Steps

### 1. Add Email Verification
Follow the guide in the main cheat sheet section **#14 JWT Authentication & Security**

### 2. Add More Entities & Controllers
```bash
# Example: Add Product entity
# 1. Create Models/Product.cs
# 2. Add DbSet to DbContext
# 3. Create migration
# 4. Create ProductController
```

### 3. Add Role Management
Create endpoints to:
- Assign roles to users
- View users by role
- Update user roles (admin only)

### 4. Production Deployment
- Move secrets to Azure Key Vault / AWS Secrets Manager
- Configure HTTPS certificate
- Set up CI/CD pipeline
- Enable Application Insights / Sentry

---

## 📚 Resources

- [ASP.NET Core Documentation](https://learn.microsoft.com/en-us/aspnet/core/)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)
- [JWT.io - Decode JWT](https://jwt.io/)
- [FluentValidation](https://docs.fluentvalidation.net/)
- [Scalar API Documentation](https://scalar.com/)

---

## 💡 Tips & Best Practices

### Security
✅ **Always** use HTTPS in production  
✅ **Never** commit secrets to Git  
✅ **Always** hash passwords (never store plain text)  
✅ **Always** validate user input  
✅ **Always** implement rate limiting  
✅ **Always** log security events  

### Performance
✅ Use async/await for database operations  
✅ Enable response caching where appropriate  
✅ Use pagination for large datasets  
✅ Index frequently queried columns  

### Code Quality
✅ Follow naming conventions  
✅ Use DTOs for API requests/responses  
✅ Keep controllers thin, logic in services  
✅ Write unit tests for services  
✅ Document your API with XML comments  

---

**🎉 Congratulations!** You now have a fully functional ASP.NET Core Web API with JWT authentication, rate limiting, audit logging, and all modern security features!

**Last Updated:** October 2024 | **Framework:** .NET 9.0 | **Database:** SQL Server