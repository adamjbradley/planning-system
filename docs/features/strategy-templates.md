# Strategy Templates

## Feature Overview

**Purpose**: Pre-built financial strategy templates that provide users with proven financial planning approaches specific to their country, automatically configured with appropriate tax optimizations and investment allocations.

**User Stories**:
- As a first-time investor in AU, I want a "Superannuation Boost" template that maximizes my retirement savings with salary sacrifice
- As a US professional, I want a "Roth Ladder" template that optimizes my retirement tax situation across different income phases
- As a UK couple, I want an "ISA Maxing" template that efficiently uses both partners' annual allowances
- As a property investor, I want country-specific "Buy-to-Let" templates with accurate tax calculations

**Priority**: P0 (Critical) - Essential for user onboarding and complexity management

**Dependencies**: Multi-Country Setup, Scenario Management, Tax Engine Integration

## Technical Specifications

### Data Models

```prisma
model StrategyTemplate {
  id                  String   @id @default(cuid())
  name                String
  description         String
  longDescription     String?
  
  // Classification
  country             Country
  category            TemplateCategory
  complexity          ComplexityLevel
  tags                String[]
  
  // Template Configuration
  defaultTimeHorizon  Int      @default(30)
  targetAgeGroup      String   // "25-35", "35-45", "45-55", "55+"
  incomeRange         String   // "0-50k", "50k-100k", "100k-200k", "200k+"
  
  // Strategy Components (JSON templates)
  housingTemplate     Json?    // HousingComponent template
  investmentTemplate  Json     // InvestmentComponent[] template
  incomeTemplate      Json?    // IncomeComponent[] template
  expenseTemplate     Json?    // ExpenseComponent[] template
  
  // Educational Content
  educationalContent  TemplateEducation?
  
  // Metadata
  isActive            Boolean  @default(true)
  createdBy           String?  // Professional/admin who created
  validatedBy         String?  // Tax professional validation
  lastValidated       DateTime?
  
  // Usage Analytics
  usageCount          Int      @default(0)
  averageRating       Decimal? @default(0)
  
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
  
  @@map("strategy_templates")
}

model TemplateEducation {
  id                  String   @id @default(cuid())
  templateId          String   @unique
  template            StrategyTemplate @relation(fields: [templateId], references: [id], onDelete: Cascade)
  
  // Educational Content
  overview            String   // Why this strategy works
  keyBenefits         String[] // Bullet points of benefits
  riskConsiderations  String[] // What to watch out for
  taxImplications     String   // Country-specific tax details
  
  // Interactive Elements
  calculatorPrompts   Json?    // Guided questions for customization
  whatIfScenarios     Json?    // Common variations to explore
  
  // Professional Insights
  professionalTips    String[] // Expert advice
  commonMistakes      String[] // Things to avoid
  
  @@map("template_education")
}

enum TemplateCategory {
  // Universal Categories
  FIRST_TIME_INVESTOR
  AGGRESSIVE_GROWTH  
  BALANCED_GROWTH
  CONSERVATIVE
  RETIREMENT_FOCUSED
  PROPERTY_INVESTOR
  HIGH_INCOME
  
  // Country Specific
  TAX_OPTIMIZATION
  NEGATIVE_GEARING    // AU
  SUPER_STRATEGIES    // AU
  RETIREMENT_ACCOUNTS // US
  ISA_STRATEGIES      // UK
  PENSION_PLANNING    // UK
}
```

### Template Definitions

