# Investment Portfolio Management

## Feature Overview

**Purpose**: Comprehensive investment portfolio management system allowing users to build, optimize, and track diversified investment strategies across multiple asset classes with country-specific tax optimization and retirement account integration.

**User Stories**:
- As an AU user, I want to optimize my superannuation contributions and ETF allocation for tax efficiency
- As a US user, I want to maximize my 401k/IRA benefits while building a taxable investment portfolio
- As a UK user, I want to utilize my ISA allowances efficiently while planning pension contributions
- As an expert user, I want granular control over asset allocation and rebalancing strategies

**Priority**: P0 (Critical) - Core wealth-building component alongside housing

**Dependencies**: Multi-Country Setup, Scenario Management, Tax Engine Integration

## Technical Specifications

### Data Models

```prisma
model InvestmentComponent {
  id                  String   @id @default(cuid())
  scenarioId          String
  scenario            Scenario @relation(fields: [scenarioId], references: [id], onDelete: Cascade)
  
  // Component Identification
  type                InvestmentType
  name                String
  description         String?
  startYear           Int      @default(0)
  endYear             Int?     // Optional sale/withdrawal year
  
  // Core Investment Details
  initialAmount       Decimal  @default(0)
  monthlyContribution Decimal  @default(0)
  contributionGrowth  Decimal? @default(0.03) // Annual contribution increase
  
  // Return Assumptions
  expectedReturn      Decimal  @default(0.07)
  volatility          Decimal? @default(0.15) // For Monte Carlo simulations
  managementFees      Decimal  @default(0.001)
  performanceFees     Decimal? @default(0)
  
  // Country-Specific Account Details
  accountType         RetirementAccountType?
  employerMatch       Decimal? @default(0)    // US 401k matching
  matchingCap         Decimal? @default(0)    // Maximum employer match
  contributionLimit   Decimal? // Annual limits (auto-populated)
  
  // Tax Treatment
  taxTreatment        TaxTreatment
  frankedDividends    Boolean @default(false) // AU franking credits
  dividendYield       Decimal? @default(0.02)
  
  // Asset Allocation
  assetAllocation     InvestmentAllocation[]
  rebalancingStrategy RebalancingStrategy @default(ANNUAL)
  
  // Advanced Settings (Expert Mode)
  dollarCostAveraging Boolean @default(true)
  autoRebalancing     Boolean @default(true)
  taxLossHarvesting   Boolean @default(false) // US specific
  
  // Calculated Fields (updated by calculation engine)
  currentValue        Decimal? // Current portfolio value
  totalContributions  Decimal? // Total contributions to date
  totalReturn         Decimal? // Total return percentage
  annualizedReturn    Decimal? // Annualized return percentage
  
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
  
  @@map("investment_components")
}

model InvestmentAllocation {
  id                  String   @id @default(cuid())
  investmentId        String
  investment          InvestmentComponent @relation(fields: [investmentId], references: [id], onDelete: Cascade)
  
  assetClass          AssetClass
  targetPercentage    Decimal
  currentPercentage   Decimal?
  
  // Specific Investment Details
  ticker              String?  // ETF/Stock ticker
  isin                String?  // International identifier
  currency            Currency
  
  // Performance Data (from market data service)
  currentPrice        Decimal?
  lastUpdated         DateTime?
  
  @@map("investment_allocations")
}

enum InvestmentType {
  RETIREMENT_ACCOUNT  // 401k, Super, ISA, etc.
  TAXABLE_BROKERAGE  // General investment account
  EDUCATION_SAVINGS  // 529, Education ISA, etc.
  HEALTH_SAVINGS     // HSA (US)
  EMERGENCY_FUND     // High-yield savings
  CRYPTO_PORTFOLIO   // Cryptocurrency investments
  ALTERNATIVE_INVESTMENTS // REITs, commodities, etc.
}

enum RetirementAccountType {
  // Australia
  SUPER
  SMSF
  
  // USA  
  TRADITIONAL_401K
  ROTH_401K
  TRADITIONAL_IRA
  ROTH_IRA
  SEP_IRA
  SIMPLE_IRA
  HSA
  
  // UK
  WORKPLACE_PENSION
  PERSONAL_PENSION
  SIPP
  STOCKS_ISA
  CASH_ISA
  INNOVATIVE_FINANCE_ISA
  LIFETIME_ISA
  JUNIOR_ISA
}

enum TaxTreatment {
  PRE_TAX      // Contributions reduce taxable income
  POST_TAX     // Contributions from after-tax income  
  TAX_FREE     // Growth and withdrawals tax-free
  TAX_DEFERRED // Growth tax-deferred, withdrawals taxed
}

enum AssetClass {
  DOMESTIC_EQUITY      // Home country stocks
  INTERNATIONAL_EQUITY // Foreign developed markets
  EMERGING_MARKETS     // Emerging market equity
  BONDS_GOVERNMENT     // Government bonds
  BONDS_CORPORATE      // Corporate bonds
  BONDS_INTERNATIONAL  // International bonds
  REAL_ESTATE         // REITs
  COMMODITIES         // Gold, oil, etc.
  CASH               // Money market, savings
  CRYPTO             // Cryptocurrency
  ALTERNATIVES       // Private equity, hedge funds
}

enum RebalancingStrategy {
  NEVER
  QUARTERLY
  SEMI_ANNUAL
  ANNUAL
  THRESHOLD_BASED    // Rebalance when allocation drifts >5%
  CALENDAR_AND_THRESHOLD
}

model InvestmentPerformance {
  id                  String   @id @default(cuid())
  investmentId        String
  investment          InvestmentComponent @relation(fields: [investmentId], references: [id], onDelete: Cascade)
  
  // Performance Metrics
  year                Int
  startValue          Decimal
  endValue            Decimal
  contributions       Decimal
  withdrawals         Decimal  @default(0)
  fees                Decimal
  taxes               Decimal
  
  // Returns
  grossReturn         Decimal  // Before fees and taxes
  netReturn           Decimal  // After fees and taxes
  benchmark           String?  // Benchmark index
  benchmarkReturn     Decimal? // Benchmark performance
  
  // Risk Metrics
  volatility          Decimal?
  sharpeRatio         Decimal?
  maxDrawdown         Decimal?
  
  createdAt           DateTime @default(now())
  
  @@unique([investmentId, year])
  @@map("investment_performance")
}
```

