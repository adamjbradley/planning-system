# Housing Strategies

## Feature Overview

**Purpose**: Comprehensive housing strategy components allowing users to model buying primary residences, investment properties, and rental scenarios with full tax optimization and country-specific considerations.

**User Stories**:
- As a first home buyer in AU, I want to model "buy vs rent" with FHOG and stamp duty considerations
- As a US investor, I want to compare rental property with depreciation schedules and state tax implications
- As a UK couple, I want to model Help to Buy schemes vs traditional mortgages for primary residence
- As an Australian investor, I want to optimize negative gearing across multiple investment properties

**Priority**: P0 (Critical) - Core wealth-building component

**Dependencies**: Multi-Country Setup, Scenario Management, Tax Engine Integration

## Technical Specifications

### Data Models

```prisma
model HousingComponent {
  id                  String   @id @default(cuid())
  scenarioId          String
  scenario            Scenario @relation(fields: [scenarioId], references: [id], onDelete: Cascade)
  
  // Component Identification
  type                HousingType
  name                String
  description         String?
  startYear           Int      @default(0)
  endYear             Int?     // Optional sale year
  
  // Location and Property Details
  location            String   // State/City for tax calculations
  propertyType        PropertyType
  
  // Purchase Configuration
  purchasePrice       Decimal?
  stampDuty           Decimal? // Auto-calculated or manual override
  legalFees           Decimal? @default(0)
  inspectionCosts     Decimal? @default(0)
  
  // Financing Details
  deposit             Decimal?
  mortgageAmount      Decimal?
  mortgageRate        Decimal?
  mortgageTerm        Int?     // Years
  mortgageType        MortgageType @default(PRINCIPAL_INTEREST)
  
  // Rental Configuration (for both renting and investment)
  rentAmount          Decimal? // Weekly rent amount
  rentGrowthRate      Decimal? @default(0.03)
  rentReviewPeriod    Int?     @default(12) // Months
  
  // Investment Property Specifics
  rentalYield         Decimal? // Calculated or manual override
  propertyGrowthRate  Decimal? @default(0.05)
  vacancy             Decimal? @default(0.02) // Vacancy rate as percentage
  
  // Ongoing Costs
  councilRates        Decimal? @default(0)
  waterRates          Decimal? @default(0)
  insurance           Decimal? @default(0)
  maintenanceCost     Decimal? @default(0)
  managementFees      Decimal? @default(0)
  bodyCorporate       Decimal? @default(0)
  
  // Tax Optimizations
  enableNegativeGearing Boolean @default(false) // AU
  depreciationClaim     Decimal? @default(0)    // US/AU
  capitalWorksDeduction Decimal? @default(0)    // AU specific
  
  // Government Schemes
  fhogAmount            Decimal? @default(0)    // AU First Home Owner Grant
  helpToBuyEquity       Decimal? @default(0)    // UK Help to Buy
  sharedEquityScheme    String?                 // Various government schemes
  
  // Sale Configuration
  salePrice             Decimal?
  saleYear              Int?
  agentCommission       Decimal? @default(0.02)
  legalFeesOnSale       Decimal? @default(0)
  
  // Calculated Fields (updated by calculation engine)
  totalCashFlow         Decimal? // Annual cash flow
  totalTaxBenefits      Decimal? // Annual tax benefits
  totalReturn           Decimal? // Annual total return
  
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
  
  @@map("housing_components")
}

enum HousingType {
  BUY_PRIMARY      // Buy primary place of residence
  BUY_INVESTMENT   // Buy investment property
  RENT_PRIMARY     // Rent primary residence
  RENT_TO_BUY      // Rent while saving to buy
}

enum PropertyType {
  HOUSE
  APARTMENT
  TOWNHOUSE
  UNIT
  COMMERCIAL
  LAND
}

enum MortgageType {
  PRINCIPAL_INTEREST  // Standard P&I
  INTEREST_ONLY      // Interest only period
  OFFSET             // Offset mortgage
  REDRAW             // Redraw facility
}

model PropertyMarketData {
  id                  String   @id @default(cuid())
  
  // Location Details
  country             Country
  state               String
  city                String
  suburb              String?
  postcode            String?
  
  // Market Data
  medianPrice         Decimal
  priceGrowthRate     Decimal  // Annual growth rate
  rentalYield         Decimal  // Gross rental yield
  medianRent          Decimal  // Weekly median rent
  
  // Market Metrics
  daysOnMarket        Int?
  clearanceRate       Decimal?
  stockLevels         Int?
  
  // Data Source
  dataSource          String   // CoreLogic, Domain, etc.
  lastUpdated         DateTime
  
  @@unique([country, state, city, suburb, postcode])
  @@map("property_market_data")
}
```

