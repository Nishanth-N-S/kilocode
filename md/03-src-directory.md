# Source Directory (src/) Documentation

The `src/` directory contains the core VS Code extension source code for Kilo Code. This is the main application logic that powers the AI agent functionality.

## Directory Structure

```
src/
├── __mocks__/              # Mock implementations for testing
├── __tests__/              # Extension-level tests
├── activate/               # Extension activation logic
├── api/                    # API providers and transformations
├── assets/                 # Static assets (icons, images, docs)
├── core/                   # Core functionality and business logic
├── extension/              # Extension entry points and configuration
├── i18n/                   # Internationalization (i18n) support
├── integrations/           # VS Code API integrations
├── services/               # Service implementations
├── shared/                 # Shared utilities and types
├── utils/                  # Utility functions
├── walkthrough/            # VS Code walkthrough content
├── workers/                # Web workers for background tasks
└── extension.ts            # Main extension entry point
```

## Main Entry Point

### `extension.ts`
**Purpose**: Main extension activation and deactivation

**Key Functions**:
- `activate()`: Called when extension is activated
- `deactivate()`: Called when extension is deactivated

**Responsibilities**:
1. Initialize output channel for logging
2. Migrate old settings to new format
3. Initialize telemetry service
4. Set up cloud services (authentication, user info)
5. Initialize i18n (internationalization)
6. Create and register ClineProvider (main webview)
7. Register commands, code actions, terminal actions
8. Set up URI handlers
9. Initialize various services (MCP, MDM, commit message, etc.)
10. Set up event handlers for auth state, settings, user info

**Key Dependencies**:
- `ClineProvider`: Main webview provider
- `CloudService`: Cloud integration
- `TelemetryService`: Usage tracking
- Various service managers

## Core Directories

### `activate/`
**Purpose**: Extension activation logic and command registration

**Key Files**:
- `handle-uri.ts`: Handle custom URI schemes
- `registerCommands.ts`: Register VS Code commands
- `registerCodeActions.ts`: Register code action providers
- `registerTerminalActions.ts`: Register terminal-related actions

**Registered Commands**:
- Open Kilo Code chat
- Create new task
- Run custom modes
- Import/export settings
- Manage API configurations
- And many more...

### `api/`
**Purpose**: API provider integrations and message transformations

#### `api/providers/`
**Purpose**: Different AI API provider implementations

**Providers**:
- `claude-code.ts`: Anthropic Claude integration
- `vercel-ai-gateway.ts`: Vercel AI Gateway
- `kilocode-openrouter.ts`: Kilo Code OpenRouter
- `ollama.ts`: Ollama local models
- `native-ollama.ts`: Native Ollama implementation
- `deepseek.ts`: DeepSeek API
- `lite-llm.ts`: LiteLLM proxy
- `glama.ts`: Glama API
- `zai.ts`: ZAI API
- `unbound.ts`: Unbound API
- `router-provider.ts`: Router for multiple providers

**Each provider implements**:
- Authentication
- Request formatting
- Response parsing
- Streaming support
- Error handling

#### `api/transform/`
**Purpose**: Message format transformations between different APIs

**Key Transformations**:
- `gemini-format.ts`: Google Gemini API format
- `openai-format.ts`: OpenAI API format
- `bedrock-converse-format.ts`: AWS Bedrock format
- `mistral-format.ts`: Mistral AI format
- `vscode-lm-format.ts`: VS Code Language Model format
- `r1-format.ts`: R1 format
- `simple-format.ts`: Simplified format

**Additional Features**:
- `stream.ts`: Streaming response handling
- `reasoning.ts`: Reasoning step processing
- `image-cleaning.ts`: Image data processing
- `model-params.ts`: Model parameter handling

**Caching**:
- `caching/anthropic.ts`: Anthropic prompt caching
- `caching/gemini.ts`: Gemini caching
- `caching/vertex.ts`: Vertex AI caching
- `caching/vercel-ai-gateway.ts`: Vercel AI Gateway caching

### `core/`
**Purpose**: Core business logic and functionality

#### `core/webview/`
**Purpose**: Webview management and message handling

**Key Files**:
- `ClineProvider.ts`: Main webview provider class
  - Manages webview lifecycle
  - Handles state persistence
  - Coordinates message passing
- `webviewMessageHandler.ts`: Handles messages from webview
  - User actions
  - Configuration updates
  - Task management
- `generateSystemPrompt.ts`: Generates system prompts for AI
- `messageEnhancer.ts`: Enhances messages with context
- `checkpointRestoreHandler.ts`: Checkpoint restoration logic

#### `core/assistant-message/`
**Purpose**: Parse and process assistant messages

