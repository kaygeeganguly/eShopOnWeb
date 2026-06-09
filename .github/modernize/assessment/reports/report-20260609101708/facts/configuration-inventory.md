# Configuration & Externalized Settings Inventory

eShopOnWeb externalizes configuration through layered ASP.NET Core settings files, launch profiles, Docker Compose environment variables, Azure deployment templates, and user-secret / Key Vault integration. The configuration model is simple but spans local development, Docker, and Azure production hosting.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Web base settings | JSON | `src/Web/appsettings.json` | Default base URLs, SQL Server LocalDB connection strings, logging |
| Web environment settings | JSON | `src/Web/appsettings.Development.json`, `src/Web/appsettings.Docker.json` | Overrides logging and Docker SQL/base URLs |
| PublicApi base settings | JSON | `src/PublicApi/appsettings.json` | Default base URLs, SQL Server LocalDB connection strings, logging |
| PublicApi environment settings | JSON | `src/PublicApi/appsettings.Development.json`, `src/PublicApi/appsettings.Docker.json` | Development logging and Docker SQL/base URL overrides |
| Blazor admin client settings | JSON | `src/BlazorAdmin/wwwroot/appsettings*.json` | Client-side base URL configuration for the hosted admin UI |
| Launch profiles | JSON | `src/Web/Properties/launchSettings.json`, `src/PublicApi/Properties/launchSettings.json`, `src/BlazorAdmin/Properties/launchSettings.json` | Define local URLs and `ASPNETCORE_ENVIRONMENT` |
| User secrets | ASP.NET Core secrets store | `UserSecretsId` in `Web.csproj` and `PublicApi.csproj` | Developer-only secret injection path |
| Docker Compose | YAML | `docker-compose.yml`, `docker-compose.override.yml` | Defines service startup order, Docker ports, SQL container, and environment variables |
| Azure deployment templates | Bicep / JSON | `infra/main.bicep`, `infra/main.parameters.json`, supporting Bicep modules | Defines App Service, Azure SQL, Key Vault, and secret output wiring |
| Azure developer metadata | YAML | `azure.yaml` | Ties the repo to Azure Developer CLI deployment conventions |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | Standard `dotnet build` / `dotnet test` default | Developer builds with diagnostics and local settings | Full project dependency graph |
| Release | `dotnet publish -c Release` and Docker builds | Production-oriented publish output | `BuildBundlerMinifier` is conditionally included for release in `Web.csproj` |
| Docker multi-stage images | `docker build` or `docker-compose build` | Produces container images for `Web` and `PublicApi` | Uses `mcr.microsoft.com/dotnet/sdk:8.0` and `aspnet:8.0` images |
| Central package management | Automatic via restore | Keeps package versions consistent across projects | `Directory.Packages.props` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Development | `ASPNETCORE_ENVIRONMENT=Development` from launch profiles | `appsettings.json` + `appsettings.Development.json` | Verbose logging, localhost base URLs |
| Docker | `ASPNETCORE_ENVIRONMENT=Docker` from Compose | `appsettings.json` + `appsettings.Docker.json` | SQL container connection strings, HTTP Docker base URLs |
| Production | `ASPNETCORE_ENVIRONMENT=Production` or Azure host default | `appsettings.json` plus environment variables and Key Vault | Azure Key Vault endpoint, Azure SQL connection string key names |

## Properties Inventory

### Web

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Development, Docker override | `appsettings.json`, `appsettings.Docker.json` |
| `baseUrls:webBase` | `https://localhost:44315/` | Development, Docker override | `appsettings.json`, `appsettings.Development.json`, `appsettings.Docker.json` |
| `ConnectionStrings:CatalogConnection` | LocalDB catalog connection | Docker override, production indirect | `appsettings.json`, `appsettings.Docker.json`, Key Vault indirection in production |
| `ConnectionStrings:IdentityConnection` | LocalDB identity connection | Docker override, production indirect | `appsettings.json`, `appsettings.Docker.json`, Key Vault indirection in production |
| `CatalogBaseUrl` | Empty string | All | `appsettings.json` |
| `UseOnlyInMemoryDatabase` | Not set | Optional toggle | Read from configuration in `Infrastructure/Dependencies.cs` |
| `Logging:LogLevel:*` | Warning / Debug mix | Development and Docker override | `appsettings*.json` |
| `AZURE_KEY_VAULT_ENDPOINT` | None | Production | Environment / App Service setting from Bicep |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | None | Production | Environment / App Service setting from Bicep |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | None | Production | Environment / App Service setting from Bicep |

