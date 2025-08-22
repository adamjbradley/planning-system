# FinPath Explorer - Implementation Roadmap

## Implementation Strategy: Vertical Slice + Progressive Enhancement

This roadmap uses a vertical slice approach, delivering basic end-to-end functionality quickly, then progressively adding complexity and country-specific features. Each phase produces working, demoable functionality.

## Phase 1: MVP Foundation (ELI5 Mode Only)
*Target: Basic financial scenario planning for one country*

### 1.1 Database Foundation & Multi-Country Setup (Week 1)
```
src/setup/
├── server/
│   ├── queries.ts          # getUserProfile, getCountryConfig
│   ├── actions.ts          # updateUserProfile, selectCountry  
│   └── schemas.ts          # Prisma schema extensions
├── client/
│   ├── CountrySelector.tsx # Simple country selection
│   ├── ProfileSetup.tsx    # Basic user profile (age, income)
│   └── ComplexityToggle.tsx # ELI5 mode only initially
└── shared/
    └── types.ts            # Core interfaces
```

**Key Tasks:**
- [ ] Extend Prisma schema with User.country, User.complexityLevel, basic Scenario model
- [ ] Create country selection with AU/US/UK flags and currency detection
- [ ] Build basic user profile setup (age, income, primary country)
- [ ] Implement ELI5-only complexity mode
- [ ] Create simple onboarding flow

**Acceptance Criteria:**
- User can select country and complete basic profile
- Database stores user preferences correctly
- Simple onboarding flow functional

### 1.2 Basic Scenario Management (Week 2)
```
src/scenario/
├── server/
│   ├── queries.ts          # getScenarios, getScenario
│   ├── actions.ts          # createScenario, updateScenario
│   └── calculations.ts     # Basic calculation engine
├── client/
│   ├── ScenarioList.tsx    # Simple list view
│   ├── ScenarioForm.tsx    # Basic creation form
│   └── ScenarioCard.tsx    # Simple scenario display
└── shared/
    └── types.ts            # Scenario interfaces
```

**Key Tasks:**
- [ ] Create basic Scenario CRUD operations
- [ ] Build simple scenario list and creation form
- [ ] Implement basic scenario calculations (no tax optimization)
- [ ] Add scenario naming and basic description
- [ ] Create simple navigation between scenarios

**Acceptance Criteria:**
- Users can create, edit, and delete scenarios
- Basic financial calculations work correctly
- Scenarios persist in database

### 1.3 Simple Strategy Templates (Week 3)
```
src/template/
├── server/
│   ├── queries.ts          # getTemplates, getTemplate
│   ├── actions.ts          # applyTemplate
│   └── templates/
│       ├── BasicTemplates.ts # 3-4 universal templates
│       └── CountryTemplates.ts # 1-2 per country
├── client/
│   ├── TemplateGallery.tsx # Simple grid of templates
│   ├── TemplateCard.tsx    # Basic template display
│   └── TemplateModal.tsx   # Simple application
└── shared/
    └── types.ts            # Template interfaces
```

**Key Tasks:**
- [ ] Create 3-4 basic universal templates (Conservative, Balanced, Growth)
- [ ] Add 1-2 country-specific templates per country
- [ ] Build simple template gallery with filtering
- [ ] Implement one-click template application
- [ ] Add basic template customization

**Acceptance Criteria:**
- Users can browse and apply templates
- Templates create valid scenarios
- Basic customization works

### 1.4 Basic Housing Component (Week 4)
```
src/housing/
├── server/
│   ├── queries.ts          # getHousingData
│   ├── actions.ts          # updateHousingComponent
│   └── calculations.ts     # Basic buy vs rent
├── client/
│   ├── HousingForm.tsx     # Simple buy/rent forms
│   ├── BuyVsRent.tsx       # Basic comparison
│   └── HousingResults.tsx  # Simple results display
└── shared/
    └── types.ts            # Housing interfaces
```

**Key Tasks:**
- [ ] Create basic housing component with buy/rent options
- [ ] Implement simple buy vs rent comparison
- [ ] Add basic mortgage calculator
- [ ] Create simple housing cost projections
- [ ] Build basic housing form inputs

**Acceptance Criteria:**
- Users can input housing scenarios (buy vs rent)
- Basic mortgage calculations work
- Simple projections display correctly

**Phase 1 Demo Criteria:**
✅ User completes country selection and profile
✅ Creates scenario using template
✅ Compares basic buy vs rent options  
✅ Views simple financial projections

---

## Phase 2: Intermediate Complexity & Country Features (Weeks 5-8)

### 2.1 Tax Engine Foundation (Week 5)
```
src/tax/
├── server/
│   ├── engines/
│   │   ├── ITaxEngine.ts      # Interface
│   │   ├── AustralianTax.ts   # AU tax calculations
│   │   ├── USATax.ts          # US tax calculations
│   │   └── UKTax.ts           # UK tax calculations
│   ├── TaxEngineFactory.ts    # Factory pattern
│   └── calculations.ts        # Shared tax utilities
├── client/
│   ├── TaxSummary.tsx         # Tax breakdown display
│   └── TaxOptimizations.tsx   # Basic suggestions
└── shared/
    └── types.ts               # Tax calculation types
```

