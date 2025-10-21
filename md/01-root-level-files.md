# Root Level Files Documentation

This document provides detailed information about all configuration and important files at the root level of the Kilo Code repository.

## Configuration Files

### Package Management

#### `package.json`
**Purpose**: Root workspace configuration for the monorepo

**Key Properties**:
- **Name**: `kilo-code`
- **Package Manager**: `pnpm@10.8.1`
- **Node Engine**: `20.19.2`

**Important Scripts**:
- `install` / `install:all`: Bootstrap the monorepo
- `build`: Build production VSIX package
- `vsix`: Create VS Code extension package
- `vsix:nightly`: Create nightly build
- `lint`: Run linting across all packages
- `test`: Run tests across all packages
- `format`: Format code using Prettier
- `cli:build`: Build CLI application
- `cli:bundle`: Bundle CLI as standalone
- `jetbrains:build`: Build JetBrains plugin
- `docs:start`: Start documentation server
- `docs:build`: Build documentation site

**Dev Dependencies**:
- TypeScript, ESLint, Prettier for code quality
- Turbo for monorepo build orchestration
- Changesets for version management
- esbuild for bundling

#### `pnpm-workspace.yaml`
**Purpose**: Defines workspace packages for pnpm

**Workspace Packages**:
- `apps/*` - All applications
- `packages/*` - Shared libraries
- `src` - Main extension
- `webview-ui` - UI package
- `cli` - CLI package
- `jetbrains/host` - JetBrains host
- `jetbrains/plugin` - JetBrains plugin

#### `pnpm-lock.yaml`
**Purpose**: Lock file for exact dependency versions
**Size**: Large file containing all resolved dependencies

### Build Configuration

#### `turbo.json`
**Purpose**: Turbo (monorepo build tool) configuration

**Defines**:
- Pipeline for building, testing, and bundling
- Task dependencies and caching strategies
- Output configurations

#### `tsconfig.json`
**Purpose**: TypeScript configuration for the workspace

**Key Settings**:
- References to all TypeScript packages
- Composite project setup for faster builds

### Version Control

#### `.gitignore`
**Purpose**: Specifies files and directories to exclude from version control

**Ignored**:
- `node_modules/`
- Build artifacts (`dist/`, `out/`, `bin/`)
- IDE files (`.vscode/`, `.idea/`)
- Environment files (`.env*`)
- OS files (`.DS_Store`)
- Temporary files

#### `.gitattributes`
**Purpose**: Git attributes for file handling
- Line ending normalization
- Binary file handling
- Merge strategies

#### `.gitmodules`
**Purpose**: Git submodules configuration
- References to external dependencies (e.g., VS Code)

#### `.git-blame-ignore-revs`
**Purpose**: Commits to ignore in git blame
- Formatting changes
- Large refactors

### Code Quality

#### `.prettierrc.json`
**Purpose**: Prettier code formatter configuration

**Settings**:
- Print width, tab width
- Semicolons, quotes, trailing commas
- Consistent formatting rules

#### `.prettierignore`
**Purpose**: Files/directories to exclude from Prettier formatting
- Build outputs
- Generated files
- Third-party code

#### `eslint.config.mjs`
**Purpose**: ESLint configuration (flat config format)
- Linting rules
- TypeScript integration
- Import/export rules

#### `knip.json`
**Purpose**: Knip configuration for finding unused files and dependencies
- Entry points
- Project files
- Ignore patterns

### Development Environment

#### `.nvmrc`
**Purpose**: Specifies Node.js version
**Version**: `20.19.2`

#### `.tool-versions`
**Purpose**: asdf version manager configuration
- Node.js version specification

#### `flake.nix` and `flake.lock`
**Purpose**: Nix flake for reproducible development environment
**Provides**:
- Node.js 20
- pnpm via corepack
- Development dependencies

#### `.envrc`
**Purpose**: direnv configuration for automatic environment setup
- Loads Nix flake environment

#### `.env.sample`
**Purpose**: Sample environment variables
- API keys templates
- Configuration examples

### Docker

