# Advanced Analytics Dashboard

## Feature Overview

**Purpose**: Sophisticated analytics dashboard providing Monte Carlo simulations, scenario comparisons, stress testing, and predictive modeling to give users deep insights into their financial strategies with stunning visualizations and expert-level analysis.

**User Stories**:
- As an intermediate user, I want to see probability-based projections to understand the likelihood of reaching my goals
- As an expert user, I want to run Monte Carlo simulations with custom volatility assumptions
- As any user, I want beautiful, intuitive charts that make complex financial data easy to understand
- As a financial advisor, I want detailed analytics to validate client strategies

**Priority**: P1 (High) - Essential for "stunning and easy to use" experience

**Dependencies**: Scenario Management, Investment Portfolio, Housing Strategies

## Technical Specifications

### Data Models

```prisma
model AnalyticsSession {
  id                  String   @id @default(cuid())
  userId              String
  user                User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Session Details
  name                String
  description         String?
  scenarioIds         String[] // Scenarios being analyzed
  
  // Analysis Configuration
  analysisType        AnalysisType
  timeHorizon         Int      @default(30) // Years
  confidenceLevel     Decimal  @default(0.95) // 95% confidence interval
  iterations          Int      @default(10000) // Monte Carlo iterations
  
  // Results Cache
  results             Json?
  charts              Json?    // Chart configuration and data
  insights            Json?    // AI-generated insights
  
  // Metadata
  lastCalculated      DateTime?
  calculationTime     Decimal? // Seconds taken
  
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
  
  @@map("analytics_sessions")
}

model MonteCarloResult {
  id                  String   @id @default(cuid())
  sessionId           String
  session             AnalyticsSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  
  // Simulation Parameters
  iterations          Int
  timeHorizon         Int
  
  // Results by Year
  yearlyResults       MonteCarloYearResult[]
  
  // Summary Statistics
  finalValuePercentiles Json // P10, P25, P50, P75, P90, P95, P99
  successProbability  Decimal  // Probability of reaching goal
  shortfallRisk       Decimal  // Probability of running out of money
  
  // Risk Metrics
  valueAtRisk         Decimal  // VaR at 95% confidence
  conditionalVaR      Decimal  // Expected shortfall
  maxDrawdown         Decimal  // Maximum portfolio decline
  
  createdAt           DateTime @default(now())
  
  @@map("monte_carlo_results")
}

model MonteCarloYearResult {
  id                  String   @id @default(cuid())
  resultId            String
  result              MonteCarloResult @relation(fields: [resultId], references: [id], onDelete: Cascade)
  
  year                Int
  
  // Percentile Results
  p10                 Decimal  // 10th percentile outcome
  p25                 Decimal  // 25th percentile outcome
  p50                 Decimal  // Median outcome
  p75                 Decimal  // 75th percentile outcome
  p90                 Decimal  // 90th percentile outcome
  
  // Additional Statistics
  mean                Decimal
  standardDeviation   Decimal
  skewness            Decimal?
  kurtosis            Decimal?
  
  @@unique([resultId, year])
  @@map("monte_carlo_year_results")
}

model StressTestResult {
  id                  String   @id @default(cuid())
  sessionId           String
  session             AnalyticsSession @relation(fields: [sessionId], references: [id], onDelete: Cascade)
  
  // Stress Test Scenarios
  scenario            StressTestScenario
  description         String
  
  // Economic Assumptions
  stockMarketDecline  Decimal? // -20%, -50%, etc.
  interestRateChange  Decimal? // +2%, -1%, etc.
  inflationRate       Decimal? // 6%, 10%, etc.
  unemploymentRate    Decimal? // 15%, 20%, etc.
  
  // Results
  portfolioImpact     Decimal  // Final portfolio value impact
  goalProbability     Decimal  // Adjusted probability of success
  timeToRecovery      Int?     // Years to recover to baseline
  
  // Recommendations
  mitigationStrategies Json    // Suggested adjustments
  
  createdAt           DateTime @default(now())
  
  @@map("stress_test_results")
}

enum AnalysisType {
  MONTE_CARLO
  DETERMINISTIC
  STRESS_TEST
  SCENARIO_COMPARISON
  SENSITIVITY_ANALYSIS
  GOAL_PROBABILITY
}

enum StressTestScenario {
  GREAT_RECESSION_2008
  DOT_COM_CRASH_2000
  HIGH_INFLATION_1970S
  INTEREST_RATE_SHOCK
  PROLONGED_RECESSION
  CUSTOM
}
```

