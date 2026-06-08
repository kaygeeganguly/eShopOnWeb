# Configuration & Externalized Settings Inventory

eShopOnWeb uses layered configuration from appsettings files, launch profiles, docker-compose environment settings, central package/runtime descriptors, and optional cloud secret lookup through Azure Key Vault in non-development environments.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| `appsettings.json` | Runtime config | `src/Web`, `src/PublicApi` | Base URLs, connection strings, logging defaults |
| `appsettings.Development.json` | Runtime profile override | `src/Web`, `src/PublicApi` | Development-specific base URLs and logging |
| `appsettings.Docker.json` | Runtime profile override | `src/Web`, `src/PublicApi` | Docker SQL connection strings and host mappings |
| `launchSettings.json` | Local run profiles | `src/Web/Properties`, `src/PublicApi/Properties`, `src/BlazorAdmin/Properties` | IIS/project/WSL/docker launch profiles and environment variables |
| `docker-compose.yml` / override | Container runtime config | repo root | Service definitions, dependencies, environment overrides |
| Azure Key Vault config hook | External secret source | `src/Web/Program.cs` | Activated in non-dev/non-docker environments |
| `Directory.Packages.props` | Build dependency config | repo root | Central package versions and target framework metadata |
| `global.json` | SDK selection config | repo root | Pins .NET SDK major feature band (`8.0.x`) |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | `dotnet build` default | Development build/debug symbols | Standard package set |
| Release | `dotnet build -c Release` | Optimized release builds | Includes `BuildBundlerMinifier` in Web project |
| Docker image build | Dockerfile / compose build | Containerized runtime packaging | ASP.NET runtime and SDK images |
| Central package management | Auto via SDK | Shared dependency versions across projects | `Directory.Packages.props` |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Development | `ASPNETCORE_ENVIRONMENT=Development` | `appsettings.json` + `appsettings.Development.json` | Local HTTPS URLs and development logging |
| Docker | `ASPNETCORE_ENVIRONMENT=Docker` | `appsettings.json` + `appsettings.Docker.json` | SQL container connection strings and HTTP bindings |
| Production | `ASPNETCORE_ENVIRONMENT=Production` (Web launch profile and host env) | `appsettings.json` + host secrets | Key Vault-based connection string resolution in Web |

## Properties Inventory

### Web

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Base, Development, Docker override | appsettings files |
| `baseUrls:webBase` | `https://localhost:44315/` | Base, Development, Docker override | appsettings files |
| `ConnectionStrings:CatalogConnection` | LocalDB connection string | Base, Docker override | appsettings files |
| `ConnectionStrings:IdentityConnection` | LocalDB connection string | Base, Docker override | appsettings files |
| `CatalogBaseUrl` | empty string | Base | appsettings.json |
| `AZURE_KEY_VAULT_ENDPOINT` | none | Production/cloud hosts | environment variable |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | none | Production/cloud hosts | environment variable |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | none | Production/cloud hosts | environment variable |
| `UseOnlyInMemoryDatabase` | false when absent | Optional flag | configuration provider chain |

### PublicApi

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Base, Development, Docker override | appsettings files |
| `baseUrls:webBase` | `https://localhost:5001/` | Base, Development, Docker override | appsettings files |
| `ConnectionStrings:CatalogConnection` | LocalDB string | Base, Docker override | appsettings files |
| `ConnectionStrings:IdentityConnection` | LocalDB string | Base, Docker override | appsettings files |
| `ASPNETCORE_URLS` | framework default | Docker/WSL profiles | launchSettings and compose |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| Web | `ASPNETCORE_ENVIRONMENT`, `ASPNETCORE_URLS` | Not explicitly constrained in repo manifests | Single instance in compose |
| PublicApi | `ASPNETCORE_ENVIRONMENT`, `ASPNETCORE_URLS` | Not explicitly constrained in repo manifests | Single instance in compose |
| SQL Edge | `SA_PASSWORD`, `ACCEPT_EULA` env values | No explicit memory limits in compose | Single instance in compose |

## Startup Dependency Chain

1. `sqlserver` container starts first.
2. `eshopwebmvc` waits for `sqlserver` via docker-compose `depends_on`.
3. `eshoppublicapi` waits for `sqlserver` via docker-compose `depends_on`.
4. Both ASP.NET services run startup seeding (`CatalogContextSeed`, `AppIdentityDbContextSeed`) and then begin serving requests.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `SA_PASSWORD` | SQL admin password | Docker compose env var `[MASKED]` |
| `ConnectionStrings:*` credentials (docker profile) | DB credential | appsettings.Docker.json `[MASKED]` |
| `UserSecretsId` values in csproj | Local development secret indirection | ASP.NET user-secrets store |
| `AZURE_KEY_VAULT_ENDPOINT` and SQL key-name env vars | Secret locator/config key references | Environment variables |

### Secrets Provisioning Workflow

For local development, secrets can flow from user-secrets and appsettings files. In Docker mode, credentials are provided as environment variables and docker JSON settings. For non-development deployment, Web startup uses Azure identity credentials to access Key Vault and then resolves actual SQL connection strings by configured key names before creating DbContext instances.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| `UseOnlyInMemoryDatabase` | false (when absent) | Configuration value/env override |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET SDK | `8.0.x` | `global.json` |
| Target framework | `net8.0` | `Directory.Packages.props` |
| ASP.NET shared stack | `8.0.2` | `Directory.Packages.props` |
| Entity Framework Core | `8.0.2` | `Directory.Packages.props` |
| Azure.Identity | `1.10.4` | `Directory.Packages.props` |
| Swashbuckle.AspNetCore | `6.5.0` | `Directory.Packages.props` |
| Docker base image (Web/PublicApi) | `mcr.microsoft.com/dotnet/aspnet:8.0` | Dockerfiles |
| Docker DB image | `mcr.microsoft.com/azure-sql-edge` | docker-compose.yml |
