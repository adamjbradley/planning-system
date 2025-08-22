# Feature Templates for Claude Code Agents

## Overview

This document provides reusable templates for implementing common feature patterns in the Planning System. Use these templates as starting points to accelerate development while maintaining consistency.

## Complete Feature Implementation Template

### Template Structure
```
src/[feature-name]/
├── operations.ts              # Wasp operations
├── [FeatureName]ListPage.tsx  # List/overview page
├── [FeatureName]DetailsPage.tsx # Detail view page
├── Create[FeatureName]Page.tsx # Creation page (optional)
├── components/
│   ├── [FeatureName]Card.tsx  # Card component for lists
│   ├── [FeatureName]Form.tsx  # Form component
│   ├── [FeatureName]Table.tsx # Table component (optional)
│   └── [FeatureName]Filters.tsx # Filter component
├── hooks/
│   ├── use[FeatureName]Data.tsx # Data management hook
│   └── use[FeatureName]Filters.tsx # Filter state hook
└── utils/
    ├── [featureName]Validation.ts # Validation schemas
    ├── [featureName]Helpers.ts    # Utility functions
    └── [featureName]Types.ts      # TypeScript types
```

## Basic CRUD Feature Template

### 1. Database Schema Template
```prisma
// schema.prisma addition
model [FeatureName] {
  id          String   @id @default(uuid())
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  // Core fields
  name        String
  description String?
  status      [FeatureName]Status @default([DEFAULT_STATUS])
  priority    Priority @default(MEDIUM)
  
  // Relationships
  owner       User     @relation("[FeatureName]Owner", fields: [ownerId], references: [id])
  ownerId     String
  
  project     Project  @relation(fields: [projectId], references: [id], onDelete: Cascade)
  projectId   String
  
  // Additional relationships as needed
  [relatedEntities] [RelatedEntity][]
  
  @@map("[table_name]")
  @@index([ownerId])
  @@index([projectId])
  @@index([status])
  @@index([createdAt])
}

enum [FeatureName]Status {
  [STATUS_1]
  [STATUS_2]
  [STATUS_3]
}
```

