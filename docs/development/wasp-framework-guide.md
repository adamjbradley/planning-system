# Wasp Framework Development Guide

This document provides comprehensive guidance for developing with the Wasp framework in the FinPath Explorer project.

## Wasp Framework Overview

### What is Wasp
- Wasp (Web Application SPecification language) is a declarative, statically typed, domain-specific language (DSL) for building modern, full-stack web applications
- Unlike traditional frameworks that are sets of libraries, Wasp is a simple programming language that understands web app concepts and generates code for you
- Wasp integrates with React (frontend), Node.js (backend), and Prisma (database ORM) to create full-stack web applications with minimal boilerplate
- The Wasp compiler reads your declarative configuration in `main.wasp` and generates all the necessary code for a working web application

### Project Structure
- A Wasp project consists of a `main.wasp` file in the root directory that defines the app's configuration
- The `schema.prisma` file in the root directory defines your database models ("entities")
- Your custom code lives in the `src/` directory, organized by features (e.g. `src/scenario/`, `src/housing/`)
- Wasp generates additional code that connects everything together when you run your app

### The Wasp Config File
The main Wasp Config File (`main.wasp`) is the central configuration file that defines your application structure. It contains declarations for app settings, pages, routes, authentication, database entities, and operations (queries and actions).

Example structure:
```wasp
app FinPathExplorer {
  wasp: {
    version: "^0.17.0"
  },
  title: "FinPath Explorer - Financial Planning Platform",
}

route HomeRoute { path: "/", to: HomePage }
page HomePage {
  component: import { HomePage } from "@src/client/pages/HomePage.tsx"
}

// Operations are defined here
query getScenarios {
  fn: import { getScenarios } from "@src/scenario/operations.ts",
  entities: [Scenario]
}
```

## Import Conventions & Rules

### Wasp Imports (.ts/.tsx files)
```typescript
// ✅ Correct Wasp imports - always use 'wasp/' prefix
import { Scenario } from 'wasp/entities'
import { getScenarios, useQuery } from 'wasp/client/operations'
import type { GetScenarios } from 'wasp/server/operations'
import { HttpError } from 'wasp/server'
import { useAuth } from 'wasp/client/auth'

// ❌ Incorrect - never use '@wasp/' prefix
import { Scenario } from '@wasp/entities'  // WRONG
```

### Wasp Config Imports (main.wasp)
```wasp
// ✅ Correct - imports in main.wasp must use '@src/' prefix
component: import { LoginPage } from "@src/auth/LoginPage"
fn: import { getScenarios } from "@src/scenario/operations"

// ❌ Incorrect
component: import { LoginPage } from "../src/auth/LoginPage"  // WRONG
```

### Local Imports (.ts/.tsx files)
```typescript
// ✅ Correct - use relative paths within src/
import { utils } from '../shared/utils'
import { ScenarioCard } from './components/ScenarioCard'

// ❌ Incorrect - avoid @src/ in .ts/.tsx files
import { utils } from '@src/shared/utils'  // WRONG
```

### Prisma Enum Imports
```typescript
// ✅ For enum VALUES - import from @prisma/client
import { Country } from '@prisma/client'
const australia = Country.AUSTRALIA  // Runtime value

// ✅ For enum TYPES - import from wasp/entities
import type { Country } from 'wasp/entities'
```

## Database & Operations

### Database Setup
- Uses Prisma ORM with PostgreSQL
- Models defined in `schema.prisma` automatically become Wasp Entities
- Apply schema changes with: `wasp db migrate-dev "Description"`
- Database choice: Use PostgreSQL for features like enums and PgBoss jobs

### Wasp Operations (Queries & Actions)

#### Operation Definitions (main.wasp)
```wasp
query getScenarios {
  fn: import { getScenarios } from "@src/scenario/operations",
  entities: [Scenario]  // Grant access to Scenario entity
}

action createScenario {
  fn: import { createScenario } from "@src/scenario/operations",
  entities: [Scenario]
}
```

