finalStatus: success
successCriteriaStatus:
  passBuild: true
  generateNewUnitTests: false
  passUnitTests: true
summary: Upgraded all 10 eShopOnWeb projects to net10.0 by moving the shared SDK and target framework to .NET 10, refreshing central ASP.NET Core/EF Core/Azure/test package versions, removing obsolete package references, updating configuration binding and AutoMapper registration for .NET 10 compatibility, and fixing the PublicApi integration test host type.
