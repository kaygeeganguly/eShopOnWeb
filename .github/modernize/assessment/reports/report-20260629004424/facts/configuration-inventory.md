# Configuration & Externalized Settings Inventory

eShopOnWeb uses three configuration sources per service (base `appsettings.json`, environment-specific overrides, and environment variables), with Azure Key Vault for production secrets and Docker-specific profiles for containerized deployments.

## Configuration Sources

| Source | Type | Path / Location | Notes |
|--------|------|-----------------|-------|
| appsettings.json | JSON file | src/Web/, src/PublicApi/, src/BlazorAdmin/wwwroot/ | Base configuration for each service |
| appsettings.Development.json | JSON file (env override) | src/Web/, src/PublicApi/, src/BlazorAdmin/wwwroot/ | Development overrides; verbose logging |
| appsettings.Docker.json | JSON file (env override) | src/Web/, src/PublicApi/, src/BlazorAdmin/wwwroot/ | Docker SQL Server connection strings; internal hostnames |
| launchSettings.json | IDE launch profile | src/Web/Properties/, src/PublicApi/Properties/, src/BlazorAdmin/Properties/ | Developer launch profiles; not deployed |
| docker-compose.yml | Docker Compose | ./docker-compose.yml | Service definitions; SQL Server container |
| docker-compose.override.yml | Docker Compose | ./docker-compose.override.yml | Port mappings; volume mounts for HTTPS certs and user secrets |
| Environment variables | OS / Container env | Runtime | `ASPNETCORE_ENVIRONMENT`, `ASPNETCORE_URLS`, `AZURE_KEY_VAULT_ENDPOINT`, `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY`, `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` |
| Azure Key Vault | Secret store | URI from `AZURE_KEY_VAULT_ENDPOINT` env var | Production only; loaded via `Azure.Extensions.AspNetCore.Configuration.Secrets` |
| .NET User Secrets | Local secret store | `~/.microsoft/usersecrets/{UserSecretsId}` | Development only; mounted into Docker via volume |
| appsettings.test.json | JSON file (test) | tests/PublicApiIntegrationTests/ | Test overrides; sets `UseOnlyInMemoryDatabase: true` |

## Build Profiles

| Profile | Activation | Purpose | Key Effects |
|---------|-----------|---------|-------------|
| Debug | Default in Visual Studio / `dotnet build` | Development build | Full debug symbols; no optimization; `BuildBundlerMinifier` excluded |
| Release | `-c Release` flag / `dotnet publish` | Production artifact | Optimized IL; `BuildBundlerMinifier` runs (CSS/JS bundling); publish output trimmed |
| Docker (Dockerfile) | `docker build` / `docker-compose build` | Container image build | Multi-stage build: `sdk:8.0` for build, `aspnet:8.0` for runtime |

The `BuildBundlerMinifier` package is conditioned on `'$(Configuration)'=='Release'` in `Web.csproj`, so CSS/JS bundling only runs during Release builds.

## Runtime Profiles

| Profile | Activation Method | Config Files Loaded | Key Overrides |
|---------|-----------------|---------------------|---------------|
| Development | `ASPNETCORE_ENVIRONMENT=Development` (launchSettings default) | appsettings.json → appsettings.Development.json | Log level Default=Debug/Information; developer exception page |
| Docker | `ASPNETCORE_ENVIRONMENT=Docker` (docker-compose.override.yml) | appsettings.json → appsettings.Docker.json | SQL Server connection strings point to `sqlserver:1433`; base URLs use `host.docker.internal` |
| Production | `ASPNETCORE_ENVIRONMENT=Production` (explicit; "Web - PROD" launch profile) | appsettings.json | Azure Key Vault loaded; Azure SQL connection strings from Key Vault secrets; `ChainedTokenCredential` used |
| Test | `UseOnlyInMemoryDatabase=true` in appsettings.test.json | appsettings.test.json | Both DbContexts use EF Core InMemory provider; no SQL Server required |

ASP.NET Core's configuration system applies files in layered order: `appsettings.json` → `appsettings.{Environment}.json` → environment variables → user secrets (Development). Later sources override earlier ones.

## Properties Inventory

### Web Service (`src/Web/appsettings*.json`)