### 2. Operations Template
```typescript
// src/[feature-name]/operations.ts
import { HttpError } from 'wasp/server'
import type { 
  Get[FeatureName]List,
  Get[FeatureName]Details,
  Create[FeatureName],
  Update[FeatureName],
  Delete[FeatureName]
} from 'wasp/server/operations'
import type { [FeatureName] } from 'wasp/entities'
import { [FeatureName]Schema } from './utils/[featureName]Validation'

// List query with filtering and pagination
export const get[FeatureName]List: Get[FeatureName]List<ListArgs, ListResponse> = 
  async (args, context) => {
    if (!context.user) {
      throw new HttpError(401, 'Authentication required')
    }
    
    const { projectId, page = 1, limit = 20, status, search } = args
    const skip = (page - 1) * limit
    
    // Verify project access
    await verifyProjectAccess(projectId, context.user.id, context)
    
    const whereClause = {
      projectId,
      ...(status && { status }),
      ...(search && {
        OR: [
          { name: { contains: search, mode: 'insensitive' } },
          { description: { contains: search, mode: 'insensitive' } }
        ]
      })
    }
    
    const [items, total] = await Promise.all([
      context.entities.[FeatureName].findMany({
        where: whereClause,
        include: {
          owner: { select: { id: true, firstName: true, lastName: true, avatar: true } }
        },
        orderBy: { createdAt: 'desc' },
        skip,
        take: limit
      }),
      context.entities.[FeatureName].count({ where: whereClause })
    ])
    
    return {
      items,
      pagination: { page, limit, total, totalPages: Math.ceil(total / limit) }
    }
  }

// Details query
export const get[FeatureName]Details: Get[FeatureName]Details<{ id: string }, [FeatureName]> = 
  async ({ id }, context) => {
    if (!context.user) {
      throw new HttpError(401, 'Authentication required')
    }
    
    const [featureName] = await context.entities.[FeatureName].findUnique({
      where: { id },
      include: {
        owner: { select: { id: true, firstName: true, lastName: true, avatar: true } },
        project: { select: { id: true, name: true } }
      }
    })
    
    if (![featureName]) {
      throw new HttpError(404, '[FeatureName] not found')
    }
    
    // Verify access through project membership
    await verifyProjectAccess([featureName].projectId, context.user.id, context)
    
    return [featureName]
  }

// Create action
export const create[FeatureName]: Create[FeatureName]<CreateArgs, [FeatureName]> = 
  async (args, context) => {
    if (!context.user) {
      throw new HttpError(401, 'Authentication required')
    }
    
    const validatedData = [FeatureName]Schema.parse(args)
    
    // Verify project access
    await verifyProjectAccess(validatedData.projectId, context.user.id, context, 'MEMBER')
    
    const [featureName] = await context.entities.[FeatureName].create({
      data: {
        ...validatedData,
        ownerId: context.user.id
      },
      include: {
        owner: { select: { id: true, firstName: true, lastName: true } },
        project: { select: { id: true, name: true } }
      }
    })
    
    // Create activity log
    await createActivityEntry({
      action: 'CREATED',
      entityType: '[FEATURE_TYPE]',
      entityId: [featureName].id,
      userId: context.user.id,
      projectId: [featureName].projectId
    }, context)
    
    return [featureName]
  }

// Update action
export const update[FeatureName]: Update[FeatureName]<UpdateArgs, [FeatureName]> = 
  async ({ id, ...updates }, context) => {
    if (!context.user) {
      throw new HttpError(401, 'Authentication required')
    }
    
    const existing = await context.entities.[FeatureName].findUnique({
      where: { id },
      include: { project: true }
    })
    
    if (!existing) {
      throw new HttpError(404, '[FeatureName] not found')
    }
    
    // Verify edit permissions
    await verifyProjectAccess(existing.projectId, context.user.id, context, 'MEMBER')
    
    const validatedData = [FeatureName]Schema.partial().parse(updates)
    
    const updated[FeatureName] = await context.entities.[FeatureName].update({
      where: { id },
      data: {
        ...validatedData,
        updatedAt: new Date()
      },
      include: {
        owner: { select: { id: true, firstName: true, lastName: true } },
        project: { select: { id: true, name: true } }
      }
    })
    
    // Create activity log
    await createActivityEntry({
      action: 'UPDATED',
      entityType: '[FEATURE_TYPE]',
      entityId: id,
      userId: context.user.id,
      projectId: updated[FeatureName].projectId
    }, context)
    
    return updated[FeatureName]
  }

// Delete action
export const delete[FeatureName]: Delete[FeatureName]<{ id: string }, { success: boolean }> = 
  async ({ id }, context) => {
    if (!context.user) {
      throw new HttpError(401, 'Authentication required')
    }
    
    const existing = await context.entities.[FeatureName].findUnique({
      where: { id },
      include: { project: true }
    })
    
    if (!existing) {
      throw new HttpError(404, '[FeatureName] not found')
    }
    
    // Verify delete permissions
    await verifyProjectAccess(existing.projectId, context.user.id, context, 'ADMIN')
    
    await context.entities.[FeatureName].delete({ where: { id } })
    
    // Create activity log
    await createActivityEntry({
      action: 'DELETED',
      entityType: '[FEATURE_TYPE]',
      entityId: id,
      userId: context.user.id,
      projectId: existing.projectId
    }, context)
    
    return { success: true }
  }
```

