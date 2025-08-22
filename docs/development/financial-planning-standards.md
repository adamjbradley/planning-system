# Financial Planning Development Standards

This document outlines the development standards, patterns, and requirements specific to building the FinPath Explorer financial planning platform.

## Core Principles

### Modified Vertical Slice Implementation
We use a modified vertical slice implementation approach for LLM-assisted coding:
1. Start with basic implementations of features first
2. Add complexity progressively (ELI5 → Intermediate → Expert)  
3. Leverage Wasp's batteries-included framework for heavy lifting
4. Build end-to-end functionality quickly, then enhance

### Financial Calculation Requirements

#### Tax Compliance Standards
- **ALL tax calculations MUST be ATO/IRS/HMRC compliant**
- Use precise decimal arithmetic (never floating point for currency)
- Validate all calculations against professional tax software
- Include regulatory change tracking and version control
- Document assumptions and calculation methodologies

#### Calculation Accuracy Requirements
- Financial projections must have <0.01% variance from expected results
- Cache expensive calculations with proper invalidation
- Implement comprehensive testing for edge cases
- Use real-world scenarios for validation testing

## Multi-Country Architecture Patterns

### Tax Engine Pattern
```typescript
// ✅ Correct: Use factory pattern for country-specific tax engines
interface ITaxEngine {
  calculateIncomeTax(income: number, deductions: number): TaxCalculation
  calculateCapitalGains(gain: number, holdingPeriod: number): TaxCalculation
  suggestOptimizations(scenario: Scenario): TaxOptimization[]
}

export class TaxEngineFactory {
  static create(country: Country): ITaxEngine {
    switch (country) {
      case 'AUSTRALIA': return new AustralianTaxEngine()
      case 'USA': return new USATaxEngine()
      case 'UK': return new UKTaxEngine()
      default: throw new Error(`Tax engine not implemented for ${country}`)
    }
  }
}

// ❌ Incorrect: Hardcoded country logic in generic functions
function calculateTax(income: number, country: string) {
  if (country === 'AU') {
    // Australian tax logic...
  } else if (country === 'US') {
    // US tax logic...
  }
}
```

### Currency Handling
```typescript
// ✅ Correct: Use Decimal for all financial calculations
import { Decimal } from 'decimal.js'

const result = new Decimal(income).mul(taxRate).toNumber()

// ❌ Incorrect: Never use floating point for currency
const result = income * 0.325 // Precision errors!
```

### Country-Specific Components
```typescript
// ✅ Correct: Country-agnostic interface with localized implementations
interface HousingStrategy {
  calculateTaxBenefits(component: HousingComponent): TaxBenefits
}

class AustralianHousingStrategy implements HousingStrategy {
  calculateTaxBenefits(component: HousingComponent): TaxBenefits {
    // Negative gearing, CGT discount, depreciation
  }
}

// ❌ Incorrect: Mixed country logic in single component
function calculateHousingTax(component: HousingComponent) {
  if (component.country === 'AU') {
    // Australian negative gearing...
  } else if (component.country === 'US') {
    // US depreciation...
  }
}
```

## Progressive Complexity Patterns

### Adaptive UI Components
```typescript
// ✅ Correct: Progressive disclosure based on complexity level
interface ComplexityAwareProps {
  complexityLevel: ComplexityLevel
  children: React.ReactNode
}

export function ProgressiveDisclosure({ complexityLevel, children }: ComplexityAwareProps) {
  if (complexityLevel === 'ELI5') {
    return <SimpleView>{children}</SimpleView>
  } else if (complexityLevel === 'INTERMEDIATE') {
    return <IntermediateView>{children}</IntermediateView>
  } else {
    return <ExpertView>{children}</ExpertView>
  }
}

// ❌ Incorrect: Showing all complexity at once
function FinancialForm() {
  return (
    <form>
      <BasicInputs />
      <AdvancedInputs />  {/* Always visible */}
      <ExpertInputs />    {/* Always visible */}
    </form>
  )
}
```

### Feature Progression
```typescript
// ✅ Correct: Unlock features based on user progression
export function useFeatureAccess(user: User) {
  const canAccessIntermediateFeatures = user.complexityLevel !== 'ELI5'
  const canAccessExpertFeatures = user.complexityLevel === 'EXPERT'
  
  return {
    canAccessAdvancedTaxOptimizations: canAccessIntermediateFeatures,
    canAccessMonteCarloSimulations: canAccessExpertFeatures,
    canAccessCrossBorderPlanning: canAccessExpertFeatures
  }
}
```

## Scenario Management Patterns

