# 🚀 ASP.NET Core Web API - Complete Guide

<div align="center">

![.NET](https://img.shields.io/badge/.NET-9.0-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![EF Core](https://img.shields.io/badge/EF%20Core-9.0-512BD4?style=for-the-badge&logo=nuget&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Last Updated](https://img.shields.io/badge/Updated-October%202024-blue?style=for-the-badge)

**Complete cheat sheet for building modern .NET Web APIs with Entity Framework Core & JWT Authentication**

[📖 View Guide](#-documentation) • [🚀 Quick Start](#-quick-start) • [💬 Report Issue](../../issues)

</div>

---

## 📚 Documentation

Choose your preferred format to get started:

<table>
<tr>
<td width="33%" align="center">

### 🎯 Quick Start
**[30-Minute Tutorial →](./Quick_Start.md)**

Perfect for beginners

Step-by-step with code

JWT Authentication

</td>
<td width="33%" align="center">

### 📖 Full Guide
**[Complete Reference →](./dotnet-api-cheat-sheet.md)**

Comprehensive examples

Advanced patterns

Production tips

</td>
<td width="33%" align="center">

### 📝 Notion
**[Interactive Template →](https://seifbenali.notion.site/dotnet-api-guide-notion-28bfe8fdb58580929399e392bbbb74a0)**

Collapsible sections

Easy to customize

Duplicate to workspace

</td>
</tr>
</table>

---

## ✨ What's Inside

<details open>
<summary><b>🔰 Core Features</b></summary>
<br>

- ✅ Complete project setup templates
- ✅ DbContext and model examples
- ✅ CRUD controller patterns
- ✅ Entity Framework Core commands
- ✅ Relationship configurations (1:1, 1:N, N:M)
- ✅ Best practices & troubleshooting

</details>

<details>
<summary><b>🔐 Advanced Security</b></summary>
<br>

- ✅ **JWT Authentication & Authorization**
- ✅ **Refresh Token Rotation**
- ✅ **Audit Logging System**
- ✅ **Rate Limiting & DDoS Protection**
- ✅ **Email Verification**
- ✅ **Password Reset Flow**
- ✅ **Account Lockout Protection**
- ✅ **CORS Configuration**

</details>

<details>
<summary><b>🏗️ Architecture & Patterns</b></summary>
<br>

- ✅ Repository pattern implementation
- ✅ DTO (Data Transfer Objects) pattern
- ✅ Service layer architecture
- ✅ FluentValidation integration
- ✅ Dependency injection examples

</details>

---

## 🚀 Quick Start

### Prerequisites

<table>
<tr>
<td>

**Required:**
- Visual Studio 2022 or VS Code
- .NET 9.0 SDK
- SQL Server (Express/Full)

</td>
<td>

**Optional:**
- SQL Server Management Studio
- Postman or similar API client
- Git for version control

</td>
</tr>
</table>

### ⚡ Fast Track (Recommended for Beginners)

```bash
# 1. Follow the Quick Start Guide
# 2. Install packages (commands provided)
# 3. Copy code templates
# 4. Run migrations
# 5. Test your API
```

**[👉 Open Quick Start Guide](./Quick_Start.md)** - Get your API running in 30 minutes!

### 🛠️ Manual Setup

```bash
# Clone or download repository
git clone <repository-url>

# Navigate to project
cd your-project-folder

# Follow the comprehensive guide
# 1. Update appsettings.json connection string
# 2. Run: Add-Migration InitialCreate
# 3. Run: Update-Database
# 4. Start: dotnet run
# 5. Visit: https://localhost:7020/scalar/v1
```

---

## 📂 Repository Structure

```
📦 Repository
├── 📄 README.md                    ← You are here
├── 🚀 Quick_Start.md              ← 30-min tutorial
├── 📖 dotnet-api-cheat-sheet.md   ← Complete reference
└── 📝 dotnet-api-guide-notion.md  ← Notion import file
```

---

## 🎓 Learning Path

```mermaid
graph LR
    A[Quick Start] --> B[Build Basic API]
    B --> C[Add Authentication]
    C --> D[Advanced Features]
    D --> E[Production Ready]
    
    style A fill:#4CAF50
    style B fill:#2196F3
    style C fill:#FF9800
    style D fill:#9C27B0
    style E fill:#F44336
```

<table>
<tr>
<th width="20%">Step</th>
<th width="40%">What You'll Learn</th>
<th width="20%">Time</th>
<th width="20%">Guide</th>
</tr>
<tr>
<td>1️⃣ Basics</td>
<td>
• Project setup<br>
• Database configuration<br>
• CRUD operations
</td>
<td>30 min</td>
<td><a href="./Quick_Start.md">Quick Start</a></td>
</tr>
<tr>
<td>2️⃣ Security</td>
<td>
• JWT authentication<br>
• User registration/login<br>
• Token management
</td>
<td>45 min</td>
<td><a href="./Quick_Start.md#authentication--authorization">Auth Guide</a></td>
</tr>
<tr>
<td>3️⃣ Advanced</td>
<td>
• Complex relationships<br>
• Validation patterns<br>
• Performance optimization
</td>
<td>60 min</td>
<td><a href="./dotnet-api-cheat-sheet.md">Full Guide</a></td>
</tr>
<tr>
<td>4️⃣ Production</td>
<td>
• Security hardening<br>
• Monitoring setup<br>
• Deployment strategies
</td>
<td>90 min</td>
<td><a href="./dotnet-api-cheat-sheet.md#18-best-practices-de-sécurité">Best Practices</a></td>
</tr>
</table>

---

## 💻 Technologies

<div align="center">

| Technology | Version | Purpose |
|------------|---------|---------|
| ![.NET](https://img.shields.io/badge/.NET-9.0-512BD4?logo=dotnet) | 9.0 | Framework |
| ![EF Core](https://img.shields.io/badge/EF_Core-9.0-512BD4?logo=nuget) | 9.0 | ORM |
| ![SQL Server](https://img.shields.io/badge/SQL_Server-2022-CC2927?logo=microsoftsqlserver) | 2019+ | Database |
| ![JWT](https://img.shields.io/badge/JWT-Bearer-000000?logo=jsonwebtokens) | Latest | Auth |
| ![Scalar](https://img.shields.io/badge/Scalar-API_Docs-00C7B7) | Latest | Documentation |

</div>

---

## 🎯 What You'll Build

Following this guide, you'll create:

<table>
<tr>
<td width="50%">

### 🔧 Technical Features

- ✅ RESTful API with CRUD operations
- ✅ SQL Server database with EF Core
- ✅ JWT authentication system
- ✅ Role-based authorization
- ✅ Token refresh mechanism
- ✅ Interactive API documentation

</td>
<td width="50%">

### 🛡️ Security Features

- ✅ Password hashing & salting
- ✅ Email verification system
- ✅ Password reset flow
- ✅ Rate limiting protection
- ✅ Audit logging
- ✅ Account lockout

</td>
</tr>
</table>

---

## 📖 Quick Links

<div align="center">

| Category | Links |
|----------|-------|
| **🎯 Getting Started** | [Quick Start](./Quick_Start.md) • [Prerequisites](./Quick_Start.md#prerequisites) • [Installation](./Quick_Start.md#nuget-packages-installation) |
| **📚 Core Concepts** | [DbContext Setup](./dotnet-api-cheat-sheet.md#3-dataappdbcontextcs) • [Models](./dotnet-api-cheat-sheet.md#4-modelsyourmodelcs) • [Controllers](./dotnet-api-cheat-sheet.md#5-controllersyourcontrollercs) |
| **🔐 Authentication** | [JWT Setup](./dotnet-api-cheat-sheet.md#14-jwt-authentication--security) • [Quick Start Auth](./Quick_Start.md#authentication--authorization) • [Security Guide](./dotnet-api-cheat-sheet.md#18-best-practices-de-sécurité) |
| **🛠️ Tools & Commands** | [EF Core Commands](./dotnet-api-cheat-sheet.md#7-essential-ef-core-commands) • [Testing](./Quick_Start.md#testing-your-api) • [Troubleshooting](./Quick_Start.md#troubleshooting) |

</div>

---

## 🧪 Testing Your API

After setup, access these endpoints:

```bash
# API Documentation (Interactive)
https://localhost:7020/scalar/v1

# Health Check
https://localhost:7020/health

# Sample Endpoints
POST   /api/auth/register      # Create account
POST   /api/auth/login         # Login
GET    /api/auth/me           # Get current user (protected)
POST   /api/auth/refresh      # Refresh tokens
```

**[📝 Complete Testing Guide →](./Quick_Start.md#testing-your-api)**

---

## 🐛 Troubleshooting

<details>
<summary><b>Cannot connect to SQL Server</b></summary>
<br>

**Solution:**
1. Verify SQL Server is running in Services
2. Check connection string format in `appsettings.json`
3. Ensure `TrustServerCertificate=True` is included

**[Full Solution →](./Quick_Start.md#troubleshooting)**

</details>

<details>
<summary><b>401 Unauthorized with valid token</b></summary>
<br>

**Solution:**
1. Check middleware order in `Program.cs`
2. `UseAuthentication()` must come before `UseAuthorization()`
3. Verify token format: `Bearer YOUR_TOKEN`

**[Full Solution →](./Quick_Start.md#troubleshooting)**

</details>

<details>
<summary><b>Migration fails</b></summary>
<br>

**Solution:**
```bash
# Remove last migration
dotnet ef migrations remove

# Fix your models
# Create new migration
dotnet ef migrations add NewMigration
dotnet ef database update
```

**[Full Solution →](./Quick_Start.md#troubleshooting)**

</details>

---

## 🎓 Who Is This For?

<table>
<tr>
<td align="center" width="25%">

### 👨‍🎓 Students
Learning .NET development

</td>
<td align="center" width="25%">

### 🆕 Beginners
First API project

</td>
<td align="center" width="25%">

### 👨‍💻 Developers
Need quick reference

</td>
<td align="center" width="25%">

### 👥 Teams
Establishing standards

</td>
</tr>
</table>

---

## 📊 Version History

<table>
<tr>
<th>Version</th>
<th>Date</th>
<th>Changes</th>
</tr>
<tr>
<td><b>v1.3</b></td>
<td>Oct 2024</td>
<td>
• Added Quick Start guide<br>
• Complete JWT tutorial<br>
• Enhanced security features
</td>
</tr>
<tr>
<td>v1.2</td>
<td>Sep 2024</td>
<td>
• JWT authentication section<br>
• Advanced security features
</td>
</tr>
<tr>
<td>v1.1</td>
<td>Aug 2024</td>
<td>
• Relationship patterns<br>
• Repository pattern
</td>
</tr>
<tr>
<td>v1.0</td>
<td>Jul 2024</td>
<td>Initial release</td>
</tr>
</table>

---

## 🌟 Show Your Support

If this guide helped you, please:

<div align="center">

[![Star this repo](https://img.shields.io/badge/⭐_Star_this_repo-yellow?style=for-the-badge)](../../stargazers)
[![Share on Twitter](https://img.shields.io/badge/Share_on_Twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white)](https://twitter.com/intent/tweet?text=Check%20out%20this%20awesome%20.NET%20Web%20API%20guide!)
[![Duplicate Notion](https://img.shields.io/badge/Duplicate_Notion-000000?style=for-the-badge&logo=notion&logoColor=white)](https://seifbenali.notion.site/dotnet-api-guide-notion-28bfe8fdb58580929399e392bbbb74a0)

</div>

---

## 📚 Additional Resources

<div align="center">

| Resource | Link |
|----------|------|
| 📘 Official .NET Docs | [docs.microsoft.com/dotnet](https://learn.microsoft.com/en-us/dotnet/) |
| 📗 EF Core Documentation | [docs.microsoft.com/ef/core](https://learn.microsoft.com/en-us/ef/core/) |
| 🔐 JWT Decoder | [jwt.io](https://jwt.io/) |
| 🛡️ OWASP Security Guide | [owasp.org/cheatsheets](https://cheatsheetseries.owasp.org/) |
| 📊 API Best Practices | [restfulapi.net](https://restfulapi.net/) |

</div>

---

## 🤝 Contributing

Found an error or want to improve this guide?

1. 🍴 Fork this repository
2. 🔧 Make your changes
3. 📝 Submit a pull request
4. 🎉 Get credit in contributors list!

---

## 📄 License

This guide is free to use for learning and reference. Feel free to:

- ✅ Use for personal projects
- ✅ Share with your team
- ✅ Modify for your needs
- ✅ Contribute improvements

---

## 💬 Need Help?

- 📧 Open an [issue](../../issues) for questions
- 💡 Check [troubleshooting guide](./Quick_Start.md#troubleshooting)
- 📚 Review [complete documentation](./dotnet-api-cheat-sheet.md)

---

<div align="center">

**Built with ❤️ for the .NET Community**

⭐ **Star this repo** if you found it helpful! ⭐

[🚀 Get Started Now](./Quick_Start.md) • [📖 Full Documentation](./dotnet-api-cheat-sheet.md) • [💬 Ask Questions](../../issues)

---

**Last Updated:** October 2024 | **Framework:** .NET 9.0 | **EF Core:** 9.0

</div>