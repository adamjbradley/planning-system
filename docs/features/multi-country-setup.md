# Multi-Country Setup Feature

## Feature Overview

### Purpose
Enable users to select their tax residency country and configure the platform for localized financial planning, including currency, tax systems, investment options, and regulatory frameworks specific to Australia, USA, or UK.

### User Stories

#### **Australian Users**
- **As Emma (Sydney resident)**, I want to select Australia so the platform shows AUD currency, ASX investments, and negative gearing strategies
- **As Mark (Business owner)**, I want my superannuation contributions calculated according to ATO rules and current caps
- **As Sarah (Teacher)**, I want property calculations to include stamp duty and FHOG eligibility for my state

#### **US Users**
- **As David (San Francisco tech worker)**, I want to select USA and California so state income tax is included in all calculations
- **As Lisa (Austin resident)**, I want 401k and IRA options with federal and Texas-specific tax implications
- **As Mike (Chicago consultant)**, I want tax-loss harvesting strategies that comply with wash sale rules

#### **UK Users**
- **As James (London finance professional)**, I want to select UK so ISA allowances and pension tax relief are calculated correctly
- **As Rachel (Manchester NHS worker)**, I want my pension contributions optimized for current and lifetime allowances
- **As Tom (Edinburgh self-employed)**, I want Scottish tax rates applied to my income calculations

#### **Cross-Border Users**
- **As Alex (US citizen in Australia)**, I want to declare dual tax obligations and see PFIC implications for Australian investments
- **As Sarah (UK resident with US 401k)**, I want tax treaty benefits applied to my existing US retirement accounts

### Priority
**P0 - Critical** (Must implement first - foundation for all other features)

### Dependencies
- Existing User authentication system
- Existing subscription/payment system for multi-currency handling
- Basic User model extension for country-specific fields

## Technical Specifications

### Data Models (schema.prisma extensions)

```prisma
// Extend existing User model
model User {
  // ... existing fields ...
  
  // Country-specific fields
  primaryCountry      Country
  taxResidencyCountry Country? // For dual residents
  currency           Currency
  state              String?   // US states, AU states, UK regions
  timezone           String?   @default("UTC")
  
  // Financial profile
  taxYear            String?   // e.g., "2024" for current tax year
  birthYear          Int?      // For age-based calculations
  retirementAge      Int?      @default(65)
  
  // Platform preferences
  complexityLevel    ComplexityLevel @default(ELI5)
  hasCompletedSetup  Boolean         @default(false)
  setupCompletedAt   DateTime?
  
  // Existing relationships
  scenarios          Scenario[]
}

model CountryConfig {
  id                 String   @id @default(uuid())
  country            Country
  isActive           Boolean  @default(true)
  
  // Tax system configuration
  taxYearStart       String   // "2024-07-01" for AU, "2024-01-01" for US/UK
  taxYearEnd         String
  incomeTaxBrackets  Json     // Current year tax brackets
  
  // Investment universe
  defaultETFs        Json     // VAS/VGS for AU, VTI/VXUS for US, etc.
  retirementAccounts Json     // Super for AU, 401k/IRA for US, Pensions for UK
  
  // Country-specific rates
  defaultMortgageRate Float
  defaultPropertyGrowth Float
  defaultInflationRate Float
  
  createdAt          DateTime @default(now())
  updatedAt          DateTime @updatedAt
  
  @@map("country_configs")
}

model StateConfig {
  id                 String   @id @default(uuid())
  country            Country
  stateCode          String   // "NSW", "CA", "LON"
  stateName          String   // "New South Wales", "California", "London"
  
  // State-specific tax rules
  stateTaxRate       Float?   // State income tax rate
  stampDutyRates     Json?    // Property transaction costs
  propertyTaxRates   Json?    // Annual property taxes
  
  // Local incentives
  firstHomeBuyer     Json?    // FHOG, Help to Buy, etc.
  investmentIncentives Json?  // Local tax breaks, grants
  
  isActive           Boolean  @default(true)
  
  @@unique([country, stateCode])
  @@map("state_configs")
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

enum ComplexityLevel {
  ELI5
  INTERMEDIATE
  EXPERT
}
```