### Housing Calculator Engine

```typescript
export class HousingCalculator {
  constructor(
    private taxEngine: ITaxEngine,
    private marketDataService: PropertyMarketDataService
  ) {}
  
  async calculateHousingComponent(
    component: HousingComponent,
    scenario: Scenario,
    year: number
  ): Promise<HousingResults> {
    
    switch (component.type) {
      case 'BUY_PRIMARY':
        return this.calculatePrimaryResidence(component, scenario, year)
      case 'BUY_INVESTMENT':
        return this.calculateInvestmentProperty(component, scenario, year)
      case 'RENT_PRIMARY':
        return this.calculateRentalCosts(component, scenario, year)
      case 'RENT_TO_BUY':
        return this.calculateRentToBuy(component, scenario, year)
      default:
        throw new Error(`Unknown housing type: ${component.type}`)
    }
  }
  
  private async calculateInvestmentProperty(
    component: HousingComponent,
    scenario: Scenario,
    year: number
  ): Promise<HousingResults> {
    
    // Property value calculation
    const currentValue = this.calculatePropertyValue(component, year)
    
    // Rental income calculation
    const grossRentalIncome = this.calculateRentalIncome(component, year)
    const vacancyLoss = grossRentalIncome * (component.vacancy || 0.02)
    const netRentalIncome = grossRentalIncome - vacancyLoss
    
    // Operating expenses
    const operatingExpenses = this.calculateOperatingExpenses(component, year)
    
    // Mortgage calculations
    const mortgageDetails = this.calculateMortgage(component, year)
    
    // Cash flow before tax
    const cashFlowBeforeTax = netRentalIncome - operatingExpenses - mortgageDetails.interest
    
    // Tax calculations
    const taxCalculation = await this.calculatePropertyTax(
      component,
      scenario,
      year,
      {
        rentalIncome: netRentalIncome,
        operatingExpenses,
        interestExpense: mortgageDetails.interest,
        depreciation: component.depreciationClaim || 0,
        capitalWorks: component.capitalWorksDeduction || 0
      }
    )
    
    // Net cash flow after tax
    const netCashFlow = cashFlowBeforeTax + taxCalculation.taxBenefit
    
    // Equity calculations
    const outstandingDebt = mortgageDetails.outstandingBalance
    const equity = currentValue - outstandingDebt
    
    return {
      propertyValue: currentValue,
      debt: outstandingDebt,
      equity,
      grossRentalIncome,
      netRentalIncome,
      operatingExpenses,
      interestExpense: mortgageDetails.interest,
      principalRepayment: mortgageDetails.principal,
      cashFlowBeforeTax,
      taxBenefit: taxCalculation.taxBenefit,
      netCashFlow,
      capitalGains: year > 0 ? this.calculateCapitalGains(component, year) : 0,
      totalReturn: netCashFlow + (year > 0 ? this.calculateCapitalGains(component, year) : 0)
    }
  }
  
  private async calculatePropertyTax(
    component: HousingComponent,
    scenario: Scenario,
    year: number,
    incomeStatement: PropertyIncomeStatement
  ): Promise<PropertyTaxCalculation> {
    
    const deductions = this.calculateTaxDeductions(component, incomeStatement)
    const negativeGearing = component.enableNegativeGearing && deductions.total > incomeStatement.rentalIncome ? 
      deductions.total - incomeStatement.rentalIncome : 0
    
    // Country-specific tax calculations
    switch (scenario.country) {
      case 'AUSTRALIA':
        return this.calculateAustralianPropertyTax(component, incomeStatement, deductions, negativeGearing)
      case 'USA':
        return this.calculateUSPropertyTax(component, incomeStatement, deductions)
      case 'UK':
        return this.calculateUKPropertyTax(component, incomeStatement, deductions)
      default:
        throw new Error(`Tax calculations not implemented for ${scenario.country}`)
    }
  }
  
  private calculateAustralianPropertyTax(
    component: HousingComponent,
    incomeStatement: PropertyIncomeStatement,
    deductions: PropertyDeductions,
    negativeGearing: number
  ): PropertyTaxCalculation {
    
    // Australian negative gearing benefits
    const marginalTaxRate = 0.325 // Simplified - should come from user's income
    const negativeGearingBenefit = negativeGearing * marginalTaxRate
    
    // Depreciation benefits
    const depreciationBenefit = incomeStatement.depreciation * marginalTaxRate
    
    // Capital works deduction
    const capitalWorksBenefit = incomeStatement.capitalWorks * marginalTaxRate
    
    const totalTaxBenefit = negativeGearingBenefit + depreciationBenefit + capitalWorksBenefit
    
    return {
      taxableIncome: Math.max(0, incomeStatement.rentalIncome - deductions.total),
      deductions: deductions.total,
      negativeGearing,
      taxBenefit: totalTaxBenefit,
      marginalTaxRate
    }
  }
  
  private calculateUSPropertyTax(
    component: HousingComponent,
    incomeStatement: PropertyIncomeStatement,
    deductions: PropertyDeductions
  ): PropertyTaxCalculation {
    
    // US depreciation (residential property over 27.5 years)
    const depreciationAmount = component.purchasePrice ? 
      (component.purchasePrice * 0.8) / 27.5 : 0 // 80% of value, 27.5 year schedule
    
    // Calculate taxable rental income
    const taxableIncome = Math.max(0, incomeStatement.rentalIncome - deductions.total - depreciationAmount)
    
    // Tax benefit from depreciation and expenses
    const marginalTaxRate = 0.22 // Simplified - should be calculated from user's total income
    const taxBenefit = (deductions.total + depreciationAmount) * marginalTaxRate
    
    return {
      taxableIncome,
      deductions: deductions.total + depreciationAmount,
      negativeGearing: 0, // US doesn't have negative gearing concept
      taxBenefit,
      marginalTaxRate
    }
  }
  
  private calculateMortgage(component: HousingComponent, year: number): MortgageCalculation {
    if (!component.mortgageAmount || !component.mortgageRate || !component.mortgageTerm) {
      return { interest: 0, principal: 0, outstandingBalance: 0 }
    }
    
    const monthlyRate = component.mortgageRate / 12
    const totalPayments = component.mortgageTerm * 12
    const monthsElapsed = year * 12
    
    // Calculate monthly payment
    const monthlyPayment = this.calculateMonthlyPayment(
      component.mortgageAmount,
      monthlyRate,
      totalPayments
    )
    
    // Calculate outstanding balance
    const outstandingBalance = this.calculateOutstandingBalance(
      component.mortgageAmount,
      monthlyRate,
      totalPayments,
      monthsElapsed
    )
    
    // Calculate annual interest and principal for current year
    let annualInterest = 0
    let annualPrincipal = 0
    
    for (let month = 1; month <= 12; month++) {
      const currentMonth = monthsElapsed + month
      if (currentMonth <= totalPayments) {
        const monthlyInterest = outstandingBalance * monthlyRate
        const monthlyPrincipal = monthlyPayment - monthlyInterest
        
        annualInterest += monthlyInterest
        annualPrincipal += monthlyPrincipal
      }
    }
    
    return {
      interest: annualInterest,
      principal: annualPrincipal,
      outstandingBalance: Math.max(0, outstandingBalance)
    }
  }
  
  calculateFirstHomeBuyerIncentives(
    component: HousingComponent,
    scenario: Scenario
  ): FirstHomeBuyerIncentives {
    
    const incentives: FirstHomeBuyerIncentives = {
      fhog: 0,
      stampDutyDiscount: 0,
      sharedEquity: 0,
      totalIncentives: 0
    }
    
    if (component.type !== 'BUY_PRIMARY') {
      return incentives
    }
    
    switch (scenario.country) {
      case 'AUSTRALIA':
        incentives.fhog = this.calculateAustralianFHOG(component, scenario)
        incentives.stampDutyDiscount = this.calculateStampDutyDiscount(component, scenario)
        break
        
      case 'UK':
        incentives.sharedEquity = component.helpToBuyEquity || 0
        incentives.stampDutyDiscount = this.calculateUKStampDutyDiscount(component, scenario)
        break
        
      case 'USA':
        // US programs vary by state and locality
        incentives.fhog = this.calculateUSFirstTimeBuyerPrograms(component, scenario)
        break
    }
    
    incentives.totalIncentives = incentives.fhog + incentives.stampDutyDiscount + incentives.sharedEquity
    
    return incentives
  }
}

interface PropertyIncomeStatement {
  rentalIncome: number
  operatingExpenses: number
  interestExpense: number
  depreciation: number
  capitalWorks: number
}

interface PropertyDeductions {
  interest: number
  maintenance: number
  management: number
  insurance: number
  rates: number
  depreciation: number
  capitalWorks: number
  total: number
}

interface PropertyTaxCalculation {
  taxableIncome: number
  deductions: number
  negativeGearing: number
  taxBenefit: number
  marginalTaxRate: number
}

interface MortgageCalculation {
  interest: number
  principal: number
  outstandingBalance: number
}

interface FirstHomeBuyerIncentives {
  fhog: number                // First Home Owner Grant
  stampDutyDiscount: number   // Stamp duty concessions
  sharedEquity: number        // Government equity schemes
  totalIncentives: number
}
```

