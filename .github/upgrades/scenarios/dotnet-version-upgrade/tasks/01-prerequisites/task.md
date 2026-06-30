# 01-prerequisites: Update SDK and central package versions

## Objective
Update the project-wide toolchain and centralized package registry before individual projects are touched.

## Scope
- `global.json` — SDK version
- `Directory.Packages.props` — all package versions
- `src/ApplicationCore/ApplicationCore.csproj` — remove System.Security.Claims reference
- `src/PublicApi/PublicApi.csproj` — remove Microsoft.VisualStudio.Azure.Containers.Tools.Targets reference

## Research Findings

### Files Modified
- `global.json` — Update SDK from 8.0.x to 10.0.x
- `Directory.Packages.props` — Update all versioned properties and package entries

### Packages Updated
| Package | Old | New |
|---------|-----|-----|
| AspNetVersion variable | 8.0.2 | 10.0.9 |
| SystemExtensionVersion variable | 8.0.0 | 10.0.9 |
| EntityFramworkCoreVersion variable | 8.0.2 | 10.0.9 |
| VSCodeGeneratorVersion variable | 8.0.0 | 10.0.2 |
| Azure.Identity | 1.10.4 | 1.21.0 (security fix) |
| System.IdentityModel.Tokens.Jwt | 7.3.1 | 8.0.1 (required by JwtBearer 10.0.9 transitive dep) |

### Packages Removed from CPM
- System.Security.Claims — included in framework, also removed from ApplicationCore.csproj
- Microsoft.VisualStudio.Azure.Containers.Tools.Targets — incompatible, also removed from PublicApi.csproj
- Microsoft.AspNetCore.Mvc 2.2.0 — old/unused entry

### Packages Deferred (deprecated but left in place)
- AutoMapper.Extensions.Microsoft.DependencyInjection — deferred
- xunit, xunit.runner.console — deferred

## Done When
- global.json references .NET 10 SDK ✅
- All 19 upgrade-recommended packages reflect target versions ✅
- Incompatible packages removed from Directory.Packages.props ✅
- Solution restores without errors ✅
