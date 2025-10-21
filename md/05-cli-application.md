# CLI Application (cli/) Documentation

The `cli/` directory contains the command-line interface for Kilo Code, providing a terminal-based user interface (TUI) for AI-powered development.

## Directory Structure

```
cli/
├── docs/                   # CLI documentation
├── src/                    # Source code
│   ├── __tests__/          # Tests
│   ├── commands/           # CLI commands
│   ├── communication/      # IPC communication
│   ├── config/             # Configuration management
│   ├── constants/          # Constants and defaults
│   ├── host/               # Host process management
│   ├── services/           # Service implementations
│   ├── state/              # State management
│   ├── types/              # TypeScript types
│   ├── ui/                 # UI components (TUI)
│   ├── utils/              # Utility functions
│   └── index.ts            # Main entry point
├── package.json            # Dependencies and scripts
├── package.dist.json       # Distribution package.json
├── npm-shrinkwrap.dist.json # Distribution lockfile
├── esbuild.config.mjs      # Build configuration
└── tsconfig.json           # TypeScript configuration
```

## Overview

The Kilo Code CLI is a terminal user interface that provides:
- Interactive chat with AI agent
- Autonomous mode for CI/CD pipelines
- File operations and code generation
- Project workspace integration
- MCP server support
- Multiple AI models and modes

## Installation

### Via npm (Global)
```bash
npm install -g @kilocode/cli
```

### Via pnpm (Global)
```bash
pnpm add -g @kilocode/cli
```

### From Source
```bash
cd cli
pnpm install
pnpm build
pnpm link --global
```

## Configuration

### Config File Location
- **Linux/macOS**: `~/.config/kilocode/config.json`
- **Windows**: `%APPDATA%\kilocode\config.json`

### Edit Configuration
```bash
kilocode config
```

This opens your default editor with the config file.

### Configuration Schema

```json
{
  "kilocodeToken": "your-api-token",
  "theme": "dark",
  "autoApproval": {
    "enabled": true,
    "read": {
      "enabled": true,
      "outside": true
    },
    "write": {
      "enabled": true,
      "outside": false,
      "protected": false
    },
    "execute": {
      "enabled": true,
      "allowed": ["npm", "git", "pnpm"],
      "denied": ["rm -rf", "sudo"]
    },
    "browser": {
      "enabled": false
    },
    "mcp": {
      "enabled": true
    },
    "mode": {
      "enabled": true
    },
    "subtasks": {
      "enabled": true
    },
    "question": {
      "enabled": false,
      "timeout": 60
    },
    "retry": {
      "enabled": true,
      "delay": 10
    },
    "todo": {
      "enabled": true
    }
  },
  "defaultMode": "coder",
  "defaultModel": "claude-3-5-sonnet"
}
```

## Usage Modes

### Interactive Mode

**Basic Usage**:
```bash
kilocode
```

**With Options**:
```bash
# Start with specific mode
kilocode --mode architect

# Start in specific workspace
kilocode --workspace /path/to/project

# Specify model
kilocode --model gpt-4
```

**Interactive Features**:
- Real-time chat with AI
- File browsing and editing
- Command execution approval
- Tool usage visualization
- Progress tracking

### Autonomous Mode

**Purpose**: Run without user interaction (CI/CD, automation)

**Basic Usage**:
```bash
# Run with prompt
kilocode --auto "Fix all linting errors"

# Run with piped input
echo "Implement feature X" | kilocode --auto

# Run with timeout (seconds)
kilocode --auto "Run tests and fix failures" --timeout 600
```

**Autonomous Behavior**:
1. No user prompts - all decisions automatic
2. Auto-approval based on configuration
3. Follow-up questions handled automatically
4. Exits when task completes or times out

**Exit Codes**:
- `0`: Success
- `1`: Error
- `124`: Timeout

**CI/CD Example** (GitHub Actions):
```yaml
- name: Run Kilo Code
  run: |
    echo "Implement the feature" | kilocode --auto --timeout 600
  env:
    KILOCODE_TOKEN: ${{ secrets.KILOCODE_TOKEN }}
```

## Source Code Structure

### `src/index.ts`
**Purpose**: Main entry point

**Responsibilities**:
- Parse command-line arguments
- Load configuration
- Initialize services
- Start UI or autonomous mode
- Handle errors and exit codes

### `src/commands/`
**Purpose**: CLI command implementations

**Command Registry** (`core/registry.ts`):
- Command registration
- Command lookup
- Help text generation

**Command Parser** (`core/parser.ts`):
- Parse user input
- Extract command and arguments
- Validate syntax

**Available Commands**:

#### `exit.ts` or `/exit`
**Purpose**: Exit the CLI
**Usage**: `/exit` or Ctrl+C