### 3. Main.wasp Additions Template
```wasp
// Add to main.wasp

// Queries
query get[FeatureName]List {
  fn: import { get[FeatureName]List } from "@src/[feature-name]/operations",
  entities: [[FeatureName], Project, ProjectMember]
}

query get[FeatureName]Details {
  fn: import { get[FeatureName]Details } from "@src/[feature-name]/operations",
  entities: [[FeatureName], Project, ProjectMember]
}

// Actions
action create[FeatureName] {
  fn: import { create[FeatureName] } from "@src/[feature-name]/operations",
  entities: [[FeatureName], Project, ProjectMember, ActivityFeed]
}

action update[FeatureName] {
  fn: import { update[FeatureName] } from "@src/[feature-name]/operations",
  entities: [[FeatureName], Project, ProjectMember, ActivityFeed]
}

action delete[FeatureName] {
  fn: import { delete[FeatureName] } from "@src/[feature-name]/operations",
  entities: [[FeatureName], Project, ProjectMember, ActivityFeed]
}

// Routes
route [FeatureName]ListRoute { path: "/projects/:projectId/[feature-name]", to: [FeatureName]ListPage }
page [FeatureName]ListPage {
  component: import [FeatureName]List from "@src/[feature-name]/[FeatureName]ListPage",
  authRequired: true
}

route [FeatureName]DetailsRoute { path: "/projects/:projectId/[feature-name]/:id", to: [FeatureName]DetailsPage }
page [FeatureName]DetailsPage {
  component: import [FeatureName]Details from "@src/[feature-name]/[FeatureName]DetailsPage",
  authRequired: true
}
```

### 4. List Page Template
```typescript
// src/[feature-name]/[FeatureName]ListPage.tsx
import React, { useState } from 'react'
import { useParams } from 'react-router-dom'
import { useQuery } from 'wasp/client/operations'
import { get[FeatureName]List } from 'wasp/client/operations'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { Plus, Search } from 'lucide-react'
import { [FeatureName]Card } from './components/[FeatureName]Card'
import { Create[FeatureName]Modal } from './components/Create[FeatureName]Modal'
import { LoadingSpinner } from '@/components/LoadingSpinner'
import { ErrorMessage } from '@/components/ErrorMessage'

export const [FeatureName]ListPage = () => {
  const { projectId } = useParams()
  const [search, setSearch] = useState('')
  const [status, setStatus] = useState<string>('')
  const [isCreateModalOpen, setIsCreateModalOpen] = useState(false)
  const [page, setPage] = useState(1)
  
  const { data, isLoading, error, refetch } = useQuery(
    get[FeatureName]List,
    { 
      projectId: projectId!,
      page,
      limit: 20,
      ...(search && { search }),
      ...(status && { status })
    }
  )
  
  if (isLoading) {
    return (
      <div className="container mx-auto p-6">
        <div className="flex items-center justify-between mb-6">
          <h1 className="text-3xl font-bold">[Feature Name]</h1>
          <Button disabled>
            <Plus className="h-4 w-4 mr-2" />
            Create New
          </Button>
        </div>
        <LoadingSpinner />
      </div>
    )
  }
  
  if (error) {
    return (
      <div className="container mx-auto p-6">
        <h1 className="text-3xl font-bold mb-6">[Feature Name]</h1>
        <ErrorMessage error={error} onRetry={refetch} />
      </div>
    )
  }
  
  return (
    <div className="container mx-auto p-6">
      <div className="flex items-center justify-between mb-6">
        <h1 className="text-3xl font-bold">[Feature Name]</h1>
        <Button onClick={() => setIsCreateModalOpen(true)}>
          <Plus className="h-4 w-4 mr-2" />
          Create New
        </Button>
      </div>
      
      {/* Filters */}
      <div className="flex gap-4 mb-6">
        <div className="relative flex-1 max-w-md">
          <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
          <Input
            placeholder="Search [feature name]..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="pl-10"
          />
        </div>
        <Select value={status} onValueChange={setStatus}>
          <SelectTrigger className="w-48">
            <SelectValue placeholder="Filter by status" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="">All Status</SelectItem>
            <SelectItem value="STATUS_1">Status 1</SelectItem>
            <SelectItem value="STATUS_2">Status 2</SelectItem>
            <SelectItem value="STATUS_3">Status 3</SelectItem>
          </SelectContent>
        </Select>
      </div>
      
      {/* Content */}
      {data?.items.length === 0 ? (
        <div className="text-center py-12">
          <h3 className="text-lg font-semibold mb-2">No [feature name] found</h3>
          <p className="text-muted-foreground mb-4">
            Create your first [feature name] to get started.
          </p>
          <Button onClick={() => setIsCreateModalOpen(true)}>
            Create Your First [Feature Name]
          </Button>
        </div>
      ) : (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
          {data?.items.map(item => (
            <[FeatureName]Card key={item.id} [featureName]={item} />
          ))}
        </div>
      )}
      
      {/* Pagination */}
      {data && data.pagination.totalPages > 1 && (
        <div className="flex justify-center mt-8">
          <div className="flex gap-2">
            <Button
              variant="outline"
              disabled={page === 1}
              onClick={() => setPage(page - 1)}
            >
              Previous
            </Button>
            <span className="flex items-center px-4">
              Page {page} of {data.pagination.totalPages}
            </span>
            <Button
              variant="outline"
              disabled={page === data.pagination.totalPages}
              onClick={() => setPage(page + 1)}
            >
              Next
            </Button>
          </div>
        </div>
      )}
      
      {/* Create Modal */}
      <Create[FeatureName]Modal
        isOpen={isCreateModalOpen}
        onClose={() => setIsCreateModalOpen(false)}
        projectId={projectId!}
        onSuccess={() => {
          setIsCreateModalOpen(false)
          refetch()
        }}
      />
    </div>
  )
}
```