### Analytics Engine

```typescript
export class AnalyticsEngine {
  constructor(
    private scenarioCalculator: ScenarioCalculator,
    private marketDataService: MarketDataService
  ) {}
  
  async runMonteCarloSimulation(
    scenarios: Scenario[],
    config: MonteCarloConfig
  ): Promise<MonteCarloResult> {
    
    const startTime = performance.now()
    const results: MonteCarloIteration[] = []
    
    // Run Monte Carlo iterations
    for (let i = 0; i < config.iterations; i++) {
      const iteration = await this.runSingleIteration(scenarios, config, i)
      results.push(iteration)
      
      // Progress callback for UI updates
      if (i % 100 === 0 && config.onProgress) {
        config.onProgress(i / config.iterations)
      }
    }
    
    // Calculate summary statistics
    const summaryStats = this.calculateSummaryStatistics(results, config)
    
    const endTime = performance.now()
    const calculationTime = (endTime - startTime) / 1000
    
    return {
      iterations: config.iterations,
      timeHorizon: config.timeHorizon,
      yearlyResults: summaryStats.yearlyResults,
      finalValuePercentiles: summaryStats.finalValuePercentiles,
      successProbability: summaryStats.successProbability,
      shortfallRisk: summaryStats.shortfallRisk,
      valueAtRisk: summaryStats.valueAtRisk,
      conditionalVaR: summaryStats.conditionalVaR,
      maxDrawdown: summaryStats.maxDrawdown,
      calculationTime
    }
  }
  
  private async runSingleIteration(
    scenarios: Scenario[],
    config: MonteCarloConfig,
    iteration: number
  ): Promise<MonteCarloIteration> {
    
    const results: number[] = []
    
    for (let year = 0; year <= config.timeHorizon; year++) {
      // Generate random market returns based on historical volatility
      const marketConditions = this.generateMarketConditions(year, iteration)
      
      // Calculate scenario value with random returns
      let totalValue = 0
      for (const scenario of scenarios) {
        const scenarioValue = await this.calculateScenarioWithRandomReturns(
          scenario,
          year,
          marketConditions
        )
        totalValue += scenarioValue
      }
      
      results.push(totalValue)
    }
    
    return {
      iteration,
      yearlyValues: results,
      finalValue: results[results.length - 1]
    }
  }
  
  private generateMarketConditions(year: number, seed: number): MarketConditions {
    // Correlated random number generation for different asset classes
    const random = this.createSeededRandom(seed + year)
    
    return {
      stockReturns: this.generateRandomReturn(0.10, 0.16, random), // 10% mean, 16% volatility
      bondReturns: this.generateRandomReturn(0.04, 0.05, random), // 4% mean, 5% volatility
      realEstateReturns: this.generateRandomReturn(0.08, 0.12, random),
      inflationRate: this.generateRandomReturn(0.025, 0.015, random),
      interestRates: this.generateRandomReturn(0.04, 0.02, random)
    }
  }
  
  private generateRandomReturn(mean: number, volatility: number, random: () => number): number {
    // Box-Muller transform for normal distribution
    const u1 = random()
    const u2 = random()
    const z0 = Math.sqrt(-2 * Math.log(u1)) * Math.cos(2 * Math.PI * u2)
    
    return mean + volatility * z0
  }
  
  async runStressTest(
    scenarios: Scenario[],
    stressScenario: StressTestScenario
  ): Promise<StressTestResult> {
    
    const stressParameters = this.getStressTestParameters(stressScenario)
    
    // Calculate baseline performance
    const baselineResults = await this.calculateDeterministicProjection(scenarios)
    
    // Apply stress conditions
    const stressedResults = await this.calculateStressedProjection(
      scenarios,
      stressParameters
    )
    
    // Calculate impact
    const portfolioImpact = (stressedResults.finalValue - baselineResults.finalValue) / baselineResults.finalValue
    const goalProbability = this.calculateGoalProbabilityUnderStress(stressedResults, scenarios)
    
    // Generate mitigation strategies
    const mitigationStrategies = this.generateMitigationStrategies(
      portfolioImpact,
      stressParameters
    )
    
    return {
      scenario: stressScenario,
      description: this.getStressScenarioDescription(stressScenario),
      portfolioImpact,
      goalProbability,
      timeToRecovery: this.calculateRecoveryTime(stressedResults, baselineResults),
      mitigationStrategies
    }
  }
  
  private getStressTestParameters(scenario: StressTestScenario): StressTestParameters {
    const parameters = {
      GREAT_RECESSION_2008: {
        stockMarketDecline: -0.37,
        bondReturns: 0.05,
        realEstateDecline: -0.19,
        unemploymentRate: 0.10,
        duration: 2
      },
      DOT_COM_CRASH_2000: {
        stockMarketDecline: -0.49,
        bondReturns: 0.08,
        realEstateDecline: 0.05,
        unemploymentRate: 0.063,
        duration: 3
      },
      HIGH_INFLATION_1970S: {
        stockMarketDecline: -0.07,
        bondReturns: -0.02,
        inflationRate: 0.09,
        realEstateGains: 0.12,
        duration: 5
      }
    }
    
    return parameters[scenario] || parameters.GREAT_RECESSION_2008
  }
  
  generateInsights(
    results: MonteCarloResult,
    scenarios: Scenario[]
  ): AnalyticsInsights {
    
    const insights: AnalyticsInsights = {
      keyFindings: [],
      recommendations: [],
      riskAssessment: '',
      optimizationSuggestions: []
    }
    
    // Analyze success probability
    if (results.successProbability < 0.8) {
      insights.keyFindings.push({
        type: 'WARNING',
        message: `${(results.successProbability * 100).toFixed(1)}% chance of reaching your goal`,
        impact: 'HIGH'
      })
      
      insights.recommendations.push({
        action: 'INCREASE_SAVINGS',
        description: 'Consider increasing monthly contributions by 20%',
        expectedImpact: 'Could improve success probability to 85%'
      })
    }
    
    // Analyze risk metrics
    if (results.maxDrawdown > 0.5) {
      insights.keyFindings.push({
        type: 'RISK',
        message: `Portfolio could decline by up to ${(results.maxDrawdown * 100).toFixed(1)}%`,
        impact: 'MEDIUM'
      })
      
      insights.recommendations.push({
        action: 'REDUCE_RISK',
        description: 'Consider reducing equity allocation by 10%',
        expectedImpact: 'Lower volatility but slightly reduced expected returns'
      })
    }
    
    // Analyze asset allocation
    const allocationAnalysis = this.analyzeAssetAllocation(scenarios)
    insights.optimizationSuggestions.push(...allocationAnalysis)
    
    return insights
  }
}

interface MonteCarloConfig {
  iterations: number
  timeHorizon: number
  confidenceLevel: number
  goalAmount?: number
  onProgress?: (progress: number) => void
}

interface MonteCarloIteration {
  iteration: number
  yearlyValues: number[]
  finalValue: number
}

interface MarketConditions {
  stockReturns: number
  bondReturns: number
  realEstateReturns: number
  inflationRate: number
  interestRates: number
}

interface AnalyticsInsights {
  keyFindings: Finding[]
  recommendations: Recommendation[]
  riskAssessment: string
  optimizationSuggestions: OptimizationSuggestion[]
}

interface Finding {
  type: 'SUCCESS' | 'WARNING' | 'RISK' | 'OPPORTUNITY'
  message: string
  impact: 'LOW' | 'MEDIUM' | 'HIGH'
}

interface Recommendation {
  action: string
  description: string
  expectedImpact: string
}
```