### API Operations (main.wasp additions)

```wasp
// Country Setup Queries
query getCountryConfigs {
  fn: import { getCountryConfigs } from "@src/multi-country-setup/operations",
  entities: [CountryConfig]
}

query getStateConfigs {
  fn: import { getStateConfigs } from "@src/multi-country-setup/operations", 
  entities: [StateConfig]
}

query getUserCountrySetup {
  fn: import { getUserCountrySetup } from "@src/multi-country-setup/operations",
  entities: [User, CountryConfig, StateConfig]
}

// Setup Actions
action completeCountrySetup {
  fn: import { completeCountrySetup } from "@src/multi-country-setup/operations",
  entities: [User, CountryConfig, StateConfig]
}

action updateUserComplexity {
  fn: import { updateUserComplexity } from "@src/multi-country-setup/operations",
  entities: [User]
}

action updateTaxResidency {
  fn: import { updateTaxResidency } from "@src/multi-country-setup/operations",
  entities: [User]
}

// Tax Engine Initialization
query initializeTaxEngine {
  fn: import { initializeTaxEngine } from "@src/multi-country-setup/operations",
  entities: [User, CountryConfig, StateConfig]
}
```

### Tax Engine Integration

#### **Tax Engine Factory Pattern**
```typescript
// Tax engine selection based on user country
interface TaxEngineFactory {
  createTaxEngine(user: User, config: CountryConfig): TaxEngine
}

class MultiCountryTaxEngineFactory implements TaxEngineFactory {
  createTaxEngine(user: User, config: CountryConfig): TaxEngine {
    switch (user.primaryCountry) {
      case 'AUSTRALIA':
        return new AustralianTaxEngine(user, config)
      case 'UNITED_STATES':
        return new AmericanTaxEngine(user, config)
      case 'UNITED_KINGDOM':
        return new BritishTaxEngine(user, config)
      default:
        throw new Error(`Unsupported country: ${user.primaryCountry}`)
    }
  }
}
```

#### **Currency Conversion Service**
```typescript
interface CurrencyService {
  convertToUserCurrency(amount: number, fromCurrency: Currency, user: User): number
  getCurrentExchangeRate(from: Currency, to: Currency): number
  formatCurrency(amount: number, currency: Currency): string
}

class CurrencyConversionService implements CurrencyService {
  // TODO: Mock implementation for MVP, real-time for Phase 2
  private static MOCK_RATES = {
    'AUD_USD': 0.67,
    'AUD_GBP': 0.53,
    'USD_GBP': 0.79,
    'USD_AUD': 1.49,
    'GBP_USD': 1.27,
    'GBP_AUD': 1.89
  }
  
  convertToUserCurrency(amount: number, fromCurrency: Currency, user: User): number {
    if (fromCurrency === user.currency) return amount
    
    const rateKey = `${fromCurrency}_${user.currency}`
    const rate = CurrencyConversionService.MOCK_RATES[rateKey] || 1
    return amount * rate
  }
}
```

### Integration Points
- **Authentication**: Extend existing User model seamlessly
- **Payments**: Multi-currency subscription handling via existing Stripe/LemonSqueezy
- **File Upload**: Country-specific document templates and export formats
- **Admin Dashboard**: Multi-country user analytics and configuration management

## Multi-Country Requirements

### Australia Implementation
```typescript
class AustralianSetupService {
  static STATES = [
    { code: 'NSW', name: 'New South Wales', stampDuty: 'progressive', fhog: 10000 },
    { code: 'VIC', name: 'Victoria', stampDuty: 'progressive', fhog: 10000 },
    { code: 'QLD', name: 'Queensland', stampDuty: 'progressive', fhog: 15000 },
    { code: 'WA', name: 'Western Australia', stampDuty: 'progressive', fhog: 10000 },
    { code: 'SA', name: 'South Australia', stampDuty: 'progressive', fhog: 15000 },
    { code: 'TAS', name: 'Tasmania', stampDuty: 'progressive', fhog: 20000 },
    { code: 'ACT', name: 'Australian Capital Territory', stampDuty: 'flat', fhog: 7000 },
    { code: 'NT', name: 'Northern Territory', stampDuty: 'flat', fhog: 26000 }
  ]
  
  static TAX_YEAR = { start: '2024-07-01', end: '2025-06-30' }
  static SUPER_GUARANTEE_RATE = 0.115 // 11.5% for 2024-25
  static SUPER_CAPS = {
    concessional: 30000,      // 2024-25
    nonConcessional: 120000,  // 2024-25
    transferBalance: 1900000  // 2024-25
  }
}
```

