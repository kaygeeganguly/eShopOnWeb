# Architecture Diagram

eShopOnWeb is a reference ASP.NET Core application implementing a multi-layer e-commerce store, with a Razor Pages / MVC web frontend, a Blazor WebAssembly admin UI, and a minimal REST API (PublicApi).

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
        BlazorWasm["Blazor WebAssembly Admin"]
    end
    subgraph Web["Presentation Layer - ASP.NET Core MVC / Razor Pages"]
        RazorPages["Razor Pages / MVC Views"]
        BlazorServer["Blazor Admin Server Host"]
        ApiControllers["Web API Controllers"]
    end
    subgraph PublicApi["Public API Layer - ASP.NET Core Minimal API"]
        Endpoints["Catalog / Auth / Brand / Type Endpoints"]
        Swagger["Swagger / OpenAPI UI"]
    end
    subgraph AppCore["Application Core - Domain + Services"]
        Entities["Domain Entities (Catalog, Basket, Order)"]
        Services["Application Services (Basket, Order)"]
        Interfaces["Repository + Service Interfaces"]
    end
    subgraph Infra["Infrastructure Layer - EF Core + Identity"]
        EFRepo["EfRepository (Ardalis Specification)"]
        CatalogCtx["CatalogContext (EF Core)"]
        IdentityCtx["AppIdentityDbContext (EF Core)"]
        EmailSvc["EmailSender Service"]
    end
    subgraph Data["Data Layer"]
        SqlServer[("SQL Server - Catalog DB")]
        IdentityDb[("SQL Server - Identity DB")]
    end
    subgraph External["External Services"]
        AzureKV["Azure Key Vault"]
        AzureIdentity["Azure Managed Identity"]
    end

    Browser -->|"HTTP/HTTPS"| RazorPages
    BlazorWasm -->|"REST calls"| ApiControllers
    BlazorWasm -->|"REST calls"| Endpoints
    RazorPages --> ApiControllers
    BlazorServer --> BlazorWasm
    ApiControllers -->|"delegates"| Services
    RazorPages -->|"delegates"| Services
    Endpoints -->|"delegates"| Services
    Services -->|"via interfaces"| Interfaces
    Interfaces -->|"implemented by"| EFRepo
    EFRepo -->|"queries"| CatalogCtx
    EFRepo -->|"queries"| IdentityCtx
    CatalogCtx -->|"SQL"| SqlServer
    IdentityCtx -->|"SQL"| IdentityDb
    Services -->|"sends email"| EmailSvc
    Web -->|"secrets (prod)"| AzureKV
    AzureKV -->|"auth"| AzureIdentity
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| Presentation (Web) | ASP.NET Core MVC + Razor Pages | .NET 8/9 | Server-side web UI for storefront |
| Presentation (Admin) | Blazor WebAssembly | .NET 8/9 | Client-side admin interface |
| Public API | ASP.NET Core Minimal API (Ardalis.ApiEndpoints) | .NET 8/9 | REST API for catalog and auth |
| Application Core | C# Domain Model + MediatR | Latest | Business logic, domain entities, interfaces |
| Data Access | Entity Framework Core + Ardalis.Specification | Latest | ORM, repository pattern |
| Identity | ASP.NET Core Identity + EF Core | Latest | Authentication and authorization |
| Database | SQL Server (LocalDB dev / Azure SQL prod) | - | Catalog and identity storage |
| Configuration (prod) | Azure Key Vault + Azure Identity | Latest | Secrets management |
| Mapping | AutoMapper | Latest | DTO ↔ entity mapping |

### Data Storage & External Services

The application uses two SQL Server databases: `CatalogDb` (stores catalog items, brands, and types, as well as basket and order data) and an Identity database (stores user accounts and roles via ASP.NET Core Identity). In development, SQL Server LocalDB is used; in production the connection strings are retrieved from Azure Key Vault via `DefaultAzureCredential` / `AzureDeveloperCliCredential`. There are no external caches or message brokers; email notifications are sent via a configurable `IEmailSender` abstraction backed by `EmailSender`.

### Key Architectural Decisions