### Chart Configuration System

```typescript
export class ChartConfigurationService {
  
  generateMonteCarloChart(results: MonteCarloResult): ChartConfig {
    return {
      type: 'fan_chart',
      title: 'Portfolio Projections with Confidence Intervals',
      data: {
        years: Array.from({ length: results.timeHorizon + 1 }, (_, i) => i),
        percentiles: {
          p10: results.yearlyResults.map(r => r.p10),
          p25: results.yearlyResults.map(r => r.p25),
          p50: results.yearlyResults.map(r => r.p50),
          p75: results.yearlyResults.map(r => r.p75),
          p90: results.yearlyResults.map(r => r.p90)
        }
      },
      styling: {
        colors: {
          p10: '#fee2e2',  // Light red
          p25: '#fecaca',  // Medium light red
          p50: '#3b82f6',  // Blue (median)
          p75: '#bbf7d0',  // Medium light green
          p90: '#dcfce7'   // Light green
        },
        opacity: {
          bands: 0.3,
          median: 1.0
        }
      },
      annotations: [
        {
          type: 'line',
          value: results.goalAmount || 1000000,
          label: 'Target Goal',
          style: { stroke: '#ef4444', strokeDasharray: '5,5' }
        }
      ]
    }
  }
  
  generateScenarioComparisonChart(scenarios: Scenario[], results: ScenarioComparisonResult[]): ChartConfig {
    return {
      type: 'multi_line',
      title: 'Scenario Comparison Over Time',
      data: {
        years: Array.from({ length: 31 }, (_, i) => i),
        series: scenarios.map((scenario, index) => ({
          name: scenario.name,
          data: results[index].yearlyValues,
          color: this.getScenarioColor(index)
        }))
      },
      styling: {
        legend: {
          position: 'top',
          align: 'left'
        },
        tooltip: {
          shared: true,
          formatter: (point: any) => `Year ${point.x}: $${point.y.toLocaleString()}`
        }
      }
    }
  }
  
  generateRiskReturnChart(portfolios: Portfolio[]): ChartConfig {
    return {
      type: 'scatter',
      title: 'Risk vs Return Analysis',
      data: {
        series: [{
          name: 'Portfolios',
          data: portfolios.map(p => ({
            x: p.expectedRisk,
            y: p.expectedReturn,
            name: p.name,
            size: p.totalValue
          }))
        }]
      },
      axes: {
        x: {
          title: 'Risk (Standard Deviation)',
          format: 'percentage'
        },
        y: {
          title: 'Expected Return',
          format: 'percentage'
        }
      },
      styling: {
        bubbles: {
          sizeRange: [10, 50],
          opacity: 0.7
        }
      }
    }
  }
}

interface ChartConfig {
  type: string
  title: string
  data: any
  styling?: any
  annotations?: any[]
  axes?: any
}
```