### Country-Specific Implementations

```typescript
// Australia Housing Strategies
export class AustralianHousingStrategies {
  
  calculateNegativeGearing(
    rentalIncome: number,
    totalDeductions: number,
    marginalTaxRate: number
  ): NegativeGearingResult {
    
    const netRentalIncome = rentalIncome - totalDeductions
    const negativeGearingAmount = Math.max(0, -netRentalIncome)
    const taxBenefit = negativeGearingAmount * marginalTaxRate
    
    return {
      netRentalIncome,
      negativeGearingAmount,
      taxBenefit,
      afterTaxCashFlow: netRentalIncome + taxBenefit
    }
  }
  
  calculateCapitalGainsDiscount(
    purchasePrice: number,
    salePrice: number,
    holdingPeriod: number, // months
    marginalTaxRate: number
  ): CapitalGainsResult {
    
    const capitalGain = salePrice - purchasePrice
    const isEligibleForDiscount = holdingPeriod >= 12
    const discountRate = isEligibleForDiscount ? 0.5 : 0
    
    const taxableCapitalGain = capitalGain * (1 - discountRate)
    const capitalGainsTax = taxableCapitalGain * marginalTaxRate
    const netCapitalGain = capitalGain - capitalGainsTax
    
    return {
      grossCapitalGain: capitalGain,
      discountApplied: capitalGain * discountRate,
      taxableCapitalGain,
      capitalGainsTax,
      netCapitalGain
    }
  }
  
  calculateStampDuty(purchasePrice: number, state: string, isFirstHomeBuyer: boolean): number {
    const stampDutyRates = {
      'NSW': this.calculateNSWStampDuty,
      'VIC': this.calculateVICStampDuty,
      'QLD': this.calculateQLDStampDuty,
      'WA': this.calculateWAStampDuty,
      'SA': this.calculateSAStampDuty,
      'TAS': this.calculateTASStampDuty,
      'ACT': this.calculateACTStampDuty,
      'NT': this.calculateNTStampDuty
    }
    
    const calculator = stampDutyRates[state]
    if (!calculator) throw new Error(`Stamp duty calculator not implemented for ${state}`)
    
    const baseStampDuty = calculator(purchasePrice)
    
    // Apply first home buyer concessions
    if (isFirstHomeBuyer) {
      return this.applyFirstHomeBuyerConcession(baseStampDuty, purchasePrice, state)
    }
    
    return baseStampDuty
  }
}

// USA Housing Strategies  
export class USAHousingStrategies {
  
  calculateDepreciation(
    purchasePrice: number,
    propertyType: PropertyType,
    year: number
  ): DepreciationResult {
    
    // Residential rental property: 27.5 years
    // Commercial property: 39 years
    const depreciationPeriod = propertyType === 'COMMERCIAL' ? 39 : 27.5
    const depreciableAmount = purchasePrice * 0.8 // 80% of purchase price (exclude land)
    
    const annualDepreciation = depreciableAmount / depreciationPeriod
    const accumulatedDepreciation = Math.min(annualDepreciation * year, depreciableAmount)
    const remainingDepreciation = depreciableAmount - accumulatedDepreciation
    
    return {
      annualDepreciation,
      accumulatedDepreciation,
      remainingDepreciation,
      depreciableAmount
    }
  }
  
  calculate1031Exchange(
    salePrice: number,
    purchasePrice: number,
    accumulatedDepreciation: number
  ): Exchange1031Result {
    
    // Defer capital gains through like-kind exchange
    const capitalGain = salePrice - purchasePrice
    const depreciationRecapture = accumulatedDepreciation
    
    // If properly structured 1031 exchange
    const deferredCapitalGain = capitalGain
    const deferredDepreciationRecapture = depreciationRecapture
    
    return {
      capitalGain,
      depreciationRecapture,
      deferredCapitalGain,
      deferredDepreciationRecapture,
      totalTaxDeferred: (capitalGain * 0.20) + (depreciationRecapture * 0.25) // Estimated tax rates
    }
  }
}

// UK Housing Strategies
export class UKHousingStrategies {
  
  calculateBuyToLetTax(
    rentalIncome: number,
    mortgageInterest: number,
    otherExpenses: number,
    marginalTaxRate: number
  ): BuyToLetTaxResult {
    
    // UK mortgage interest relief changes (gradual phase-out)
    const interestRelief = mortgageInterest * 0.20 // Basic rate relief only
    const taxableRentalIncome = rentalIncome - otherExpenses
    const incomeTax = taxableRentalIncome * marginalTaxRate
    const netIncomeTax = Math.max(0, incomeTax - interestRelief)
    
    return {
      taxableRentalIncome,
      incomeTax,
      interestRelief,
      netIncomeTax,
      netRentalIncome: rentalIncome - otherExpenses - netIncomeTax
    }
  }
  
  calculateHelpToBuyScheme(
    purchasePrice: number,
    deposit: number,
    helpToBuyLoan: number
  ): HelpToBuyResult {
    
    // Help to Buy equity loan (up to 20% outside London, 40% in London)
    const totalEquity = deposit + helpToBuyLoan
    const mortgage = purchasePrice - totalEquity
    
    // Interest-free for first 5 years, then RPI + 1.75%
    const interestFreeYears = 5
    const interestRate = 0.0375 // Approximate RPI + 1.75%
    
    return {
      mortgageAmount: mortgage,
      equityLoanAmount: helpToBuyLoan,
      equityPercentage: helpToBuyLoan / purchasePrice,
      interestFreeYears,
      subsequentInterestRate: interestRate
    }
  }
}
```

