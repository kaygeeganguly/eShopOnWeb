# .NET Upgrade Plan: net8.0 → net10.0

## Overview

Upgrade the **eShopOnWeb** solution from **.NET 8.0** to **.NET 10.0 LTS**.

The upgrade was explicitly requested by the user. .NET 10 is the current latest LTS, providing long-term support, improved performance, and the latest platform features.

## Source Version

- **Target Framework**: `net8.0`
- **SDK**: `8.0.x`

## Target Version

- **Target Framework**: `net10.0`
- **SDK**: `10.0.x`

## Projects in Solution

| Project | Path |
|---------|------|
| ApplicationCore | `src/ApplicationCore/ApplicationCore.csproj` |
| BlazorAdmin | `src/BlazorAdmin/BlazorAdmin.csproj` |
| BlazorShared | `src/BlazorShared/BlazorShared.csproj` |
| Infrastructure | `src/Infrastructure/Infrastructure.csproj` |
| PublicApi | `src/PublicApi/PublicApi.csproj` |
| Web | `src/Web/Web.csproj` |
| UnitTests | `tests/UnitTests/UnitTests.csproj` |
| IntegrationTests | `tests/IntegrationTests/IntegrationTests.csproj` |
| PublicApiIntegrationTests | `tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj` |
| FunctionalTests | `tests/FunctionalTests/FunctionalTests.csproj` |

## Upgrade Scope

- Update `<TargetFramework>` from `net8.0` to `net10.0` in `Directory.Packages.props` (applied to all projects centrally)
- Update `global.json` SDK version from `8.0.x` to `10.0.x`
- Update all NuGet package versions (ASP.NET Core, EF Core, Microsoft.Extensions.*) from `8.x` to `10.x` compatible versions
- Review and resolve any breaking API changes between .NET 8 and .NET 10
- Ensure all projects build and all tests pass on net10.0
