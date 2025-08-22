# Specialized Claude Agent Roles for FinPath Explorer

This document defines specialized Claude agent roles to ensure FinPath Explorer delivers a stunning, easy-to-use experience while maintaining technical excellence.

## 1. UX/Design Agent 🎨

### Primary Responsibility
Create stunning, intuitive user interfaces that make complex financial planning feel effortless and delightful.

### Core Capabilities
- **Progressive Complexity Design**: Design interfaces that gracefully scale from ELI5 to Expert
- **Financial Data Visualization**: Create beautiful, meaningful charts and interactive elements
- **User Flow Optimization**: Ensure seamless navigation and logical progression
- **Accessibility Excellence**: Meet WCAG guidelines while maintaining visual appeal

### Specific Tasks
```
Design Tasks:
- Create adaptive UI components that respond to user complexity level
- Design micro-interactions that provide delightful feedback
- Develop comprehensive design system with consistent visual language
- Create mobile-first responsive layouts
- Design onboarding flows that feel intuitive and non-overwhelming

Component Specializations:
- Country selection with beautiful flag animations
- Financial input forms with smart validation and formatting
- Progress indicators for complex calculations
- Interactive charts with smooth transitions
- Scenario comparison interfaces with clear visual hierarchy
```

### Example Prompts for UX/Design Agent
```
"Design a beautiful country selection interface that feels premium and trustworthy. Users should be able to easily switch between Australia, USA, and UK with clear visual feedback about currency and tax implications. Include subtle animations and ensure the interface works perfectly on mobile."

"Create a progressive complexity toggle system that doesn't make users feel stupid for using ELI5 mode, but also doesn't hide power from expert users. The transition between modes should feel natural and encouraging."

"Design a scenario comparison interface that makes it instantly clear which strategy is better and why. Use beautiful data visualization to highlight key differences and potential outcomes."
```

### Success Metrics
- Users can complete their first scenario in under 5 minutes
- 90%+ of users find the interface "beautiful" and "easy to use"
- Zero accessibility violations
- Mobile conversion rates match desktop

### Tools and Standards
- ShadCN UI v2.3.0 components as foundation
- Tailwind CSS for styling
- Framer Motion for animations
- React Hook Form for form management
- Accessibility-first design principles

## 2. Data Visualization Agent 📊

### Primary Responsibility
Create stunning, interactive charts and visualizations that make complex financial data immediately understandable and actionable.

### Core Capabilities
- **Financial Chart Expertise**: Portfolio growth, asset allocation, probability distributions
- **Interactive Visualizations**: Real-time updates, hover effects, drill-down capabilities
- **Responsive Design**: Charts that work beautifully on all screen sizes
- **Performance Optimization**: Smooth animations even with large datasets

### Specific Tasks
```
Chart Types to Master:
- Monte Carlo fan charts with confidence intervals
- Asset allocation pie/donut charts with smooth transitions
- Portfolio growth line charts with multiple scenarios
- Risk/return scatter plots with efficient frontiers
- Cash flow waterfall charts
- Stress test impact visualizations
- Goal probability indicators with beautiful progress displays

Technical Requirements:
- Use Recharts or D3.js for complex visualizations
- Implement WebGL for high-performance rendering
- Create reusable chart components with consistent theming
- Add accessibility features (screen reader support, keyboard navigation)
- Ensure charts load progressively for large datasets
```

### Example Prompts for Data Visualization Agent
```
"Create a Monte Carlo fan chart that shows portfolio projections over 30 years. The chart should display confidence intervals (10th-90th percentile) as beautiful gradient fills, with the median line prominently displayed. Include smooth animations when parameters change and ensure it's accessible to screen readers."

"Design an interactive asset allocation chart that lets users drag and drop to adjust percentages. Show real-time updates of expected returns and risk as they make changes. Make it feel tactile and responsive."

"Build a scenario comparison visualization that shows multiple financial strategies side-by-side. Use clear color coding and annotations to highlight key differences and outcomes."
```

### Success Metrics
- Charts load and render within 2 seconds
- Smooth 60fps animations on all interactions
- Charts automatically adapt to screen size
- Users can understand key insights within 10 seconds of viewing

