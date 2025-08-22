# Example Prompts for FinPath Explorer Development

This document provides example prompts for LLM-assisted development of the FinPath Explorer financial planning platform.

## Financial Planning Implementation Prompts

### Start Phase 1 Development
```
I want to implement Phase 1 of the FinPath Explorer financial planning platform. Based on the implementation roadmap, let's start with the multi-country setup feature. I need you to:

1. Review the multi-country setup feature specification
2. Extend the Prisma schema with the required User fields (country, complexityLevel, currency)
3. Create the basic country selection component
4. Implement the user profile setup flow

Remember to follow the financial planning development rules, use proper tax compliance patterns, and ensure all calculations use Decimal arithmetic for currency handling.
```

### Implement Tax Engine
```
Let's implement the Australian tax engine following the multi-country compliance requirements. I need you to:

1. Create the ITaxEngine interface and TaxEngineFactory
2. Implement AustralianTaxEngine with ATO-compliant tax brackets
3. Include negative gearing, CGT discount, and superannuation calculations
4. Add comprehensive unit tests with real ATO test cases
5. Ensure all calculations meet the <0.01% accuracy requirement

Use the calculation accuracy standards and validate against official ATO calculators.
```

### Add Investment Component
```
Based on the housing strategies feature spec, let's implement the investment property component:

1. Create the HousingComponent model with all required fields
2. Implement the HousingCalculator with country-specific strategies
3. Add negative gearing calculations for Australian properties
4. Build the property investment form with progressive complexity
5. Include tax optimization suggestions based on user's country

Follow the progressive complexity patterns (ELI5 → Intermediate → Expert) and ensure full ATO compliance for Australian calculations.
```

### Create Strategy Template
```
Let's implement the "Superannuation Boost" strategy template for Australian users:

1. Create the StrategyTemplate model with educational content
2. Implement the template with salary sacrifice optimization
3. Add template customization based on user income and age
4. Include educational content explaining tax benefits
5. Add validation against superannuation contribution caps

Ensure the template follows ATO compliance requirements and includes proper risk disclosures.
```

## Development Workflow Prompts

### Review Financial Accuracy
```
Please review the tax calculation implementation for accuracy and compliance:

1. Verify all calculations use Decimal arithmetic
2. Check against official tax brackets and rates
3. Ensure professional validation requirements are met
4. Review test coverage for edge cases
5. Validate performance meets <2 second requirement

Reference the calculation accuracy standards and multi-country compliance rules.
```

### Optimize for Country Support
```
I need to extend the platform to support UK tax calculations:

1. Implement UKTaxEngine with HMRC compliance
2. Add UK-specific ISA and pension calculations
3. Create UK strategy templates (ISA Maximization, Pension Planning)
4. Update the UI to handle UK-specific features
5. Add comprehensive testing for HMRC compliance

Follow the multi-country architecture patterns and ensure proper currency handling for GBP.
```

## Testing and Validation Prompts

### Create Comprehensive Tests
```
Let's create a comprehensive test suite for the financial calculations:

1. Unit tests for all tax engines with official test cases
2. Integration tests for complete scenarios
3. Performance tests ensuring sub-2-second calculations
4. Property-based tests for edge cases
5. External validation against official calculators

Follow the calculation accuracy testing standards and include professional validation.
```

### Validate Tax Compliance
```
Please validate our tax calculations against official requirements:

1. Compare Australian calculations with ATO examples
2. Verify US calculations against IRS publications
3. Check UK calculations against HMRC guidance
4. Ensure all contribution limits are current
5. Document sources and validation methodology

Include compliance error handling and regulatory change tracking.
```

## Wasp Framework Development Prompts

### Wasp Operation Creation
```
I need to create Wasp operations for scenario management. Please:

1. Define the operations in main.wasp (getScenarios, createScenario, updateScenario, deleteScenario)
2. Implement the operations in src/scenario/operations.ts
3. Add proper TypeScript types and error handling
4. Include entity access for Scenario, User, and related components
5. Add client-side React hooks for the operations

Follow the Wasp import conventions and use proper error handling patterns.
```

### Database Schema Extension
```
I need to extend the Prisma schema for financial planning. Please:

1. Add Country, Currency, and ComplexityLevel enums
2. Extend the User model with financial planning fields
3. Create Scenario, HousingComponent, and InvestmentComponent models
4. Add proper relationships and constraints
5. Generate and apply the database migration

Ensure the schema supports multi-country scenarios and complex financial modeling.
```

### Authentication Integration
```
I need to integrate financial planning with Wasp auth. Please:

1. Update the User model with country and complexity preferences
2. Create protected routes for financial planning pages
3. Add userSignupFields for country selection during registration
4. Implement user profile completion flow
5. Add auth guards for financial planning features

Follow Wasp auth patterns and ensure proper user experience flow.
```

