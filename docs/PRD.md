# FinPath Explorer - Product Requirements Document (PRD)

## Executive Summary

### Vision Statement
Create a comprehensive financial planning platform that enables users across Australia, USA, and UK to explore pre-retirement financial scenarios through intuitive drag-and-drop strategy building, with full tax compliance and adaptive complexity from beginner to expert levels.

### App Type
**Global Financial Scenario Planning Platform**

### Core Value Proposition
**"Be comfortable you have enough money, and get advice on how to improve your situation"** - delivered through sophisticated financial modeling with an interface that adapts from ELI5 simplicity to expert-level customization.

## Strategic Decisions

### 1. **Multi-Country Coverage**: Australia + USA + UK from Launch
- Global technical architecture designed from day one
- Localized tax engines with full compliance (ATO, IRS, HMRC)
- Country-specific investment universes and strategy templates
- Cross-border scenarios for expats and dual tax residents

### 2. **Full Tax Compliance**: Comprehensive but User-Friendly
- **Australia**: Complete ATO compliance (negative gearing, franking credits, CGT, superannuation)
- **USA**: Full federal + state tax integration (401k, Roth, tax-loss harvesting)
- **UK**: Complete HMRC compliance (ISAs, pensions, capital gains)
- Sophisticated calculations hidden behind adaptive UI complexity

### 3. **Template-First with Easy Unlocking**: Guided Progression
- Start users with 6 country-specific strategy templates
- Easy unlock system for customization (not restrictive barriers)
- Progressive complexity as users engage and learn
- Expert users can access full customization immediately

### 4. **Mock Data Integration**: Focus on Core Logic First
- Static/mock data for property values and market returns (MVP)
- Real-time integration for interest rates and major prices only
- **TODO Phase 2**: Full real-time data integration with property APIs and market data

## Target Users

### Global User Segmentation

#### **ELI5 (Beginner) Users - 40% of Market**
- **Profile**: New to investing, overwhelmed by financial planning complexity
- **Age**: 25-45, early to mid-career
- **Goals**: "Will I have enough to retire?" and "What should I do first?"
- **Interface Needs**: Simple sliders, visual progress bars, plain English explanations
- **Success Metric**: Complete first scenario assessment and understand their status

#### **Intermediate Users - 45% of Market**
- **Profile**: Some investment knowledge, want more control and options
- **Age**: 35-55, peak earning years
- **Goals**: "How do I optimize my strategy?" and "What are my options?"
- **Interface Needs**: Detailed inputs, comparison charts, moderate financial terminology
- **Success Metric**: Create and compare multiple scenarios, take optimization actions

#### **Expert Users - 15% of Market**
- **Profile**: High financial literacy, want maximum control and validation
- **Age**: 30-65, includes financial professionals and sophisticated investors
- **Goals**: "Validate my complex strategy" and "Fine-tune tax optimization"
- **Interface Needs**: Full parameter control, advanced analytics, detailed assumptions
- **Success Metric**: Create custom scenarios and validate against existing strategies

### Country-Specific Personas

#### **Australia**
- **Emma (32, Sydney)**: Software engineer, $120k salary, wants to know rent vs buy decision
- **Mark (45, Melbourne)**: Business owner, $180k income, interested in negative gearing strategy
- **Sarah (38, Brisbane)**: Teacher, $80k salary, worried about superannuation adequacy

#### **USA**
- **David (34, San Francisco)**: Tech worker, $150k salary, maximizing 401k and considering FIRE
- **Lisa (42, Austin)**: Marketing manager, $95k salary, optimizing between Roth and Traditional IRA
- **Mike (48, Chicago)**: Consultant, $130k salary, tax-loss harvesting and state tax optimization

#### **UK**
- **James (36, London)**: Finance professional, £85k salary, ISA optimization and pension planning
- **Rachel (40, Manchester)**: NHS worker, £55k salary, concerned about pension adequacy
- **Tom (44, Edinburgh)**: Self-employed, £70k income, optimizing taxes and retirement planning

## Core Features - Scenario-Based Strategy Builder

### Scenario Architecture

#### Universal Scenario Structure
```
Scenario: "Conservative Property + Index Strategy"
├── Country Selection: Australia | USA | UK
├── Housing Strategy: Buy/Rent/Invest components
├── Investment Strategy: Local + International options
├── Tax Strategy: Country-specific optimization
├── Timeline: Age-based planning horizon
└── Results: Projections with adaptive complexity display
```

