# FinPath Explorer - Feature Documentation Guide

This directory contains detailed specifications for each major feature of the FinPath Explorer financial planning platform. Each feature document follows a standardized format to ensure consistency and enable effective LLM-assisted development.

## Feature Documentation Structure

Each feature file includes the following sections:

### 1. Feature Overview
- **Purpose**: What the feature accomplishes
- **User Stories**: Key user scenarios across AU/US/UK
- **Priority**: Implementation priority (P0=Critical, P1=High, P2=Medium, P3=Low)
- **Dependencies**: Other features or infrastructure requirements

### 2. Technical Specifications
- **Data Models**: Required Prisma schema additions/modifications
- **API Operations**: Wasp queries and actions needed
- **Tax Engine Integration**: Country-specific tax calculations
- **Integration Points**: How it connects with existing SaaS features

### 3. Multi-Country Requirements
- **Australia**: ATO compliance and local considerations
- **USA**: IRS/state tax integration and 401k/IRA features
- **UK**: HMRC compliance and ISA/pension features
- **Cross-Border**: Expat and dual-resident scenarios

### 4. User Interface Requirements
- **Adaptive Complexity**: ELI5, Intermediate, and Expert interfaces
- **Components**: UI elements to be created
- **User Flows**: Step-by-step user interactions
- **Design Considerations**: UX patterns and accessibility requirements

### 5. Implementation Details
- **File Structure**: Where code should be organized
- **Tax Calculations**: Specific algorithms and compliance requirements
- **Testing Requirements**: Financial accuracy and multi-country validation

### 6. Success Criteria
- **Acceptance Criteria**: Definition of done
- **Performance Requirements**: Financial calculation speed and accuracy
- **Compliance Requirements**: Tax authority compliance verification

## Feature List

### Core Platform Features (P0 - Critical)
1. **[Multi-Country Setup](./multi-country-setup.md)** - Country selection and localization
2. **[Scenario Management](./scenario-management.md)** - Create, save, and manage financial scenarios
3. **[Strategy Templates](./strategy-templates.md)** - Pre-built financial strategies per country

### Financial Components (P0 - Critical)
4. **[Housing Strategies](./housing-strategies.md)** - Buy/rent/invest property components
5. **[Investment Portfolio](./investment-portfolio.md)** - ETFs, stocks, retirement accounts
6. **[Tax Optimization](./tax-optimization.md)** - Country-specific tax calculations and optimization

### User Experience Features (P1 - High Priority)
7. **[Adaptive Interface](./adaptive-interface.md)** - ELI5 to Expert complexity progression
8. **[Scenario Comparison](./scenario-comparison.md)** - Side-by-side analysis and optimization
9. **[Drag-Drop Builder](./drag-drop-builder.md)** - Visual strategy construction interface

### Advanced Features (P2 - Medium Priority)
10. **[Cross-Border Planning](./cross-border-planning.md)** - Expat and dual-resident scenarios
11. **[Advanced Analytics](./advanced-analytics.md)** - Monte Carlo simulations and projections
12. **[Financial Reports](./financial-reports.md)** - Export and professional reporting

### Integration Features (P3 - Future Enhancements)
13. **[Data Integration](./data-integration.md)** - Real-time market and property data
14. **[Advisor Marketplace](./advisor-marketplace.md)** - Financial advisor connections
15. **[Portfolio Tracking](./portfolio-tracking.md)** - Live portfolio integration and monitoring

## Country-Specific Considerations

### Australia Features
- **Negative Gearing**: Investment property tax optimization
- **Franking Credits**: Dividend imputation calculations
- **Superannuation**: Concessional/non-concessional contributions
- **CGT Discount**: 50% capital gains discount after 12 months
- **FHOG**: First home owner grants and stamp duty

### USA Features
- **401k/IRA Optimization**: Traditional vs Roth strategies
- **Tax-Loss Harvesting**: Wash sale rule compliance
- **State Tax Optimization**: Multi-state tax planning
- **Depreciation**: Real estate depreciation schedules
- **HSA Integration**: Health savings account optimization

### UK Features
- **ISA Optimization**: S&S ISA vs Cash ISA strategies
- **Pension Planning**: Workplace pensions, SIPP, tax relief
- **Capital Gains Management**: Annual exemption optimization
- **Buy-to-Let**: Mortgage interest relief and rental income
- **Help to Buy**: Government schemes and stamp duty

## Development Guidelines

### Multi-Country Architecture
- Build country-agnostic interfaces with localized implementations
- Use dependency injection for tax engines
- Implement currency conversion and multi-currency support
- Design for easy addition of new countries

### Financial Calculation Standards
- All tax calculations must be ATO/IRS/HMRC compliant
- Implement comprehensive testing for financial accuracy
- Use precise decimal arithmetic for currency calculations
- Cache expensive calculations with proper invalidation

### User Experience Principles
- Start users with appropriate complexity level
- Progressive disclosure of advanced features
- Country-specific onboarding and education
- Clear disclaimers about financial advice vs education

### For Claude Code Agents
- Read the relevant feature document before implementing
- Follow multi-country patterns and tax engine integration
- Ensure comprehensive testing for financial calculations
- Update documentation if implementation differs from specification
- Validate tax calculations with professional review

---

*Each feature document is designed to be comprehensive yet actionable for LLM-assisted development of FinPath Explorer's sophisticated financial planning capabilities.*