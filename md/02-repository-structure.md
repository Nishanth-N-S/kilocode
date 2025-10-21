# Repository Structure Documentation

This document provides a comprehensive overview of the Kilo Code repository structure, explaining the purpose and contents of each major directory.

## Directory Tree Overview

```
kilocode/
├── .changeset/              # Changeset files for versioning
├── .devcontainer/           # VS Code dev container configuration
├── .github/                 # GitHub Actions, templates, and workflows
├── .husky/                  # Git hooks configuration
├── .kilocode/              # Kilo-specific configurations and rules
├── .vscode/                # VS Code workspace settings
├── apps/                   # Supporting applications
│   ├── kilocode-docs/      # Documentation website (Docusaurus)
│   ├── playwright-e2e/     # Playwright end-to-end tests
│   ├── storybook/          # Component development environment
│   ├── vscode-e2e/         # VS Code extension E2E tests
│   ├── vscode-nightly/     # Nightly build configuration
│   ├── web-evals/          # Web-based evaluation tools
│   └── web-roo-code/       # Web version of Roo Code
├── benchmark/              # Performance benchmarking tools
├── cli/                    # Command-line interface application
├── deps/                   # External dependencies (VS Code fork)
├── jetbrains/              # JetBrains IDE plugin
│   ├── host/               # Host application for JetBrains
│   ├── plugin/             # JetBrains plugin code
│   └── scripts/            # Build scripts for JetBrains
├── launch/                 # Launch configurations
├── md/                     # This documentation directory
├── packages/               # Shared packages and libraries
│   ├── build/              # Build utilities
│   ├── cloud/              # Cloud service integration
│   ├── config-eslint/      # Shared ESLint configuration
│   ├── config-typescript/  # Shared TypeScript configuration
│   ├── evals/              # Evaluation framework
│   ├── ipc/                # Inter-process communication
│   ├── telemetry/          # Telemetry service
│   └── types/              # Shared TypeScript types
├── releases/               # Release artifacts and binaries
├── scripts/                # Build and utility scripts
├── src/                    # Core VS Code extension source code
│   ├── activate/           # Extension activation logic
│   ├── api/                # API providers and transformations
│   ├── assets/             # Static assets (icons, images, docs)
│   ├── core/               # Core functionality
│   ├── extension/          # Extension entry points
│   ├── i18n/               # Internationalization
│   ├── integrations/       # VS Code integrations
│   ├── services/           # Service implementations
│   ├── shared/             # Shared utilities
│   ├── utils/              # Utility functions
│   ├── walkthrough/        # VS Code walkthrough content
│   └── workers/            # Web workers
└── webview-ui/             # React-based user interface
    ├── audio/              # Audio assets
    ├── public/             # Public assets
    └── src/                # React components and logic
```

## Root Level Directories

### `.changeset/`
**Purpose**: Version management using Changesets

**Contains**:
- Individual changeset files for tracking changes
- Configuration for changelog generation
- Version bump specifications

**Usage**: Developers create changeset files when making changes that should be included in release notes.

### `.devcontainer/`
**Purpose**: VS Code development container configuration

**Contains**:
- `devcontainer.json`: Container specification
- Dockerfile or image reference
- Post-create commands

**Benefits**: Provides consistent development environment across different machines.

### `.github/`
**Purpose**: GitHub-specific configurations

**Contains**:
- `workflows/`: CI/CD GitHub Actions
  - Build workflows
  - Test workflows
  - Release workflows
  - Deployment workflows
- `ISSUE_TEMPLATE/`: Issue templates
- `PULL_REQUEST_TEMPLATE/`: PR templates
- `copilot-instructions.md`: GitHub Copilot configuration

**Key Workflows**:
- Continuous Integration (build, test, lint)
- Automated releases
- Dependency updates
- Code quality checks

### `.husky/`
**Purpose**: Git hooks for automated checks

**Contains**:
- `pre-commit`: Runs linting and formatting
- `pre-push`: Runs type checking and changeset verification

**Benefits**: Ensures code quality before commits/pushes.

### `.kilocode/`
**Purpose**: Kilo Code specific configurations

**Contains**:
- `rules/`: Custom rules for code generation
- `rules-translate/`: Internationalization rules
- `workflows/`: Custom workflow definitions

### `.vscode/`
**Purpose**: VS Code workspace settings

**Contains**:
- `settings.json`: Editor settings
- `launch.json`: Debug configurations
- `tasks.json`: Build tasks
- `extensions.json`: Recommended extensions

## Main Application Directories

### `apps/`
**Purpose**: Supporting applications and tools

#### `apps/kilocode-docs/`
**Type**: Docusaurus documentation website
**Purpose**: User-facing documentation
**Tech Stack**: React, Docusaurus, MDX
**Contains**:
- Documentation pages
- Blog posts
- API references
- Tutorials

#### `apps/playwright-e2e/`
**Type**: End-to-end testing
**Purpose**: Browser automation tests
**Tech Stack**: Playwright
**Contains**:
- Test scenarios
- Test helpers
- Page objects

#### `apps/storybook/`
**Type**: Component development environment
**Purpose**: UI component documentation and testing
**Tech Stack**: Storybook, React
**Contains**:
- Component stories
- Visual regression tests
- Component documentation

#### `apps/vscode-e2e/`
**Type**: VS Code extension testing
**Purpose**: Integration tests for VS Code extension
**Tech Stack**: VS Code Test Runner
**Contains**:
- Extension test suites
- Test fixtures