### Drag-and-Drop Strategy Components

#### **Housing Components (Country-Localized)**

**Australia:**
- **Buy Primary Residence**: Stamp duty, FHOG, mortgage rates, property growth
- **Investment Property**: Multiple properties, LVR ratios, rental yields, negative gearing
- **Rent Vest**: Rent primary residence, invest in property elsewhere

**USA:**
- **Buy Primary Residence**: Mortgage interest deduction, property tax, PMI considerations
- **Investment Property**: Depreciation schedules, 1031 exchanges, rental income taxation
- **Rent vs Buy**: State-specific tax implications, mobility considerations

**UK:**
- **Buy Primary Residence**: Stamp duty, Help to Buy schemes, mortgage types
- **Buy to Let**: Capital gains tax, mortgage interest relief, rental income tax
- **Rent vs Buy**: Regional property market considerations

#### **Investment Components (Country-Localized)**

**Australia:**
- **ASX ETFs**: VAS, VGS, VTS, VAE with franking credit calculations
- **Individual Shares**: ASX stocks with dividend imputation modeling
- **International Shares**: Currency hedging, foreign tax implications
- **Superannuation**: Concessional/non-concessional contributions, transition strategies

**USA:**
- **US ETFs**: VTI, VXUS, BND, VNQ with tax-efficient structures
- **Individual Stocks**: Qualified vs ordinary dividends, tax implications
- **International Funds**: Foreign tax credit optimization
- **Retirement Accounts**: 401k, Traditional IRA, Roth IRA, HSA optimization

**UK:**
- **UK Funds**: FTSE trackers, global equity funds within ISA wrappers
- **Individual Shares**: ISA vs GIA tax implications, dividend taxation
- **International Investments**: Currency considerations, withholding taxes
- **Pensions**: Workplace pensions, SIPP, contribution optimization

#### **Tax Strategy Components (Country-Specific)**

**Australia:**
- **Negative Gearing**: Automatic calculation for investment properties
- **Franking Credits**: Full dividend imputation modeling and refunds
- **Capital Gains Tax**: 50% discount calculation, timing optimization
- **Superannuation**: Contribution caps, co-contribution, government benefits

**USA:**
- **Tax-Loss Harvesting**: Wash sale rule compliance, optimal timing
- **Roth Conversions**: Tax bracket optimization, conversion timing
- **Depreciation**: Real estate and equipment depreciation schedules
- **Tax-Advantaged Accounts**: 401k, IRA, HSA contribution optimization

**UK:**
- **ISA Optimization**: Annual allowance maximization, S&S vs Cash ISA
- **Pension Contributions**: Annual allowance, lifetime allowance, tax relief
- **Capital Gains Management**: Annual exemption utilization, timing strategies
- **Dividend Tax**: Allowance optimization, tax-efficient fund selection

### Pre-Built Strategy Templates (6 per Country)

#### **Australia Templates**
1. **"Conservative Growth"**: Buy home + VAS/VGS ETFs + extra super
2. **"Property Investor"**: Multiple investment properties with negative gearing
3. **"Rent Vest Accelerator"**: Rent primary + invest in high-yield property elsewhere
4. **"Index Fund FIRE"**: Rent + aggressive ETF savings + early retirement
5. **"Balanced Approach"**: Property + shares + super optimization
6. **"Young Professional"**: High growth shares + minimal insurance + super

#### **USA Templates**
1. **"Traditional Path"**: Buy home + 401k max + index fund allocation
2. **"FIRE Strategy"**: Rent + aggressive savings + tax-loss harvesting
3. **"Real Estate Focus"**: Investment properties + REITs + depreciation optimization
4. **"Roth Conversion"**: Traditional to Roth optimization + tax management
5. **"Balanced Growth"**: Home + diversified portfolio + retirement accounts
6. **"Tech Worker Special"**: RSU management + mega backdoor Roth + tax optimization

#### **UK Templates**
1. **"ISA Maximizer"**: Buy home + max ISA contributions + pension optimization
2. **"Buy to Let"**: Property investment + mortgage interest optimization
3. **"Pension Focus"**: Aggressive pension contributions + tax relief maximization
4. **"Balanced Portfolio"**: Property + ISA + pension balanced approach
5. **"Early Retirement"**: High savings rate + tax-efficient withdrawal strategy
6. **"Higher Rate Relief"**: Pension contributions to optimize tax bands

### Adaptive Complexity Interface

