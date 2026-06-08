# Configuration & Externalized Settings Inventory

The solution externalizes settings across appsettings files, environment-specific overrides, Docker compose variables, and user secrets/Azure Key Vault integration. Configuration is split by runtime environment (Development, Docker, Production patterns) across Web and PublicApi hosts.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Web base settings | JSON | `src/Web/appsettings.json` | Base URLs, connection strings, logging |
| Web development settings | JSON | `src/Web/appsettings.Development.json` | Dev log levels and base URL overrides |
| Web docker settings | JSON | `src/Web/appsettings.Docker.json` | Docker SQL connection strings and container URL base |
| PublicApi base settings | JSON | `src/PublicApi/appsettings.json` | API base URLs, connection strings, logging |
| PublicApi development settings | JSON | `src/PublicApi/appsettings.Development.json` | Dev log level overrides |
| PublicApi docker settings | JSON | `src/PublicApi/appsettings.Docker.json` | Docker SQL connection strings and URL base |
| Launch profiles | JSON | `src/Web/Properties/launchSettings.json`, `src/PublicApi/Properties/launchSettings.json` | Environment variables and local application URLs |
| Docker compose environment | YAML | `docker-compose.yml`, `docker-compose.override.yml` | Service ports, ASPNETCORE_ENVIRONMENT, SQL container setup |
| User secrets | Secret store | `UserSecretsId` in Web/PublicApi csproj | Local secret storage for development |
| Azure Key Vault provider | External secret provider | `AddAzureKeyVault(...)` in `src/Web/Program.cs` | Production-style secret source via managed credentials |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
|---|---|---|---|
| Debug | Build configuration | Local development build | Includes standard package set |
| Release | Build configuration | Production publish optimization | `BuildBundlerMinifier` activated on Release in Web.csproj |
| Docker profile | Launch profile (`commandName: Docker`) | Container-based local execution | Docker tooling targets and compose integration |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Development | `ASPNETCORE_ENVIRONMENT=Development` | `appsettings.json` + `appsettings.Development.json` | Verbose logging and localhost URLs |
| Docker | `ASPNETCORE_ENVIRONMENT=Docker` | `appsettings.json` + `appsettings.Docker.json` | SQL container connection strings, docker host URLs |
| Production | `ASPNETCORE_ENVIRONMENT=Production` and non-dev branch in Program | `appsettings.json` + env vars + Key Vault | Key Vault-based connection string lookup |

## Properties Inventory

| Property Key | Default | Profiles | Source |
|---|---|---|---|
| `ConnectionStrings:CatalogConnection` | LocalDB SQL connection | Base, Docker override | Web/PublicApi appsettings files |
| `ConnectionStrings:IdentityConnection` | LocalDB SQL connection | Base, Docker override | Web/PublicApi appsettings files |
| `baseUrls:apiBase` | `https://localhost:5099/api/` | Development, Docker override | appsettings + env-specific files |
| `baseUrls:webBase` | Web localhost URL | Development, Docker override | appsettings + env-specific files |
| `CatalogBaseUrl` | empty string | Base | `src/Web/appsettings.json` |
| `UseOnlyInMemoryDatabase` | false when missing | Optional override | `Infrastructure/Dependencies.cs` via configuration |
| `AZURE_KEY_VAULT_ENDPOINT` | none | Production path | Environment variable consumed in Web Program |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | none | Production path | Environment variable key indirection |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | none | Production path | Environment variable key indirection |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
|---|---|---|---|
| Web | ASP.NET Core runtime, HTTPS redirection, cookie auth | Not explicitly configured in repo manifests | Single instance per local run |
| PublicApi | ASP.NET Core runtime, JWT auth, CORS | Not explicitly configured in repo manifests | Single instance per local run |
| SQL container | `mcr.microsoft.com/azure-sql-edge` with SA password/EULA | Default container memory (no explicit limit) | Single compose service |

## Startup Dependency Chain

1. `sqlserver` container starts first in docker compose.
2. `eshopwebmvc` and `eshoppublicapi` depend on `sqlserver` (`depends_on`).
3. On startup, Web and PublicApi run database seeding and migration paths before serving traffic.
4. Health check endpoints expose service readiness once hosts are initialized.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
|---|---|---|
| `ConnectionStrings:*` (Docker profile) | DB credentials | appsettings.Docker.json contains masked-sensitive inline secrets |
| `SA_PASSWORD` | SQL admin password | docker-compose environment variable (`[MASKED]`) |
| `UserSecretsId` values | Secret store pointers | csproj metadata (`[MASKED]` actual secret content) |
| `AZURE_KEY_VAULT_ENDPOINT` | Secret provider URI | Environment variable / external configuration |
| `AZURE_SQL_*_CONNECTION_STRING_KEY` | Key indirection names | Environment variables |

### Secrets Provisioning Workflow

For local development, secrets are sourced through ASP.NET Core user secrets and environment variables. In Docker scenarios, compose files inject SQL credentials and service environment values. In production-style startup paths, Web uses chained Azure credentials to access Key Vault, resolves connection-string key names from environment variables, and then reads actual secret values from the vault-backed configuration provider for database binding.

## Feature Flags

| Flag Name | Default | Controlled By |
|---|---|---|
| `UseOnlyInMemoryDatabase` | false | Configuration value/environment override |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET SDK | 8.0.x (roll-forward latestFeature) | `global.json` |
| Target framework | net8.0 | `Directory.Packages.props` |
| ASP.NET Core packages | 8.0.2 | `Directory.Packages.props` (`AspNetVersion`) |
| EF Core packages | 8.0.2 | `Directory.Packages.props` (`EntityFramworkCoreVersion`) |
| Azure Identity | 1.10.4 | `Directory.Packages.props` |
| Swashbuckle.AspNetCore | 6.5.0 | `Directory.Packages.props` |
| SQL container image | latest tag of azure-sql-edge | `docker-compose.yml` |
