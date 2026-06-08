# Architecture Diagram

This repository is a multi-project ASP.NET Core 8 solution centered on a single e-commerce application. It combines an MVC/Razor storefront, a minimal/API endpoint surface, shared domain/infrastructure libraries, and an embedded Blazor admin experience.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Shopper["Browser Storefront"]
        Admin["Blazor Admin Client"]
    end
    subgraph App["Application Layer - ASP.NET Core 8"]
        Web["Web MVC and Razor Pages"]
        Api["PublicApi Minimal API and Controllers"]
        Core["ApplicationCore Domain Services"]
    end
    subgraph Data["Data Layer"]
        Infra["Infrastructure EF Core Repositories"]
        CatalogDb[("Catalog SQL Server DB")]
        IdentityDb[("Identity SQL Server DB")]
        Cache["In Memory Cache"]
    end
    subgraph External["External Services"]
        KeyVault["Azure Key Vault"]
        AzureHost["Azure App Service via azd"]
    end

    Shopper -->|"HTTPS requests"| Web
    Admin -->|"HTTP API calls"| Api
    Web -->|"uses"| Core
    Api -->|"uses"| Core
    Core -->|"persists through"| Infra
    Infra -->|"catalog and basket data"| CatalogDb
    Infra -->|"identity data"| IdentityDb
    Web -->|"cached catalog lookups"| Cache
    Admin -->|"local item cache"| Cache
    Web -->|"production secrets"| KeyVault
    AzureHost -->|"hosts"| Web
    AzureHost -->|"hosts"| Api
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
|---|---|---:|---|
| Presentation | ASP.NET Core MVC, Razor Pages | 8.0.2 | Storefront UI, checkout, account and order pages |
| API | MinimalApi.Endpoint, Ardalis.ApiEndpoints, Swashbuckle | 1.3.0, 4.1.0, 6.5.0 | Catalog/admin API surface and Swagger |
| Client | Blazor WebAssembly, Blazored.LocalStorage | 8.0.2, 4.5.0 | Admin UI running inside the Web host |
| Domain | ApplicationCore, MediatR, Ardalis.Specification | custom, 12.0.1, 7.0.0 | Business rules, queries, order and basket orchestration |
| Data | EF Core SQL Server and InMemory | 8.0.2 | Repository-backed persistence and local/test fallback |
| Security | ASP.NET Core Identity, JWT bearer, Azure Key Vault config | 8.0.2, 8.0.2, 1.3.1 | User auth, admin API auth, secret resolution |

### Data Storage & External Services

The application uses two SQL Server-backed EF Core contexts: one for catalog, basket, and order data and one for ASP.NET Core Identity data. Runtime caching is local to the process through `IMemoryCache` in the storefront and browser local storage in the Blazor admin client, while production configuration can be supplemented from Azure Key Vault and Azure App Service deployment settings.

### Key Architectural Decisions

- Uses a modular monolith structure: deployable Web and PublicApi projects share ApplicationCore and Infrastructure rather than communicating through separate microservices.
- Applies repository plus specification patterns on top of EF Core to keep query logic outside controllers and page models.
- Keeps the admin experience as a Blazor WebAssembly client hosted by the Web app while routing catalog management operations through the PublicApi surface.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation
        WebPages["Razor Pages and MVC Controllers"]
        AdminUi["Blazor Admin Components"]
        ApiEndpoints["PublicApi Endpoints"]
    end
    subgraph Business["Business Logic"]
        BasketVm["BasketViewModelService"]
        CatalogVm["CatalogViewModelService"]
        BasketSvc["BasketService"]
        OrderSvc["OrderService"]
        Mediator["MediatR Handlers"]
    end
    subgraph DataAccess["Data Access"]
        Repo["EfRepository and IReadRepository"]
        BasketQuery["BasketQueryService"]
        Specs["Specification Classes"]
        DbCtx["CatalogContext and AppIdentityDbContext"]
    end
    subgraph Infra["Infrastructure"]
        Identity["Identity and Token Services"]
        Health["Health Checks"]
        MemoryCache["IMemoryCache"]
    end

    WebPages -->|"maps and renders"| BasketVm
    WebPages -->|"checkout and order commands"| BasketSvc
    WebPages -->|"order queries"| Mediator
    AdminUi -->|"catalog CRUD"| ApiEndpoints
    ApiEndpoints -->|"delegates"| Repo
    ApiEndpoints -->|"issues tokens"| Identity
    BasketVm -->|"reads"| Repo
    CatalogVm -->|"reads with filters"| Specs
    BasketSvc -->|"loads basket"| Specs
    BasketSvc -->|"updates basket"| Repo
    OrderSvc -->|"loads basket and catalog items"| Specs
    OrderSvc -->|"creates orders"| Repo
    Mediator -->|"reads order views"| Repo
    BasketQuery -->|"aggregates basket totals"| DbCtx
    Repo -->|"executes"| DbCtx
    Health -.->|"checks storefront and api"| WebPages
    Health -.->|"checks catalog endpoint"| ApiEndpoints
    CatalogVm -->|"caches catalog lists"| MemoryCache
    AdminUi -->|"caches item lists"| MemoryCache
```

### Component Inventory

| Component | Layer | Type | Responsibility |
|---|---|---|---|
| `Web` controllers and Razor pages | Presentation | MVC and page models | Handle catalog browsing, basket, account, and order flows |
| `PublicApi` endpoint classes | Presentation | Minimal API endpoints and controllers | Expose catalog lookup, catalog CRUD, and authentication contracts |
| `BasketService` | Business Logic | Domain service | Creates, updates, transfers, and deletes baskets |
| `OrderService` | Business Logic | Domain service | Validates baskets and creates orders from basket contents |
| `GetMyOrdersHandler` / `GetOrderDetailsHandler` | Business Logic | MediatR handlers | Read-side order projections for authenticated users |
| `EfRepository<T>` | Data Access | Repository | Generic aggregate persistence via EF Core and specifications |
| `BasketQueryService` | Data Access | Query service | Computes basket totals directly in SQL |
| `CatalogContext` | Data Access | DbContext | Owns catalog, basket, and order persistence |
| `AppIdentityDbContext` | Data Access | DbContext | Owns ASP.NET Core Identity persistence |
| `IdentityTokenClaimService` | Infrastructure | Security service | Generates JWT tokens for authenticated API users |
| `ApiHealthCheck` / `HomePageHealthCheck` | Infrastructure | Health checks | Validate storefront and API availability |
| `CachedCatalogViewModelService` | Infrastructure | Cache decorator | Caches catalog, brand, and type lookups in memory |
