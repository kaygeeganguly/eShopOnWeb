finalStatus: success
successCriteriaStatus:
  passBuild: true
  generateNewUnitTests: true
  passUnitTests: true
summary: Upgraded eShopOnWeb from net8.0 to net10.0 by updating the central target framework and package versions, the SDK/tooling/runtime pins used by local development, Docker, CI, and Azure App Service infrastructure, removing obsolete framework package references, and updating PublicApi integration tests to stay scoped to the API host under the new top-level Program shape. The solution test suite passes on net10.0.
