# .NET Version Upgrade Plan

## Overview

**Target**: Upgrade eShopOnWeb from net8.0 to net10.0
**Scope**: 10 projects across src/ and tests/, all on modern .NET (net8.0 → net10.0)

### Selected Strategy
**Top-Down (Application-First)** — Applications upgraded with priority attention to their issues; shared libraries upgraded alongside their app consumers. No multi-targeting needed for this modern-to-modern upgrade.
**Rationale**: 5-tier dependency graph and 3 projects with binary/source-incompatible API changes (Web, PublicApi, ApplicationCore) require careful, ordered approach with per-group validation.

## Tasks

### 01-prerequisites: Verify SDK and update global configuration

Verify the .NET 10 SDK is installed, update `global.json` to target the .NET 10 SDK, and update the centralized version variables in `Directory.Packages.props` (the solution already uses CPM). The properties `AspNetVersion`, `EntityFramworkCoreVersion`, `SystemExtensionVersion`, and `VSCodeGeneratorVersion` all need updating to their net10.0-compatible versions. The `TargetFramework` property in `Directory.Packages.props` should also be updated to `net10.0`.

Package version targets: `Microsoft.AspNetCore.*` → 10.0.9, `Microsoft.EntityFrameworkCore.*` → 10.0.9, `System.Net.Http.Json` + `System.Text.Json` → 10.0.9, `Microsoft.VisualStudio.Web.CodeGeneration.Design` → 10.0.2. Also update `Azure.Identity` (1.10.4 → 1.21.0, security vulnerability fix) and `System.IdentityModel.Tokens.Jwt` (7.3.1 → 8.19.1, deprecated). Remove `System.Security.Claims` (4.3.0) — its functionality is included in the framework reference (NuGet.0003). Remove `Microsoft.VisualStudio.Azure.Containers.Tools.Targets` (1.19.6) — incompatible with net10.0 with no compatible version available.

**Done when**: `global.json` targets a .NET 10 SDK, `Directory.Packages.props` has `TargetFramework=net10.0` and all version variables updated; `dotnet --version` confirms a .NET 10 SDK is available.

---

### 02-shared-libraries: Upgrade BlazorShared, ApplicationCore, Infrastructure, and BlazorAdmin

Upgrade the four foundational library projects to net10.0. These are dependency-order prerequisites for the application tasks.

**BlazorShared** (`src/BlazorShared`) — minimal issues, just a TFM change. **ApplicationCore** (`src/ApplicationCore`) — has 2 issues: a source-incompatible API (Api.0002) and a security vulnerability in `Azure.Identity` (addressed in prerequisites). Investigate which API is source-incompatible and fix it. **Infrastructure** (`src/Infrastructure`) — package upgrades only (no API issues), references ApplicationCore. **BlazorAdmin** (`src/BlazorAdmin`) — has 1 behavioral API change (Api.0003) plus package upgrades for `Microsoft.AspNetCore.Components.*` and `Blazored.LocalStorage`; investigate the behavioral change in the Blazor component pipeline and resolve as needed.

Research starting points: check `ApplicationCore` for any usage of APIs removed between .NET 8 and .NET 10; check `BlazorAdmin` for `Api.0003` behavioral changes in the Blazor hosting model or component lifecycle.

**Done when**: All four projects (`BlazorShared`, `ApplicationCore`, `Infrastructure`, `BlazorAdmin`) build successfully targeting net10.0 with no errors and no warnings in modified files. API fixes applied inline.

---

### 03-web-app: Upgrade the ASP.NET Core Web application

Upgrade `src/Web/Web.csproj` to net10.0. This is the most complex project with 29 issues including binary-incompatible and source-incompatible API changes.

Binary-incompatible APIs (must fix to compile): `IConfiguration.Get<T>()` at `ConfigureCoreServices.cs:21` and `IServiceCollection.Configure<T>(IConfiguration)` at `ConfigureWebServices.cs:15` — these extension method signatures changed between .NET 8 and .NET 10 (ref: `ConfigurationBinder.Get<T>` and `OptionsConfigurationServiceCollectionExtensions.Configure<T>`). Source-incompatible: `TimeSpan.FromMinutes(double)` at `ConfigureCookieSettings.cs:25` — the return type or overload resolution changed. Behavioral API changes in `HomePageHealthCheck.cs:26` and `ApiHealthCheck.cs:25` (`HttpContent.ReadAsStringAsync()`) and `Program.cs:179` (`IApplicationBuilder.UseExceptionHandler(string)`) — review and adapt if needed.