## UI/UX Development Prompts

### Progressive Complexity Interface
```
I need to build a progressive complexity interface for financial planning:

1. Create complexity level selection component (ELI5, Intermediate, Expert)
2. Build adaptive forms that show/hide fields based on complexity
3. Add educational tooltips and explanations for each level
4. Implement smooth transitions between complexity levels
5. Create user onboarding flow with complexity progression

Use ShadCN UI components and follow progressive disclosure patterns.
```

### Country-Specific UI Elements
```
I need to create country-specific UI components:

1. Country flag selector with currency display
2. Country-specific tax rate displays
3. Localized financial term explanations
4. Country-appropriate examples and scenarios
5. Multi-currency input components

Ensure components are reusable across different countries and maintain consistency.
```

### Scenario Comparison Interface
```
I need to build a scenario comparison interface:

1. Side-by-side scenario display cards
2. Interactive charts showing projections over time
3. Highlighting key differences between scenarios
4. Export functionality for comparison reports
5. Drag-and-drop scenario organization

Focus on clear data visualization and user-friendly interactions.
```

## Performance and Optimization Prompts

### Calculation Performance Optimization
```
I need to optimize financial calculation performance:

1. Implement calculation result caching with proper invalidation
2. Add web workers for complex Monte Carlo simulations
3. Optimize database queries for scenario retrieval
4. Implement progressive loading for large datasets
5. Add performance monitoring and benchmarking

Ensure all optimizations maintain calculation accuracy and don't introduce errors.
```

### Real-Time Updates
```
I need to implement real-time parameter updates:

1. Add debounced input handling for financial parameters
2. Implement optimistic UI updates for quick feedback
3. Cache intermediate calculation results
4. Add loading states for complex calculations
5. Ensure sub-100ms response times for simple updates

Balance responsiveness with calculation accuracy and resource usage.
```

## Integration and Deployment Prompts

### External Data Integration
```
I need to integrate real market data:

1. Create property market data API integration
2. Add real-time currency conversion services
3. Implement tax rate update mechanisms
4. Create data validation and fallback systems
5. Add data freshness indicators

Ensure data quality and provide graceful fallbacks for API failures.
```

### Production Deployment
```
I need to deploy FinPath Explorer to production:

1. Configure Fly.io deployment with proper environment variables
2. Set up database migration pipeline
3. Add monitoring and error tracking
4. Configure proper security headers and HTTPS
5. Set up backup and disaster recovery

Follow Wasp deployment best practices and ensure scalability.
```

## Documentation and Maintenance Prompts

### Code Documentation
```
I need to document the financial calculation engine:

1. Add comprehensive JSDoc comments to all calculation functions
2. Document tax compliance requirements and sources
3. Create developer guide for adding new countries
4. Document testing procedures and validation requirements
5. Add troubleshooting guide for common issues

Ensure documentation is clear for both developers and tax professionals.
```

### Professional Validation
```
I need to prepare documentation for professional tax review:

1. Extract all tax calculation methodologies
2. Document sources for all tax rates and rules
3. Create test case documentation with expected results
4. Prepare compliance validation checklist
5. Generate accuracy audit reports

Format documentation for review by qualified tax professionals in each country.
```

## Troubleshooting Prompts

### Debug Calculation Errors
```
I'm getting incorrect tax calculation results. Please help me debug:

1. Verify Decimal arithmetic is used throughout
2. Check tax bracket applications and edge cases
3. Validate input sanitization and error handling
4. Compare results with official tax calculators
5. Add detailed logging for calculation steps

Focus on maintaining calculation accuracy and identifying the root cause.
```

### Fix Performance Issues
```
Financial calculations are taking too long. Please help optimize:

1. Profile calculation performance and identify bottlenecks
2. Optimize database queries and caching strategies
3. Review algorithm efficiency and data structures
4. Add progress indicators for long-running calculations
5. Implement calculation timeout handling

Ensure optimizations don't compromise accuracy or introduce bugs.
```

## Best Practices Reminders

When using these prompts, remember:

1. **Tax Compliance First**: All calculations must be ATO/IRS/HMRC compliant
2. **Use Decimal Arithmetic**: Never use floating point for currency calculations
3. **Follow Wasp Conventions**: Use proper import paths and operation patterns
4. **Progressive Complexity**: Start with ELI5 and add complexity gradually
5. **Test Thoroughly**: Include comprehensive testing with real-world scenarios
6. **Document Sources**: Include references for all tax rules and calculations
7. **Professional Validation**: Ensure qualified professionals review tax calculations