| Property Key | Default | Docker Override | Source |
|-------------|---------|----------------|--------|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | `http://localhost:5200/api/` | appsettings.json / Docker override |
| `baseUrls:webBase` | `https://localhost:44315/` | `http://host.docker.internal:5106/` | appsettings.json / Docker override |
| `ConnectionStrings:CatalogConnection` | `Server=(localdb)\mssqllocaldb;...CatalogDb` | `Server=sqlserver,1433;...User Id=sa;****** | appsettings.json / Docker override |
| `ConnectionStrings:IdentityConnection` | `Server=(localdb)\mssqllocaldb;...Identity` | `Server=sqlserver,1433;...User Id=sa;****** | appsettings.json / Docker override |
| `CatalogBaseUrl` | `""` (empty) | — | appsettings.json |
| `AZURE_KEY_VAULT_ENDPOINT` | — (required in Production) | — | Environment variable |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | — (required in Production) | — | Environment variable (Key Vault key name) |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | — (required in Production) | — | Environment variable (Key Vault key name) |
| `Logging:LogLevel:Default` | `Warning` | `Debug` | appsettings.json / Docker override |

### PublicApi Service (`src/PublicApi/appsettings*.json`)

| Property Key | Default | Docker Override | Source |
|-------------|---------|----------------|--------|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | `http://localhost:5200/api/` | appsettings.json / Docker override |
| `baseUrls:webBase` | `https://localhost:5001/` | `http://host.docker.internal:5106/` | appsettings.json / Docker override |
| `ConnectionStrings:CatalogConnection` | `Server=(localdb)\mssqllocaldb;...CatalogDb` | `Server=sqlserver,1433;...User Id=sa;****** | appsettings.json / Docker override |
| `ConnectionStrings:IdentityConnection` | `Server=(localdb)\mssqllocaldb;...Identity` | `Server=sqlserver,1433;...User Id=sa;****** | appsettings.json / Docker override |
| `CatalogBaseUrl` | `""` (empty) | — | appsettings.json |
| `Logging:LogLevel:Default` | `Warning` | `Information` | appsettings.json / Docker override |

### BlazorAdmin Client (`src/BlazorAdmin/wwwroot/appsettings*.json`)

| Property Key | Default | Development Override | Docker Override | Source |
|-------------|---------|--------------------|-----------------|----|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | `https://localhost:5099/api/` | `http://localhost:5200/api/` | appsettings.json |
| `baseUrls:webBase` | `https://localhost:44315/` | `https://localhost:44315/` | `http://host.docker.internal:5106/` | appsettings.json |
| `Logging:LogLevel:Default` | `Information` | `Information` | — | appsettings.json |

### Test Overrides (`tests/PublicApiIntegrationTests/appsettings.test.json`)

| Property Key | Value | Source |
|-------------|-------|--------|
| `UseOnlyInMemoryDatabase` | `true` | appsettings.test.json |

## Startup Parameters & Resource Requirements

| Service | Runtime | Environment Variable | Ports | Notes |
|---------|---------|----------------------|-------|-------|
| Web (eshopwebmvc) | `dotnet Web.dll` | `ASPNETCORE_ENVIRONMENT=Docker`, `ASPNETCORE_URLS=http://+:8080` | Host 5106 → Container 8080 | HTTPS cert via volume mount |
| PublicApi (eshoppublicapi) | `dotnet PublicApi.dll` | `ASPNETCORE_ENVIRONMENT=Docker`, `ASPNETCORE_URLS=http://+:8080` | Host 5200 → Container 8080 | HTTPS cert via volume mount |
| SQL Server | `mcr.microsoft.com/azure-sql-edge` | `SA_PASSWORD=[MASKED]`, `ACCEPT_EULA=Y` | Host 1433 → Container 1433 | Azure SQL Edge image (ARM64 compatible) |

No JVM heap settings (not applicable). No explicit memory or CPU limits are configured in `docker-compose.yml`. Instance count is 1 per service (no horizontal scaling configuration present).

## Startup Dependency Chain

```
sqlserver
  └── eshopwebmvc (depends_on: sqlserver)
  └── eshoppublicapi (depends_on: sqlserver)
```