### Investment Calculator Engine

```typescript
export class InvestmentCalculator {
  constructor(
    private taxEngine: ITaxEngine,
    private marketDataService: MarketDataService
  ) {}
  
  async calculateInvestmentComponent(
    component: InvestmentComponent,
    scenario: Scenario,
    year: number
  ): Promise<InvestmentResults> {
    
    // Calculate contributions for the year
    const yearlyContribution = this.calculateYearlyContribution(component, year)
    
    // Calculate returns based on asset allocation
    const returns = await this.calculatePortfolioReturns(component, year)
    
    // Calculate fees
    const fees = this.calculateFees(component, returns.portfolioValue)
    
    // Calculate tax implications
    const taxCalculation = await this.calculateInvestmentTax(
      component,
      scenario,
      year,
      returns,
      fees
    )
    
    // Calculate portfolio rebalancing
    const rebalancingResults = this.calculateRebalancing(component, year)
    
    return {
      portfolioValue: returns.portfolioValue,
      contributions: yearlyContribution,
      grossReturn: returns.grossReturn,
      netReturn: returns.netReturn - fees.totalFees - taxCalculation.totalTax,
      fees: fees.totalFees,
      taxes: taxCalculation.totalTax,
      assetAllocation: returns.assetAllocation,
      rebalancingCost: rebalancingResults.cost,
      taxOptimizations: taxCalculation.optimizations
    }
  }
  
  private calculateYearlyContribution(
    component: InvestmentComponent,
    year: number
  ): number {
    const baseContribution = component.monthlyContribution * 12
    const growthRate = component.contributionGrowth || 0
    const adjustedContribution = baseContribution * Math.pow(1 + growthRate, year)
    
    // Apply contribution limits for retirement accounts
    if (component.accountType) {
      const limits = this.getContributionLimits(component.accountType, year)
      return Math.min(adjustedContribution, limits.annual)
    }
    
    return adjustedContribution
  }
  
  private async calculatePortfolioReturns(
    component: InvestmentComponent,
    year: number
  ): Promise<PortfolioReturns> {
    
    let totalReturn = 0
    const assetReturns = new Map<AssetClass, number>()
    
    for (const allocation of component.assetAllocation) {
      const assetReturn = await this.getAssetClassReturn(allocation.assetClass, year)
      const weightedReturn = assetReturn * (allocation.targetPercentage / 100)
      totalReturn += weightedReturn
      assetReturns.set(allocation.assetClass, assetReturn)
    }
    
    // Apply volatility for Monte Carlo simulations (if enabled)
    if (component.volatility) {
      const volatilityAdjustment = this.generateVolatilityAdjustment(component.volatility)
      totalReturn += volatilityAdjustment
    }
    
    const portfolioValue = this.calculateCompoundGrowth(
      component.initialAmount,
      component.monthlyContribution,
      totalReturn,
      year
    )
    
    return {
      portfolioValue,
      grossReturn: totalReturn,
      netReturn: totalReturn,
      assetAllocation: assetReturns,
      dividendIncome: this.calculateDividendIncome(component, portfolioValue)
    }
  }
  
  private async calculateInvestmentTax(
    component: InvestmentComponent,
    scenario: Scenario,
    year: number,
    returns: PortfolioReturns,
    fees: FeeCalculation
  ): Promise<InvestmentTaxCalculation> {
    
    switch (scenario.country) {
      case 'AUSTRALIA':
        return this.calculateAustralianInvestmentTax(component, returns, fees)
      case 'USA':
        return this.calculateUSInvestmentTax(component, returns, fees)
      case 'UK':
        return this.calculateUKInvestmentTax(component, returns, fees)
      default:
        throw new Error(`Investment tax calculations not implemented for ${scenario.country}`)
    }
  }
  
  private calculateAustralianInvestmentTax(
    component: InvestmentComponent,
    returns: PortfolioReturns,
    fees: FeeCalculation
  ): InvestmentTaxCalculation {
    
    const marginalTaxRate = 0.325 // Should come from user's total income
    let totalTax = 0
    const optimizations: TaxOptimization[] = []
    
    // Dividend tax calculation
    let dividendTax = 0
    if (returns.dividendIncome > 0) {
      if (component.frankedDividends) {
        // Franking credits calculation
        const frankingCredits = returns.dividendIncome * 0.3 / 0.7 // 30% company tax rate
        const grossDividend = returns.dividendIncome + frankingCredits
        dividendTax = Math.max(0, grossDividend * marginalTaxRate - frankingCredits)
        
        optimizations.push({
          type: 'FRANKING_CREDITS',
          description: 'Franking credits reduce your tax liability',
          benefit: frankingCredits * marginalTaxRate,
          action: 'Consider Australian shares with high franking ratios'
        })
      } else {
        dividendTax = returns.dividendIncome * marginalTaxRate
      }
    }
    
    // Capital gains tax (for taxable accounts)
    let capitalGainsTax = 0
    if (component.taxTreatment === 'POST_TAX') {
      const capitalGains = Math.max(0, returns.netReturn - returns.dividendIncome)
      if (capitalGains > 0) {
        // 50% CGT discount for assets held >12 months
        const discountedGains = capitalGains * 0.5
        capitalGainsTax = discountedGains * marginalTaxRate
        
        optimizations.push({
          type: 'CGT_DISCOUNT',
          description: '50% CGT discount for long-term holdings',
          benefit: capitalGains * 0.5 * marginalTaxRate,
          action: 'Hold investments for >12 months to qualify for CGT discount'
        })
      }
    }
    
    totalTax = dividendTax + capitalGainsTax
    
    return {
      totalTax,
      dividendTax,
      capitalGainsTax,
      optimizations
    }
  }
  
  private calculateUSInvestmentTax(
    component: InvestmentComponent,
    returns: PortfolioReturns,
    fees: FeeCalculation
  ): InvestmentTaxCalculation {
    
    let totalTax = 0
    const optimizations: TaxOptimization[] = []
    
    // Tax-loss harvesting for taxable accounts
    if (component.taxLossHarvesting && component.taxTreatment === 'POST_TAX') {
      const potentialLosses = this.identifyTaxLossOpportunities(component)
      if (potentialLosses > 0) {
        const taxSavings = Math.min(potentialLosses, 3000) * 0.22 // $3k limit, 22% tax rate
        totalTax -= taxSavings
        
        optimizations.push({
          type: 'TAX_LOSS_HARVESTING',
          description: 'Realize losses to offset gains',
          benefit: taxSavings,
          action: 'Consider harvesting losses from underperforming positions'
        })
      }
    }
    
    // Retirement account contributions reduce current tax
    if (component.taxTreatment === 'PRE_TAX') {
      const contributionDeduction = component.monthlyContribution * 12
      const taxSavings = contributionDeduction * 0.22
      totalTax -= taxSavings
      
      optimizations.push({
        type: 'RETIREMENT_CONTRIBUTION',
        description: 'Reduce current tax through retirement contributions',
        benefit: taxSavings,
        action: `Maximize ${component.accountType} contributions`
      })
    }
    
    return {
      totalTax,
      dividendTax: 0, // Simplified
      capitalGainsTax: 0, // Simplified
      optimizations
    }
  }
  
  calculatePortfolioOptimization(
    components: InvestmentComponent[],
    goals: InvestmentGoals
  ): PortfolioOptimization {
    
    // Modern Portfolio Theory optimization
    const correlationMatrix = this.buildCorrelationMatrix(components)
    const expectedReturns = this.calculateExpectedReturns(components)
    const riskLevels = this.calculateRiskLevels(components)
    
    // Optimize based on risk tolerance and goals
    const efficientFrontier = this.calculateEfficientFrontier(
      expectedReturns,
      correlationMatrix,
      riskLevels
    )
    
    const optimalAllocation = this.findOptimalAllocation(
      efficientFrontier,
      goals.riskTolerance,
      goals.timeHorizon
    )
    
    return {
      currentAllocation: this.getCurrentAllocation(components),
      recommendedAllocation: optimalAllocation,
      expectedReturn: this.calculatePortfolioReturn(optimalAllocation, expectedReturns),
      expectedRisk: this.calculatePortfolioRisk(optimalAllocation, correlationMatrix),
      rebalancingRequired: this.calculateRebalancingNeeds(components, optimalAllocation)
    }
  }
}

interface PortfolioReturns {
  portfolioValue: number
  grossReturn: number
  netReturn: number
  assetAllocation: Map<AssetClass, number>
  dividendIncome: number
}

interface InvestmentTaxCalculation {
  totalTax: number
  dividendTax: number
  capitalGainsTax: number
  optimizations: TaxOptimization[]
}

interface TaxOptimization {
  type: string
  description: string
  benefit: number
  action: string
}

interface InvestmentGoals {
  riskTolerance: 'conservative' | 'moderate' | 'aggressive'
  timeHorizon: number // years
  targetAmount: number
  liquidityNeeds: number
}

interface PortfolioOptimization {
  currentAllocation: Map<AssetClass, number>
  recommendedAllocation: Map<AssetClass, number>
  expectedReturn: number
  expectedRisk: number
  rebalancingRequired: RebalancingAction[]
}
```

