```markdown
# ASP.NET Core Web API - Setup Guide

![.NET](https://img.shields.io/badge/.NET-9.0-512BD4?logo=dotnet)
![EF Core](https://img.shields.io/badge/EF%20Core-9.0-512BD4)
![License](https://img.shields.io/badge/license-MIT-green)

> Complete cheat sheet for building .NET Web APIs with Entity Framework Core.

---

## 📚 View the Guide

**Choose your preferred format:**

### 📖 [GitHub Markdown Version 1.3](./dotnet-api-cheat-sheet.md)
Read directly on GitHub with syntax highlighting and navigation.

### 🚀 [Quick Start Guide - Get Started in 30 Minutes!](./Quick_Start.md)
Step-by-step tutorial to build your first API with JWT authentication from scratch.

### 📝 [Notion Template Version 1.2 → 1.3 ![updating](https://img.shields.io/badge/status-updating-blue?style=flat&logo=notion&logoColor=white)](https://seifbenali.notion.site/dotnet-api-guide-notion-28bfe8fdb58580929399e392bbbb74a0)
Interactive version with collapsible sections. Click to **duplicate to your workspace**!

---

## 🚀 Quick Links

### Getting Started
- [🎯 Quick Start Tutorial](./Quick_Start.md) - **NEW!** Build your first API in 30 minutes
- [Program.cs Setup](./dotnet-api-cheat-sheet.md#1-programcs)
- [Database Configuration](./dotnet-api-cheat-sheet.md#2-appsettingsjson)
- [NuGet Packages Installation](./Quick_Start.md#nuget-packages-installation)

### Core Features
- [DbContext Setup](./dotnet-api-cheat-sheet.md#3-dataappdbcontextcs)
- [Models & Relationships](./dotnet-api-cheat-sheet.md#4-modelsyourmodelcs)
- [Controllers & CRUD](./dotnet-api-cheat-sheet.md#5-controllersyourcontrollercs)
- [Essential Commands](./dotnet-api-cheat-sheet.md#7-essential-ef-core-commands)

### Authentication & Security
- [JWT Authentication Setup](./dotnet-api-cheat-sheet.md#14-jwt-authentication--security)
- [Quick Start Auth Guide](./Quick_Start.md#authentication--authorization)
- [Security Best Practices](./dotnet-api-cheat-sheet.md#18-best-practices-de-sécurité)

---

## 📦 What's Inside

### Core Features
- ✅ Complete project setup templates
- ✅ DbContext and model examples
- ✅ CRUD controller patterns
- ✅ Entity Framework Core commands
- ✅ Relationship configurations (One-to-One, One-to-Many, Many-to-Many)
- ✅ Best practices and troubleshooting

### Advanced Features
- ✅ **JWT Authentication & Authorization** 🔐
- ✅ **Refresh Token Rotation** 🔄
- ✅ **Audit Logging** 📊
- ✅ **Rate Limiting** ⚡
- ✅ **Email Verification** ✉️
- ✅ **Password Reset** 🔑
- ✅ **Account Lockout** 🔒

### Patterns & Architecture
- ✅ Repository and DTO patterns
- ✅ Service layer implementation
- ✅ FluentValidation integration
- ✅ CORS configuration examples

---

## 🛠️ Technologies

- **.NET 9.0** - Latest framework
- **Entity Framework Core 9.0** - ORM for database operations
- **SQL Server** - Database engine
- **JWT Bearer Authentication** - Secure token-based auth
- **FluentValidation** - Request validation
- **AspNetCoreRateLimit** - Rate limiting & DDoS protection
- **Scalar API Documentation** - Interactive API testing

---

## 🎯 Quick Start

### Option 1: Fast Track (Recommended for Beginners)
Follow the [Quick Start Guide](./Quick_Start.md) - includes everything you need:
- ✅ Prerequisites checklist
- ✅ Step-by-step instructions
- ✅ Copy-paste ready code
- ✅ Testing examples
- ✅ Troubleshooting tips

```bash
# 1. Follow Quick_Start.md guide
# 2. Install NuGet packages (commands provided)
# 3. Copy code templates
# 4. Run migrations
# 5. Test your API
```

### Option 2: Manual Setup
```bash
# 1. Clone or download this repository
# 2. Copy templates from dotnet-api-cheat-sheet.md
# 3. Update connection strings in appsettings.json
# 4. Run migrations: Add-Migration InitialCreate
# 5. Update database: Update-Database
# 6. Run your API: dotnet run
# 7. Visit https://localhost:7020/scalar/v1
```

---

## 📁 Repository Structure

```
├── README.md                    # This file (navigation hub)
├── Quick_Start.md              # 🚀 NEW! Step-by-step tutorial
├── dotnet-api-cheat-sheet.md   # Complete reference guide
└── dotnet-api-guide-notion.md  # Notion import file
```

---

## 💡 How to Use This Guide

### 🆕 New to ASP.NET Core?
**Start here:** [Quick_Start.md](./Quick_Start.md)
- Complete walkthrough from zero to working API
- Includes JWT authentication setup
- Visual Studio 2022 AND VS Code instructions
- Ready-to-use code templates
- Estimated time: 30-60 minutes

### 📚 Need a Reference?
**Use:** [dotnet-api-cheat-sheet.md](./dotnet-api-cheat-sheet.md)
- Comprehensive code examples
- Advanced patterns and features
- Security best practices
- Troubleshooting section

### 🎨 Prefer Notion?
1. Click the [Notion Template Link](https://seifbenali.notion.site/dotnet-api-guide-notion-28bfe8fdb58580929399e392bbbb74a0)
2. Click **"Duplicate"** to add to your workspace
3. Customize and add your own notes
4. Use built-in table of contents for navigation

### 💻 For Local Reference:
1. Download `Quick_Start.md` or `dotnet-api-cheat-sheet.md`
2. Open in VS Code with Markdown Preview (Ctrl+Shift+V)
3. Keep it open while coding for quick reference

---

## 🎓 Who Is This For?

- **Complete Beginners** - Start with [Quick_Start.md](./Quick_Start.md)
- **Intermediate developers** - Use [dotnet-api-cheat-sheet.md](./dotnet-api-cheat-sheet.md) for reference
- **Teams** establishing coding standards
- **Students** learning .NET development
- **Anyone** building REST APIs with Entity Framework Core and JWT authentication

---

## 🎯 Learning Path

```
1. Quick_Start.md          ← Start here!
   └─ Basic API setup
   └─ Database configuration
   └─ JWT Authentication
   └─ Testing your API