**Key Files**:
- `parseAssistantMessage.ts`: Parse assistant responses
- `parseAssistantMessageV2.ts`: V2 parser implementation
- `AssistantMessageParser.ts`: Parser class
- `presentAssistantMessage.ts`: Format messages for display

**Handles**:
- Tool calls extraction
- Text content parsing
- Thinking/reasoning steps
- Multi-modal content

#### `core/tools/`
**Purpose**: Tool implementations for AI agent

**Available Tools**:
- File operations (read, write, edit)
- Command execution (bash, shell)
- Browser automation
- Code search and analysis
- Git operations
- And many more...

**Tool Types**:
- Synchronous tools
- Asynchronous tools
- Interactive tools
- Stateful tools

#### `core/context/`
**Purpose**: Context management for AI conversations

**Features**:
- Context window management
- Token counting
- Context pruning
- History management

#### `core/prompts/`
**Purpose**: System prompts and instructions

**Contains**:
- Base system prompt
- Mode-specific prompts
- Tool usage instructions
- Coding guidelines

#### `core/config/`
**Purpose**: Configuration management

**Key Files**:
- `ContextProxy.ts`: VS Code context proxy
- Configuration validation
- Settings migration

#### `core/task/`
**Purpose**: Task execution and management

**Features**:
- Task state machine
- Task persistence
- Task history
- Task cancellation

#### `core/slash-commands/`
**Purpose**: Slash command handling

**Commands**:
- `/file`: Include file in context
- `/web`: Search web
- `/mode`: Switch modes
- And more...

#### Other Core Modules:
- `core/checkpoints/`: Checkpoint system for task state
- `core/condense/`: Message condensation for context
- `core/diff/`: Diff generation and application
- `core/environment/`: Environment detection
- `core/ignore/`: File ignore patterns (.rooignore)
- `core/mentions/`: @-mention handling
- `core/message-queue/`: Message queuing
- `core/protect/`: Protected file handling
- `core/sliding-window/`: Sliding window context
- `core/kilocode/`: Kilo-specific features

### `integrations/`
**Purpose**: VS Code API integrations

#### `integrations/editor/`
**Purpose**: Editor-related integrations

**Features**:
- Diff view provider
- Text editor utilities
- Selection handling
- Cursor management

#### `integrations/terminal/`
**Purpose**: Terminal integration

**Features**:
- Terminal registry
- Terminal execution
- Output capture
- Interactive commands

#### `integrations/workspace/`
**Purpose**: Workspace integration

**Features**:
- Workspace detection
- Multi-folder workspaces
- Workspace configuration

#### `integrations/diagnostics/`
**Purpose**: Diagnostic information

**Features**:
- Error detection
- Warning collection
- Problem panel integration

#### `integrations/notifications/`
**Purpose**: User notifications

**Features**:
- Toast notifications
- Progress indicators
- Error messages

#### `integrations/theme/`
**Purpose**: Theme integration

**Features**:
- Theme detection
- Color customization
- Icon themes

#### `integrations/claude-code/`
**Purpose**: Claude Code specific integrations

#### `integrations/misc/`
**Purpose**: Miscellaneous integrations

### `services/`
**Purpose**: Service implementations

#### `services/mcp/`
**Purpose**: Model Context Protocol (MCP) server management

**Features**:
- MCP server discovery
- MCP tool execution
- Server lifecycle management

**Key Files**:
- `McpServerManager.ts`: Main manager class
- Server configurations
- Tool registry

#### `services/browser/`
**Purpose**: Browser automation service

**Features**:
- Playwright integration
- Page navigation
- Element interaction
- Screenshot capture

#### `services/code-index/`
**Purpose**: Code indexing and search

**Features**:
- File indexing
- Symbol search
- Full-text search
- Dependency analysis

#### `services/tree-sitter/`
**Purpose**: Tree-sitter integration for code parsing

**Features**:
- Syntax tree parsing
- Symbol extraction
- Code structure analysis

#### `services/ripgrep/`
**Purpose**: Fast file searching using ripgrep

**Features**:
- File content search
- Regex support
- Multi-file search

#### `services/glob/`
**Purpose**: File globbing and pattern matching

**Features**:
- Glob pattern parsing
- File matching
- Ignore pattern support

#### `services/marketplace/`
**Purpose**: VS Code marketplace integration

**Features**:
- Extension search
- Extension installation
- MCP server marketplace

#### `services/commit-message/`
**Purpose**: AI-powered commit message generation

**Features**:
- Analyze git diff
- Generate commit messages
- Follow conventional commits

#### `services/ghost/`
**Purpose**: Ghost text suggestions (Kilo-specific)

**Features**:
- Inline suggestions
- Code completion
- AI-powered hints

