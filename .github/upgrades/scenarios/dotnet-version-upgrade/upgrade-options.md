# Upgrade Options — eShopOnWeb

Assessment: 10 projects (net8.0 → net10.0), 6-tier dependency graph, 5 incompatible/deprecated packages, 12 breaking API changes (9 binary, 3 source incompatible), all SDK-style with CPM.

## Strategy

### Upgrade Strategy
The solution's 6-tier dependency graph exceeds the ≤3-tier threshold for All-at-Once; incremental tier-by-tier validation is warranted to keep the solution buildable throughout.

| Value | Description |
|-------|-------------|
| **Top-Down** (selected) | Upgrade entry-point applications first, temporarily multi-targeting shared libraries so the solution stays buildable throughout; libraries are consolidated in a second phase after all apps are upgraded. |
| All-at-Once | Upgrade all projects simultaneously in a single atomic pass — fastest, no multi-targeting overhead, but the solution may be temporarily broken until all projects are updated. |

## Compatibility

### Unsupported Packages
5 packages are flagged as incompatible or deprecated (`Microsoft.VisualStudio.Azure.Containers.Tools.Targets`, `AutoMapper.Extensions.Microsoft.DependencyInjection`, `System.IdentityModel.Tokens.Jwt`, `xunit`, `xunit.runner.console`), exceeding the 3-package threshold where inline resolution becomes impractical.

| Value | Description |
|-------|-------------|
| **Defer Resolution** (selected) | Make each incompatible package compile without the package by generating minimal type stubs, then create dedicated follow-up tasks for real replacements — no incompatible package blocks the initial upgrade pass. |
| Resolve Inline | Research and resolve each incompatible package within the same task — suitable when 1–3 packages lack known replacements. |
| Compatibility Mode | Keeps .NET Framework reference, adds `Microsoft.NETFramework.ReferenceAssemblies`, and suppresses NU1701 — use only for transitive dependencies where consuming code does not call package APIs directly. |

### Unsupported API Handling
12 breaking API changes were identified (9 binary incompatible including `ConfigurationBinder.Get` / `Configure`, 3 source incompatible including `Exception` serialization constructor and `TimeSpan.FromMinutes`); this is a modern-to-modern upgrade and the affected APIs are well-understood with clear replacements.

| Value | Description |
|-------|-------------|
| **Fix Inline** (selected) | Resolve every API change in the same task, including complex ones — may take longer per task but leaves no deferred work and no stubs to clean up later. |
| Defer Complex Changes | Apply simple replacements inline; for complex changes generate a minimal compilable stub and create a dedicated resolution subtask. |
