# .NET Upgrade Plan: net8.0 → net10.0

## Overview

Upgrade the **eShopOnWeb** solution from **.NET 8.0** to **.NET 10.0 (LTS)**.

The target framework is currently defined centrally in `Directory.Packages.props` as `net8.0`, and the SDK version is pinned to `8.0.x` in `global.json`. This upgrade will update all projects to `net10.0`, refresh NuGet packages to .NET 10-compatible versions, and resolve any breaking API changes.

## Source Version

- **Current .NET version**: `net8.0` (SDK `8.0.x`)

## Target Version

- **Target .NET version**: `net10.0` (LTS)

## Projects in Solution

### Source Projects

| Project | Path |
|---------|------|
| ApplicationCore | `src/ApplicationCore/ApplicationCore.csproj` |
| Web | `src/Web/Web.csproj` |
| BlazorAdmin | `src/BlazorAdmin/BlazorAdmin.csproj` |
| BlazorShared | `src/BlazorShared/BlazorShared.csproj` |
| Infrastructure | `src/Infrastructure/Infrastructure.csproj` |
| PublicApi | `src/PublicApi/PublicApi.csproj` |

### Test Projects

| Project | Path |
|---------|------|
| UnitTests | `tests/UnitTests/UnitTests.csproj` |
| IntegrationTests | `tests/IntegrationTests/IntegrationTests.csproj` |
| PublicApiIntegrationTests | `tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj` |
| FunctionalTests | `tests/FunctionalTests/FunctionalTests.csproj` |

## Upgrade Scope

1. **`Directory.Packages.props`** — Update `<TargetFramework>` from `net8.0` to `net10.0` and bump version variables (`AspNetVersion`, `SystemExtensionVersion`, `EntityFrameworkCoreVersion`, etc.) to .NET 10-compatible versions.
2. **`global.json`** — Update SDK version from `8.0.x` to `10.0.x`.
3. **NuGet packages** — Update all `Microsoft.*`, `Azure.*`, and third-party packages to versions that support `net10.0`.
4. **API compatibility** — Address any breaking changes between .NET 8 and .NET 10 (ASP.NET Core, EF Core, Blazor WebAssembly, JWT, Identity).
5. **Build validation** — Ensure all projects compile and all tests pass after the upgrade.

## Tasks

| # | Task | Status |
|---|------|--------|
| 1 | Upgrade eShopOnWeb from net8.0 to net10.0 | Pending |
