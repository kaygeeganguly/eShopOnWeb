# 01-prerequisites: 01-prerequisites

Execute task 01-prerequisites.

## Research findings
- `dotnet --version` reports `10.0.301`.
- `dotnet --list-sdks` includes `10.0.109`, `10.0.204`, and `10.0.301`, so a .NET 10 SDK is available locally.
- `global.json` was pinned to the .NET 8 SDK feature band (`8.0.x`).
- `Directory.Packages.props` was targeting `net8.0` with central ASP.NET Core, EF Core, and System extension version properties still on 8.0.x values.
- `ApplicationCore.csproj` still references `System.Security.Claims`, and `PublicApi.csproj` still references `Microsoft.VisualStudio.Azure.Containers.Tools.Targets`; a restore attempt confirmed that deleting those CPM entries now causes `NU1010`, so their actual removal must happen in a later task that updates the project files.
