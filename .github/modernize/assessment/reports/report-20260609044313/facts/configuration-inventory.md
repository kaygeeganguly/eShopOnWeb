# Configuration & Externalized Settings Inventory

eShopOnWeb uses **four configuration sources** per service (base appsettings, environment-specific overrides, Docker overrides, and environment variables/User Secrets), with Azure Key Vault integration for production secrets management.

## Configuration Sources

| Source | Type | Path / Location | Notes |
|---|---|---|---|
| `appsettings.json` | JSON file | `src/Web/appsettings.json`, `src/PublicApi/appsettings.json`, `src/BlazorAdmin/wwwroot/appsettings.json` | Base defaults for all environments |
| `appsettings.Development.json` | JSON file | `src/Web/`, `src/PublicApi/`, `src/BlazorAdmin/wwwroot/` | Development overrides (verbose logging, local URLs) |
| `appsettings.Docker.json` | JSON file | `src/Web/`, `src/PublicApi/`, `src/BlazorAdmin/wwwroot/` | Docker Compose overrides (SQL Server container connection strings, `host.docker.internal` URLs) |
| `appsettings.test.json` | JSON file | `tests/PublicApiIntegrationTests/` | Test-only: sets `UseOnlyInMemoryDatabase=true` |
| User Secrets | .NET User Secrets | IDs: Web=`aspnet-Web2-...`, PublicApi=`5b662463-...` | Local dev secrets (never committed); loaded when `ASPNETCORE_ENVIRONMENT=Development` |
| Environment Variables | OS / Docker Compose | `docker-compose.override.yml` env sections | Override any config key at runtime |
| Azure Key Vault | Secret store | URI from env var `AZURE_KEY_VAULT_ENDPOINT` | Production secrets; loaded when environment is not Development or Docker |
| `launchSettings.json` | IDE profile | `src/Web/Properties/`, `src/PublicApi/Properties/`, `src/BlazorAdmin/Properties/` | Dev-only; sets `ASPNETCORE_ENVIRONMENT` and launch URLs |
| `docker-compose.yml` + `docker-compose.override.yml` | Container config | Repository root | Service definitions, port mappings, SQL Server container |

## Build Profiles

| Profile | Activation | Purpose | Key Differences |
|---|---|---|---|
| Debug | Default in Visual Studio / `dotnet build` | Development build with debug symbols | `ASPNETCORE_ENVIRONMENT=Development`, no optimizations |
| Release | `-c Release` / `dotnet publish` | Production/Docker image build | Optimized, minified; used in all Dockerfiles |
| `BuildBundlerMinifier` | Release only (conditional `PackageReference`) | CSS/JS bundling and minification | Only active in Release builds (`Condition="'$(Configuration)'=='Release'"`) |

No MSBuild custom profiles or conditional compilation symbols beyond Debug/Release are defined.

## Runtime Profiles

| Profile | Activation Method | Config Files Loaded | Key Overrides |
|---|---|---|---|
| Development | `ASPNETCORE_ENVIRONMENT=Development` | `appsettings.json` + `appsettings.Development.json` + User Secrets | LocalDB connection strings, verbose logging, developer exception page |
| Docker | `ASPNETCORE_ENVIRONMENT=Docker` | `appsettings.json` + `appsettings.Docker.json` | SQL Server container connection strings (sa user), `host.docker.internal` URLs, plain HTTP |
| Production | `ASPNETCORE_ENVIRONMENT=Production` (any non-Dev, non-Docker value) | `appsettings.json` + env vars + Azure Key Vault | Azure SQL connection strings from Key Vault, `ChainedTokenCredential` for managed identity |
| Test | `UseOnlyInMemoryDatabase=true` in `appsettings.test.json` | `appsettings.json` + `appsettings.test.json` | In-memory EF Core, no SQL Server dependency |

## Properties Inventory

### Web (MVC Storefront)