## Multi-Country Requirements

### Australia (ATO Compliance)
- **Superannuation**: Concessional/non-concessional contribution limits and tax treatment
- **Franking Credits**: Dividend imputation calculations for Australian shares
- **CGT Discount**: 50% discount for assets held >12 months
- **SMSF Integration**: Self-managed superannuation fund capabilities
- **Negative Gearing**: Investment property within portfolio context

### USA (IRS Compliance)
- **401k/IRA Optimization**: Contribution limits, employer matching, Roth conversions
- **Tax-Loss Harvesting**: Systematic loss realization with wash sale rule compliance
- **HSA Integration**: Triple tax advantage for health savings accounts
- **Backdoor Roth**: High-income Roth IRA access strategies
- **Asset Location**: Tax-efficient placement of assets across account types

### UK (HMRC Compliance)
- **ISA Optimization**: Annual allowance maximization across different ISA types
- **Pension Planning**: Workplace pension, SIPP, and annual allowance management
- **Dividend Allowance**: £2,000 tax-free dividend income
- **Capital Gains Allowance**: Annual exemption optimization
- **Bed and Breakfast Rules**: UK equivalent of wash sale rules

## User Interface Requirements

### Adaptive Complexity

**ELI5 Mode**:
- Simple "Set and Forget" portfolio templates
- Visual risk tolerance questionnaire
- Progress tracking with clear milestones
- Basic performance charts with plain English explanations

