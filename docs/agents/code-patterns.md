# Code Patterns for Claude Code Agents

## Overview

This document provides specific code patterns and examples for implementing Planning System features in the Wasp framework. Use these patterns as templates for consistent, maintainable code.

## Database Patterns

### Entity Definition Patterns

#### Standard Entity Structure
```prisma
model [EntityName] {
  id          String   @id @default(uuid())
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  // Core fields
  name        String
  description String?
  status      [EntityName]Status @default([DEFAULT_STATUS])
  
  // Relationships
  owner       User     @relation("[EntityName]Owner", fields: [ownerId], references: [id])
  ownerId     String
  
  // Collections
  [relatedEntities] [RelatedEntity][]
  
  @@map("[table_name]")
  @@index([ownerId])
  @@index([status])
  @@index([createdAt])
}

enum [EntityName]Status {
  [STATUS_1]
  [STATUS_2]
  [STATUS_3]
}
```

#### Relationship Patterns
```prisma
// One-to-Many
model Project {
  tasks Task[]
}
model Task {
  project   Project @relation(fields: [projectId], references: [id], onDelete: Cascade)
  projectId String
}

// Many-to-Many with Join Table
model Project {
  members ProjectMember[]
}
model User {
  projectMemberships ProjectMember[]
}
model ProjectMember {
  user      User    @relation(fields: [userId], references: [id])
  userId    String
  project   Project @relation(fields: [projectId], references: [id], onDelete: Cascade)
  projectId String
  role      ProjectRole @default(MEMBER)
  
  @@unique([userId, projectId])
}

// Self-Referencing
model Task {
  parentTask   Task?  @relation("TaskSubtasks", fields: [parentTaskId], references: [id])
  parentTaskId String?
  subtasks     Task[] @relation("TaskSubtasks")
}

// Polymorphic Relationships (via optional foreign keys)
model File {
  project   Project? @relation(fields: [projectId], references: [id])
  projectId String?
  task      Task?    @relation(fields: [taskId], references: [id])
  taskId    String?
}
```

## Operation Patterns

### Query Operation Pattern
```typescript
import { HttpError } from 'wasp/server'
import type { [QueryName] } from 'wasp/server/operations'
import type { [EntityName] } from 'wasp/entities'

// Standard query with filtering and pagination
export const [queryName]: [QueryName]<[QueryArgs], [ReturnType]> = async (args, context) => {
  // 1. Authentication
  if (!context.user) {
    throw new HttpError(401, 'Authentication required')
  }
  
  // 2. Input validation
  const { page = 1, limit = 20, status, search } = args
  const skip = (page - 1) * limit
  
  // 3. Permission-based filtering
  const whereClause = {
    // Base permission filter
    OR: [
      { ownerId: context.user.id },
      { 
        members: {
          some: { userId: context.user.id }
        }
      }
    ],
    // Additional filters
    ...(status && { status }),
    ...(search && {
      OR: [
        { name: { contains: search, mode: 'insensitive' } },
        { description: { contains: search, mode: 'insensitive' } }
      ]
    })
  }
  
  // 4. Data fetching with optimization
  const [items, total] = await Promise.all([
    context.entities.[EntityName].findMany({
      where: whereClause,
      include: {
        owner: { select: { id: true, firstName: true, lastName: true, avatar: true } },
        members: {
          include: {
            user: { select: { id: true, firstName: true, lastName: true, avatar: true } }
          }
        },
        _count: { select: { [relatedEntity]: true } }
      },
      orderBy: { createdAt: 'desc' },
      skip,
      take: limit
    }),
    context.entities.[EntityName].count({ where: whereClause })
  ])
  
  return {
    items,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit)
    }
  }
}
```