### 5. Card Component Template
```typescript
// src/[feature-name]/components/[FeatureName]Card.tsx
import React from 'react'
import { format } from 'date-fns'
import { useNavigate } from 'react-router-dom'
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar'
import { Calendar, User } from 'lucide-react'

interface [FeatureName]CardProps {
  [featureName]: [FeatureType]
}

export const [FeatureName]Card: React.FC<[FeatureName]CardProps> = ({ [featureName] }) => {
  const navigate = useNavigate()
  
  const getStatusColor = (status: string) => {
    const colors = {
      'STATUS_1': 'bg-blue-100 text-blue-800',
      'STATUS_2': 'bg-green-100 text-green-800',
      'STATUS_3': 'bg-yellow-100 text-yellow-800'
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
  
  const handleViewDetails = () => {
    navigate(`/projects/${[featureName].projectId}/[feature-name]/${[featureName].id}`)
  }
  
  return (
    <Card className="hover:shadow-lg transition-shadow">
      <CardHeader className="pb-3">
        <div className="flex items-start justify-between">
          <CardTitle className="text-lg line-clamp-1">{[featureName].name}</CardTitle>
          <div className="flex gap-1">
            <Badge className={getStatusColor([featureName].status)}>
              {[featureName].status}
            </Badge>
            <Badge variant="outline" className={getPriorityColor([featureName].priority)}>
              {[featureName].priority}
            </Badge>
          </div>
        </div>
      </CardHeader>
      
      <CardContent className="pb-3">
        {[featureName].description && (
          <p className="text-muted-foreground text-sm line-clamp-2 mb-4">
            {[featureName].description}
          </p>
        )}
        
        <div className="space-y-2">
          <div className="flex items-center gap-2 text-sm text-muted-foreground">
            <User className="h-4 w-4" />
            <Avatar className="h-5 w-5">
              <AvatarImage src={[featureName].owner.avatar} />
              <AvatarFallback className="text-xs">
                {[featureName].owner.firstName?.[0]}{[featureName].owner.lastName?.[0]}
              </AvatarFallback>
            </Avatar>
            <span>{[featureName].owner.firstName} {[featureName].owner.lastName}</span>
          </div>
          
          <div className="flex items-center gap-2 text-sm text-muted-foreground">
            <Calendar className="h-4 w-4" />
            <span>Created {format(new Date([featureName].createdAt), 'MMM d, yyyy')}</span>
          </div>
        </div>
      </CardContent>
      
      <CardFooter className="pt-3 border-t">
        <Button variant="ghost" size="sm" onClick={handleViewDetails} className="w-full">
          View Details
        </Button>
      </CardFooter>
    </Card>
  )
}
```