```typescript
// Australia Templates
export const AustraliaTemplates: StrategyTemplate[] = [
  {
    name: "Superannuation Boost",
    description: "Maximize retirement savings through salary sacrifice and government co-contributions",
    country: "AUSTRALIA",
    category: "SUPER_STRATEGIES", 
    complexity: "ELI5",
    tags: ["retirement", "tax-deduction", "government-bonus"],
    targetAgeGroup: "25-45",
    incomeRange: "50k-100k",
    
    investmentTemplate: [
      {
        type: "RETIREMENT_ACCOUNT",
        accountType: "SUPER",
        name: "Salary Sacrifice Super",
        monthlyContribution: 500, // $6k annually (under concessional cap)
        expectedReturn: 0.07,
        managementFees: 0.008,
        taxTreatment: "PRE_TAX",
        startYear: 0
      },
      {
        type: "ETF", 
        name: "Growth ETF Portfolio",
        monthlyContribution: 800,
        expectedReturn: 0.08,
        managementFees: 0.002,
        taxTreatment: "POST_TAX",
        startYear: 0
      }
    ],
    
    educationalContent: {
      overview: "Salary sacrifice into super reduces your taxable income while building long-term wealth with government support.",
      keyBenefits: [
        "Tax deduction on contributions up to $27,500 annually",
        "Government co-contribution up to $500 for eligible earners", 
        "Concessional 15% tax rate inside super",
        "Tax-free income after age 60"
      ],
      riskConsiderations: [
        "Money locked until preservation age (currently 60)",
        "Contribution caps - excess contributions taxed heavily",
        "Super guarantee rate changes affect employer contributions"
      ],
      taxImplications: "Contributions taxed at 15% in super vs marginal tax rate (potentially 32.5%+). Significant tax savings for middle-high income earners."
    }
  },
  
  {
    name: "Negative Gearing Property",
    description: "Build wealth through investment property with tax-deductible losses",
    country: "AUSTRALIA", 
    category: "NEGATIVE_GEARING",
    complexity: "INTERMEDIATE",
    tags: ["property", "tax-deduction", "rental-income"],
    targetAgeGroup: "30-50",
    incomeRange: "80k-200k",
    
    housingTemplate: {
      type: "BUY_INVESTMENT",
      name: "Investment Property",
      purchasePrice: 600000,
      deposit: 120000, // 20%
      mortgageAmount: 480000,
      mortgageRate: 0.055,
      mortgageTerm: 30,
      rentalIncome: 520, // per week
      propertyGrowthRate: 0.05,
      maintenanceCost: 2000, // annually
      managementFees: 520, // 2 weeks rent
      enableNegativeGearing: true
    },
    
    educationalContent: {
      overview: "Offset rental losses against your taxable income while building equity through capital growth.",
      keyBenefits: [
        "Tax deductions for interest, maintenance, and depreciation",
        "Capital growth builds long-term wealth", 
        "Rental income provides cash flow over time",
        "50% CGT discount if held >12 months"
      ],
      riskConsiderations: [
        "Property values can decline",
        "Vacancy periods reduce rental income",
        "Interest rate rises increase holding costs",
        "Depreciation benefits reduce over time"
      ],
      taxImplications: "Rental losses reduce taxable income. On sale, capital gains taxed at marginal rate with 50% discount if held >12 months."
    }
  }
]

// USA Templates  
export const USATemplates: StrategyTemplate[] = [
  {
    name: "Roth Ladder Strategy", 
    description: "Optimize retirement taxes by converting Traditional IRA to Roth over time",
    country: "USA",
    category: "RETIREMENT_ACCOUNTS",
    complexity: "EXPERT", 
    tags: ["roth-conversion", "tax-planning", "retirement"],
    targetAgeGroup: "35-55",
    incomeRange: "100k-200k",
    
    investmentTemplate: [
      {
        type: "RETIREMENT_ACCOUNT",
        accountType: "TRADITIONAL_401K", 
        name: "Current 401k Contributions",
        monthlyContribution: 1250, // $15k annually  
        employerMatch: 625, // 50% match up to $7.5k
        expectedReturn: 0.07,
        managementFees: 0.005,
        taxTreatment: "PRE_TAX",
        startYear: 0
      },
      {
        type: "RETIREMENT_ACCOUNT",
        accountType: "ROTH_IRA",
        name: "Roth Conversion Ladder",
        monthlyContribution: 833, // $10k conversions annually
        expectedReturn: 0.07, 
        managementFees: 0.003,
        taxTreatment: "POST_TAX",
        startYear: 0
      }
    ],
    
    educationalContent: {
      overview: "Systematically convert Traditional retirement funds to Roth to minimize lifetime taxes and create tax-free retirement income.",
      keyBenefits: [
        "Tax-free withdrawals in retirement",
        "No required minimum distributions (RMDs)", 
        "Tax diversification in retirement",
        "Estate planning benefits for heirs"
      ],
      riskConsiderations: [
        "Pay taxes now on conversions",
        "Tax rate assumptions may be wrong",
        "5-year waiting period for each conversion",
        "Income limits for direct Roth contributions"
      ],
      taxImplications: "Conversions count as taxable income. Best done in lower income years or systematically to stay in favorable tax brackets."
    }
  },
  
  {
    name: "Tax-Loss Harvesting",
    description: "Optimize after-tax investing with systematic loss realization",
    country: "USA",
    category: "TAX_OPTIMIZATION", 
    complexity: "INTERMEDIATE",
    tags: ["tax-efficiency", "capital-gains", "after-tax"],
    targetAgeGroup: "30-50", 
    incomeRange: "100k+",
    
    investmentTemplate: [
      {
        type: "ETF",
        name: "Total Stock Market (VTI)",
        monthlyContribution: 2000,
        expectedReturn: 0.08,
        managementFees: 0.0003,
        taxTreatment: "POST_TAX",
        startYear: 0
      },
      {
        type: "ETF", 
        name: "International Stock (VTIAX)",
        monthlyContribution: 1000,
        expectedReturn: 0.075,
        managementFees: 0.0005,
        taxTreatment: "POST_TAX", 
        startYear: 0
      }
    ],
    
    educationalContent: {
      overview: "Systematically realize investment losses to offset gains and reduce tax drag on portfolio growth.",
      keyBenefits: [
        "Offset capital gains with realized losses",
        "Deduct up to $3,000 losses against ordinary income", 
        "Carry forward unused losses indefinitely",
        "Maintain market exposure with similar investments"
      ],
      riskConsiderations: [
        "Wash sale rule prevents buying same security within 30 days",
        "Tracking cost basis becomes complex",
        "May force suboptimal investment decisions",
        "Long-term capital gains rates may be low"
      ],
      taxImplications: "Realized losses offset capital gains dollar-for-dollar. Excess losses reduce ordinary income up to $3,000 annually."
    }
  }
]

// UK Templates
export const UKTemplates: StrategyTemplate[] = [
  {
    name: "ISA Maximization",
    description: "Efficiently use annual ISA allowances for tax-free growth",
    country: "UK",
    category: "ISA_STRATEGIES",
    complexity: "ELI5", 
    tags: ["isa", "tax-free", "allowances"],
    targetAgeGroup: "25-65",
    incomeRange: "25k+",
    
    investmentTemplate: [
      {
        type: "RETIREMENT_ACCOUNT",
        accountType: "STOCKS_ISA",
        name: "Stocks & Shares ISA", 
        monthlyContribution: 1667, // £20k annually
        expectedReturn: 0.07,
        managementFees: 0.002,
        taxTreatment: "TAX_FREE",
        contributionLimit: 20000,
        startYear: 0
      }
    ],
    
    educationalContent: {
      overview: "Use your full ISA allowance for completely tax-free investment growth and income.",
      keyBenefits: [
        "No capital gains tax on profits",
        "No income tax on dividends", 
        "£20,000 annual allowance (2023/24)",
        "Funds accessible without penalties"
      ],
      riskConsiderations: [
        "Allowance is use-it-or-lose-it annually",
        "Investment risk in stocks & shares ISA",
        "No tax relief on contributions",
        "Cash ISA offers lower returns but capital protection"
      ],
      taxImplications: "All growth and income within ISA is completely tax-free. No reporting required to HMRC."
    }
  },
  
  {
    name: "Pension Tax Relief Optimization",
    description: "Maximize pension contributions to reclaim higher-rate tax relief",
    country: "UK", 
    category: "PENSION_PLANNING",
    complexity: "INTERMEDIATE",
    tags: ["pension", "tax-relief", "higher-rate"],
    targetAgeGroup: "35-55",
    incomeRange: "50k-150k",
    
    investmentTemplate: [
      {
        type: "RETIREMENT_ACCOUNT",
        accountType: "WORKPLACE_PENSION",
        name: "Workplace Pension",
        monthlyContribution: 800, // 8% of £120k salary
        employerMatch: 400, // 4% employer contribution
        expectedReturn: 0.06,
        managementFees: 0.0075,
        taxTreatment: "PRE_TAX",
        startYear: 0
      },
      {
        type: "RETIREMENT_ACCOUNT", 
        accountType: "SIPP",
        name: "Additional SIPP Contributions",
        monthlyContribution: 500, // Top up to annual allowance
        expectedReturn: 0.065,
        managementFees: 0.004,
        taxTreatment: "PRE_TAX",
        startYear: 0
      }
    ],
    
    educationalContent: {
      overview: "Maximize pension contributions to reduce taxable income and reclaim higher-rate tax relief.",
      keyBenefits: [
        "20% basic rate tax relief automatic",
        "Additional 20% relief for higher-rate taxpayers",
        "£40,000 annual allowance (2023/24)",
        "Employer contributions don't count toward your limit"
      ],
      riskConsiderations: [
        "Money locked until minimum pension age (57, rising to 58)",
        "Annual allowance tapering for high earners (>£240k)",
        "Lifetime allowance charges (currently £1.073m)",
        "Investment risk within pension wrapper"
      ],
      taxImplications: "Contributions reduce taxable income. Higher-rate taxpayers claim additional relief via self-assessment or tax code adjustment."
    }
  }
]
```

