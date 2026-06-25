# Configuration & Externalized Settings Inventory

eShopOnWeb uses a layered ASP.NET Core configuration system with three runtime environment profiles (Development, Docker, Production), Azure Key Vault as the production secrets store, and User Secrets for local development.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|--------|------|--------------|-------|
| appsettings.json | JSON file | `src/Web/appsettings.json`, `src/PublicApi/appsettings.json` | Base defaults for all environments |
| appsettings.Development.json | JSON file | `src/Web/appsettings.Development.json`, `src/PublicApi/appsettings.Development.json` | Development overrides — higher log levels, localhost URLs |
| appsettings.Docker.json | JSON file | `src/Web/appsettings.Docker.json`, `src/PublicApi/appsettings.Docker.json` | Docker Compose overrides — `sqlserver` hostname, container ports |
| appsettings.json (BlazorAdmin) | JSON file | `src/BlazorAdmin/wwwroot/appsettings.json` | Client-side Blazor config — base URLs only |
| appsettings.Development.json (BlazorAdmin) | JSON file | `src/BlazorAdmin/wwwroot/appsettings.Development.json` | Blazor dev overrides |
| appsettings.Docker.json (BlazorAdmin) | JSON file | `src/BlazorAdmin/wwwroot/appsettings.Docker.json` | Blazor Docker overrides |
| appsettings.test.json | JSON file | `tests/PublicApiIntegrationTests/appsettings.test.json` | Forces `UseOnlyInMemoryDatabase=true` for all integration tests |
| User Secrets | Secret Manager | `~/.microsoft/usersecrets/{UserSecretsId}/secrets.json` | Local developer secrets; Web UserSecretsId: `aspnet-Web2-1FA3F72E-E7E3-4360-9E49-1CCCD7FE85F7`; PublicApi: `5b662463-1efd-4bae-bde4-befe0be3e8ff` |
| Azure Key Vault | Cloud secret store | URI from env var `AZURE_KEY_VAULT_ENDPOINT` | Production only — loaded via `AddAzureKeyVault` using `ChainedTokenCredential` |
| Environment Variables | Process env | `ASPNETCORE_ENVIRONMENT`, `ASPNETCORE_URLS`, `AZURE_KEY_VAULT_ENDPOINT`, `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY`, `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | Injected in Docker Compose and production deployments |
| docker-compose.yml | Docker Compose | `docker-compose.yml`, `docker-compose.override.yml` | Defines services, SQL Server container, and environment variable injection |

## Build Profiles

| Profile | Activation | Purpose | Key Changes |
|---------|-----------|---------|------------|
| Debug | Default MSBuild configuration | Local development with debug symbols | No minification; `BuildBundlerMinifier` excluded (condition: `Configuration=='Release'`) |
| Release | `dotnet publish -c Release` or CI build | Production build for deployment | Runs `BuildBundlerMinifier` to minify CSS/JS; optimized output |
| net8.0 (TFM) | Implicit — set in `Directory.Packages.props` | Single target framework for all projects | All packages resolved for .NET 8 |

## Runtime Profiles

| Profile | Activation Method | Config Files Loaded | Key Overrides |
|---------|-----------------|-------------------|--------------|
| Development | `ASPNETCORE_ENVIRONMENT=Development` (default in `launchSettings.json`) | `appsettings.json` + `appsettings.Development.json` | Log level Debug; localhost URLs (5001/5099); LocalDB connection strings |
| Docker | `ASPNETCORE_ENVIRONMENT=Docker` (set in `docker-compose.override.yml`) | `appsettings.json` + `appsettings.Docker.json` | SQL Server at `sqlserver:1433` with SA credentials; internal container ports 8080; mapped to 5106 (web) and 5200 (API) |
| Production | `ASPNETCORE_ENVIRONMENT=Production` (or any non-Development/Docker value) | `appsettings.json` only + Azure Key Vault | Azure Key Vault loaded via `AZURE_KEY_VAULT_ENDPOINT`; DB connection strings fetched from Key Vault via `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` and `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` |

## Properties Inventory

### Web (src/Web)

| Property Key | Default Value | Profile/Environment | Source |
|-------------|--------------|-------------------|--------|
| `ConnectionStrings:CatalogConnection` | `Server=(localdb)\mssqllocaldb;...CatalogDb` | Development (base) | `appsettings.json` |
| `ConnectionStrings:CatalogConnection` | `Server=sqlserver,1433;...User Id=sa;****** | Docker | `appsettings.Docker.json` |
| `ConnectionStrings:CatalogConnection` | Key Vault secret name from `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | Production | Azure Key Vault |
| `ConnectionStrings:IdentityConnection` | `Server=(localdb)\mssqllocaldb;...Identity` | Development (base) | `appsettings.json` |
| `ConnectionStrings:IdentityConnection` | `Server=sqlserver,1433;...User Id=sa;****** | Docker | `appsettings.Docker.json` |
| `ConnectionStrings:IdentityConnection` | Key Vault secret name from `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | Production | Azure Key Vault |
| `AZURE_KEY_VAULT_ENDPOINT` | (empty string fallback) | Production | Environment variable |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | (not set) | Production | Environment variable |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | (not set) | Production | Environment variable |
| `CatalogBaseUrl` | `""` (empty) | All | `appsettings.json` |
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Development | `appsettings.Development.json` |
| `baseUrls:apiBase` | `http://localhost:5200/api/` | Docker | `appsettings.Docker.json` |
| `baseUrls:webBase` | `https://localhost:44315/` | Development | `appsettings.Development.json` |
| `baseUrls:webBase` | `http://host.docker.internal:5106/` | Docker | `appsettings.Docker.json` |
| `Logging:LogLevel:Default` | `Warning` | All | `appsettings.json` |
| `Logging:LogLevel:Default` | `Debug` | Development, Docker | `appsettings.Development.json`, `appsettings.Docker.json` |