### PublicApi

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Docker override | `appsettings.json`, `appsettings.Docker.json` |
| `baseUrls:webBase` | `https://localhost:5001/` | Docker override | `appsettings.json`, `appsettings.Development.json`, `appsettings.Docker.json` |
| `ConnectionStrings:CatalogConnection` | LocalDB catalog connection | Docker override | `appsettings.json`, `appsettings.Docker.json` |
| `ConnectionStrings:IdentityConnection` | LocalDB identity connection | Docker override | `appsettings.json`, `appsettings.Docker.json` |
| `Logging:LogLevel:*` | Warning / Information mix | Development and Docker override | `appsettings*.json` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| Web | `ASPNETCORE_ENVIRONMENT`, `ASPNETCORE_URLS` in launch profiles and Docker | No explicit limit in repo; Azure App Service plan SKU B1 in Bicep | 1 by default |
| PublicApi | `ASPNETCORE_ENVIRONMENT`, `ASPNETCORE_URLS` in launch profiles and Docker | No explicit limit in repo | 1 by default |
| SQL container | `SA_PASSWORD`, `ACCEPT_EULA` | No explicit limit in repo | 1 by default |

## Startup Dependency Chain

1. `sqlserver` starts first in Docker Compose; both `eshopwebmvc` and `eshoppublicapi` declare `depends_on` for it.
2. `Web` and `PublicApi` start, build their EF Core contexts, and attempt startup seeding with `Database.Migrate()` where SQL Server is active.
3. In production, `Web` also resolves Azure Key Vault configuration before building the SQL-backed contexts.
4. The Web host exposes `/health`, `/home_page_health_check`, and `/api_health_check` after startup completes.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `ConnectionStrings:CatalogConnection` | Database connection string | Local JSON or Key Vault backed value `[MASKED]` |
| `ConnectionStrings:IdentityConnection` | Database connection string | Local JSON or Key Vault backed value `[MASKED]` |
| `SA_PASSWORD` | SQL administrator password | Docker Compose environment value `[MASKED]` |
| `sqlAdminPassword` | Azure SQL admin secret | Generated and stored through deployment workflow `[MASKED]` |
| `appUserPassword` | Azure app user secret | Generated and stored through deployment workflow `[MASKED]` |
| `UserSecretsId` values | Developer secret indirection | ASP.NET Core user secrets store |

### Secrets Provisioning Workflow

Local development can use `appsettings.json`, Docker overrides, or ASP.NET Core user secrets for connection material. Azure deployment provisions Azure SQL databases and Azure Key Vault through Bicep, then injects the Key Vault endpoint plus connection-string key names into the Web app as App Service settings. At runtime the Web host authenticates with Azure developer credentials or the default Azure credential chain, resolves the Key Vault values, and builds its SQL Server contexts from those secret-backed settings.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| `UseOnlyInMemoryDatabase` | False / unset | Configuration property in Web or PublicApi settings |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET target framework | `net8.0` | `Directory.Packages.props` |
| ASP.NET Core packages | `8.0.2` | `Directory.Packages.props` |
| EF Core packages | `8.0.2` | `Directory.Packages.props` |
| MediatR | `12.0.1` | `Directory.Packages.props` |
| AutoMapper.Extensions.Microsoft.DependencyInjection | `12.0.1` | `Directory.Packages.props` |
| Swashbuckle.AspNetCore | `6.5.0` | `Directory.Packages.props` |
| Azure.Identity | `1.10.4` | `Directory.Packages.props` |
| Docker SDK image | `mcr.microsoft.com/dotnet/sdk:8.0` | `src/Web/Dockerfile`, `src/PublicApi/Dockerfile` |
| Docker runtime image | `mcr.microsoft.com/dotnet/aspnet:8.0` | `src/Web/Dockerfile`, `src/PublicApi/Dockerfile` |
| .NET SDK roll-forward | `latestFeature` | `global.json` |
