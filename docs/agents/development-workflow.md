# Development Workflow for Claude Code Agents

## Overview

This document provides specific guidance for Claude Code agents working on the Planning System. It outlines the development workflow, best practices, and step-by-step procedures for implementing features effectively.

## Pre-Development Checklist

### Before Starting Any Feature
1. **Read the PRD**: Review `/docs/PRD.md` for overall project context
2. **Study Feature Specification**: Read the relevant feature document in `/docs/features/`
3. **Check Implementation Roadmap**: Verify current phase in `/docs/planning/implementation-roadmap.md`
4. **Review Integration Strategy**: Understand existing infrastructure in `/docs/planning/integration-strategy.md`
5. **Reference CLAUDE.md**: Follow Wasp-specific conventions and patterns

### Environment Setup Verification
```bash
# Verify Wasp environment is ready
wasp start db          # Database should start successfully
wasp start              # Application should compile and run
wasp db migrate-dev     # Migrations should apply cleanly
```

## Development Workflow Steps

### Step 1: Database Schema Implementation

#### 1.1 Schema Design
- Review the data models specified in the feature document
- Ensure all relationships and constraints are properly defined
- Add appropriate indexes for performance
- Follow existing naming conventions

#### 1.2 Schema Update Process
```bash
# 1. Update schema.prisma with new models
# 2. Create migration
wasp db migrate-dev "Add [feature-name] models"

# 3. Verify migration success
wasp db migrate-dev  # Should run without errors

# 4. Test with seed data
wasp db seed
```

#### 1.3 Schema Validation Checklist
- [ ] All required fields have appropriate types
- [ ] Relationships use correct foreign key constraints
- [ ] Enums are properly defined
- [ ] Indexes are added for performance-critical queries
- [ ] Existing models are extended without breaking changes

### Step 2: Wasp Operations Implementation

#### 2.1 Create Operations File
```typescript
// src/[feature-name]/operations.ts
import { HttpError } from 'wasp/server'
import type { [OperationName] } from 'wasp/server/operations'
import type { [EntityName] } from 'wasp/entities'

// Follow this pattern for all operations
export const [operationName]: [OperationName]<Args, Return> = async (args, context) => {
  // 1. Authentication check
  if (!context.user) {
    throw new HttpError(401, 'Authentication required')
  }
  
  // 2. Input validation
  const validatedArgs = validateInput(args)
  
  // 3. Permission check
  await verifyPermissions(validatedArgs, context.user, context)
  
  // 4. Business logic
  const result = await performOperation(validatedArgs, context)
  
  // 5. Side effects (logging, notifications)
  await handleSideEffects(result, context)
  
  return result
}
```

#### 2.2 Add Operations to main.wasp
```wasp
// Follow existing patterns for operation definitions
query [queryName] {
  fn: import { [queryName] } from "@src/[feature-name]/operations",
  entities: [[RequiredEntity1], [RequiredEntity2]]
}

action [actionName] {
  fn: import { [actionName] } from "@src/[feature-name]/operations",
  entities: [[RequiredEntity1], [RequiredEntity2]]
}
```

#### 2.3 Operations Testing
```bash
# Test operations manually first
wasp start  # Restart to load new operations

# Use browser network tab or API testing tool to verify:
# - Operations are accessible
# - Authentication works
# - Permissions are enforced
# - Error handling works correctly
```

### Step 3: UI Component Development

#### 3.1 Page Component Structure
```typescript
// src/[feature-name]/[PageName]Page.tsx
import { useQuery } from 'wasp/client/operations'
import { [operationName] } from 'wasp/client/operations'

export const [PageName]Page = () => {
  // 1. Data fetching
  const { data, isLoading, error } = useQuery([operationName], args)
  
  // 2. Loading state
  if (isLoading) return <LoadingSpinner />
  
  // 3. Error state
  if (error) return <ErrorMessage error={error} />
  
  // 4. Main content
  return (
    <div className="container mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">[Page Title]</h1>
      {/* Page content using existing UI components */}
    </div>
  )
}
```

#### 3.2 Component Development Guidelines
- **Reuse Existing Components**: Always check `src/components/ui/` first
- **Follow ShadCN Patterns**: Use existing component patterns and styling
- **Responsive Design**: Ensure mobile compatibility
- **Accessibility**: Include proper ARIA labels and keyboard navigation
- **Loading States**: Implement skeleton loaders or spinners
- **Error Handling**: Graceful error display and recovery

