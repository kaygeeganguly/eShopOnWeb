finalStatus: success
successCriteriaStatus:
  passBuild: true
  passUnitTests: true
summary: Upgraded the eShopOnWeb solution from .NET 8 to .NET 10 by moving the central target framework and SDK to net10.0/10.0.301, updating ASP.NET Core, EF Core, Microsoft.Extensions, test, security, and tooling package versions, fixing PublicApi OpenAPI and AutoMapper compatibility changes, updating Dockerfiles/CI/debug configs, and validating with a successful solution build plus passing test suites (44 + 3 + 12 + 15 tests).
