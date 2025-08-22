# Real-Time Calculation Engine Architecture

## Overview

The Real-Time Calculation Engine is the heart of FinPath Explorer's stunning user experience, providing sub-2-second financial calculations with progressive updates and seamless interactivity.

## Core Principles

### 1. **Progressive Calculation**
- Start with quick estimates, refine to full precision
- Show intermediate results while calculations continue
- Prioritize user-visible metrics first

### 2. **Intelligent Caching**
- Cache calculation results at multiple granularities
- Invalidate smartly when inputs change
- Precompute common scenarios

### 3. **Parallel Processing**
- Use Web Workers for heavy calculations
- Parallelize Monte Carlo simulations
- Leverage multi-core architectures

### 4. **Optimistic Updates**
- Show immediate feedback for parameter changes
- Calculate deltas instead of full recalculations
- Smooth interpolation between states

## Architecture Components

### 1. Calculation Orchestrator

```typescript
export class CalculationOrchestrator {
  private workers: Map<string, Worker> = new Map()
  private cache: CalculationCache
  private scheduler: CalculationScheduler
  
  constructor() {
    this.cache = new IntelligentCache()
    this.scheduler = new PriorityScheduler()
    this.initializeWorkers()
  }
  
  async calculateScenario(
    scenario: Scenario,
    options: CalculationOptions = {}
  ): Promise<CalculationResult> {
    
    const cacheKey = this.generateCacheKey(scenario, options)
    
    // Check cache first
    const cached = await this.cache.get(cacheKey)
    if (cached && !this.isStale(cached, scenario)) {
      return this.enhanceWithRealTimeData(cached)
    }
    
    // Start progressive calculation
    const calculation = new ProgressiveCalculation(scenario, options)
    
    // Return immediate estimate while full calculation proceeds
    const estimate = this.generateQuickEstimate(scenario)
    this.scheduleFullCalculation(calculation, cacheKey)
    
    return {
      ...estimate,
      isEstimate: true,
      fullCalculationPromise: calculation.promise
    }
  }
  
  private generateQuickEstimate(scenario: Scenario): CalculationResult {
    // Use simplified formulas for immediate feedback
    const simpleProjection = this.calculateSimpleCompoundGrowth(scenario)
    
    return {
      netWorth: simpleProjection,
      yearlyProjections: this.generateSimpleProjections(scenario),
      confidence: 'estimate',
      calculationTime: 0.05 // 50ms for estimate
    }
  }
  
  private async scheduleFullCalculation(
    calculation: ProgressiveCalculation,
    cacheKey: string
  ): Promise<void> {
    
    // Add to priority queue
    this.scheduler.schedule({
      calculation,
      priority: this.calculatePriority(calculation),
      onProgress: (progress) => this.broadcastProgress(cacheKey, progress),
      onComplete: (result) => this.cache.set(cacheKey, result)
    })
  }
}
```

### 2. Intelligent Caching System

```typescript
export class IntelligentCache {
  private memoryCache: Map<string, CachedResult> = new Map()
  private persistentCache: IDBCache
  private dependencyGraph: DependencyGraph
  
  constructor() {
    this.persistentCache = new IDBCache('finpath-calculations')
    this.dependencyGraph = new DependencyGraph()
  }
  
  async get(key: string): Promise<CalculationResult | null> {
    // Check memory first
    const memoryResult = this.memoryCache.get(key)
    if (memoryResult && !this.isExpired(memoryResult)) {
      return memoryResult.data
    }
    
    // Check persistent cache
    const persistentResult = await this.persistentCache.get(key)
    if (persistentResult && !this.isExpired(persistentResult)) {
      // Promote to memory cache
      this.memoryCache.set(key, persistentResult)
      return persistentResult.data
    }
    
    return null
  }
  
  async set(key: string, data: CalculationResult): Promise<void> {
    const cachedResult: CachedResult = {
      data,
      timestamp: Date.now(),
      dependencies: this.extractDependencies(data),
      size: this.calculateSize(data)
    }
    
    // Store in memory with LRU eviction
    this.memoryCache.set(key, cachedResult)
    this.enforceMemoryLimits()
    
    // Store in persistent cache
    await this.persistentCache.set(key, cachedResult)
    
    // Update dependency graph
    this.dependencyGraph.addNode(key, cachedResult.dependencies)
  }
  
  invalidate(dependencies: string[]): void {
    // Find all cache entries that depend on changed values
    const toInvalidate = this.dependencyGraph.findDependents(dependencies)
    
    toInvalidate.forEach(key => {
      this.memoryCache.delete(key)
      this.persistentCache.delete(key)
    })
  }
  
  // Smart cache warming for common scenarios
  async warmCache(user: User): Promise<void> {
    const commonScenarios = await this.generateCommonScenarios(user)
    
    // Pre-calculate in background
    commonScenarios.forEach(scenario => {
      this.scheduleBackgroundCalculation(scenario)
    })
  }
}
```

