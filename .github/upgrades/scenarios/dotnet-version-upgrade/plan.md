# .NET Version Upgrade Plan

## Overview

**Target**: Upgrade eShopOnWeb from net8.0 to net10.0 across all 10 projects  
**Scope**: Modern .NET solution — all projects are SDK-style and use centralized package management (Directory.Packages.props). 302 code files, 48 NuGet packages (24 requiring attention), 117 total issues including 9 binary-incompatible APIs, 3 source-incompatible APIs, and 49 behavioral changes.

### Selected Strategy
**Bottom-Up (Dependency-First)** — Upgrade from leaf nodes to root applications, tier by tier.  
**Rationale**: 10 projects with a 6-tier dependency graph. Organizing work bottom-up ensures each library is stable on net10.0 before the projects that consume it are upgraded.

### Dependency Graph

```
Tier 1: BlazorShared
Tier 2: ApplicationCore  → BlazorShared
        BlazorAdmin      → BlazorShared
Tier 3: Infrastructure   → ApplicationCore
Tier 4: PublicApi        → ApplicationCore, Infrastructure
        Web              → ApplicationCore, BlazorAdmin, BlazorShared, Infrastructure
Tier 5: FunctionalTests           → ApplicationCore, PublicApi, Web
        PublicApiIntegrationTests → PublicApi, Web
        UnitTests                 → ApplicationCore, Web
Tier 6: IntegrationTests          → Infrastructure, UnitTests
```

### Tier Summary

| Tier | Projects | Upgrade Scope |
|------|----------|---------------|
| 1 | BlazorShared | TFM only — no package or API issues |
| 2 | ApplicationCore | TFM + 3 package upgrades + source-incompatible API fixes + remove framework-included package |
|   | BlazorAdmin | TFM + 7 package upgrades + behavioral API review |
| 3 | Infrastructure | TFM + 4 package upgrades; deprecated System.IdentityModel.Tokens.Jwt deferred |
| 4 | PublicApi | TFM + 11 package issues + binary/behavioral API fixes + incompatible package removal |
|   | Web | TFM + 13 package issues + binary/source/behavioral API fixes + security vulnerability resolved |
| 5 | FunctionalTests | TFM + 3 package upgrades + 29 behavioral API fixes (HttpContent/Uri) |
|   | PublicApiIntegrationTests | TFM + 1 package upgrade + 7 behavioral API fixes |
|   | UnitTests | TFM + 2 package upgrades — no API issues |
| 6 | IntegrationTests | TFM + 2 package upgrades — no API issues |

## Tasks

### 01-prerequisites: Update SDK and central package versions

Update the project-wide toolchain and centralized package registry before any individual project is touched. This ensures all subsequent project upgrades operate against consistent, net10.0-aligned package versions and toolchain from the outset.

Update `global.json` to require a .NET 10 SDK (10.0.x). In `Directory.Packages.props`, bump all 19 upgrade-recommended packages to their target versions: all Microsoft.AspNetCore.* and Microsoft.EntityFrameworkCore.* packages from 8.0.x to 10.0.9, Microsoft.VisualStudio.Web.CodeGeneration.Design from 8.0.0 to 10.0.2, Azure.Identity from 1.10.4 to 1.21.0 (resolves a known security vulnerability), System.Net.Http.Json and System.Text.Json to 10.0.9. Also remove the `System.Security.Claims` entry from Directory.Packages.props — its functionality is now part of the framework reference and the package is no longer needed. Remove the `Microsoft.VisualStudio.Azure.Containers.Tools.Targets` entry from Directory.Packages.props — it is incompatible with net10.0 and will be removed from its consuming project (PublicApi) in task 06. Do not attempt to replace deprecated packages (AutoMapper.Extensions.Microsoft.DependencyInjection, System.IdentityModel.Tokens.Jwt, xunit, xunit.runner.console) — these are deferred per the agreed resolution strategy.

**Done when**: `global.json` references a .NET 10 SDK; all 19 upgrade-recommended packages reflect their target versions in `Directory.Packages.props`; the incompatible and framework-included package entries are removed from `Directory.Packages.props`; the solution restores without errors (individual TFM changes come in subsequent tasks).

---

### 02-blazorshared: Upgrade BlazorShared to net10.0

BlazorShared is the foundation of the solution — a pure class library with no internal project dependencies, no package issues, and no API issues. It is the simplest upgrade in the solution and unblocks all Tier 2 projects.

