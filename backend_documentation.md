# Backend Documentation for AI Text Editor Application

## Executive Overview

This documentation is designed to help your backend development team build a **production-level server architecture** for an AI-powered text editing application. The backend must handle real-time communication, AI model integration, context management, document processing, and user session management—all while maintaining high performance and security.

As your senior software architect, I've analyzed Kilocode's backend architecture (a VS Code extension) and adapted it for a **standalone web server**. This document provides the blueprint for building a scalable, maintainable, and secure backend system.

---

## Table of Contents

1. [Technology Stack](#technology-stack)
2. [Architecture Overview](#architecture-overview)
3. [Core Backend Components](#core-backend-components)
4. [Communication Layer](#communication-layer)
5. [AI Provider Integration](#ai-provider-integration)
6. [Task & Session Management](#task--session-management)
7. [Context Management System](#context-management-system)
8. [Tool Execution Engine](#tool-execution-engine)
9. [Database Schema](#database-schema)
10. [API Endpoints](#api-endpoints)
11. [Real-time Message Flow](#real-time-message-flow)
12. [Security & Authentication](#security--authentication)
13. [Deployment Architecture](#deployment-architecture)
14. [Implementation Guidelines](#implementation-guidelines)

---

## 1. Technology Stack

### Core Technologies

```json
{
  "runtime": "Node.js 20+ with TypeScript",
  "framework": "Express.js or Fastify",
  "websocket": "Socket.IO or ws library",
  "database": "PostgreSQL 15+ (primary) + Redis (cache/sessions)",
  "orm": "Prisma or TypeORM",
  "aiLibrary": "@anthropic-ai/sdk, openai, google-generative-ai",
  "queue": "Bull (Redis-based job queue)",
  "validation": "Zod",
  "testing": "Vitest + Supertest"
}
```

### Key Dependencies

```bash
npm install express socket.io
npm install @anthropic-ai/sdk openai
npm install prisma @prisma/client
npm install redis bull
npm install zod
npm install jsonwebtoken bcrypt
npm install dotenv
```

---

## 2. Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Web Frontend                            │
│                    (React Application)                       │
└──────────────────────┬──────────────────────────────────────┘
                       │ WebSocket + REST API
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   API Gateway (Express)                      │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐  │
│  │  REST Routes   │  │ WebSocket Hub  │  │ Auth Middleware│
│  └────────────────┘  └────────────────┘  └──────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│   Task      │ │   AI        │ │  Context    │
│   Manager   │ │   Handler   │ │  Manager    │
└─────────────┘ └─────────────┘ └─────────────┘
       │               │               │
       └───────────────┼───────────────┘
                       ▼
       ┌───────────────────────────────┐
       │      Tool Execution Engine     │
       │  (Text Operations: Append,     │
       │   Replace, Delete, Format)     │
       └───────────────────────────────┘
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ PostgreSQL  │ │   Redis     │ │  File       │
│ (Documents) │ │  (Cache)    │ │  Storage    │
└─────────────┘ └─────────────┘ └─────────────┘
```

### Service Layer Architecture

```
src/
├── api/
│   ├── routes/          # HTTP endpoints
│   ├── websocket/       # WebSocket handlers
│   └── middleware/      # Auth, validation, etc.
├── services/
│   ├── task/            # Task lifecycle management
│   ├── ai/              # AI provider integrations
│   ├── context/         # Context management
│   ├── tools/           # Tool execution (text operations)
│   ├── document/        # Document CRUD
│   └── user/            # User management
├── models/              # Database models (Prisma)
├── lib/
│   ├── prompts/         # System prompt generation
│   ├── streaming/       # Response streaming
│   └── utils/           # Utilities
└── workers/             # Background jobs (Bull queue)
```

---

## 3. Core Backend Components

### 3.1 Main Application Server

**File**: `src/server.ts`

```typescript
import express from 'express'
import { createServer } from 'http'
import { Server as SocketIOServer } from 'socket.io'
import { PrismaClient } from '@prisma/client'
import Redis from 'ioredis'

const app = express()
const httpServer = createServer(app)
const io = new SocketIOServer(httpServer, {
  cors: {
    origin: process.env.FRONTEND_URL,
    credentials: true
  }
})

// Initialize services
const prisma = new PrismaClient()
const redis = new Redis(process.env.REDIS_URL)

// Middleware
app.use(express.json({ limit: '50mb' }))
app.use(authMiddleware)

// REST API routes
app.use('/api/auth', authRoutes)
app.use('/api/documents', documentRoutes)
app.use('/api/history', historyRoutes)
app.use('/api/settings', settingsRoutes)

// WebSocket connection handling
io.on('connection', (socket) => {
  console.log('Client connected:', socket.id)
  
  // Authenticate socket
  const userId = socket.handshake.auth.userId
  socket.join(`user:${userId}`)
  
  // Message handlers
  socket.on('newTask', handleNewTask)
  socket.on('askResponse', handleAskResponse)
  socket.on('cancelTask', handleCancelTask)
  
  socket.on('disconnect', () => {
    console.log('Client disconnected:', socket.id)
  })
})

httpServer.listen(3000, () => {
  console.log('Server running on port 3000')
})
```

### 3.2 Task Manager

**Purpose**: Manages AI conversation sessions (tasks).

**File**: `src/services/task/TaskManager.ts`

```typescript
import { EventEmitter } from 'events'
import { PrismaClient } from '@prisma/client'

interface Task {
  id: string
  userId: string
  documentId: string
  status: 'active' | 'paused' | 'completed' | 'error'
  messages: ClineMessage[]
  metadata: {
    tokenUsage: number
    cost: number
    startTime: Date
  }
}

export class TaskManager extends EventEmitter {
  private prisma: PrismaClient
  private activeTasks: Map<string, Task> = new Map()
  
  constructor(prisma: PrismaClient) {
    super()
    this.prisma = prisma
  }
  
  async createTask(userId: string, documentId: string, initialMessage: string): Promise<Task> {
    const task: Task = {
      id: generateUUID(),
      userId,
      documentId,
      status: 'active',
      messages: [
        {
          type: 'ask',
          ask: 'user',
          text: initialMessage,
          ts: Date.now()
        }
      ],
      metadata: {
        tokenUsage: 0,
        cost: 0,
        startTime: new Date()
      }
    }
    
    // Save to database
    await this.prisma.task.create({
      data: {
        id: task.id,
        userId: task.userId,
        documentId: task.documentId,
        status: task.status,
        messages: JSON.stringify(task.messages),
        metadata: JSON.stringify(task.metadata)
      }
    })
    
    this.activeTasks.set(task.id, task)
    this.emit('taskCreated', task)
    
    return task
  }
  
  async addMessage(taskId: string, message: ClineMessage) {
    const task = this.activeTasks.get(taskId)
    if (!task) throw new Error('Task not found')
    
    task.messages.push(message)
    
    // Update database
    await this.prisma.task.update({
      where: { id: taskId },
      data: {
        messages: JSON.stringify(task.messages),
        updatedAt: new Date()
      }
    })
    
    this.emit('messageAdded', taskId, message)
  }
  
  async updateTaskStatus(taskId: string, status: Task['status']) {
    const task = this.activeTasks.get(taskId)
    if (!task) throw new Error('Task not found')
    
    task.status = status
    
    await this.prisma.task.update({
      where: { id: taskId },
      data: { status }
    })
    
    this.emit('taskStatusChanged', taskId, status)
  }
  
  getTask(taskId: string): Task | undefined {
    return this.activeTasks.get(taskId)
  }
}
```

### 3.3 AI Handler (Provider Abstraction)

**Purpose**: Abstracts different AI providers (Anthropic, OpenAI, etc.).

**File**: `src/services/ai/AIHandler.ts`

```typescript
import Anthropic from '@anthropic-ai/sdk'
import OpenAI from 'openai'

export interface AIMessage {
  role: 'user' | 'assistant'
  content: string | Array<{ type: string; text?: string; image?: string }>
}

export interface AIStreamChunk {
  type: 'text' | 'tool_use' | 'thinking'
  content: string
  toolName?: string
  toolInput?: any
}

export abstract class BaseAIHandler {
  abstract createMessage(
    systemPrompt: string,
    messages: AIMessage[],
    options?: {
      model?: string
      maxTokens?: number
      temperature?: number
      tools?: any[]
    }
  ): AsyncIterable<AIStreamChunk>
  
  abstract getModelInfo(): { id: string; maxTokens: number; costPer1kTokens: number }
}

export class AnthropicHandler extends BaseAIHandler {
  private client: Anthropic
  
  constructor(apiKey: string) {
    super()
    this.client = new Anthropic({ apiKey })
  }
  
  async *createMessage(
    systemPrompt: string,
    messages: AIMessage[],
    options = {}
  ): AsyncIterable<AIStreamChunk> {
    const stream = await this.client.messages.stream({
      model: options.model || 'claude-3-5-sonnet-20241022',
      max_tokens: options.maxTokens || 8000,
      system: systemPrompt,
      messages: messages.map(msg => ({
        role: msg.role,
        content: typeof msg.content === 'string' ? msg.content : msg.content
      })),
      tools: options.tools || []
    })
    
    for await (const chunk of stream) {
      if (chunk.type === 'content_block_delta') {
        if (chunk.delta.type === 'text_delta') {
          yield {
            type: 'text',
            content: chunk.delta.text
          }
        } else if (chunk.delta.type === 'tool_use') {
          yield {
            type: 'tool_use',
            content: '',
            toolName: chunk.delta.name,
            toolInput: chunk.delta.input
          }
        }
      }
    }
  }
  
  getModelInfo() {
    return {
      id: 'claude-3-5-sonnet-20241022',
      maxTokens: 200000,
      costPer1kTokens: 0.003
    }
  }
}

export class OpenAIHandler extends BaseAIHandler {
  private client: OpenAI
  
  constructor(apiKey: string) {
    super()
    this.client = new OpenAI({ apiKey })
  }
  
  async *createMessage(
    systemPrompt: string,
    messages: AIMessage[],
    options = {}
  ): AsyncIterable<AIStreamChunk> {
    const stream = await this.client.chat.completions.create({
      model: options.model || 'gpt-4o',
      max_tokens: options.maxTokens || 4000,
      messages: [
        { role: 'system', content: systemPrompt },
        ...messages.map(msg => ({
          role: msg.role,
          content: typeof msg.content === 'string' ? msg.content : JSON.stringify(msg.content)
        }))
      ],
      tools: options.tools,
      stream: true
    })
    
    for await (const chunk of stream) {
      const delta = chunk.choices[0]?.delta
      if (delta?.content) {
        yield {
          type: 'text',
          content: delta.content
        }
      } else if (delta?.tool_calls) {
        for (const toolCall of delta.tool_calls) {
          yield {
            type: 'tool_use',
            content: '',
            toolName: toolCall.function?.name,
            toolInput: JSON.parse(toolCall.function?.arguments || '{}')
          }
        }
      }
    }
  }
  
  getModelInfo() {
    return {
      id: 'gpt-4o',
      maxTokens: 128000,
      costPer1kTokens: 0.005
    }
  }
}

// Factory function
export function createAIHandler(provider: string, apiKey: string): BaseAIHandler {
  switch (provider) {
    case 'anthropic':
      return new AnthropicHandler(apiKey)
    case 'openai':
      return new OpenAIHandler(apiKey)
    default:
      throw new Error(`Unknown provider: ${provider}`)
  }
}
```

---

## 4. Communication Layer

### 4.1 WebSocket Message Handler

**File**: `src/api/websocket/messageHandler.ts`

```typescript
import { Socket } from 'socket.io'
import { TaskManager } from '@/services/task/TaskManager'
import { AIOrchestrator } from '@/services/ai/AIOrchestrator'

export function setupWebSocketHandlers(
  socket: Socket,
  taskManager: TaskManager,
  aiOrchestrator: AIOrchestrator
) {
  // Handle new task creation
  socket.on('newTask', async (data: { text: string; images?: string[]; documentId: string }) => {
    try {
      const userId = socket.data.userId
      
      // Create task
      const task = await taskManager.createTask(userId, data.documentId, data.text)
      
      // Send task created confirmation
      socket.emit('taskCreated', { taskId: task.id })
      
      // Start AI processing (async)
      aiOrchestrator.processTask(task, (update) => {
        socket.emit('messageUpdated', update)
      })
    } catch (error) {
      socket.emit('error', { message: error.message })
    }
  })
  
  // Handle approval/rejection of AI actions
  socket.on('askResponse', async (data: { taskId: string; askTs: number; response: 'approve' | 'reject' }) => {
    try {
      await aiOrchestrator.handleUserResponse(data.taskId, data.askTs, data.response)
    } catch (error) {
      socket.emit('error', { message: error.message })
    }
  })
  
  // Handle task cancellation
  socket.on('cancelTask', async (data: { taskId: string }) => {
    try {
      await aiOrchestrator.cancelTask(data.taskId)
      socket.emit('taskCancelled', { taskId: data.taskId })
    } catch (error) {
      socket.emit('error', { message: error.message })
    }
  })
}
```

### 4.2 REST API Routes

**File**: `src/api/routes/documents.ts`

```typescript
import { Router } from 'express'
import { PrismaClient } from '@prisma/client'
import { authMiddleware } from '@/api/middleware/auth'

const router = Router()
const prisma = new PrismaClient()

// Get all documents for user
router.get('/', authMiddleware, async (req, res) => {
  const userId = req.user.id
  
  const documents = await prisma.document.findMany({
    where: { userId },
    select: {
      id: true,
      title: true,
      content: true,
      createdAt: true,
      updatedAt: true
    },
    orderBy: { updatedAt: 'desc' }
  })
  
  res.json({ documents })
})

// Get single document
router.get('/:id', authMiddleware, async (req, res) => {
  const { id } = req.params
  const userId = req.user.id
  
  const document = await prisma.document.findFirst({
    where: { id, userId }
  })
  
  if (!document) {
    return res.status(404).json({ error: 'Document not found' })
  }
  
  res.json({ document })
})

// Create document
router.post('/', authMiddleware, async (req, res) => {
  const { title, content } = req.body
  const userId = req.user.id
  
  const document = await prisma.document.create({
    data: {
      title,
      content: content || '',
      userId
    }
  })
  
  res.status(201).json({ document })
})

// Update document
router.patch('/:id', authMiddleware, async (req, res) => {
  const { id } = req.params
  const { title, content } = req.body
  const userId = req.user.id
  
  const document = await prisma.document.updateMany({
    where: { id, userId },
    data: { title, content, updatedAt: new Date() }
  })
  
  res.json({ document })
})

// Delete document
router.delete('/:id', authMiddleware, async (req, res) => {
  const { id } = req.params
  const userId = req.user.id
  
  await prisma.document.deleteMany({
    where: { id, userId }
  })
  
  res.status(204).send()
})

export default router
```

---

## 5. AI Provider Integration

### 5.1 AI Orchestrator

**Purpose**: Coordinates AI interactions, tool execution, and streaming.

**File**: `src/services/ai/AIOrchestrator.ts`

```typescript
import { BaseAIHandler } from './AIHandler'
import { TaskManager } from '../task/TaskManager'
import { ToolExecutor } from '../tools/ToolExecutor'
import { ContextManager } from '../context/ContextManager'
import { generateSystemPrompt } from '@/lib/prompts/system'

export class AIOrchestrator {
  constructor(
    private aiHandler: BaseAIHandler,
    private taskManager: TaskManager,
    private toolExecutor: ToolExecutor,
    private contextManager: ContextManager
  ) {}
  
  async processTask(task: Task, onUpdate: (message: ClineMessage) => void) {
    try {
      // Generate system prompt
      const systemPrompt = await generateSystemPrompt({
        userId: task.userId,
        documentId: task.documentId,
        contextItems: await this.contextManager.getContext(task.id)
      })
      
      // Convert task messages to AI format
      const aiMessages = this.convertToAIMessages(task.messages)
      
      // Stream AI response
      let currentMessage: ClineMessage = {
        type: 'say',
        say: 'text',
        text: '',
        ts: Date.now()
      }
      
      for await (const chunk of this.aiHandler.createMessage(systemPrompt, aiMessages, {
        tools: this.getAvailableTools()
      })) {
        if (chunk.type === 'text') {
          currentMessage.text += chunk.content
          onUpdate(currentMessage)
        } else if (chunk.type === 'tool_use') {
          // AI wants to execute a tool
          const toolMessage: ClineMessage = {
            type: 'say',
            say: 'tool_use',
            tool: {
              name: chunk.toolName!,
              input: chunk.toolInput
            },
            ts: Date.now()
          }
          
          await this.taskManager.addMessage(task.id, toolMessage)
          onUpdate(toolMessage)
          
          // Request user approval
          const approvalMessage: ClineMessage = {
            type: 'ask',
            ask: 'tool',
            text: `Approve ${chunk.toolName}?`,
            ts: Date.now()
          }
          
          await this.taskManager.addMessage(task.id, approvalMessage)
          onUpdate(approvalMessage)
          
          // Wait for user response (handled by handleUserResponse)
          await this.waitForApproval(task.id, approvalMessage.ts)
        }
      }
      
      // Finalize task
      await this.taskManager.addMessage(task.id, currentMessage)
      await this.taskManager.updateTaskStatus(task.id, 'completed')
      
    } catch (error) {
      console.error('Error processing task:', error)
      await this.taskManager.updateTaskStatus(task.id, 'error')
    }
  }
  
  async handleUserResponse(taskId: string, askTs: number, response: 'approve' | 'reject') {
    if (response === 'approve') {
      const task = this.taskManager.getTask(taskId)
      const message = task?.messages.find(m => m.ts === askTs)
      
      if (message && message.type === 'ask' && message.ask === 'tool') {
        // Find the tool use message before this ask
        const toolMessage = task.messages[task.messages.length - 2]
        if (toolMessage.type === 'say' && toolMessage.say === 'tool_use') {
          // Execute tool
          const result = await this.toolExecutor.executeTool(
            toolMessage.tool.name,
            toolMessage.tool.input,
            { taskId, documentId: task.documentId }
          )
          
          // Add result to conversation
          const resultMessage: ClineMessage = {
            type: 'say',
            say: 'tool_result',
            text: result.output,
            ts: Date.now()
          }
          
          await this.taskManager.addMessage(taskId, resultMessage)
        }
      }
    }
    
    // Notify approval received
    this.emit(`approval:${taskId}:${askTs}`, response)
  }
  
  private waitForApproval(taskId: string, askTs: number): Promise<'approve' | 'reject'> {
    return new Promise((resolve) => {
      this.once(`approval:${taskId}:${askTs}`, resolve)
    })
  }
  
  private getAvailableTools() {
    return [
      {
        name: 'append_text',
        description: 'Append text to the document at a specific position or end',
        input_schema: {
          type: 'object',
          properties: {
            text: { type: 'string', description: 'Text to append' },
            position: { type: 'string', description: 'Position: "end", "cursor", or line number' }
          },
          required: ['text']
        }
      },
      {
        name: 'replace_text',
        description: 'Replace text in the document',
        input_schema: {
          type: 'object',
          properties: {
            old_text: { type: 'string', description: 'Text to find' },
            new_text: { type: 'string', description: 'Replacement text' },
            all_occurrences: { type: 'boolean', description: 'Replace all occurrences' }
          },
          required: ['old_text', 'new_text']
        }
      },
      // ... more tools
    ]
  }
  
  private convertToAIMessages(messages: ClineMessage[]): AIMessage[] {
    return messages.map(msg => {
      if (msg.type === 'ask') {
        return {
          role: 'user',
          content: msg.text
        }
      } else if (msg.type === 'say') {
        return {
          role: 'assistant',
          content: msg.text
        }
      }
      return null
    }).filter(Boolean) as AIMessage[]
  }
}
```

---

## 6. Task & Session Management

### 6.1 Session Store (Redis)

```typescript
import Redis from 'ioredis'

export class SessionStore {
  private redis: Redis
  
  constructor(redisUrl: string) {
    this.redis = new Redis(redisUrl)
  }
  
  async saveSession(sessionId: string, data: any, ttl: number = 3600) {
    await this.redis.setex(`session:${sessionId}`, ttl, JSON.stringify(data))
  }
  
  async getSession(sessionId: string): Promise<any | null> {
    const data = await this.redis.get(`session:${sessionId}`)
    return data ? JSON.parse(data) : null
  }
  
  async deleteSession(sessionId: string) {
    await this.redis.del(`session:${sessionId}`)
  }
}
```

### 6.2 Task Persistence

Tasks are saved to PostgreSQL for durability.

```typescript
// Prisma schema (prisma/schema.prisma)
model Task {
  id          String   @id @default(uuid())
  userId      String
  documentId  String
  status      String   // 'active', 'paused', 'completed', 'error'
  messages    Json     // Array of ClineMessage
  metadata    Json     // { tokenUsage, cost, startTime }
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  user        User     @relation(fields: [userId], references: [id])
  document    Document @relation(fields: [documentId], references: [id])
  
  @@index([userId, status])
}
```

---

## 7. Context Management System

### 7.1 Context Manager

**Purpose**: Manages additional information the AI needs (documents, style guides, etc.).

**File**: `src/services/context/ContextManager.ts`

```typescript
export interface ContextItem {
  type: 'document' | 'style_guide' | 'glossary' | 'url' | 'custom'
  content: string
  metadata?: Record<string, any>
}

export class ContextManager {
  constructor(private prisma: PrismaClient) {}
  
  async getContext(taskId: string): Promise<ContextItem[]> {
    const task = await this.prisma.task.findUnique({
      where: { id: taskId },
      include: { document: true }
    })
    
    if (!task) return []
    
    const contextItems: ContextItem[] = []
    
    // Add current document
    contextItems.push({
      type: 'document',
      content: task.document.content,
      metadata: { title: task.document.title }
    })
    
    // Add style guide if exists
    const styleGuide = await this.prisma.styleGuide.findFirst({
      where: { userId: task.userId }
    })
    if (styleGuide) {
      contextItems.push({
        type: 'style_guide',
        content: styleGuide.content
      })
    }
    
    // Add glossary
    const glossary = await this.prisma.glossary.findMany({
      where: { userId: task.userId }
    })
    if (glossary.length > 0) {
      contextItems.push({
        type: 'glossary',
        content: glossary.map(g => `${g.term}: ${g.definition}`).join('\n')
      })
    }
    
    return contextItems
  }
  
  async addContextItem(userId: string, item: ContextItem) {
    // Store custom context items
    await this.prisma.contextItem.create({
      data: {
        userId,
        type: item.type,
        content: item.content,
        metadata: item.metadata || {}
      }
    })
  }
}
```

### 7.2 Mention Resolver

Resolve `@mentions` in user input (e.g., `@style-guide`, `@document:file.txt`).

```typescript
export class MentionResolver {
  async resolve(text: string, userId: string): Promise<ContextItem[]> {
    const mentions = this.parseMentions(text)
    const resolvedItems: ContextItem[] = []
    
    for (const mention of mentions) {
      switch (mention.type) {
        case 'style-guide':
          const styleGuide = await prisma.styleGuide.findFirst({ where: { userId } })
          if (styleGuide) {
            resolvedItems.push({
              type: 'style_guide',
              content: styleGuide.content
            })
          }
          break
        
        case 'document':
          const doc = await prisma.document.findFirst({
            where: { userId, title: mention.value }
          })
          if (doc) {
            resolvedItems.push({
              type: 'document',
              content: doc.content,
              metadata: { title: doc.title }
            })
          }
          break
        
        case 'url':
          const content = await this.fetchUrl(mention.value)
          resolvedItems.push({
            type: 'url',
            content,
            metadata: { url: mention.value }
          })
          break
      }
    }
    
    return resolvedItems
  }
  
  private parseMentions(text: string): Array<{ type: string; value?: string }> {
    const regex = /@(\w+)(?::(.+?))?(?:\s|$)/g
    const mentions = []
    let match
    
    while ((match = regex.exec(text)) !== null) {
      mentions.push({
        type: match[1],
        value: match[2]
      })
    }
    
    return mentions
  }
  
  private async fetchUrl(url: string): Promise<string> {
    const response = await fetch(url)
    return await response.text()
  }
}
```

---

## 8. Tool Execution Engine

### 8.1 Tool Executor

**Purpose**: Executes text modification tools.

**File**: `src/services/tools/ToolExecutor.ts`

```typescript
import { PrismaClient } from '@prisma/client'

export interface ToolResult {
  success: boolean
  output: string
  updatedContent?: string
}

export class ToolExecutor {
  constructor(private prisma: PrismaClient) {}
  
  async executeTool(
    toolName: string,
    input: any,
    context: { taskId: string; documentId: string }
  ): Promise<ToolResult> {
    switch (toolName) {
      case 'append_text':
        return await this.appendText(input, context)
      
      case 'replace_text':
        return await this.replaceText(input, context)
      
      case 'delete_text':
        return await this.deleteText(input, context)
      
      case 'format_text':
        return await this.formatText(input, context)
      
      default:
        throw new Error(`Unknown tool: ${toolName}`)
    }
  }
  
  private async appendText(
    input: { text: string; position?: string },
    context: { documentId: string }
  ): Promise<ToolResult> {
    const document = await this.prisma.document.findUnique({
      where: { id: context.documentId }
    })
    
    if (!document) {
      return { success: false, output: 'Document not found' }
    }
    
    let updatedContent = document.content
    
    if (input.position === 'end' || !input.position) {
      updatedContent += '\n' + input.text
    } else if (input.position === 'cursor') {
      // For cursor position, you'd need to track cursor in frontend
      updatedContent += '\n' + input.text
    } else if (!isNaN(Number(input.position))) {
      // Insert at line number
      const lines = updatedContent.split('\n')
      lines.splice(Number(input.position), 0, input.text)
      updatedContent = lines.join('\n')
    }
    
    // Update document
    await this.prisma.document.update({
      where: { id: context.documentId },
      data: { content: updatedContent, updatedAt: new Date() }
    })
    
    return {
      success: true,
      output: `Appended ${input.text.length} characters to document`,
      updatedContent
    }
  }
  
  private async replaceText(
    input: { old_text: string; new_text: string; all_occurrences?: boolean },
    context: { documentId: string }
  ): Promise<ToolResult> {
    const document = await this.prisma.document.findUnique({
      where: { id: context.documentId }
    })
    
    if (!document) {
      return { success: false, output: 'Document not found' }
    }
    
    let updatedContent = document.content
    let count = 0
    
    if (input.all_occurrences) {
      const regex = new RegExp(this.escapeRegex(input.old_text), 'g')
      updatedContent = updatedContent.replace(regex, () => {
        count++
        return input.new_text
      })
    } else {
      const index = updatedContent.indexOf(input.old_text)
      if (index !== -1) {
        updatedContent = updatedContent.slice(0, index) + input.new_text + updatedContent.slice(index + input.old_text.length)
        count = 1
      }
    }
    
    await this.prisma.document.update({
      where: { id: context.documentId },
      data: { content: updatedContent, updatedAt: new Date() }
    })
    
    return {
      success: true,
      output: `Replaced ${count} occurrence(s)`,
      updatedContent
    }
  }
  
  private async deleteText(
    input: { text: string },
    context: { documentId: string }
  ): Promise<ToolResult> {
    const document = await this.prisma.document.findUnique({
      where: { id: context.documentId }
    })
    
    if (!document) {
      return { success: false, output: 'Document not found' }
    }
    
    const updatedContent = document.content.replace(input.text, '')
    
    await this.prisma.document.update({
      where: { id: context.documentId },
      data: { content: updatedContent, updatedAt: new Date() }
    })
    
    return {
      success: true,
      output: 'Text deleted',
      updatedContent
    }
  }
  
  private async formatText(
    input: { format: 'bold' | 'italic' | 'heading' | 'list'; text: string },
    context: { documentId: string }
  ): Promise<ToolResult> {
    let formattedText = input.text
    
    switch (input.format) {
      case 'bold':
        formattedText = `**${input.text}**`
        break
      case 'italic':
        formattedText = `*${input.text}*`
        break
      case 'heading':
        formattedText = `## ${input.text}`
        break
      case 'list':
        formattedText = input.text.split('\n').map(line => `- ${line}`).join('\n')
        break
    }
    
    // This would typically replace the selected text
    // For simplicity, we'll append it
    return await this.appendText({ text: formattedText }, context)
  }
  
  private escapeRegex(str: string): string {
    return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
  }
}
```

---

## 9. Database Schema

### 9.1 Prisma Schema

**File**: `prisma/schema.prisma`

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id              String         @id @default(uuid())
  email           String         @unique
  passwordHash    String
  name            String?
  createdAt       DateTime       @default(now())
  updatedAt       DateTime       @updatedAt
  
  documents       Document[]
  tasks           Task[]
  apiKeys         ApiKey[]
  styleGuides     StyleGuide[]
  glossaryEntries GlossaryEntry[]
  settings        UserSettings?
  
  @@index([email])
}

model Document {
  id          String   @id @default(uuid())
  userId      String
  title       String
  content     String   @db.Text
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  tasks       Task[]
  checkpoints Checkpoint[]
  
  @@index([userId, updatedAt])
}

model Task {
  id          String   @id @default(uuid())
  userId      String
  documentId  String
  status      String   // 'active', 'paused', 'completed', 'error'
  messages    Json     // Array of ClineMessage
  metadata    Json     // { tokenUsage, cost, startTime }
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  document    Document @relation(fields: [documentId], references: [id], onDelete: Cascade)
  
  @@index([userId, status])
  @@index([documentId])
}

model Checkpoint {
  id          String   @id @default(uuid())
  documentId  String
  content     String   @db.Text
  description String?
  createdAt   DateTime @default(now())
  
  document    Document @relation(fields: [documentId], references: [id], onDelete: Cascade)
  
  @@index([documentId, createdAt])
}

model StyleGuide {
  id          String   @id @default(uuid())
  userId      String
  title       String
  content     String   @db.Text
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([userId])
}

model GlossaryEntry {
  id          String   @id @default(uuid())
  userId      String
  term        String
  definition  String   @db.Text
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([userId])
}

model ApiKey {
  id          String   @id @default(uuid())
  userId      String
  provider    String   // 'anthropic', 'openai', etc.
  keyHash     String   // Hashed API key
  name        String?
  createdAt   DateTime @default(now())
  
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([userId])
}

model UserSettings {
  id                    String   @id @default(uuid())
  userId                String   @unique
  defaultProvider       String   @default("anthropic")
  defaultModel          String   @default("claude-3-5-sonnet-20241022")
  autoApprovalEnabled   Boolean  @default(false)
  maxTokensPerRequest   Int      @default(8000)
  temperature           Float    @default(0.7)
  
  user                  User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}
```

---

## 10. API Endpoints

### Complete REST API

```
Authentication
POST   /api/auth/register          # Create account
POST   /api/auth/login             # Login
POST   /api/auth/logout            # Logout
GET    /api/auth/me                # Get current user

Documents
GET    /api/documents              # List user's documents
GET    /api/documents/:id          # Get document by ID
POST   /api/documents              # Create document
PATCH  /api/documents/:id          # Update document
DELETE /api/documents/:id          # Delete document

Tasks (History)
GET    /api/tasks                  # List user's tasks
GET    /api/tasks/:id              # Get task by ID
DELETE /api/tasks/:id              # Delete task

Checkpoints
GET    /api/checkpoints/:documentId # List checkpoints for document
POST   /api/checkpoints            # Create checkpoint
POST   /api/checkpoints/:id/restore # Restore checkpoint

Settings
GET    /api/settings               # Get user settings
PATCH  /api/settings               # Update settings

Style Guides
GET    /api/style-guides           # List style guides
POST   /api/style-guides           # Create style guide
PATCH  /api/style-guides/:id       # Update style guide
DELETE /api/style-guides/:id       # Delete style guide

Glossary
GET    /api/glossary               # List glossary entries
POST   /api/glossary               # Create entry
DELETE /api/glossary/:id           # Delete entry

AI Providers
GET    /api/providers/models       # List available models
POST   /api/providers/test         # Test API key
```

---

## 11. Real-time Message Flow

### Complete Flow Diagram

```
User Input
    │
    ├──▶ Frontend: User types "Append 'Hello World' to end"
    │
    ▼
WebSocket: newTask
    │
    ├──▶ Backend: TaskManager.createTask()
    │    │
    │    ├──▶ Save to DB
    │    └──▶ Emit taskCreated
    │
    ▼
AIOrchestrator.processTask()
    │
    ├──▶ Generate system prompt (ContextManager)
    │
    ├──▶ AIHandler.createMessage() → Stream response
    │    │
    │    ├──▶ Chunk 1 (text): "I'll help you..."
    │    │    └──▶ WebSocket: messageUpdated → Frontend
    │    │
    │    ├──▶ Chunk 2 (tool_use): append_text
    │    │    └──▶ WebSocket: messageUpdated → Frontend
    │    │
    │    └──▶ Wait for user approval
    │
    ▼
User Approval
    │
    ├──▶ Frontend: User clicks "Approve"
    │
    ▼
WebSocket: askResponse { response: 'approve' }
    │
    ├──▶ Backend: AIOrchestrator.handleUserResponse()
    │    │
    │    └──▶ ToolExecutor.executeTool('append_text', {...})
    │         │
    │         ├──▶ Update document in DB
    │         └──▶ Return result
    │
    ▼
WebSocket: documentUpdated
    │
    └──▶ Frontend: Update document view
```

---

## 12. Security & Authentication

### 12.1 JWT Authentication

**File**: `src/api/middleware/auth.ts`

```typescript
import jwt from 'jsonwebtoken'
import { Request, Response, NextFunction } from 'express'

interface JWTPayload {
  userId: string
  email: string
}

export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '')
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' })
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload
    req.user = { id: decoded.userId, email: decoded.email }
    next()
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' })
  }
}
```

### 12.2 API Key Storage

Store user AI provider API keys **encrypted** in the database.

```typescript
import crypto from 'crypto'

const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY! // 32-byte key
const ALGORITHM = 'aes-256-cbc'

export function encryptApiKey(apiKey: string): string {
  const iv = crypto.randomBytes(16)
  const cipher = crypto.createCipheriv(ALGORITHM, Buffer.from(ENCRYPTION_KEY, 'hex'), iv)
  let encrypted = cipher.update(apiKey, 'utf8', 'hex')
  encrypted += cipher.final('hex')
  return iv.toString('hex') + ':' + encrypted
}

export function decryptApiKey(encryptedKey: string): string {
  const [ivHex, encrypted] = encryptedKey.split(':')
  const iv = Buffer.from(ivHex, 'hex')
  const decipher = crypto.createDecipheriv(ALGORITHM, Buffer.from(ENCRYPTION_KEY, 'hex'), iv)
  let decrypted = decipher.update(encrypted, 'hex', 'utf8')
  decrypted += decipher.final('utf8')
  return decrypted
}
```

### 12.3 Rate Limiting

```typescript
import rateLimit from 'express-rate-limit'

export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests, please try again later'
})

app.use('/api/', apiLimiter)
```

---

## 13. Deployment Architecture

### 13.1 Docker Setup

**File**: `Dockerfile`

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile

COPY . .
RUN pnpm build

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

**File**: `docker-compose.yml`

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/textai
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
    depends_on:
      - postgres
      - redis
  
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=textai
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 13.2 Production Checklist

- ✅ Enable HTTPS (SSL certificate)
- ✅ Set up reverse proxy (Nginx)
- ✅ Configure environment variables
- ✅ Set up database backups
- ✅ Enable logging (Winston, Pino)
- ✅ Set up monitoring (Prometheus, Grafana)
- ✅ Configure CORS properly
- ✅ Enable rate limiting
- ✅ Encrypt sensitive data

---

## 14. Implementation Guidelines

### 14.1 Development Workflow

1. **Start with MVP**:
   - User authentication
   - Document CRUD
   - Basic AI conversation
   - Single tool (append_text)

2. **Iterate**:
   - Add more tools
   - Implement checkpoints
   - Add context management
   - Optimize performance

3. **Test Continuously**:
   - Unit tests for tools
   - Integration tests for API
   - Load testing for WebSocket

### 14.2 Code Quality

- **TypeScript**: Strong typing everywhere
- **Linting**: ESLint with strict rules
- **Formatting**: Prettier
- **Testing**: ≥80% code coverage
- **Documentation**: JSDoc for public APIs

### 14.3 Performance Tips

- **Database**:
  - Use indexes on frequently queried fields
  - Implement connection pooling
  - Cache frequent queries in Redis

- **WebSocket**:
  - Use Redis adapter for horizontal scaling
  - Implement heartbeat/ping-pong
  - Handle reconnection gracefully

- **AI Requests**:
  - Stream responses to avoid timeout
  - Implement request queuing
  - Cache expensive prompts

---

## Final Notes for Your Team

### Success Criteria

- Users can authenticate and manage documents
- AI conversations are real-time and responsive
- Tool execution is reliable and safe
- System handles 100+ concurrent users
- All data is persisted reliably

### Key Differentiators

1. **Safety**: All AI actions require user approval (unless auto-approved)
2. **Context-aware**: AI has access to style guides, glossaries, etc.
3. **Versioning**: Checkpoints allow undo/redo
4. **Scalable**: Horizontal scaling via Redis + load balancer

### Next Steps

1. Set up development environment
2. Initialize database with Prisma
3. Build authentication system
4. Implement document API
5. Build WebSocket server
6. Integrate AI provider (Anthropic/OpenAI)
7. Implement tool execution engine
8. Add context management
9. Build frontend (see frontend_documentation.md)
10. Test end-to-end

---

**Good luck building an exceptional AI text editor backend! This documentation should provide a solid foundation for your team.**

---

*Document Version: 1.0*  
*Last Updated: [Current Date]*  
*Author: Senior Software Architect*
