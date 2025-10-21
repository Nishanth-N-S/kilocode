# JetBrains Plugin (jetbrains/) Documentation

The `jetbrains/` directory contains the JetBrains IDE plugin implementation for Kilo Code, supporting IntelliJ IDEA, PyCharm, WebStorm, and other JetBrains IDEs.

## Directory Structure

```
jetbrains/
├── host/                          # Extension Host (Node.js/TypeScript)
│   ├── src/                       # TypeScript source code
│   │   ├── main.ts                # Main entry point
│   │   ├── extension.ts           # Extension initialization
│   │   ├── extensionManager.ts    # Extension lifecycle management
│   │   ├── rpcManager.ts          # RPC communication manager
│   │   ├── webViewManager.ts      # WebView management
│   │   └── config.ts              # Configuration handling
│   ├── typings/                   # VS Code type definitions
│   ├── bootstrap-*.ts             # Bootstrap files for different contexts
│   ├── package.json               # Node.js dependencies
│   ├── tsconfig.json              # TypeScript configuration
│   └── tsup.config.ts             # Build configuration
├── plugin/                        # IntelliJ Plugin (Kotlin/Java)
│   ├── src/main/kotlin/           # Kotlin source code
│   │   └── ai/kilocode/jetbrains/
│   │       ├── webview/           # WebView implementation
│   │       ├── events/            # Event system
│   │       ├── commands/          # Command handlers
│   │       ├── actors/            # Main thread actors (RPC)
│   │       ├── model/             # Data models
│   │       └── util/              # Utilities
│   ├── src/main/resources/        # Plugin resources
│   │   ├── META-INF/              # Plugin manifest
│   │   └── themes/                # UI themes
│   ├── build.gradle.kts           # Gradle build configuration
│   ├── gradle.properties          # Plugin version and settings
│   ├── genPlatform.gradle         # VS Code platform generation
│   ├── scripts/                   # Build scripts
│   └── .run/                      # Run configurations
├── resources/                     # Runtime resources (generated)
│   └── node_modules/              # Node.js dependencies for runtime
└── scripts/                       # Build and utility scripts
    └── check-dependencies.sh      # Dependency verification
```

## Architecture

The JetBrains plugin uses a **two-process architecture**:

### 1. Plugin Process (JVM)
**Technology**: Kotlin, Java, IntelliJ Platform SDK
**Responsibilities**:
- IDE integration (menus, toolbars, actions)
- WebView rendering (JCEF - Java Chromium Embedded Framework)
- RPC communication with Extension Host
- File system operations
- IDE API calls

### 2. Extension Host Process (Node.js)
**Technology**: TypeScript, Node.js
**Responsibilities**:
- Run VS Code extension code
- AI model communication
- Tool execution
- File operations
- Message handling

**Communication**: JSON-RPC over stdin/stdout between processes

## Prerequisites

### Required Software

#### Java Development Kit (JDK) 17
**Required**: Java 17 LTS
**Installation**:
```bash
# Using SDKMAN (recommended for macOS/Linux)
curl -s "https://get.sdkman.io" | bash
sdk install java 17.0.12-tem
sdk use java 17.0.12-tem

# Using Homebrew (macOS)
brew install openjdk@17
export JAVA_HOME=/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home

# Verify
java -version  # Should show Java 17.x.x
```

#### Node.js and pnpm
**Required**: Node.js 20.x
**Installation**:
```bash
# Using nvm
nvm install 20
nvm use 20

# Install pnpm
npm install -g pnpm
```

#### VS Code Submodule
**Required**: VS Code source code as git submodule
**Initialization**:
```bash
# From project root
git submodule update --init --recursive
```

### Dependency Check

**Automatic Check** (runs before builds):
```bash
# Integrated into build system
pnpm jetbrains:build
```

**Manual Check**:
```bash
./jetbrains/scripts/check-dependencies.sh
```

## Build Modes

The plugin supports three build modes via `debugMode` property:

### 1. Development Mode (`debugMode=idea`)
**Purpose**: Local development and debugging

**Features**:
- Creates `.env` file for Extension Host
- Copies theme resources to debug location
- Enables hot-reloading
- Uses local VS Code plugin integration

**Build Command**:
```bash
./gradlew prepareSandbox -PdebugMode=idea

# Or from project root
pnpm jetbrains:run
```

