# .NET Version Upgrade Progress

## Overview

Upgrading eShopOnWeb from net8.0 to net10.0 across all 10 projects using a Bottom-Up (Dependency-First) strategy. Projects are upgraded tier by tier from leaf libraries through to web applications and test projects, ensuring each dependency is stable before its consumers are upgraded.

**Progress**: 1/9 tasks complete <progress value="11" max="100"></progress> 11%

## Tasks

- ✅ 01-prerequisites: Update SDK and central package versions ([Content](tasks/01-prerequisites/task.md), [Progress](tasks/01-prerequisites/progress-details.md))
- 🔄 02-blazorshared: Upgrade BlazorShared to net10.0 ([Content](tasks/02-blazorshared/task.md))
- 🔲 03-applicationcore: Upgrade ApplicationCore to net10.0
- 🔲 04-blazoradmin: Upgrade BlazorAdmin to net10.0
- 🔲 05-infrastructure: Upgrade Infrastructure to net10.0
- 🔲 06-publicapi: Upgrade PublicApi to net10.0
- 🔲 07-web: Upgrade Web to net10.0
- 🔲 08-test-projects: Upgrade all test projects to net10.0
- 🔲 09-final-validation: Full solution build and test suite validation
