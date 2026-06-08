# Dependency Map

This .NET solution uses centrally managed NuGet packages with a moderate dependency surface across web, API, data, security, and utility areas. The map below highlights declared non-test dependencies by category.

## Dependencies

```mermaid
flowchart LR
    App["eShopOnWeb Solution"]

    subgraph Web["Web Frameworks"]
        AspNet["ASP.NET Core 8.0.2"]
        MinimalApi["MinimalApi.Endpoint 1.3.0"]
        Blazor["Blazor WebAssembly 8.0.2"]
    end
    subgraph DB["Database and ORM"]
        EFCore["EF Core SqlServer 8.0.2"]
        EFInMemory["EF Core InMemory 8.0.2"]
        IdentityEF["Identity EF Core 8.0.2"]
    end
    subgraph Security["Security"]
        Jwt["JwtBearer 8.0.2"]
        JwtToken["System.IdentityModel.Tokens.Jwt 7.3.1"]
        AzureId["Azure.Identity 1.10.4"]
    end
    subgraph Logging["Logging"]
        ExtLogging["Microsoft.Extensions.Logging.Configuration 8.0.0"]
    end
    subgraph Utils["Utilities"]
        ArdalisSpec["Ardalis.Specification 7.0.0"]
        ArdalisResult["Ardalis.Result 7.0.0"]
        AutoMapper["AutoMapper DI 12.0.1"]
        MediatR["MediatR 12.0.1"]
        Swagger["Swashbuckle 6.5.0"]
    end

    App -->|"web"| Web
    App -->|"persistence"| DB
    App -->|"auth"| Security
    App -->|"logging"| Logging
    App -->|"utilities"| Utils
```

### Dependency Summary

| Category | Count | Key Libraries | Notes |
|---|---:|---|---|
| Web Frameworks | 3 | ASP.NET Core, MinimalApi.Endpoint, Blazor | Main HTTP/UI stack for MVC, API, and admin UI |
| Database / ORM | 3 | EF Core SqlServer, EF Core InMemory, Identity EF Core | SQL Server primary persistence with in-memory option |
| Security | 3 | JwtBearer, System.IdentityModel.Tokens.Jwt, Azure.Identity | JWT auth plus optional Key Vault integration |
| Logging | 1 | Microsoft.Extensions.Logging.Configuration | Standard ASP.NET Core logging configuration |
| Utilities | 5 | Ardalis.*, AutoMapper, MediatR, Swashbuckle | Domain patterns, mapping, mediation, API documentation |

### Version & Compatibility Risks

The solution targets .NET 8 and uses package versions mostly aligned to 8.0.x, but baseline test output already flags known advisories in `System.Text.Json` 8.0.3 and `Azure.Identity` 1.10.4. These should be reviewed during modernization and dependency refresh.

### Notable Observations

- Versions are centrally managed in `Directory.Packages.props`, reducing drift across projects.
- Both SQL Server and InMemory providers are declared, enabling production and test/dev persistence modes.
- Public API uses both controller-based endpoints and Minimal API endpoint classes.
- Swagger/annotations dependencies are explicitly included for API contract visibility.

## Test Dependencies

| Framework | Version | Notes |
|---|---|---|
| xUnit | 2.7.0 | Primary unit and integration test framework |
| xunit.runner.visualstudio | 2.5.6 | Visual Studio test runner integration |
| xunit.runner.console | 2.7.0 | Console execution support |
| Microsoft.NET.Test.Sdk | 17.9.0 | .NET test hosting infrastructure |
| MSTest.TestFramework / Adapter | 3.2.2 | Additional test compatibility packages |
| coverlet.collector | 6.0.2 | Code coverage collection |
| NSubstitute | 5.1.0 | Mocking framework in test projects |

Total test-scope dependencies: 7

The test stack is complete for unit, functional, and integration test execution and includes mocking and coverage tooling.