| Property Key | Default Value | Development Override | Docker Override | Source |
|---|---|---|---|---|
| `ConnectionStrings:CatalogConnection` | `Server=(localdb)\mssqllocaldb;...CatalogDb` | Same (LocalDB) | `Server=sqlserver,1433;...sa;****** | appsettings.json / appsettings.Docker.json |
| `ConnectionStrings:IdentityConnection` | `Server=(localdb)\mssqllocaldb;...Identity` | Same (LocalDB) | `Server=sqlserver,1433;...sa;****** | appsettings.json / appsettings.Docker.json |
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Same | `http://localhost:5200/api/` | appsettings.json / appsettings.Docker.json |
| `baseUrls:webBase` | `https://localhost:44315/` | Same | `http://host.docker.internal:5106/` | appsettings.json / appsettings.Docker.json |
| `CatalogBaseUrl` | `""` | `""` | `""` | appsettings.json |
| `AZURE_KEY_VAULT_ENDPOINT` | — | — | — | Environment variable (Production only) |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | — | — | — | Environment variable (Production only); points to Key Vault secret name |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | — | — | — | Environment variable (Production only); points to Key Vault secret name |
| `Logging:LogLevel:Default` | `Warning` | `Warning` | `Debug` | appsettings.json |
| `ASPNETCORE_ENVIRONMENT` | — | `Development` | `Docker` | launchSettings.json / docker-compose.override.yml |
| `ASPNETCORE_URLS` | — | — | `http://+:8080` | docker-compose.override.yml |

### PublicApi (REST API)

| Property Key | Default Value | Development Override | Docker Override | Source |
|---|---|---|---|---|
| `ConnectionStrings:CatalogConnection` | `Server=(localdb)\mssqllocaldb;...CatalogDb` | Same | `Server=sqlserver,1433;...sa;****** | appsettings.json / appsettings.Docker.json |
| `ConnectionStrings:IdentityConnection` | `Server=(localdb)\mssqllocaldb;...Identity` | Same | `Server=sqlserver,1433;...sa;****** | appsettings.json / appsettings.Docker.json |
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Same | `http://localhost:5200/api/` | appsettings.json / appsettings.Docker.json |
| `baseUrls:webBase` | `https://localhost:5001/` | Same | `http://host.docker.internal:5106/` | appsettings.json / appsettings.Docker.json |
| `CatalogBaseUrl` | `""` | `""` | `""` | appsettings.json |
| `Logging:LogLevel:Default` | `Warning` | `Information` | `Information` | appsettings.json |
| `ASPNETCORE_ENVIRONMENT` | — | `Development` | `Docker` | launchSettings.json / docker-compose.override.yml |

### BlazorAdmin (Blazor WebAssembly)

| Property Key | Default Value | Development Override | Docker Override | Source |
|---|---|---|---|---|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Same | `http://localhost:5200/api/` | appsettings.json / appsettings.Docker.json |
| `baseUrls:webBase` | `https://localhost:44315/` | Same | `http://host.docker.internal:5106/` | appsettings.json / appsettings.Docker.json |

### Test Projects

| Property Key | Value | Source |
|---|---|---|
| `UseOnlyInMemoryDatabase` | `true` | `tests/PublicApiIntegrationTests/appsettings.test.json` |

## Startup Parameters & Resource Requirements

| Service | Runtime | URL / Port | Docker Port Mapping | Memory Limit | Notes |
|---|---|---|---|---|---|
| Web (MVC) | `dotnet Web.dll` | https://localhost:5001, http://localhost:5000 | 5106:8080 | Not configured | `ASPNETCORE_URLS=http://+:8080` in Docker |
| PublicApi | `dotnet PublicApi.dll` | https://localhost:5099, http://localhost:5098 | 5200:8080 | Not configured | `ASPNETCORE_URLS=http://+:8080` in Docker |
| SQL Server | `azure-sql-edge` container | 1433 | 1433:1433 | Not configured | SA password set via `SA_PASSWORD` env var |

No JVM heap settings (not applicable — .NET runtime). No Kubernetes resource limits configured.

## Startup Dependency Chain

```
sqlserver (SQL Server container)
  ↑  depends_on (Docker Compose declaration only — no health-check wait)
  ├── eshopwebmvc   (Web MVC)
  └── eshoppublicapi (PublicApi)
```