### USA Implementation
```typescript
class AmericanSetupService {
  static STATES = [
    { code: 'CA', name: 'California', stateTax: 0.133, noStateTax: false },
    { code: 'NY', name: 'New York', stateTax: 0.109, noStateTax: false },
    { code: 'TX', name: 'Texas', stateTax: 0, noStateTax: true },
    { code: 'FL', name: 'Florida', stateTax: 0, noStateTax: true },
    { code: 'WA', name: 'Washington', stateTax: 0, noStateTax: true },
    // ... all 50 states
  ]
  
  static TAX_YEAR = { start: '2024-01-01', end: '2024-12-31' }
  static RETIREMENT_CAPS_2024 = {
    traditional401k: 23000,
    catch_up_401k: 7500,      // Age 50+
    ira_contribution: 7000,
    catch_up_ira: 1000,       // Age 50+
    hsa_individual: 4300,
    hsa_family: 8550
  }
}
```

### UK Implementation
```typescript
class BritishSetupService {
  static REGIONS = [
    { code: 'ENG', name: 'England', isScotland: false },
    { code: 'SCT', name: 'Scotland', isScotland: true }, // Different tax rates
    { code: 'WLS', name: 'Wales', isScotland: false },
    { code: 'NIR', name: 'Northern Ireland', isScotland: false }
  ]
  
  static TAX_YEAR = { start: '2024-04-06', end: '2025-04-05' }
  static ALLOWANCES_2024_25 = {
    isa_annual: 20000,
    pension_annual: 60000,    // 2024-25
    pension_lifetime: 1073100, // 2024-25
    capital_gains_annual: 3000,
    dividend_allowance: 1000,
    personal_allowance: 12570
  }
}
```

## User Interface Requirements

### Country Selection Flow

#### **Step 1: Welcome & Country Selection**
```typescript
const CountrySelectionPage = () => {
  return (
    <div className="max-w-2xl mx-auto p-8">
      <h1>Welcome to FinPath Explorer</h1>
      <p>Let's set up your personalized financial planning experience.</p>
      
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mt-8">
        <CountryCard 
          country="AUSTRALIA"
          flag="🇦🇺"
          currency="AUD"
          features={["Negative Gearing", "Franking Credits", "Superannuation"]}
        />
        <CountryCard 
          country="UNITED_STATES"
          flag="🇺🇸" 
          currency="USD"
          features={["401k/IRA", "Tax-Loss Harvesting", "State Optimization"]}
        />
        <CountryCard 
          country="UNITED_KINGDOM"
          flag="🇬🇧"
          currency="GBP"
          features={["ISA", "Pensions", "Buy-to-Let"]}
        />
      </div>
    </div>
  )
}
```

#### **Step 2: State/Region Selection**
```typescript
const StateSelectionStep = ({ country }: { country: Country }) => {
  const { data: states } = useQuery(getStateConfigs, { country })
  
  return (
    <div className="space-y-6">
      <h2>Select your state/region</h2>
      <p>This helps us apply the correct tax rates and local incentives.</p>
      
      <Select>
        {states?.map(state => (
          <SelectItem key={state.code} value={state.code}>
            {state.name}
            {state.firstHomeBuyer && <Badge>FHOG Available</Badge>}
            {state.noStateTax && <Badge>No State Tax</Badge>}
          </SelectItem>
        ))}
      </Select>
    </div>
  )
}
```