### 6. Form Component Template
```typescript
// src/[feature-name]/components/[FeatureName]Form.tsx
import React from 'react'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { Button } from '@/components/ui/button'
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { [FeatureName]Schema, type [FeatureName]FormData } from '../utils/[featureName]Validation'

interface [FeatureName]FormProps {
  onSubmit: (data: [FeatureName]FormData) => Promise<void>
  initialData?: Partial<[FeatureName]FormData>
  isSubmitting?: boolean
  onCancel?: () => void
}

export const [FeatureName]Form: React.FC<[FeatureName]FormProps> = ({
  onSubmit,
  initialData,
  isSubmitting = false,
  onCancel
}) => {
  const form = useForm<[FeatureName]FormData>({
    resolver: zodResolver([FeatureName]Schema),
    defaultValues: {
      name: '',
      description: '',
      status: 'STATUS_1',
      priority: 'MEDIUM',
      ...initialData
    }
  })
  
  const handleSubmit = async (data: [FeatureName]FormData) => {
    try {
      await onSubmit(data)
      if (!initialData) {
        form.reset()
      }
    } catch (error) {
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
                <Input placeholder="Enter [feature name] name" {...field} />
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
                  placeholder="Enter [feature name] description" 
                  className="min-h-[100px]"
                  {...field} 
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        
        <div className="grid grid-cols-2 gap-4">
          <FormField
            control={form.control}
            name="status"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Status</FormLabel>
                <Select onValueChange={field.onChange} defaultValue={field.value}>
                  <FormControl>
                    <SelectTrigger>
                      <SelectValue placeholder="Select status" />
                    </SelectTrigger>
                  </FormControl>
                  <SelectContent>
                    <SelectItem value="STATUS_1">Status 1</SelectItem>
                    <SelectItem value="STATUS_2">Status 2</SelectItem>
                    <SelectItem value="STATUS_3">Status 3</SelectItem>
                  </SelectContent>
                </Select>
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
        </div>
        
        <div className="flex justify-end space-x-4">
          {onCancel && (
            <Button type="button" variant="outline" onClick={onCancel}>
              Cancel
            </Button>
          )}
          <Button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Saving...' : initialData ? 'Update' : 'Create'}
          </Button>
        </div>
      </form>
    </Form>
  )
}
```

### 7. Validation Schema Template
```typescript
// src/[feature-name]/utils/[featureName]Validation.ts
import { z } from 'zod'

export const [FeatureName]Schema = z.object({
  name: z.string().min(1, 'Name is required').max(100, 'Name too long'),
  description: z.string().max(1000, 'Description too long').optional(),
  status: z.enum(['STATUS_1', 'STATUS_2', 'STATUS_3']),
  priority: z.enum(['LOW', 'MEDIUM', 'HIGH', 'CRITICAL']).default('MEDIUM'),
  projectId: z.string().uuid('Invalid project ID')
})

export type [FeatureName]FormData = z.infer<typeof [FeatureName]Schema>

export const Update[FeatureName]Schema = [FeatureName]Schema.partial().omit({ projectId: true })

export type Update[FeatureName]Data = z.infer<typeof Update[FeatureName]Schema>
```