## User Interface Requirements

### Adaptive Complexity

**ELI5 Mode**:
- Simple probability displays ("8 out of 10 times you'll reach your goal")
- Traffic light indicators (green/yellow/red for different scenarios)
- Basic charts with clear explanations
- One-click stress testing with plain English results

**Intermediate Mode**:
- Interactive Monte Carlo visualizations
- Scenario comparison side-by-side
- Risk tolerance adjustment with real-time updates
- Goal probability calculator

**Expert Mode**:
- Full Monte Carlo configuration (iterations, confidence levels)
- Custom stress test scenarios
- Advanced risk metrics (VaR, CVaR, Sharpe ratio)
- Correlation matrix analysis
- Custom time horizons and market assumptions

### Components

```tsx
// Main Analytics Dashboard
export function AnalyticsDashboard({ scenarios }: { scenarios: Scenario[] }) {
  const [analysisType, setAnalysisType] = useState<AnalysisType>('MONTE_CARLO')
  const [config, setConfig] = useState<AnalysisConfig>()
  
  return (
    <div className="space-y-6">
      <AnalysisTypeSelector 
        selectedType={analysisType}
        onTypeChange={setAnalysisType}
      />
      
      {analysisType === 'MONTE_CARLO' && (
        <MonteCarloAnalysis scenarios={scenarios} />
      )}
      
      {analysisType === 'STRESS_TEST' && (
        <StressTestAnalysis scenarios={scenarios} />
      )}
      
      {analysisType === 'SCENARIO_COMPARISON' && (
        <ScenarioComparison scenarios={scenarios} />
      )}
    </div>
  )
}

// Monte Carlo Visualization
export function MonteCarloAnalysis({ scenarios }: { scenarios: Scenario[] }) {
  const [config, setConfig] = useState<MonteCarloConfig>({
    iterations: 10000,
    timeHorizon: 30,
    confidenceLevel: 0.95
  })
  
  const { data: results, isLoading } = useQuery(
    runMonteCarloSimulation,
    { scenarios, config }
  )
  
  if (isLoading) {
    return <MonteCarloLoadingState progress={results?.progress} />
  }
  
  return (
    <div className="space-y-6">
      <MonteCarloChart results={results} />
      <ProbabilityMetrics results={results} />
      <RiskMetrics results={results} />
      <InsightsPanel insights={results.insights} />
    </div>
  )
}

// Interactive Fan Chart for Monte Carlo
export function MonteCarloChart({ results }: { results: MonteCarloResult }) {
  const chartConfig = useMemo(() => 
    generateMonteCarloChart(results), [results]
  )
  
  return (
    <Card>
      <CardHeader>
        <CardTitle>Portfolio Projections</CardTitle>
        <CardDescription>
          Showing {results.iterations.toLocaleString()} simulation outcomes
        </CardDescription>
      </CardHeader>
      <CardContent>
        <FanChart config={chartConfig} height={400} />
        <ChartLegend 
          items={[
            { color: '#fee2e2', label: '10th-90th percentile' },
            { color: '#fecaca', label: '25th-75th percentile' },
            { color: '#3b82f6', label: 'Median outcome' }
          ]}
        />
      </CardContent>
    </Card>
  )
}

// Stress Test Interface
export function StressTestAnalysis({ scenarios }: { scenarios: Scenario[] }) {
  const [selectedScenario, setSelectedScenario] = useState<StressTestScenario>('GREAT_RECESSION_2008')
  
  const { data: results } = useQuery(
    runStressTest,
    { scenarios, stressScenario: selectedScenario }
  )
  
  return (
    <div className="space-y-6">
      <StressScenarioSelector 
        selected={selectedScenario}
        onSelect={setSelectedScenario}
      />
      
      <StressTestResults results={results} />
      <MitigationStrategies strategies={results?.mitigationStrategies} />
    </div>
  )
}

// Beautiful Goal Probability Display
export function GoalProbabilityDisplay({ probability }: { probability: number }) {
  const getColor = (prob: number) => {
    if (prob >= 0.9) return 'text-green-600'
    if (prob >= 0.8) return 'text-blue-600'
    if (prob >= 0.7) return 'text-yellow-600'
    return 'text-red-600'
  }
  
  const getMessage = (prob: number) => {
    if (prob >= 0.9) return 'Excellent chance of success'
    if (prob >= 0.8) return 'Good chance of success'
    if (prob >= 0.7) return 'Moderate chance of success'
    return 'Consider adjusting your strategy'
  }
  
  return (
    <Card>
      <CardContent className="pt-6">
        <div className="text-center space-y-4">
          <div className={`text-6xl font-bold ${getColor(probability)}`}>
            {(probability * 100).toFixed(0)}%
          </div>
          <div className="text-lg text-muted-foreground">
            {getMessage(probability)}
          </div>
          <Progress value={probability * 100} className="w-full" />
        </div>
      </CardContent>
    </Card>
  )
}
```