#### Operation Implementation
```typescript
import { HttpError } from 'wasp/server'
import type { GetScenarios, CreateScenario } from 'wasp/server/operations'
import type { Scenario } from 'wasp/entities'

export const getScenarios: GetScenarios<void, Scenario[]> = async (_args, context) => {
  if (!context.user) {
    throw new HttpError(401, 'Not authorized');
  }
  return context.entities.Scenario.findMany({
    where: { userId: context.user.id }
  });
}

export const createScenario: CreateScenario<CreateScenarioInput, Scenario> = async (args, context) => {
  if (!context.user) {
    throw new HttpError(401, 'Not authorized');
  }
  return context.entities.Scenario.create({
    data: {
      name: args.name,
      country: args.country,
      userId: context.user.id,
    }
  });
}
```

#### Client-Side Usage
```typescript
// Queries - use useQuery hook
import { useQuery } from 'wasp/client/operations'
import { getScenarios } from 'wasp/client/operations'

const { data: scenarios, isLoading, error } = useQuery(getScenarios)

// Actions - call directly with async/await (DO NOT use useAction unless optimistic updates needed)
import { createScenario } from 'wasp/client/operations'

const handleCreate = async () => {
  const result = await createScenario({ name: 'New scenario', country: 'AUSTRALIA' })
}
```

## Authentication

### Auth Configuration (main.wasp)
```wasp
auth: {
  userEntity: User,
  methods: {
    email: {
      fromField: {
        name: "FinPath Explorer",
        email: "support@finpathexplorer.com"
      },
      emailVerification: {
        clientRoute: EmailVerificationRoute,
        getEmailContentFn: import { getVerificationEmailContent } from "@src/auth/emails",
      },
      passwordReset: {
        clientRoute: PasswordResetRoute,
        getEmailContentFn: import { getPasswordResetEmailContent } from "@src/auth/emails",
      },
    },
  },
  onAuthFailedRedirectTo: "/login",
  onAuthSucceededRedirectTo: "/scenarios",
}
```

### User Model Rules
- Wasp handles auth fields internally (email, password, provider IDs)
- Your User model typically only needs `id` field for Wasp linkage
- Add non-auth fields as needed (profile info, preferences, relations)
- If you need frequent access to email/username, add to User model and populate via `userSignupFields`

### Protected Routes & Components
```typescript
import { useAuth } from 'wasp/client/auth'

const MyProtectedPage = () => {
  const { data: user, isLoading, error } = useAuth()

  if (isLoading) return <div>Loading...</div>
  if (error || !user) {
    return <div>Please log in to access this page.</div>
  }

  // User is authenticated
  return <div>Welcome back!</div>
}
```

## UI Components (ShadCN UI)

### Important Version Constraint
**Use ShadCN UI v2.3.0 only** - Wasp cannot use Tailwind v4 (required by newer ShadCN versions)

### Adding New ShadCN Components
```bash
# 1. Add component
npx shadcn@2.3.0 add button

# 2. Fix import path in generated component
# Change: import { cn } from "s/lib/utils"
# To:     import { cn } from "../../lib/utils"

# 3. Use component
import { Button } from './components/ui/button'
```

### Existing Components Location
All ShadCN components are in `src/components/ui/` and ready to use.

## Code Organization & Conventions

### Feature Structure
- Group feature code in `src/{featureName}/` directories
- Combine operations (queries/actions) in `operations.ts` files within feature directories
- Organize Wasp config sections using `//#region` directives:

```wasp
//#region Scenario Management
route ScenariosRoute { path: "/scenarios", to: ScenariosPage }
page ScenariosPage {
  component: import ScenariosPage from "@src/scenario/ScenariosPage"
}
//#endregion
```

### Common Patterns
- Define app structure in `main.wasp` configuration file
- Define data models in `schema.prisma` (not in main.wasp)
- Use Wasp operations (queries/actions) for client-server communication
- Reference the main.wasp file as source of truth for app configuration
- Always use TypeScript for Wasp code (.ts/.tsx)
- Install dependencies via `npm install` (updates package.json)

