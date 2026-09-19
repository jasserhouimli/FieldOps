# FieldOps

FieldOps is a modular field service management platform built with ASP.NET Core and .NET 10. It is designed for organizations that manage field technicians, service work, parts inventory, schedules, and billing operations from a single backend system.

The solution follows a modular monolith architecture with separate modules for identity, customers, technicians, work orders, inventory, scheduling, and invoicing. Each module exposes its own minimal API endpoints and uses its own EF Core DbContext with a dedicated schema in a shared PostgreSQL database.

## Why FieldOps?

Field service businesses often need to coordinate several moving parts at once:

- customer records and service history
- technician profiles and skills
- work orders and assignments
- inventory tracking and stock movements
- technician schedules and availability
- invoicing and billing workflows

FieldOps brings these concerns together in a single backend API, making it easier to extend and maintain while keeping the system organized around business domains.

## Features

- Secure authentication and authorization with JWT
- Refresh token support using HttpOnly cookies
- Role-agnostic modular architecture
- Minimal API endpoints with clean vertical-slice organization
- Fluent validation on incoming requests
- Shared result pattern for consistent API responses
- Global exception handling
- Rate limiting for public endpoints
- Swagger/OpenAPI documentation
- PostgreSQL persistence with EF Core
- Health check endpoint

## Tech Stack

- .NET 10
- ASP.NET Core Minimal APIs
- Entity Framework Core
- PostgreSQL
- JWT authentication
- FluentValidation
- Serilog
- Swagger / OpenAPI
- C# modular monolith design

## Architecture

This project is built around a modular monolith pattern:

- each business domain is isolated in its own module
- each module has its own persistence layer and migrations
- the API project composes all modules at startup
- modules expose endpoints using a clean feature-based structure
- the infrastructure project contains shared abstractions and middleware

This gives you the maintainability of modular boundaries without the overhead of a distributed system.

## Modules

| Module | Schema | Purpose |
|--------|--------|---------|
| Identity | `identity` | User registration, login, JWT auth, token refresh, logout |
| Customers | `customers` | Customer management and lookup |
| Technicians | `technicians` | Technician profiles, skill tracking, maintenance |
| Work Orders | `work_orders` | Service requests, assignments, job lifecycle |
| Inventory | `inventory` | Parts, stock levels, stock transactions |
| Scheduling | `scheduling` | Appointments and date-based scheduling |
| Invoicing | `invoicing` | Invoice creation, line items, calculation, status workflow |

## Project Structure

```text
FieldOps/
├── src/
│   ├── FieldOps.Api/                     # API host, authentication, middleware, Swagger
│   ├── FieldOps.Infrastructure/          # Shared infrastructure and utilities
│   └── Modules/
│       ├── FieldOps.Modules.Identity/     # Auth and user management
│       ├── FieldOps.Modules.Customers/    # Customer domain
│       ├── FieldOps.Modules.Technicians/  # Technician domain
│       ├── FieldOps.Modules.WorkOrders/   # Work order domain
│       ├── FieldOps.Modules.Inventory/    # Inventory domain
│       ├── FieldOps.Modules.Scheduling/   # Scheduling domain
│       └── FieldOps.Modules.Invoicing/   # Billing domain
├── FieldOps.slnx
├── README.md
├── .gitignore
└── ...
```

## Getting Started

### Prerequisites

- .NET 10 SDK
- PostgreSQL instance
- A local or remote PostgreSQL database for development

### 1. Clone the repository

```bash
git clone https://github.com/jasserhouimli/FieldOps.git
cd FieldOps
```

### 2. Configure app settings

Copy the existing appsettings file and provide your actual database and JWT values:

```bash
cp src/FieldOps.Api/appsettings.json src/FieldOps.Api/appsettings.Development.json
```

Then update `src/FieldOps.Api/appsettings.Development.json` with:

```json
{
  "ConnectionStrings": {
    "FieldOps": "Host=localhost;Database=fieldops;Username=postgres;Password=your_password"
  },
  "Jwt": {
    "Key": "your-super-secret-key-at-least-32-characters",
    "Issuer": "FieldOps",
    "Audience": "FieldOps",
    "AccessTokenExpirationMinutes": 60,
    "RefreshTokenExpirationDays": 7
  }
}
```

### 3. Apply database migrations

Each module has its own EF Core DbContext and migrations. Apply them with:

```bash
dotnet ef database update --context IdentityDbContext --project src/Modules/FieldOps.Modules.Identity --startup-project src/FieldOps.Api
dotnet ef database update --context CustomersDbContext --project src/Modules/FieldOps.Modules.Customers --startup-project src/FieldOps.Api
dotnet ef database update --context TechniciansDbContext --project src/Modules/FieldOps.Modules.Technicians --startup-project src/FieldOps.Api
dotnet ef database update --context WorkOrdersDbContext --project src/Modules/FieldOps.Modules.WorkOrders --startup-project src/FieldOps.Api
dotnet ef database update --context InventoryDbContext --project src/Modules/FieldOps.Modules.Inventory --startup-project src/FieldOps.Api
dotnet ef database update --context SchedulingDbContext --project src/Modules/FieldOps.Modules.Scheduling --startup-project src/FieldOps.Api
dotnet ef database update --context InvoicingDbContext --project src/Modules/FieldOps.Modules.Invoicing --startup-project src/FieldOps.Api
```

### 4. Run the API

```bash
dotnet restore
dotnet run --project src/FieldOps.Api
```

### 5. Open Swagger

Once the app is running, open:

```text
https://localhost:5001/swagger
```

or the port configured by your local environment.

## API Overview

Most endpoints are secured and require authentication unless explicitly marked otherwise.

### Authentication

- `POST /auth/register`
- `POST /auth/login`
- `GET /auth/me`
- `POST /auth/refresh`
- `POST /auth/logout`

### Customers

- `POST /customers`
- `GET /customers`
- `GET /customers/{id}`
- `PUT /customers/{id}`
- `DELETE /customers/{id}`

### Technicians

- `POST /technicians`
- `GET /technicians`
- `GET /technicians/{id}`
- `PUT /technicians/{id}`
- `DELETE /technicians/{id}`
- `POST /technicians/{id}/skills`

### Work Orders

- `POST /work-orders`
- `GET /work-orders`
- `GET /work-orders/{id}`
- `PUT /work-orders/{id}`
- `DELETE /work-orders/{id}`
- `PUT /work-orders/{id}/assign-technician`

### Inventory

- `POST /inventory`
- `GET /inventory`
- `GET /inventory/{id}`
- `PUT /inventory/{id}`
- `DELETE /inventory/{id}`
- `POST /inventory/transactions`

### Scheduling

- `POST /schedules`
- `GET /schedules`
- `GET /schedules/{id}`
- `PUT /schedules/{id}`
- `DELETE /schedules/{id}`

### Invoicing

- `POST /invoices`
- `GET /invoices`
- `GET /invoices/{id}`
- `PUT /invoices/{id}`
- `DELETE /invoices/{id}`
- `POST /invoices/{id}/items`

## Health Check

```http
GET /health
```

Returns a simple API health response with current UTC timestamp.

## Contributing

Contributions are welcome. To contribute:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add or update tests if applicable
5. Submit a pull request

## License

This project currently does not specify a license. If you plan to distribute or commercialize it, you should add an appropriate license such as MIT or Apache 2.0.

## Status

FieldOps is currently structured as a backend API foundation for field service operations and is well-suited for extension into a full business platform with frontends, reporting, mobile integrations, and advanced workflow automation.