#### `apps/vscode-nightly/`
**Type**: Build configuration
**Purpose**: Nightly build packaging
**Contains**: Configuration for automated nightly builds

#### `apps/web-evals/`
**Type**: Evaluation tools
**Purpose**: AI model evaluation and testing
**Tech Stack**: React, Docker
**Contains**:
- Evaluation UI
- Test cases
- Result visualization

#### `apps/web-roo-code/`
**Type**: Web application
**Purpose**: Web-based version of Roo Code
**Tech Stack**: React
**Contains**:
- Web UI
- API integration

### `benchmark/`
**Purpose**: Performance benchmarking

**Contains**:
- Benchmark scripts
- Performance test cases
- Result analysis tools

**Use Cases**:
- Measure extension startup time
- Test API response times
- Compare performance across versions

### `cli/`
**Purpose**: Command-line interface application

**Technology**: Node.js, TypeScript
**Build**: Bundled as standalone executable

**Structure**:
```
cli/
├── docs/           # CLI documentation
├── src/            # Source code
│   ├── commands/   # CLI commands
│   ├── config/     # Configuration
│   ├── services/   # Services
│   ├── ui/         # UI components (prompts, etc.)
│   └── utils/      # Utilities
└── package.json    # Dependencies
```

**Commands**: Code generation, task automation, file management

### `deps/`
**Purpose**: External dependencies and patches

**Contains**:
- `vscode/`: Forked VS Code dependency
- `patches/`: Patch files for dependencies

**Why**: Custom modifications to VS Code or other dependencies

### `jetbrains/`
**Purpose**: JetBrains IDE integration

**Structure**:
```
jetbrains/
├── host/           # Host application (Node.js)
├── plugin/         # IntelliJ plugin (Kotlin/Java)
└── scripts/        # Build scripts
```

**Technology**: Kotlin, Java, Gradle for plugin; Node.js for host

**Supported IDEs**:
- IntelliJ IDEA
- PyCharm
- WebStorm
- Other JetBrains IDEs

### `packages/`
**Purpose**: Shared libraries and utilities

#### `packages/build/`
**Purpose**: Build utilities and helpers
**Exports**: Build scripts, bundling utilities

#### `packages/cloud/`
**Purpose**: Cloud service integration
**Exports**: Authentication, API communication, user management

#### `packages/config-eslint/`
**Purpose**: Shared ESLint configuration
**Exports**: ESLint rules and plugins

#### `packages/config-typescript/`
**Purpose**: Shared TypeScript configuration
**Exports**: tsconfig.json presets

#### `packages/evals/`
**Purpose**: Evaluation framework
**Exports**: Testing harness, evaluation metrics

#### `packages/ipc/`
**Purpose**: Inter-process communication
**Exports**: IPC protocol, message handling

#### `packages/telemetry/`
**Purpose**: Telemetry and analytics
**Exports**: Event tracking, usage metrics

#### `packages/types/`
**Purpose**: Shared TypeScript type definitions
**Exports**: Common types used across packages

### `releases/`
**Purpose**: Release artifacts

**Contains**:
- Built VSIX files
- Release notes
- Binaries

### `scripts/`
**Purpose**: Build and utility scripts

**Contains**:
- Build scripts
- Deployment scripts
- Utility scripts
- Kilocode-specific scripts in `scripts/kilocode/`

**Examples**:
- Bootstrap script
- Install VSIX script
- Update contributors script

### `src/`
**Purpose**: Core VS Code extension source code

**See**: [03-src-directory.md](./03-src-directory.md) for detailed documentation

**High-Level Structure**:
- Extension activation and lifecycle
- Core AI agent functionality
- VS Code integrations
- API providers and transformations
- Service implementations

### `webview-ui/`
**Purpose**: React-based user interface

**See**: [04-webview-ui.md](./04-webview-ui.md) for detailed documentation

**Technology**: React, TypeScript, Vite
**Build Output**: Bundled into VS Code extension

## Build Outputs (Git-Ignored)

### `node_modules/`
**Purpose**: Installed npm dependencies
**Created by**: `pnpm install`

### `dist/`
**Purpose**: Compiled and bundled code
**Created by**: Build scripts

### `out/`
**Purpose**: TypeScript compilation output
**Created by**: `tsc`

### `bin/`
**Purpose**: Built VSIX files
**Created by**: `pnpm vsix`

### `.turbo/`
**Purpose**: Turbo cache
**Created by**: Turbo build system

## Key Characteristics

### Monorepo Structure
- Uses **pnpm workspaces** for package management
- Uses **Turbo** for build orchestration
- Multiple packages can be developed and tested together

### TypeScript Configuration
- Composite projects for incremental builds
- Shared configurations via packages
- Strict type checking enabled

### Build System
- **Turbo**: Parallel builds, caching
- **esbuild**: Fast bundling
- **TypeScript**: Compilation
- **Vite**: Webview UI bundling

### Testing Strategy
- Unit tests: Vitest
- Integration tests: VS Code Test Runner
- E2E tests: Playwright
- Component tests: Storybook

## Navigation Tips

1. **Start with `src/extension.ts`**: Main extension entry point
2. **Explore `src/core/`**: Core AI agent functionality
3. **Check `webview-ui/src/`**: UI components
4. **Review `packages/`**: Shared utilities
5. **Look at `apps/`**: Supporting tools

---

*Last updated: 2025-10-21*
