# Assessment Overview

This directory contains supplementary analysis documents generated as part of the application assessment for **eShopOnWeb** — a reference ASP.NET Core 8 e-commerce application implementing Clean Architecture with MVC, Blazor WebAssembly, and a REST API.

## Supplementary Documents

| Document | Description |
|----------|-------------|
| [Architecture Diagram](./architecture-diagram.md) | Two-layer architecture visualization: high-level application architecture with technology stack summary, and detailed component relationship diagram showing interactions between controllers, services, repositories, and infrastructure. |
| [Dependency Map](./dependency-map.md) | Visual map of all external NuGet dependencies grouped by functional category (Web Frameworks, Database/ORM, Security, Application Frameworks, API Documentation, Utilities). Includes version/compatibility risk analysis and test dependency inventory. |
| [API & Service Contracts](./api-service-contracts.md) | Full inventory of REST API endpoints across the Web MVC and PublicApi services, including HTTP methods, paths, request/response types, authentication requirements, management endpoints, DTOs, communication patterns, and a sequence diagram showing the primary API flows. |
| [Data Architecture](./data-architecture.md) | Entity model with ER diagram, database configuration per environment (Development/Docker/Production), EF Core entity configurations, repository interfaces, caching strategy (IMemoryCache + browser localStorage), and data ownership/sensitivity classification. |
| [Configuration Inventory](./configuration-inventory.md) | Comprehensive inventory of all configuration sources (appsettings files, Azure Key Vault, User Secrets, Docker Compose environment variables), build and runtime profiles, properties per environment, startup dependency chain, secrets provisioning workflow, feature flags, and framework/runtime version matrix. |
| [Business Workflows](./business-workflows.md) | Core business domain documentation covering domain entities, service-to-domain mapping, primary workflows (browse catalog, add to basket, checkout/order creation, admin catalog management, order history), business rules, validation constraints, state transitions, authorization rules, and a Mermaid sequence diagram of the end-to-end checkout flow. |
