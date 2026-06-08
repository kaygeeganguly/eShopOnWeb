# Architecture Diagram

This document summarizes the current eShopOnWeb architecture at two levels: high-level runtime structure and key component relationships.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Browser"]
        BlazorAdminUI["Blazor Admin UI"]
    end
    subgraph App["Application Layer - ASP.NET Core net8.0"]
        WebApp["Web MVC and Razor Pages"]
        PublicApi["Public API Minimal API and Controllers"]
        AppCore["ApplicationCore Services"]
    end
    subgraph Data["Data Layer - EF Core"]
        Repo["EfRepository and Specifications"]
        CatalogCtx["CatalogContext"]
        IdentityCtx["AppIdentityDbContext"]
        SqlDb[("SQL Server or LocalDB")]
        InMemoryDb[("EF InMemory for tests")]
        MemCache[("In-memory cache")]
    end
    subgraph External["External Services"]
        KeyVault["Azure Key Vault"]
    end

    Browser -->|"HTTPS"| WebApp
    BlazorAdminUI -->|"HTTP API calls"| PublicApi
    WebApp -->|"service calls"| AppCore
    PublicApi -->|"service and repository calls"| AppCore
    AppCore -->|"CRUD via repository"| Repo
    Repo -->|"EF Core operations"| CatalogCtx
    Repo -->|"Identity operations"| IdentityCtx
    CatalogCtx -->|"SQL"| SqlDb
    IdentityCtx -->|"SQL"| SqlDb
    CatalogCtx -->|"test profile"| InMemoryDb
    IdentityCtx -->|"test profile"| InMemoryDb
    WebApp -->|"cached catalog views"| MemCache
    PublicApi -->|"endpoint caching and auth cache"| MemCache
    WebApp -->|"loads secrets in non-dev"| KeyVault
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Presentation | ASP.NET Core MVC/Razor Pages | net8.0 | Customer web experience and account/order pages |
| API | ASP.NET Core Controllers + MinimalApi.Endpoint | net8.0 | Catalog/authentication APIs for web and Blazor admin |
| Business Logic | ApplicationCore services + specifications | net8.0 | Basket/order/catalog business rules and orchestration |
| Data Access | Entity Framework Core + Ardalis.Specification | EF Core 8.0.2 | Persistence and query abstraction |
| Security | ASP.NET Core Identity + JWT/Cookies | ASP.NET 8.0.2 | User auth, token issuance, role enforcement |

### Data Storage & External Services

The application persists catalog/order and identity data in SQL Server/LocalDB via `CatalogContext` and `AppIdentityDbContext`, with optional EF in-memory databases for test-like profiles. In non-development hosting, the web app resolves connection strings from Azure Key Vault. Both web and API layers also use in-memory cache for read-heavy UI and token-revocation scenarios.

### Key Architectural Decisions

- Uses a layered architecture with `ApplicationCore` for domain/business logic and `Infrastructure` for EF Core/identity implementations.
- Uses repository plus specification patterns (`EfRepository`, `CatalogFilterSpecification`, etc.) to centralize query logic.
- Supports multiple runtime modes: local development, Docker, and cloud with Key Vault-backed secret resolution.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation"]
        WebPages["Razor Pages and MVC Controllers"]
        ApiEndpoints["PublicApi Endpoints and Auth Controller"]
        OrderCtrl["OrderController"]
    end
    subgraph Business["Business Logic"]
        BasketSvc["BasketService"]
        OrderSvc["OrderService"]
        CatalogVmSvc["CatalogViewModelService"]
        CachedCatalogVmSvc["CachedCatalogViewModelService"]
        MediatRHandlers["MediatR Order Query Handlers"]
    end
    subgraph DataAccess["Data Access"]
        Repo["IRepository and EfRepository"]
        Specs["Specification classes"]
        CatalogCtx["CatalogContext"]
        IdentityCtx["AppIdentityDbContext"]
    end
    subgraph Infra["Infrastructure and Cross-Cutting"]
        AuthMw["Authentication and Authorization middleware"]
        HealthChecks["Health checks"]
        MemoryCache["IMemoryCache"]
        Logging["ILogger and IAppLogger"]
    end

    WebPages -->|"calls"| BasketSvc
    WebPages -->|"calls"| OrderSvc
    WebPages -->|"queries catalog"| CachedCatalogVmSvc
    CachedCatalogVmSvc -->|"cache miss delegates"| CatalogVmSvc
    ApiEndpoints -->|"catalog CRUD and list"| Repo
    ApiEndpoints -->|"authenticate user"| IdentityCtx
    OrderCtrl -->|"queries"| MediatRHandlers
    MediatRHandlers -->|"read orders"| Repo
    BasketSvc -->|"query and update"| Repo
    OrderSvc -->|"query and create"| Repo
    Repo -->|"applies filters"| Specs
    Repo -->|"persists"| CatalogCtx
    Repo -->|"identity persistence"| IdentityCtx
    MemoryCache -.->|"used by"| CachedCatalogVmSvc
    AuthMw -.->|"protects"| WebPages
    AuthMw -.->|"protects"| ApiEndpoints
    HealthChecks -.->|"monitors"| Presentation
    Logging -.->|"cross-cutting logs"| Business
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| `Program` (Web) | Presentation | Startup composition root | Configures middleware, data sources, auth, health checks |
| `Program` (PublicApi) | Presentation | Startup composition root | Configures API routes, JWT auth, swagger, endpoint registration |
| `OrderController` | Presentation | MVC Controller | Serves authenticated order history/detail views |
| `CatalogItem*Endpoint` classes | Presentation | Minimal API endpoints | Expose catalog list/get/create/update/delete APIs |
| `BasketService` | Business Logic | Domain service | Basket item add/update/delete and basket transfer |
| `OrderService` | Business Logic | Domain service | Creates orders from basket contents |
| `CatalogViewModelService` | Business Logic | UI service | Builds catalog listing/filter view models |
| `CachedCatalogViewModelService` | Business Logic | Decorator service | Wraps catalog view-model service with memory cache |
| `EfRepository<T>` | Data Access | Repository | Generic persistence operations using EF Core |
| `CatalogContext` / `AppIdentityDbContext` | Data Access | DbContext | Domain/identity persistence boundary |
