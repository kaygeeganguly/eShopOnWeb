## Files Modified
- global.json — SDK version changed from 8.0.x to 10.0.x
- Directory.Packages.props — Updated all version variables and package entries
- src/ApplicationCore/ApplicationCore.csproj — Removed System.Security.Claims PackageReference
- src/PublicApi/PublicApi.csproj — Removed Microsoft.VisualStudio.Azure.Containers.Tools.Targets PackageReference

## Build Result
- Errors: 0 (restore only — TFM changes in subsequent tasks)
- Warnings: Security vulnerability warnings for AutoMapper and NuGet packages (deferred/known)
- dotnet restore eShopOnWeb.sln: SUCCESS

## Test Result
- N/A for this task (prerequisites only — no TFM changes)

## Changes Summary
- Updated global.json SDK to 10.0.x
- Updated Directory.Packages.props: AspNetVersion→10.0.9, SystemExtensionVersion→10.0.9, EntityFramworkCoreVersion→10.0.9, VSCodeGeneratorVersion→10.0.2
- Azure.Identity updated to 1.21.0 (security vulnerability resolved)
- System.IdentityModel.Tokens.Jwt updated to 8.0.1 (resolves transitive dep conflict with JwtBearer 10.0.9)
- Removed System.Security.Claims from CPM and ApplicationCore.csproj (framework-included)
- Removed Microsoft.VisualStudio.Azure.Containers.Tools.Targets from CPM and PublicApi.csproj (incompatible)
- Removed Microsoft.AspNetCore.Mvc 2.2.0 (stale/unused entry)
- Deprecated packages (AutoMapper, xunit, xunit.runner.console) left in place per deferred strategy

## Issues Encountered
- System.IdentityModel.Tokens.Jwt 7.3.1 caused NU1605 downgrade error — updated to 8.0.1 (available in cache)