### PublicApi (src/PublicApi)

| Property Key | Default Value | Profile/Environment | Source |
|-------------|--------------|-------------------|--------|
| `ConnectionStrings:CatalogConnection` | `Server=(localdb)\mssqllocaldb;...CatalogDb` | Development | `appsettings.json` |
| `ConnectionStrings:CatalogConnection` | `Server=sqlserver,1433;...User Id=sa;****** | Docker | `appsettings.Docker.json` |
| `ConnectionStrings:IdentityConnection` | `Server=(localdb)\mssqllocaldb;...Identity` | Development | `appsettings.json` |
| `ConnectionStrings:IdentityConnection` | `Server=sqlserver,1433;...User Id=sa;****** | Docker | `appsettings.Docker.json` |
| `baseUrls:apiBase` | `https://localhost:5099/api/` | All | `appsettings.json` |
| `baseUrls:webBase` | `https://localhost:5001/` | All | `appsettings.json` |
| `CatalogBaseUrl` | `""` | All | `appsettings.json` |
| `Logging:LogLevel:Default` | `Warning` | All | `appsettings.json` |
| `Logging:LogLevel:Default` | `Information` | Development, Docker | `appsettings.Development.json`, `appsettings.Docker.json` |

### BlazorAdmin (src/BlazorAdmin/wwwroot)

| Property Key | Default Value | Profile/Environment | Source |
|-------------|--------------|-------------------|--------|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Development | `appsettings.json` |
| `baseUrls:apiBase` | `http://localhost:5200/api/` | Docker | `appsettings.Docker.json` |
| `baseUrls:webBase` | `https://localhost:44315/` | Development | `appsettings.json` |
| `baseUrls:webBase` | `http://host.docker.internal:5106/` | Docker | `appsettings.Docker.json` |

### Test Projects

| Property Key | Default Value | Profile/Environment | Source |
|-------------|--------------|-------------------|--------|
| `UseOnlyInMemoryDatabase` | `true` | Test | `tests/PublicApiIntegrationTests/appsettings.test.json` |

## Startup Parameters and Resource Requirements