- **Clean Architecture / Onion pattern**: `ApplicationCore` has no infrastructure dependencies; `Infrastructure` and `Web` depend on `ApplicationCore` interfaces, not concrete implementations.
- **Repository + Specification pattern**: `EfRepository<T>` (Ardalis.Specification.EntityFrameworkCore) provides a generic, testable data access layer driven by composable `Specification` objects.
- **Dual frontend strategy**: A traditional Razor Pages / MVC storefront coexists with a Blazor WebAssembly admin panel served from the same ASP.NET Core host, communicating via the PublicApi REST layer.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation"]
        HomeCtrl["HomeController"]
        OrderCtrl["OrderController"]
        BasketCtrl["BasketController (API)"]
        ManageCtrl["ManageController"]
        BlazorAdminPages["BlazorAdmin Pages"]
    end
    subgraph PublicApiLayer["Public API"]
        CatalogEndpoints["CatalogItemEndpoints"]
        AuthEndpoints["AuthEndpoints"]
        BrandEndpoints["CatalogBrandEndpoints"]
        TypeEndpoints["CatalogTypeEndpoints"]
    end
    subgraph BusinessLogic["Business Logic"]
        BasketSvc["BasketService"]
        OrderSvc["OrderService"]
        CatalogViewSvc["CatalogViewModelService"]
        BasketViewSvc["BasketViewModelService"]
        TokenClaimsSvc["IdentityTokenClaimService"]
    end
    subgraph DataAccess["Data Access"]
        EFRepo["EfRepository"]
        CatalogCtx["CatalogContext"]
        IdentityCtx["AppIdentityDbContext"]
        BasketQuery["BasketQueryService"]
    end
    subgraph Infrastructure["Infrastructure"]
        EmailSvc["EmailSender"]
        SeedData["CatalogContextSeed"]
    end

    HomeCtrl -->|"delegates"| CatalogViewSvc
    BasketCtrl -->|"delegates"| BasketSvc
    OrderCtrl -->|"delegates"| OrderSvc
    ManageCtrl -->|"delegates"| TokenClaimsSvc
    BlazorAdminPages -->|"HTTP calls"| CatalogEndpoints
    CatalogEndpoints -->|"queries"| EFRepo
    AuthEndpoints -->|"token"| TokenClaimsSvc
    BrandEndpoints -->|"queries"| EFRepo
    TypeEndpoints -->|"queries"| EFRepo
    BasketSvc -->|"reads/writes"| EFRepo
    OrderSvc -->|"reads/writes"| EFRepo
    OrderSvc -->|"sends"| EmailSvc
    CatalogViewSvc -->|"reads"| EFRepo
    BasketViewSvc -->|"reads"| BasketQuery
    BasketQuery -->|"queries"| CatalogCtx
    EFRepo -->|"uses"| CatalogCtx
    EFRepo -->|"uses"| IdentityCtx
    SeedData -.->|"seeds"| CatalogCtx
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|-----------|-------|------|----------------|
| HomeController | Presentation | MVC Controller | Renders catalog listing page |
| OrderController | Presentation | MVC Controller | Handles order creation and history |
| BasketController | Presentation | API Controller | REST basket operations for Blazor admin |
| ManageController | Presentation | MVC Controller | User account management |
| BlazorAdmin Pages | Presentation | Blazor WASM Components | Admin catalog item management UI |
| CatalogItemEndpoints | Public API | Minimal API Endpoint | CRUD for catalog items |
| AuthEndpoints | Public API | Minimal API Endpoint | JWT token issuance |
| CatalogBrandEndpoints | Public API | Minimal API Endpoint | Read catalog brands |
| CatalogTypeEndpoints | Public API | Minimal API Endpoint | Read catalog types |
| BasketService | Business Logic | Domain Service | Add/remove basket items, checkout |
| OrderService | Business Logic | Domain Service | Create orders from baskets |
| CatalogViewModelService | Business Logic | View Service | Paginated catalog view models |
| BasketViewModelService | Business Logic | View Service | Basket summary view models |
| IdentityTokenClaimService | Business Logic | Security Service | Generate JWT tokens for Identity users |
| EfRepository | Data Access | Generic Repository | EF Core CRUD + Ardalis Specification queries |
| CatalogContext | Data Access | DbContext | Catalog, basket, order entities |
| AppIdentityDbContext | Data Access | DbContext | ASP.NET Core Identity entities |
| BasketQueryService | Data Access | Query Service | Basket detail queries via EF Core |
| EmailSender | Infrastructure | Service | Sends transactional email via IEmailSender |
| CatalogContextSeed | Infrastructure | Seed Helper | Seeds initial catalog data on startup |
