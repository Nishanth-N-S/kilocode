# Kilo Code Repository Documentation

Welcome to the comprehensive documentation for the Kilo Code repository. This documentation covers every file, directory, and major component of the codebase.

## Table of Contents

1. [Overview](#overview)
2. [Documentation Index](#documentation-index)
3. [Quick Start](#quick-start)
4. [Repository Structure](#repository-structure)

## Overview

Kilo Code is an open-source VS Code AI agent that helps developers generate code from natural language, automate tasks, and improve productivity. This repository contains:

- **VS Code Extension**: The main extension code (src/)
- **Webview UI**: React-based user interface (webview-ui/)
- **CLI Application**: Command-line interface (cli/)
- **JetBrains Plugin**: IntelliJ IDEA and other JetBrains IDEs support (jetbrains/)
- **Packages**: Shared libraries and tools (packages/)
- **Apps**: Supporting applications including documentation, testing, and evaluation tools (apps/)
- **Scripts**: Build and utility scripts (scripts/)

## Documentation Index

### Core Documentation
- [Root Level Files](./01-root-level-files.md) - Configuration files, package.json, etc.
- [Repository Structure](./02-repository-structure.md) - Complete directory tree and organization

### Main Components
- [Source Directory (src/)](./03-src-directory.md) - Core extension code
- [Webview UI (webview-ui/)](./04-webview-ui.md) - Frontend React application
- [CLI Application (cli/)](./05-cli-application.md) - Command-line interface
- [JetBrains Plugin (jetbrains/)](./06-jetbrains-plugin.md) - IntelliJ IDEA integration

### Packages
- [Packages Directory](./07-packages.md) - Shared libraries and utilities

### Applications
- [Apps Directory](./08-apps.md) - Supporting applications (docs, tests, evals)

### Development
- [Scripts Directory](./09-scripts.md) - Build and utility scripts
- [Development Guide](./10-development-guide.md) - Setting up and developing
- [Testing Guide](./11-testing-guide.md) - Testing strategies and frameworks
- [Build Process](./12-build-process.md) - Building and packaging

### Reference
- [File Reference](./13-file-reference.md) - Complete alphabetical file listing
- [Configuration Files](./14-configuration-files.md) - All config files explained
- [Dependencies](./15-dependencies.md) - Package dependencies and versions

## Quick Start

For first-time developers:

1. Read [Root Level Files](./01-root-level-files.md) to understand the project setup
2. Review [Repository Structure](./02-repository-structure.md) for an overview
3. Follow [Development Guide](./10-development-guide.md) to set up your environment
4. Explore [Source Directory](./03-src-directory.md) to understand the core code

## Repository Structure

```
kilocode/
├── src/                    # Core VS Code extension code
├── webview-ui/            # React-based UI
├── cli/                   # Command-line interface
├── jetbrains/             # JetBrains IDE plugin
├── packages/              # Shared libraries
├── apps/                  # Supporting applications
├── scripts/               # Build and utility scripts
├── deps/                  # External dependencies
├── benchmark/             # Performance benchmarks
├── releases/              # Release artifacts
└── md/                    # This documentation
```

## Additional Resources

- [DEVELOPMENT.md](../DEVELOPMENT.md) - Official development guide
- [README.md](../README.md) - Main repository README
- [CONTRIBUTING.md](../CONTRIBUTING.md) - Contribution guidelines
- [Official Documentation](https://kilocode.ai/docs) - User-facing documentation

---

*Last updated: 2025-10-21*
*Documentation version: 1.0*
