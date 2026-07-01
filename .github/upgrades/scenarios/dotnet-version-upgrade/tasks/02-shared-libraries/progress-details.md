# 02-shared-libraries progress

## Summary
Completed the shared library upgrade work for BlazorShared, ApplicationCore, Infrastructure, and BlazorAdmin.

## Changes made
- Added explicit `<TargetFramework>net10.0</TargetFramework>` to:
  - `src/BlazorShared/BlazorShared.csproj`
  - `src/ApplicationCore/ApplicationCore.csproj`
  - `src/Infrastructure/Infrastructure.csproj`
  - `src/BlazorAdmin/BlazorAdmin.csproj`
- Removed redundant framework-provided package references from `src/ApplicationCore/ApplicationCore.csproj`:
  - `System.Security.Claims`
  - `System.Text.Json`
- Removed the obsolete formatter-based serialization constructor from `src/ApplicationCore/Exceptions/EmptyBasketOnCheckoutException.cs` to eliminate the .NET 10 warning tied to legacy serialization APIs.
- Removed redundant framework-provided package reference `System.Net.Http.Json` from `src/BlazorAdmin/BlazorAdmin.csproj`.
- Updated `src/BlazorAdmin/Program.cs` to build the host once and clear local storage through `host.Services` instead of creating a second service provider from `builder.Services`. This avoids startup/service-pipeline behavioral issues in newer Blazor hosting behavior.

## Research notes
- The assessment query tool was unavailable in this session because the upgrade state manager was not initialized, so issue research was completed by inspecting project files, affected source files, and the build output.
- ApplicationCore’s actionable compatibility issue surfaced as the legacy exception serialization path warning under .NET 10.
- BlazorAdmin’s likely behavioral risk area was startup/service initialization; the app was manually building an extra service provider before `Build()`, which is fragile with current Blazor hosting behavior.

## Validation
Executed restore/build in dependency order:
1. `dotnet restore src/BlazorShared/BlazorShared.csproj`
2. `dotnet build src/BlazorShared/BlazorShared.csproj`
3. `dotnet restore src/ApplicationCore/ApplicationCore.csproj`
4. `dotnet build src/ApplicationCore/ApplicationCore.csproj`
5. `dotnet restore src/Infrastructure/Infrastructure.csproj`
6. `dotnet build src/Infrastructure/Infrastructure.csproj`
7. `dotnet restore src/BlazorAdmin/BlazorAdmin.csproj`
8. `dotnet build src/BlazorAdmin/BlazorAdmin.csproj`

## Results
- `BlazorShared`: build succeeded, 0 warnings, 0 errors
- `ApplicationCore`: build succeeded, 0 warnings, 0 errors
- `Infrastructure`: build succeeded, 0 warnings, 0 errors
- `BlazorAdmin`: build succeeded, 0 warnings, 0 errors