## Implementation Details

### File Structure
```
src/
├── analytics/
│   ├── client/
│   │   ├── AnalyticsDashboard.tsx
│   │   ├── MonteCarloAnalysis.tsx
│   │   ├── StressTestAnalysis.tsx
│   │   ├── ScenarioComparison.tsx
│   │   ├── charts/
│   │   │   ├── FanChart.tsx
│   │   │   ├── MultiLineChart.tsx
│   │   │   ├── ScatterChart.tsx
│   │   │   └── ProbabilityChart.tsx
│   │   └── components/
│   │       ├── GoalProbabilityDisplay.tsx
│   │       ├── RiskMetrics.tsx
│   │       ├── InsightsPanel.tsx
│   │       └── MitigationStrategies.tsx
│   ├── server/
│   │   ├── queries.ts
│   │   ├── actions.ts
│   │   ├── engines/
│   │   │   ├── AnalyticsEngine.ts
│   │   │   ├── MonteCarloEngine.ts
│   │   │   └── StressTestEngine.ts
│   │   ├── services/
│   │   │   ├── ChartConfigurationService.ts
│   │   │   ├── InsightsGenerator.ts
│   │   │   └── StatisticsCalculator.ts
│   │   └── workers/
│   │       └── MonteCarloWorker.ts
│   └── shared/
│       ├── types.ts
│       └── constants.ts
```