- `docker-compose.yml` declares `depends_on: sqlserver` for both application services.
- No health-check condition is configured on `depends_on` (Docker Compose `condition: service_healthy` is not used), meaning the application containers start as soon as the SQL Server container starts — not when SQL Server is ready to accept connections. This can cause connection failures during first startup; the application relies on EF Core's `EnableRetryOnFailure()` (in Production mode) to retry.
- In Development mode (without Docker), no explicit startup order mechanism is used. The application will fail on startup if SQL Server LocalDB is unavailable.
- No Spring Cloud Config server, Kubernetes readiness probes, or dockerize wait-for-TCP patterns are present.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage | Profile |
|-----------------|------|---------|---------|
| `ConnectionStrings:CatalogConnection` (password) | SQL Server SA password | `[MASKED]` in appsettings.Docker.json | Docker |
| `ConnectionStrings:IdentityConnection` (password) | SQL Server SA password | `[MASKED]` in appsettings.Docker.json | Docker |
| `SA_PASSWORD` | SQL Server SA password | docker-compose.yml environment variable `[MASKED]` | Docker |
| `AZURE_KEY_VAULT_ENDPOINT` | Key Vault URI | Environment variable | Production |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | Key Vault secret name (key for catalog connection string) | Environment variable | Production |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | Key Vault secret name (key for identity connection string) | Environment variable | Production |
| `AuthorizationConstants.JWT_SECRET_KEY` | JWT HMAC signing key | Hardcoded constant in source code (`AuthorizationConstants.cs`) | All |
| `AuthorizationConstants.DEFAULT_PASSWORD` | Default seed user password | Hardcoded constant in source code (`AuthorizationConstants.cs`) | Development/seed |
| `UserSecretsId` (Web) | ASP.NET User Secrets | `aspnet-Web2-1FA3F72E-E7E3-4360-9E49-1CCCD7FE85F7` | Development |

### Secrets Provisioning Workflow

**Development**: Developers use .NET User Secrets (`dotnet user-secrets set`) or override properties in `appsettings.Development.json`. User secrets are stored in `~/.microsoft/usersecrets/{UserSecretsId}` and are never committed to source control. In Docker Development mode, the user secrets folder is volume-mounted into the container.

**Docker**: Connection strings (including the SQL Server SA password `@someThingComplicated1234`) are hardcoded in `appsettings.Docker.json` and committed to source control — **this is a security risk** as the password is visible in the repository. The `SA_PASSWORD` is also exposed in `docker-compose.yml`.

**Production**: The application reads `AZURE_KEY_VAULT_ENDPOINT` from the environment and uses `ChainedTokenCredential` (`AzureDeveloperCliCredential` → `DefaultAzureCredential`) to authenticate to Azure Key Vault. The Key Vault then provides the actual SQL connection strings. No service principal credentials are stored in config; Managed Identity or `azd` CLI credentials are used.

**Security concern**: `AuthorizationConstants.JWT_SECRET_KEY` and `DEFAULT_PASSWORD` are hardcoded string constants in `ApplicationCore/Constants/AuthorizationConstants.cs`. The code comments acknowledge this with `// TODO: Don't use this in production` and `// TODO: Change this to an environment variable`, indicating known technical debt.

## Feature Flags

| Flag / Condition | Default | Controlled By | Effect |
|-----------------|---------|--------------|--------|
| `UseOnlyInMemoryDatabase` | `false` | `appsettings.test.json` or environment variable | Switches both DbContexts from SQL Server to EF Core InMemory provider |
| `ASPNETCORE_ENVIRONMENT == "Development" or "Docker"` | Runtime check in `Program.cs` | `ASPNETCORE_ENVIRONMENT` env var | Selects local SQL Server config vs Azure Key Vault + Azure SQL |

No formal feature flag framework (LaunchDarkly, .NET FeatureManagement) is present. Conditional behavior is implemented via environment checks in `Program.cs`.

## Framework & Runtime Versions

| Component | Version | Source |
|-----------|---------|--------|
| .NET Runtime | 8.0 | Directory.Build.props (`TargetFramework: net8.0`) |
| ASP.NET Core | 8.0.2 | Directory.Packages.props (`AspNetVersion=8.0.2`) |
| Entity Framework Core | 8.0.2 | Directory.Packages.props (`EntityFramworkCoreVersion=8.0.2`) |
| Blazor WebAssembly | 8.0.2 | Directory.Packages.props |
| Docker base image (runtime) | `mcr.microsoft.com/dotnet/aspnet:8.0` | src/Web/Dockerfile, src/PublicApi/Dockerfile |
| Docker base image (build) | `mcr.microsoft.com/dotnet/sdk:8.0` | src/Web/Dockerfile, src/PublicApi/Dockerfile |
| SQL Server (Docker) | `mcr.microsoft.com/azure-sql-edge` (latest) | docker-compose.yml |
| Ardalis.ApiEndpoints | 4.1.0 | Directory.Packages.props |
| Ardalis.Specification | 7.0.0 | Directory.Packages.props |
| AutoMapper | 12.0.1 | Directory.Packages.props |
| MediatR | 12.0.1 | Directory.Packages.props |
| FluentValidation | 11.9.0 | Directory.Packages.props |
| Azure.Identity | 1.10.4 | Directory.Packages.props |
| Swashbuckle.AspNetCore | 6.5.0 | Directory.Packages.props |
| xUnit | 2.7.0 | Directory.Packages.props |
