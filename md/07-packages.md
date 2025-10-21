# Packages Directory (packages/) Documentation

The `packages/` directory contains shared libraries and utilities used across different parts of the Kilo Code project. These packages are published to the workspace and can be imported by the extension, CLI, JetBrains plugin, and other applications.

## Directory Structure

```
packages/
├── build/              # Build utilities and helpers
├── cloud/              # Cloud service integration
├── config-eslint/      # Shared ESLint configuration
├── config-typescript/  # Shared TypeScript configuration
├── evals/              # Evaluation framework
├── ipc/                # Inter-process communication
├── telemetry/          # Telemetry and analytics
└── types/              # Shared TypeScript type definitions
```

## Package Overview

### `packages/build/`

**Name**: `@roo-code/build`
**Purpose**: Build utilities and helper functions

**Contents**:
```
build/
├── src/
│   └── index.ts        # Build utilities
├── package.json
├── tsconfig.json
└── vitest.config.ts
```

**Features**:
- Build script utilities
- File manipulation helpers
- Path resolution
- Bundle optimization utilities

**Usage**:
```typescript
import { buildUtils } from '@roo-code/build'
```

**Scripts**:
- `lint`: ESLint checking
- `check-types`: TypeScript type checking
- `clean`: Clean build artifacts

---

### `packages/cloud/`

**Name**: `@roo-code/cloud`
**Purpose**: Cloud service integration for authentication, user management, and remote features

**Contents**:
```
cloud/
├── src/
│   ├── index.ts                # Main exports
│   ├── CloudService.ts         # Main cloud service class
│   ├── BridgeOrchestrator.ts   # Bridge between local and cloud
│   ├── auth/                   # Authentication logic
│   ├── api/                    # API client
│   ├── socket/                 # WebSocket communication
│   └── types/                  # Type definitions
├── package.json
├── tsconfig.json
└── vitest.config.ts
```

**Key Dependencies**:
- `socket.io-client`: Real-time communication
- `ioredis`: Redis client for caching
- `jwt-decode`: JWT token parsing
- `zod`: Schema validation

**Features**:

#### Authentication
- OAuth flow
- Token management
- Session handling
- User identification

#### User Management
- User profile
- Credits tracking
- Subscription management
- Team management

#### Real-time Communication
- WebSocket connections
- Event streaming
- State synchronization
- Notification delivery

#### API Integration
- REST API client
- Request/response handling
- Error handling
- Rate limiting

**Main Classes**:

```typescript
// CloudService - Main service class
class CloudService {
  static createInstance(context, logger): Promise<CloudService>
  authenticate(token: string): Promise<void>
  getUserInfo(): Promise<UserInfo>
  getCredits(): Promise<number>
  // ... more methods
}

// BridgeOrchestrator - Coordinates cloud operations
class BridgeOrchestrator {
  connect(): Promise<void>
  disconnect(): void
  sendMessage(message: Message): Promise<void>
  onMessage(callback: (message: Message) => void): void
}
```

**Usage**:
```typescript
import { CloudService } from '@roo-code/cloud'

const cloud = await CloudService.createInstance(context, logger)
await cloud.authenticate(apiToken)
const userInfo = await cloud.getUserInfo()
```

**Scripts**:
- `lint`: ESLint checking
- `check-types`: TypeScript type checking
- `test`: Run tests
- `clean`: Clean build artifacts

---

### `packages/config-eslint/`

**Name**: `@roo-code/config-eslint`
**Purpose**: Shared ESLint configuration for consistent code quality

**Contents**:
```
config-eslint/
├── base.js         # Base ESLint configuration
├── react.js        # React-specific rules
├── next.js         # Next.js-specific rules
└── package.json
```

**Configurations**:

#### `base.js`
**Purpose**: Base ESLint rules for TypeScript projects

**Includes**:
- TypeScript ESLint parser
- Recommended rules
- Import/export rules
- Code style rules

**Usage** (in `eslint.config.mjs`):
```javascript
import baseConfig from '@roo-code/config-eslint/base'

export default [
  ...baseConfig,
  // Your custom rules
]
```

#### `react.js`
**Purpose**: React-specific ESLint rules

**Includes**:
- React plugin
- React hooks rules
- JSX rules
- Accessibility rules

**Usage**:
```javascript
import reactConfig from '@roo-code/config-eslint/react'

export default [
  ...reactConfig,
  // Your custom rules
]
```

#### `next.js`
**Purpose**: Next.js-specific ESLint rules

**Includes**:
- Next.js plugin
- Next.js specific rules
- Performance rules

**Usage**:
```javascript
import nextConfig from '@roo-code/config-eslint/next'

export default [
  ...nextConfig,
  // Your custom rules
]
```

**Dependencies**:
- `@eslint/js`
- `@typescript-eslint/eslint-plugin`
- `@typescript-eslint/parser`
- `eslint-plugin-react`
- `eslint-plugin-react-hooks`
- `eslint-plugin-import`

---

### `packages/config-typescript/`

**Name**: `@roo-code/config-typescript`
**Purpose**: Shared TypeScript configuration for consistent compilation