**Intermediate Mode**:
- Asset allocation pie charts with drag-and-drop adjustment
- Goal-based investing interface
- Tax optimization suggestions
- Performance comparison against benchmarks

**Expert Mode**:
- Full portfolio optimization tools
- Custom asset allocation with correlation matrices
- Advanced tax strategies and harvesting
- Monte Carlo simulation controls
- Detailed performance analytics

### Components

```tsx
// Portfolio Overview Dashboard
export function PortfolioOverview({ scenario }: { scenario: Scenario }) {
  const { data: portfolioData } = useQuery(getPortfolioSummary, { scenarioId: scenario.id })
  
  return (
    <div className="space-y-6">
      <PortfolioValueChart data={portfolioData} />
      <AssetAllocationChart allocations={portfolioData.assetAllocation} />
      <PerformanceMetrics metrics={portfolioData.performance} />
      <TaxOptimizationPanel optimizations={portfolioData.taxOptimizations} />
    </div>
  )
}

// Investment Component Builder
export function InvestmentBuilder({ scenario }: { scenario: Scenario }) {
  const [selectedType, setSelectedType] = useState<InvestmentType>('TAXABLE_BROKERAGE')
  const { user } = useAuth()
  
  return (
    <div className="space-y-6">
      <InvestmentTypeSelector 
        selectedType={selectedType}
        onTypeChange={setSelectedType}
        country={scenario.country}
        complexityLevel={user.complexityLevel}
      />
      
      <InvestmentConfigurationForm 
        type={selectedType}
        scenario={scenario}
      />
      
      <AssetAllocationBuilder 
        investmentType={selectedType}
        country={scenario.country}
      />
    </div>
  )
}

// Portfolio Optimization Tool
export function PortfolioOptimizer({ investments }: { investments: InvestmentComponent[] }) {
  const [optimizationGoals, setOptimizationGoals] = useState<InvestmentGoals>()
  
  const { data: optimization } = useQuery(
    optimizePortfolio,
    { investments, goals: optimizationGoals }
  )
  
  return (
    <div className="grid gap-6 lg:grid-cols-2">
      <Card>
        <CardHeader>
          <CardTitle>Current Allocation</CardTitle>
        </CardHeader>
        <CardContent>
          <AllocationChart allocations={optimization?.currentAllocation} />
        </CardContent>
      </Card>
      
      <Card>
        <CardHeader>
          <CardTitle>Recommended Allocation</CardTitle>
        </CardHeader>
        <CardContent>
          <AllocationChart allocations={optimization?.recommendedAllocation} />
          <OptimizationSuggestions suggestions={optimization?.rebalancingRequired} />
        </CardContent>
      </Card>
    </div>
  )
}
```