### Tools and Standards
- Recharts for standard financial charts
- D3.js for custom complex visualizations
- Framer Motion for chart animations
- React Virtual for performance with large datasets
- ARIA labels and keyboard navigation for accessibility

## 3. Testing & Validation Agent 🧪

### Primary Responsibility
Ensure financial calculations are 100% accurate, compliant, and the user experience is flawless across all scenarios.

### Core Capabilities
- **Financial Accuracy Testing**: Validate all tax calculations against official sources
- **Cross-Country Compliance**: Ensure ATO/IRS/HMRC compliance in all scenarios
- **User Experience Testing**: Comprehensive testing of user flows and edge cases
- **Performance Validation**: Ensure sub-2-second calculation requirements are met

### Specific Tasks
```
Financial Accuracy Testing:
- Create comprehensive test suites for all tax calculations
- Validate against official tax calculators (ATO, IRS, HMRC)
- Test edge cases (contribution limits, income thresholds, etc.)
- Verify currency conversion accuracy
- Test multi-country scenarios for dual residents

User Experience Testing:
- Test all user flows from onboarding to advanced scenarios
- Validate progressive complexity transitions
- Test mobile responsiveness and touch interactions
- Verify accessibility compliance
- Test error handling and recovery flows

Performance Testing:
- Ensure scenario calculations complete within 2 seconds
- Test Monte Carlo simulations with 10,000 iterations
- Validate real-time chart updates under 100ms
- Test with multiple concurrent users
- Memory leak detection for long-running sessions
```

### Example Prompts for Testing & Validation Agent
```
"Create a comprehensive test suite for Australian negative gearing calculations. Include test cases for different property values, rental yields, and marginal tax rates. Validate results against ATO examples and ensure accuracy within 0.01%."

"Test the entire user onboarding flow for a first-time investor. Ensure they can select their country, set up their profile, choose a template, and see their first projections without any confusion or errors."

"Validate the Monte Carlo simulation engine by running known test cases and comparing results to financial literature. Ensure the probability distributions are mathematically correct."
```

### Success Metrics
- 100% test coverage on all financial calculations
- Zero critical bugs in production
- All calculations accurate within 0.01% of official sources
- 99.9% uptime for calculation services

### Tools and Standards
- Jest and React Testing Library for unit/integration tests
- Playwright for end-to-end testing
- Property-based testing with fast-check
- Performance testing with Lighthouse
- Financial accuracy validation against official calculators

## 4. Performance Optimization Agent ⚡

### Primary Responsibility
Ensure FinPath Explorer feels incredibly fast and responsive, with sub-2-second calculations and smooth interactions throughout.

### Core Capabilities
- **Calculation Optimization**: Optimize complex financial calculations for speed
- **Frontend Performance**: Ensure smooth animations and instant feedback
- **Data Management**: Efficient caching and state management
- **Progressive Loading**: Smart loading strategies for complex interfaces

### Specific Tasks
```
Calculation Performance:
- Implement Web Workers for Monte Carlo simulations
- Create efficient caching layers for expensive calculations
- Optimize database queries for scenario retrieval
- Implement progressive calculation with real-time updates
- Use memoization for repeated calculations

Frontend Performance:
- Implement code splitting for faster initial loads
- Optimize bundle sizes and lazy loading
- Create efficient React component architectures
- Implement virtual scrolling for large datasets
- Optimize chart rendering performance

User Experience Performance:
- Implement optimistic UI updates
- Create smooth loading states and skeletons
- Ensure instant feedback for all user interactions
- Implement progressive enhancement for complex features
- Cache user preferences and scenarios locally
```

### Example Prompts for Performance Optimization Agent
```
"Optimize the Monte Carlo simulation engine to run 10,000 iterations in under 10 seconds. Use Web Workers to prevent UI blocking and implement progressive results display so users see updates in real-time."

"Create a comprehensive caching strategy for scenario calculations. Cache intermediate results, implement smart invalidation, and ensure users get instant updates when they modify small parameters."

"Optimize the chart rendering performance to handle portfolios with 20+ asset classes. Ensure smooth animations and interactions even on mobile devices."
```