**Key Tasks:**
- [ ] Build tax engine factory with country-specific implementations
- [ ] Implement basic tax calculations for each country
- [ ] Add marginal tax rate calculations
- [ ] Create tax optimization suggestions
- [ ] Build tax summary components

### 2.2 Advanced Housing Strategies (Week 6)
```
src/housing/
├── server/
│   ├── country-specific/
│   │   ├── AustralianHousing.ts # Negative gearing, CGT
│   │   ├── USAHousing.ts        # Depreciation, 1031 exchange
│   │   └── UKHousing.ts         # Buy-to-let, Help to Buy
│   └── PropertyDataService.ts   # Mock market data
├── client/
│   ├── InvestmentProperty.tsx   # Investment property form
│   ├── TaxOptimizations.tsx     # Housing tax optimizations
│   └── PropertySearch.tsx       # Basic property search
└── shared/
    └── types.ts
```

**Key Tasks:**
- [ ] Add investment property calculations
- [ ] Implement country-specific tax optimizations (negative gearing, depreciation)
- [ ] Create mock property market data service
- [ ] Build investment property forms and calculations
- [ ] Add basic property search functionality

### 2.3 Investment Portfolio Component (Week 7)
```
src/investment/
├── server/
│   ├── calculations.ts         # Portfolio calculations
│   ├── country-specific/
│   │   ├── RetirementAccounts.ts # 401k, Super, ISA calculations
│   │   └── TaxOptimizations.ts   # Country-specific optimizations
│   └── MarketDataService.ts    # Mock investment data
├── client/
│   ├── PortfolioBuilder.tsx    # Investment allocation
│   ├── RetirementAccounts.tsx  # Retirement account optimization
│   └── InvestmentResults.tsx   # Performance projections
└── shared/
    └── types.ts
```

**Key Tasks:**
- [ ] Create investment portfolio component
- [ ] Add retirement account optimization (401k, Super, ISA)
- [ ] Implement country-specific investment tax rules
- [ ] Build portfolio allocation interface
- [ ] Add investment performance projections

### 2.4 Intermediate UI & Complexity Toggle (Week 8)
```
src/shared/
├── components/
│   ├── ComplexityManager.tsx   # ELI5 ↔ Intermediate toggle
│   ├── ProgressiveDisclosure.tsx # Show/hide advanced options
│   └── EducationalTooltips.tsx # Context-sensitive help
└── hooks/
    ├── useComplexity.ts        # Complexity state management
    └── useProgression.ts       # Feature progression logic
```

**Key Tasks:**
- [ ] Build ELI5 ↔ Intermediate complexity toggle
- [ ] Implement progressive disclosure for advanced features
- [ ] Add educational tooltips and help content
- [ ] Create guided feature progression
- [ ] Build intermediate-level forms and interfaces

**Phase 2 Demo Criteria:**
✅ Country-specific tax optimizations working
✅ Investment property and portfolio components functional
✅ Users can toggle between ELI5 and Intermediate modes
✅ Tax calculations show country-specific benefits

---

## Phase 3: Expert Mode & Advanced Features (Weeks 9-12)

### 3.1 Expert Mode Interface (Week 9)
```
src/expert/
├── client/
│   ├── ExpertDashboard.tsx     # Advanced dashboard
│   ├── AdvancedCalculations.tsx # Detailed projections
│   ├── ScenarioComparison.tsx  # Side-by-side analysis
│   └── TaxOptimizationPanel.tsx # Advanced tax strategies
└── components/
    ├── ParameterCustomization.tsx # Full parameter control
    └── MonteCarloSettings.tsx     # Simulation parameters
```

**Key Tasks:**
- [ ] Build expert dashboard with advanced controls
- [ ] Create full parameter customization interface
- [ ] Add side-by-side scenario comparison
- [ ] Implement advanced tax optimization panel
- [ ] Build detailed projection views

### 3.2 Advanced Analytics & Simulations (Week 10)
```
src/analytics/
├── server/
│   ├── MonteCarloSimulation.ts # Probability analysis
│   ├── SensitivityAnalysis.ts  # Parameter sensitivity
│   └── OptimizationEngine.ts   # Strategy optimization
├── client/
│   ├── MonteCarloResults.tsx   # Simulation visualization
│   ├── SensitivityCharts.tsx   # Sensitivity analysis
│   └── OptimizationSuggestions.tsx # AI-powered suggestions
└── shared/
    └── types.ts
```

**Key Tasks:**
- [ ] Implement Monte Carlo simulations
- [ ] Add sensitivity analysis for key parameters
- [ ] Create optimization engine for strategy suggestions
- [ ] Build advanced visualization components
- [ ] Add probability-based projections