Change `<TargetFramework>` from `net8.0` to `net10.0` in `src/BlazorShared/BlazorShared.csproj`. No code changes are expected. Restore and build to confirm a clean baseline.

**Done when**: `src/BlazorShared/BlazorShared.csproj` targets `net10.0` and builds without errors or warnings.

---

### 03-applicationcore: Upgrade ApplicationCore to net10.0

ApplicationCore is a class library that depends on BlazorShared. It has a small set of package issues and two source-incompatible API changes that require inline code fixes before the project will compile cleanly.

Change `<TargetFramework>` to `net10.0` in `src/ApplicationCore/ApplicationCore.csproj`. Remove the explicit `<PackageReference>` for `System.Security.Claims` — this package's functionality is now part of the framework reference and the standalone package is no longer needed. The remaining package upgrades (System.Text.Json to 10.0.9) are already applied centrally in task 01.

For API fixes: `System.Exception` lost its serialization constructor `Exception(SerializationInfo, StreamingContext)` — any custom exception classes that override this constructor must be updated, typically by removing the obsolete overload or replacing it with a supported alternative. `System.TimeSpan.FromMinutes(double)` became source-incompatible — confirm all call sites compile correctly and apply explicit casts or method replacements where needed.

**Done when**: `ApplicationCore` targets `net10.0`; the `System.Security.Claims` package reference is removed from the project file; both source-incompatible API issues are resolved; the project builds warning-free.

---

### 04-blazoradmin: Upgrade BlazorAdmin to net10.0

BlazorAdmin is a Blazor WebAssembly project that depends only on BlazorShared. It has 7 package upgrades (all Microsoft Blazor/Extensions packages) and 3 behavioral API changes that require review and testing.

Change `<TargetFramework>` to `net10.0` in `src/BlazorAdmin/BlazorAdmin.csproj`. All 7 package upgrades (Microsoft.AspNetCore.Components.Authorization, Microsoft.AspNetCore.Components.WebAssembly, Microsoft.AspNetCore.Components.WebAssembly.Authentication, Microsoft.AspNetCore.Components.WebAssembly.DevServer, Microsoft.Extensions.Identity.Core, Microsoft.Extensions.Logging.Configuration, System.Net.Http.Json) are already applied centrally via task 01.

Behavioral changes to review: `ConsoleLoggerExtensions.AddConsole` behavior changed — confirm logging configuration still produces expected output. `System.Net.Http.HttpContent` behavioral changes may affect how Blazor HTTP calls handle content reading and headers — review any code that reads or constructs HTTP content. `System.Uri` behavioral changes may affect URL construction or navigation — verify any Uri-based routing logic.

**Done when**: `BlazorAdmin` targets `net10.0`, builds without errors or warnings, and all behavioral API usage has been reviewed and verified correct.

---

### 05-infrastructure: Upgrade Infrastructure to net10.0

Infrastructure is a class library that depends on ApplicationCore. It has 4 package issues — 3 framework packages upgraded centrally and 1 deprecated package to defer. No API code changes are needed.

Change `<TargetFramework>` to `net10.0` in `src/Infrastructure/Infrastructure.csproj`. The 3 framework package upgrades (Microsoft.AspNetCore.Identity.EntityFrameworkCore, Microsoft.EntityFrameworkCore.InMemory, Microsoft.EntityFrameworkCore.SqlServer) are already applied via task 01. `System.IdentityModel.Tokens.Jwt` is deprecated — per the agreed deferral strategy, leave it in place and note it as a deferred migration item (recommended future migration: Microsoft.IdentityModel.Tokens from the `Microsoft.Identity.Web` family). No API issues are reported for this project.

**Done when**: `Infrastructure` targets `net10.0`, builds without errors or warnings, and the deprecated `System.IdentityModel.Tokens.Jwt` dependency is documented as a known deferred item.

---

### 06-publicapi: Upgrade PublicApi to net10.0

PublicApi is an ASP.NET Core Minimal API project depending on ApplicationCore and Infrastructure. It has the second-highest package issue count (11) and carries both binary-incompatible and behavioral API changes that require inline code fixes.

Change `<TargetFramework>` to `net10.0` in `src/PublicApi/PublicApi.csproj`. Remove the `<PackageReference>` for `Microsoft.VisualStudio.Azure.Containers.Tools.Targets` from the project file — this package is incompatible with net10.0 and serves only a Visual Studio tooling purpose; its removal does not affect runtime behavior. All remaining package upgrades are covered by task 01. `AutoMapper.Extensions.Microsoft.DependencyInjection` is deprecated — deferred per strategy, leave in place.

