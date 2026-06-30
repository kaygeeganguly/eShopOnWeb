# 03-applicationcore: Upgrade ApplicationCore to net10.0

## Objective
Upgrade ApplicationCore (Level 1 library) to net10.0.

## Research Findings
- TFM: net10.0 via Directory.Packages.props (no change needed in csproj)
- System.Security.Claims: already removed in task 01
- API issue: Exception(SerializationInfo, StreamingContext) constructor removed in .NET 9+ — must remove from EmptyBasketOnCheckoutException
- System.Text.Json: package reference exists but redundant (framework-included in .NET 10) — NU1510 warning

## Files to Modify
- src/ApplicationCore/Exceptions/EmptyBasketOnCheckoutException.cs — Remove obsolete serialization constructor
- src/ApplicationCore/ApplicationCore.csproj — Remove System.Text.Json PackageReference

## Done When
- ApplicationCore targets net10.0 ✅
- System.Security.Claims removed ✅
- Exception serialization constructor removed ✅
- System.Text.Json PackageReference removed ✅
- Builds warning-free ✅