### Scenario Structure
```typescript
// ✅ Correct: Composable scenario components
interface Scenario {
  id: string
  country: Country
  housingComponents: HousingComponent[]
  investmentComponents: InvestmentComponent[]
  incomeComponents: IncomeComponent[]
  expenseComponents: ExpenseComponent[]
}

// Each component is self-contained and country-aware
interface HousingComponent {
  type: HousingType
  taxOptimizations: CountrySpecificTaxOptimizations
  projections: YearlyProjection[]
}
```

### Template Application
```typescript
// ✅ Correct: Template system with customization
export class TemplateEngine {
  async applyTemplate(templateId: string, customizations: TemplateCustomizations): Promise<Scenario> {
    const template = await this.getTemplate(templateId)
    const customizedScenario = this.customizeTemplate(template, customizations)
    return this.validateScenario(customizedScenario)
  }
}
```

## Testing Requirements for Financial Features

### Calculation Testing
```typescript
// ✅ Required: Comprehensive financial calculation tests
describe('Australian Tax Engine', () => {
  it('should calculate negative gearing benefits correctly', () => {
    const scenario = createTestScenario({
      rentalIncome: 26000,
      interestExpense: 30000,
      marginalTaxRate: 0.325
    })
    
    const result = australianTaxEngine.calculateNegativeGearing(scenario)
    
    expect(result.negativeGearingBenefit).toBeCloseTo(1300, 2) // $4000 * 32.5%
  })
  
  it('should apply CGT discount for assets held >12 months', () => {
    const capitalGain = new Decimal(100000)
    const holdingPeriod = 18 // months
    
    const result = australianTaxEngine.calculateCapitalGains(capitalGain, holdingPeriod)
    
    expect(result.discountApplied).toEqual(new Decimal(50000)) // 50% discount
  })
})
```

### Multi-Country Validation
```typescript
// ✅ Required: Cross-country scenario testing
describe('Cross-Country Scenarios', () => {
  it('should handle dual tax residency correctly', () => {
    const scenario = createDualResidencyScenario('AUSTRALIA', 'USA')
    const result = calculateCrossBorderTax(scenario)
    
    expect(result.totalTaxLiability).toBeLessThan(
      result.australianTax + result.usaTax
    ) // Treaty benefits applied
  })
})
```

## File Organization for Financial Features

### Feature Structure
```
src/
├── setup/              # Multi-country setup and user profile
├── scenario/           # Core scenario management
├── template/           # Strategy templates
├── housing/           # Housing strategies
├── investment/        # Investment portfolio
├── tax/              # Tax calculation engines
│   ├── engines/      # Country-specific implementations
│   ├── calculations/ # Shared tax utilities
│   └── types.ts      # Tax calculation interfaces
├── analytics/        # Advanced analytics and simulations
└── shared/
    ├── types/        # Financial calculation types
    ├── utils/        # Currency and calculation utilities
    └── constants/    # Tax rates, limits, etc.
```

### Naming Conventions
- Use `Calculator` suffix for calculation engines
- Use `Engine` suffix for tax processing
- Use `Component` suffix for scenario building blocks
- Use `Template` suffix for strategy templates
- Use `Projection` suffix for time-based calculations

## Error Handling for Financial Operations

### Tax Calculation Errors
```typescript
// ✅ Correct: Specific error types for financial operations
export class TaxCalculationError extends Error {
  constructor(
    message: string,
    public readonly country: Country,
    public readonly calculationType: string,
    public readonly inputData: any
  ) {
    super(message)
    this.name = 'TaxCalculationError'
  }
}

// Usage
if (income < 0) {
  throw new TaxCalculationError(
    'Income cannot be negative',
    'AUSTRALIA',
    'income_tax',
    { income }
  )
}
```

### Validation Requirements
- Validate all user inputs before calculations
- Check for negative values where inappropriate
- Verify contribution limits against current regulations
- Validate dates and time periods
- Check currency consistency across components

## Performance Requirements

### Calculation Performance
- Scenario calculations must complete within 2 seconds for 30-year projections
- Real-time parameter updates with <100ms latency
- Cache calculation results with proper invalidation
- Use web workers for complex Monte Carlo simulations

### Data Loading
- Lazy load country-specific tax engines
- Progressive loading for complex scenarios
- Optimize database queries for scenario retrieval
- Implement proper loading states for all calculations

## Multi-Country Tax Compliance

### Australia (ATO) Compliance
```typescript
// ✅ ATO Compliant: 2024-25 tax brackets
const AUSTRALIAN_TAX_BRACKETS_2024_25 = [
  { min: 0, max: 18200, rate: 0.00, description: 'Tax-free threshold' },
  { min: 18201, max: 45000, rate: 0.19, description: '19% rate' },
  { min: 45001, max: 120000, rate: 0.325, description: '32.5% rate' },
  { min: 120001, max: 180000, rate: 0.37, description: '37% rate' },
  { min: 180001, max: Infinity, rate: 0.45, description: '45% rate' }
]

// Superannuation contribution caps 2024-25
const SUPER_CONTRIBUTION_CAPS_2024_25 = {
  concessional: 27500,        // Before-tax contributions
  nonConcessional: 110000,    // After-tax contributions
  totalSuperBalance: 1900000, // Total super balance cap
  preservationAge: 60         // Current preservation age
}
```