| Service | Runtime Options | Memory Limit | Exposed Ports | Notes |
|---------|----------------|-------------|--------------|-------|
| eshopwebmvc (Docker) | `ASPNETCORE_ENVIRONMENT=Docker`, `ASPNETCORE_URLS=http://+:8080` | Not configured | 5106 → 8080 | User secrets and HTTPS certs mounted from host |
| eshoppublicapi (Docker) | `ASPNETCORE_ENVIRONMENT=Docker`, `ASPNETCORE_URLS=http://+:8080` | Not configured | 5200 → 8080 | User secrets and HTTPS certs mounted from host |
| sqlserver (Docker) | `SA_PASSWORD=[MASKED]`, `ACCEPT_EULA=Y` | Not configured | 1433 → 1433 | Uses `mcr.microsoft.com/azure-sql-edge` image |
| Web (local) | `ASPNETCORE_ENVIRONMENT=Development` | N/A | 5001 (HTTPS), 5000 (HTTP) | .NET CLI / IIS Express |
| PublicApi (local) | `ASPNETCORE_ENVIRONMENT=Development` | N/A | 5099 (HTTPS), 5098 (HTTP) | .NET CLI / IIS Express |

No JVM heap settings (N/A — .NET runtime). No Kubernetes resource limits or requests are defined — no K8s manifests exist in the repository.

## Startup Dependency Chain

Docker Compose startup order:

1. **sqlserver** — SQL Server (Azure SQL Edge) container starts first; no explicit health check configured
2. **eshopwebmvc** — depends on `sqlserver` via `depends_on`; calls `CatalogContextSeed` which runs `database.Migrate()` automatically on first start
3. **eshoppublicapi** — depends on `sqlserver` via `depends_on`; runs EF Core migrations via `Dependencies.ConfigureServices`

**Wait mechanism:** Docker Compose `depends_on` only waits for the container to start, **not** for SQL Server to be ready to accept connections. There is no `healthcheck`, `dockerize`, or retry loop configured — the application may fail on first startup if SQL Server is not ready. The `EnableRetryOnFailure()` on EF Core SQL connections provides limited retry tolerance.

**Local development:** No explicit startup order — developers run Web and PublicApi independently, both connecting to LocalDB.

## Secrets and Sensitive Configuration

| Secret Reference | Type | Storage / Location | Environment |
|-----------------|------|--------------------|------------|
| `ConnectionStrings:CatalogConnection` (with SA password) | Database credentials | `appsettings.Docker.json` (plaintext — hardcoded `@someThingComplicated1234`) | Docker only |
| `ConnectionStrings:IdentityConnection` (with SA password) | Database credentials | `appsettings.Docker.json` (plaintext — hardcoded `@someThingComplicated1234`) | Docker only |
| `sqlserver:SA_PASSWORD` | SQL Server SA password | `docker-compose.yml` (plaintext — `@someThingComplicated1234`) | Docker only |
| `AZURE_KEY_VAULT_ENDPOINT` | Key Vault URI | Environment variable | Production |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | Key Vault secret name | Environment variable | Production |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | Key Vault secret name | Environment variable | Production |
| User Secrets (local dev) | DB connection strings, API keys | `~/.microsoft/usersecrets/` (local file, not committed) | Development |
| `AuthorizationConstants.JWT_SECRET_KEY` | JWT signing key | Hardcoded in `ApplicationCore/Constants/AuthorizationConstants.cs` | All — **CRITICAL RISK** |
| `AuthorizationConstants.AUTH_KEY` | Auth key | Hardcoded in `ApplicationCore/Constants/AuthorizationConstants.cs` | All — **CRITICAL RISK** |
| `AuthorizationConstants.DEFAULT_PASSWORD` | Default user password `Pass@word1` | Hardcoded in `ApplicationCore/Constants/AuthorizationConstants.cs` | All — **HIGH RISK** |

### Secrets Provisioning Workflow

**Local development:** Developers use .NET User Secrets (`dotnet user-secrets set`) to store connection strings outside the repository. The `UserSecretsId` is embedded in the `.csproj` files. Visual Studio and the .NET CLI automatically load secrets from `~/.microsoft/usersecrets/{id}/secrets.json` in the Development environment.