For binary-incompatible API fixes: `ConfigurationBinder.Get<T>(IConfiguration)` has a breaking signature change — update all call sites to the new overload form. `OptionsConfigurationServiceCollectionExtensions.Configure<T>(IServiceCollection, IConfiguration)` similarly changed — update registrations in the DI setup. `ConfigurationBinder.GetValue(IConfiguration, Type, string)` also changed — update any direct uses.

Behavioral changes: `System.Uri` constructor behavior changed — review all endpoint and base-address Uri construction. `UseExceptionHandler(IApplicationBuilder, string)` behavior changed — verify the exception-handling middleware path is correctly configured and produces the expected responses.

**Done when**: `PublicApi` targets `net10.0`; the incompatible package reference is removed; all binary-incompatible API call sites are updated; the project builds and starts without errors.

---

### 07-web: Upgrade Web to net10.0

Web is the main ASP.NET Core web application and carries the broadest change surface in the solution — 13 package issues and 15 API issues spanning binary-incompatible, source-incompatible, and behavioral categories.

Change `<TargetFramework>` to `net10.0` in `src/Web/Web.csproj`. All package upgrades are covered by task 01. `AutoMapper.Extensions.Microsoft.DependencyInjection` and `System.IdentityModel.Tokens.Jwt` are deprecated — deferred per strategy.

For binary-incompatible API fixes (same APIs as PublicApi): update all `ConfigurationBinder.Get<T>` and `Configure<T>(IServiceCollection, IConfiguration)` call sites to the new overload forms. For source-incompatible fixes: remove or update any custom exception constructors using `Exception(SerializationInfo, StreamingContext)`; verify and fix any `TimeSpan.FromMinutes(double)` call sites affected by the signature change.

Behavioral changes are the largest category: `System.Net.Http.HttpContent` handling changed — review all `HttpClient`-based code, especially response-content reading, to confirm behavior is preserved. `System.Uri` behavioral changes — validate URL construction and any base-address configuration. `ConsoleLoggerExtensions.AddConsole` — confirm logging setup. `UseExceptionHandler` — confirm exception middleware configuration.

**Done when**: `Web` targets `net10.0`; all binary and source-incompatible APIs are fixed; behavioral API usage is reviewed; the project builds and starts without errors.

---

### 08-test-projects: Upgrade all test projects to net10.0

Four test projects need upgrading. FunctionalTests carries the highest individual issue count in the solution (29 behavioral API issues). IntegrationTests and UnitTests have minimal issues. All four use deprecated xunit packages — deferred per strategy.

Change `<TargetFramework>` to `net10.0` in all four test project files: `tests/FunctionalTests/FunctionalTests.csproj`, `tests/IntegrationTests/IntegrationTests.csproj`, `tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj`, and `tests/UnitTests/UnitTests.csproj`. Package upgrades (Microsoft.AspNetCore.Mvc.Testing, Microsoft.EntityFrameworkCore.InMemory) are already applied via task 01. `xunit` and `xunit.runner.console` are deprecated — deferred per strategy.

For FunctionalTests: 29 `System.Net.Http.HttpContent` behavioral issues dominate. HttpContent behavior changed around content headers and stream handling — review all test code that reads response content or constructs request bodies. `System.Uri` usage should also be validated. For PublicApiIntegrationTests: 7 behavioral API issues follow the same HttpContent/Uri pattern. IntegrationTests and UnitTests have no API issues; verify they build and tests pass.

Run all four test suites after upgrading. Failures here may surface behavioral impacts not caught by inline fixes in earlier tasks.

**Done when**: All four test projects target `net10.0`; all test projects build without errors; all tests pass (or failing tests are documented with root-cause analysis).

---

### 09-final-validation: Full solution build and test suite validation

Verify the entire solution builds cleanly and the complete test suite passes after all 10 projects have been upgraded to net10.0.

Build the full solution (`eShopOnWeb.sln`) and confirm zero build errors and zero warnings. Run the complete test suite across all four test projects. Confirm Azure.Identity is at version 1.21.0 (security vulnerability resolved). Document all deferred items: deprecated packages (AutoMapper.Extensions.Microsoft.DependencyInjection, System.IdentityModel.Tokens.Jwt, xunit, xunit.runner.console) with their recommended future migration paths.

**Done when**: `dotnet build eShopOnWeb.sln` completes with no errors or warnings; `dotnet test` passes for all test projects; deferred package deprecations are documented in a final summary.