#### **Step 3: Complexity Level Selection**
```typescript
const ComplexitySelectionStep = () => {
  return (
    <div className="space-y-6">
      <h2>Choose your experience level</h2>
      
      <div className="grid gap-4">
        <ComplexityCard
          level="ELI5"
          title="Beginner"
          description="Simple interface, plain English, guided experience"
          features={["Visual progress bars", "Simple recommendations", "Educational tooltips"]}
        />
        <ComplexityCard
          level="INTERMEDIATE"
          title="Intermediate"
          description="More options, comparison tools, moderate detail"
          features={["Multiple scenarios", "Tax optimization", "Investment comparisons"]}
        />
        <ComplexityCard
          level="EXPERT"
          title="Expert"
          description="Full control, advanced analytics, detailed assumptions"
          features={["Custom parameters", "Monte Carlo", "Tax strategy optimization"]}
        />
      </div>
      
      <Alert>
        <Info className="h-4 w-4" />
        <AlertDescription>
          You can change your experience level anytime in settings.
        </AlertDescription>
      </Alert>
    </div>
  )
}
```

#### **Step 4: Basic Financial Profile**
```typescript
const FinancialProfileStep = () => {
  return (
    <Form>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <FormField name="birthYear">
          <FormLabel>Birth Year</FormLabel>
          <FormControl>
            <Input type="number" placeholder="1985" />
          </FormControl>
          <FormDescription>Used for age-based calculations</FormDescription>
        </FormField>
        
        <FormField name="retirementAge">
          <FormLabel>Target Retirement Age</FormLabel>
          <FormControl>
            <Input type="number" placeholder="65" />
          </FormControl>
        </FormField>
        
        <FormField name="annualIncome">
          <FormLabel>Annual Income ({user.currency})</FormLabel>
          <FormControl>
            <Input type="number" placeholder="75000" />
          </FormControl>
        </FormField>
        
        <FormField name="currentSavings">
          <FormLabel>Current Savings ({user.currency})</FormLabel>
          <FormControl>
            <Input type="number" placeholder="25000" />
          </FormControl>
        </FormField>
      </div>
    </Form>
  )
}
```

### Adaptive Complexity Interface

#### **ELI5 Dashboard Example**
```typescript
const ELI5Dashboard = ({ user }: { user: User }) => {
  return (
    <div className="space-y-6">
      <WelcomeCard>
        <h2>Welcome back, {user.firstName}!</h2>
        <p>You're based in {user.state}, {user.primaryCountry}</p>
        <Badge>{user.currency} currency</Badge>
      </WelcomeCard>
      
      <QuickStatusCard>
        <h3>Your retirement status</h3>
        <Progress value={72} className="w-full" />
        <p>You're 72% on track for comfortable retirement</p>
        <Button>See how to improve</Button>
      </QuickStatusCard>
      
      <SimpleActionCards>
        <ActionCard title="Start a scenario" icon="🎯" />
        <ActionCard title="Compare strategies" icon="⚖️" />
        <ActionCard title="Level up" icon="📈" />
      </SimpleActionCards>
    </div>
  )
}
```

#### **Expert Dashboard Example**
```typescript
const ExpertDashboard = ({ user }: { user: User }) => {
  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <TaxSummaryCard user={user}>
        <h3>Tax Optimization Status</h3>
        <dl>
          <dt>Current Tax Rate</dt>
          <dd>{user.marginalTaxRate}%</dd>
          <dt>Annual Tax Savings Identified</dt>
          <dd>{formatCurrency(user.potentialTaxSavings, user.currency)}</dd>
        </dl>
      </TaxSummaryCard>
      
      <ScenarioSummaryCard>
        <h3>Active Scenarios</h3>
        <ScenarioList scenarios={user.scenarios} />
        <Button>Create new scenario</Button>
      </ScenarioSummaryCard>
      
      <OptimizationCard>
        <h3>Recommendations</h3>
        <OptimizationList recommendations={user.optimizations} />
      </OptimizationCard>
    </div>
  )
}
```

## Implementation Details

