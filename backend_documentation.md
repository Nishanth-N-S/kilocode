# Backend Documentation for AI Text Manipulation Website

## Overview

This document provides comprehensive backend architecture documentation for building an AI-powered text manipulation website. The backend is responsible for handling AI model interactions, managing text operations, user authentication, data persistence, and real-time communication with the frontend.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Technology Stack](#technology-stack)
3. [Project Structure](#project-structure)
4. [Core Components](#core-components)
5. [API Design](#api-design)
6. [AI Integration](#ai-integration)
7. [Tool System](#tool-system)
8. [Context Management](#context-management)
9. [Data Models](#data-models)
10. [Authentication & Authorization](#authentication--authorization)
11. [Real-time Communication](#real-time-communication)
12. [Database Design](#database-design)
13. [Deployment Architecture](#deployment-architecture)

---

## Architecture Overview

The backend follows a layered architecture pattern with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                        API Gateway Layer                     │
│              (Express.js / FastAPI Routes)                   │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                      Service Layer                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Chat Service │  │ Text Service │  │  User Service    │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                      Business Logic Layer                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ AI Handler   │  │ Tool System  │  │ Context Manager  │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                      Data Access Layer                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ PostgreSQL   │  │    Redis     │  │  File Storage    │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                    External Services Layer                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ OpenAI API   │  │ Anthropic    │  │  Other LLMs      │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Technology Stack

### Primary Technologies

**Option 1: Node.js/TypeScript Stack**
```json
{
  "runtime": "Node.js 20+",
  "language": "TypeScript 5.x",
  "framework": "Express.js 4.x",
  "database": "PostgreSQL 15+",
  "cache": "Redis 7+",
  "orm": "Prisma 5.x or TypeORM 0.3.x",
  "websocket": "Socket.io 4.x",
  "testing": "Jest + Supertest",
  "validation": "Zod or Joi"
}
```

**Option 2: Python Stack**
```json
{
  "runtime": "Python 3.11+",
  "framework": "FastAPI 0.104+",
  "database": "PostgreSQL 15+",
  "cache": "Redis 7+",
  "orm": "SQLAlchemy 2.x",
  "websocket": "FastAPI WebSocket / Socket.io",
  "testing": "pytest + httpx",
  "validation": "Pydantic V2"
}
```

### Supporting Technologies

```json
{
  "ai_sdks": [
    "@anthropic-ai/sdk",
    "openai",
    "@google/genai"
  ],
  "queue": "Bull (Node.js) / Celery (Python)",
  "monitoring": "Prometheus + Grafana",
  "logging": "Winston (Node.js) / Loguru (Python)",
  "authentication": "JWT + Passport.js / FastAPI-Users",
  "file_storage": "AWS S3 / MinIO"
}
```

---

## Project Structure

### Node.js/TypeScript Structure

```
backend/
├── src/
│   ├── api/
│   │   ├── routes/
│   │   │   ├── auth.routes.ts
│   │   │   ├── chat.routes.ts
│   │   │   ├── document.routes.ts
│   │   │   ├── user.routes.ts
│   │   │   └── index.ts
│   │   ├── middleware/
│   │   │   ├── auth.middleware.ts
│   │   │   ├── validation.middleware.ts
│   │   │   ├── error.middleware.ts
│   │   │   └── rateLimit.middleware.ts
│   │   └── validators/
│   │       ├── chat.validator.ts
│   │       └── document.validator.ts
│   ├── services/
│   │   ├── ai/
│   │   │   ├── AIService.ts
│   │   │   ├── providers/
│   │   │   │   ├── OpenAIProvider.ts
│   │   │   │   ├── AnthropicProvider.ts
│   │   │   │   └── BaseProvider.ts
│   │   │   └── streaming/
│   │   │       └── StreamHandler.ts
│   │   ├── chat/
│   │   │   ├── ChatService.ts
│   │   │   ├── MessageService.ts
│   │   │   └── ConversationService.ts
│   │   ├── document/
│   │   │   ├── DocumentService.ts
│   │   │   ├── VersionService.ts
│   │   │   └── StorageService.ts
│   │   ├── text/
│   │   │   ├── TextManipulationService.ts
│   │   │   ├── AppendService.ts
│   │   │   ├── ModifyService.ts
│   │   │   └── RemoveService.ts
│   │   └── user/
│   │       ├── UserService.ts
│   │       └── AuthService.ts
│   ├── core/
│   │   ├── tools/
│   │   │   ├── ToolManager.ts
│   │   │   ├── BaseTool.ts
│   │   │   ├── AppendTool.ts
│   │   │   ├── ModifyTool.ts
│   │   │   ├── RemoveTool.ts
│   │   │   └── SearchTool.ts
│   │   ├── context/
│   │   │   ├── ContextManager.ts
│   │   │   ├── ContextAssembler.ts
│   │   │   └── ContextPruner.ts
│   │   ├── prompts/
│   │   │   ├── PromptBuilder.ts
│   │   │   ├── SystemPrompts.ts
│   │   │   └── templates/
│   │   └── streaming/
│   │       ├── StreamProcessor.ts
│   │       └── ResponseFormatter.ts
│   ├── database/
│   │   ├── models/
│   │   │   ├── User.ts
│   │   │   ├── Document.ts
│   │   │   ├── Chat.ts
│   │   │   ├── Message.ts
│   │   │   └── Version.ts
│   │   ├── repositories/
│   │   │   ├── UserRepository.ts
│   │   │   ├── DocumentRepository.ts
│   │   │   └── ChatRepository.ts
│   │   ├── migrations/
│   │   └── seeds/
│   ├── utils/
│   │   ├── logger.ts
│   │   ├── errors.ts
│   │   ├── validation.ts
│   │   └── crypto.ts
│   ├── config/
│   │   ├── database.config.ts
│   │   ├── redis.config.ts
│   │   ├── ai.config.ts
│   │   └── app.config.ts
│   ├── types/
│   │   ├── api.types.ts
│   │   ├── chat.types.ts
│   │   ├── document.types.ts
│   │   └── ai.types.ts
│   └── app.ts
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── scripts/
│   ├── setup-db.ts
│   └── seed-data.ts
├── .env.example
├── .env
├── package.json
├── tsconfig.json
├── docker-compose.yml
└── Dockerfile
```

---

## Core Components

### 1. AI Service (`AIService.ts`)

The AI Service is the central component for managing interactions with various AI providers.

```typescript
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export interface AIProvider {
  name: string;
  generateCompletion(request: CompletionRequest): Promise<CompletionResponse>;
  generateStream(request: CompletionRequest): AsyncIterable<StreamChunk>;
  getAvailableModels(): Promise<Model[]>;
}

export interface CompletionRequest {
  messages: Message[];
  model: string;
  temperature?: number;
  maxTokens?: number;
  stream?: boolean;
  tools?: Tool[];
}

export interface CompletionResponse {
  content: string;
  model: string;
  usage: {
    promptTokens: number;
    completionTokens: number;
    totalTokens: number;
  };
  toolCalls?: ToolCall[];
}

export class AIService {
  private providers: Map<string, AIProvider> = new Map();
  private defaultProvider: string = 'anthropic';

  constructor() {
    this.initializeProviders();
  }

  private initializeProviders(): void {
    // Initialize Anthropic
    const anthropicProvider = new AnthropicProvider({
      apiKey: process.env.ANTHROPIC_API_KEY!,
    });
    this.providers.set('anthropic', anthropicProvider);

    // Initialize OpenAI
    const openaiProvider = new OpenAIProvider({
      apiKey: process.env.OPENAI_API_KEY!,
    });
    this.providers.set('openai', openaiProvider);
  }

  async generateCompletion(
    request: CompletionRequest,
    providerName?: string
  ): Promise<CompletionResponse> {
    const provider = this.providers.get(providerName || this.defaultProvider);
    if (!provider) {
      throw new Error(`Provider ${providerName} not found`);
    }

    return await provider.generateCompletion(request);
  }

  async *generateStream(
    request: CompletionRequest,
    providerName?: string
  ): AsyncIterable<StreamChunk> {
    const provider = this.providers.get(providerName || this.defaultProvider);
    if (!provider) {
      throw new Error(`Provider ${providerName} not found`);
    }

    yield* provider.generateStream(request);
  }

  getProvider(name: string): AIProvider | undefined {
    return this.providers.get(name);
  }

  listProviders(): string[] {
    return Array.from(this.providers.keys());
  }
}
```

### 2. Anthropic Provider Implementation

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

export class AnthropicProvider implements AIProvider {
  name = 'anthropic';
  private client: Anthropic;

  constructor(config: { apiKey: string }) {
    this.client = new Anthropic({
      apiKey: config.apiKey,
    });
  }

  async generateCompletion(request: CompletionRequest): Promise<CompletionResponse> {
    const response = await this.client.messages.create({
      model: request.model || 'claude-3-5-sonnet-20241022',
      max_tokens: request.maxTokens || 4096,
      temperature: request.temperature || 0.7,
      messages: request.messages.map(msg => ({
        role: msg.role as 'user' | 'assistant',
        content: msg.content,
      })),
      tools: request.tools?.map(tool => ({
        name: tool.name,
        description: tool.description,
        input_schema: tool.inputSchema,
      })),
    });

    return {
      content: this.extractContent(response),
      model: response.model,
      usage: {
        promptTokens: response.usage.input_tokens,
        completionTokens: response.usage.output_tokens,
        totalTokens: response.usage.input_tokens + response.usage.output_tokens,
      },
      toolCalls: this.extractToolCalls(response),
    };
  }

  async *generateStream(request: CompletionRequest): AsyncIterable<StreamChunk> {
    const stream = await this.client.messages.stream({
      model: request.model || 'claude-3-5-sonnet-20241022',
      max_tokens: request.maxTokens || 4096,
      temperature: request.temperature || 0.7,
      messages: request.messages.map(msg => ({
        role: msg.role as 'user' | 'assistant',
        content: msg.content,
      })),
      tools: request.tools?.map(tool => ({
        name: tool.name,
        description: tool.description,
        input_schema: tool.inputSchema,
      })),
    });

    for await (const chunk of stream) {
      if (chunk.type === 'content_block_delta') {
        yield {
          type: 'content',
          content: chunk.delta.text || '',
        };
      } else if (chunk.type === 'message_stop') {
        yield {
          type: 'done',
          usage: {
            promptTokens: stream.usage.input_tokens,
            completionTokens: stream.usage.output_tokens,
            totalTokens: stream.usage.input_tokens + stream.usage.output_tokens,
          },
        };
      }
    }
  }

  async getAvailableModels(): Promise<Model[]> {
    return [
      {
        id: 'claude-3-5-sonnet-20241022',
        name: 'Claude 3.5 Sonnet',
        contextWindow: 200000,
        maxOutput: 8192,
      },
      {
        id: 'claude-3-opus-20240229',
        name: 'Claude 3 Opus',
        contextWindow: 200000,
        maxOutput: 4096,
      },
    ];
  }

  private extractContent(response: any): string {
    const textContent = response.content.find((c: any) => c.type === 'text');
    return textContent?.text || '';
  }

  private extractToolCalls(response: any): ToolCall[] {
    const toolUseBlocks = response.content.filter((c: any) => c.type === 'tool_use');
    return toolUseBlocks.map((block: any) => ({
      id: block.id,
      name: block.name,
      input: block.input,
    }));
  }
}
```


### 3. Text Manipulation Service

```typescript
export class TextManipulationService {
  constructor(
    private aiService: AIService,
    private contextManager: ContextManager,
    private toolManager: ToolManager
  ) {}

  async appendText(request: AppendTextRequest): Promise<AppendTextResponse> {
    // 1. Prepare context
    const context = await this.contextManager.assembleContext({
      documentId: request.documentId,
      targetText: request.target,
      operation: 'append',
    });

    // 2. Build prompt
    const prompt = this.buildAppendPrompt(request, context);

    // 3. Call AI service
    const aiResponse = await this.aiService.generateCompletion({
      messages: [
        {
          role: 'system',
          content: SYSTEM_PROMPTS.APPEND_TEXT,
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      model: request.model || 'claude-3-5-sonnet-20241022',
      tools: [this.toolManager.getTool('append_text')],
    });

    // 4. Process tool calls
    const result = await this.processTool Calls(aiResponse.toolCalls);

    // 5. Return response
    return {
      success: true,
      result: result.content,
      originalText: request.target,
      modifiedText: result.modifiedText,
      explanation: aiResponse.content,
    };
  }

  async modifyText(request: ModifyTextRequest): Promise<ModifyTextResponse> {
    const context = await this.contextManager.assembleContext({
      documentId: request.documentId,
      targetText: request.target,
      operation: 'modify',
    });

    const prompt = this.buildModifyPrompt(request, context);

    const aiResponse = await this.aiService.generateCompletion({
      messages: [
        {
          role: 'system',
          content: SYSTEM_PROMPTS.MODIFY_TEXT,
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      model: request.model || 'claude-3-5-sonnet-20241022',
      tools: [this.toolManager.getTool('modify_text')],
    });

    const result = await this.processToolCalls(aiResponse.toolCalls);

    return {
      success: true,
      result: result.content,
      originalText: request.target,
      modifiedText: result.modifiedText,
      explanation: aiResponse.content,
      changes: this.calculateChanges(request.target, result.modifiedText),
    };
  }

  async removeText(request: RemoveTextRequest): Promise<RemoveTextResponse> {
    const context = await this.contextManager.assembleContext({
      documentId: request.documentId,
      targetText: request.target,
      operation: 'remove',
    });

    const prompt = this.buildRemovePrompt(request, context);

    const aiResponse = await this.aiService.generateCompletion({
      messages: [
        {
          role: 'system',
          content: SYSTEM_PROMPTS.REMOVE_TEXT,
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      model: request.model || 'claude-3-5-sonnet-20241022',
      tools: [this.toolManager.getTool('remove_text')],
    });

    const result = await this.processToolCalls(aiResponse.toolCalls);

    return {
      success: true,
      result: result.content,
      originalText: request.target,
      removedParts: result.removedParts,
      explanation: aiResponse.content,
    };
  }

  private buildAppendPrompt(request: AppendTextRequest, context: Context): string {
    return `
Task: Append text to the following passage

Target Text:
${request.target}

Instructions:
${request.instruction}

Position: ${request.position || 'after'}

Context:
${this.formatContext(context)}

Please append appropriate text based on the instructions while maintaining coherence and style.
    `.trim();
  }

  private buildModifyPrompt(request: ModifyTextRequest, context: Context): string {
    return `
Task: Modify the following text

Target Text:
${request.target}

Modification Instructions:
${request.instruction}

Context:
${this.formatContext(context)}

Please modify the text according to the instructions while maintaining coherence and the original intent.
    `.trim();
  }

  private buildRemovePrompt(request: RemoveTextRequest, context: Context): string {
    return `
Task: Remove text from the following passage

Target Text:
${request.target}

Removal Criteria:
${request.criteria}

Context:
${this.formatContext(context)}

Please identify and remove the text that matches the criteria while maintaining coherence.
    `.trim();
  }

  private formatContext(context: Context): string {
    return context.items
      .map(item => `[${item.type}]: ${item.content}`)
      .join('\n\n');
  }

  private async processToolCalls(toolCalls?: ToolCall[]): Promise<any> {
    if (!toolCalls || toolCalls.length === 0) {
      throw new Error('No tool calls received from AI');
    }

    const results = await Promise.all(
      toolCalls.map(call => this.toolManager.executeTool(call.name, call.input))
    );

    return results[0]; // Return first result for now
  }

  private calculateChanges(original: string, modified: string): TextChange[] {
    // Implement diff algorithm to calculate changes
    // This is a simplified version
    return [
      {
        type: 'modification',
        original,
        modified,
        position: 0,
      },
    ];
  }
}
```

---

## API Design

### RESTful API Endpoints

#### Authentication Endpoints

```typescript
// POST /api/auth/register
interface RegisterRequest {
  email: string;
  password: string;
  name: string;
}

interface RegisterResponse {
  user: {
    id: string;
    email: string;
    name: string;
  };
  token: string;
}

// POST /api/auth/login
interface LoginRequest {
  email: string;
  password: string;
}

interface LoginResponse {
  user: {
    id: string;
    email: string;
    name: string;
  };
  token: string;
}

// POST /api/auth/refresh
interface RefreshTokenRequest {
  refreshToken: string;
}

interface RefreshTokenResponse {
  token: string;
  refreshToken: string;
}
```

#### Chat Endpoints

```typescript
// POST /api/chat/messages
interface SendMessageRequest {
  text: string;
  context?: ContextItem[];
  documentId?: string;
  conversationId?: string;
  model?: string;
}

interface SendMessageResponse {
  message: {
    id: string;
    text: string;
    role: 'user' | 'assistant';
    timestamp: number;
  };
  conversationId: string;
}

// GET /api/chat/conversations
interface GetConversationsResponse {
  conversations: Array<{
    id: string;
    title: string;
    lastMessage: string;
    timestamp: number;
    messageCount: number;
  }>;
  total: number;
}

// GET /api/chat/conversations/:id/messages
interface GetMessagesResponse {
  messages: Message[];
  total: number;
}

// POST /api/chat/stream
// Server-Sent Events endpoint for streaming responses
```

#### Document Endpoints

```typescript
// GET /api/documents
interface GetDocumentsResponse {
  documents: Array<{
    id: string;
    name: string;
    content: string;
    createdAt: number;
    updatedAt: number;
    wordCount: number;
  }>;
  total: number;
}

// GET /api/documents/:id
interface GetDocumentResponse {
  document: {
    id: string;
    name: string;
    content: string;
    metadata: {
      wordCount: number;
      characterCount: number;
      createdAt: number;
      updatedAt: number;
      author: string;
    };
  };
}

// POST /api/documents
interface CreateDocumentRequest {
  name: string;
  content: string;
}

interface CreateDocumentResponse {
  document: {
    id: string;
    name: string;
    content: string;
    createdAt: number;
  };
}

// PUT /api/documents/:id
interface UpdateDocumentRequest {
  name?: string;
  content?: string;
}

interface UpdateDocumentResponse {
  document: {
    id: string;
    name: string;
    content: string;
    updatedAt: number;
  };
}

// DELETE /api/documents/:id
interface DeleteDocumentResponse {
  success: boolean;
}
```

#### Text Manipulation Endpoints

```typescript
// POST /api/text/append
interface AppendTextRequest {
  documentId: string;
  target: string;
  instruction: string;
  position?: 'before' | 'after';
  context?: ContextItem[];
  model?: string;
}

interface AppendTextResponse {
  success: boolean;
  result: {
    originalText: string;
    appendedText: string;
    finalText: string;
    position: number;
  };
  explanation: string;
  usage: TokenUsage;
}

// POST /api/text/modify
interface ModifyTextRequest {
  documentId: string;
  target: string;
  instruction: string;
  context?: ContextItem[];
  model?: string;
}

interface ModifyTextResponse {
  success: boolean;
  result: {
    originalText: string;
    modifiedText: string;
    changes: TextChange[];
  };
  explanation: string;
  usage: TokenUsage;
}

// POST /api/text/remove
interface RemoveTextRequest {
  documentId: string;
  target: string;
  criteria: string;
  context?: ContextItem[];
  model?: string;
}

interface RemoveTextResponse {
  success: boolean;
  result: {
    originalText: string;
    remainingText: string;
    removedParts: string[];
  };
  explanation: string;
  usage: TokenUsage;
}
```

### API Implementation Example (Express.js)

```typescript
import express, { Router } from 'express';
import { authenticateToken } from '../middleware/auth.middleware';
import { validateRequest } from '../middleware/validation.middleware';
import { TextManipulationService } from '../services/text/TextManipulationService';

export function createTextRoutes(textService: TextManipulationService): Router {
  const router = express.Router();

  // POST /api/text/append
  router.post(
    '/append',
    authenticateToken,
    validateRequest(appendTextSchema),
    async (req, res, next) => {
      try {
        const result = await textService.appendText({
          ...req.body,
          userId: req.user.id,
        });
        res.json(result);
      } catch (error) {
        next(error);
      }
    }
  );

  // POST /api/text/modify
  router.post(
    '/modify',
    authenticateToken,
    validateRequest(modifyTextSchema),
    async (req, res, next) => {
      try {
        const result = await textService.modifyText({
          ...req.body,
          userId: req.user.id,
        });
        res.json(result);
      } catch (error) {
        next(error);
      }
    }
  );

  // POST /api/text/remove
  router.post(
    '/remove',
    authenticateToken,
    validateRequest(removeTextSchema),
    async (req, res, next) => {
      try {
        const result = await textService.removeText({
          ...req.body,
          userId: req.user.id,
        });
        res.json(result);
      } catch (error) {
        next(error);
      }
    }
  );

  return router;
}
```


---

## Tool System

The tool system provides a structured way for the AI to perform specific operations on text.

### Tool Manager

```typescript
export interface Tool {
  name: string;
  description: string;
  inputSchema: {
    type: 'object';
    properties: Record<string, any>;
    required: string[];
  };
  execute: (input: any) => Promise<any>;
}

export class ToolManager {
  private tools: Map<string, Tool> = new Map();

  constructor() {
    this.registerDefaultTools();
  }

  private registerDefaultTools(): void {
    this.registerTool(new AppendTextTool());
    this.registerTool(new ModifyTextTool());
    this.registerTool(new RemoveTextTool());
    this.registerTool(new SearchTextTool());
    this.registerTool(new AnalyzeTextTool());
  }

  registerTool(tool: Tool): void {
    this.tools.set(tool.name, tool);
  }

  getTool(name: string): Tool | undefined {
    return this.tools.get(name);
  }

  getAllTools(): Tool[] {
    return Array.from(this.tools.values());
  }

  async executeTool(name: string, input: any): Promise<any> {
    const tool = this.tools.get(name);
    if (!tool) {
      throw new Error(`Tool ${name} not found`);
    }

    return await tool.execute(input);
  }

  getToolDefinitions(): any[] {
    return Array.from(this.tools.values()).map(tool => ({
      name: tool.name,
      description: tool.description,
      input_schema: tool.inputSchema,
    }));
  }
}
```

### Example Tool Implementation

```typescript
export class AppendTextTool implements Tool {
  name = 'append_text';
  description = 'Append text to a specific location in a document';
  inputSchema = {
    type: 'object' as const,
    properties: {
      target: {
        type: 'string',
        description: 'The target text or location where content should be appended',
      },
      content: {
        type: 'string',
        description: 'The content to append',
      },
      position: {
        type: 'string',
        enum: ['before', 'after'],
        description: 'Whether to append before or after the target',
      },
    },
    required: ['target', 'content'],
  };

  async execute(input: {
    target: string;
    content: string;
    position?: 'before' | 'after';
  }): Promise<any> {
    const position = input.position || 'after';

    // Implement the logic to append text
    const result = position === 'before'
      ? `${input.content}\n${input.target}`
      : `${input.target}\n${input.content}`;

    return {
      success: true,
      modifiedText: result,
      content: input.content,
      position,
    };
  }
}

export class ModifyTextTool implements Tool {
  name = 'modify_text';
  description = 'Modify existing text based on instructions';
  inputSchema = {
    type: 'object' as const,
    properties: {
      target: {
        type: 'string',
        description: 'The text to be modified',
      },
      modification: {
        type: 'string',
        description: 'The modified version of the text',
      },
      reason: {
        type: 'string',
        description: 'Explanation of what was changed and why',
      },
    },
    required: ['target', 'modification', 'reason'],
  };

  async execute(input: {
    target: string;
    modification: string;
    reason: string;
  }): Promise<any> {
    return {
      success: true,
      originalText: input.target,
      modifiedText: input.modification,
      reason: input.reason,
      changes: this.calculateDiff(input.target, input.modification),
    };
  }

  private calculateDiff(original: string, modified: string): any[] {
    // Implement diff calculation (can use libraries like diff or diff-match-patch)
    return [];
  }
}

export class SearchTextTool implements Tool {
  name = 'search_text';
  description = 'Search for specific text or patterns in a document';
  inputSchema = {
    type: 'object' as const,
    properties: {
      query: {
        type: 'string',
        description: 'The search query',
      },
      documentId: {
        type: 'string',
        description: 'The document ID to search in',
      },
      caseSensitive: {
        type: 'boolean',
        description: 'Whether the search should be case-sensitive',
      },
    },
    required: ['query', 'documentId'],
  };

  async execute(input: {
    query: string;
    documentId: string;
    caseSensitive?: boolean;
  }): Promise<any> {
    // Implement search logic
    // This would typically interact with a document repository

    return {
      success: true,
      matches: [],
      totalMatches: 0,
    };
  }
}
```

---

## Context Management

### Context Manager Implementation

```typescript
export interface ContextItem {
  id: string;
  type: 'document' | 'conversation' | 'selection' | 'reference';
  content: string;
  metadata: {
    source: string;
    timestamp: number;
    relevance: number;
    wordCount: number;
  };
}

export interface Context {
  items: ContextItem[];
  totalTokens: number;
  maxTokens: number;
}

export class ContextManager {
  private maxContextTokens: number = 100000; // Configurable based on model

  constructor(
    private documentRepository: DocumentRepository,
    private chatRepository: ChatRepository
  ) {}

  async assembleContext(options: {
    documentId?: string;
    conversationId?: string;
    targetText?: string;
    operation: string;
    additionalContext?: ContextItem[];
  }): Promise<Context> {
    const items: ContextItem[] = [];

    // Add document context
    if (options.documentId) {
      const documentContext = await this.getDocumentContext(options.documentId);
      items.push(...documentContext);
    }

    // Add conversation context
    if (options.conversationId) {
      const conversationContext = await this.getConversationContext(options.conversationId);
      items.push(...conversationContext);
    }

    // Add target text context
    if (options.targetText) {
      items.push({
        id: 'target',
        type: 'selection',
        content: options.targetText,
        metadata: {
          source: 'user_selection',
          timestamp: Date.now(),
          relevance: 1.0,
          wordCount: options.targetText.split(/\s+/).length,
        },
      });
    }

    // Add additional context
    if (options.additionalContext) {
      items.push(...options.additionalContext);
    }

    // Prune context if necessary
    const prunedItems = this.pruneContext(items);

    return {
      items: prunedItems,
      totalTokens: this.calculateTotalTokens(prunedItems),
      maxTokens: this.maxContextTokens,
    };
  }

  private async getDocumentContext(documentId: string): Promise<ContextItem[]> {
    const document = await this.documentRepository.findById(documentId);
    if (!document) {
      return [];
    }

    return [
      {
        id: documentId,
        type: 'document',
        content: document.content,
        metadata: {
          source: document.name,
          timestamp: document.updatedAt,
          relevance: 0.8,
          wordCount: document.content.split(/\s+/).length,
        },
      },
    ];
  }

  private async getConversationContext(conversationId: string): Promise<ContextItem[]> {
    const messages = await this.chatRepository.getConversationMessages(conversationId, {
      limit: 10, // Get last 10 messages
      order: 'desc',
    });

    return messages.map((msg, index) => ({
      id: msg.id,
      type: 'conversation' as const,
      content: `${msg.role}: ${msg.content}`,
      metadata: {
        source: 'conversation',
        timestamp: msg.timestamp,
        relevance: 0.9 - (index * 0.05), // More recent messages have higher relevance
        wordCount: msg.content.split(/\s+/).length,
      },
    }));
  }

  private pruneContext(items: ContextItem[]): ContextItem[] {
    // Sort by relevance (descending)
    const sorted = [...items].sort((a, b) => b.metadata.relevance - a.metadata.relevance);

    const pruned: ContextItem[] = [];
    let totalTokens = 0;

    for (const item of sorted) {
      const itemTokens = this.estimateTokens(item.content);
      if (totalTokens + itemTokens <= this.maxContextTokens) {
        pruned.push(item);
        totalTokens += itemTokens;
      } else {
        break;
      }
    }

    // Sort back by timestamp to maintain chronological order
    return pruned.sort((a, b) => a.metadata.timestamp - b.metadata.timestamp);
  }

  private estimateTokens(text: string): number {
    // Rough estimation: ~4 characters per token
    return Math.ceil(text.length / 4);
  }

  private calculateTotalTokens(items: ContextItem[]): number {
    return items.reduce((sum, item) => sum + this.estimateTokens(item.content), 0);
  }

  setMaxContextTokens(maxTokens: number): void {
    this.maxContextTokens = maxTokens;
  }
}
```

### Context Assembler for Prompts

```typescript
export class ContextAssembler {
  formatContextForPrompt(context: Context): string {
    if (context.items.length === 0) {
      return '';
    }

    const sections: string[] = [];

    // Group by type
    const grouped = this.groupByType(context.items);

    // Format documents
    if (grouped.document.length > 0) {
      sections.push(this.formatDocumentSection(grouped.document));
    }

    // Format conversation history
    if (grouped.conversation.length > 0) {
      sections.push(this.formatConversationSection(grouped.conversation));
    }

    // Format selections
    if (grouped.selection.length > 0) {
      sections.push(this.formatSelectionSection(grouped.selection));
    }

    // Format references
    if (grouped.reference.length > 0) {
      sections.push(this.formatReferenceSection(grouped.reference));
    }

    return sections.join('\n\n---\n\n');
  }

  private groupByType(items: ContextItem[]): Record<string, ContextItem[]> {
    return items.reduce((acc, item) => {
      if (!acc[item.type]) {
        acc[item.type] = [];
      }
      acc[item.type].push(item);
      return acc;
    }, {} as Record<string, ContextItem[]>);
  }

  private formatDocumentSection(items: ContextItem[]): string {
    return `
<document_context>
${items.map(item => `
<document source="${item.metadata.source}">
${item.content}
</document>
`).join('\n')}
</document_context>
    `.trim();
  }

  private formatConversationSection(items: ContextItem[]): string {
    return `
<conversation_history>
${items.map(item => item.content).join('\n')}
</conversation_history>
    `.trim();
  }

  private formatSelectionSection(items: ContextItem[]): string {
    return `
<selected_text>
${items.map(item => item.content).join('\n\n')}
</selected_text>
    `.trim();
  }

  private formatReferenceSection(items: ContextItem[]): string {
    return `
<references>
${items.map(item => `
<reference source="${item.metadata.source}">
${item.content}
</reference>
`).join('\n')}
</references>
    `.trim();
  }
}
```

---

## Database Design

### PostgreSQL Schema

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  last_login TIMESTAMP,
  is_active BOOLEAN DEFAULT TRUE,
  settings JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Documents table
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(500) NOT NULL,
  content TEXT NOT NULL,
  content_type VARCHAR(50) DEFAULT 'text/plain',
  word_count INTEGER DEFAULT 0,
  character_count INTEGER DEFAULT 0,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_documents_user_id ON documents(user_id);
CREATE INDEX idx_documents_created_at ON documents(created_at);
CREATE INDEX idx_documents_updated_at ON documents(updated_at);

-- Full-text search index on content
CREATE INDEX idx_documents_content_fts ON documents USING gin(to_tsvector('english', content));

-- Document versions table (for version history)
CREATE TABLE document_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  version_number INTEGER NOT NULL,
  content TEXT NOT NULL,
  description VARCHAR(500),
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  created_by UUID NOT NULL REFERENCES users(id),
  metadata JSONB DEFAULT '{}'::jsonb,
  UNIQUE(document_id, version_number)
);

CREATE INDEX idx_document_versions_document_id ON document_versions(document_id);
CREATE INDEX idx_document_versions_created_at ON document_versions(created_at);

-- Conversations table
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(500) NOT NULL,
  document_id UUID REFERENCES documents(id) ON DELETE SET NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW(),
  message_count INTEGER DEFAULT 0,
  metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_conversations_user_id ON conversations(user_id);
CREATE INDEX idx_conversations_document_id ON conversations(document_id);
CREATE INDEX idx_conversations_updated_at ON conversations(updated_at);

-- Messages table
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
  role VARCHAR(20) NOT NULL CHECK (role IN ('user', 'assistant', 'system')),
  content TEXT NOT NULL,
  timestamp TIMESTAMP NOT NULL DEFAULT NOW(),
  model VARCHAR(100),
  tokens_used INTEGER,
  metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX idx_messages_timestamp ON messages(timestamp);
CREATE INDEX idx_messages_role ON messages(role);

-- Text operations table (audit log for text manipulations)
CREATE TABLE text_operations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  document_id UUID REFERENCES documents(id) ON DELETE SET NULL,
  conversation_id UUID REFERENCES conversations(id) ON DELETE SET NULL,
  operation_type VARCHAR(50) NOT NULL CHECK (operation_type IN ('append', 'modify', 'remove', 'search')),
  target_text TEXT,
  result_text TEXT,
  instruction TEXT,
  model VARCHAR(100),
  tokens_used INTEGER,
  cost_usd DECIMAL(10, 6),
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_text_operations_user_id ON text_operations(user_id);
CREATE INDEX idx_text_operations_document_id ON text_operations(document_id);
CREATE INDEX idx_text_operations_created_at ON text_operations(created_at);
CREATE INDEX idx_text_operations_operation_type ON text_operations(operation_type);

-- API keys table (for external API management)
CREATE TABLE api_keys (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  provider VARCHAR(50) NOT NULL CHECK (provider IN ('openai', 'anthropic', 'google', 'custom')),
  key_name VARCHAR(255) NOT NULL,
  encrypted_key TEXT NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  last_used_at TIMESTAMP,
  metadata JSONB DEFAULT '{}'::jsonb
);

CREATE INDEX idx_api_keys_user_id ON api_keys(user_id);
CREATE INDEX idx_api_keys_provider ON api_keys(provider);

-- User preferences table
CREATE TABLE user_preferences (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  default_model VARCHAR(100) DEFAULT 'claude-3-5-sonnet-20241022',
  theme VARCHAR(20) DEFAULT 'light',
  language VARCHAR(10) DEFAULT 'en',
  auto_save BOOLEAN DEFAULT TRUE,
  show_token_usage BOOLEAN DEFAULT TRUE,
  preferences JSONB DEFAULT '{}'::jsonb,
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Refresh tokens table (for JWT authentication)
CREATE TABLE refresh_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token VARCHAR(500) UNIQUE NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  revoked BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_refresh_tokens_user_id ON refresh_tokens(user_id);
CREATE INDEX idx_refresh_tokens_token ON refresh_tokens(token);
CREATE INDEX idx_refresh_tokens_expires_at ON refresh_tokens(expires_at);

-- Function to update updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Triggers for automatic updated_at
CREATE TRIGGER update_users_updated_at
  BEFORE UPDATE ON users
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_documents_updated_at
  BEFORE UPDATE ON documents
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_conversations_updated_at
  BEFORE UPDATE ON conversations
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```


### Repository Pattern Implementation

```typescript
import { Pool } from 'pg';

export class DocumentRepository {
  constructor(private db: Pool) {}

  async findById(id: string): Promise<Document | null> {
    const result = await this.db.query(
      'SELECT * FROM documents WHERE id = $1',
      [id]
    );
    return result.rows[0] || null;
  }

  async findByUserId(userId: string, options?: {
    limit?: number;
    offset?: number;
    orderBy?: string;
  }): Promise<Document[]> {
    const limit = options?.limit || 50;
    const offset = options?.offset || 0;
    const orderBy = options?.orderBy || 'updated_at DESC';

    const result = await this.db.query(
      `SELECT * FROM documents 
       WHERE user_id = $1 
       ORDER BY ${orderBy}
       LIMIT $2 OFFSET $3`,
      [userId, limit, offset]
    );
    return result.rows;
  }

  async create(document: CreateDocumentDTO): Promise<Document> {
    const result = await this.db.query(
      `INSERT INTO documents (user_id, name, content, word_count, character_count)
       VALUES ($1, $2, $3, $4, $5)
       RETURNING *`,
      [
        document.userId,
        document.name,
        document.content,
        this.countWords(document.content),
        document.content.length,
      ]
    );
    return result.rows[0];
  }

  async update(id: string, updates: Partial<Document>): Promise<Document> {
    const fields: string[] = [];
    const values: any[] = [];
    let paramCount = 1;

    if (updates.name !== undefined) {
      fields.push(`name = $${paramCount++}`);
      values.push(updates.name);
    }

    if (updates.content !== undefined) {
      fields.push(`content = $${paramCount++}`);
      values.push(updates.content);
      fields.push(`word_count = $${paramCount++}`);
      values.push(this.countWords(updates.content));
      fields.push(`character_count = $${paramCount++}`);
      values.push(updates.content.length);
    }

    values.push(id);

    const result = await this.db.query(
      `UPDATE documents 
       SET ${fields.join(', ')}, updated_at = NOW()
       WHERE id = $${paramCount}
       RETURNING *`,
      values
    );

    return result.rows[0];
  }

  async delete(id: string): Promise<boolean> {
    const result = await this.db.query(
      'DELETE FROM documents WHERE id = $1',
      [id]
    );
    return result.rowCount > 0;
  }

  async search(userId: string, query: string): Promise<Document[]> {
    const result = await this.db.query(
      `SELECT * FROM documents 
       WHERE user_id = $1 
       AND to_tsvector('english', content) @@ plainto_tsquery('english', $2)
       ORDER BY updated_at DESC
       LIMIT 50`,
      [userId, query]
    );
    return result.rows;
  }

  private countWords(text: string): number {
    return text.split(/\s+/).filter(word => word.length > 0).length;
  }
}
```

---

## Authentication & Authorization

### JWT Authentication Implementation

```typescript
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';
import { Request, Response, NextFunction } from 'express';

const JWT_SECRET = process.env.JWT_SECRET!;
const JWT_EXPIRES_IN = '1h';
const REFRESH_TOKEN_EXPIRES_IN = '7d';

export interface TokenPayload {
  userId: string;
  email: string;
}

export class AuthService {
  constructor(
    private userRepository: UserRepository,
    private refreshTokenRepository: RefreshTokenRepository
  ) {}

  async register(data: {
    email: string;
    password: string;
    name: string;
  }): Promise<{ user: User; token: string; refreshToken: string }> {
    // Check if user already exists
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) {
      throw new Error('User already exists');
    }

    // Hash password
    const passwordHash = await bcrypt.hash(data.password, 10);

    // Create user
    const user = await this.userRepository.create({
      email: data.email,
      passwordHash,
      name: data.name,
    });

    // Generate tokens
    const token = this.generateToken(user);
    const refreshToken = await this.generateRefreshToken(user.id);

    return { user, token, refreshToken };
  }

  async login(data: {
    email: string;
    password: string;
  }): Promise<{ user: User; token: string; refreshToken: string }> {
    // Find user
    const user = await this.userRepository.findByEmail(data.email);
    if (!user) {
      throw new Error('Invalid credentials');
    }

    // Verify password
    const valid = await bcrypt.compare(data.password, user.passwordHash);
    if (!valid) {
      throw new Error('Invalid credentials');
    }

    // Update last login
    await this.userRepository.updateLastLogin(user.id);

    // Generate tokens
    const token = this.generateToken(user);
    const refreshToken = await this.generateRefreshToken(user.id);

    return { user, token, refreshToken };
  }

  async refreshTokens(refreshToken: string): Promise<{
    token: string;
    refreshToken: string;
  }> {
    // Verify refresh token
    const tokenRecord = await this.refreshTokenRepository.findByToken(refreshToken);
    if (!tokenRecord || tokenRecord.revoked) {
      throw new Error('Invalid refresh token');
    }

    if (new Date(tokenRecord.expiresAt) < new Date()) {
      throw new Error('Refresh token expired');
    }

    // Get user
    const user = await this.userRepository.findById(tokenRecord.userId);
    if (!user) {
      throw new Error('User not found');
    }

    // Revoke old refresh token
    await this.refreshTokenRepository.revoke(tokenRecord.id);

    // Generate new tokens
    const token = this.generateToken(user);
    const newRefreshToken = await this.generateRefreshToken(user.id);

    return { token, refreshToken: newRefreshToken };
  }

  private generateToken(user: User): string {
    const payload: TokenPayload = {
      userId: user.id,
      email: user.email,
    };

    return jwt.sign(payload, JWT_SECRET, {
      expiresIn: JWT_EXPIRES_IN,
    });
  }

  private async generateRefreshToken(userId: string): Promise<string> {
    const token = jwt.sign({ userId }, JWT_SECRET, {
      expiresIn: REFRESH_TOKEN_EXPIRES_IN,
    });

    const expiresAt = new Date();
    expiresAt.setDate(expiresAt.getDate() + 7);

    await this.refreshTokenRepository.create({
      userId,
      token,
      expiresAt,
    });

    return token;
  }

  verifyToken(token: string): TokenPayload {
    try {
      return jwt.verify(token, JWT_SECRET) as TokenPayload;
    } catch (error) {
      throw new Error('Invalid token');
    }
  }
}

// Middleware for protecting routes
export function authenticateToken(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const payload = jwt.verify(token, JWT_SECRET) as TokenPayload;
    req.user = payload;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid token' });
  }
}
```

---

## Real-time Communication

### WebSocket Server Setup

```typescript
import { Server as SocketIOServer } from 'socket.io';
import { Server as HTTPServer } from 'http';
import { verifyToken } from './auth';

export class WebSocketService {
  private io: SocketIOServer;
  private connectedClients: Map<string, Set<string>> = new Map();

  constructor(httpServer: HTTPServer) {
    this.io = new SocketIOServer(httpServer, {
      cors: {
        origin: process.env.FRONTEND_URL,
        credentials: true,
      },
    });

    this.setupMiddleware();
    this.setupEventHandlers();
  }

  private setupMiddleware() {
    // Authentication middleware
    this.io.use((socket, next) => {
      const token = socket.handshake.auth.token;
      if (!token) {
        return next(new Error('Authentication error'));
      }

      try {
        const payload = verifyToken(token);
        socket.data.userId = payload.userId;
        next();
      } catch (error) {
        next(new Error('Authentication error'));
      }
    });
  }

  private setupEventHandlers() {
    this.io.on('connection', (socket) => {
      const userId = socket.data.userId;
      console.log(`User ${userId} connected`);

      // Track connection
      if (!this.connectedClients.has(userId)) {
        this.connectedClients.set(userId, new Set());
      }
      this.connectedClients.get(userId)!.add(socket.id);

      // Join user's personal room
      socket.join(`user:${userId}`);

      // Handle document room join
      socket.on('join_document', (documentId: string) => {
        socket.join(`document:${documentId}`);
        this.broadcastToDocument(documentId, 'user_joined', {
          userId,
          socketId: socket.id,
        });
      });

      // Handle document changes
      socket.on('document_change', (data: {
        documentId: string;
        change: any;
      }) => {
        socket.to(`document:${data.documentId}`).emit('remote_change', {
          userId,
          change: data.change,
        });
      });

      // Handle AI streaming
      socket.on('start_ai_stream', async (data: {
        conversationId: string;
        message: string;
      }) => {
        try {
          // Start AI streaming
          for await (const chunk of this.streamAIResponse(data.message)) {
            socket.emit('ai_stream_chunk', {
              conversationId: data.conversationId,
              chunk,
            });
          }
          socket.emit('ai_stream_complete', {
            conversationId: data.conversationId,
          });
        } catch (error) {
          socket.emit('ai_stream_error', {
            conversationId: data.conversationId,
            error: error.message,
          });
        }
      });

      // Handle disconnection
      socket.on('disconnect', () => {
        console.log(`User ${userId} disconnected`);
        const userSockets = this.connectedClients.get(userId);
        if (userSockets) {
          userSockets.delete(socket.id);
          if (userSockets.size === 0) {
            this.connectedClients.delete(userId);
          }
        }
      });
    });
  }

  private async *streamAIResponse(message: string): AsyncIterable<string> {
    // Implement AI streaming logic
    // This is a placeholder
    yield 'Streaming response...';
  }

  broadcastToUser(userId: string, event: string, data: any) {
    this.io.to(`user:${userId}`).emit(event, data);
  }

  broadcastToDocument(documentId: string, event: string, data: any) {
    this.io.to(`document:${documentId}`).emit(event, data);
  }
}
```

---

## System Prompts

### Prompt Templates

```typescript
export const SYSTEM_PROMPTS = {
  APPEND_TEXT: `
You are an expert text editor helping users enhance their documents. Your task is to append text to existing content.

Guidelines:
- Maintain the style and tone of the original text
- Ensure logical flow and coherence
- Add value and relevant information
- Keep the appended content concise and focused
- Follow the user's specific instructions

When appending text, use the append_text tool with:
- target: The exact text where content should be added
- content: The new content to append
- position: Whether to add before or after the target
  `.trim(),

  MODIFY_TEXT: `
You are an expert text editor helping users improve their documents. Your task is to modify existing text based on instructions.

Guidelines:
- Preserve the core meaning unless instructed otherwise
- Improve clarity, grammar, and readability
- Maintain the original style and tone unless asked to change
- Make targeted modifications based on user instructions
- Explain what you changed and why

When modifying text, use the modify_text tool with:
- target: The original text to modify
- modification: The improved version
- reason: Explanation of changes made
  `.trim(),

  REMOVE_TEXT: `
You are an expert text editor helping users refine their documents. Your task is to remove unnecessary or unwanted content.

Guidelines:
- Carefully identify content that matches removal criteria
- Maintain document coherence after removal
- Preserve important context and meaning
- Be conservative - when in doubt, ask for clarification
- Explain what was removed and why

When removing text, use the remove_text tool with:
- target: The text containing content to remove
- removedParts: Array of removed text segments
- remainingText: The final text after removal
  `.trim(),

  GENERAL_ASSISTANT: `
You are an AI text manipulation assistant designed to help scholars, students, and professionals improve their written content.

Your capabilities include:
- Appending text: Add relevant content before or after existing text
- Modifying text: Improve, rephrase, or transform existing content
- Removing text: Delete unnecessary or unwanted content
- Searching: Find specific content in documents
- Analyzing: Provide insights about text quality and structure

Guidelines:
- Always ask for clarification when instructions are ambiguous
- Explain your reasoning and changes clearly
- Respect the user's original intent and style
- Provide suggestions when you see opportunities for improvement
- Be helpful, professional, and precise

Use the available tools to perform text operations accurately and efficiently.
  `.trim(),
};
```

---

## Error Handling

### Global Error Handler

```typescript
import { Request, Response, NextFunction } from 'express';

export class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
    Error.captureStackTrace(this, this.constructor);
  }
}

export function errorHandler(
  err: Error | AppError,
  req: Request,
  res: Response,
  next: NextFunction
) {
  // Default error values
  let statusCode = 500;
  let message = 'Internal Server Error';
  let isOperational = false;

  // Check if it's an operational error
  if (err instanceof AppError) {
    statusCode = err.statusCode;
    message = err.message;
    isOperational = err.isOperational;
  }

  // Log error
  console.error('Error:', {
    statusCode,
    message,
    stack: err.stack,
    path: req.path,
    method: req.method,
    ip: req.ip,
  });

  // Send error response
  res.status(statusCode).json({
    error: {
      message,
      ...(process.env.NODE_ENV === 'development' && {
        stack: err.stack,
        details: err,
      }),
    },
  });
}

// Async error wrapper
export function asyncHandler(fn: Function) {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}
```

---

## Deployment Architecture

### Docker Configuration

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY pnpm-lock.yaml ./

# Install pnpm and dependencies
RUN npm install -g pnpm
RUN pnpm install --frozen-lockfile

# Copy source code
COPY . .

# Build application
RUN pnpm run build

# Production stage
FROM node:20-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
COPY pnpm-lock.yaml ./

# Install production dependencies
RUN npm install -g pnpm
RUN pnpm install --frozen-lockfile --prod

# Copy built application
COPY --from=builder /app/dist ./dist

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "dist/app.js"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  backend:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@postgres:5432/textai
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  postgres:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=textai
    ports:
      - "5432:5432"
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - backend
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

### Environment Variables

```bash
# .env.example
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/textai
DATABASE_POOL_MIN=2
DATABASE_POOL_MAX=10

# Redis
REDIS_URL=redis://localhost:6379
REDIS_TTL=3600

# JWT Authentication
JWT_SECRET=your-super-secret-jwt-key-change-this
JWT_EXPIRES_IN=1h
REFRESH_TOKEN_EXPIRES_IN=7d

# AI Providers
ANTHROPIC_API_KEY=your-anthropic-api-key
OPENAI_API_KEY=your-openai-api-key
GOOGLE_AI_API_KEY=your-google-ai-api-key

# Application
FRONTEND_URL=http://localhost:3000
MAX_REQUEST_SIZE=10mb
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# File Storage
FILE_STORAGE_TYPE=local
FILE_STORAGE_PATH=./storage
# AWS_S3_BUCKET=your-bucket
# AWS_ACCESS_KEY_ID=your-access-key
# AWS_SECRET_ACCESS_KEY=your-secret-key
# AWS_REGION=us-east-1

# Monitoring
LOG_LEVEL=info
ENABLE_METRICS=true

# Feature Flags
ENABLE_COLLABORATION=true
ENABLE_VERSION_HISTORY=true
MAX_DOCUMENT_SIZE=1000000
```

---

## Production Deployment Checklist

### Pre-Deployment

- [ ] Set up production database (PostgreSQL)
- [ ] Configure Redis for caching
- [ ] Set up file storage (S3 or equivalent)
- [ ] Configure environment variables
- [ ] Set up SSL certificates
- [ ] Configure CORS policies
- [ ] Set up monitoring (Prometheus/Grafana)
- [ ] Configure logging aggregation
- [ ] Set up backup strategy
- [ ] Configure rate limiting
- [ ] Set up CI/CD pipeline

### Security

- [ ] Use strong JWT secrets
- [ ] Encrypt API keys in database
- [ ] Enable HTTPS only
- [ ] Implement rate limiting
- [ ] Add request validation
- [ ] Sanitize user inputs
- [ ] Implement CSRF protection
- [ ] Set secure headers
- [ ] Enable database connection pooling
- [ ] Implement proper error handling

### Performance

- [ ] Enable Redis caching
- [ ] Optimize database queries
- [ ] Add database indexes
- [ ] Implement connection pooling
- [ ] Enable gzip compression
- [ ] Optimize API response sizes
- [ ] Implement pagination
- [ ] Add request timeouts
- [ ] Monitor memory usage
- [ ] Set up auto-scaling

### Monitoring

- [ ] Set up application monitoring
- [ ] Configure error tracking (Sentry)
- [ ] Enable performance monitoring
- [ ] Set up log aggregation (ELK Stack)
- [ ] Configure alerts for errors
- [ ] Monitor API response times
- [ ] Track token usage and costs
- [ ] Monitor database performance
- [ ] Set up uptime monitoring
- [ ] Create operational dashboards

---

## Testing Strategy

### Unit Tests Example

```typescript
import { TextManipulationService } from './TextManipulationService';
import { AIService } from '../ai/AIService';
import { ContextManager } from '../../core/context/ContextManager';
import { ToolManager } from '../../core/tools/ToolManager';

describe('TextManipulationService', () => {
  let service: TextManipulationService;
  let mockAIService: jest.Mocked<AIService>;
  let mockContextManager: jest.Mocked<ContextManager>;
  let mockToolManager: jest.Mocked<ToolManager>;

  beforeEach(() => {
    mockAIService = {
      generateCompletion: jest.fn(),
    } as any;

    mockContextManager = {
      assembleContext: jest.fn(),
    } as any;

    mockToolManager = {
      getTool: jest.fn(),
      executeTool: jest.fn(),
    } as any;

    service = new TextManipulationService(
      mockAIService,
      mockContextManager,
      mockToolManager
    );
  });

  describe('appendText', () => {
    it('should append text successfully', async () => {
      const request = {
        documentId: 'doc-123',
        target: 'Original text',
        instruction: 'Add a conclusion',
        position: 'after' as const,
      };

      mockContextManager.assembleContext.mockResolvedValue({
        items: [],
        totalTokens: 0,
        maxTokens: 100000,
      });

      mockAIService.generateCompletion.mockResolvedValue({
        content: 'AI response',
        model: 'claude-3-5-sonnet',
        usage: { promptTokens: 100, completionTokens: 50, totalTokens: 150 },
        toolCalls: [{
          id: '1',
          name: 'append_text',
          input: {
            target: 'Original text',
            content: 'Conclusion text',
            position: 'after',
          },
        }],
      });

      mockToolManager.executeTool.mockResolvedValue({
        success: true,
        modifiedText: 'Original text\nConclusion text',
        content: 'Conclusion text',
      });

      const result = await service.appendText(request);

      expect(result.success).toBe(true);
      expect(result.originalText).toBe('Original text');
      expect(mockAIService.generateCompletion).toHaveBeenCalled();
      expect(mockToolManager.executeTool).toHaveBeenCalledWith(
        'append_text',
        expect.any(Object)
      );
    });
  });
});
```

---

## Conclusion

This backend documentation provides a comprehensive foundation for building a production-ready AI text manipulation website. The architecture is designed to be:

- **Scalable**: Can handle growing user base and requests
- **Maintainable**: Clean code structure with clear separation of concerns
- **Secure**: Proper authentication, authorization, and data protection
- **Performant**: Optimized for speed with caching and efficient queries
- **Extensible**: Easy to add new features and AI providers
- **Reliable**: Robust error handling and monitoring

### Key Implementation Steps:

1. Set up the development environment
2. Initialize the database schema
3. Implement authentication and user management
4. Build the AI service layer with multiple providers
5. Create the tool system for text manipulation
6. Implement context management
7. Build RESTful API endpoints
8. Add WebSocket support for real-time features
9. Implement comprehensive error handling
10. Add logging and monitoring
11. Write tests for all critical components
12. Deploy to production with proper security measures
13. Set up monitoring and alerting
14. Create backup and disaster recovery plans

### Next Development Phases:

**Phase 1: Core Features** (Weeks 1-4)
- User authentication
- Document management
- Basic text manipulation (append, modify, remove)
- Chat interface

**Phase 2: Advanced Features** (Weeks 5-8)
- Real-time collaboration
- Version history
- Advanced AI features
- Context management

**Phase 3: Production Ready** (Weeks 9-12)
- Performance optimization
- Security hardening
- Monitoring and logging
- Deployment automation

For frontend integration details, refer to the `frontend_documentation.md` file.