## Implementation Details

### File Structure
```
src/
├── investment/
│   ├── client/
│   │   ├── PortfolioOverview.tsx
│   │   ├── InvestmentBuilder.tsx
│   │   ├── PortfolioOptimizer.tsx
│   │   ├── components/
│   │   │   ├── AssetAllocationChart.tsx
│   │   │   ├── PerformanceChart.tsx
│   │   │   ├── TaxOptimizationPanel.tsx
│   │   │   └── RebalancingAlert.tsx
│   │   └── hooks/
│   │       ├── usePortfolioCalculations.ts
│   │       ├── useAssetAllocation.ts
│   │       └── useOptimization.ts
│   ├── server/
│   │   ├── queries.ts
│   │   ├── actions.ts
│   │   ├── calculations/
│   │   │   ├── InvestmentCalculator.ts
│   │   │   ├── PortfolioOptimizer.ts
│   │   │   └── RiskCalculator.ts
│   │   ├── services/
│   │   │   ├── MarketDataService.ts
│   │   │   └── AssetClassService.ts
│   │   └── country-specific/
│   │       ├── AustralianInvestments.ts
│   │       ├── USAInvestments.ts
│   │       └── UKInvestments.ts
│   └── shared/
│       ├── types.ts
│       └── constants.ts
```

