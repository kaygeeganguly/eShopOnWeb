# Assessment Overview

This document is the navigation entry point for the supplementary analysis documents generated for the eShopOnWeb assessment (report ID: `20260629091838`). Each document covers a specific aspect of the application's architecture, dependencies, and business workflows.

## Supplementary Documents

| Document | Description |
|----------|-------------|
| [Architecture Diagram](architecture-diagram.md) | Two-layer visualization: high-level application architecture (technology stack, data storage, external services) and detailed component relationships grouped by architectural layer |
| [Dependency Map](dependency-map.md) | Visual map of all external NuGet package dependencies grouped by functional category (web frameworks, ORM, security, API documentation, utilities), with version and compatibility risk analysis |
| [API & Service Communication Contracts](api-service-contracts.md) | Complete inventory of all REST API endpoints across the PublicApi and Web services, DTO/contract definitions, communication patterns, security posture, and a service communication sequence diagram |
| [Data Architecture & Persistence Layer](data-architecture.md) | Database configuration per environment, EF Core entity model with ER diagram, repository methods, caching strategy, data ownership boundaries, and PII/sensitive data classification |
| [Configuration & Externalized Settings Inventory](configuration-inventory.md) | Comprehensive inventory of all configuration sources, build/runtime profiles, properties per service, secrets provisioning workflow (Azure Key Vault), startup dependency chain, and framework versions |
| [Core Business Workflows](business-workflows.md) | End-to-end documentation of primary business processes (catalog browsing, basket management, checkout, order history, admin catalog management), business rules, validation logic, and a business workflow sequence diagram |
