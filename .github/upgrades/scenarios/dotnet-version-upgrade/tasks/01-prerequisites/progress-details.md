# 01-prerequisites progress details

## Summary
- Updated `global.json` to pin the repository to the installed .NET 10 SDK `10.0.301` with `rollForward` still set to `latestFeature`.
- Updated `Directory.Packages.props` to switch the centralized `TargetFramework` property to `net10.0`.
- Updated central version properties to .NET 10-compatible values:
  - `AspNetVersion` → `10.0.9`
  - `SystemExtensionVersion` → `10.0.9`
  - `EntityFramworkCoreVersion` → `10.0.9`
  - `VSCodeGeneratorVersion` → `10.0.2`
- Updated direct central package versions:
  - `Azure.Identity` → `1.21.0`
  - `System.Text.Json` → `10.0.9`
  - `System.IdentityModel.Tokens.Jwt` → `8.19.1`
  - `Microsoft.NET.Test.Sdk` → `18.7.0`
- Removed the obsolete CPM entry for `Microsoft.AspNetCore.Mvc`.

## Validation
- Verified SDK availability with `dotnet --version` (`10.0.301`) and `dotnet --list-sdks` (includes multiple 10.0 SDKs).
- Ran `dotnet restore /home/runner/work/eShopOnWeb/eShopOnWeb/eShopOnWeb.sln` successfully after the updates.

## Notes / follow-up
- Attempting to remove the CPM entries for `System.Security.Claims` and `Microsoft.VisualStudio.Azure.Containers.Tools.Targets` caused `NU1010` because `src/ApplicationCore/ApplicationCore.csproj` and `src/PublicApi/PublicApi.csproj` still contain matching `PackageReference` items. Those package removals need to happen in a later task that is allowed to edit project files.
- Restore completed with existing vulnerability warnings for unrelated packages (`AutoMapper`, `NuGet.Packaging`, `NuGet.Protocol`) and prune warnings for framework-provided packages (`System.Security.Claims`, `System.Text.Json`, `System.Net.Http.Json`).
