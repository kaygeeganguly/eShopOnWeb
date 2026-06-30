# Projects and dependencies analysis

This document provides a comprehensive overview of the projects and their dependencies in the context of upgrading to .NETCoreApp,Version=v10.0.

## Table of Contents

- [Executive Summary](#executive-Summary)
  - [Highlevel Metrics](#highlevel-metrics)
  - [Projects Compatibility](#projects-compatibility)
  - [Package Compatibility](#package-compatibility)
  - [API Compatibility](#api-compatibility)
  - [Binding Redirect Configuration](#binding-redirect-configuration)
- [Aggregate NuGet packages details](#aggregate-nuget-packages-details)
- [Top API Migration Challenges](#top-api-migration-challenges)
  - [Technologies and Features](#technologies-and-features)
  - [Most Frequent API Issues](#most-frequent-api-issues)
- [Projects Relationship Graph](#projects-relationship-graph)
- [Project Details](#project-details)

  - [src/ApplicationCore/ApplicationCore.csproj](#srcapplicationcoreapplicationcorecsproj)
  - [src/BlazorAdmin/BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj)
  - [src/BlazorShared/BlazorShared.csproj](#srcblazorsharedblazorsharedcsproj)
  - [src/Infrastructure/Infrastructure.csproj](#srcinfrastructureinfrastructurecsproj)
  - [src/PublicApi/PublicApi.csproj](#srcpublicapipublicapicsproj)
  - [src/Web/Web.csproj](#srcwebwebcsproj)
  - [tests/FunctionalTests/FunctionalTests.csproj](#testsfunctionaltestsfunctionaltestscsproj)
  - [tests/IntegrationTests/IntegrationTests.csproj](#testsintegrationtestsintegrationtestscsproj)
  - [tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj](#testspublicapiintegrationtestspublicapiintegrationtestscsproj)
  - [tests/UnitTests/UnitTests.csproj](#testsunittestsunittestscsproj)


## Executive Summary

### Highlevel Metrics

| Metric | Count | Status |
| :--- | :---: | :--- |
| Total Projects | 10 | All require upgrade |
| Total NuGet Packages | 48 | 24 need upgrade |
| Total Code Files | 302 |  |
| Total Code Files with Incidents | 32 |  |
| Total Lines of Code | 12159 |  |
| Total Number of Issues | 117 |  |
| Estimated LOC to modify | 61+ | at least 0.5% of codebase |

### Projects Compatibility

| Project | Target Framework | Difficulty | Package Issues | API Issues | Binding Issues | Est. LOC Impact | Description |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| [src/ApplicationCore/ApplicationCore.csproj](#srcapplicationcoreapplicationcorecsproj) | net8.0 | 🟢 Low | 3 | 2 | 0 | 2+ | ClassLibrary, Sdk Style = True |
| [src/BlazorAdmin/BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj) | net8.0 | 🟢 Low | 7 | 3 | 0 | 3+ | AspNetCore, Sdk Style = True |
| [src/BlazorShared/BlazorShared.csproj](#srcblazorsharedblazorsharedcsproj) | net8.0 | 🟢 Low | 0 | 0 | 0 |  | ClassLibrary, Sdk Style = True |
| [src/Infrastructure/Infrastructure.csproj](#srcinfrastructureinfrastructurecsproj) | net8.0 | 🟢 Low | 4 | 0 | 0 |  | ClassLibrary, Sdk Style = True |
| [src/PublicApi/PublicApi.csproj](#srcpublicapipublicapicsproj) | net8.0 | 🟢 Low | 11 | 5 | 0 | 5+ | AspNetCore, Sdk Style = True |
| [src/Web/Web.csproj](#srcwebwebcsproj) | net8.0 | 🟢 Low | 13 | 15 | 0 | 15+ | AspNetCore, Sdk Style = True |
| [tests/FunctionalTests/FunctionalTests.csproj](#testsfunctionaltestsfunctionaltestscsproj) | net8.0 | 🟢 Low | 3 | 29 | 0 | 29+ | ClassLibrary, Sdk Style = True |
| [tests/IntegrationTests/IntegrationTests.csproj](#testsintegrationtestsintegrationtestscsproj) | net8.0 | 🟢 Low | 2 | 0 | 0 |  | ClassLibrary, Sdk Style = True |
| [tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj](#testspublicapiintegrationtestspublicapiintegrationtestscsproj) | net8.0 | 🟢 Low | 1 | 7 | 0 | 7+ | ClassLibrary, Sdk Style = True |
| [tests/UnitTests/UnitTests.csproj](#testsunittestsunittestscsproj) | net8.0 | 🟢 Low | 2 | 0 | 0 |  | ClassLibrary, Sdk Style = True |

### Package Compatibility

| Status | Count | Percentage |
| :--- | :---: | :---: |
| ✅ Compatible | 24 | 50.0% |
| ⚠️ Incompatible | 5 | 10.4% |
| 🔄 Upgrade Recommended | 19 | 39.6% |
| ***Total NuGet Packages*** | ***48*** | ***100%*** |

### API Compatibility

| Category | Count | Impact |
| :--- | :---: | :--- |
| 🔴 Binary Incompatible | 9 | High - Require code changes |
| 🟡 Source Incompatible | 3 | Medium - Needs re-compilation and potential conflicting API error fixing |
| 🔵 Behavioral change | 49 | Low - Behavioral changes that may require testing at runtime |
| ✅ Compatible | 6892 |  |
| ***Total APIs Analyzed*** | ***6953*** |  |

## Aggregate NuGet packages details

| Package | Current Version | Suggested Version | Projects | Description |
| :--- | :---: | :---: | :--- | :--- |
| Ardalis.ApiEndpoints | 4.1.0 |  | [PublicApi.csproj](#srcpublicapipublicapicsproj) | ✅Compatible |
| Ardalis.GuardClauses | 4.0.1 |  | [ApplicationCore.csproj](#srcapplicationcoreapplicationcorecsproj) | ✅Compatible |
| Ardalis.ListStartupServices | 1.1.4 |  | [Web.csproj](#srcwebwebcsproj) | ✅Compatible |
| Ardalis.Result | 7.0.0 |  | [ApplicationCore.csproj](#srcapplicationcoreapplicationcorecsproj) | ✅Compatible |
| Ardalis.Specification | 7.0.0 |  | [ApplicationCore.csproj](#srcapplicationcoreapplicationcorecsproj)<br/>[Web.csproj](#srcwebwebcsproj) | ✅Compatible |
| Ardalis.Specification.EntityFrameworkCore | 7.0.0 |  | [Infrastructure.csproj](#srcinfrastructureinfrastructurecsproj) | ✅Compatible |
| AutoMapper.Extensions.Microsoft.DependencyInjection | 12.0.1 |  | [PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | ⚠️NuGet package is deprecated |
| Azure.Extensions.AspNetCore.Configuration.Secrets | 1.3.1 |  | [Web.csproj](#srcwebwebcsproj) | ✅Compatible |
| Azure.Identity | 1.10.4 | 1.21.0 | [Web.csproj](#srcwebwebcsproj) | NuGet package contains security vulnerability |
| Blazored.LocalStorage | 4.5.0 |  | [BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj) | ✅Compatible |
| BlazorInputFile | 0.2.0 |  | [BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj)<br/>[BlazorShared.csproj](#srcblazorsharedblazorsharedcsproj) | ✅Compatible |
| coverlet.collector | 6.0.2 |  | [PublicApiIntegrationTests.csproj](#testspublicapiintegrationtestspublicapiintegrationtestscsproj) | ✅Compatible |
| FluentValidation | 11.9.0 |  | [BlazorShared.csproj](#srcblazorsharedblazorsharedcsproj) | ✅Compatible |
| MediatR | 12.0.1 |  | [Web.csproj](#srcwebwebcsproj) | ✅Compatible |
| Microsoft.AspNetCore.Authentication.JwtBearer | 8.0.2 | 10.0.9 | [PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | NuGet package upgrade is recommended |
| Microsoft.AspNetCore.Components.Authorization | 8.0.2 | 10.0.9 | [BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj) | NuGet package upgrade is recommended |
| Microsoft.AspNetCore.Components.WebAssembly | 8.0.2 | 10.0.9 | [BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj) | NuGet package upgrade is recommended |
| Microsoft.AspNetCore.Components.WebAssembly.Authentication | 8.0.2 | 10.0.9 | [BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj) | NuGet package upgrade is recommended |
| Microsoft.AspNetCore.Components.WebAssembly.DevServer | 8.0.2 | 10.0.9 | [BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj) | NuGet package upgrade is recommended |
| Microsoft.AspNetCore.Components.WebAssembly.Server | 8.0.2 | 10.0.9 | [Web.csproj](#srcwebwebcsproj) | NuGet package upgrade is recommended |
| Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore | 8.0.2 | 10.0.9 | [PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | NuGet package upgrade is recommended |
| Microsoft.AspNetCore.Identity.EntityFrameworkCore | 8.0.2 | 10.0.9 | [Infrastructure.csproj](#srcinfrastructureinfrastructurecsproj)<br/>[PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | NuGet package upgrade is recommended |
| Microsoft.AspNetCore.Identity.UI | 8.0.2 | 10.0.9 | [PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | NuGet package upgrade is recommended |
| Microsoft.AspNetCore.Mvc.Testing | 8.0.2 | 10.0.9 | [FunctionalTests.csproj](#testsfunctionaltestsfunctionaltestscsproj)<br/>[PublicApiIntegrationTests.csproj](#testspublicapiintegrationtestspublicapiintegrationtestscsproj) | NuGet package upgrade is recommended |
| Microsoft.EntityFrameworkCore.InMemory | 8.0.2 | 10.0.9 | [FunctionalTests.csproj](#testsfunctionaltestsfunctionaltestscsproj)<br/>[Infrastructure.csproj](#srcinfrastructureinfrastructurecsproj)<br/>[IntegrationTests.csproj](#testsintegrationtestsintegrationtestscsproj)<br/>[PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | NuGet package upgrade is recommended |
| Microsoft.EntityFrameworkCore.SqlServer | 8.0.2 | 10.0.9 | [Infrastructure.csproj](#srcinfrastructureinfrastructurecsproj)<br/>[PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | NuGet package upgrade is recommended |
| Microsoft.EntityFrameworkCore.Tools | 8.0.2 | 10.0.9 | [PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | NuGet package upgrade is recommended |
| Microsoft.Extensions.Identity.Core | 8.0.2 | 10.0.9 | [BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj) | NuGet package upgrade is recommended |
| Microsoft.Extensions.Logging.Configuration | 8.0.0 | 10.0.9 | [BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj) | NuGet package upgrade is recommended |
| Microsoft.NET.Test.Sdk | 17.9.0 |  | [FunctionalTests.csproj](#testsfunctionaltestsfunctionaltestscsproj)<br/>[IntegrationTests.csproj](#testsintegrationtestsintegrationtestscsproj)<br/>[PublicApiIntegrationTests.csproj](#testspublicapiintegrationtestspublicapiintegrationtestscsproj)<br/>[UnitTests.csproj](#testsunittestsunittestscsproj) | ✅Compatible |
| Microsoft.VisualStudio.Azure.Containers.Tools.Targets | 1.19.6 |  | [PublicApi.csproj](#srcpublicapipublicapicsproj) | ⚠️NuGet package is incompatible |
| Microsoft.VisualStudio.Web.CodeGeneration.Design | 8.0.0 | 10.0.2 | [PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | NuGet package upgrade is recommended |
| Microsoft.Web.LibraryManager.Build | 2.1.175 |  | [Web.csproj](#srcwebwebcsproj) | ✅Compatible |
| MinimalApi.Endpoint | 1.3.0 |  | [PublicApi.csproj](#srcpublicapipublicapicsproj) | ✅Compatible |
| MSTest.TestAdapter | 3.2.2 |  | [PublicApiIntegrationTests.csproj](#testspublicapiintegrationtestspublicapiintegrationtestscsproj) | ✅Compatible |
| MSTest.TestFramework | 3.2.2 |  | [PublicApiIntegrationTests.csproj](#testspublicapiintegrationtestspublicapiintegrationtestscsproj) | ✅Compatible |
| NSubstitute | 5.1.0 |  | [IntegrationTests.csproj](#testsintegrationtestsintegrationtestscsproj)<br/>[UnitTests.csproj](#testsunittestsunittestscsproj) | ✅Compatible |
| NSubstitute.Analyzers.CSharp | 1.0.17 |  | [IntegrationTests.csproj](#testsintegrationtestsintegrationtestscsproj)<br/>[UnitTests.csproj](#testsunittestsunittestscsproj) | ✅Compatible |
| Swashbuckle.AspNetCore | 6.5.0 |  | [PublicApi.csproj](#srcpublicapipublicapicsproj) | ✅Compatible |
| Swashbuckle.AspNetCore.Annotations | 6.5.0 |  | [PublicApi.csproj](#srcpublicapipublicapicsproj) | ✅Compatible |
| Swashbuckle.AspNetCore.SwaggerUI | 6.5.0 |  | [PublicApi.csproj](#srcpublicapipublicapicsproj) | ✅Compatible |
| System.IdentityModel.Tokens.Jwt | 7.3.1 |  | [Infrastructure.csproj](#srcinfrastructureinfrastructurecsproj)<br/>[PublicApi.csproj](#srcpublicapipublicapicsproj)<br/>[Web.csproj](#srcwebwebcsproj) | ⚠️NuGet package is deprecated |
| System.Net.Http.Json | 8.0.0 | 10.0.9 | [BlazorAdmin.csproj](#srcblazoradminblazoradmincsproj) | NuGet package upgrade is recommended |
| System.Security.Claims | 4.3.0 |  | [ApplicationCore.csproj](#srcapplicationcoreapplicationcorecsproj) | NuGet package functionality is included with framework reference |
| System.Text.Json | 8.0.3 | 10.0.9 | [ApplicationCore.csproj](#srcapplicationcoreapplicationcorecsproj) | NuGet package upgrade is recommended |
| xunit | 2.7.0 |  | [FunctionalTests.csproj](#testsfunctionaltestsfunctionaltestscsproj)<br/>[IntegrationTests.csproj](#testsintegrationtestsintegrationtestscsproj)<br/>[UnitTests.csproj](#testsunittestsunittestscsproj) | ⚠️NuGet package is deprecated |
| xunit.runner.console | 2.7.0 |  | [UnitTests.csproj](#testsunittestsunittestscsproj) | ⚠️NuGet package is deprecated |
| xunit.runner.visualstudio | 2.5.6 |  | [FunctionalTests.csproj](#testsfunctionaltestsfunctionaltestscsproj)<br/>[IntegrationTests.csproj](#testsintegrationtestsintegrationtestscsproj)<br/>[UnitTests.csproj](#testsunittestsunittestscsproj) | ✅Compatible |

## Top API Migration Challenges

### Technologies and Features

| Technology | Issues | Percentage | Migration Path |
| :--- | :---: | :---: | :--- |

### Most Frequent API Issues

| API | Count | Percentage | Category |
| :--- | :---: | :---: | :--- |
| T:System.Net.Http.HttpContent | 32 | 52.5% | Behavioral Change |
| T:System.Uri | 11 | 18.0% | Behavioral Change |
| M:Microsoft.Extensions.Configuration.ConfigurationBinder.Get''1(Microsoft.Extensions.Configuration.IConfiguration) | 4 | 6.6% | Binary Incompatible |
| M:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure''1(Microsoft.Extensions.DependencyInjection.IServiceCollection,Microsoft.Extensions.Configuration.IConfiguration) | 4 | 6.6% | Binary Incompatible |
| M:System.Exception.#ctor(System.Runtime.Serialization.SerializationInfo,System.Runtime.Serialization.StreamingContext) | 2 | 3.3% | Source Incompatible |
| M:Microsoft.Extensions.Logging.ConsoleLoggerExtensions.AddConsole(Microsoft.Extensions.Logging.ILoggingBuilder) | 2 | 3.3% | Behavioral Change |
| M:System.Uri.#ctor(System.String) | 2 | 3.3% | Behavioral Change |
| M:System.Uri.#ctor(System.String,System.UriKind) | 1 | 1.6% | Behavioral Change |
| M:System.TimeSpan.FromMinutes(System.Double) | 1 | 1.6% | Source Incompatible |
| M:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler(Microsoft.AspNetCore.Builder.IApplicationBuilder,System.String) | 1 | 1.6% | Behavioral Change |
| M:Microsoft.Extensions.Configuration.ConfigurationBinder.GetValue(Microsoft.Extensions.Configuration.IConfiguration,System.Type,System.String) | 1 | 1.6% | Binary Incompatible |

## Projects Relationship Graph

Legend:
📦 SDK-style project
⚙️ Classic project

```mermaid
flowchart LR
    P1 --> P3
    P2 --> P3
    P4 --> P1
    P5 --> P1
    P5 --> P4
    P6 --> P1
    P6 --> P2
    P6 --> P3
    P6 --> P4
    P7 --> P1
    P7 --> P5
    P7 --> P6
    P8 --> P4
    P8 --> P10
    P9 --> P5
    P9 --> P6
    P10 --> P1
    P10 --> P6

```

## Project Details

