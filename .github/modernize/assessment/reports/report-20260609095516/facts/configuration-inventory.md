# Configuration & Externalized Settings Inventory

Configuration is primarily file-based (`appsettings*.json`, compose files, launch profiles) with environment variable overlays and optional Azure Key Vault integration for production.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| `appsettings.json` | Runtime config | `src/Web/appsettings.json`, `src/PublicApi/appsettings.json` | Base URLs, connection strings, logging |
| `appsettings.Development.json` / env variants | Runtime config | host project folders | Environment-specific overrides |
| `appsettings.test.json` | Test/runtime config | `src/PublicApi` | Loaded explicitly in API startup |
| `launchSettings.json` | Local run profile | `src/*/Properties` | Local ports and env variables |
| `docker-compose.yml` | Container orchestration config | repo root | Service definitions and DB env vars |
| Environment variables | Externalized values | process/container env | Added via `AddEnvironmentVariables()` |
| Azure Key Vault | Secret source | configured by endpoint env var | Enabled in non-development web runtime |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | `dotnet build` default | Developer build and diagnostics | SDK defaults |
| Release | `-c Release` | Production-ready optimized build | SDK defaults |
| Docker build path | Docker compose / Dockerfiles | Containerized packaging of web/api services | `mcr.microsoft.com/dotnet/*` images |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Development | `ASPNETCORE_ENVIRONMENT=Development` | `appsettings.json` + development overrides | Local SQL/localdb behavior |
| Docker | `ASPNETCORE_ENVIRONMENT=Docker` | appsettings + compose env | Container startup and SQL dependency |
| Production-like | Non-development environment | base appsettings + env vars + Key Vault | Key Vault-backed secret resolution |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `baseUrls:apiBase` | localhost HTTPS URL | all | appsettings |
| `baseUrls:webBase` | localhost HTTPS URL | all | appsettings |
| `ConnectionStrings:CatalogConnection` | localdb connection string | dev/docker default | appsettings/env override |
| `ConnectionStrings:IdentityConnection` | localdb connection string | dev/docker default | appsettings/env override |
| `CatalogBaseUrl` | empty | optional | appsettings/env override |
| `Logging:LogLevel:*` | Warning | all | appsettings |
| `AllowedHosts` | `*` | all | appsettings |
| `UseOnlyInMemoryDatabase` | false when absent | test/custom | configuration key in infrastructure setup |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| Web | .NET runtime defaults + ASP.NET env vars | Not explicitly set in repo | 1 by default |
| PublicApi | .NET runtime defaults + ASP.NET env vars | Not explicitly set in repo | 1 by default |
| SQL container | SQL Edge container env settings | Not explicitly set in compose | 1 by default |

## Startup Dependency Chain

1. `sqlserver` container starts first in docker-compose.
2. `eshopwebmvc` and `eshoppublicapi` declare `depends_on: sqlserver`.
3. Web and PublicApi execute EF seed logic on startup (`CatalogContextSeed`, identity seed).
4. Health checks and Swagger endpoints become available after app pipeline initialization.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `ConnectionStrings:*` | DB connection info | appsettings/env (`[MASKED]`) |
| `SA_PASSWORD` in compose | SQL admin password | docker-compose env (`[MASKED]`) |
| `AZURE_KEY_VAULT_ENDPOINT` | Key Vault endpoint | environment variable |
| `AZURE_SQL_*_CONNECTION_STRING_KEY` | Key indirection for connection strings | environment variable |

### Secrets Provisioning Workflow

In local and docker scenarios, secrets are sourced from appsettings and environment variables. In production-style startup for the Web host, the app authenticates using Azure credentials and pulls secret values from Azure Key Vault by key name indirection before creating DbContexts.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| `UseOnlyInMemoryDatabase` | false/absent | configuration key |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET target framework | net8.0 | `Directory.Packages.props` |
| ASP.NET packages | 8.0.2 | `Directory.Packages.props` |
| EF Core | 8.0.2 | `Directory.Packages.props` |
| Swashbuckle | 6.5.0 | `Directory.Packages.props` |
| Azure Identity | 1.10.4 | `Directory.Packages.props` |
| Docker SQL image | azure-sql-edge | `docker-compose.yml` |