### 8. Data Hook Template
```typescript
// src/[feature-name]/hooks/use[FeatureName]Data.tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { 
  get[FeatureName]List,
  get[FeatureName]Details,
  create[FeatureName],
  update[FeatureName],
  delete[FeatureName]
} from 'wasp/client/operations'
import { toast } from 'react-hot-toast'

interface Use[FeatureName]DataProps {
  projectId: string
  [featureName]Id?: string
  filters?: {
    status?: string
    search?: string
  }
}

export const use[FeatureName]Data = ({
  projectId,
  [featureName]Id,
  filters = {}
}: Use[FeatureName]DataProps) => {
  const queryClient = useQueryClient()
  
  // List query
  const {
    data: listData,
    isLoading: isLoadingList,
    error: listError,
    refetch: refetchList
  } = useQuery(
    get[FeatureName]List,
    { projectId, ...filters },
    { enabled: !![featureName]Id }
  )
  
  // Details query
  const {
    data: [featureName]Data,
    isLoading: isLoading[FeatureName],
    error: [featureName]Error
  } = useQuery(
    get[FeatureName]Details,
    { id: [featureName]Id! },
    { enabled: !![featureName]Id }
  )
  
  // Create mutation
  const createMutation = useMutation(create[FeatureName], {
    onSuccess: (new[FeatureName]) => {
      queryClient.setQueryData([get[FeatureName]List], (old: any) => {
        if (!old) return old
        return {
          ...old,
          items: [new[FeatureName], ...old.items]
        }
      })
      toast.success('[FeatureName] created successfully')
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to create [feature name]')
    }
  })
  
  // Update mutation
  const updateMutation = useMutation(update[FeatureName], {
    onSuccess: (updated[FeatureName]) => {
      // Update list cache
      queryClient.setQueryData([get[FeatureName]List], (old: any) => {
        if (!old) return old
        return {
          ...old,
          items: old.items.map((item: any) => 
            item.id === updated[FeatureName].id ? updated[FeatureName] : item
          )
        }
      })
      
      // Update details cache
      queryClient.setQueryData([get[FeatureName]Details], updated[FeatureName])
      
      toast.success('[FeatureName] updated successfully')
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to update [feature name]')
    }
  })
  
  // Delete mutation
  const deleteMutation = useMutation(delete[FeatureName], {
    onSuccess: (_, deletedId) => {
      queryClient.setQueryData([get[FeatureName]List], (old: any) => {
        if (!old) return old
        return {
          ...old,
          items: old.items.filter((item: any) => item.id !== deletedId)
        }
      })
      
      queryClient.removeQueries([get[FeatureName]Details, deletedId])
      
      toast.success('[FeatureName] deleted successfully')
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to delete [feature name]')
    }
  })
  
  return {
    // Data
    listData,
    [featureName]Data,
    
    // Loading states
    isLoadingList,
    isLoading[FeatureName],
    isLoading: isLoadingList || isLoading[FeatureName],
    
    // Errors
    listError,
    [featureName]Error,
    error: listError || [featureName]Error,
    
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

## Usage Instructions

### How to Use These Templates

1. **Replace Placeholders**: Replace all `[FeatureName]`, `[featureName]`, `[feature-name]` placeholders with your actual feature name
2. **Customize Fields**: Modify the database schema fields, form fields, and validation based on your specific requirements
3. **Adjust Relationships**: Update the database relationships to match your feature's needs
4. **Implement Business Logic**: Add feature-specific business logic to the operations
5. **Style Components**: Customize the UI components to match your design requirements
6. **Add Tests**: Create tests following the existing patterns in the codebase

### Placeholder Replacement Guide

- `[FeatureName]` → PascalCase (e.g., `ProjectPhase`, `TaskComment`)
- `[featureName]` → camelCase (e.g., `projectPhase`, `taskComment`)
- `[feature-name]` → kebab-case (e.g., `project-phase`, `task-comment`)
- `[table_name]` → snake_case (e.g., `project_phases`, `task_comments`)
- `[FEATURE_TYPE]` → UPPER_CASE (e.g., `PROJECT_PHASE`, `TASK_COMMENT`)

---

*These templates provide a solid foundation for implementing consistent, well-structured features in the Planning System. Adapt them to your specific requirements while maintaining the established patterns.*