**Contents**:
```
config-typescript/
├── base.json              # Base TypeScript config
├── cjs.json               # CommonJS config
├── nextjs.json            # Next.js config
├── vscode-library.json    # VS Code library config
└── package.json
```

**Configurations**:

#### `base.json`
**Purpose**: Base TypeScript configuration

**Settings**:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  }
}
```

**Usage** (in `tsconfig.json`):
```json
{
  "extends": "@roo-code/config-typescript/base.json",
  "compilerOptions": {
    // Your overrides
  }
}
```

#### `cjs.json`
**Purpose**: CommonJS module configuration

**Usage**: For Node.js libraries using CommonJS

#### `nextjs.json`
**Purpose**: Next.js specific configuration

**Usage**: For Next.js applications

#### `vscode-library.json`
**Purpose**: VS Code extension library configuration

**Usage**: For VS Code extension development

---

### `packages/evals/`

**Name**: `@roo-code/evals`
**Purpose**: Evaluation framework for testing AI agent performance

**Contents**:
```
evals/
├── .docker/                # Docker configurations
├── src/
│   ├── db/                 # Database (Drizzle ORM)
│   ├── server/             # Express server
│   ├── runner/             # Test runner
│   ├── evaluators/         # Evaluation logic
│   └── types/              # Type definitions
├── scripts/                # Utility scripts
├── docker-compose.yml      # Docker Compose setup
├── Dockerfile.runner       # Runner container
├── Dockerfile.web          # Web UI container
├── ARCHITECTURE.md         # Architecture documentation
├── ADDING-EVALS.md         # Guide for adding evaluations
└── README.md               # Setup and usage guide
```

**Purpose**: Test and evaluate AI agent capabilities

**Components**:

#### Database (Drizzle ORM)
- Test cases storage
- Results tracking
- Metrics collection

#### Server (Express)
- REST API
- Test execution
- Results API

#### Runner
- Execute test cases
- Capture results
- Report metrics

#### Evaluators
- Code quality evaluation
- Task completion checking
- Performance metrics
- Correctness validation

**Usage**:
```bash
# Start evaluation server
pnpm evals

# Run specific evaluation
pnpm --filter @roo-code/evals run-eval <eval-name>
```

**Docker Compose**:
- `server`: Web server and API
- `runner`: Test execution
- `db`: PostgreSQL database
- `redis`: Caching layer

**Documentation**:
- `ARCHITECTURE.md`: System architecture
- `ADDING-EVALS.md`: How to add new evaluations
- `README.md`: Setup guide

---

### `packages/ipc/`

**Name**: `@roo-code/ipc`
**Purpose**: Inter-process communication library

**Contents**:
```
ipc/
├── src/
│   ├── index.ts        # Main exports
│   ├── protocol.ts     # IPC protocol definition
│   ├── client.ts       # IPC client
│   └── server.ts       # IPC server
├── README.md
├── package.json
└── tsconfig.json
```

**Purpose**: Enable communication between different processes (e.g., extension host and plugin)

**Features**:

#### Protocol
- JSON-RPC 2.0 based
- Request/response pattern
- Notification support
- Error handling

#### Transport
- stdin/stdout (CLI)
- WebSocket (browser)
- Named pipes (Windows)
- Unix sockets (Linux/macOS)

#### Client
```typescript
import { IPCClient } from '@roo-code/ipc'

const client = new IPCClient(transport)
await client.connect()

// Send request
const response = await client.request('method', params)

// Send notification
client.notify('event', data)
```

#### Server
```typescript
import { IPCServer } from '@roo-code/ipc'

const server = new IPCServer(transport)
server.on('request', async (method, params) => {
  // Handle request
  return result
})

server.on('notification', (method, params) => {
  // Handle notification
})
```

**Use Cases**:
- CLI ↔ Extension Host communication
- JetBrains Plugin ↔ Extension Host communication
- Multi-process architectures

---

### `packages/telemetry/`

**Name**: `@roo-code/telemetry`
**Purpose**: Telemetry and analytics service

**Contents**:
```
telemetry/
├── src/
│   ├── index.ts                # Main exports
│   ├── TelemetryService.ts     # Main service
│   ├── TelemetryClient.ts      # Client interface
│   ├── PostHogClient.ts        # PostHog implementation
│   └── types.ts                # Type definitions
├── package.json
├── tsconfig.json
└── vitest.config.ts
```

**Purpose**: Track usage, errors, and analytics

**Features**:

#### Event Tracking
- User actions
- Feature usage
- Performance metrics
- Error tracking

#### Clients
- PostHog (default)
- Custom implementations

#### Privacy
- Anonymous by default
- User opt-out support
- GDPR compliance
- Data minimization

**Main Classes**:

```typescript
// TelemetryService - Singleton service
class TelemetryService {
  static instance: TelemetryService
  static createInstance(): TelemetryService
  
  register(client: TelemetryClient): void
  track(event: string, properties?: object): void
  identify(userId: string, traits?: object): void
  setEnabled(enabled: boolean): void
}