#### **ELI5 Interface Example**
```
Australian Property + ETF Strategy
🏠 Buy house: $800k (20% deposit needed: $160k)
📈 Invest: $500/month in Australian + International shares
💰 Superannuation: Employer contributions + $200/month extra
📊 Result at age 60: $1.4M total wealth
✅ "You're on track for comfortable retirement!"
💡 "Increase your savings by $100/month to retire 2 years earlier"
```

#### **Expert Interface Example**
```
Investment Property Analysis:
Property Details:
- Purchase Price: $650,000 (80% LVR)
- Rental Yield: 4.2% gross (3.1% net after expenses)
- Capital Growth Assumption: 6% p.a.
- Loan Interest Rate: 5.8% p.a.

Tax Implications:
- Negative Gearing: -$8,700 p.a.
- Tax Saving: $3,219 p.a. (37% marginal bracket)
- Net Cash Flow: -$448/month after tax benefits

10-Year Projection:
- Total Return (IRR): 8.4% p.a. including tax benefits
- Capital Growth: $358,000
- Tax Savings: $32,190
- Net Investment: $53,760
```

### Cross-Border Scenarios (Advanced Feature)

#### **Dual Tax Residents**
- **US Citizens in Australia**: PFIC implications, superannuation taxation
- **UK Residents with US Assets**: Tax treaty benefits, 401k treatment
- **Australian Expats**: Superannuation access, non-resident tax rates

#### **Investment Restrictions**
- **US Citizens**: Cannot efficiently own foreign ETFs (PFIC rules)
- **UK Residents**: ISA restrictions for non-UK funds
- **Australian Non-Residents**: Superannuation access limitations

## Technical Requirements

### Multi-Country Tax Engine Architecture

#### **Tax Engine Interface**
```typescript
interface TaxEngine {
  country: 'AU' | 'US' | 'UK'
  calculateIncomeTax(income: Income[], deductions: Deduction[]): TaxResult
  calculateCapitalGains(gains: CapitalGain[], holdingPeriod: number): TaxResult
  calculatePropertyTax(properties: Property[]): TaxResult
  calculateInvestmentTax(investments: Investment[]): TaxResult
  optimizeWithdrawals(accounts: Account[], targetAmount: number): WithdrawalStrategy
}
```

#### **Australian Tax Engine Specifications**
```typescript
class AustralianTaxEngine implements TaxEngine {
  // Core calculations
  - calculateNegativeGearing(properties: InvestmentProperty[]): NegativeGearingResult
  - calculateFrankingCredits(dividends: AustralianDividend[]): FrankingCreditResult
  - calculateCGTDiscount(gains: CapitalGain[]): CGTResult
  - calculateSuperContributions(contributions: SuperContribution[]): SuperResult
  
  // Tax brackets and rates (updated annually)
  - incomeTaxBrackets: TaxBracket[]
  - medicareLevyRates: MedicareLevyRate[]
  - superGuaranteeRate: number
  - superContributionCaps: SuperCaps
}
```

#### **US Tax Engine Specifications**
```typescript
class AmericanTaxEngine implements TaxEngine {
  // Federal + State calculations
  - calculateFederalIncomeTax(income: Income[], state: USState): FederalTaxResult
  - calculateStateTax(income: Income[], state: USState): StateTaxResult
  - calculateTaxLossHarvesting(portfolio: Portfolio[]): TaxLossResult
  - calculateRetirementAccountTax(accounts: RetirementAccount[]): RetirementTaxResult
  - calculateDepreciation(properties: RentalProperty[]): DepreciationResult
  
  // Tax optimization
  - optimizeRothConversions(accounts: Account[], targetBracket: TaxBracket): ConversionStrategy
  - calculateQualifiedDividends(dividends: Dividend[]): DividendTaxResult
}
```

#### **UK Tax Engine Specifications**
```typescript
class BritishTaxEngine implements TaxEngine {
  // UK-specific calculations
  - calculateISAOptimization(contributions: ISAContribution[]): ISAResult
  - calculatePensionRelief(contributions: PensionContribution[]): PensionReliefResult
  - calculateCapitalGainsTax(gains: CapitalGain[], allowances: CGTAllowance[]): CGTResult
  - calculateDividendTax(dividends: UKDividend[], allowances: DividendAllowance[]): DividendTaxResult
  
  // Tax band optimization
  - optimizeTaxBands(income: Income[], contributions: Contribution[]): BandOptimization
  - calculateNationalInsurance(earnings: Earnings[]): NIResult
}
```