### 2. Release Mode (`debugMode=release`)
**Purpose**: Production builds

**Features**:
- Self-contained deployment package
- Includes all runtime dependencies
- Requires `platform.zip` file
- Optimized for distribution

**Build Command**:
```bash
# Generate platform files first
./gradlew genPlatform

# Build plugin
./gradlew buildPlugin -PdebugMode=release

# Or from project root
pnpm jetbrains:bundle
```

### 3. Lightweight Mode (`debugMode=none`)
**Purpose**: Testing and CI (default)

**Features**:
- Minimal resource preparation
- No VS Code runtime dependencies
- Fast builds
- Suitable for static analysis

**Build Command**:
```bash
./gradlew prepareSandbox

# Or
pnpm jetbrains:build
```

## Extension Host (host/)

### Main Components

#### `main.ts`
**Purpose**: Main entry point for Extension Host

**Responsibilities**:
- Initialize Node.js environment
- Set up RPC communication
- Start extension
- Handle lifecycle events

#### `extension.ts`
**Purpose**: Extension initialization and activation

**Responsibilities**:
- Load VS Code extension code
- Register commands and providers
- Set up webview
- Initialize services

#### `extensionManager.ts`
**Purpose**: Manage extension lifecycle

**Functions**:
- `activate()`: Activate extension
- `deactivate()`: Deactivate extension
- `reload()`: Reload extension
- `getExtensionContext()`: Get extension context

#### `rpcManager.ts`
**Purpose**: RPC communication with plugin

**Protocol**: JSON-RPC
**Transport**: stdin/stdout
**Message Types**:
- Commands
- Events
- Requests/Responses
- Notifications

#### `webViewManager.ts`
**Purpose**: Manage webview communication

**Responsibilities**:
- Handle webview messages
- Send messages to webview
- Manage webview state
- Coordinate with plugin process

#### `config.ts`
**Purpose**: Configuration management

**Features**:
- Load configuration from IDE
- Sync settings between processes
- Validate configuration
- Default values

### Bootstrap Files

**Purpose**: Different entry points for various contexts

**Files**:
- `bootstrap-node.ts`: Node.js context
- `bootstrap-window.ts`: Browser window context
- `bootstrap-fork.ts`: Forked process
- `bootstrap-cli.ts`: CLI context
- `bootstrap-server.ts`: Server context
- `bootstrap-esm.ts`: ES module context
- `bootstrap-import.ts`: Import context
- `bootstrap-meta.ts`: Meta context

### Build Configuration

#### `tsup.config.ts`
**Purpose**: TypeScript bundling configuration

**Features**:
- Bundle TypeScript to JavaScript
- Tree-shaking
- Minification
- Source maps

**Output**: `dist/` directory

#### `package.json`
**Scripts**:
- `build`: Build Extension Host
- `dev`: Development mode with watch
- `deps:check`: Check dependencies
- `deps:install`: Install runtime dependencies
- `deps:patch`: Apply dependency patches

## IntelliJ Plugin (plugin/)

### Main Components

#### Plugin Manifest (`src/main/resources/META-INF/plugin.xml`)
**Purpose**: Plugin configuration

**Defines**:
- Plugin ID and name
- Version and compatibility
- Dependencies
- Extensions and actions
- Services

#### `webview/`
**Purpose**: WebView implementation

**Key Files**:

##### `WebViewManager.kt`
**Purpose**: Manage JCEF webview

**Responsibilities**:
- Create and configure JCEF browser
- Load webview HTML
- Handle JavaScript bridge
- Manage webview lifecycle

**Features**:
- Custom scheme handlers
- Drag and drop support
- Context menu integration

##### `LocalResHandler.kt`
**Purpose**: Handle local resource requests

**Features**:
- Serve static files
- Handle custom protocols
- Resource caching

##### `DragDropHandler.kt`
**Purpose**: Handle drag and drop in webview

**Features**:
- File drag and drop
- Text drag and drop
- Image handling

#### `events/`
**Purpose**: Event system

**Key Files**:

##### `EventBus.kt`
**Purpose**: Event bus for plugin-wide events

**Pattern**: Publish-Subscribe

**Events**:
- Workspace changes
- File operations
- Settings updates
- UI events

##### `WorkspaceEvents.kt`
**Purpose**: Workspace-specific events