## Multi-Country Requirements

### Australia (ATO Compliance)
- **Negative Gearing**: Full calculation of rental property losses offset against taxable income
- **Capital Gains Discount**: 50% discount for properties held >12 months  
- **Depreciation Claims**: Building allowance (2.5%) and plant & equipment depreciation
- **Capital Works Deductions**: Construction costs over 40 years
- **Stamp Duty**: State-by-state calculations with first home buyer concessions
- **FHOG Integration**: First Home Owner Grant eligibility and amounts by state

### USA (IRS Compliance)
- **Depreciation Schedules**: 27.5-year residential, 39-year commercial
- **1031 Like-Kind Exchanges**: Capital gains deferral strategies
- **State Tax Variations**: Property tax and income tax by state
- **Depreciation Recapture**: 25% tax rate on accumulated depreciation
- **Passive Activity Rules**: Rental loss limitations based on income
- **Section 199A Deduction**: 20% deduction for pass-through rental income

### UK (HMRC Compliance)  
- **Mortgage Interest Relief**: Transition to basic rate relief only
- **Capital Gains Tax**: Annual exemption and rates on property sales
- **Buy-to-Let Regulations**: Portfolio landlord requirements
- **Help to Buy Schemes**: Equity loans and shared ownership
- **Stamp Duty Land Tax**: Progressive rates with additional rate for second homes
- **Section 24 Changes**: Mortgage interest restriction for higher-rate taxpayers

