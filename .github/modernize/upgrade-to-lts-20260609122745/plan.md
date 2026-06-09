# .NET Upgrade Plan: eShopOnWeb

## Overview

Upgrade **eShopOnWeb** from **.NET 8.0** to **.NET 10.0** (latest LTS).

- **Source version**: .NET 8.0 (`net8.0`)
- **Target version**: .NET 10.0 (`net10.0`)
- **Reason**: User requested upgrade to the latest LTS version. .NET 10.0 is the current latest Long-Term Support (LTS) release, providing extended support, improved performance, and the latest platform features.

## Projects in Solution

### Source Projects
| Project | Path |
|---------|------|
| ApplicationCore | `src/ApplicationCore/ApplicationCore.csproj` |
| BlazorAdmin | `src/BlazorAdmin/BlazorAdmin.csproj` |
| BlazorShared | `src/BlazorShared/BlazorShared.csproj` |
| Infrastructure | `src/Infrastructure/Infrastructure.csproj` |
| PublicApi | `src/PublicApi/PublicApi.csproj` |
| Web | `src/Web/Web.csproj` |

### Test Projects
| Project | Path |
|---------|------|
| FunctionalTests | `tests/FunctionalTests/FunctionalTests.csproj` |
| IntegrationTests | `tests/IntegrationTests/IntegrationTests.csproj` |
| PublicApiIntegrationTests | `tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj` |
| UnitTests | `tests/UnitTests/UnitTests.csproj` |

## Upgrade Scope

The upgrade encompasses the following areas:

1. **Target Framework Moniker (TFM)**: Update `<TargetFramework>` from `net8.0` to `net10.0` in `Directory.Packages.props`
2. **SDK version**: Update `global.json` SDK version from `8.0.x` to `10.0.x`
3. **NuGet packages**: Update all package versions pinned to ASP.NET Core 8.x, EF Core 8.x, and .NET 8.x system extensions to their .NET 10-compatible counterparts
4. **BlazorWebAssembly**: Update `Microsoft.AspNetCore.Components.WebAssembly` and related packages to .NET 10 versions
5. **API compatibility**: Address any breaking changes or deprecated APIs between .NET 8 and .NET 10

## Tasks

| # | Task | Status |
|---|------|--------|
| 1 | Upgrade .NET from net8.0 to net10.0 | Pending |
