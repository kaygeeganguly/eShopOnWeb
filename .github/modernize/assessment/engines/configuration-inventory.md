# Configuration & Externalized Settings Inventory

eShopOnWeb uses three layered configuration environments (Development, Docker, Production) across four appsettings file families, with User Secrets for local development and Azure Key Vault as the secrets store in production.

## Configuration Sources

| Source | Type | Path / Location | Notes |
|--------|------|-----------------|-------|
| `appsettings.json` | JSON file | `src/Web/`, `src/PublicApi/`, `src/BlazorAdmin/wwwroot/` | Base configuration; default connection strings point to LocalDB |
| `appsettings.Development.json` | JSON file | `src/Web/`, `src/PublicApi/`, `src/BlazorAdmin/wwwroot/` | Development overrides (log level, base URLs) |
| `appsettings.Docker.json` | JSON file | `src/Web/`, `src/PublicApi/`, `src/BlazorAdmin/wwwroot/` | Docker Compose overrides (SQL Server container connection strings, internal URLs) |
| `appsettings.test.json` | JSON file | `tests/PublicApiIntegrationTests/` | Forces in-memory database (`UseOnlyInMemoryDatabase: true`) |
| `launchSettings.json` | JSON file | `src/Web/Properties/`, `src/PublicApi/Properties/`, `src/BlazorAdmin/Properties/` | Development launch profiles; sets `ASPNETCORE_ENVIRONMENT` and ports; not deployed |
| `docker-compose.override.yml` | YAML | repo root | Sets `ASPNETCORE_ENVIRONMENT=Docker`, maps ports 5106 and 5200, mounts HTTPS certificate and User Secrets volumes |
| User Secrets | .NET Secret Manager | `~/.microsoft/usersecrets/{UserSecretsId}/secrets.json` | Local development; never committed; mounted via volume in Docker |
| Azure Key Vault | Cloud secret store | URI from `AZURE_KEY_VAULT_ENDPOINT` env var | Production only; connection strings retrieved via Key Vault key names stored in `AZURE_SQL_*_KEY` env vars |
| Environment Variables | OS / container env | `ASPNETCORE_ENVIRONMENT`, `ASPNETCORE_URLS`, `AZURE_*` | Highest-precedence override layer at runtime |

## Build Profiles

| Profile | Activation | Purpose | Key Effect |
|---------|-----------|---------|------------|
| Debug | Default in Visual Studio / `dotnet build` | Development build | No optimization; Roslyn analyzers active |
| Release | `-c Release` or Dockerfile | Production / container build | Full optimization; `BuildBundlerMinifier` runs to bundle/minify CSS/JS |
| Central Package Management | Always active (Directory.Packages.props) | Centralized NuGet version control | All `<PackageReference>` entries omit `Version`; versions resolved from `Directory.Packages.props` |

## Runtime Profiles

| Profile | Activation Method | Config Files Loaded | Key Overrides |
|---------|------------------|---------------------|---------------|
| Development | `ASPNETCORE_ENVIRONMENT=Development` (launchSettings.json) | `appsettings.json` + `appsettings.Development.json` | Log level: Debug/Information; LocalDB connection strings; base URLs to localhost:5099/44315 |
| Docker | `ASPNETCORE_ENVIRONMENT=Docker` (docker-compose.override.yml) | `appsettings.json` + `appsettings.Docker.json` | SQL Server container connection strings (sa password); internal Docker URLs; HTTP-only on port 8080 |
| Production | `ASPNETCORE_ENVIRONMENT=Production` (deploy target) | `appsettings.json` + Azure Key Vault | Connection strings from Azure Key Vault; `AzureDeveloperCliCredential` / `DefaultAzureCredential` |
| Test | `ASPNETCORE_ENVIRONMENT=Development` + `appsettings.test.json` | `appsettings.json` + `appsettings.test.json` | `UseOnlyInMemoryDatabase=true`; no SQL Server needed |

## Properties Inventory

### Web (`src/Web`)

| Property Key | Default Value | Profile Override | Source |
|-------------|---------------|-----------------|--------|
| `ConnectionStrings:CatalogConnection` | `Server=(localdb)\mssqllocaldb;...CatalogDb` | Docker: SQL Server container; Production: from Key Vault key named by `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | `appsettings.json` |
| `ConnectionStrings:IdentityConnection` | `Server=(localdb)\mssqllocaldb;...Identity` | Docker: SQL Server container; Production: from Key Vault key named by `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | `appsettings.json` |
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Development: `https://localhost:5099/api/`; Docker: `http://localhost:5200/api/` | `appsettings.json` |
| `baseUrls:webBase` | `https://localhost:44315/` | Development: `https://localhost:44315/`; Docker: `http://host.docker.internal:5106/` | `appsettings.json` |
| `CatalogBaseUrl` | `""` (empty) | None | `appsettings.json` |
| `Logging:LogLevel:Default` | `Warning` | Development: `Debug`; Docker: `Debug` | `appsettings.json` |
| `AZURE_KEY_VAULT_ENDPOINT` | Not set | Production only | Environment variable |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | Not set | Production only | Environment variable — Key Vault key name |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | Not set | Production only | Environment variable — Key Vault key name |

