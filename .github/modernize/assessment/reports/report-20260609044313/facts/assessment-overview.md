# Assessment Overview

This document serves as the navigation entry point for all supplementary analysis documents generated as part of the eShopOnWeb application assessment. Each document provides a focused perspective on a specific aspect of the application.

## Supplementary Documents

| Document | Description |
|---|---|
| [Architecture Diagram](./architecture-diagram.md) | Two-layer architecture visualization: high-level application architecture (ASP.NET Core 8 MVC, Blazor WebAssembly, Public REST API, SQL Server) and detailed component relationship diagram showing how controllers, services, repositories, and middleware interact. |
| [Dependency Map](./dependency-map.md) | Visual map of all 38 production NuGet packages grouped by functional category (Web Frameworks, Database/ORM, Security, API Documentation, Cloud Configuration, Utilities) with version compatibility risk analysis and test dependency inventory. |
| [API & Service Communication Contracts](./api-service-contracts.md) | Complete catalog of 9 REST API endpoints across the Web MVC and PublicApi services, including HTTP methods, request/response types, authentication requirements, DTOs, CORS configuration, and a sequence diagram of the primary authentication and catalog management flows. |
| [Data Architecture & Persistence Layer](./data-architecture.md) | EF Core 8 entity model with ER diagram across 11 entities (Basket, Order, Catalog, Identity aggregates), repository patterns, EF Core configuration, migration history, caching strategy, and data classification/sensitivity analysis (PII, PCI-adjacent fields). |
| [Configuration & Externalized Settings Inventory](./configuration-inventory.md) | Comprehensive inventory of all configuration sources (appsettings.json variants, Docker overrides, Azure Key Vault, User Secrets), runtime profiles (Development/Docker/Production/Test), properties inventory per service, startup dependency chain, and secrets provisioning workflow. |
| [Core Business Workflows](./business-workflows.md) | End-to-end documentation of the four primary business workflows: anonymous/authenticated basket management, checkout and order placement, catalog administration via Blazor Admin, and user authentication/JWT issuance. Includes domain entity descriptions, business rules, validation logic, and a sequence diagram of the checkout flow. |