### Template Engine

```typescript
export class TemplateEngine {
  constructor(private taxEngine: ITaxEngine) {}
  
  async getTemplatesForUser(user: User): Promise<StrategyTemplate[]> {
    const { country, complexityLevel, income, age } = user
    
    const templates = await this.getTemplatesByCountry(country)
    
    return templates.filter(template => 
      this.isTemplateAppropriate(template, user)
    ).sort((a, b) => this.scoreTemplate(b, user) - this.scoreTemplate(a, user))
  }
  
  private isTemplateAppropriate(template: StrategyTemplate, user: User): boolean {
    // Complexity filter
    if (user.complexityLevel === 'ELI5' && template.complexity === 'EXPERT') {
      return false
    }
    
    // Age group filter
    const userAge = user.age || 30
    const ageGroups = {
      '25-35': [25, 35],
      '35-45': [35, 45], 
      '45-55': [45, 55],
      '55+': [55, 100]
    }
    
    const [minAge, maxAge] = ageGroups[template.targetAgeGroup] || [0, 100]
    if (userAge < minAge || userAge > maxAge) {
      return false
    }
    
    // Income range filter (optional)
    if (template.incomeRange && user.income) {
      const incomeRanges = {
        '0-50k': [0, 50000],
        '50k-100k': [50000, 100000],
        '100k-200k': [100000, 200000], 
        '200k+': [200000, Infinity]
      }
      
      const [minIncome, maxIncome] = incomeRanges[template.incomeRange] || [0, Infinity]
      if (user.income < minIncome || user.income > maxIncome) {
        return false
      }
    }
    
    return true
  }
  
  private scoreTemplate(template: StrategyTemplate, user: User): number {
    let score = 0
    
    // Complexity match
    if (template.complexity === user.complexityLevel) {
      score += 10
    }
    
    // Usage popularity
    score += Math.min(template.usageCount / 100, 5)
    
    // Rating
    score += (template.averageRating || 0) * 2
    
    // Recent validation
    if (template.lastValidated && 
        Date.now() - template.lastValidated.getTime() < 365 * 24 * 60 * 60 * 1000) {
      score += 3
    }
    
    return score
  }
  
  async applyTemplate(templateId: string, user: User, customizations: TemplateCustomizations = {}): Promise<Scenario> {
    const template = await this.getTemplate(templateId)
    if (!template) throw new Error('Template not found')
    
    // Apply user-specific customizations
    const customizedInvestments = this.customizeInvestments(
      template.investmentTemplate,
      user,
      customizations
    )
    
    const customizedHousing = template.housingTemplate ? 
      this.customizeHousing(template.housingTemplate, user, customizations) : null
    
    // Create scenario from template
    const scenario: Partial<Scenario> = {
      name: customizations.name || `${template.name} - ${new Date().toLocaleDateString()}`,
      description: customizations.description || template.description,
      country: user.primaryCountry,
      currency: getCurrencyForCountry(user.primaryCountry),
      timeHorizonYears: customizations.timeHorizon || template.defaultTimeHorizon,
      currentAge: user.age || 30,
      currentIncome: user.income || 0,
      currentSavings: user.savings || 0,
      currentDebt: user.debt || 0,
      parentScenarioId: templateId,
      housingComponents: customizedHousing ? [customizedHousing] : [],
      investmentComponents: customizedInvestments
    }
    
    return scenario as Scenario
  }
  
  private customizeInvestments(
    template: any[], 
    user: User, 
    customizations: TemplateCustomizations
  ): InvestmentComponent[] {
    return template.map(investment => ({
      ...investment,
      monthlyContribution: customizations.monthlyBudget ? 
        this.allocateContribution(investment, customizations.monthlyBudget, template) :
        investment.monthlyContribution,
      expectedReturn: customizations.riskTolerance === 'conservative' ?
        investment.expectedReturn * 0.8 :
        customizations.riskTolerance === 'aggressive' ?
        investment.expectedReturn * 1.2 :
        investment.expectedReturn
    }))
  }
  
  async validateTemplate(template: StrategyTemplate): Promise<TemplateValidation> {
    const validationResults: TemplateValidation = {
      templateId: template.id,
      isValid: true,
      warnings: [],
      errors: []
    }
    
    // Validate tax calculations
    const taxValidation = await this.taxEngine.validateTemplate(template)
    if (!taxValidation.isValid) {
      validationResults.isValid = false
      validationResults.errors.push(...taxValidation.errors)
    }
    
    // Validate contribution limits
    const limitsValidation = this.validateContributionLimits(template)
    if (!limitsValidation.isValid) {
      validationResults.warnings.push(...limitsValidation.warnings)
    }
    
    // Validate investment allocations
    const allocationValidation = this.validateAllocations(template)
    if (!allocationValidation.isValid) {
      validationResults.warnings.push(...allocationValidation.warnings)
    }
    
    return validationResults
  }
}

interface TemplateCustomizations {
  name?: string
  description?: string
  timeHorizon?: number
  monthlyBudget?: number
  riskTolerance?: 'conservative' | 'moderate' | 'aggressive'
  specificGoals?: string[]
}

interface TemplateValidation {
  templateId: string
  isValid: boolean
  warnings: string[]
  errors: string[]
}
```