### Market Data Integration

```typescript
export interface MarketDataService {
  getAssetClassReturns(assetClass: AssetClass, period: number): Promise<number[]>
  getCurrentPrices(tickers: string[]): Promise<Map<string, number>>
  getHistoricalData(ticker: string, startDate: Date, endDate: Date): Promise<PriceData[]>
  getCorrelationMatrix(assetClasses: AssetClass[]): Promise<number[][]>
}

export class MockMarketDataService implements MarketDataService {
  // Mock implementation for MVP with realistic historical returns
  async getAssetClassReturns(assetClass: AssetClass, period: number): Promise<number[]> {
    const historicalReturns = {
      DOMESTIC_EQUITY: 0.10,
      INTERNATIONAL_EQUITY: 0.08,
      BONDS_GOVERNMENT: 0.03,
      REAL_ESTATE: 0.07,
      CASH: 0.02
    }
    
    return Array(period).fill(historicalReturns[assetClass] || 0.07)
  }
}
```

## Testing Requirements

### Financial Accuracy Testing
- Portfolio value calculations with compound interest accuracy
- Asset allocation rebalancing algorithms
- Tax optimization strategies validation
- Risk-return optimization verification

### Performance Testing
- Portfolio calculations for 1000+ holdings under 2 seconds
- Real-time asset allocation updates
- Monte Carlo simulations with 10,000 iterations
- Optimization algorithms for complex portfolios

## Success Criteria

### Acceptance Criteria
- [ ] Users can build diversified portfolios across all supported countries
- [ ] All investment tax calculations are country-compliant
- [ ] Portfolio optimization suggests appropriate asset allocations
- [ ] Rebalancing alerts and recommendations work correctly
- [ ] Tax optimization strategies provide measurable benefits
- [ ] Performance tracking and benchmarking are accurate

### Performance Requirements
- Portfolio calculations complete within 2 seconds for 30-year projections
- Real-time allocation updates with <100ms latency
- Monte Carlo simulations complete within 10 seconds
- Optimization algorithms handle 20+ asset classes efficiently

### Compliance Requirements
- All retirement account limits verified against current regulations
- Tax calculations validated by qualified professionals
- Asset allocation recommendations meet fiduciary standards
- Performance calculations match industry standard methodologies