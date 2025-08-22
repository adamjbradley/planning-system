# Scenario Management

## Feature Overview

**Purpose**: Core system for creating, saving, managing, and comparing financial scenarios across multiple time horizons and countries.

**User Stories**:
- As an AU user, I want to create scenarios comparing "buy vs rent" with negative gearing implications
- As a US user, I want to model different 401k contribution strategies across various income projections  
- As a UK user, I want to compare ISA vs pension optimization scenarios for retirement planning
- As an expat, I want to model cross-border scenarios with dual tax residency considerations

**Priority**: P0 (Critical) - Core platform functionality

**Dependencies**: Multi-Country Setup, Tax Engine Integration

## Technical Specifications

### Data Models

```prisma
model Scenario {
  id                  String   @id @default(cuid())
  name                String
  description         String?
  userId              String
  user                User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Core Configuration
  country             Country
  currency            Currency
  timeHorizonYears    Int      @default(30)
  
  // User Profile Snapshot
  currentAge          Int
  currentIncome       Decimal
  currentSavings      Decimal
  currentDebt         Decimal
  
  // Strategy Components
  housingComponents     HousingComponent[]
  investmentComponents  InvestmentComponent[]
  incomeComponents      IncomeComponent[]
  expenseComponents     ExpenseComponent[]
  
  // Scenario Management
  isTemplate          Boolean  @default(false)
  templateType        TemplateType?
  parentScenarioId    String?
  parentScenario      Scenario? @relation("ScenarioVariations", fields: [parentScenarioId], references: [id])
  variations          Scenario[] @relation("ScenarioVariations")
  
  // Results Cache
  calculationResults  Json?
  lastCalculated      DateTime?
  
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
  
  @@map("scenarios")
}

model HousingComponent {
  id                  String   @id @default(cuid())
  scenarioId          String
  scenario            Scenario @relation(fields: [scenarioId], references: [id], onDelete: Cascade)
  
  type                HousingType // BUY_PRIMARY, BUY_INVESTMENT, RENT
  name                String
  startYear           Int      @default(0)
  
  // Purchase Details
  purchasePrice       Decimal?
  deposit             Decimal?
  mortgageAmount      Decimal?
  mortgageRate        Decimal?
  mortgageTerm        Int?
  
  // Rental Details  
  rentAmount          Decimal?
  rentGrowthRate      Decimal? @default(0.03)
  
  // Investment Property Specifics
  rentalIncome        Decimal?
  propertyGrowthRate  Decimal? @default(0.05)
  maintenanceCost     Decimal?
  managementFees      Decimal?
  
  // Tax Optimizations
  enableNegativeGearing Boolean @default(false) // AU
  depreciation        Decimal? // US/AU
  
  @@map("housing_components")
}

model InvestmentComponent {
  id                  String   @id @default(cuid())
  scenarioId          String
  scenario            Scenario @relation(fields: [scenarioId], references: [id], onDelete: Cascade)
  
  type                InvestmentType // ETF, STOCK, RETIREMENT_ACCOUNT, PROPERTY_INVESTMENT
  name                String
  startYear           Int      @default(0)
  
  // Core Investment Details
  initialAmount       Decimal  @default(0)
  monthlyContribution Decimal  @default(0)
  expectedReturn      Decimal  @default(0.07)
  managementFees      Decimal  @default(0.001)
  
  // Country-Specific Accounts
  accountType         RetirementAccountType? // 401K, IRA, SUPER, ISA, SIPP
  employerMatch       Decimal? // US 401k matching
  contributionLimit   Decimal? // Annual limits
  
  // Tax Treatment
  taxTreatment        TaxTreatment // PRE_TAX, POST_TAX, TAX_FREE
  frankedDividends    Boolean @default(false) // AU
  
  @@map("investment_components")
}

enum HousingType {
  BUY_PRIMARY
  BUY_INVESTMENT  
  RENT
}

enum InvestmentType {
  ETF
  STOCK
  RETIREMENT_ACCOUNT
  PROPERTY_INVESTMENT
  CASH_SAVINGS
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
  HSA
  
  // UK
  WORKPLACE_PENSION
  PERSONAL_PENSION
  SIPP
  STOCKS_ISA
  CASH_ISA
  LISA
}

enum TaxTreatment {
  PRE_TAX      // Contributions reduce taxable income
  POST_TAX     // Contributions from after-tax income  
  TAX_FREE     // Growth and withdrawals tax-free
}

enum TemplateType {
  // Universal Templates
  AGGRESSIVE_GROWTH
  BALANCED_GROWTH
  CONSERVATIVE
  FIRST_HOME_BUYER
  
  // Australia Specific
  NEGATIVE_GEARING
  SUPER_BOOST
  FRANKING_CREDITS
  
  // USA Specific  
  ROTH_LADDER
  TAX_LOSS_HARVEST
  BACKDOOR_ROTH
  
  // UK Specific
  ISA_MAXING
  PENSION_TAPERING
  BUY_TO_LET
}
```