Package updates: `AutoMapper.Extensions.Microsoft.DependencyInjection` is deprecated (its functionality is merged into AutoMapper directly — check usage patterns), `Azure.Identity` security fix (already in Directory.Packages.props from Task 01), `System.IdentityModel.Tokens.Jwt` deprecated upgrade.

**Done when**: `src/Web/Web.csproj` builds targeting net10.0 with no errors. All binary/source-incompatible API calls replaced. No warnings in modified files.

---

### 04-public-api: Upgrade the PublicApi application

Upgrade `src/PublicApi/PublicApi.csproj` to net10.0. This project has 17 issues including binary-incompatible APIs and one incompatible package.

Binary-incompatible APIs in `Program.cs` (must fix to compile): `configSection.Get<BaseUrlConfiguration>()` at line 49, `builder.Services.Configure<BaseUrlConfiguration>(configSection)` at line 48, `builder.Configuration.Get<CatalogSettings>()` at line 42, and `builder.Services.Configure<CatalogSettings>(builder.Configuration)` at line 41 — same `ConfigurationBinder`/`OptionsConfigurationServiceCollectionExtensions` pattern as Web. Behavioral change at `Program.cs:31` for `ILoggingBuilder.AddConsole()`.

Incompatible package: `Microsoft.VisualStudio.Azure.Containers.Tools.Targets` 1.19.6 — no net10.0-compatible version exists; remove the package reference from the project file (it is a build-time Visual Studio tooling package, not a runtime dependency). Deprecated packages: `AutoMapper.Extensions.Microsoft.DependencyInjection` and `System.IdentityModel.Tokens.Jwt` (already updated via CPM in Task 01, but verify no direct project references).

**Done when**: `src/PublicApi/PublicApi.csproj` builds targeting net10.0 with no errors. All binary-incompatible APIs fixed. `Microsoft.VisualStudio.Azure.Containers.Tools.Targets` removed. No warnings in modified files.

---

### 05-test-projects: Upgrade all test projects

Upgrade the four test projects to net10.0: `tests/UnitTests`, `tests/IntegrationTests`, `tests/FunctionalTests`, `tests/PublicApiIntegrationTests`.

**UnitTests** — deprecated package `xunit` (2.7.0) flagged; check if updated xunit version is needed or if the deprecated marker is informational. **IntegrationTests** — deprecated packages (`xunit`, `xunit.runner.console`) and EF package upgrades (handled via CPM). **FunctionalTests** — 1 mandatory issue (likely TFM), behavioral API changes (Api.0003), and deprecated packages. **PublicApiIntegrationTests** — behavioral API changes and package upgrades.

All test package versions (`Microsoft.AspNetCore.Mvc.Testing`, `Microsoft.NET.Test.Sdk`, etc.) should already be updated via `Directory.Packages.props` from Task 01. Check if any test projects have version overrides at the project level that conflict with CPM centralization.

**Done when**: All four test projects build targeting net10.0 with no errors. The solution builds cleanly end-to-end.

---

### 06-final-validation: Build solution and run all tests

Run a full solution build to confirm no cross-project issues remain. Then run the full test suite to verify no regressions were introduced by the API behavioral changes fixed in earlier tasks.

Special attention: API behavioral changes (Api.0003) were flagged in `HttpContent.ReadAsStringAsync()` calls and `ILoggingBuilder.AddConsole()` — verify health check tests and any logging-related tests still pass. Confirm `Microsoft.VisualStudio.Azure.Containers.Tools.Targets` removal does not affect any build targets that tests depend on.

**Done when**: `dotnet build eShopOnWeb.sln` completes with zero errors and zero warnings in modified projects. `dotnet test eShopOnWeb.sln` passes all unit and integration tests. All 10 projects targeting net10.0.