**Events**:
- Workspace opened
- Workspace closed
- Files added/removed
- Project structure changes

##### `WebviewEvents.kt`
**Purpose**: Webview-related events

**Events**:
- Webview created
- Webview destroyed
- Message received
- State changed

#### `commands/`
**Purpose**: Command handlers

**Key Files**:

##### `Commands.kt`
**Purpose**: Define available commands

**Commands**:
- Open Kilo Code
- New task
- Settings
- Help

##### `KiloCodeAuthProtocolCommand.kt`
**Purpose**: Handle authentication protocol

**Features**:
- OAuth flow
- Token management
- URL scheme handling

#### `actors/`
**Purpose**: Main thread actors for RPC

**Pattern**: Actor model for thread-safe RPC

**Key Actors**:

##### `MainThreadCommandsShape.kt`
**Purpose**: Command execution actor

**Methods**:
- `executeCommand()`
- `getCommands()`
- `registerCommand()`

##### `MainThreadWebviewViewsShape.kt`
**Purpose**: Webview management actor

**Methods**:
- `createWebviewPanel()`
- `postMessage()`
- `setHtml()`

##### `MainThreadLanguageFeaturesShape.kt`
**Purpose**: Language features actor

**Methods**:
- `registerCompletionProvider()`
- `registerCodeActionProvider()`
- `registerDefinitionProvider()`

##### `MainThreadTaskShape.kt`
**Purpose**: Task management actor

**Methods**:
- `registerTaskProvider()`
- `executeTask()`

##### `MainThreadOutputServiceShape.kt`
**Purpose**: Output service actor

**Methods**:
- `createOutputChannel()`
- `appendLine()`
- `show()`

##### `MainThreadDebugServiceShape.kt`
**Purpose**: Debug service actor

**Methods**:
- `startDebugging()`
- `stopDebugging()`

##### `MainThreadSecretStateShape.kt`
**Purpose**: Secret storage actor

**Methods**:
- `store()`
- `get()`
- `delete()`

##### `MainThreadLanguageModelToolsShape.kt`
**Purpose**: Language model tools actor

**Methods**:
- `registerTool()`
- `invokeTool()`

#### `model/`
**Purpose**: Data models

##### `WorkspaceData.kt`
**Purpose**: Workspace information model

**Fields**:
- `workspaceFolders`: List of workspace folders
- `name`: Workspace name
- `uri`: Workspace URI

#### `util/`
**Purpose**: Utility functions

**Key Utils**:
- Reflection utilities
- String manipulation
- File operations
- JSON serialization

### Build Configuration

#### `build.gradle.kts`
**Purpose**: Gradle build configuration

**Key Tasks**:
- `prepareSandbox`: Prepare plugin sandbox
- `buildPlugin`: Build plugin JAR
- `runIde`: Run IDE with plugin
- `publishPlugin`: Publish to JetBrains Marketplace

**Dependencies**:
- IntelliJ Platform SDK
- Kotlin standard library
- JCEF (Java Chromium Embedded Framework)

#### `gradle.properties`
**Purpose**: Plugin version and settings

**Properties**:
- `pluginVersion`: Plugin version
- `platformVersion`: Target IDE version
- `pluginSinceBuild`: Minimum IDE build
- `pluginUntilBuild`: Maximum IDE build

#### `genPlatform.gradle`
**Purpose**: Generate VS Code platform files

**What it does**:
1. Downloads VS Code
2. Extracts platform-specific binaries
3. Creates `platform.zip` with:
   - Node.js runtime
   - Native modules
   - VS Code dependencies

**Usage**:
```bash
./gradlew genPlatform
```

### Resources

#### `themes/`
**Purpose**: UI themes for webview

**Themes**:
- Dark theme
- Light theme
- High contrast themes

**Format**: CSS files

#### `META-INF/`
**Purpose**: Plugin metadata

**Files**:
- `plugin.xml`: Plugin configuration
- `pluginIcon.svg`: Plugin icon

## Building the Plugin

### Development Build

**From project root**:
```bash
pnpm jetbrains:run
```

**Manual**:
```bash
cd jetbrains/plugin
./gradlew runIde -PdebugMode=idea
```

**Features**:
- Automatic IDE launch
- Hot reload enabled
- Debug logging
- Development webview

### Production Build