- Docker Compose `depends_on` is declared but **no `condition: service_healthy`** is configured, meaning containers start without waiting for SQL Server readiness.
- The Web project registers two custom `IHealthCheck` implementations (`ApiHealthCheck`, `HomePageHealthCheck`) exposed at `/health`, `/home_page_health_check`, `/api_health_check` — these can serve as readiness indicators but are not wired into Docker Compose startup.
- No Kubernetes readiness probes, `dockerize`, or wait-for-TCP mechanisms are configured.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage | Environment |
|---|---|---|---|
| `ConnectionStrings:CatalogConnection` (SQL Server sa password) | Database credential | `appsettings.Docker.json` (hardcoded in repo) | Docker only |
| `ConnectionStrings:IdentityConnection` (SQL Server sa password) | Database credential | `appsettings.Docker.json` (hardcoded in repo) | Docker only |
| `AZURE_KEY_VAULT_ENDPOINT` | Azure Key Vault URI | Environment variable | Production |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | Key Vault secret name pointer | Environment variable | Production |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | Key Vault secret name pointer | Environment variable | Production |
| JWT signing key (`AuthorizationConstants.JWT_SECRET_KEY`) | Symmetric signing key | Compiled into `ApplicationCore.Constants` | All environments |
| User Secrets (Web, PublicApi) | Any local overrides | `~/.microsoft/usersecrets/` (local machine only) | Development |

> ⚠️ The Docker sa password (`@someThingComplicated1234`) is hardcoded in `appsettings.Docker.json` which is committed to the repository. This should only be used for local Docker development — never for production workloads.

> ⚠️ The JWT secret key (`AuthorizationConstants.JWT_SECRET_KEY`) is a compile-time constant. It must be rotated and externalized to a secret store before production deployment.

### Secrets Provisioning Workflow

**Development**: Connection strings use Windows Integrated Security against LocalDB (no password). Sensitive overrides go into .NET User Secrets (stored at `~/.microsoft/usersecrets/<UserSecretsId>`, never committed).

**Docker (local)**: Connection strings with the SA password are in `appsettings.Docker.json`. The Docker Compose override mounts `~/.microsoft/usersecrets` and `~/.aspnet/https` volumes into the containers, enabling local HTTPS certificates and User Secret overrides.

**Production (Azure)**: The Web service uses `ChainedTokenCredential` (AzureDeveloperCliCredential → DefaultAzureCredential) to authenticate to Azure Key Vault. The Key Vault URI is supplied via `AZURE_KEY_VAULT_ENDPOINT` environment variable. Connection string secret names are supplied via `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` and `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` environment variables. The Key Vault secrets themselves hold the full SQL connection strings for Azure SQL. A Managed Identity (system-assigned) must have `get` and `list` Key Vault secret permissions assigned.

## Feature Flags

| Flag | Default | Controlled By | Effect |
|---|---|---|---|
| `UseOnlyInMemoryDatabase` | `false` (absent = SQL Server) | `appsettings.test.json` or environment variable | When `true`, forces EF Core to use the InMemory provider instead of SQL Server |

No feature flag framework (LaunchDarkly, .NET Microsoft.FeatureManagement, custom toggles) is configured beyond this single boolean configuration switch.

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET SDK | 8.0.x (latest feature) | `global.json` (`rollForward: latestFeature`) |
| ASP.NET Core | 8.0.2 | `Directory.Packages.props` `AspNetVersion` |
| Entity Framework Core | 8.0.2 | `Directory.Packages.props` `EntityFramworkCoreVersion` |
| Blazor WebAssembly | 8.0.2 | Same as ASP.NET Core |
| Target Framework | net8.0 | `Directory.Packages.props` / each `.csproj` |
| C# Language Version | `latest` (CS 12 with .NET 8 SDK) | `Web.csproj` `LangVersion` |
| Docker base image (runtime) | `mcr.microsoft.com/dotnet/aspnet:8.0` | `src/PublicApi/Dockerfile`, `src/Web/Dockerfile` |
| Docker base image (build) | `mcr.microsoft.com/dotnet/sdk:8.0` | `src/PublicApi/Dockerfile`, `src/Web/Dockerfile` |
| SQL Server (Docker) | `mcr.microsoft.com/azure-sql-edge` (latest tag) | `docker-compose.yml` |
| Ardalis.Specification | 7.0.0 | `Directory.Packages.props` |
| MediatR | 12.0.1 | `Directory.Packages.props` |
| AutoMapper | 12.0.1 | `Directory.Packages.props` |
| Swashbuckle.AspNetCore | 6.5.0 | `Directory.Packages.props` |
| Azure.Identity | 1.10.4 | `Directory.Packages.props` |