#### `new.ts` or `/new`
**Purpose**: Start a new conversation
**Usage**: `/new`
**Effect**: Clears history, starts fresh

#### `mode.ts` or `/mode`
**Purpose**: Switch AI modes
**Usage**: `/mode <mode-name>`
**Examples**: 
- `/mode architect` - Planning mode
- `/mode coder` - Coding mode
- `/mode debugger` - Debugging mode

#### `model.ts` or `/model`
**Purpose**: Switch AI model
**Usage**: `/model <model-name>`
**Examples**:
- `/model claude-3-5-sonnet`
- `/model gpt-4`
**Features**: Auto-completion of model names

#### `teams.ts` or `/teams`
**Purpose**: Manage team settings
**Usage**: `/teams`

#### `profile.ts` or `/profile`
**Purpose**: View user profile
**Usage**: `/profile`
**Shows**: Credits, subscription, usage

#### `help.ts` or `/help`
**Purpose**: Show help
**Usage**: `/help` or `/help <command>`

### `src/ui/`
**Purpose**: Terminal UI components

**Technology**: Ink (React for CLI)

**Components**:

#### `App.tsx`
**Purpose**: Main application component
**Features**:
- Chat interface
- Message list
- Input box
- Status bar

#### `MessageList.tsx`
**Purpose**: Display conversation history
**Features**:
- User messages
- AI responses
- Tool usage
- Thinking indicators
- Code blocks with syntax highlighting

#### `InputBox.tsx`
**Purpose**: User input component
**Features**:
- Text input
- Multi-line support
- Command auto-complete
- Keyboard shortcuts

#### `StatusBar.tsx`
**Purpose**: Status information
**Features**:
- Current mode
- Model selection
- Token usage
- Credits remaining

#### `CodeBlock.tsx`
**Purpose**: Render code blocks
**Features**:
- Syntax highlighting
- Language detection
- Copy functionality

#### `Spinner.tsx`
**Purpose**: Loading indicator
**Features**:
- Animated spinner
- Status messages

#### `ApprovalPrompt.tsx`
**Purpose**: Request user approval
**Features**:
- File operation approval
- Command execution approval
- Customizable messages

### `src/services/`
**Purpose**: Service layer

#### `extension.ts`
**Purpose**: Extension host communication
**Features**:
- Start extension host process
- Message passing
- Process lifecycle management

#### `commandExecutor.ts`
**Purpose**: Execute shell commands
**Features**:
- Command execution
- Output capture
- Error handling
- Interactive commands

#### `approvalDecision.ts`
**Purpose**: Auto-approval logic
**Features**:
- Check approval rules
- Make decisions based on config
- Log approval decisions

#### `autocomplete.ts`
**Purpose**: Auto-completion service
**Features**:
- Command suggestions
- Model name completion
- Mode name completion
- File path completion

#### `telemetry/`
**Purpose**: Usage analytics

**Files**:
- `TelemetryService.ts`: Main service
- `TelemetryClient.ts`: Client implementation
- `identity.ts`: User identification
- `events.ts`: Event definitions

**Events Tracked**:
- CLI start/stop
- Commands executed
- Errors encountered
- Task completion
- Feature usage

### `src/communication/`
**Purpose**: Inter-process communication

**Protocol**:
- JSON-RPC messages
- Request/response pattern
- Event streaming

**Message Types**:
- User input
- AI responses
- Tool calls
- Approval requests
- State updates

### `src/config/`
**Purpose**: Configuration management

**Features**:
- Load config from file
- Save config updates
- Validate configuration
- Default values
- Migration from old formats

**Config Manager**:
```typescript
class ConfigManager {
  load(): Config
  save(config: Config): void
  get<T>(key: string): T
  set<T>(key: string, value: T): void
}
```

### `src/state/`
**Purpose**: Application state management

**State Structure**:
```typescript
interface AppState {
  conversation: Message[]
  currentTask: Task | null
  mode: string
  model: string
  workspace: string
  user: UserInfo | null
}
```

**State Manager**:
- Centralized state
- State updates
- State persistence
- Undo/redo support

### `src/host/`
**Purpose**: Host process management

**Responsibilities**:
- Start extension host (Node.js process)
- Manage host lifecycle
- Handle host crashes
- Restart on errors

**Host Process**:
- Runs core extension code
- Executes tools
- Manages file system
- Communicates with CLI via IPC

### `src/utils/`
**Purpose**: Utility functions

**Key Utils**:

#### `env-loader.ts`
**Purpose**: Load environment variables
**Features**:
- .env file loading
- Environment detection
- Variable validation

#### `extension-paths.ts`
**Purpose**: Extension path resolution
**Features**:
- Find extension installation
- Resolve extension resources
- Handle bundled vs development

#### `paths.ts`
**Purpose**: Path utilities
**Features**:
- Path normalization
- Workspace detection
- Config directory