### API Operations

```typescript
// Queries
export const getScenarios: Query<Scenario[], void> = async (args, context) => {
  return context.entities.Scenario.findMany({
    where: { userId: context.user.id },
    include: {
      housingComponents: true,
      investmentComponents: true,
      variations: true
    },
    orderBy: { updatedAt: 'desc' }
  })
}

export const getScenario: Query<Scenario | null, Pick<Scenario, 'id'>> = async ({ id }, context) => {
  const scenario = await context.entities.Scenario.findFirst({
    where: { id, userId: context.user.id },
    include: {
      housingComponents: true,
      investmentComponents: true,
      incomeComponents: true,
      expenseComponents: true,
      variations: true,
      parentScenario: true
    }
  })
  
  return scenario
}

export const getTemplateScenarios: Query<Scenario[], { country: Country }> = async ({ country }, context) => {
  return context.entities.Scenario.findMany({
    where: { 
      isTemplate: true,
      country: country
    },
    include: {
      housingComponents: true,
      investmentComponents: true
    }
  })
}

// Actions  
export const createScenario: Action<Scenario, CreateScenarioArgs> = async (args, context) => {
  const { country, name, description, fromTemplate } = args
  
  let scenarioData = {
    name,
    description,
    userId: context.user.id,
    country,
    currency: getCurrencyForCountry(country),
    currentAge: context.user.age || 25,
    currentIncome: 0,
    currentSavings: 0,
    currentDebt: 0
  }
  
  if (fromTemplate) {
    const template = await context.entities.Scenario.findFirst({
      where: { id: fromTemplate, isTemplate: true },
      include: {
        housingComponents: true,
        investmentComponents: true
      }
    })
    
    if (template) {
      scenarioData = {
        ...scenarioData,
        parentScenarioId: template.id,
        housingComponents: {
          create: template.housingComponents.map(component => ({
            ...component,
            id: undefined,
            scenarioId: undefined
          }))
        },
        investmentComponents: {
          create: template.investmentComponents.map(component => ({
            ...component, 
            id: undefined,
            scenarioId: undefined
          }))
        }
      }
    }
  }
  
  const scenario = await context.entities.Scenario.create({
    data: scenarioData,
    include: {
      housingComponents: true,
      investmentComponents: true
    }
  })
  
  return scenario
}

export const updateScenario: Action<Scenario, UpdateScenarioArgs> = async (args, context) => {
  const { id, ...updateData } = args
  
  const scenario = await context.entities.Scenario.update({
    where: { id, userId: context.user.id },
    data: {
      ...updateData,
      lastCalculated: null, // Invalidate cache
      calculationResults: null
    },
    include: {
      housingComponents: true,
      investmentComponents: true
    }
  })
  
  return scenario
}

export const calculateScenario: Action<ScenarioResults, { scenarioId: string }> = async ({ scenarioId }, context) => {
  const scenario = await context.entities.Scenario.findFirst({
    where: { id: scenarioId, userId: context.user.id },
    include: {
      housingComponents: true,
      investmentComponents: true,
      incomeComponents: true,
      expenseComponents: true
    }
  })
  
  if (!scenario) throw new HttpError(404, 'Scenario not found')
  
  const taxEngine = TaxEngineFactory.create(scenario.country)
  const calculator = new ScenarioCalculator(taxEngine)
  
  const results = await calculator.calculate(scenario)
  
  // Cache results
  await context.entities.Scenario.update({
    where: { id: scenarioId },
    data: {
      calculationResults: results,
      lastCalculated: new Date()
    }
  })
  
  return results
}

export const duplicateScenario: Action<Scenario, { scenarioId: string; name: string }> = async (args, context) => {
  const { scenarioId, name } = args
  
  const original = await context.entities.Scenario.findFirst({
    where: { id: scenarioId, userId: context.user.id },
    include: {
      housingComponents: true,
      investmentComponents: true,
      incomeComponents: true,
      expenseComponents: true
    }
  })
  
  if (!original) throw new HttpError(404, 'Scenario not found')
  
  const duplicate = await context.entities.Scenario.create({
    data: {
      ...original,
      id: undefined,
      name,
      parentScenarioId: original.id,
      createdAt: undefined,
      updatedAt: undefined,
      calculationResults: null,
      lastCalculated: null,
      housingComponents: {
        create: original.housingComponents.map(comp => ({
          ...comp,
          id: undefined,
          scenarioId: undefined
        }))
      },
      investmentComponents: {
        create: original.investmentComponents.map(comp => ({
          ...comp,
          id: undefined, 
          scenarioId: undefined
        }))
      }
    },
    include: {
      housingComponents: true,
      investmentComponents: true
    }
  })
  
  return duplicate
}
```

