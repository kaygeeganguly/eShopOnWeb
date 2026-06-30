# 02-blazorshared: Upgrade BlazorShared to net10.0

## Objective
Upgrade BlazorShared (foundation library, Level 0) to net10.0.

## Research Findings
- All projects inherit TargetFramework from Directory.Packages.props (no per-project TFM)
- Since Directory.Packages.props was updated in task 01 to net10.0, BlazorShared is already targeting net10.0
- No package issues, no API issues — pure TFM upgrade
- BlazorShared.csproj has no explicit TargetFramework element

## Done When
- BlazorShared targets net10.0 ✅ (via Directory.Packages.props)
- Builds without errors or warnings ✅
