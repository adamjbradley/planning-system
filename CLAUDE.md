# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack SaaS application built with the Wasp framework (v0.17.0), based on the Open SaaS template. The project consists of three main components:

1. **`planning-system/app/`** - Main web application (Wasp + React + Node.js)
2. **`planning-system/e2e-tests/`** - Playwright end-to-end tests
3. **`planning-system/blog/`** - Documentation/blog site (Astro + Starlight)

## Development Commands

### Main Application (from `planning-system/app/`)
```bash
# Start development environment
wasp start db          # Start PostgreSQL database (keep running)
wasp start              # Start the application (keep running)
wasp db migrate-dev     # Run database migrations (after schema changes)

# Database operations
wasp db seed            # Seed database with mock data
wasp db reset           # Reset database
```

### Blog/Documentation (from `planning-system/blog/`)
```bash
npm run dev             # Start development server
npm run build           # Build for production
```

### End-to-End Tests (from `planning-system/e2e-tests/`)
```bash
npm install                     # Install test dependencies
npm run local:e2e:start        # Run tests with Playwright UI (requires Stripe setup)
```

## Architecture

### Wasp Framework Structure
- **`main.wasp`** - Central configuration file defining app structure, routes, pages, auth, and operations
- **`schema.prisma`** - Database models and relationships (PostgreSQL)  
- **`src/`** - Application source code organized by features

### Key Features
- **Authentication**: Email/password with verification, optional social auth (Google, GitHub, Discord)
- **Payments**: Stripe and Lemon Squeezy integration
- **Admin Dashboard**: Analytics, user management, settings
- **File Upload**: S3 integration for file handling
- **Financial Planning**: Multi-country scenario modeling, tax optimization, investment strategies
- **Analytics**: Daily stats tracking with multiple providers

### Detailed Development Guidance
For comprehensive development guidance, see these detailed documentation files:

- **[Wasp Framework Guide](./docs/development/wasp-framework-guide.md)** - Complete Wasp development patterns, import conventions, operations, authentication, and troubleshooting
- **[Financial Planning Standards](./docs/development/financial-planning-standards.md)** - Financial calculation requirements, multi-country architecture, tax compliance, and testing standards

## FinPath Explorer - Financial Planning Platform

### For Financial Planning Development
Before implementing any financial planning features, consult these comprehensive documentation resources:

#### Product Requirements
- **[PRD](./docs/PRD.md)** - Complete FinPath Explorer product requirements and multi-country specifications
- **[Feature Specifications](./docs/features/)** - Detailed specifications for financial planning features
- **[Implementation Roadmap](./docs/IMPLEMENTATION_ROADMAP.md)** - 4-phase development plan with vertical slice approach

#### Core Financial Features (P0 - Critical)
- **[Multi-Country Setup](./docs/features/multi-country-setup.md)** - Country selection, tax engine initialization, adaptive complexity
- **[Scenario Management](./docs/features/scenario-management.md)** - Create, save, and manage financial scenarios
- **[Strategy Templates](./docs/features/strategy-templates.md)** - Pre-built financial strategies per country
- **[Housing Strategies](./docs/features/housing-strategies.md)** - Buy/rent/invest property components with tax optimization

#### Development Priority Order (4-Phase Approach)
1. **Phase 1 (Weeks 1-4)**: MVP Foundation - ELI5 mode, basic scenarios, simple templates
2. **Phase 2 (Weeks 5-8)**: Intermediate Complexity - Tax engines, country-specific features
3. **Phase 3 (Weeks 9-12)**: Expert Mode - Advanced analytics, cross-border planning
4. **Phase 4 (Weeks 13-16)**: Production Polish - Performance optimization, real data integration

#### Financial Planning Specific Requirements
- **Tax Compliance**: All calculations must be ATO/IRS/HMRC compliant with professional validation
- **Multi-Country Support**: Australia, USA, UK with country-specific tax optimizations
- **Progressive Complexity**: ELI5 → Intermediate → Expert user progression
- **Calculation Accuracy**: <0.01% variance for all financial projections

### Quick Start for Financial Planning Agents
1. Read the [FinPath Explorer PRD](./docs/PRD.md) for project context
2. Review the specific [financial feature specification](./docs/features/) you're implementing
3. Follow the [Implementation Roadmap](./docs/IMPLEMENTATION_ROADMAP.md) phase-by-phase approach
4. Use [Example Prompts](./docs/development/example-prompts.md) for common development tasks
5. Ensure all tax calculations meet country-specific compliance requirements
6. Test financial accuracy with real-world scenarios before deployment

## Environment Setup
- Requires `.env.client` and `.env.server` files
- PostgreSQL database (managed by Wasp)
- Optional: Stripe CLI for payment testing
- Remember we're using Wasp, a full-stack framework with batteries included, that can do some of the heavy lifting for us, and we want to use a modified vertical slice implementation approach for LLM-assisted coding so we can start with basic implementations of features first, and add on complexity from there.