#### `git.ts`
**Purpose**: Git utilities
**Features**:
- Detect git repository
- Get current branch
- Check for changes
- Get git root

#### `context.ts`
**Purpose**: Context utilities
**Features**:
- Workspace context
- File context
- Environment context

#### `providers.ts`
**Purpose**: AI provider utilities
**Features**:
- Provider detection
- Model listing
- Provider configuration

#### `auto-update.ts`
**Purpose**: Auto-update checking
**Features**:
- Check for updates
- Notify user
- Download updates

### `src/types/`
**Purpose**: TypeScript type definitions

**Type Files**:

#### `cli.ts`
**Purpose**: CLI-specific types
**Types**:
- CommandLineArgs
- CLIOptions
- ExitCode

#### `messages.ts`
**Purpose**: Message types
**Types**:
- UserMessage
- AssistantMessage
- ToolMessage
- SystemMessage

#### `theme.ts`
**Purpose**: Theme types
**Types**:
- Theme (light/dark)
- ColorScheme
- ThemeConfig

#### `keyboard.ts`
**Purpose**: Keyboard types
**Types**:
- KeyBinding
- Shortcut
- KeyEvent

### `src/constants/`
**Purpose**: Constants and defaults

**Constants**:
- Default configuration values
- Exit codes
- Timeout values
- API endpoints
- File paths

## Build Process

### `esbuild.config.mjs`
**Purpose**: Build configuration

**Build Steps**:
1. Compile TypeScript
2. Bundle with esbuild
3. Create standalone executable
4. Copy runtime dependencies
5. Create distribution package

**Output**:
- `dist/index.js` - Bundled CLI
- `dist/cli` - Executable binary (optional)

### Build Scripts

**Development Build**:
```bash
pnpm build
```

**Bundle for Distribution**:
```bash
pnpm bundle
```

**Create Executable**:
```bash
pnpm pkg
```

## Distribution

### Package Types

#### npm Package
**Name**: `@kilocode/cli`
**Files**:
- Bundled JavaScript
- Runtime dependencies
- Binary executable

#### Standalone Binary
**Platforms**:
- Linux (x64, arm64)
- macOS (x64, arm64)
- Windows (x64)

**Distribution**:
- GitHub Releases
- Direct download
- Package managers (brew, choco)

### Dependencies

**Runtime Dependencies** (bundled):
- Core extension code
- Node.js built-ins
- Third-party libraries

**External Dependencies** (not bundled):
- Node.js runtime (required)

## Testing

### Test Framework
- Vitest for unit tests
- Mock services for integration tests

### Running Tests
```bash
pnpm test              # Run all tests
pnpm test:watch        # Watch mode
pnpm test:coverage     # With coverage
```

### Test Coverage
- Commands: Fully tested
- Services: Integration tests
- Utils: Unit tests
- UI: Component tests

## Known Issues

### Theme Detection
- No automatic theme detection
- Default dark theme doesn't work well on light terminals
- **Workaround**: Use `kilocode config` to switch to light theme

### Dependency Warnings
- Some outdated dependency warnings during installation
- Planned to be fixed in future releases

### Windows Support
- Limited testing on Windows
- Some issues with path handling
- **Recommendation**: Use WSL on Windows

## Development

### Local Development

**Setup**:
```bash
cd cli
pnpm install
```

**Run in Development**:
```bash
pnpm start
```

**With DevTools**:
```bash
DEV=true pnpm start
# In another terminal:
npx react-devtools
```

**Build and Test**:
```bash
pnpm build
pnpm test
node dist/index.js
```

### Debugging

**Enable Debug Logging**:
```bash
DEBUG=* kilocode
```

**VS Code Debugging**:
- Use launch configuration
- Set breakpoints in TypeScript
- Source maps enabled

## Architecture

### Process Model

```
┌─────────────────┐
│   CLI Process   │
│  (User Input)   │
└────────┬────────┘
         │ IPC
         ▼
┌─────────────────┐
│  Host Process   │
│  (Extension)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   AI Provider   │
│  (API Calls)    │
└─────────────────┘
```

**CLI Process**:
- Renders UI
- Handles user input
- Manages state

**Host Process**:
- Executes tools
- File operations
- API communication

**IPC Communication**:
- JSON-RPC protocol
- Bidirectional messaging
- Event streaming

### Data Flow

1. **User Input** → CLI receives command/message
2. **Parse** → Command parser or direct message
3. **Send to Host** → IPC message to extension host
4. **Process** → Host executes tools, calls AI
5. **Stream Response** → AI response streamed back
6. **Update UI** → CLI updates display
7. **Request Approval** → If needed, prompt user
8. **Continue** → Loop until task complete

---

*Last updated: 2025-10-21*