### Tax Engine Integration

```typescript
// Scenario Calculator with Tax Engine
export class ScenarioCalculator {
  constructor(private taxEngine: ITaxEngine) {}
  
  async calculate(scenario: Scenario): Promise<ScenarioResults> {
    const projections: YearlyProjection[] = []
    
    for (let year = 0; year <= scenario.timeHorizonYears; year++) {
      const projection = await this.calculateYear(scenario, year)
      projections.push(projection)
    }
    
    return {
      scenarioId: scenario.id,
      projections,
      summary: this.calculateSummary(projections),
      taxOptimizations: await this.getTaxOptimizations(scenario, projections)
    }
  }
  
  private async calculateYear(scenario: Scenario, year: number): Promise<YearlyProjection> {
    // Calculate income
    const totalIncome = this.calculateIncome(scenario, year)
    
    // Calculate housing costs/returns
    const housingResults = await this.calculateHousing(scenario, year)
    
    // Calculate investment returns
    const investmentResults = await this.calculateInvestments(scenario, year)
    
    // Calculate taxes
    const taxCalculation = await this.taxEngine.calculateTax({
      income: totalIncome,
      deductions: housingResults.deductions,
      capitalGains: investmentResults.capitalGains,
      dividends: investmentResults.dividends,
      negativeGearing: housingResults.negativeGearing
    })
    
    return {
      year,
      income: totalIncome,
      expenses: housingResults.expenses + investmentResults.expenses,
      taxPayable: taxCalculation.totalTax,
      netWorth: this.calculateNetWorth(housingResults, investmentResults),
      housing: housingResults,
      investments: investmentResults
    }
  }
  
  private async getTaxOptimizations(scenario: Scenario, projections: YearlyProjection[]): Promise<TaxOptimization[]> {
    return this.taxEngine.suggestOptimizations(scenario, projections)
  }
}
```

## Multi-Country Requirements