#### `.dockerignore`
**Purpose**: Files to exclude from Docker context
- Similar to `.gitignore` but for Docker builds

### IDE Configuration

#### `.vscode/`
**Directory containing VS Code workspace settings**:
- `settings.json`: Editor settings
- `launch.json`: Debug configurations
- `tasks.json`: Task definitions
- `extensions.json`: Recommended extensions

#### `.vscodeignore`
**Purpose**: Files to exclude from VSIX package
- Source files
- Test files
- Development-only dependencies

### Documentation

#### `README.md`
**Purpose**: Main repository README
**Contains**:
- Project overview
- Quick start guide
- Key features
- Installation instructions
- Links to resources

#### `DEVELOPMENT.md`
**Purpose**: Development setup guide
**Contains**:
- Prerequisites
- Installation instructions
- Development workflow
- Testing guide
- Contribution guidelines

#### `CONTRIBUTING.md`
**Purpose**: Contribution guidelines
**Contains**:
- How to contribute
- Code of conduct reference
- Pull request process
- Issue reporting

#### `CODE_OF_CONDUCT.md`
**Purpose**: Community code of conduct
**Defines**:
- Expected behavior
- Unacceptable behavior
- Enforcement

#### `CHANGELOG.md`
**Purpose**: Version history and changes
**Format**: Keep a Changelog format
**Contains**: All notable changes for each version

#### `LICENSE`
**Purpose**: Software license
**Type**: Apache 2.0 License

#### `NOTICE`
**Purpose**: Legal notices and attributions

#### `PRIVACY.md`
**Purpose**: Privacy policy
**Contains**: Data collection and usage policies

### CI/CD and Automation

#### `.github/`
**Directory containing GitHub-specific files**:
- `workflows/`: GitHub Actions CI/CD pipelines
- `ISSUE_TEMPLATE/`: Issue templates
- `PULL_REQUEST_TEMPLATE/`: PR templates
- `copilot-instructions.md`: GitHub Copilot instructions

#### `.husky/`
**Directory containing Git hooks**:
- `pre-commit`: Runs before commit (linting, type checking)
- `pre-push`: Runs before push (compilation, changeset check)

#### `.changeset/`
**Directory for changeset files**:
- Version management
- Changelog generation

### Deployment

#### `renovate.json`
**Purpose**: Renovate bot configuration for dependency updates
**Settings**:
- Update schedules
- Automerge rules
- Package grouping

#### `ellipsis.yaml`
**Purpose**: Ellipsis CI configuration

### Build Artifacts

#### `releases/`
**Directory**: Contains release artifacts and binaries

#### `bin/` (generated)
**Directory**: Build output for VSIX files

#### `dist/` (generated)
**Directory**: Compiled/bundled code

#### `out/` (generated)
**Directory**: TypeScript compilation output

### Assets

#### `kilo.gif`
**Purpose**: Demo GIF shown in README
**Type**: Animated demonstration

### Custom Configuration

#### `.kilocode/`
**Directory containing Kilo-specific configurations**:
- Custom rules
- Workflows
- Mode definitions

#### `.kilocodemodes`
**Purpose**: Custom mode definitions for Kilo Code

#### `.rooignore`
**Purpose**: Files to ignore for Roo Code processing

### DevContainer

#### `.devcontainer/`
**Directory**: VS Code dev container configuration
- `devcontainer.json`: Container setup
- Docker configuration for development environment

### Launch Configuration

#### `launch/`
**Directory**: Launch scripts and configurations

## File Organization Best Practices

1. **Root level** should contain only essential configuration files
2. **Source code** belongs in `src/`, `webview-ui/`, `cli/`, etc.
3. **Documentation** goes in `md/` or top-level markdown files
4. **Build artifacts** are git-ignored but may exist locally
5. **Configuration** files use standard formats (JSON, YAML, TOML)

## Maintenance Notes

- Keep configuration files in sync with documentation
- Update `.nvmrc` when changing Node.js version
- Review and update `.gitignore` when adding new build outputs
- Keep `package.json` scripts documented and organized
- Regularly update dependencies through Renovate

---

*Last updated: 2025-10-21*