// TelemetryClient - Client interface
interface TelemetryClient {
  track(event: string, properties?: object): void
  identify(userId: string, traits?: object): void
  flush(): Promise<void>
}
```

**Usage**:
```typescript
import { TelemetryService, PostHogTelemetryClient } from '@roo-code/telemetry'

// Initialize
const telemetry = TelemetryService.createInstance()
telemetry.register(new PostHogTelemetryClient())

// Track event
telemetry.track('task_started', {
  mode: 'coder',
  model: 'claude-3-5-sonnet'
})

// Identify user
telemetry.identify('user-id', {
  plan: 'pro'
})
```

**Events Tracked**:
- Extension activation
- Task start/completion
- Command execution
- Tool usage
- Errors
- Performance metrics

**Dependencies**:
- `posthog-js`: PostHog analytics

---

### `packages/types/`

**Name**: `@roo-code/types`
**Purpose**: Shared TypeScript type definitions

**Contents**:
```
types/
├── src/
│   ├── index.ts            # Main exports
│   ├── api/                # API types
│   ├── cloud/              # Cloud service types
│   ├── config/             # Configuration types
│   ├── extension/          # Extension types
│   ├── messages/           # Message types
│   └── tools/              # Tool types
├── npm/                    # NPM package configuration
├── scripts/                # Build scripts
├── package.json
└── tsconfig.json
```

**Purpose**: Centralized type definitions shared across packages

**Type Categories**:

#### API Types
```typescript
export interface ApiConfiguration {
  apiProvider: string
  apiKey?: string
  apiModelId: string
  apiBaseUrl?: string
}

export interface ModelInfo {
  id: string
  name: string
  maxTokens: number
  supportsImages: boolean
  supportsPromptCache: boolean
}
```

#### Cloud Types
```typescript
export interface CloudUserInfo {
  id: string
  email: string
  name?: string
  credits: number
  subscription?: {
    plan: string
    status: string
  }
}

export interface AuthState {
  authenticated: boolean
  token?: string
  expiresAt?: number
}
```

#### Configuration Types
```typescript
export interface ExtensionSettings {
  autoApproval: AutoApprovalSettings
  diffEnabled: boolean
  alwaysAllowReadOnly: boolean
  // ... more settings
}
```

#### Message Types
```typescript
export interface UserMessage {
  type: 'user'
  text: string
  images?: string[]
}

export interface AssistantMessage {
  type: 'assistant'
  text?: string
  toolCalls?: ToolCall[]
}
```

#### Tool Types
```typescript
export interface ToolCall {
  id: string
  name: string
  arguments: object
}

export interface ToolResult {
  id: string
  success: boolean
  result?: any
  error?: string
}
```

**Publishing**:
```bash
# Build types
pnpm --filter @roo-code/types build

# Publish to npm
pnpm npm:publish:types
```

**Usage**:
```typescript
import type { 
  ApiConfiguration, 
  UserMessage, 
  ToolCall 
} from '@roo-code/types'

const config: ApiConfiguration = {
  apiProvider: 'anthropic',
  apiModelId: 'claude-3-5-sonnet'
}
```

**Dependencies**: None (pure types)

---

## Workspace Configuration

All packages use:

### pnpm Workspaces
```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
```

### Turbo Build System
```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "lint": {},
    "test": {}
  }
}
```

### TypeScript Project References
Each package's `tsconfig.json` can reference others:
```json
{
  "references": [
    { "path": "../types" }
  ]
}
```

## Development Workflow

### Adding a New Package

1. **Create directory**:
```bash
mkdir packages/my-package
cd packages/my-package
```

2. **Initialize package.json**:
```json
{
  "name": "@roo-code/my-package",
  "version": "0.0.0",
  "type": "module",
  "exports": "./src/index.ts",
  "scripts": {
    "lint": "eslint src --ext=ts",
    "check-types": "tsc --noEmit",
    "test": "vitest run"
  }
}
```

3. **Add TypeScript config**:
```json
{
  "extends": "@roo-code/config-typescript/base.json",
  "compilerOptions": {
    "outDir": "dist"
  }
}
```

4. **Create source files**:
```typescript
// src/index.ts
export * from './my-feature'
```

5. **Update workspace** (if needed):
```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'packages/my-package'  # Usually automatic
```

### Using a Package

In any workspace package:

```json
{
  "dependencies": {
    "@roo-code/my-package": "workspace:^"
  }
}
```

Then import:
```typescript
import { myFunction } from '@roo-code/my-package'
```

### Building Packages

```bash
# Build all packages
pnpm build

# Build specific package
pnpm --filter @roo-code/cloud build

# Build with dependencies
turbo build --filter=@roo-code/cloud
```

## Best Practices

1. **Type Safety**: Use TypeScript for all packages
2. **Shared Config**: Extend base ESLint and TypeScript configs
3. **Documentation**: Include README.md in each package
4. **Testing**: Write tests for all packages
5. **Versioning**: Use semantic versioning
6. **Exports**: Use clear, documented exports
7. **Dependencies**: Minimize external dependencies
8. **Scope**: Keep packages focused and single-purpose

---

*Last updated: 2025-10-21*
