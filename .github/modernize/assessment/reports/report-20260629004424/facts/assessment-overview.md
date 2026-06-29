# Assessment Overview

This document serves as the navigation entry point for the supplementary analysis documents generated for the eShopOnWeb application assessment. Each document focuses on a specific dimension of the application's architecture, dependencies, and business logic.

## Supplementary Documents

| Document | Description |
|----------|-------------|
| [Architecture Diagram](architecture-diagram.md) | Two-layer architecture visualization: high-level application architecture (technology stack, data flow, external integrations) and detailed component relationship diagram (controllers, services, repositories, middleware). |
| [Dependency Map](dependency-map.md) | Visual map of all external NuGet package dependencies grouped by functional category (Web Frameworks, Database/ORM, Security, Cloud/Azure, Utilities). Includes version details, compatibility risks, and test dependency inventory. |
| [API & Service Communication Contracts](api-service-contracts.md) | Full inventory of REST API endpoints across the Web and PublicApi services, DTO definitions, communication patterns, authentication/authorization posture, health check endpoints, and a service communication sequence diagram. |
| [Data Architecture & Persistence Layer](data-architecture.md) | Entity model with ER diagram, database configuration per environment, EF Core DbContext ownership, key repository methods and specifications, caching strategy, and data classification (PII/PCI sensitivity). |
| [Configuration & Externalized Settings Inventory](configuration-inventory.md) | Comprehensive inventory of all configuration sources (appsettings files, Docker Compose, Azure Key Vault, environment variables), build and runtime profiles, properties per service, secrets provisioning workflow, and framework/runtime versions. |
| [Core Business Workflows](business-workflows.md) | End-to-end documentation of the application's business processes: catalog browsing, basket management, anonymous-to-authenticated basket transfer, order checkout, and admin catalog management. Includes domain entity descriptions, business rules, and a checkout sequence diagram. |