### 3. Web Worker Architecture

```typescript
// Main thread calculation coordinator
export class WorkerManager {
  private workers: Worker[] = []
  private taskQueue: CalculationTask[] = []
  private activeCalculations: Map<string, CalculationContext> = new Map()
  
  constructor() {
    this.initializeWorkerPool()
  }
  
  private initializeWorkerPool(): void {
    const workerCount = Math.min(navigator.hardwareConcurrency || 4, 8)
    
    for (let i = 0; i < workerCount; i++) {
      const worker = new Worker('/workers/financial-calculator.js')
      worker.onmessage = this.handleWorkerMessage.bind(this)
      this.workers.push(worker)
    }
  }
  
  async calculateAsync<T>(
    type: CalculationType,
    data: any,
    options: CalculationOptions = {}
  ): Promise<T> {
    
    const taskId = this.generateTaskId()
    const task: CalculationTask = {
      id: taskId,
      type,
      data,
      options,
      priority: options.priority || 'normal'
    }
    
    return new Promise((resolve, reject) => {
      this.activeCalculations.set(taskId, {
        resolve,
        reject,
        startTime: performance.now(),
        onProgress: options.onProgress
      })
      
      this.scheduleTask(task)
    })
  }
  
  private scheduleTask(task: CalculationTask): void {
    // Add to priority queue
    this.taskQueue.push(task)
    this.taskQueue.sort((a, b) => this.getPriorityValue(b.priority) - this.getPriorityValue(a.priority))
    
    // Try to assign to available worker
    this.assignTasksToWorkers()
  }
  
  private assignTasksToWorkers(): void {
    const availableWorkers = this.workers.filter(w => !w.busy)
    
    while (availableWorkers.length > 0 && this.taskQueue.length > 0) {
      const worker = availableWorkers.pop()!
      const task = this.taskQueue.shift()!
      
      worker.busy = true
      worker.postMessage({
        type: 'calculate',
        task
      })
    }
  }
}

// Web Worker implementation (financial-calculator.js)
class FinancialCalculatorWorker {
  constructor() {
    self.onmessage = this.handleMessage.bind(this)
  }
  
  handleMessage(event: MessageEvent): void {
    const { type, task } = event.data
    
    switch (type) {
      case 'calculate':
        this.performCalculation(task)
        break
      case 'cancel':
        this.cancelCalculation(task.id)
        break
    }
  }
  
  async performCalculation(task: CalculationTask): Promise<void> {
    try {
      const startTime = performance.now()
      let result: any
      
      switch (task.type) {
        case 'MONTE_CARLO':
          result = await this.runMonteCarloSimulation(task.data, task.options)
          break
        case 'SCENARIO_PROJECTION':
          result = await this.calculateScenarioProjection(task.data, task.options)
          break
        case 'TAX_OPTIMIZATION':
          result = await this.optimizeTaxStrategy(task.data, task.options)
          break
        case 'PORTFOLIO_OPTIMIZATION':
          result = await this.optimizePortfolio(task.data, task.options)
          break
        default:
          throw new Error(`Unknown calculation type: ${task.type}`)
      }
      
      const calculationTime = (performance.now() - startTime) / 1000
      
      self.postMessage({
        type: 'result',
        taskId: task.id,
        result: {
          ...result,
          calculationTime
        }
      })
      
    } catch (error) {
      self.postMessage({
        type: 'error',
        taskId: task.id,
        error: error.message
      })
    }
  }
  
  async runMonteCarloSimulation(
    data: MonteCarloData,
    options: CalculationOptions
  ): Promise<MonteCarloResult> {
    
    const { iterations, timeHorizon, scenario } = data
    const results: number[] = []
    const progressInterval = Math.max(1, Math.floor(iterations / 100))
    
    for (let i = 0; i < iterations; i++) {
      // Generate random market conditions
      const marketConditions = this.generateRandomMarketConditions(i)
      
      // Calculate scenario outcome with random returns
      const outcome = this.calculateScenarioWithRandomReturns(
        scenario,
        marketConditions,
        timeHorizon
      )
      
      results.push(outcome)
      
      // Report progress
      if (i % progressInterval === 0) {
        self.postMessage({
          type: 'progress',
          taskId: data.taskId,
          progress: i / iterations
        })
      }
    }
    
    // Calculate summary statistics
    const sortedResults = results.sort((a, b) => a - b)
    
    return {
      iterations,
      finalValues: sortedResults,
      percentiles: {
        p10: sortedResults[Math.floor(iterations * 0.1)],
        p25: sortedResults[Math.floor(iterations * 0.25)],
        p50: sortedResults[Math.floor(iterations * 0.5)],
        p75: sortedResults[Math.floor(iterations * 0.75)],
        p90: sortedResults[Math.floor(iterations * 0.9)]
      },
      successProbability: this.calculateSuccessProbability(sortedResults, data.goalAmount),
      statistics: this.calculateStatistics(sortedResults)
    }
  }
}
```