## Multi-Country Requirements

### Australia Templates
- **Superannuation Strategies**: Salary sacrifice, co-contributions, transition to retirement
- **Negative Gearing**: Property investment with tax-deductible losses
- **Franking Credits**: Dividend-focused portfolios for retirees
- **First Home Buyer**: FHSS scheme and stamp duty optimization
- **SMSFs**: Self-managed super for high-net-worth individuals

### USA Templates  
- **401k Optimization**: Traditional vs Roth, employer matching maximization
- **Roth Conversion Ladders**: Strategic tax planning for early retirement
- **Tax-Loss Harvesting**: After-tax investment optimization
- **HSA Triple Advantage**: Health savings account maximization
- **Backdoor Roth**: High-income Roth access strategies

### UK Templates
- **ISA Maximization**: Stocks & Shares vs Cash ISA optimization
- **Pension Tax Relief**: Higher-rate tax relief recapture
- **Buy-to-Let**: Property investment with mortgage interest relief
- **Help to Buy**: Government scheme optimization
- **SIPP Strategies**: Self-invested personal pension planning

### Cross-Border Templates
- **Expat Optimization**: Tax-efficient strategies for overseas residents
- **Repatriation Planning**: Moving funds between countries efficiently
- **Dual Tax Residency**: Managing obligations in multiple countries
- **International Pensions**: Cross-border retirement planning

