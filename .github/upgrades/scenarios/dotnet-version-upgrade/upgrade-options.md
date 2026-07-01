# Upgrade Options — eShopOnWeb

Assessment: 10 projects (net8.0), 5-tier dependency graph, 3 projects with breaking API changes, 1 incompatible package, security vulnerabilities in Azure.Identity

## Strategy

### Upgrade Strategy
Solution has a 5-tier dependency graph and 3 projects with binary/source-incompatible API changes (Web, PublicApi, ApplicationCore), exceeding the ≤3-tier and ≤2 high-risk thresholds for All-at-Once.

| Value | Description |
|-------|-------------|
| **Top-Down** (selected) | Upgrade entry-point applications first; libraries upgraded alongside their app consumers. No multi-targeting needed for modern-to-modern. |
| All-at-Once | Upgrade all projects simultaneously. Simpler but risk is higher given deep graph and API issues. |

## Compatibility

### Unsupported Packages
1 incompatible package identified: `Microsoft.VisualStudio.Azure.Containers.Tools.Targets` has no compatible version for net10.0.

| Value | Description |
|-------|-------------|
| **Resolve Inline** (selected) | Research and resolve the incompatible package within the same task — small count (1 package) makes this practical. |
| Defer Resolution | Generate minimal stubs to keep project compiling, create follow-up tasks for real replacements. |

### Unsupported API Handling
Breaking API changes flagged in Web (binary+source), PublicApi (binary), and ApplicationCore (source). Modern-to-modern upgrades typically have minor, fixable changes.

| Value | Description |
|-------|-------------|
| **Fix Inline** (selected) | Resolve every API change in the same task, including complex ones. Best for modern-to-modern where changes are minor. |
| Defer Complex Changes | Apply simple replacements inline; stub complex changes and create follow-up tasks. |