### Cross-Border Considerations
- **Non-Resident Tax**: Withholding taxes on rental income
- **Double Tax Treaties**: Relief from double taxation
- **Currency Hedging**: Exchange rate impact on returns
- **Reporting Requirements**: Foreign property disclosure obligations

## User Interface Requirements

### Adaptive Complexity

**ELI5 Mode**:
- Simple "Buy vs Rent" calculator with basic inputs
- Pre-populated property values based on location
- Traffic light indicators for affordability
- Plain English explanations of costs and benefits

**Intermediate Mode**:
- Component-based housing strategy builder  
- Property search integration with market data
- Side-by-side scenario comparison
- Basic tax optimization suggestions

**Expert Mode**:
- Full parameter customization
- Advanced tax optimization strategies
- Multiple property portfolio modeling
- Detailed cash flow and tax projections

### Components

```tsx
// Housing Strategy Builder
export function HousingStrategyBuilder({ scenario }: { scenario: Scenario }) {
  const [components, setComponents] = useState(scenario.housingComponents)
  const [selectedType, setSelectedType] = useState<HousingType>('BUY_PRIMARY')
  
  return (
    <div className="space-y-6">
      <HousingTypeSelector 
        selectedType={selectedType}
        onTypeChange={setSelectedType}
        country={scenario.country}
      />
      
      <div className="grid gap-6">
        {components.map(component => (
          <HousingComponentCard
            key={component.id}
            component={component}
            scenario={scenario}
            onUpdate={(updated) => updateComponent(component.id, updated)}
            onDelete={() => deleteComponent(component.id)}
          />
        ))}
      </div>
      
      <Button 
        onClick={() => addHousingComponent(selectedType)}
        className="w-full"
      >
        Add {getHousingTypeLabel(selectedType)}
      </Button>
    </div>
  )
}

// Housing Component Editor
export function HousingComponentEditor({ 
  component, 
  scenario, 
  onUpdate 
}: HousingComponentEditorProps) {
  
  const [activeTab, setActiveTab] = useState<'basic' | 'financing' | 'tax' | 'projections'>('basic')
  
  return (
    <Card>
      <CardHeader>
        <CardTitle>{component.name}</CardTitle>
        <HousingComponentTabs activeTab={activeTab} onTabChange={setActiveTab} />
      </CardHeader>
      
      <CardContent>
        {activeTab === 'basic' && (
          <BasicPropertyDetails 
            component={component}
            scenario={scenario}
            onChange={onUpdate}
          />
        )}
        
        {activeTab === 'financing' && (
          <FinancingDetails
            component={component}
            onChange={onUpdate}
          />
        )}
        
        {activeTab === 'tax' && (
          <TaxOptimizationSettings
            component={component}
            scenario={scenario}
            onChange={onUpdate}
          />
        )}
        
        {activeTab === 'projections' && (
          <HousingProjections
            component={component}
            scenario={scenario}
          />
        )}
      </CardContent>
    </Card>
  )
}

// Property Search Integration
export function PropertySearchWidget({ 
  onPropertySelect,
  country 
}: PropertySearchWidgetProps) {
  
  const [searchQuery, setSearchQuery] = useState('')
  const [priceRange, setPriceRange] = useState<[number, number]>([0, 1000000])
  const [propertyType, setPropertyType] = useState<PropertyType>('HOUSE')
  
  const { data: properties, isLoading } = useQuery(
    searchProperties,
    { 
      query: searchQuery,
      priceRange,
      propertyType,
      country 
    }
  )
  
  return (
    <div className="space-y-4">
      <div className="flex gap-4">
        <Input 
          placeholder="Search by location..."
          value={searchQuery}
          onChange={(e) => setSearchQuery(e.target.value)}
        />
        <PropertyTypeSelect 
          value={propertyType}
          onChange={setPropertyType}
        />
      </div>
      
      <PriceRangeSlider 
        value={priceRange}
        onChange={setPriceRange}
        max={getMaxPriceForCountry(country)}
      />
      
      {isLoading ? (
        <PropertySearchSkeleton />
      ) : (
        <div className="grid gap-4 md:grid-cols-2">
          {properties?.map(property => (
            <PropertyCard 
              key={property.id}
              property={property}
              onSelect={() => onPropertySelect(property)}
            />
          ))}
        </div>
      )}
    </div>
  )
}

// Buy vs Rent Comparison
export function BuyVsRentComparison({ scenario }: { scenario: Scenario }) {
  const [buyScenario, setBuyScenario] = useState<HousingComponent>()
  const [rentScenario, setRentScenario] = useState<HousingComponent>()
  
  const { data: comparison } = useQuery(
    compareBuyVsRent,
    { buyScenario, rentScenario, scenario }
  )
  
  return (
    <div className="space-y-6">
      <div className="grid gap-6 md:grid-cols-2">
        <Card>
          <CardHeader>
            <CardTitle>Buying Scenario</CardTitle>
          </CardHeader>
          <CardContent>
            <BuyScenarioBuilder 
              scenario={buyScenario}
              onChange={setBuyScenario}
            />
          </CardContent>
        </Card>
        
        <Card>
          <CardHeader>
            <CardTitle>Renting Scenario</CardTitle>
          </CardHeader>
          <CardContent>
            <RentScenarioBuilder 
              scenario={rentScenario}
              onChange={setRentScenario}
            />
          </CardContent>
        </Card>
      </div>
      
      {comparison && (
        <ComparisonResults comparison={comparison} />
      )}
    </div>
  )
}
```