### Database Schema for Multi-Country Scenarios

#### **Core Scenario Models**
```prisma
model Scenario {
  id          String   @id @default(uuid())
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  name        String
  description String?
  country     Country
  currency    Currency
  isTemplate  Boolean  @default(false)
  
  // User relationship
  user        User     @relation(fields: [userId], references: [id])
  userId      String
  
  // Scenario components
  housingComponents     HousingComponent[]
  investmentComponents  InvestmentComponent[]
  taxComponents        TaxComponent[]
  
  // Results cache
  projectionResults    Json?
  lastCalculated      DateTime?
  
  @@map("scenarios")
}

model HousingComponent {
  id          String   @id @default(uuid())
  type        HousingType // BUY_PRIMARY, INVESTMENT_PROPERTY, RENT, RENT_VEST
  
  // Universal fields
  propertyValue    Float?
  deposit          Float?
  mortgageRate     Float?
  mortgageTerm     Int?
  
  // Country-specific fields (JSON for flexibility)
  countrySpecific  Json // Stamp duty, FHOG, PMI, etc.
  
  scenario    Scenario @relation(fields: [scenarioId], references: [id], onDelete: Cascade)
  scenarioId  String
  
  @@map("housing_components")
}

model InvestmentComponent {
  id              String   @id @default(uuid())
  type            InvestmentType // ETF, INDIVIDUAL_STOCK, PROPERTY, RETIREMENT_ACCOUNT
  
  // Universal fields
  monthlyContribution  Float?
  initialAmount       Float?
  expectedReturn      Float?
  
  // Investment specific
  ticker              String? // VAS, VTI, etc.
  allocation          Float?  // Portfolio percentage
  
  // Country-specific fields
  countrySpecific     Json // Franking credits, tax treatment, etc.
  
  scenario    Scenario @relation(fields: [scenarioId], references: [id], onDelete: Cascade)
  scenarioId  String
  
  @@map("investment_components")
}

enum Country {
  AUSTRALIA
  UNITED_STATES
  UNITED_KINGDOM
}

enum Currency {
  AUD
  USD
  GBP
}

enum HousingType {
  BUY_PRIMARY
  INVESTMENT_PROPERTY
  RENT
  RENT_VEST
  BUY_TO_LET
}

enum InvestmentType {
  ETF
  INDIVIDUAL_STOCK
  MUTUAL_FUND
  PROPERTY
  RETIREMENT_ACCOUNT
  SAVINGS_ACCOUNT
  BONDS
  ALTERNATIVES
}
```

### Wasp Operations for Financial Calculations

#### **Main.wasp Additions**
```wasp
// Scenario Management
query getUserScenarios {
  fn: import { getUserScenarios } from "@src/financial-planning/operations",
  entities: [User, Scenario]
}

query getScenarioDetails {
  fn: import { getScenarioDetails } from "@src/financial-planning/operations",
  entities: [Scenario, HousingComponent, InvestmentComponent]
}

query getCountryTemplates {
  fn: import { getCountryTemplates } from "@src/financial-planning/operations",
  entities: [Scenario]
}

action createScenario {
  fn: import { createScenario } from "@src/financial-planning/operations",
  entities: [Scenario, HousingComponent, InvestmentComponent, User]
}

action updateScenario {
  fn: import { updateScenario } from "@src/financial-planning/operations",
  entities: [Scenario, HousingComponent, InvestmentComponent]
}

action calculateScenarioProjections {
  fn: import { calculateScenarioProjections } from "@src/financial-planning/operations",
  entities: [Scenario, HousingComponent, InvestmentComponent]
}

action compareScenarios {
  fn: import { compareScenarios } from "@src/financial-planning/operations",
  entities: [Scenario]
}

// Template Management
query getStrategyTemplates {
  fn: import { getStrategyTemplates } from "@src/financial-planning/operations",
  entities: [Scenario]
}

action cloneTemplate {
  fn: import { cloneTemplate } from "@src/financial-planning/operations",
  entities: [Scenario, HousingComponent, InvestmentComponent, User]
}

// Tax Optimization
action optimizeTaxStrategy {
  fn: import { optimizeTaxStrategy } from "@src/financial-planning/operations",
  entities: [Scenario, TaxComponent]
}

// Data Integration (Mock for MVP)
query getMarketData {
  fn: import { getMarketData } from "@src/financial-planning/operations",
  entities: []
}
```