### Action Operation Pattern
```typescript
import { HttpError } from 'wasp/server'
import type { [ActionName] } from 'wasp/server/operations'
import type { [EntityName] } from 'wasp/entities'
import { z } from 'zod'

// Input validation schema
const [ActionName]Schema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(1000).optional(),
  // ... other fields
})

export const [actionName]: [ActionName]<[ActionArgs], [ReturnType]> = async (args, context) => {
  // 1. Authentication
  if (!context.user) {
    throw new HttpError(401, 'Authentication required')
  }
  
  // 2. Input validation
  const validatedData = [ActionName]Schema.parse(args)
  
  // 3. Permission checking
  if (args.id) {
    const existingEntity = await context.entities.[EntityName].findUnique({
      where: { id: args.id },
      include: { members: true }
    })
    
    if (!existingEntity) {
      throw new HttpError(404, '[EntityName] not found')
    }
    
    const hasPermission = existingEntity.ownerId === context.user.id ||
      existingEntity.members.some(m => m.userId === context.user.id && m.role !== 'VIEWER')
    
    if (!hasPermission) {
      throw new HttpError(403, 'Insufficient permissions')
    }
  }
  
  // 4. Business logic with transaction
  const result = await context.entities.$transaction(async (tx) => {
    // Create or update entity
    const entity = await tx.[EntityName].create({ // or update
      data: {
        ...validatedData,
        ownerId: context.user.id
      },
      include: {
        owner: { select: { id: true, firstName: true, lastName: true } },
        members: true
      }
    })
    
    // Create activity log entry
    await tx.ActivityFeed.create({
      data: {
        action: 'CREATED',
        entityType: '[ENTITY_TYPE]',
        entityId: entity.id,
        userId: context.user.id,
        projectId: entity.projectId || entity.id, // For projects, use entity.id
        details: JSON.stringify({ name: entity.name })
      }
    })
    
    return entity
  })
  
  // 5. Side effects (notifications, etc.)
  if (result.members.length > 0) {
    // Send notifications to team members
    await Promise.all(
      result.members.map(member => 
        createNotification({
          userId: member.userId,
          type: '[NOTIFICATION_TYPE]',
          title: `New [EntityName] created`,
          message: `${context.user.firstName} created "${result.name}"`,
          projectId: result.projectId || result.id
        }, context)
      )
    )
  }
  
  return result
}
```

### Permission Helper Patterns
```typescript
// Permission checking utilities
export const verifyProjectAccess = async (
  projectId: string, 
  userId: string, 
  context: any,
  requiredRole: ProjectRole = 'VIEWER'
) => {
  const membership = await context.entities.ProjectMember.findUnique({
    where: {
      userId_projectId: { userId, projectId }
    },
    include: { project: true }
  })
  
  if (!membership && membership?.project.ownerId !== userId) {
    throw new HttpError(403, 'Access denied to project')
  }
  
  if (requiredRole !== 'VIEWER' && !hasRequiredRole(membership.role, requiredRole)) {
    throw new HttpError(403, `Requires ${requiredRole} role or higher`)
  }
  
  return membership
}

const hasRequiredRole = (userRole: ProjectRole, requiredRole: ProjectRole): boolean => {
  const roleHierarchy = ['VIEWER', 'MEMBER', 'ADMIN', 'OWNER']
  return roleHierarchy.indexOf(userRole) >= roleHierarchy.indexOf(requiredRole)
}
```

## React Component Patterns

### Page Component Pattern
```typescript
import React from 'react'
import { useQuery } from 'wasp/client/operations'
import { [operationName] } from 'wasp/client/operations'
import { LoadingSpinner } from '@/components/LoadingSpinner'
import { ErrorMessage } from '@/components/ErrorMessage'
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'

interface [PageName]PageProps {
  // Optional props
}

export const [PageName]Page: React.FC<[PageName]PageProps> = () => {
  // Data fetching
  const { data: items, isLoading, error, refetch } = useQuery([operationName], {
    page: 1,
    limit: 20
  })
  
  // Loading state
  if (isLoading) {
    return (
      <div className="container mx-auto p-6">
        <div className="flex items-center justify-between mb-6">
          <h1 className="text-3xl font-bold">[Page Title]</h1>
          <Button disabled>Create New</Button>
        </div>
        <LoadingSpinner />
      </div>
    )
  }
  
  // Error state
  if (error) {
    return (
      <div className="container mx-auto p-6">
        <h1 className="text-3xl font-bold mb-6">[Page Title]</h1>
        <ErrorMessage 
          error={error} 
          onRetry={refetch}
          message="Failed to load [entities]. Please try again."
        />
      </div>
    )
  }
  
  // Main content
  return (
    <div className="container mx-auto p-6">
      <div className="flex items-center justify-between mb-6">
        <h1 className="text-3xl font-bold">[Page Title]</h1>
        <Button onClick={() => {/* Handle create */}}>
          Create New
        </Button>
      </div>
      
      {items?.items.length === 0 ? (
        <EmptyState />
      ) : (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          {items?.items.map(item => (
            <[EntityName]Card key={item.id} [entity]={item} />
          ))}
        </div>
      )}
    </div>
  )
}

// Empty state component
const EmptyState = () => (
  <Card className="text-center py-12">
    <CardContent>
      <h3 className="text-lg font-semibold mb-2">No [entities] yet</h3>
      <p className="text-muted-foreground mb-4">
        Get started by creating your first [entity].
      </p>
      <Button>Create Your First [Entity]</Button>
    </CardContent>
  </Card>
)
```

