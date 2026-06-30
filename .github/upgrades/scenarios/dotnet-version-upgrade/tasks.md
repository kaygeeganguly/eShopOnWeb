# .NET Version Upgrade Progress

## Overview

Upgrading eShopOnWeb from net8.0 to net10.0 across all 10 projects using a Bottom-Up (Dependency-First) strategy. Projects are upgraded tier by tier from leaf libraries through to web applications and test projects, ensuring each dependency is stable before its consumers are upgraded.

**Progress**: 9/9 tasks complete <progress value="100" max="100"></progress> 100%

## Tasks

- ✅ 01-prerequisites: Update SDK and central package versions ([Content](tasks/01-prerequisites/task.md), [Progress](tasks/01-prerequisites/progress-details.md))
- ✅ 02-blazorshared: Upgrade BlazorShared to net10.0 ([Content](tasks/02-blazorshared/task.md), [Progress](tasks/02-blazorshared/progress-details.md))
- ✅ 03-applicationcore: Upgrade ApplicationCore to net10.0 ([Content](tasks/03-applicationcore/task.md), [Progress](tasks/03-applicationcore/progress-details.md))
- ✅ 04-blazoradmin: Upgrade BlazorAdmin to net10.0 ([Content](tasks/04-blazoradmin/task.md), [Progress](tasks/04-blazoradmin/progress-details.md))
- ✅ 05-infrastructure: Upgrade Infrastructure to net10.0 ([Content](tasks/05-infrastructure/task.md), [Progress](tasks/05-infrastructure/progress-details.md))
- ✅ 06-publicapi: Upgrade PublicApi to net10.0 ([Content](tasks/06-publicapi/task.md), [Progress](tasks/06-publicapi/progress-details.md))
- ✅ 07-web: Upgrade Web to net10.0 ([Content](tasks/07-web/task.md), [Progress](tasks/07-web/progress-details.md))
- ✅ 08-test-projects: Upgrade all test projects to net10.0 ([Content](tasks/08-test-projects/task.md), [Progress](tasks/08-test-projects/progress-details.md))
- ✅ 09-final-validation: Full solution build and test suite validation ([Content](tasks/09-final-validation/task.md), [Progress](tasks/09-final-validation/progress-details.md))
