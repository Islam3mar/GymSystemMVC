<div align="center">

# 🏋️ Gym Management System

**An ASP.NET Core MVC application for running a gym: members with health records, membership plans, trainers, sessions, and bookings, built on a clean three-layer architecture with EF Core and SQL Server.**

[![.NET](https://img.shields.io/badge/.NET-10-512BD4?logo=dotnet&logoColor=white)](https://dotnet.microsoft.com/)
[![C#](https://img.shields.io/badge/C%23-239120?logo=csharp&logoColor=white)](https://learn.microsoft.com/dotnet/csharp/)
[![EF Core](https://img.shields.io/badge/EF%20Core-10-512BD4?logo=dotnet&logoColor=white)](https://learn.microsoft.com/ef/core/)
[![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?logo=microsoftsqlserver&logoColor=white)](https://www.microsoft.com/sql-server)
[![Bootstrap](https://img.shields.io/badge/Bootstrap-7952B3?logo=bootstrap&logoColor=white)](https://getbootstrap.com/)

[Features](#-features) · [Data Model](#-data-model) · [Architecture](#-architecture) · [Getting Started](#-getting-started) · [Configuration](#%EF%B8%8F-configuration) · [Roadmap](#-roadmap)

</div>

---

## 📖 Overview

**Gym Management System** ("Power Fitness") is a full-stack ASP.NET Core MVC application for managing the day-to-day operations of a gym. The domain model covers **members and their health records, membership plans, trainers, class categories, sessions, and bookings**, with the member and plan modules already wired end to end.

The project is built around **layered architecture** (Web, Business Logic, Data Access), **database-level integrity** (check constraints, unique indexes, composite keys), and **clean separation of concerns** through ViewModels, services, and a generic repository.

## ✨ Features

### 🧑‍🤝‍🧑 Member Management
- Full **CRUD** for members: list, create, details, edit, and delete.
- **Member details page** that also shows the member's **active plan** with its start and end dates.
- Each member has a **Health Record** (height, weight, blood type, notes) created together with the member and viewable on its own page.
- **Address** stored as an owned value object (building number, street, city).
- Gender and blood type selection through dropdowns.
- **Flash messages** (success, failure, not found) with an auto-fading alert box.

### ✅ Validation & Data Integrity
- **Egyptian mobile validation** (11 digits, prefixes `010`, `011`, `012`, `015`) on the ViewModels, the entity, and as a **database check constraint**.
- Email format validation, also enforced by a database check constraint.
- **Unique email and phone** per person, checked in the service layer and guaranteed by **unique indexes**.
- Name, city, and street are validated for allowed characters and length.
- Health ranges are validated (height and weight must be positive).
- **Smart deletion**: a member who still has a booking for a session that has not ended **cannot be deleted**.

### 💳 Membership Plans
- Plans are displayed as **cards** with name, price, duration, description, and an **active / inactive** state.
- Plan **details page** with the full information.
- Database rules: duration must be between **1 and 365 days**, price stored as `decimal(10,2)`.

### 🗓 Sessions, Trainers & Bookings (data layer)
- **Trainers** with a specialty (General Fitness, Yoga, Boxing, CrossFit) and hiring date.
- **Sessions** linked to a trainer and a category, with capacity and start/end dates.
- **Bookings** link members to sessions with an attendance flag, using a **composite primary key** so a member can only book a session once.
- Database rules: session capacity between **1 and 25**, and **end date must be after start date**.
- **Seeded categories**: Cardio, Strength, Yoga, Boxing, CrossFit.

### 🎨 UI
- Responsive layout built with **Bootstrap** and custom styling (Rubik font, Bootstrap Icons).
- Landing page with a stats bar, shared layout with navigation, and partial views for validation scripts and alert handling.

## 🗂 Data Model

| Entity | Key points |
|---|---|
| **Member** | Inherits shared user data (name, email, phone, birth date, gender, address); has a photo, one Health Record, many Memberships, many Bookings. Join date is stored in `JoinDate` |
| **Trainer** | Inherits shared user data; has a specialty and hiring date (`HireDate`), and many Sessions |
| **HealthRecord** | One-to-one with Member: height, weight, blood type, note |
| **Plan** | Name, description, price, duration in days, active flag |
| **MemberShip** | Member + Plan + start date + end date; **status is computed** (active while the end date is in the future) |
| **Category** | Class category (seeded) |
| **Session** | Trainer + Category + capacity + start and end dates |
| **Booking** | Member + Session (composite key), booking date, attendance flag |

**Relationships & rules**

- Member and Trainer share their base properties and configuration through a generic `GymUserConfigurations<T>`, and each maps to its own table.
- `MemberShip → Plan` uses **Restrict** delete, so a plan in use can't be removed; `MemberShip → Member` uses **Cascade**.
- All entities inherit `BaseEntity` (`Id`, `CreatedAt`, `UpdatedAt`).
- `CreatedAt` is mapped to meaningful columns per table (`JoinDate`, `HireDate`, `StartDate`, `BookingDate`) with a `GETDATE()` default.

## 🏗 Architecture

The solution is split into three projects with a one-way dependency flow:

```
┌──────────────────────────────────────────────┐
│  GymSystem  (Web)                            │
│  MVC Controllers · Razor Views ·             │
│  Layout & partials · Program.cs              │
└──────────────────────┬───────────────────────┘
                       │ depends on
┌──────────────────────▼───────────────────────┐
│  GymSystem.BLL                               │
│  Services · ViewModels · Validation rules    │
└──────────────────────┬───────────────────────┘
                       │ depends on
┌──────────────────────▼───────────────────────┐
│  GymSystem.DAL                               │
│  Entities · DbContext · Configurations ·     │
│  Migrations · Repositories · Unit of Work    │
└──────────────────────────────────────────────┘
```

**Patterns & practices**

| Pattern | Where |
|---|---|
| Generic Repository | `IGenaricRepository<T>` / `GenaricRepository<T>` |
| Specific repositories | `IMemberRepository`, `IPlanRepository` (extension points) |
| Unit of Work | `IUnitOfWork` / `UnitOfWork` (lazily creates and caches one repository per entity per request) |
| Service layer | `IMemberService` / `MemberService` |
| ViewModels (Model → ViewModel → View) | `BLL/ViewModels` with Data Annotations |
| Dependency Injection | `Program.cs` |
| Fluent API configurations | One `IEntityTypeConfiguration` per entity, loaded with `ApplyConfigurationsFromAssembly` |
| Owned types | `Address` |
| Data seeding | `HasData` for categories |
| Async everywhere | `async/await` with `CancellationToken` support from controller to repository |

## 🛠 Tech Stack

| Area | Technology |
|---|---|
| Framework | ASP.NET Core MVC on **.NET 10** |
| Data | Entity Framework Core 10, SQL Server |
| Validation | Data Annotations (server-side) + jQuery unobtrusive validation scripts |
| Frontend | Razor Views, Bootstrap, Bootstrap Icons, custom CSS |

## 📁 Project Structure

```
GymSystemSolution/
├── GymSystem/                       # Web (MVC)
│   ├── Controllers/                 # Home, Member, Plan
│   ├── Views/                       # Member, Plan, Home, Shared (layout + partials)
│   ├── wwwroot/                     # css, js, images
│   ├── appsettings.json
│   └── Program.cs
├── GymSystem.BLL/
│   ├── Services/
│   │   ├── Interfaces/              # IMemberService
│   │   └── Classes/                 # MemberService
│   └── ViewModels/
│       └── MembersViewModels/       # Create, Update, Details, HealthRecord
└── GymSystem.DAL/
    ├── Entities/                    # Member, Trainer, Plan, MemberShip, Session, Booking, ...
    │   └── Enums/                   # Gender, Specialites
    ├── Contexts/                    # GymDbContext
    ├── Configurations/              # Fluent API per entity
    ├── Migrations/
    └── Repositories/
        ├── Interfaces/
        └── Classes/
```

## 🚀 Getting Started

### Prerequisites
- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- SQL Server (LocalDB, Express, or full)
- EF Core CLI: `dotnet tool install --global dotnet-ef`

### Run locally

```bash
# 1. Clone
git clone https://github.com/Islam3mar/GymManagementSystem.git
cd GymManagementSystem

# 2. Set your connection string (never commit real credentials)
cd GymSystem
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=.;Database=GymSystemDb;Trusted_Connection=True;TrustServerCertificate=True;"
cd ..

# 3. Create the database from the migrations
dotnet ef database update --project GymSystem.DAL --startup-project GymSystem

# 4. Run
dotnet run --project GymSystem
```

The app starts on `https://localhost:7146` (or `http://localhost:5081`).

> The database is **not** migrated automatically on startup, so step 3 is required. Categories are seeded by the migration; **plans are not**, so add a few to see the Plans page:

```sql
INSERT INTO Plans (Name, Description, Price, Duration, IsActive)
VALUES ('Monthly', 'Access to gym equipment during staffed hours', 300, 30, 1),
       ('Quarterly', 'Three months of full gym access', 800, 90, 1);
```

## ⚙️ Configuration

| Key | Description |
|---|---|
| `ConnectionStrings:DefaultConnection` | SQL Server connection string for `GymSystemDb` |

## 🧪 Core Business Rules

- Member email and phone must be **unique** and the phone must be a valid **Egyptian mobile number**.
- Every member is created together with a **Health Record**.
- A member with an **upcoming or ongoing booked session cannot be deleted**.
- A plan's duration must be between **1 and 365 days**.
- A membership is **active** only while its end date is in the future.
- A session's capacity must be between **1 and 25**, and it must end after it starts.
- A member can book a given session **only once**.
- A plan that has memberships **cannot be deleted**.

## 🗺 Roadmap

- [x] Members: full CRUD, health record, active plan details
- [x] Plans: list and details
- [x] Full data model with constraints, indexes, and seeded categories
- [ ] Plans: create, edit, activate / deactivate
- [ ] Trainers management
- [ ] Sessions management and schedule
- [ ] Bookings and attendance
- [ ] Memberships: subscribe a member to a plan, renewals
- [ ] Live dashboard statistics on the home page
- [ ] Authentication and authorization
- [ ] Member photo upload

## 🗺 Screenshots

> _Add screenshots here: home page, members list, create member, member details, health record, plans._

```md
![Members](docs/screenshots/members.png)
```

## 👤 Author

**Islam Omar**, Back-End Developer (.NET)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?logo=linkedin&logoColor=white)](https://linkedin.com/in/islam-omar-3ba867266)
[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github&logoColor=white)](https://github.com/Islam3mar)
[![Portfolio](https://img.shields.io/badge/Portfolio-1F5F7A?logoColor=white)](https://islam3mar.github.io/portfolio/)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail&logoColor=white)](mailto:islam3marahmed@gmail.com)