### Form Component Pattern
```typescript
import React from 'react'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { Button } from '@/components/ui/button'
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'

// Form validation schema
const [EntityName]FormSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100, 'Name too long'),
  description: z.string().max(1000, 'Description too long').optional(),
  status: z.enum(['STATUS_1', 'STATUS_2', 'STATUS_3']),
  priority: z.enum(['LOW', 'MEDIUM', 'HIGH', 'CRITICAL']).default('MEDIUM')
})

type [EntityName]FormData = z.infer<typeof [EntityName]FormSchema>

interface [EntityName]FormProps {
  onSubmit: (data: [EntityName]FormData) => Promise<void>
  initialData?: Partial<[EntityName]FormData>
  isSubmitting?: boolean
  onCancel?: () => void
}

export const [EntityName]Form: React.FC<[EntityName]FormProps> = ({
  onSubmit,
  initialData,
  isSubmitting = false,
  onCancel
}) => {
  const form = useForm<[EntityName]FormData>({
    resolver: zodResolver([EntityName]FormSchema),
    defaultValues: {
      name: '',
      description: '',
      status: 'STATUS_1',
      priority: 'MEDIUM',
      ...initialData
    }
  })
  
  const handleSubmit = async (data: [EntityName]FormData) => {
    try {
      await onSubmit(data)
      form.reset()
    } catch (error) {
      // Error handling is done by parent component
      console.error('Form submission error:', error)
    }
  }
  
  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(handleSubmit)} className="space-y-6">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input placeholder="Enter [entity] name" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        
        <FormField
          control={form.control}
          name="description"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Description</FormLabel>
              <FormControl>
                <Textarea 
                  placeholder="Enter [entity] description" 
                  className="min-h-[100px]"
                  {...field} 
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        
        <FormField
          control={form.control}
          name="priority"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Priority</FormLabel>
              <Select onValueChange={field.onChange} defaultValue={field.value}>
                <FormControl>
                  <SelectTrigger>
                    <SelectValue placeholder="Select priority" />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  <SelectItem value="LOW">Low</SelectItem>
                  <SelectItem value="MEDIUM">Medium</SelectItem>
                  <SelectItem value="HIGH">High</SelectItem>
                  <SelectItem value="CRITICAL">Critical</SelectItem>
                </SelectContent>
              </Select>
              <FormMessage />
            </FormItem>
          )}
        />
        
        <div className="flex justify-end space-x-4">
          {onCancel && (
            <Button type="button" variant="outline" onClick={onCancel}>
              Cancel
            </Button>
          )}
          <Button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Saving...' : 'Save'}
          </Button>
        </div>
      </form>
    </Form>
  )
}
```

