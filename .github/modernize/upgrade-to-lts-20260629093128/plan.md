# .NET Upgrade Plan: eShopOnWeb

## Overview

Upgrade the **eShopOnWeb** solution from **.NET 8 (`net8.0`)** to **.NET 10 (`net10.0`)** — the current latest Long-Term Support (LTS) release.

## Current State

- **Source .NET version**: `net8.0` (defined centrally in `Directory.Packages.props`)
- **SDK version**: `8.0.x` (pinned in `global.json`)
- **Project type**: Modern SDK-style .NET (no SDK-style conversion required)

## Target State

- **Target .NET version**: `net10.0`
- **SDK version**: `10.0.x`

## Projects in Solution

### Source Projects
| Project | Path |
|---------|------|
| ApplicationCore | `src/ApplicationCore/ApplicationCore.csproj` |
| BlazorShared | `src/BlazorShared/BlazorShared.csproj` |
| BlazorAdmin | `src/BlazorAdmin/BlazorAdmin.csproj` |
| Infrastructure | `src/Infrastructure/Infrastructure.csproj` |
| PublicApi | `src/PublicApi/PublicApi.csproj` |
| Web | `src/Web/Web.csproj` |

### Test Projects
| Project | Path |
|---------|------|
| UnitTests | `tests/UnitTests/UnitTests.csproj` |
| IntegrationTests | `tests/IntegrationTests/IntegrationTests.csproj` |
| FunctionalTests | `tests/FunctionalTests/FunctionalTests.csproj` |
| PublicApiIntegrationTests | `tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj` |

## Upgrade Scope

The upgrade encompasses the following changes across all projects:

1. **Target Framework Moniker (TFM)**: Update `<TargetFramework>` from `net8.0` to `net10.0` in `Directory.Packages.props`
2. **SDK version**: Update `global.json` to target the .NET 10 SDK (`10.0.x`)
3. **NuGet packages**: Update all package versions to their .NET 10 compatible equivalents (ASP.NET Core, Entity Framework Core, Azure SDK, etc.)
4. **API compatibility**: Address any breaking changes or deprecated APIs introduced between .NET 8 and .NET 10
5. **Build verification**: Ensure all projects compile and all tests pass on .NET 10

## Tasks

- [x] Upgrade .NET to net10.0 (`001-upgrade-dotnet-to-net10`)