### 4. Progressive UI Updates

```typescript
export class ProgressiveCalculationHook {
  
  useProgressiveCalculation<T>(
    calculationFn: () => Promise<T>,
    dependencies: any[]
  ): ProgressiveCalculationResult<T> {
    
    const [state, setState] = useState<ProgressiveCalculationState<T>>({
      estimate: null,
      fullResult: null,
      isCalculating: false,
      progress: 0,
      error: null
    })
    
    const [debouncedDeps] = useDebounce(dependencies, 300) // Debounce rapid changes
    
    useEffect(() => {
      const calculate = async () => {
        setState(prev => ({
          ...prev,
          isCalculating: true,
          progress: 0,
          error: null
        }))
        
        try {
          // Start with quick estimate
          const estimate = await generateQuickEstimate(debouncedDeps)
          setState(prev => ({
            ...prev,
            estimate,
            progress: 0.1
          }))
          
          // Run full calculation
          const fullResult = await calculationFn()
          setState(prev => ({
            ...prev,
            fullResult,
            isCalculating: false,
            progress: 1.0
          }))
          
        } catch (error) {
          setState(prev => ({
            ...prev,
            error: error.message,
            isCalculating: false
          }))
        }
      }
      
      calculate()
    }, [debouncedDeps])
    
    return {
      ...state,
      currentResult: state.fullResult || state.estimate
    }
  }
}

// Usage in components
export function ScenarioResults({ scenario }: { scenario: Scenario }) {
  const {
    currentResult,
    isCalculating,
    progress,
    error
  } = useProgressiveCalculation(
    () => calculateScenarioProjection(scenario),
    [scenario.id, scenario.updatedAt]
  )
  
  return (
    <div>
      {isCalculating && (
        <ProgressBar value={progress} className="mb-4" />
      )}
      
      {currentResult && (
        <ScenarioChart 
          data={currentResult.projections}
          isEstimate={!currentResult.isComplete}
        />
      )}
      
      {error && (
        <ErrorDisplay error={error} />
      )}
    </div>
  )
}
```

### 5. Real-Time Parameter Optimization

```typescript
export class ParameterOptimizer {
  private optimizationWorker: Worker
  private activeOptimizations: Map<string, OptimizationContext> = new Map()
  
  constructor() {
    this.optimizationWorker = new Worker('/workers/optimization.js')
    this.optimizationWorker.onmessage = this.handleOptimizationResult.bind(this)
  }
  
  optimizeInRealTime(
    scenario: Scenario,
    objectives: OptimizationObjective[],
    constraints: OptimizationConstraint[]
  ): Observable<OptimizationResult> {
    
    return new Observable(subscriber => {
      const optimizationId = this.generateOptimizationId()
      
      this.activeOptimizations.set(optimizationId, {
        subscriber,
        startTime: performance.now()
      })
      
      this.optimizationWorker.postMessage({
        type: 'optimize',
        id: optimizationId,
        scenario,
        objectives,
        constraints,
        config: {
          algorithm: 'genetic', // or 'gradient_descent', 'simulated_annealing'
          maxIterations: 1000,
          convergenceTolerance: 0.001,
          progressInterval: 10
        }
      })
      
      return () => {
        // Cleanup on unsubscribe
        this.optimizationWorker.postMessage({
          type: 'cancel',
          id: optimizationId
        })
        this.activeOptimizations.delete(optimizationId)
      }
    })
  }
  
  private handleOptimizationResult(event: MessageEvent): void {
    const { type, id, data } = event.data
    const context = this.activeOptimizations.get(id)
    
    if (!context) return
    
    switch (type) {
      case 'progress':
        context.subscriber.next({
          type: 'progress',
          iteration: data.iteration,
          bestObjectiveValue: data.bestObjectiveValue,
          currentSolution: data.currentSolution
        })
        break
        
      case 'result':
        context.subscriber.next({
          type: 'complete',
          optimalSolution: data.solution,
          objectiveValue: data.objectiveValue,
          iterations: data.iterations,
          convergenceTime: performance.now() - context.startTime
        })
        context.subscriber.complete()
        this.activeOptimizations.delete(id)
        break
        
      case 'error':
        context.subscriber.error(new Error(data.message))
        this.activeOptimizations.delete(id)
        break
    }
  }
}
```