### 3.3 Cross-Border Planning (Week 11)
```
src/cross-border/
├── server/
│   ├── DualTaxResidency.ts     # Multiple country tax calculations
│   ├── CurrencyConversion.ts   # Multi-currency handling
│   └── ExpatScenarios.ts       # Expat-specific calculations
├── client/
│   ├── CrossBorderSetup.tsx    # Multi-country configuration
│   ├── DualResidencyForm.tsx   # Tax residency setup
│   └── ExpatDashboard.tsx      # Expat-specific interface
└── shared/
    └── types.ts
```

**Key Tasks:**
- [ ] Implement dual tax residency calculations
- [ ] Add multi-currency scenario support
- [ ] Create expat-specific planning features
- [ ] Build cross-border tax optimization
- [ ] Add currency hedging considerations

### 3.4 Advanced Templates & Professional Features (Week 12)
```
src/professional/
├── server/
│   ├── AdvancedTemplates.ts    # Professional strategy templates
│   ├── ReportGeneration.ts     # PDF report generation
│   └── ValidationEngine.ts    # Professional validation
├── client/
│   ├── ProfessionalTemplates.tsx # Advanced template gallery
│   ├── ReportBuilder.tsx       # Custom report builder
│   └── ValidationResults.tsx   # Validation feedback
└── shared/
    └── types.ts
```

**Key Tasks:**
- [ ] Create advanced professional templates
- [ ] Implement PDF report generation
- [ ] Add professional validation engine
- [ ] Build custom report builder
- [ ] Create template validation system

**Phase 3 Demo Criteria:**
✅ Expert mode with full parameter control
✅ Monte Carlo simulations and advanced analytics
✅ Cross-border planning for expats
✅ Professional-grade reports and validation

---

## Phase 4: Production Polish & Optimization (Weeks 13-16)

### 4.1 Performance Optimization (Week 13)
- [ ] Implement calculation caching and optimization
- [ ] Add database query optimization
- [ ] Create progressive loading for complex scenarios
- [ ] Optimize bundle size and lazy loading
- [ ] Add performance monitoring

### 4.2 Testing & Quality Assurance (Week 14)
- [ ] Comprehensive unit test suite for all calculations
- [ ] Integration tests for multi-country scenarios
- [ ] End-to-end testing for user workflows
- [ ] Tax calculation validation with professional review
- [ ] Cross-browser and device testing

### 4.3 Real Data Integration (Week 15)
- [ ] Integrate real property market data APIs
- [ ] Add real-time investment data feeds
- [ ] Implement currency conversion APIs
- [ ] Create data validation and fallback systems
- [ ] Add data freshness indicators

### 4.4 Production Deployment & Monitoring (Week 16)
- [ ] Production deployment configuration
- [ ] Error tracking and monitoring setup
- [ ] User analytics and usage tracking
- [ ] Performance monitoring and alerting
- [ ] Backup and disaster recovery

**Phase 4 Demo Criteria:**
✅ Production-ready performance and reliability
✅ Real market data integration
✅ Comprehensive testing coverage
✅ Full monitoring and error tracking

---

## Development Guidelines

### File Organization (Following SaaS Boilerplate Pattern)
```
src/
├── setup/          # Multi-country setup and user profile
├── scenario/       # Core scenario management
├── template/       # Strategy templates
├── housing/        # Housing strategies
├── investment/     # Investment portfolio
├── tax/           # Tax calculation engines
├── analytics/     # Advanced analytics and simulations
├── cross-border/  # Cross-border planning
├── professional/  # Professional features
└── shared/        # Shared components and utilities
    ├── components/
    ├── hooks/
    ├── utils/
    └── types/
```

### Each Feature Folder Structure:
```
feature/
├── client/         # React components and hooks
│   ├── components/ # Feature-specific components
│   ├── hooks/     # Feature-specific hooks
│   └── pages/     # Feature pages/routes
├── server/        # Wasp queries, actions, and business logic
│   ├── queries.ts # Wasp queries
│   ├── actions.ts # Wasp actions
│   └── services/  # Business logic services
└── shared/        # Shared types and utilities
    ├── types.ts   # TypeScript interfaces
    └── utils.ts   # Shared utilities
```

### Implementation Principles

1. **Start Simple**: Begin each feature with ELI5 complexity, add sophistication progressively
2. **Country-Agnostic Core**: Build interfaces that work for any country, implement country-specific logic separately
3. **Tax Compliance First**: All tax calculations must be accurate and compliant from day one
4. **Progressive Enhancement**: Each phase should build on the previous without breaking existing functionality
5. **LLM-Friendly**: Clear interfaces, good TypeScript types, and comprehensive comments for AI assistance

### Key Success Metrics

**Phase 1**: Basic end-to-end user journey functional
**Phase 2**: Country-specific optimizations working correctly  
**Phase 3**: Expert users can create sophisticated strategies
**Phase 4**: Production-ready with real data and monitoring

### Risk Mitigation

- **Tax Compliance**: Validate all calculations with qualified professionals
- **Performance**: Monitor calculation times and optimize early
- **User Experience**: Regular user testing throughout development
- **Technical Debt**: Refactor progressively as complexity increases

This roadmap balances rapid delivery of user value with the sophisticated financial planning capabilities required for the final product. Each phase produces working, demoable functionality while building toward the full FinPath Explorer vision.