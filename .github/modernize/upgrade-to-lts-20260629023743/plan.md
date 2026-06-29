# .NET Upgrade Plan: eShopOnWeb

## Overview

Upgrade the **eShopOnWeb** solution from **.NET 8.0** to **.NET 10.0 LTS**.

The user has explicitly requested an upgrade to the latest LTS version. .NET 10.0 is the current latest Long-Term Support release.

## Source and Target Versions

| | Version |
|---|---|
| **Source** | .NET 8.0 (`net8.0`) |
| **Target** | .NET 10.0 (`net10.0`) |

## Projects in Solution

| Project | Path |
|---|---|
| Microsoft.eShopWeb.Web | `src/Web/Web.csproj` |
| Microsoft.eShopWeb.ApplicationCore | `src/ApplicationCore/ApplicationCore.csproj` |
| Microsoft.eShopWeb.Infrastructure | `src/Infrastructure/Infrastructure.csproj` |
| Microsoft.eShopWeb.PublicApi | `src/PublicApi/PublicApi.csproj` |
| BlazorShared | `src/BlazorShared/BlazorShared.csproj` |
| BlazorAdmin | `src/BlazorAdmin/BlazorAdmin.csproj` |
| UnitTests | `tests/UnitTests/UnitTests.csproj` |
| FunctionalTests | `tests/FunctionalTests/FunctionalTests.csproj` |
| IntegrationTests | `tests/IntegrationTests/IntegrationTests.csproj` |
| PublicApiIntegrationTests | `tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj` |

## Upgrade Scope

The upgrade encompasses the following areas:

1. **Target Framework Moniker (TFM)**: Update `<TargetFramework>` from `net8.0` to `net10.0` in `Directory.Packages.props`, and update the SDK version in `global.json` from `8.0.x` to `10.0.x`.
2. **NuGet Package Updates**: Upgrade all ASP.NET Core, Entity Framework Core, and related packages (e.g., `Microsoft.AspNetCore.*`, `Microsoft.EntityFrameworkCore.*`) from their `8.0.x` versions to `10.0.x` counterparts.
3. **API Compatibility**: Identify and resolve any breaking API changes between .NET 8 and .NET 10, including ASP.NET Core, Blazor WebAssembly, and EF Core changes.
4. **Blazor WebAssembly**: Update the BlazorAdmin project (SDK: `Microsoft.NET.Sdk.BlazorWebAssembly`) to target `net10.0`.
5. **Build Verification**: Ensure the entire solution compiles and all existing unit/integration/functional tests pass after the upgrade.