#### 3.3 Form Implementation
```typescript
// Use React Hook Form with Zod validation
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const FormSchema = z.object({
  // Define form fields with validation
})

export const [FeatureName]Form = ({ onSubmit }: FormProps) => {
  const form = useForm<z.infer<typeof FormSchema>>({
    resolver: zodResolver(FormSchema)
  })
  
  const handleSubmit = async (data: z.infer<typeof FormSchema>) => {
    try {
      await onSubmit(data)
      form.reset()
    } catch (error) {
      // Handle form errors
    }
  }
  
  return (
    <Form {...form}>
      {/* Form fields using ShadCN form components */}
    </Form>
  )
}
```

### Step 4: Route Configuration

#### 4.1 Add Routes to main.wasp
```wasp
// Follow existing route patterns
route [RouteName]Route { path: "/[path]", to: [PageName]Page }
page [PageName]Page {
  component: import [PageName] from "@src/[feature-name]/[PageName]Page",
  authRequired: true  // Most planning features require auth
}
```

#### 4.2 Navigation Integration
```typescript
// Add to existing navigation components
// Update Sidebar.tsx or NavBar.tsx as appropriate
const planningNavItems = [
  { href: '/projects', label: 'Projects', icon: ProjectIcon },
  { href: '/tasks', label: 'Tasks', icon: TaskIcon },
  // ... other nav items
]
```

### Step 5: Integration Testing

#### 5.1 Manual Testing Checklist
- [ ] Pages load without errors
- [ ] Forms submit successfully
- [ ] Data displays correctly
- [ ] Navigation works properly
- [ ] Authentication is enforced
- [ ] Permissions are respected
- [ ] Mobile responsiveness works
- [ ] Loading and error states function

#### 5.2 E2E Test Implementation
```typescript
// e2e-tests/tests/[feature-name].spec.ts
import { test, expect } from '@playwright/test'

test.describe('[Feature Name]', () => {
  test.beforeEach(async ({ page }) => {
    // Login with test user
    await login(page, 'test@example.com', 'password')
  })
  
  test('can create new [entity]', async ({ page }) => {
    await page.goto('/[feature-path]')
    
    // Test the complete workflow
    await page.click('[data-testid="create-button"]')
    await page.fill('[data-testid="name-input"]', 'Test Name')
    await page.click('[data-testid="submit-button"]')
    
    // Verify success
    await expect(page.locator('[data-testid="success-message"]')).toBeVisible()
  })
})
```

## Quality Assurance

### Code Review Checklist

#### Before Submitting Changes
- [ ] All TypeScript errors resolved
- [ ] No console.log statements in production code
- [ ] Proper error handling implemented
- [ ] Input validation added where needed
- [ ] Performance considerations addressed
- [ ] Security best practices followed
- [ ] Documentation updated if needed

#### Testing Requirements
- [ ] Manual testing completed
- [ ] E2E tests written and passing
- [ ] Edge cases considered and tested
- [ ] Error scenarios tested
- [ ] Permission boundaries tested

### Performance Guidelines

#### Database Queries
- Use appropriate indexes for all queries
- Implement pagination for large datasets
- Avoid N+1 query problems
- Use selective field loading

#### Client-Side Performance
- Implement proper loading states
- Use React.memo for expensive components
- Optimize re-renders with useMemo and useCallback
- Consider virtualization for large lists

#### Real-time Features
- Implement efficient WebSocket connections
- Batch updates when possible
- Provide fallback for connection failures
- Optimize update frequency

## Troubleshooting Guide

### Common Issues and Solutions

#### "Cannot find module 'wasp/...'" Error
**Solution**: Restart Wasp development server
```bash
# Stop current server (Ctrl+C)
wasp start  # Restart to regenerate types
```

#### Database Migration Errors
**Solution**: Check schema syntax and relationships
```bash
# Reset database if needed (development only)
wasp db reset
wasp db migrate-dev
wasp db seed
```

#### Permission Errors
**Solution**: Verify user roles and project membership
```typescript
// Debug permission checking
console.log('User role:', user.role)
console.log('Project membership:', projectMember)
console.log('Required permission:', requiredPermission)
```

#### Type Errors
**Solution**: Ensure proper imports and type definitions
```typescript
// Import types correctly
import type { [EntityName] } from 'wasp/entities'
import type { [OperationName] } from 'wasp/server/operations'
```

### Getting Help

#### Resources
1. **CLAUDE.md**: Wasp-specific development guidance
2. **Feature Documents**: Detailed specifications in `/docs/features/`
3. **Wasp Documentation**: https://wasp.sh/docs (reference the LLM-optimized versions)
4. **Existing Code**: Study patterns in current SaaS features

#### Debug Information to Collect
- Wasp version: Check `main.wasp` file
- Error messages: Full stack traces
- Browser console: Client-side errors
- Database state: Current data and schema
- Authentication state: Current user and permissions

---

*Follow this workflow systematically to ensure consistent, high-quality implementation of Planning System features.*