### Card Component Pattern
```typescript
import React from 'react'
import { format } from 'date-fns'
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar'
import { Progress } from '@/components/ui/progress'
import { MoreHorizontal, Users, Calendar } from 'lucide-react'

interface [EntityName]CardProps {
  [entity]: [EntityType]
  onEdit?: (id: string) => void
  onDelete?: (id: string) => void
  onView?: (id: string) => void
}

export const [EntityName]Card: React.FC<[EntityName]CardProps> = ({
  [entity],
  onEdit,
  onDelete,
  onView
}) => {
  const getStatusColor = (status: string) => {
    const colors = {
      'ACTIVE': 'bg-green-100 text-green-800',
      'COMPLETED': 'bg-blue-100 text-blue-800',
      'ON_HOLD': 'bg-yellow-100 text-yellow-800',
      'CANCELLED': 'bg-red-100 text-red-800'
    }
    return colors[status as keyof typeof colors] || 'bg-gray-100 text-gray-800'
  }
  
  const getPriorityColor = (priority: string) => {
    const colors = {
      'LOW': 'bg-gray-100 text-gray-800',
      'MEDIUM': 'bg-blue-100 text-blue-800', 
      'HIGH': 'bg-orange-100 text-orange-800',
      'CRITICAL': 'bg-red-100 text-red-800'
    }
    return colors[priority as keyof typeof colors] || 'bg-gray-100 text-gray-800'
  }
  
  return (
    <Card className="hover:shadow-lg transition-shadow cursor-pointer">
      <CardHeader className="pb-3">
        <div className="flex items-start justify-between">
          <CardTitle className="text-lg line-clamp-1">{[entity].name}</CardTitle>
          <div className="flex space-x-1">
            <Badge className={getStatusColor([entity].status)}>
              {[entity].status}
            </Badge>
            <Badge variant="outline" className={getPriorityColor([entity].priority)}>
              {[entity].priority}
            </Badge>
          </div>
        </div>
      </CardHeader>
      
      <CardContent className="pb-3">
        {[entity].description && (
          <p className="text-muted-foreground text-sm line-clamp-2 mb-4">
            {[entity].description}
          </p>
        )}
        
        {/* Progress indicator (if applicable) */}
        {[entity].completionPercentage !== undefined && (
          <div className="mb-4">
            <div className="flex justify-between text-sm mb-1">
              <span>Progress</span>
              <span>{[entity].completionPercentage}%</span>
            </div>
            <Progress value={[entity].completionPercentage} />
          </div>
        )}
        
        {/* Team members */}
        {[entity].members && [entity].members.length > 0 && (
          <div className="flex items-center space-x-2 mb-2">
            <Users className="h-4 w-4 text-muted-foreground" />
            <div className="flex -space-x-2">
              {[entity].members.slice(0, 3).map((member) => (
                <Avatar key={member.id} className="h-6 w-6 border-2 border-background">
                  <AvatarImage src={member.user.avatar} />
                  <AvatarFallback className="text-xs">
                    {member.user.firstName?.[0]}{member.user.lastName?.[0]}
                  </AvatarFallback>
                </Avatar>
              ))}
              {[entity].members.length > 3 && (
                <div className="h-6 w-6 rounded-full bg-muted border-2 border-background flex items-center justify-center">
                  <span className="text-xs font-medium">+{[entity].members.length - 3}</span>
                </div>
              )}
            </div>
          </div>
        )}
        
        {/* Due date */}
        {[entity].endDate && (
          <div className="flex items-center space-x-2 text-sm text-muted-foreground">
            <Calendar className="h-4 w-4" />
            <span>Due {format(new Date([entity].endDate), 'MMM d, yyyy')}</span>
          </div>
        )}
      </CardContent>
      
      <CardFooter className="pt-3 border-t">
        <div className="flex justify-between w-full">
          <Button variant="ghost" size="sm" onClick={() => onView?.([entity].id)}>
            View Details
          </Button>
          <div className="flex space-x-1">
            {onEdit && (
              <Button variant="ghost" size="sm" onClick={() => onEdit([entity].id)}>
                Edit
              </Button>
            )}
            <Button variant="ghost" size="sm">
              <MoreHorizontal className="h-4 w-4" />
            </Button>
          </div>
        </div>
      </CardFooter>
    </Card>
  )
}
```

## Hook Patterns