## User Interface Requirements

### Adaptive Complexity

**ELI5 Mode**:
- Simple template cards with plain English descriptions
- Guided questionnaire to recommend templates
- One-click template application with minimal customization
- Visual indicators showing "why this works for you"

**Intermediate Mode**:
- Template categories with filtering options
- Side-by-side template comparison
- Basic customization sliders before applying
- Educational tooltips explaining tax benefits

**Expert Mode**:
- Full template library with advanced filtering
- Detailed parameter customization before application
- Template validation and optimization suggestions
- Ability to create and save custom templates

### Components

```tsx
// Template Gallery Component
export function TemplateGallery() {
  const { data: templates } = useQuery(getRecommendedTemplates)
  const [selectedCategory, setSelectedCategory] = useState<TemplateCategory | null>(null)
  const [complexityFilter, setComplexityFilter] = useState<ComplexityLevel | null>(null)
  
  return (
    <div className="space-y-6">
      <TemplateFilters 
        selectedCategory={selectedCategory}
        onCategoryChange={setSelectedCategory}
        complexityFilter={complexityFilter}
        onComplexityChange={setComplexityFilter}
      />
      
      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {templates?.map(template => (
          <TemplateCard 
            key={template.id}
            template={template}
            onSelect={(template) => handleTemplateSelect(template)}
          />
        ))}
      </div>
    </div>
  )
}

// Template Customization Modal
export function TemplateCustomizationModal({ 
  template, 
  isOpen, 
  onClose, 
  onApply 
}: TemplateCustomizationModalProps) {
  const [customizations, setCustomizations] = useState<TemplateCustomizations>({})
  
  return (
    <Dialog open={isOpen} onOpenChange={onClose}>
      <DialogContent className="max-w-4xl">
        <DialogHeader>
          <DialogTitle>Customize: {template.name}</DialogTitle>
          <DialogDescription>{template.description}</DialogDescription>
        </DialogHeader>
        
        <div className="space-y-6">
          <TemplateOverview template={template} />
          
          <CustomizationControls 
            template={template}
            customizations={customizations}
            onChange={setCustomizations}
          />
          
          <TemplatePreview 
            template={template}
            customizations={customizations}
          />
        </div>
        
        <DialogFooter>
          <Button variant="outline" onClick={onClose}>Cancel</Button>
          <Button onClick={() => onApply(template, customizations)}>
            Create Scenario
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  )
}

// Template Card Component
export function TemplateCard({ template, onSelect }: TemplateCardProps) {
  return (
    <Card className="cursor-pointer hover:shadow-lg transition-shadow" onClick={() => onSelect(template)}>
      <CardHeader>
        <div className="flex justify-between items-start">
          <CardTitle className="text-lg">{template.name}</CardTitle>
          <div className="flex gap-2">
            <ComplexityBadge level={template.complexity} />
            <CountryFlag country={template.country} />
          </div>
        </div>
        <CardDescription>{template.description}</CardDescription>
      </CardHeader>
      
      <CardContent>
        <div className="space-y-3">
          <div className="flex flex-wrap gap-1">
            {template.tags.map(tag => (
              <Badge key={tag} variant="secondary" className="text-xs">
                {tag}
              </Badge>
            ))}
          </div>
          
          <div className="text-sm text-muted-foreground">
            <div>Target Age: {template.targetAgeGroup}</div>
            <div>Income Range: {template.incomeRange}</div>
            <div>Used by {template.usageCount.toLocaleString()} people</div>
          </div>
          
          {template.averageRating && (
            <div className="flex items-center gap-1">
              <StarRating rating={template.averageRating} />
              <span className="text-sm text-muted-foreground">
                ({template.averageRating.toFixed(1)})
              </span>
            </div>
          )}
        </div>
      </CardContent>
    </Card>
  )
}
```