### USA (IRS) Compliance
```typescript
// ✅ IRS Compliant: 2024 retirement limits
const US_RETIREMENT_LIMITS_2024 = {
  traditional401k: 23000,        // Regular contribution limit
  traditional401kCatchUp: 7500,  // Age 50+ catch-up
  traditionalIRA: 7000,          // Regular contribution limit
  traditionalIRACatchUp: 1000,   // Age 50+ catch-up
  rothIRA: 7000,                 // Same as traditional IRA
  hsaContribution: 4300,         // Individual coverage
  hsaContributionFamily: 8550    // Family coverage
}
```

### UK (HMRC) Compliance
```typescript
// ✅ HMRC Compliant: 2024-25 allowances
const UK_ISA_ALLOWANCES_2024_25 = {
  annualLimit: 20000,           // Total ISA allowance
  lifetimeISA: 4000,           // Maximum LISA contribution
  juniorISA: 9000,             // Junior ISA allowance
  lifetimeISABonus: 0.25       // 25% government bonus on LISA
}

const UK_PENSION_ALLOWANCES_2024_25 = {
  annualAllowance: 60000,       // Maximum annual contribution
  lifetimeAllowance: 1073100,   // Lifetime allowance (frozen)
  taperThreshold: 260000,       // Annual allowance taper threshold
  taperRate: 0.5,               // £1 reduction for every £2 over threshold
  minimumAllowance: 10000       // Minimum tapered allowance
}
```

## Calculation Accuracy Standards

### Precision Standards
```typescript
// Configure decimal precision globally
import { Decimal } from 'decimal.js'

Decimal.set({
  precision: 10,           // 10 significant digits
  rounding: Decimal.ROUND_HALF_EVEN, // Banker's rounding
  toExpNeg: -9,           // Use exponential notation for small numbers
  toExpPos: 21,           // Use exponential notation for large numbers
  modulo: Decimal.ROUND_DOWN
})

// ✅ Correct: All currency operations
const taxAmount = new Decimal(income).mul(taxRate).toDecimalPlaces(2)
const netIncome = new Decimal(grossIncome).minus(taxAmount)

// ❌ Incorrect: Never use floating point for currency
const taxAmount = income * 0.325 // Precision errors!
```

### Currency Type with Validation
```typescript
class CurrencyAmount {
  constructor(
    private amount: Decimal,
    private currency: Currency
  ) {
    this.validateAmount()
  }
  
  private validateAmount(): void {
    if (this.amount.isNaN() || !this.amount.isFinite()) {
      throw new CalculationError('Invalid currency amount')
    }
  }
  
  add(other: CurrencyAmount): CurrencyAmount {
    this.validateSameCurrency(other)
    return new CurrencyAmount(
      this.amount.plus(other.amount),
      this.currency
    )
  }
  
  toDisplayString(): string {
    return `${this.currency.symbol}${this.amount.toFixed(this.currency.decimals)}`
  }
}
```

## Professional Validation Requirements

### Required Professional Review
All tax calculations must be reviewed by qualified professionals:
- **Australia**: Certified Practicing Accountant (CPA) or Chartered Accountant (CA)
- **USA**: Certified Public Accountant (CPA) or Enrolled Agent (EA)
- **UK**: Chartered Accountant (ACA/ACCA) or Chartered Tax Adviser (CTA)

### Documentation Requirements
- Include source references for all tax rates and limits
- Document calculation methodologies and assumptions
- Maintain audit trail for all regulatory changes
- Provide disclaimers about professional advice vs. educational content

### Update Cycle Requirements
- Review tax rates and limits annually before start of tax year
- Monitor for mid-year changes and update within 30 days
- Validate all changes with qualified professionals
- Update user-facing content to reflect changes

## Quality Assurance Checklist

### Pre-Deployment Testing
- [ ] All unit tests pass with 100% coverage on calculation functions
- [ ] Integration tests validate complete user scenarios
- [ ] Performance tests meet sub-2-second requirements
- [ ] Memory usage tests show no leaks
- [ ] Property-based tests pass 1000+ random cases
- [ ] External validation against official calculators
- [ ] Professional review by qualified accountant/CPA
- [ ] Cross-browser testing on calculation accuracy
- [ ] Mobile device performance testing

### Ongoing Monitoring
- [ ] Daily automated testing against official tax calculators
- [ ] Weekly performance benchmarking
- [ ] Monthly accuracy audits with sample calculations
- [ ] Quarterly professional review of all tax calculations
- [ ] Annual update cycle for tax rates and regulations