## Performance Targets

### Calculation Speed
- **Simple Scenarios**: < 100ms for basic projections
- **Complex Scenarios**: < 2 seconds for full multi-component analysis
- **Monte Carlo (10k iterations)**: < 10 seconds with progress updates
- **Portfolio Optimization**: < 5 seconds for 20+ asset classes

### User Experience
- **Initial Response**: < 50ms for any user interaction
- **Progressive Updates**: Every 100ms during calculations
- **Chart Animations**: 60fps for all transitions
- **Parameter Changes**: < 100ms feedback for slider adjustments

### Memory Management
- **Cache Size**: Maximum 100MB for calculation cache
- **Worker Memory**: Each worker limited to 50MB
- **Garbage Collection**: Proactive cleanup of unused calculations
- **Progressive Loading**: Load only visible data initially

## Monitoring and Observability

### Performance Metrics
```typescript
export class PerformanceMonitor {
  private metrics: Map<string, PerformanceMetric[]> = new Map()
  
  trackCalculation(
    type: CalculationType,
    duration: number,
    complexity: number,
    success: boolean
  ): void {
    
    const metric: PerformanceMetric = {
      type,
      duration,
      complexity,
      success,
      timestamp: Date.now(),
      userAgent: navigator.userAgent,
      hardwareConcurrency: navigator.hardwareConcurrency
    }
    
    this.metrics.get(type)?.push(metric) || this.metrics.set(type, [metric])
    
    // Report to analytics if significantly slower than target
    if (duration > this.getTargetDuration(type) * 1.5) {
      this.reportSlowCalculation(metric)
    }
  }
  
  generatePerformanceReport(): PerformanceReport {
    const report: PerformanceReport = {
      calculationTypes: {},
      overall: {
        totalCalculations: 0,
        averageDuration: 0,
        successRate: 0
      }
    }
    
    this.metrics.forEach((metrics, type) => {
      const successfulMetrics = metrics.filter(m => m.success)
      const totalDuration = successfulMetrics.reduce((sum, m) => sum + m.duration, 0)
      
      report.calculationTypes[type] = {
        count: metrics.length,
        successCount: successfulMetrics.length,
        averageDuration: totalDuration / successfulMetrics.length,
        p95Duration: this.calculatePercentile(successfulMetrics.map(m => m.duration), 0.95),
        successRate: successfulMetrics.length / metrics.length
      }
      
      report.overall.totalCalculations += metrics.length
    })
    
    return report
  }
}
```

## Implementation Strategy

### Phase 1: Basic Real-Time Engine
1. Implement progressive calculation for simple scenarios
2. Add basic caching for common calculations
3. Create Web Worker for Monte Carlo simulations
4. Build progress indicators and loading states

### Phase 2: Advanced Optimization
1. Implement intelligent cache dependency tracking
2. Add real-time parameter optimization
3. Create advanced Web Worker pool management
4. Build comprehensive performance monitoring

### Phase 3: Production Optimization
1. Add cache warming for common user patterns
2. Implement predictive calculation for likely parameter changes
3. Create adaptive algorithm selection based on device capabilities
4. Build comprehensive error recovery and fallback systems

This architecture ensures FinPath Explorer delivers the stunning, responsive experience users expect while handling complex financial calculations with professional-grade accuracy.