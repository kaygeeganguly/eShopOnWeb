# Dependency Map

The eShopOnWeb solution centrally manages 42 production dependencies and 8 test dependencies through `Directory.Packages.props`. The main dependency surface is typical for an ASP.NET Core 8 modular monolith with EF Core, Identity, Swagger, Blazor, and a small set of supporting libraries.

## Dependencies

```mermaid
flowchart LR
    App["eShopOnWeb"]

    subgraph Web["Web Frameworks"]
        AspNet["ASP.NET Core MVC and Identity UI 8.0.2"]
        Blazor["Blazor WebAssembly 8.0.2"]
        MinimalApi["MinimalApi.Endpoint 1.3.0"]
        ApiEp["Ardalis.ApiEndpoints 4.1.0"]
    end
    subgraph DB["Database / ORM"]
        EfSql["EF Core SQL Server 8.0.2"]
        EfMem["EF Core InMemory 8.0.2"]
        ArdalisEf["Ardalis.Specification.EntityFrameworkCore 7.0.0"]
    end
    subgraph Sec["Security"]
        Jwt["JWT ******"]
        Tokens["System.IdentityModel.Tokens.Jwt 7.3.1"]
        AzureId["Azure.Identity 1.10.4"]
        KeyVault["Azure Key Vault Config 1.3.1"]
    end
    subgraph Obs["Observability"]
        Diagnostics["Diagnostics EF Core 8.0.2"]
        Logging["Logging Configuration 8.0.0"]
        Startup["Ardalis.ListStartupServices 1.1.4"]
    end
    subgraph Util["Utilities"]
        AutoMapper["AutoMapper DI 12.0.1"]
        MediatR["MediatR 12.0.1"]
        Guard["GuardClauses 4.0.1"]
        Result["Ardalis.Result 7.0.0"]
        Fluent["FluentValidation 11.9.0"]
    end
    subgraph Client["Client and UI Helpers"]
        LocalStore["Blazored.LocalStorage 4.5.0"]
        InputFile["BlazorInputFile 0.2.0"]
    end
    subgraph Tooling["Build and Tooling"]
        CodeGen["Web Code Generation 8.0.0"]
        LibMan["LibraryManager.Build 2.1.175"]
        Bundler["BuildBundlerMinifier 3.2.449"]
        Containers["Azure Containers Targets 1.19.6"]
    end

    App -->|"web"| Web
    App -->|"persistence"| DB
    App -->|"security"| Sec
    App -->|"observability"| Obs
    App -->|"utilities"| Util
    App -->|"client"| Client
    App -->|"tooling"| Tooling
    AspNet -.->|"swagger support"| ApiEp
    Jwt -.->|"token handling"| Tokens
    EfSql -.->|"repository integration"| ArdalisEf
```

### Dependency Summary

| Category | Count | Key Libraries | Notes |
|---|---:|---|---|
| Web Frameworks | 4 | ASP.NET Core MVC/Identity UI, Blazor WebAssembly, MinimalApi.Endpoint, Ardalis.ApiEndpoints | Supports storefront UI, admin client, and API endpoints |
| Database / ORM | 3 | EF Core SQL Server, EF Core InMemory, Ardalis.Specification.EntityFrameworkCore | SQL Server is the main persistence path; in-memory is used for local/test scenarios |
| Security | 4 | Azure.Identity, Azure Key Vault config, JwtBearer, System.IdentityModel.Tokens.Jwt | Combines Azure secret access with JWT-based API auth |
| Observability | 3 | Diagnostics.EntityFrameworkCore, Logging.Configuration, ListStartupServices | Mostly diagnostics and service registration visibility |
| Utilities | 5 | AutoMapper, MediatR, GuardClauses, Ardalis.Result, FluentValidation | Shared application plumbing and validation helpers |
| Client and UI Helpers | 2 | Blazored.LocalStorage, BlazorInputFile | Support client-side admin caching and file uploads |
| Build and Tooling | 4 | Web Code Generation, LibraryManager, BuildBundlerMinifier, Azure Containers Targets | Developer tooling and container-oriented build support |

### Version & Compatibility Risks

The solution targets `net8.0`, which is current, but the baseline restore already flagged `Azure.Identity` 1.10.4 and `System.Text.Json` 8.0.3 for known vulnerabilities. The dependency set also mixes newer ASP.NET Core 8 packages with older helper packages such as `Microsoft.AspNetCore.Mvc` 2.2.0 and `BlazorInputFile` 0.2.0, which are worth reviewing during future modernization or dependency refresh work.

### Notable Observations

- Package versions are managed centrally in `Directory.Packages.props`, which simplifies coordinated upgrades across all projects.
- Both `Microsoft.EntityFrameworkCore.SqlServer` and `Microsoft.EntityFrameworkCore.InMemory` are first-class dependencies, reflecting the split between persistent and lightweight local/test execution paths.
- The repository uses two endpoint styles side-by-side in `PublicApi`: Minimal API endpoints and `Ardalis.ApiEndpoints` controller-style endpoints.
- Security-sensitive dependencies are concentrated in the web/API tier rather than isolated into a gateway or separate auth service.

## Test Dependencies

| Framework | Version | Notes |
|---|---:|---|
| xUnit | 2.7.0 | Primary unit, integration, and functional test framework |
| xUnit runner visualstudio | 2.5.6 | IDE and CLI runner integration |
| xUnit runner console | 2.7.0 | Console execution support |
| MSTest.TestFramework / Adapter | 3.2.2 | Used by PublicApi integration tests |
| Microsoft.AspNetCore.Mvc.Testing | 8.0.2 | Boots ASP.NET Core apps in integration/functional tests |
| NSubstitute | 5.1.0 | Test doubles and interaction assertions |
| coverlet.collector | 6.0.2 | Code coverage collection |
| Microsoft.NET.Test.Sdk | 17.9.0 | Test host and runner plumbing |

Total test-scope dependencies: 8

The test stack is mature and already covers unit, integration, functional, and API-hosted scenarios. There is no separate contract-testing framework, but the existing ASP.NET Core test host coverage is strong for this repository structure.
