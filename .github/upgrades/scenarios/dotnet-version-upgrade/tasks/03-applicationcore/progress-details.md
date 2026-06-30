## Files Modified
- src/ApplicationCore/Exceptions/EmptyBasketOnCheckoutException.cs — Removed obsolete Exception(SerializationInfo, StreamingContext) constructor
- src/ApplicationCore/ApplicationCore.csproj — Removed System.Text.Json PackageReference (framework-included in .NET 10)

## Build Result
- Errors: 0
- Warnings: 0
- Projects built: src/ApplicationCore/ApplicationCore.csproj → net10.0 ✅

## Test Result
- Tests run in task 08

## Changes Summary
- Removed obsolete serialization constructor from EmptyBasketOnCheckoutException (Api.0002 fix)
- Removed System.Text.Json PackageReference (NU1510 warning fix — included in framework)
- System.Security.Claims was already removed in task 01

## Issues Encountered
- NU1510 warning for System.Text.Json — fixed by removing the explicit package reference