#### `services/checkpoints/`
**Purpose**: Checkpoint service for task state

#### `services/command/`
**Purpose**: Command execution service

#### `services/mdm/`
**Purpose**: Multi-Device Management service

#### `services/mocking/`
**Purpose**: Mocking utilities for testing

#### `services/roo-config/`
**Purpose**: Roo Code configuration management

#### `services/search/`
**Purpose**: Search utilities

#### `services/terminal-welcome/`
**Purpose**: Terminal welcome message

### `shared/`
**Purpose**: Shared utilities and types

**Key Modules**:
- `shared/package.ts`: Package information
- `shared/language.ts`: Language formatting
- `shared/utils/`: Common utilities
- `shared/kilocode/`: Kilo-specific shared code

### `utils/`
**Purpose**: Utility functions

**Key Utilities**:
- `outputChannelLogger.ts`: Logging utilities
- `path.ts`: Path manipulation
- `migrateSettings.ts`: Settings migration
- `autoImportSettings.ts`: Auto-import configuration
- `autoLaunchingTask.ts`: Auto-launch tasks
- `logging/`: Logging infrastructure

### `assets/`
**Purpose**: Static assets

**Contains**:
- `icons/`: Extension icons
- `images/`: Images for UI
- `docs/`: Documentation assets
- `codicons/`: VS Code icons

### `i18n/`
**Purpose**: Internationalization support

**Features**:
- Translation loading
- Locale detection
- Message formatting

**Locales**:
- Multiple language support
- Translation files in `locales/`

### `walkthrough/`
**Purpose**: VS Code walkthrough content

**Features**:
- Getting started guide
- Feature introduction
- Tutorial steps

### `workers/`
**Purpose**: Web workers for background tasks

**Use Cases**:
- Heavy computations
- Non-blocking operations
- Parallel processing

### `extension/`
**Purpose**: Extension configuration and API

**Key Files**:
- `api.ts`: Public extension API
- Extension manifest helpers

## Package Configuration

### `package.json`
**Purpose**: Extension manifest and dependencies

**Key Sections**:
- `name`: Extension identifier
- `displayName`: User-facing name
- `contributes`: VS Code contributions
  - Commands
  - Views
  - Configuration
  - Keybindings
  - Menus
- `activationEvents`: When to activate
- `dependencies`: Runtime dependencies
- `devDependencies`: Development dependencies

**Activation Events**:
- `onStartupFinished`: Activate on startup
- `onCommand`: Activate on specific commands
- `onUri`: Activate on URI schemes

## Build Configuration

### `esbuild.mjs`
**Purpose**: esbuild bundling configuration

**Features**:
- TypeScript compilation
- Code bundling
- Minification
- Source maps
- External dependencies

### `tsconfig.json`
**Purpose**: TypeScript compiler configuration

**Settings**:
- Strict mode enabled
- Module resolution
- Path aliases
- Output directory

## Testing

### `__tests__/`
**Purpose**: Extension-level integration tests

**Test Framework**: Vitest or Jest

### `__mocks__/`
**Purpose**: Mock implementations

**Mocks**:
- VS Code API mocks
- File system mocks
- Network mocks

**Usage**: Used by tests to simulate VS Code environment

## Key Patterns and Practices

### Dependency Injection
- Services are injected into components
- Easier testing and mocking

### Event-Driven Architecture
- VS Code events trigger actions
- Custom event emitters for internal communication

### State Management
- Centralized state in ClineProvider
- Message passing for state updates
- Persistence to extension context

### Error Handling
- Try-catch blocks
- Error logging
- User-friendly error messages
- Telemetry for errors

### Async/Await
- Extensive use of promises
- Async operations throughout
- Proper error propagation

## Development Workflow

1. **Make changes** in relevant source files
2. **Press F5** to launch extension development host
3. **Test changes** in development window
4. **Check output** in Output panel (Kilo Code)
5. **Debug** using VS Code debugger
6. **Write tests** for new functionality
7. **Run tests** with `pnpm test`

## Common Tasks

### Adding a New Command
1. Define command in `package.json` > `contributes.commands`
2. Implement command handler in `activate/registerCommands.ts`
3. Add keyboard shortcut if needed in `package.json` > `contributes.keybindings`

### Adding a New Tool
1. Create tool implementation in `core/tools/`
2. Register tool in tool registry
3. Add tool to system prompt
4. Write tests for tool

### Adding a New API Provider
1. Create provider in `api/providers/`
2. Implement provider interface
3. Add format transformer if needed in `api/transform/`
4. Register provider in configuration

### Adding a New Service
1. Create service directory in `services/`
2. Implement service class
3. Initialize service in `extension.ts`
4. Inject service where needed

---

*Last updated: 2025-10-21*
