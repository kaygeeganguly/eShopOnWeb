# .NET Version Upgrade

## Preferences
- **Flow Mode**: Automatic
- **Target Framework**: net10.0

## Strategy
**Selected**: Top-Down (Application-First)
**Rationale**: 5-tier dependency graph and 3 projects with binary/source-incompatible API changes exceed the ≤3-tier and ≤2 high-risk thresholds for All-at-Once.

### Execution Constraints
- Upgrade shared libraries (BlazorShared, ApplicationCore, Infrastructure, BlazorAdmin) before upgrading applications (Web, PublicApi)
- Fix all binary/source-incompatible API calls inline — no stubs, no deferral
- Resolve incompatible package (Microsoft.VisualStudio.Azure.Containers.Tools.Targets) by removal
- Validate build after each task group before proceeding to next
- Commit strategy: After Each Task (Bottom-Up tasks validate independently)

## Upgrade Options
**Source**: .github/upgrades/scenarios/dotnet-version-upgrade/upgrade-options.md

### Strategy
- Upgrade Strategy: Top-Down

### Compatibility
- Unsupported Packages: Resolve Inline (1 incompatible package)
- Unsupported API Handling: Fix Inline

## Source Control
- **Source Branch**: main
- **Working Branch**: upgrade-dotnet-10
- **Commit Strategy**: After Each Task
- **Branch Sync**: Auto (Merge)