### Data Management Hook Pattern
```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { [queryOperation], [mutationOperation] } from 'wasp/client/operations'
import { toast } from 'react-hot-toast'

interface Use[EntityName]DataProps {
  [entityId]?: string
  filters?: {
    status?: string
    search?: string
  }
  pagination?: {
    page?: number
    limit?: number
  }
}

export const use[EntityName]Data = ({
  [entityId],
  filters = {},
  pagination = { page: 1, limit: 20 }
}: Use[EntityName]DataProps = {}) => {
  const queryClient = useQueryClient()
  
  // Query for list data
  const {
    data: listData,
    isLoading: isLoadingList,
    error: listError,
    refetch: refetchList
  } = useQuery(
    [queryOperation],
    { ...filters, ...pagination },
    {
      enabled: ![entityId], // Only load list when not viewing specific entity
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000 // 10 minutes
    }
  )
  
  // Query for specific entity
  const {
    data: entityData,
    isLoading: isLoadingEntity,
    error: entityError
  } = useQuery(
    [entityQueryOperation],
    { id: [entityId] },
    {
      enabled: !!([entityId]),
      staleTime: 2 * 60 * 1000 // 2 minutes
    }
  )
  
  // Create mutation
  const createMutation = useMutation([createOperation], {
    onSuccess: (newEntity) => {
      // Update cache
      queryClient.setQueryData([queryOperation], (old: any) => {
        if (!old) return old
        return {
          ...old,
          items: [newEntity, ...old.items]
        }
      })
      
      toast.success('[Entity] created successfully')
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to create [entity]')
    }
  })
  
  // Update mutation
  const updateMutation = useMutation([updateOperation], {
    onSuccess: (updatedEntity) => {
      // Update list cache
      queryClient.setQueryData([queryOperation], (old: any) => {
        if (!old) return old
        return {
          ...old,
          items: old.items.map((item: any) => 
            item.id === updatedEntity.id ? updatedEntity : item
          )
        }
      })
      
      // Update entity cache
      queryClient.setQueryData([entityQueryOperation], updatedEntity)
      
      toast.success('[Entity] updated successfully')
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to update [entity]')
    }
  })
  
  // Delete mutation
  const deleteMutation = useMutation([deleteOperation], {
    onSuccess: (_, deletedId) => {
      // Remove from list cache
      queryClient.setQueryData([queryOperation], (old: any) => {
        if (!old) return old
        return {
          ...old,
          items: old.items.filter((item: any) => item.id !== deletedId)
        }
      })
      
      // Remove entity cache
      queryClient.removeQueries([entityQueryOperation, deletedId])
      
      toast.success('[Entity] deleted successfully')
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to delete [entity]')
    }
  })
  
  return {
    // Data
    listData,
    entityData,
    
    // Loading states
    isLoadingList,
    isLoadingEntity,
    isLoading: isLoadingList || isLoadingEntity,
    
    // Errors
    listError,
    entityError,
    error: listError || entityError,
    
    // Actions
    create: createMutation.mutate,
    update: updateMutation.mutate,
    delete: deleteMutation.mutate,
    refetch: refetchList,
    
    // Mutation states
    isCreating: createMutation.isLoading,
    isUpdating: updateMutation.isLoading,
    isDeleting: deleteMutation.isLoading,
    isMutating: createMutation.isLoading || updateMutation.isLoading || deleteMutation.isLoading
  }
}
```

## Error Handling Patterns

### Client-Side Error Handling
```typescript
// Error boundary component
import React from 'react'

interface ErrorBoundaryState {
  hasError: boolean
  error?: Error
}

export class ErrorBoundary extends React.Component<
  React.PropsWithChildren<{}>,
  ErrorBoundaryState
> {
  constructor(props: React.PropsWithChildren<{}>) {
    super(props)
    this.state = { hasError: false }
  }
  
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error }
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('[EntityName] Error:', error, errorInfo)
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="text-center py-12">
          <h2 className="text-xl font-semibold mb-2">Something went wrong</h2>
          <p className="text-muted-foreground mb-4">
            An error occurred while loading this page.
          </p>
          <Button onClick={() => this.setState({ hasError: false })}>
            Try Again
          </Button>
        </div>
      )
    }
    
    return this.props.children
  }
}

// Error message component
interface ErrorMessageProps {
  error: Error | string
  onRetry?: () => void
  message?: string
}

export const ErrorMessage: React.FC<ErrorMessageProps> = ({
  error,
  onRetry,
  message
}) => {
  const errorMessage = typeof error === 'string' ? error : error.message
  
  return (
    <div className="text-center py-8">
      <h3 className="text-lg font-semibold mb-2">Error</h3>
      <p className="text-muted-foreground mb-4">
        {message || errorMessage}
      </p>
      {onRetry && (
        <Button onClick={onRetry} variant="outline">
          Retry
        </Button>
      )}
    </div>
  )
}
```

---

*Use these patterns as starting points and adapt them to specific feature requirements while maintaining consistency across the application.*