### User Flows

**Creating Housing Strategy**:
1. Select housing component type (buy/rent/invest)
2. Enter property details or search existing properties
3. Configure financing (deposit, mortgage, rates)
4. Set up tax optimizations (negative gearing, depreciation)
5. Review projections and cash flows
6. Save component to scenario

**Buy vs Rent Analysis**:
1. Enter target property details
2. Configure buying scenario (deposit, mortgage)
3. Configure renting scenario (rent amount, savings)
4. Compare side-by-side projections
5. Factor in opportunity cost of capital
6. Make informed decision

**Investment Property Optimization**:
1. Add investment property component
2. Configure rental income and expenses
3. Enable tax optimizations (negative gearing)
4. Model different holding periods
5. Optimize for cash flow vs capital growth
6. Compare multiple properties

## Implementation Details

### File Structure
```
src/
├── housing/
│   ├── client/
│   │   ├── HousingStrategyBuilder.tsx
│   │   ├── BuyVsRentComparison.tsx
│   │   ├── PropertySearch.tsx
│   │   ├── components/
│   │   │   ├── HousingComponentEditor.tsx
│   │   │   ├── PropertyCard.tsx
│   │   │   ├── MortgageCalculator.tsx
│   │   │   └── TaxOptimizationPanel.tsx
│   │   └── hooks/
│   │       ├── useHousingCalculations.ts
│   │       ├── usePropertySearch.ts
│   │       └── useMortgageCalculator.ts
│   ├── server/
│   │   ├── queries.ts
│   │   ├── actions.ts
│   │   ├── calculations/
│   │   │   ├── HousingCalculator.ts
│   │   │   ├── MortgageCalculator.ts
│   │   │   └── TaxCalculator.ts
│   │   ├── integrations/
│   │   │   ├── PropertyDataAPI.ts
│   │   │   └── MarketDataService.ts
│   │   └── country-specific/
│   │       ├── AustralianHousingStrategies.ts
│   │       ├── USAHousingStrategies.ts
│   │       └── UKHousingStrategies.ts
│   └── shared/
│       ├── types.ts
│       └── constants.ts
```

