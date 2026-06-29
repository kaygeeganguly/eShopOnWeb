finalStatus: succeeded
successCriteriaStatus:
  passBuild: passed
  generateNewUnitTests: not_requested
  passUnitTests: passed
summary: |
  Upgraded the eShopOnWeb solution from .NET 8 to .NET 10 by moving the central TargetFramework to net10.0, updating global.json to 10.0.x, and refreshing central package versions for ASP.NET Core, EF Core, Azure, test, and third-party dependencies.
  Removed obsolete or unnecessary package references (BlazorInputFile, old System.* package references, Web code-generation tooling in app projects), updated PublicApi for AutoMapper and OpenAPI/Swashbuckle API changes, and simplified PublicApi integration tests to remove the unnecessary Web project dependency.
  Validation completed successfully with `dotnet build eShopOnWeb.sln` and `dotnet test eShopOnWeb.sln` passing for all existing test projects (44 + 3 + 12 + 15 tests).