### File Structure
```
src/multi-country-setup/
├── operations.ts                    # Wasp operations
├── CountrySelectionPage.tsx        # Initial country selection
├── SetupWizardPage.tsx             # Multi-step setup wizard
├── components/
│   ├── CountryCard.tsx             # Country selection cards
│   ├── StateSelector.tsx           # State/region selection
│   ├── ComplexitySelector.tsx      # Experience level selection
│   ├── FinancialProfileForm.tsx    # Basic financial info
│   └── SetupProgress.tsx           # Progress indicator
├── services/
│   ├── CountryConfigService.ts     # Country configuration logic
│   ├── TaxEngineFactory.ts         # Tax engine creation
│   ├── CurrencyService.ts          # Currency conversion
│   └── SetupValidationService.ts   # Setup validation
├── hooks/
│   ├── useCountrySetup.tsx         # Setup state management
│   ├── useCurrencyConversion.tsx   # Currency conversion
│   └── useComplexityLevel.tsx      # Complexity level management
└── utils/
    ├── countryConstants.ts         # Country-specific constants
    ├── setupValidation.ts          # Setup form validation
    └── currencyFormatting.ts       # Currency display utilities
```

### Key Operations Implementation

#### **Complete Country Setup Operation**
```typescript
export const completeCountrySetup: CompleteCountrySetup = async (args, context) => {
  if (!context.user) {
    throw new HttpError(401, 'Authentication required')
  }
  
  // Validate setup data
  const validatedData = CountrySetupSchema.parse(args)
  
  // Initialize tax engine for validation
  const countryConfig = await context.entities.CountryConfig.findFirst({
    where: { country: validatedData.primaryCountry, isActive: true }
  })
  
  if (!countryConfig) {
    throw new HttpError(400, `Country configuration not found for ${validatedData.primaryCountry}`)
  }
  
  // Update user with country setup
  const updatedUser = await context.entities.User.update({
    where: { id: context.user.id },
    data: {
      primaryCountry: validatedData.primaryCountry,
      currency: validatedData.currency,
      state: validatedData.state,
      complexityLevel: validatedData.complexityLevel,
      birthYear: validatedData.birthYear,
      retirementAge: validatedData.retirementAge,
      hasCompletedSetup: true,
      setupCompletedAt: new Date()
    }
  })
  
  // Initialize default tax engine
  const taxEngine = TaxEngineFactory.createTaxEngine(updatedUser, countryConfig)
  
  return {
    user: updatedUser,
    taxEngineInitialized: true,
    availableTemplates: await getCountryTemplates(validatedData.primaryCountry, context)
  }
}
```

### Testing Requirements

#### **Country Configuration Tests**
- Verify all tax brackets and rates are current for each country
- Test state/region-specific calculations
- Validate currency conversion accuracy
- Test tax year date handling across countries

#### **Setup Flow Tests**
- Complete setup wizard flow for each country
- Cross-border setup scenarios
- Complexity level transitions
- Data persistence and restoration

#### **Tax Engine Initialization Tests**
- Verify correct tax engine selection by country
- Test basic calculations after setup
- Validate country-specific features availability

## Success Criteria

### Acceptance Criteria
- [ ] Users can select their country and complete setup within 3 minutes
- [ ] All country-specific tax calculations initialize correctly
- [ ] Currency conversion works accurately for all supported currencies
- [ ] State/region-specific features are available and functional
- [ ] Complexity level affects interface appropriately
- [ ] Cross-border users can declare dual tax residency
- [ ] Setup can be modified and updated after completion

### Performance Requirements
- Country selection and setup completes within 5 seconds
- Tax engine initialization completes within 2 seconds
- Currency conversions complete within 500ms
- Setup wizard loads and transitions smoothly

### Compliance Requirements
- **Australia**: ATO tax year dates and current rates loaded correctly
- **USA**: Current federal and state tax rates for all 50 states
- **UK**: HMRC tax year dates and current allowances
- **All Countries**: Data privacy compliance (GDPR, CCPA, Privacy Act)

---

*Multi-Country Setup is the foundation feature that enables all other financial planning capabilities. Focus on accuracy, compliance, and smooth user experience across all three countries.*