### User Flows

**Template Discovery**:
1. User views recommended templates based on profile
2. Browse templates by category or search
3. Filter by complexity level and country
4. Read template details and educational content
5. Preview template configuration

**Template Application**:
1. Select template from gallery
2. Customize parameters in modal (optional)
3. Preview projected outcomes
4. Create scenario from template
5. Further customize in scenario builder

**Template Comparison**:
1. Select multiple templates for comparison
2. View side-by-side projections
3. Understand trade-offs between strategies
4. Create scenarios from preferred templates

## Implementation Details

### File Structure
```
src/
├── template/
│   ├── client/
│   │   ├── TemplateGallery.tsx
│   │   ├── TemplateCustomization.tsx
│   │   ├── TemplateComparison.tsx
│   │   └── components/
│   │       ├── TemplateCard.tsx
│   │       ├── TemplateFilters.tsx
│   │       └── TemplateEducation.tsx
│   ├── server/
│   │   ├── queries.ts
│   │   ├── actions.ts
│   │   ├── TemplateEngine.ts
│   │   ├── templates/
│   │   │   ├── AustraliaTemplates.ts
│   │   │   ├── USATemplates.ts
│   │   │   ├── UKTemplates.ts
│   │   │   └── CrossBorderTemplates.ts
│   │   └── validation/
│   │       ├── TemplateValidator.ts
│   │       └── TaxComplianceValidator.ts
│   └── shared/
│       ├── types.ts
│       └── constants.ts
```

