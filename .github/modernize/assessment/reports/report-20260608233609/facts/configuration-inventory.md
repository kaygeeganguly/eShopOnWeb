# Configuration & Externalized Settings Inventory

This repository uses a compact but varied configuration model built from ASP.NET Core JSON files, launch profiles, container settings, and Azure deployment metadata. Sensitive production settings are designed to flow from Azure Key Vault or environment variables rather than being hard-coded in source.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| `appsettings.json` | Runtime config | `src/Web`, `src/PublicApi` | Base URLs, connection strings, logging defaults, catalog base URL |
| `appsettings.Development.json` | Runtime profile config | `src/Web`, `src/PublicApi`, `src/BlazorAdmin/wwwroot` | Local development URL and logging overrides |
| `appsettings.Docker.json` | Runtime profile config | `src/Web`, `src/PublicApi`, `src/BlazorAdmin/wwwroot` | Docker SQL Server connection strings and localhost container URLs |
| `launchSettings.json` | Local launch profiles | `src/*/Properties/launchSettings.json` | Development, production, WSL, and Docker launch behaviors |
| `docker-compose.yml` | Container orchestration config | repository root | Builds Web and PublicApi and provisions SQL Edge with environment variables |
| `azure.yaml` | Deployment config | repository root | Declares `web` service for Azure Developer CLI/App Service deployment |
| Environment variables | Externalized runtime config | Process environment | Used for Key Vault endpoint, Azure SQL secret key names, ASP.NET Core environment |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | Default local build | Developer inner-loop build and test configuration | Standard project references and central package versions |
| Release | `dotnet build -c Release` or publish | Production-oriented publish output | `BuildBundlerMinifier` runs only in Release for Web |
| Docker | Dockerfile and compose invocation | Container image build for Web and PublicApi | `Microsoft.VisualStudio.Azure.Containers.Tools.Targets`, Dockerfiles using .NET 8 SDK/runtime images |
| azd App Service deployment | `azd up` | Provision and deploy the Web project to Azure App Service | `azure.yaml` service declaration |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Development | `ASPNETCORE_ENVIRONMENT=Development` via launch settings | `appsettings.json`, `appsettings.Development.json` | Local HTTPS URLs, verbose logging |
| Docker | `ASPNETCORE_ENVIRONMENT=Docker` or Docker launch profile | `appsettings.json`, `appsettings.Docker.json` | SQL Server container connections, localhost 5106/5200 wiring |
| Production | `ASPNETCORE_ENVIRONMENT=Production` or `Web - PROD` profile | `appsettings.json` plus environment variables | Azure Key Vault-backed secret resolution and SQL retry settings |

## Properties Inventory

### Web / PublicApi representative properties

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Development, Docker override | `appsettings*.json` |
| `baseUrls:webBase` | `https://localhost:44315/` in Web, `https://localhost:5001/` in PublicApi | Development, Docker override | `appsettings*.json` |
| `ConnectionStrings:CatalogConnection` | LocalDB SQL Server | Base, Docker override, production via secret indirection | `appsettings.json`, `appsettings.Docker.json`, Azure Key Vault lookup |
| `ConnectionStrings:IdentityConnection` | LocalDB SQL Server | Base, Docker override, production via secret indirection | `appsettings.json`, `appsettings.Docker.json`, Azure Key Vault lookup |
| `CatalogBaseUrl` | empty string | Base | `appsettings.json` |
| `UseOnlyInMemoryDatabase` | not set / false | Optional toggle | External config key parsed by Infrastructure |
| `Logging:LogLevel:*` | Warning or Information depending on file | Development and Docker override | `appsettings*.json` |
| `AZURE_KEY_VAULT_ENDPOINT` | none | Production | Environment variable |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | none | Production | Environment variable referencing a Key Vault secret name |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | none | Production | Environment variable referencing a Key Vault secret name |

### BlazorAdmin representative properties

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Development, Docker override | `wwwroot/appsettings*.json` |
| `baseUrls:webBase` | `https://localhost:44315/` | Development, Docker override | `wwwroot/appsettings*.json` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| Web | `ASPNETCORE_ENVIRONMENT`, local URLs from launch settings | Not specified in repo | 1 by default |
| PublicApi | `ASPNETCORE_ENVIRONMENT`, optional `ASPNETCORE_URLS` in WSL profile | Not specified in repo | 1 by default |
| SQL Server container | `SA_PASSWORD`, `ACCEPT_EULA` | Not specified in repo | 1 by default |

No explicit memory limits, CPU limits, replica counts, or advanced runtime tuning parameters are declared in the checked-in configuration.

## Startup Dependency Chain

1. `sqlserver` starts first in Docker Compose because both application services depend on it.
2. `eshopwebmvc` waits on SQL Server availability implicitly through compose `depends_on` and EF Core database initialization.
3. `eshoppublicapi` also depends on SQL Server and seeds its contexts on startup.
4. Web health checks additionally expect the storefront itself and the `PublicApi` catalog endpoint to be reachable before reporting healthy.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `ConnectionStrings:CatalogConnection` in Docker | SQL connection string | `[MASKED]` in `appsettings.Docker.json` |
| `ConnectionStrings:IdentityConnection` in Docker | SQL connection string | `[MASKED]` in `appsettings.Docker.json` |
| `SA_PASSWORD` | SQL admin password | `[MASKED]` in `docker-compose.yml` |
| `AZURE_KEY_VAULT_ENDPOINT` | Key Vault URI | Environment variable |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | Secret name reference | Environment variable |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | Secret name reference | Environment variable |

### Secrets Provisioning Workflow

For local Docker use, secrets are injected directly through compose environment variables and Docker-specific appsettings files. For production-style deployment, the Web project reads a Key Vault endpoint from `AZURE_KEY_VAULT_ENDPOINT`, authenticates with Azure credentials, then resolves the actual SQL connection strings indirectly via secret-name environment variables; the resulting connection strings are consumed by the EF Core DbContexts.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| `UseOnlyInMemoryDatabase` | `false` when absent | Configuration key in JSON or environment |

No dedicated feature-management framework or remote flag service is configured in the repository.

## Framework & Runtime Versions

| Component | Version | Source |
|---|---:|---|
| .NET target framework | `net8.0` | `Directory.Packages.props`, project files |
| ASP.NET Core shared packages | `8.0.2` | `Directory.Packages.props` |
| EF Core | `8.0.2` | `Directory.Packages.props` |
| Swashbuckle | `6.5.0` | `Directory.Packages.props` |
| Azure.Identity | `1.10.4` | `Directory.Packages.props` |
| MediatR | `12.0.1` | `Directory.Packages.props` |
| FluentValidation | `11.9.0` | `Directory.Packages.props` |
| Docker SDK image | `mcr.microsoft.com/dotnet/sdk:8.0` | `src/Web/Dockerfile`, `src/PublicApi/Dockerfile` |
| Docker ASP.NET runtime image | `mcr.microsoft.com/dotnet/aspnet:8.0` | `src/Web/Dockerfile`, `src/PublicApi/Dockerfile` |
| SQL container image | `mcr.microsoft.com/azure-sql-edge` | `docker-compose.yml` |