**From project root**:
```bash
# Generate platform files
pnpm jetbrains:bundle

# Or manually
cd jetbrains/plugin
./gradlew genPlatform
./gradlew buildPlugin -PdebugMode=release
```

**Output**: `jetbrains/plugin/build/distributions/*.zip`

### Extension Host Only

**Build**:
```bash
cd jetbrains/host
pnpm build
```

**Output**: `jetbrains/host/dist/`

## Common Issues and Solutions

### Java Version Errors

**Problem**: "Unsupported class file major version 68"
**Cause**: Using Java 24+ instead of Java 17

**Solution**:
```bash
# Using SDKMAN
sdk install java 17.0.12-tem
sdk use java 17.0.12-tem

# Verify
java -version  # Should show 17.x.x
```

### Native Module Architecture Mismatch

**Problem**: "slice is not valid mach-o file"
**Cause**: Native modules compiled for wrong architecture

**Solution**:
```bash
cd jetbrains/resources
rm -rf node_modules package-lock.json
cp ../host/package.json .
npm install

# Verify architecture
file node_modules/@vscode/spdlog/build/Release/spdlog.node
```

### Missing platform.zip

**Problem**: Release build fails
**Cause**: Platform files not generated

**Solution**:
```bash
cd jetbrains/plugin
./gradlew genPlatform
```

### VS Code Submodule Not Initialized

**Problem**: Missing VS Code dependencies
**Solution**:
```bash
git submodule update --init --recursive
```

### Gradle Build Hangs

**Problem**: Gradle tasks hang or fail
**Solution**:
```bash
./gradlew --stop
./gradlew clean
./gradlew build --refresh-dependencies
```

## Development Workflow

### 1. Initial Setup

**Check dependencies**:
```bash
./jetbrains/scripts/check-dependencies.sh
```

**Install dependencies**:
```bash
pnpm install
```

### 2. Development

**Start IDE with plugin**:
```bash
pnpm jetbrains:run
```

**Make changes**:
- Edit Kotlin files in `plugin/src/`
- Edit TypeScript files in `host/src/`
- Rebuild as needed

**Test changes**:
- Plugin reloads automatically in IDE
- Extension Host requires restart

### 3. Testing

**Unit tests** (Kotlin):
```bash
cd jetbrains/plugin
./gradlew test
```

**Integration tests**:
- Manual testing in IDE
- E2E scenarios

### 4. Release

**Build plugin**:
```bash
pnpm jetbrains:bundle
```

**Upload to marketplace**:
```bash
cd jetbrains/plugin
./gradlew publishPlugin
```

## Platform Support

**Supported IDEs**:
- IntelliJ IDEA
- PyCharm
- WebStorm
- PhpStorm
- RubyMine
- GoLand
- CLion
- Rider
- Android Studio
- Other JetBrains IDEs

**Supported OS**:
- Windows (x64)
- macOS (x64, ARM64)
- Linux (x64)

**Architecture Handling**:
- Automatic architecture detection
- Platform-specific native modules
- Runtime loaders for correct binaries

## Environment Variables

**Build Variables**:
- `JAVA_HOME`: Java installation directory
- `debugMode`: Build mode (idea/release/none)
- `vscodePlugin`: Plugin name (default: kilocode)
- `vscodeVersion`: VS Code version (default: 1.100.0)

**Runtime Variables**:
- `KILOCODE_HOST_PATH`: Path to Extension Host
- `KILOCODE_LOG_LEVEL`: Logging level (debug/info/warn/error)

## Turbo Integration

**Turbo Tasks**:
- `jetbrains:bundle`: Complete bundle build
- `jetbrains:run-bundle`: Run with bundle mode
- `jetbrains:run`: Run in development mode
- `jetbrains:build`: Build plugin only

**Benefits**:
- Build caching
- Parallel builds
- Dependency tracking
- Incremental builds

## Contributing

**Guidelines**:
1. Check dependencies before starting
2. Test in development mode first
3. Verify all three build modes work
4. Update README for new dependencies
5. Follow Kotlin and TypeScript style guides
6. Write unit tests for new features
7. Test on all supported platforms

**Code Style**:
- Kotlin: JetBrains style
- TypeScript: Standard TypeScript style
- Indentation: Tabs for Kotlin, spaces for TypeScript

---

*Last updated: 2025-10-21*