### Template Validation Requirements

```typescript
export interface TemplateValidator {
  validateTaxCompliance(template: StrategyTemplate): Promise<ValidationResult>
  validateContributionLimits(template: StrategyTemplate): ValidationResult  
  validateInvestmentAllocations(template: StrategyTemplate): ValidationResult
  validateEducationalContent(template: StrategyTemplate): ValidationResult
}

export class AustraliaTemplateValidator implements TemplateValidator {
  async validateTaxCompliance(template: StrategyTemplate): Promise<ValidationResult> {
    const results: ValidationResult = { isValid: true, errors: [], warnings: [] }
    
    // Validate superannuation contribution limits
    const superContributions = this.extractSuperContributions(template)
    if (superContributions.concessional > 27500) {
      results.errors.push('Concessional super contributions exceed ATO limit of $27,500')
      results.isValid = false
    }
    
    // Validate negative gearing calculations
    const housingComponents = template.housingTemplate
    if (housingComponents?.enableNegativeGearing) {
      if (!this.hasValidNegativeGearingSetup(housingComponents)) {
        results.warnings.push('Negative gearing setup may not generate tax benefits')
      }
    }
    
    return results
  }
}
```

### Testing Requirements

**Template Accuracy Testing**:
- Validate all templates produce ATO/IRS/HMRC compliant calculations
- Test template customization edge cases
- Verify educational content accuracy with tax professionals
- Integration testing with scenario calculation engine

**Multi-Country Testing**:
- Country-specific template validation suites
- Cross-border template accuracy testing
- Currency conversion in template applications
- Regulatory compliance across all jurisdictions

## Success Criteria

### Acceptance Criteria
- [ ] 95% of new users successfully apply a template within first session
- [ ] Templates available for all major financial strategies per country
- [ ] Template applications create valid scenarios with accurate projections
- [ ] Educational content helps users understand strategy implications
- [ ] Template recommendations match user profiles with 90% satisfaction
- [ ] All templates validated by qualified tax professionals

### Performance Requirements
- Template gallery loads within 1 second
- Template customization updates in real-time (<200ms)
- Template application creates scenarios within 3 seconds
- Educational content loads progressively without blocking

### Compliance Requirements
- All templates reviewed and validated by country-specific tax professionals
- Template calculations match professional financial planning software
- Educational disclaimers meet regulatory requirements for financial education
- Template versioning tracks regulatory changes and updates