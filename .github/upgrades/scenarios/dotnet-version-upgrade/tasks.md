# .NET Version Upgrade Progress

## Overview

Upgrading eShopOnWeb from net8.0 to net10.0 across all 10 projects using the Top-Down strategy. Package versions centralized in Directory.Packages.props (CPM). Breaking API changes in Web and PublicApi will be fixed inline.

**Progress**: 2/6 tasks complete <progress value="33" max="100"></progress> 33%

## Tasks

- ✅ 01-prerequisites: Verify SDK and update global configuration ([Content](tasks/01-prerequisites/task.md), [Progress](tasks/01-prerequisites/progress-details.md))
- ✅ 02-shared-libraries: Upgrade BlazorShared, ApplicationCore, Infrastructure, and BlazorAdmin ([Content](tasks/02-shared-libraries/task.md), [Progress](tasks/02-shared-libraries/progress-details.md))
- 🔲 03-web-app: Upgrade the ASP.NET Core Web application
- 🔲 04-public-api: Upgrade the PublicApi application
- 🔲 05-test-projects: Upgrade all test projects
- 🔲 06-final-validation: Build solution and run all tests