## Implementation Roadmap

### Phase 1: Australia Foundation (Months 1-3)

#### **Month 1: Core Infrastructure**
- Multi-country database schema implementation
- Australian tax engine (basic negative gearing, franking credits)
- User country selection and setup flow
- Basic scenario creation (housing + investment components)

#### **Month 2: Australian Templates & Features**
- 6 Australian strategy templates implementation
- Drag-and-drop component builder
- Australian investment universe (ASX ETFs, major stocks)
- Superannuation calculations and optimization

#### **Month 3: Australian Tax Compliance & UX**
- Full ATO compliance (CGT discount, land tax, Medicare levy)
- Adaptive complexity interface (ELI5 → Expert)
- Scenario comparison and optimization
- Mock data integration with real interest rates

**Phase 1 Success Metrics:**
- 500 Australian users registered
- 80% complete at least one scenario
- 60% create multiple scenarios for comparison
- Average 4 scenarios per active user

### Phase 2: USA Expansion (Month 4)

#### **Month 4: US Tax Engine & Templates**
- US federal + state tax engine implementation
- 6 US strategy templates (401k focus, FIRE, real estate)
- US investment universe (US ETFs, major stocks)
- Currency conversion and multi-country user handling

**Phase 2 Success Metrics:**
- 1,000 US users registered (larger market)
- 70% template → customization progression
- 50% premium conversion rate

### Phase 3: UK Expansion (Month 5)

#### **Month 5: UK Tax Engine & Templates**
- UK tax engine (ISA, pension, capital gains)
- 6 UK strategy templates (ISA focus, buy-to-let, pension optimization)
- UK investment universe (LSE stocks, UK funds)
- Cross-border scenario capabilities (basic)

**Phase 3 Success Metrics:**
- 400 UK users registered
- Strong ISA optimization engagement
- Cross-border user scenarios (expats)

### Phase 4: Advanced Features (Months 6-8)

#### **Month 6: Advanced Tax Optimization**
- Monte Carlo simulations for market volatility
- Advanced tax-loss harvesting (US)
- Complex negative gearing strategies (AU)
- Pension optimization (UK)

#### **Month 7: Cross-Border & Expert Features**
- Dual tax resident scenarios (US citizens abroad, etc.)
- Advanced portfolio optimization
- Custom scenario builder (full expert mode)
- Financial advisor collaboration tools

#### **Month 8: Data Integration & Polish**
- **TODO**: Real-time property data integration
- **TODO**: Live market data for portfolios
- Advanced reporting and export capabilities
- Mobile app development

### Phase 5: Scale & Enterprise (Months 9-12)

#### **Month 9-10: Enterprise Features**
- Financial advisor marketplace
- White-label solutions for financial advisors
- Advanced reporting and client management
- API access for integrations

#### **Month 11-12: Additional Countries & Features**
- Canada expansion (similar tax system to US/UK)
- Advanced international tax scenarios
- AI-powered optimization suggestions
- Partnership integrations (brokerages, banks)

## Success Metrics & KPIs

### Business Metrics

#### **Revenue Targets**
- **Month 6**: $50k MRR across all countries
- **Month 12**: $200k MRR with enterprise features
- **18 Months**: $500k MRR with international expansion

#### **User Acquisition**
- **Australia**: 2,000 active users by month 6
- **USA**: 5,000 active users by month 8
- **UK**: 1,500 active users by month 8
- **Total**: 10,000+ active users by year end

#### **Conversion Metrics**
- **Free to Premium**: 40% conversion rate
- **Template to Custom**: 70% progression rate
- **Scenario Completion**: 85% finish first scenario
- **Multi-Scenario**: 60% create 3+ scenarios

### Product Metrics

#### **Engagement**
- **Scenario Creation**: Average 5 scenarios per active user
- **Session Time**: 15+ minutes average session
- **Return Rate**: 70% return within 7 days
- **Feature Adoption**: 80% use template → customization flow

#### **Tax Optimization Impact**
- **Average Tax Savings Identified**: $3,000+ per user annually
- **Strategy Changes**: 40% modify approach based on scenarios
- **Confidence Improvement**: 90% report increased confidence in financial planning

### Technical Metrics

#### **Performance**
- **Calculation Speed**: <2 seconds for complex scenarios
- **Page Load**: <1 second for scenario builder
- **Uptime**: 99.9% availability
- **Tax Accuracy**: 100% compliance with tax authority rules