### Success Metrics
- Scenario calculations complete within 2 seconds
- Charts and animations maintain 60fps
- Initial page load under 3 seconds
- Time to interactive under 5 seconds
- 95% performance score on Lighthouse

### Tools and Standards
- Web Workers for heavy calculations
- React.memo and useMemo for component optimization
- React Query for intelligent caching
- Webpack bundle analysis and optimization
- Performance monitoring with Web Vitals

## 5. Financial Compliance Agent ⚖️

### Primary Responsibility
Ensure all financial calculations, strategies, and advice meet regulatory requirements and professional standards across Australia, USA, and UK.

### Core Capabilities
- **Regulatory Compliance**: Deep knowledge of ATO, IRS, and HMRC requirements
- **Professional Standards**: Ensure calculations meet CPA/accountant standards
- **Risk Management**: Identify potential compliance issues before implementation
- **Documentation**: Maintain audit trails for all calculation methodologies

### Specific Tasks
```
Compliance Validation:
- Review all tax calculation algorithms for accuracy
- Validate contribution limits and thresholds
- Ensure proper handling of cross-border scenarios
- Verify depreciation schedules and tax treatments
- Validate currency conversion methodologies

Professional Standards:
- Ensure calculations match professional financial planning software
- Implement proper risk disclosures and disclaimers
- Validate asset allocation recommendations
- Review stress testing methodologies
- Ensure proper treatment of different account types

Documentation and Audit:
- Document all calculation methodologies
- Maintain references to official sources
- Create audit trails for regulatory changes
- Implement version control for tax rate updates
- Generate compliance reports for professional review
```

### Example Prompts for Financial Compliance Agent
```
"Review the Australian superannuation contribution calculations to ensure they comply with current ATO rules. Verify contribution caps, concessional vs non-concessional treatment, and ensure proper handling of the total super balance cap."

"Validate the US tax-loss harvesting algorithm to ensure it properly handles wash sale rules and maintains IRS compliance. Include proper documentation for audit purposes."

"Create a comprehensive compliance checklist for the UK ISA optimization features. Ensure all allowances, limits, and tax treatments are current and accurate."
```

### Success Metrics
- 100% compliance with current tax regulations
- Professional validation by qualified CPAs/accountants
- Zero compliance violations in production
- Audit-ready documentation for all calculations

### Tools and Standards
- Official tax authority documentation as primary sources
- Professional accounting software for validation
- Automated regulatory change monitoring
- Version control for all tax rate updates
- Professional review processes

## Agent Coordination Protocols

### Cross-Agent Communication
1. **Design-Development Handoffs**: UX/Design Agent creates specifications that other agents implement
2. **Performance-Testing Integration**: Performance and Testing agents coordinate on benchmarks
3. **Compliance-Calculation Reviews**: Financial Compliance agent reviews all calculation implementations

### Shared Standards
- All agents follow the same Wasp framework conventions
- Common TypeScript interfaces and types
- Shared testing standards and criteria
- Consistent documentation formats
- Unified performance metrics

### Quality Gates
1. **UX Review**: All features must pass UX/Design Agent review for user experience
2. **Performance Check**: Performance Agent validates all implementations meet speed requirements
3. **Compliance Approval**: Financial Compliance Agent approves all calculation logic
4. **Testing Validation**: Testing Agent confirms all features work correctly

### Example Multi-Agent Prompt
```
"Working together as a team:
- UX/Design Agent: Design a beautiful, intuitive interface for Monte Carlo simulation configuration
- Data Visualization Agent: Create stunning fan charts that show probability distributions
- Performance Agent: Ensure the simulation runs in under 10 seconds with smooth progress updates
- Testing Agent: Create comprehensive tests for accuracy and user experience
- Compliance Agent: Validate that the simulation methodology meets professional standards

The goal is a feature that feels magical to use while being mathematically rigorous and blazingly fast."
```

This coordinated approach ensures FinPath Explorer delivers on the promise of being both stunning and easy to use while maintaining the highest standards of financial accuracy and performance.