### Australia (ATO Compliance)
- **Negative Gearing**: Full calculation of rental property losses against taxable income
- **CGT Discount**: 50% discount for assets held >12 months
- **Franking Credits**: Dividend imputation calculations and refunds  
- **Superannuation**: Concessional/non-concessional contribution limits and tax treatment
- **FHOG Integration**: First home owner grants by state

### USA (IRS Compliance)  
- **401k/IRA Optimization**: Traditional vs Roth contribution strategies
- **Tax-Loss Harvesting**: Wash sale rule compliance and optimization
- **HSA Integration**: Triple tax advantage calculations
- **State Tax Handling**: Multi-state scenarios for relocations
- **Depreciation Schedules**: Real estate depreciation calculations

### UK (HMRC Compliance)
- **ISA Optimization**: Annual allowance maximization across Cash/S&S ISA
- **Pension Relief**: Higher rate tax relief calculations  
- **CGT Annual Exemption**: Capital gains optimization within annual limits
- **Buy-to-Let**: Mortgage interest relief calculations
- **Help to Buy**: Government scheme integration

### Cross-Border Planning
- **Dual Tax Residency**: Tax liability calculations in multiple countries
- **Currency Hedging**: Multi-currency scenario modeling
- **Expat Scenarios**: Tax-efficient repatriation strategies
- **Cross-Border Pensions**: International pension transfer calculations

## User Interface Requirements

### Adaptive Complexity

**ELI5 Mode**:
- Template selection with guided questionnaire
- Simple sliders for basic parameters (income, savings goals)
- Visual scenario comparison with traffic light indicators
- Plain English explanations of tax implications

**Intermediate Mode**:
- Component-based scenario building
- Basic customization of investment allocations
- Side-by-side scenario comparison
- Educational tooltips for tax strategies

**Expert Mode**:
- Full parameter customization
- Advanced tax optimization strategies
- Monte Carlo simulation options
- Detailed tax calculation breakdowns

### Components

```tsx
// Scenario List Component
export function ScenarioList() {
  const { data: scenarios } = useQuery(getScenarios)
  
  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <h2>My Scenarios</h2>
        <CreateScenarioButton />
      </div>
      
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        {scenarios?.map(scenario => (
          <ScenarioCard key={scenario.id} scenario={scenario} />
        ))}
      </div>
    </div>
  )
}

// Scenario Builder Component  
export function ScenarioBuilder({ scenarioId }: { scenarioId?: string }) {
  const [scenario, setScenario] = useState<Scenario | null>(null)
  const [activeTab, setActiveTab] = useState<'profile' | 'housing' | 'investments' | 'results'>('profile')
  
  return (
    <div className="max-w-6xl mx-auto">
      <ScenarioTabs activeTab={activeTab} onTabChange={setActiveTab} />
      
      {activeTab === 'profile' && <ProfileSetup scenario={scenario} />}
      {activeTab === 'housing' && <HousingComponents scenario={scenario} />}  
      {activeTab === 'investments' && <InvestmentComponents scenario={scenario} />}
      {activeTab === 'results' && <ScenarioResults scenario={scenario} />}
    </div>
  )
}

// Housing Components Builder
export function HousingComponents({ scenario }: { scenario: Scenario }) {
  const [components, setComponents] = useState(scenario.housingComponents)
  
  const addComponent = (type: HousingType) => {
    const newComponent = createDefaultHousingComponent(type, scenario.country)
    setComponents([...components, newComponent])
  }
  
  return (
    <div className="space-y-6">
      <div className="flex gap-4">
        <Button onClick={() => addComponent('BUY_PRIMARY')}>
          Add Primary Residence
        </Button>
        <Button onClick={() => addComponent('BUY_INVESTMENT')}>
          Add Investment Property  
        </Button>
        <Button onClick={() => addComponent('RENT')}>
          Add Rental
        </Button>
      </div>
      
      {components.map(component => (
        <HousingComponentEditor 
          key={component.id}
          component={component}
          country={scenario.country}
          onUpdate={(updated) => updateComponent(component.id, updated)}
          onDelete={() => deleteComponent(component.id)}
        />
      ))}
    </div>
  )
}
```