### Market Data Integration

```typescript
export interface PropertyMarketDataService {
  getMedianPrice(location: PropertyLocation): Promise<number>
  getRentalYield(location: PropertyLocation): Promise<number>
  getGrowthProjections(location: PropertyLocation): Promise<GrowthProjection[]>
  searchProperties(criteria: PropertySearchCriteria): Promise<Property[]>
}

export class MockPropertyDataService implements PropertyMarketDataService {
  // Mock implementation for MVP
  async getMedianPrice(location: PropertyLocation): Promise<number> {
    const basePrices = {
      'Australia': { 'Sydney': 800000, 'Melbourne': 700000, 'Brisbane': 550000 },
      'USA': { 'San Francisco': 1200000, 'New York': 900000, 'Austin': 400000 },
      'UK': { 'London': 500000, 'Manchester': 200000, 'Birmingham': 180000 }
    }
    
    return basePrices[location.country]?.[location.city] || 400000
  }
  
  async getRentalYield(location: PropertyLocation): Promise<number> {
    const baseYields = {
      'Australia': 0.04, // 4% gross yield
      'USA': 0.08,       // 8% gross yield
      'UK': 0.05         // 5% gross yield
    }
    
    return baseYields[location.country] || 0.05
  }
}
```

### Testing Requirements

**Financial Accuracy Testing**:
- Mortgage calculation accuracy against bank calculators
- Tax calculation validation with ATO/IRS/HMRC examples
- Cash flow projections validation across different scenarios
- Capital gains tax calculations with real-world examples

**Integration Testing**:
- Property search integration with mock and real data
- Multi-component housing scenarios
- Cross-country property comparisons
- Tax optimization edge cases

## Success Criteria

### Acceptance Criteria
- [ ] Users can model any combination of buy/rent/invest housing strategies
- [ ] All tax calculations are compliant with local authorities (ATO/IRS/HMRC)
- [ ] Property search integration provides relevant market data
- [ ] Buy vs rent comparison provides clear recommendations
- [ ] Investment property calculations include all deductions and optimizations
- [ ] First home buyer incentives are accurately calculated

### Performance Requirements
- Housing component calculations complete within 1 second
- Property search returns results within 2 seconds
- Market data updates cached for 24 hours
- Complex multi-property scenarios calculate within 5 seconds

### Compliance Requirements
- All stamp duty calculations verified against state/government calculators
- Tax deduction calculations validated by qualified professionals
- Depreciation schedules comply with local tax authority guidelines
- Capital gains calculations include all applicable discounts and rules