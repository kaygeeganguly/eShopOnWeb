# .NET Version Upgrade

## Preferences
- **Flow Mode**: Automatic
- **Target Framework**: net10.0

## Source Control
- **Source Branch**: main
- **Working Branch**: dotnet-version-upgrade
- **Commit Strategy**: After Each Task
- **Branch Sync**: Auto (Merge)

## Upgrade Options
- **Upgrade Strategy**: Bottom-Up
- **Unsupported Packages**: Defer Resolution
- **Unsupported API Handling**: Fix Inline

## Strategy
**Selected**: Bottom-Up (Dependency-First)
**Rationale**: 10 projects across a 6-tier dependency graph (BlazorShared → ApplicationCore/BlazorAdmin → Infrastructure → PublicApi/Web → test tier → IntegrationTests). Upgrading from leaf nodes ensures each library is stable on net10.0 before consuming projects are upgraded. All projects are modern SDK-style targeting net8.0, making this a straightforward TFM bump with inline API fixes at each tier.

### Execution Constraints
- Each tier must build cleanly before the next tier begins — do not upgrade Tier N+1 until Tier N is verified
- Package versions are updated centrally in Directory.Packages.props during task 01 (prerequisites); individual project tasks only change the TFM and fix inline API issues
- Deprecated packages (AutoMapper.Extensions.Microsoft.DependencyInjection, System.IdentityModel.Tokens.Jwt, xunit, xunit.runner.console) are deferred — leave them in place, do not attempt replacement
- Microsoft.VisualStudio.Azure.Containers.Tools.Targets must be removed from PublicApi (incompatible); System.Security.Claims must be removed from ApplicationCore (now framework-included)
- Azure.Identity must be upgraded to 1.21.0 during prerequisites (security vulnerability in current version 1.10.4)