### PublicApi (`src/PublicApi`)

| Property Key | Default Value | Profile Override | Source |
|-------------|---------------|-----------------|--------|
| `ConnectionStrings:CatalogConnection` | `Server=(localdb)\mssqllocaldb;...CatalogDb` | Docker: SQL Server container | `appsettings.json` |
| `ConnectionStrings:IdentityConnection` | `Server=(localdb)\mssqllocaldb;...Identity` | Docker: SQL Server container | `appsettings.json` |
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Development: same; Docker: `http://localhost:5200/api/` | `appsettings.json` |
| `baseUrls:webBase` | `https://localhost:5001/` | Development: same; Docker: `http://host.docker.internal:5106/` | `appsettings.json` |
| `Logging:LogLevel:Default` | `Warning` | Development: `Information` | `appsettings.json` |

### BlazorAdmin (`src/BlazorAdmin/wwwroot`)

| Property Key | Default Value | Profile Override | Source |
|-------------|---------------|-----------------|--------|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Development: same; Docker: `http://localhost:5200/api/` | `appsettings.json` (WASM static file) |
| `baseUrls:webBase` | `https://localhost:44315/` | Development: same; Docker: `http://host.docker.internal:5106/` | `appsettings.json` (WASM static file) |
| `Logging:LogLevel:Default` | `Information` | None | `appsettings.json` |

### Test (`tests/PublicApiIntegrationTests`)

| Property Key | Default Value | Source |
|-------------|---------------|--------|
| `UseOnlyInMemoryDatabase` | `true` | `appsettings.test.json` |

## Startup Parameters & Resource Requirements

| Service | Runtime | Listen Ports | ASPNETCORE_ENVIRONMENT | Memory / CPU | Instance Count |
|---------|---------|-------------|----------------------|--------------|----------------|
| Web (Docker) | `mcr.microsoft.com/dotnet/aspnet:8.0` | 8080 (HTTP) inside container → 5106 host | Docker | Not specified (no `mem_limit`) | 1 |
| PublicApi (Docker) | `mcr.microsoft.com/dotnet/aspnet:8.0` | 8080 (HTTP) inside container → 5200 host | Docker | Not specified | 1 |
| SQL Server (Docker) | `mcr.microsoft.com/azure-sql-edge` | 1433 → 1433 host | N/A | Not specified | 1 |
| Web (Development) | .NET 8 Kestrel | 5001 (HTTPS), 5000 (HTTP) | Development | No limits | 1 |
| PublicApi (Development) | .NET 8 Kestrel | 5099 (HTTPS), 5098 (HTTP) | Development | No limits | 1 |

No JVM parameters apply (this is a .NET project). No Kubernetes resource limits are configured — Docker Compose does not specify `mem_limit` or `cpus`.

## Startup Dependency Chain

```
SQL Server (sqlserver:1433)
  └─► Web (eshopwebmvc) — depends_on: sqlserver (no health probe)
  └─► PublicApi (eshoppublicapi) — depends_on: sqlserver (no health probe)
```

**Wait mechanism**: Docker Compose `depends_on` only waits for the container to *start*, not for SQL Server to be *ready*. No `dockerize`, `wait-for-it`, or Compose `healthcheck` is configured. `CatalogContextSeed` handles this with a retry loop (up to 10 attempts) — if SQL Server is not yet accepting connections the first migration/seed attempt will fail and retry.