## Advanced Features

### Jobs (Background Tasks)
```wasp
job emailSender {
  executor: PgBoss,  // Requires PostgreSQL
  perform: {
    fn: import { sendEmail } from "@src/server/jobs/emailSender"
  },
  schedule: {
    cron: "0 * * * *"  // Every hour
  },
  entities: [User, EmailQueue]
}
```

### Custom API Endpoints
```wasp
api stripeWebhook {
  fn: import { handleStripeWebhook } from "@src/server/apis/stripe",
  httpRoute: (POST, "/webhooks/stripe"),
  entities: [User, Payment],
  auth: false  // Skip JWT parsing
}
```

## Troubleshooting

### Common Issues & Solutions

#### Wasp Type/Import Errors
- **Problem**: Missing Wasp imports or type mismatches after modifying main.wasp or schema.prisma
- **Solution**: Restart Wasp development server (`wasp start`) - Wasp needs to regenerate code

#### Operations Not Working
- Check `entities: [...]` list in operation definition (main.wasp)
- Verify import path (`fn: import { ... } from "@src/..."`) in main.wasp
- Check Wasp server console for runtime errors
- Ensure client-side calls match expected arguments and types

#### Authentication Issues
- Verify `auth` configuration in main.wasp (userEntity, methods, redirects)
- Ensure `userEntity` matches actual User model name in schema.prisma
- Check Wasp server logs for auth-related errors
- Confirm environment variables for social providers (e.g., `GOOGLE_CLIENT_ID`)

#### Database Problems
- Ensure schema.prisma syntax is correct
- Run `wasp db migrate-dev "Description"` after schema changes
- Check PostgreSQL is running (use `wasp start db`)
- Verify `DATABASE_URL` in .env.server file

#### Build/Runtime Errors
- Check import paths (follow conventions above)
- Ensure dependencies are installed (`npm install`)
- Check both Wasp server console and browser developer console
- If "Cannot find module 'wasp/...'" - check import path prefix
- If "Cannot find module '@src/...'" in .ts/.tsx - use relative paths instead

### Performance Optimization
- Use specific entity dependencies in operation definitions
- Implement pagination for large data sets
- Use React.memo, useMemo, useCallback for React optimization
- Consider optimistic updates with useAction hook for critical UX (use sparingly)

### Error Handling Patterns
```typescript
import { HttpError } from 'wasp/server'

// Throw structured errors
if (!context.user) {
  throw new HttpError(401, 'Not authorized')
}

// Handle unexpected errors
try {
  // operation logic
} catch (error) {
  if (error instanceof HttpError) {
    throw error  // Re-throw known errors
  }
  console.error('Unexpected error:', error)
  throw new HttpError(500, 'Internal server error')
}
```

## Development Commands

### Main Application (from `planning-system/app/`)
```bash
# Start development environment
wasp start db          # Start PostgreSQL database (keep running)
wasp start              # Start the application (keep running)
wasp db migrate-dev     # Run database migrations (after schema changes)

# Database operations
wasp db seed            # Seed database with mock data
wasp db reset           # Reset database
```

## Deployment (Fly.io)

### Prerequisites
- Fly.io account with billing information
- Install and authenticate `flyctl` CLI: `fly auth login`

### Deployment Commands
```bash
# Initial deployment
wasp deploy fly launch <app-name> <region>
# Example: wasp deploy fly launch finpath-explorer mia

# Updates after initial deployment
wasp deploy fly deploy

# Set environment variables (secrets)
wasp deploy fly cmd --context server secrets set VARIABLE_NAME="VALUE"

# List secrets
wasp deploy fly cmd --context server secrets list
```

### Post-Deployment
- Commit generated `fly-server.toml` and `fly-client.toml` files
- Set required environment variables as Fly.io secrets
- Monitor apps in Fly.io dashboard

## Documentation References
- Wasp documentation (LLM-optimized): https://wasp.sh/llms-full.txt
- Wasp Docs homepage: https://wasp.sh/docs