**Docker environment:** Secrets are stored as **plaintext in `appsettings.Docker.json`** and `docker-compose.yml`. The SA password `@someThingComplicated1234` is committed to the repository in these files. This is acceptable for a local demo environment but is a significant risk if this configuration is used as a template for staging or production deployments. User secrets and HTTPS developer certificates are bind-mounted from the host (`~/.microsoft/usersecrets`, `~/.aspnet/https`) to preserve local overrides.

**Production environment:** The Web application loads secrets from **Azure Key Vault** using `ChainedTokenCredential` (`AzureDeveloperCliCredential` first, then `DefaultAzureCredential`). The Key Vault URI must be provided via the `AZURE_KEY_VAULT_ENDPOINT` environment variable. Connection strings are retrieved from Key Vault using secret names specified by `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` and `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` environment variables. The PublicApi does **not** have Key Vault integration configured — it only reads from `appsettings.json` and environment variables in production.

**Critical risk — hardcoded secrets:** `JWT_SECRET_KEY` and `AUTH_KEY` are hardcoded string constants in `ApplicationCore/Constants/AuthorizationConstants.cs`. These are visible in source code and used to sign JWT tokens for the PublicApi. Any attacker with access to the source code can forge valid JWT tokens. These must be moved to environment variables or Key Vault before production deployment.

## Feature Flags

| Flag / Conditional | Default | Controlled By | Effect |
|-------------------|---------|--------------|--------|
| `UseOnlyInMemoryDatabase` | `false` | `appsettings.test.json` or env var | When `true`: both `CatalogContext` and `AppIdentityDbContext` use EF Core InMemory instead of SQL Server |
| `ASPNETCORE_ENVIRONMENT == Development or Docker` | `false` in production | `ASPNETCORE_ENVIRONMENT` env var | When true: uses `ConfigureServices` with SQL Server from connection string; when false: uses Azure SQL + Key Vault |

No formal feature flag framework (LaunchDarkly, Microsoft.FeatureManagement, etc.) is in use. The only conditional behavior is the `UseOnlyInMemoryDatabase` flag and environment-based branching in `Program.cs`.

## Framework and Runtime Versions

| Component | Version | Source |
|-----------|---------|--------|
| .NET SDK (target) | net8.0 | `Directory.Packages.props` TargetFramework |
| ASP.NET Core | 8.0.2 | `Directory.Packages.props` AspNetVersion |
| Entity Framework Core | 8.0.2 | `Directory.Packages.props` EntityFrameworkCoreVersion |
| System.* extensions | 8.0.0 | `Directory.Packages.props` SystemExtensionVersion |
| Blazor WebAssembly | 8.0.2 | Same as AspNetVersion |
| MediatR | 12.0.1 | `Directory.Packages.props` |
| AutoMapper | 12.0.1 | `Directory.Packages.props` |
| Ardalis.Specification | 7.0.0 | `Directory.Packages.props` |
| Swashbuckle.AspNetCore | 6.5.0 | `Directory.Packages.props` |
| Azure.Identity | 1.10.4 | `Directory.Packages.props` |
| Azure.Extensions.AspNetCore.Configuration.Secrets | 1.3.1 | `Directory.Packages.props` |
| System.IdentityModel.Tokens.Jwt | 7.3.1 | `Directory.Packages.props` |
| FluentValidation | 11.9.0 | `Directory.Packages.props` |
| xunit | 2.7.0 | `Directory.Packages.props` |
| Docker base image (Web, PublicApi build) | `mcr.microsoft.com/dotnet/sdk:8.0` | `src/Web/Dockerfile`, `src/PublicApi/Dockerfile` |
| Docker base image (Web, PublicApi runtime) | `mcr.microsoft.com/dotnet/aspnet:8.0` | `src/Web/Dockerfile`, `src/PublicApi/Dockerfile` |
| Docker base image (SQL Server) | `mcr.microsoft.com/azure-sql-edge` (latest) | `docker-compose.yml` |
| Visual Studio Solution Format | 17.0 (VS 2022) | `eShopOnWeb.sln` |