### User Flows

**Creating New Scenario**:
1. Choose country and complexity level
2. Select from template or start blank
3. Set up user profile (age, income, current assets)
4. Add housing strategy components
5. Add investment components  
6. Review and calculate projections
7. Save scenario with meaningful name

**Comparing Scenarios**:
1. Select 2-4 scenarios from list
2. View side-by-side projections
3. Highlight key differences and outcomes
4. Export comparison report

**Template Usage**:
1. Browse country-specific templates
2. Preview template configuration
3. Customize parameters before creating
4. Save as personal scenario for further editing

## Implementation Details

### File Structure
```
src/
├── scenario/
│   ├── client/
│   │   ├── ScenarioList.tsx
│   │   ├── ScenarioBuilder.tsx
│   │   ├── ScenarioComparison.tsx
│   │   ├── components/
│   │   │   ├── HousingComponentEditor.tsx
│   │   │   ├── InvestmentComponentEditor.tsx
│   │   │   └── ScenarioResults.tsx
│   │   └── hooks/
│   │       ├── useScenarioCalculation.ts
│   │       └── useScenarioComparison.ts
│   ├── server/
│   │   ├── queries.ts
│   │   ├── actions.ts
│   │   ├── calculations/
│   │   │   ├── ScenarioCalculator.ts
│   │   │   ├── HousingCalculator.ts
│   │   │   └── InvestmentCalculator.ts
│   │   └── templates/
│   │       ├── AustraliaTemplates.ts
│   │       ├── USATemplates.ts
│   │       └── UKTemplates.ts
│   └── shared/
│       ├── types.ts
│       └── constants.ts
```

### Calculation Engine Requirements

```typescript
// Core calculation interfaces
export interface ScenarioResults {
  scenarioId: string
  projections: YearlyProjection[]
  summary: ScenarioSummary
  taxOptimizations: TaxOptimization[]
}

export interface YearlyProjection {
  year: number
  age: number
  income: number
  expenses: number
  taxPayable: number
  netWorth: number
  housing: HousingResults
  investments: InvestmentResults
}

export interface HousingResults {
  totalValue: number
  totalDebt: number
  equity: number
  cashFlow: number
  deductions: number
  negativeGearing?: number // AU specific
  depreciation?: number    // US/AU specific
}

export interface InvestmentResults {
  totalValue: number
  unrealizedGains: number
  dividendIncome: number
  capitalGains: number
  frankingCredits?: number // AU specific
  expenses: number
}
```

### Testing Requirements

**Financial Accuracy Testing**:
- Unit tests for all tax calculations with ATO/IRS/HMRC test cases
- Integration tests for multi-component scenarios
- Edge case testing for cross-border scenarios
- Regression testing for calculation engine changes

**Multi-Country Validation**:
- Country-specific test suites with real-world scenarios
- Tax optimization validation against professional software
- Currency conversion accuracy testing
- Cross-border calculation validation

## Success Criteria

### Acceptance Criteria
- [ ] Users can create unlimited scenarios with any combination of components
- [ ] All calculations are ATO/IRS/HMRC compliant with <0.01% variance
- [ ] Scenario results update in real-time as parameters change
- [ ] Template system provides 80% coverage of common strategies per country
- [ ] Scenarios can be duplicated, compared, and exported
- [ ] Cross-border scenarios handle dual tax residency correctly

### Performance Requirements
- Scenario calculations complete within 2 seconds for 30-year projections
- Real-time parameter updates with <100ms latency
- Support for 100+ scenarios per user without performance degradation
- Calculation caching reduces repeated computation by 90%

### Compliance Requirements  
- ATO compliance validated by Australian tax professional
- IRS compliance validated by US CPA
- HMRC compliance validated by UK chartered accountant
- All tax calculations include regulatory change tracking
- Audit trail for all calculation methodologies and assumptions