**ASP.NET Core startup**: Both Web and PublicApi run `EF Core Migrations` and `CatalogContextSeed` during startup via `WebApplication` initializer code (before the HTTP pipeline begins accepting requests). The Web service registers custom health checks (`ApiHealthCheck`, `HomePageHealthCheck`) that probe the PublicApi URL and the Web home page respectively — these checks run *after* startup, not as preconditions.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage |
|-----------------|------|---------|
| `ConnectionStrings:CatalogConnection` (Docker) | SQL Server SA password embedded in connection string | `appsettings.Docker.json` — **hardcoded SA password** `@someThingComplicated1234` |
| `ConnectionStrings:IdentityConnection` (Docker) | SQL Server SA password embedded in connection string | `appsettings.Docker.json` — **hardcoded SA password** `@someThingComplicated1234` |
| `ConnectionStrings:CatalogConnection` (Production) | Azure SQL connection string | Azure Key Vault; key name from `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` env var |
| `ConnectionStrings:IdentityConnection` (Production) | Azure SQL connection string | Azure Key Vault; key name from `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` env var |
| `AZURE_KEY_VAULT_ENDPOINT` | Key Vault URI | Environment variable injected at deploy time |
| JWT Signing Key | Token signing secret | .NET User Secrets (development); expected in Key Vault or env var in production — not explicitly configured in any appsettings file |
| SQL SA password (LocalDB dev) | LocalDB uses Windows Integrated Security | No password — Integrated Security only |

**Note**: The Docker `appsettings.Docker.json` files contain a hardcoded SA password (`@someThingComplicated1234`). This is a known pattern for local Docker development but represents a security risk if these files are deployed to staging/production environments.

### Secrets Provisioning Workflow

**Development**: Developers use .NET User Secrets (`dotnet user-secrets set`) keyed by `UserSecretsId` in `Web.csproj` and `PublicApi.csproj`. Secrets are stored in `~/.microsoft/usersecrets/{id}/secrets.json` and mounted into Docker containers via the `~/.microsoft/usersecrets:/root/.microsoft/usersecrets:ro` volume in `docker-compose.override.yml`.

**Production**: The configuration pipeline is:
1. App starts with `ASPNETCORE_ENVIRONMENT` not equal to `Development` or `Docker`.
2. `ChainedTokenCredential(AzureDeveloperCliCredential, DefaultAzureCredential)` is constructed — supporting both `azd`-based developer logins and Managed Identity in Azure hosting.
3. `builder.Configuration.AddAzureKeyVault(uri, credential)` loads all Key Vault secrets as configuration entries.
4. The connection string key *names* are resolved from env vars `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` and `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` — these indirection keys point to the actual Key Vault secret names holding the connection strings.
5. Environment variables are added last via `builder.Configuration.AddEnvironmentVariables()`, giving them highest precedence.

**Identity/access model**: Managed Identity (via `DefaultAzureCredential`) is the expected production pattern. No service principal credentials or RBAC role assignments are specified in code — these must be configured at the Azure hosting level.

## Feature Flags

| Flag | Default | Controlled By | Effect |
|------|---------|--------------|--------|
| `UseOnlyInMemoryDatabase` | `false` | `appsettings.test.json` | Forces EF Core to use in-memory provider instead of SQL Server |
| `ASPNETCORE_ENVIRONMENT == "Development" or "Docker"` | `Development` (local) | `ASPNETCORE_ENVIRONMENT` env var | Selects between SQL Server config via `appsettings.json` (LocalDB/Docker) vs Azure Key Vault (Production) |

No feature flag framework (LaunchDarkly, .NET `Microsoft.FeatureManagement`, etc.) is configured. Feature branching is done via environment name checks in `Program.cs`.

## Framework & Runtime Versions

| Component | Version | Source |
|-----------|---------|--------|
| .NET (Target Framework) | net8.0 | `Directory.Packages.props` |
| ASP.NET Core (MVC, Identity, EF) | 8.0.2 | `Directory.Packages.props` (`AspNetVersion`) |
| Entity Framework Core | 8.0.2 | `Directory.Packages.props` (`EntityFramworkCoreVersion`) |
| System.* extensions | 8.0.0 | `Directory.Packages.props` (`SystemExtensionVersion`) |
| Docker base image (runtime) | `mcr.microsoft.com/dotnet/aspnet:8.0` | `src/Web/Dockerfile`, `src/PublicApi/Dockerfile` |
| Docker base image (build) | `mcr.microsoft.com/dotnet/sdk:8.0` | `src/Web/Dockerfile`, `src/PublicApi/Dockerfile` |
| SQL Server (Docker) | `mcr.microsoft.com/azure-sql-edge` (latest) | `docker-compose.yml` |
| Blazor WebAssembly | 8.0.2 | `Directory.Packages.props` |
| Ardalis.Specification | 7.0.0 | `Directory.Packages.props` |
| AutoMapper | 12.0.1 | `Directory.Packages.props` |
| MediatR | 12.0.1 | `Directory.Packages.props` |
| Swashbuckle.AspNetCore | 6.5.0 | `Directory.Packages.props` |
| xunit | 2.7.0 | `Directory.Packages.props` |
| MSTest | 3.2.2 | `Directory.Packages.props` |