### Performance Optimization

```typescript
// Web Worker for Monte Carlo calculations
export class MonteCarloWorker {
  private worker: Worker
  
  constructor() {
    this.worker = new Worker('/workers/monte-carlo.js')
  }
  
  async runSimulation(config: MonteCarloConfig): Promise<MonteCarloResult> {
    return new Promise((resolve, reject) => {
      this.worker.onmessage = (event) => {
        if (event.data.type === 'result') {
          resolve(event.data.result)
        } else if (event.data.type === 'error') {
          reject(new Error(event.data.message))
        }
      }
      
      this.worker.postMessage({
        type: 'run_simulation',
        config
      })
    })
  }
}

// Progressive calculation with caching
export class ProgressiveCalculationService {
  private cache = new Map<string, any>()
  
  async calculateWithProgress<T>(
    key: string,
    calculation: () => Promise<T>,
    onProgress: (progress: number) => void
  ): Promise<T> {
    
    // Check cache first
    if (this.cache.has(key)) {
      onProgress(1.0)
      return this.cache.get(key)
    }
    
    // Run calculation with progress updates
    const result = await calculation()
    
    // Cache result
    this.cache.set(key, result)
    
    return result
  }
}
```

## Testing Requirements

### Calculation Accuracy
- Monte Carlo convergence testing with known distributions
- Stress test scenario validation against historical data
- Risk metric calculations verified against financial literature
- Chart data accuracy across all visualization types

### Performance Testing
- 10,000 iteration Monte Carlo simulation under 10 seconds
- Real-time chart updates under 100ms
- Memory usage optimization for large datasets
- Progressive loading for complex analytics

## Success Criteria

### Acceptance Criteria
- [ ] Monte Carlo simulations provide accurate probability distributions
- [ ] Stress testing reflects realistic market scenarios
- [ ] Charts are interactive, beautiful, and responsive
- [ ] Analytics generate actionable insights
- [ ] Performance meets sub-10-second requirements for complex calculations
- [ ] Progressive complexity works across ELI5 to Expert modes

### Performance Requirements
- Monte Carlo simulations (10k iterations) complete within 10 seconds
- Chart rendering and interactions under 100ms latency
- Real-time parameter updates with immediate visual feedback
- Support for analyzing 10+ scenarios simultaneously

### User Experience Requirements
- Stunning, professional-grade visualizations
- Intuitive progressive disclosure of complexity
- Clear, actionable insights and recommendations
- Seamless integration with scenario management
- Mobile-responsive design for all chart types