#### **Data Quality**
- **Mock Data**: Reasonable approximations for MVP
- **Real-time Updates**: Daily updates for interest rates, major indices
- **TODO Phase 2**: Weekly property data updates, daily market prices

## Competitive Analysis & Differentiation

### Direct Competitors

#### **Global Financial Planning Tools**
- **Personal Capital**: US-focused, limited tax optimization
- **Sharesight**: AU focus, portfolio tracking not planning
- **Quicken**: US market, desktop-based, limited scenarios

#### **Country-Specific Competitors**
- **Australia**: MoneySmart calculator, Sorted.org.nz
- **USA**: NewRetirement, FidSafe, Mint
- **UK**: PensionBee, Nutmeg, MoneyBox

### Unique Differentiation

#### **Multi-Country Tax Sophistication**
- **Only platform** with full AU + US + UK tax compliance
- **Scenario-based approach** vs simple calculators
- **Adaptive complexity** serves beginners to experts equally well
- **Cross-border capabilities** for global users

#### **Drag-and-Drop Strategy Building**
- **Visual strategy construction** vs form-based input
- **Template starting points** with full customization
- **Real-time tax optimization** as components change
- **Scenario comparison** side-by-side analysis

#### **Leveraging Existing SaaS Infrastructure**
- **Integrated platform** vs standalone calculator
- **Secure financial data** with enterprise auth
- **Scalable payments** for global subscriptions
- **Professional reporting** and export capabilities

## Risk Assessment & Mitigation

### Technical Risks

#### **Tax Calculation Complexity**
- **Risk**: Incorrect tax calculations damage user trust
- **Mitigation**: Comprehensive testing, tax professional review, clear disclaimers
- **Monitoring**: Regular compliance audits, user feedback tracking

#### **Multi-Country Data Synchronization**
- **Risk**: Currency conversion errors, data inconsistencies
- **Mitigation**: Robust exchange rate handling, comprehensive testing
- **Monitoring**: Daily data quality checks, automated alerts

#### **Performance with Complex Scenarios**
- **Risk**: Slow calculations with multiple properties/investments
- **Mitigation**: Optimized algorithms, caching, progressive calculation
- **Monitoring**: Performance metrics, user session analysis

### Business Risks

#### **Regulatory Compliance**
- **Risk**: Financial advice regulation in multiple countries
- **Mitigation**: Clear disclaimers, educational focus, legal review
- **Monitoring**: Regulatory change tracking, compliance updates

#### **Market Competition**
- **Risk**: Large financial institutions launching competing products
- **Mitigation**: Focus on unique multi-country capabilities, rapid iteration
- **Monitoring**: Competitive analysis, feature differentiation

#### **User Adoption Challenges**
- **Risk**: Users find financial planning too complex despite UX efforts
- **Mitigation**: Extensive user testing, guided onboarding, educational content
- **Monitoring**: User completion rates, support ticket analysis

### Data & Privacy Risks

#### **Financial Data Security**
- **Risk**: Breach of sensitive financial information
- **Mitigation**: SOC2 compliance, encryption, minimal data storage
- **Monitoring**: Security audits, penetration testing

#### **Cross-Border Data**
- **Risk**: GDPR, privacy law compliance across countries
- **Mitigation**: Privacy by design, data residency options, legal compliance
- **Monitoring**: Privacy audit, regulatory compliance tracking

## Next Steps & Documentation

### Immediate Actions (Week 1)
1. **Feature Specification**: Detailed user stories for Australian tax engine
2. **Technical Architecture**: Database schema finalization and Wasp operations
3. **UX Design**: Wireframes for country selection and scenario builder
4. **Development Setup**: Multi-country infrastructure foundation

### Development Documentation Required
1. **Agent Development Guidelines**: Financial calculation patterns and testing
2. **Tax Engine Specifications**: Detailed algorithms for each country
3. **Template Definitions**: Exact parameters for each strategy template
4. **Integration Strategy**: How to leverage existing SaaS infrastructure

### Validation & Testing Strategy
1. **Tax Accuracy**: Professional review of all tax calculations
2. **User Testing**: Extensive testing across all three countries
3. **Performance Testing**: Complex scenario calculation optimization
4. **Security Testing**: Financial data protection and privacy compliance

---

*This PRD establishes FinPath Explorer as the world's first comprehensive multi-country financial planning platform, combining sophisticated tax optimization with intuitive drag-and-drop scenario building for users from beginner to expert levels.*