2. dotnet-api-cheat-sheet.md
   └─ Advanced features
   └─ Security best practices
   └─ Complex relationships
   └─ Production deployment

3. Build Your Own Project
   └─ Apply what you learned
   └─ Use guides as reference
   └─ Customize for your needs
```

---

## 📊 What You'll Build

Following the Quick Start guide, you'll create:

✅ **Fully functional REST API** with CRUD operations  
✅ **JWT Authentication System** with access & refresh tokens  
✅ **User Registration & Login**  
✅ **Token Refresh Mechanism** (automatic rotation)  
✅ **Role-based Authorization** (User, Admin roles)  
✅ **Rate Limiting** (protection against brute force)  
✅ **Audit Logging** (track all user actions)  
✅ **Account Security** (lockout after failed attempts)  
✅ **SQL Server Database** with Entity Framework Core  
✅ **Interactive API Documentation** with Scalar UI  

---

## 🔧 Installation Commands

### Visual Studio 2022 (Package Manager Console)
```powershell
# Copy these commands from Quick_Start.md
Install-Package Microsoft.EntityFrameworkCore -Version 9.0.10
Install-Package Microsoft.EntityFrameworkCore.SqlServer -Version 9.0.10
# ... (see Quick_Start.md for complete list)
```

### VS Code / Terminal
```bash
# Copy these commands from Quick_Start.md
dotnet add package Microsoft.EntityFrameworkCore --version 9.0.10
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 9.0.10
# ... (see Quick_Start.md for complete list)
```

Full installation guide: [Quick_Start.md#nuget-packages-installation](./Quick_Start.md#nuget-packages-installation)

---

## 🧪 Testing Your API

After setup, test your API at:
- **Scalar UI:** `https://localhost:7020/scalar/v1` (Interactive docs)
- **Health Check:** `https://localhost:7020/health`
- **Sample Endpoints:**
  - `POST /api/auth/register` - Register new user
  - `POST /api/auth/login` - Login and get tokens
  - `GET /api/auth/me` - Get current user (protected)
  - `POST /api/auth/refresh-tokens` - Refresh your tokens

Full testing guide: [Quick_Start.md#testing-your-api](./Quick_Start.md#testing-your-api)

---

## 🐛 Common Issues & Solutions

### "Cannot connect to SQL Server"
See: [Quick_Start.md#troubleshooting](./Quick_Start.md#troubleshooting)

### "401 Unauthorized with valid token"
See: [Quick_Start.md#troubleshooting](./Quick_Start.md#troubleshooting)

### "Migration fails"
See: [Quick_Start.md#troubleshooting](./Quick_Start.md#troubleshooting)

Full troubleshooting guide available in both documents.

---

## 📝 License

This guide is free to use for learning and reference. Feel free to fork, share, and customize!

---

## 🤝 Contributing

Found an error or want to add something? Feel free to:
- Open an issue
- Submit a pull request
- Share your feedback

---

## ⭐ Show Your Support

If you find this guide helpful, please:
- **Star** ⭐ this repository
- **Share** with your team and on social media
- **Duplicate** the Notion template
- **Contribute** improvements or suggestions

---

## 📧 Contact & Feedback

Have questions or suggestions? Open an issue on GitHub!

---

## 🔄 Version History

- **v1.3** (Current) - Added Quick_Start.md with JWT authentication tutorial
- **v1.2** - Added JWT Authentication & Security section
- **v1.1** - Added advanced relationships and patterns
- **v1.0** - Initial release with basic CRUD setup

---

## 📚 Additional Resources

- [Official .NET Documentation](https://learn.microsoft.com/en-us/dotnet/)
- [Entity Framework Core Docs](https://learn.microsoft.com/en-us/ef/core/)
- [JWT.io - Token Decoder](https://jwt.io/)
- [ASP.NET Core Security](https://learn.microsoft.com/en-us/aspnet/core/security/)

---

**Last Updated:** October 2024 | **Framework:** .NET 9.0 | **EF Core:** 9.0.10

**🚀 Ready to get started? Open [Quick_Start.md](./Quick_Start.md) and build your first API!**
```
