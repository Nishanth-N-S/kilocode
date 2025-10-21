# Build and Deployment Guide

This guide covers the build process, packaging, and deployment for Kilo Code.

## Table of Contents

1. [Build System Overview](#build-system-overview)
2. [Build Process](#build-process)
3. [Packaging](#packaging)
4. [Deployment](#deployment)
5. [CI/CD Pipeline](#cicd-pipeline)
6. [Release Process](#release-process)

## Build System Overview

### Technologies

**Monorepo Management**:
- **pnpm**: Package manager with workspace support
- **Turbo**: Build system for monorepos

**Bundlers**:
- **esbuild**: Fast JavaScript bundler (extension)
- **Vite**: Build tool (webview UI)
- **tsup**: TypeScript bundler (packages)
- **Gradle**: Build tool (JetBrains plugin)

**Compilers**:
- **TypeScript**: Type-safe JavaScript
- **Kotlin**: JetBrains plugin

### Build Architecture

```
┌─────────────┐
│   Source    │
│   Code      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  TypeScript │
│  Compiler   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Bundler   │
│  (esbuild)  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Package   │
│   (VSIX)    │
└─────────────┘
```

## Build Process

### Full Build

**Command**:
```bash
pnpm build
```

**What it does**:
1. Clean previous builds
2. Build packages (`packages/*`)
3. Build webview UI (`webview-ui/`)
4. Build extension (`src/`)
5. Create VSIX package

**Output**:
- `bin/kilo-code-{version}.vsix`

### Incremental Build

**With Turbo Cache**:
```bash
turbo build
```

**Benefits**:
- Only rebuilds changed packages
- Uses cache for unchanged packages
- Parallel builds
- Faster iteration

### Package-Specific Builds

**Extension Only**:
```bash
pnpm --filter kilo-code build
```

**Webview UI Only**:
```bash
pnpm --filter @roo-code/vscode-webview build
```

**CLI Only**:
```bash
pnpm cli:build
```

**JetBrains Plugin**:
```bash
pnpm jetbrains:build
```

## Build Modes

### Development Mode

**Environment**: `NODE_ENV=development`

**Features**:
- Source maps enabled
- Hot module replacement
- Verbose logging
- Debug symbols
- No minification

**Build**:
```bash
NODE_ENV=development pnpm build
```

### Production Mode

**Environment**: `NODE_ENV=production`

**Features**:
- Minified code
- Tree shaking
- Dead code elimination
- Optimized bundles
- No source maps (optional)

**Build**:
```bash
NODE_ENV=production pnpm build
# or
pnpm vsix:production
```

### Nightly Mode

**Purpose**: Pre-release testing

**Features**:
- Latest features
- Preview flag
- Separate version number
- Separate marketplace entry

**Build**:
```bash
pnpm bundle:nightly
pnpm vsix:nightly
```

## Build Configuration

### TypeScript (`tsconfig.json`)

**Base Configuration**:
```json
{
  "extends": "@roo-code/config-typescript/base.json",
  "compilerOptions": {
    "outDir": "out",
    "rootDir": "src",
    "composite": true,
    "sourceMap": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "out", "dist"]
}
```

**Project References**:
```json
{
  "references": [
    { "path": "./webview-ui" },
    { "path": "./packages/types" },
    { "path": "./packages/cloud" }
  ]
}
```

### esbuild (`src/esbuild.mjs`)

**Extension Build**:
```javascript
import * as esbuild from 'esbuild'

const ctx = await esbuild.context({
  entryPoints: ['src/extension.ts'],
  bundle: true,
  outfile: 'dist/extension.js',
  external: ['vscode'],
  format: 'cjs',
  platform: 'node',
  target: 'node20',
  sourcemap: true,
  minify: process.env.NODE_ENV === 'production',
})

await ctx.rebuild()
await ctx.dispose()
```

### Vite (`webview-ui/vite.config.ts`)

**Webview Build**:
```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: '../src/webview-ui',
    emptyOutDir: true,
    rollupOptions: {
      output: {
        entryFileNames: 'assets/[name].js',
        chunkFileNames: 'assets/[name].js',
        assetFileNames: 'assets/[name].[ext]',
      },
    },
  },
})
```

### Turbo (`turbo.json`)

**Pipeline Configuration**:
```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", "out/**", "build/**"]
    },
    "bundle": {
      "dependsOn": ["build"],
      "outputs": ["dist/**"]
    },
    "vsix": {
      "dependsOn": ["bundle"],
      "outputs": ["bin/**"]
    },
    "lint": {},
    "test": {
      "dependsOn": ["build"]
    }
  }
}
```

## Packaging

### VSIX Package

**What is VSIX**:
- VS Code Extension Package
- ZIP file with `.vsix` extension
- Contains all extension files

**Create VSIX**:
```bash
pnpm vsix
```

**Output**: `bin/kilo-code-{version}.vsix`

**Contents**:
```
kilo-code-0.1.0.vsix
├── extension.js           # Bundled extension code
├── package.json           # Extension manifest
├── webview-ui/            # Webview assets
│   ├── index.html
│   ├── assets/
│   │   ├── index.js
│   │   └── index.css
├── assets/                # Extension assets
│   ├── icons/
│   └── images/
└── node_modules/          # Runtime dependencies (if any)
```

### VSIX Configuration

**Package.json Fields**:
```json
{
  "name": "kilo-code",
  "displayName": "Kilo Code",
  "version": "0.1.0",
  "publisher": "kilocode",
  "engines": {
    "vscode": "^1.95.0"
  },
  "categories": ["Other"],
  "icon": "assets/icons/icon.png",
  "repository": {
    "type": "git",
    "url": "https://github.com/Kilo-Org/kilocode.git"
  }
}
```

**.vscodeignore**:
```
src/**
webview-ui/src/**
node_modules/**
.vscode/**
.github/**
*.md
tsconfig.json
.eslintrc.js
```

### CLI Packaging

**Standalone Binary**:
```bash
pnpm cli:bundle
```

**What it does**:
1. Bundle with esbuild
2. Create standalone executable
3. Include Node.js runtime
4. Platform-specific builds

**Output**:
- `cli/dist/kilocode-linux-x64`
- `cli/dist/kilocode-macos-x64`
- `cli/dist/kilocode-macos-arm64`
- `cli/dist/kilocode-win-x64.exe`

### JetBrains Plugin Packaging

**Build Plugin**:
```bash
cd jetbrains/plugin
./gradlew buildPlugin
```

**Output**: `jetbrains/plugin/build/distributions/kilo-code-{version}.zip`

**Contents**:
- Plugin JAR
- Extension Host bundle
- Node.js runtime
- Native modules
- Platform-specific binaries

## Deployment

### VS Code Marketplace

**Prerequisites**:
- Publisher account
- Personal Access Token (PAT)

**Publish**:
```bash
# Using vsce
vsce publish

# Using ovsx (Open VSX)
ovsx publish
```

**Environment Variables**:
```bash
export VSCE_PAT="your-pat-token"
```

**Automated Publishing**:
```yaml
# .github/workflows/publish.yml
- name: Publish to Marketplace
  run: vsce publish
  env:
    VSCE_PAT: ${{ secrets.VSCE_PAT }}
```

### JetBrains Marketplace

**Publish**:
```bash
cd jetbrains/plugin
./gradlew publishPlugin
```

**Configuration** (`gradle.properties`):
```properties
jetbrainsToken=your-token
```

### npm (CLI)

**Publish**:
```bash
cd cli
npm publish
```

**Prerequisites**:
- npm account
- Authentication token

### Documentation Site

**Build**:
```bash
pnpm docs:build
```

**Deploy** (e.g., to Vercel):
```bash
cd apps/kilocode-docs
vercel --prod
```

## CI/CD Pipeline

### GitHub Actions

**Workflow Structure**:
```
.github/workflows/
├── ci.yml              # Continuous Integration
├── release.yml         # Release process
├── nightly.yml         # Nightly builds
└── publish.yml         # Marketplace publishing
```

### Continuous Integration

**Workflow**: `.github/workflows/ci.yml`

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm lint
      
  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm check-types
      
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm test
      
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm build
      - uses: actions/upload-artifact@v3
        with:
          name: vsix
          path: bin/*.vsix
```

### Release Process

**Workflow**: `.github/workflows/release.yml`

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build-and-publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm vsix:production
      
      - name: Publish to VS Code Marketplace
        run: vsce publish
        env:
          VSCE_PAT: ${{ secrets.VSCE_PAT }}
          
      - name: Create GitHub Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false
          
      - name: Upload VSIX to Release
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Nightly Builds

**Workflow**: `.github/workflows/nightly.yml`

```yaml
name: Nightly Build

on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
  workflow_dispatch:      # Manual trigger

jobs:
  build-nightly:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm bundle:nightly
      - run: pnpm vsix:nightly
      
      - name: Publish Nightly
        run: vsce publish --pre-release
        env:
          VSCE_PAT: ${{ secrets.VSCE_PAT }}
```

## Release Process

### Version Management

**Using Changesets**:

**1. Create Changeset**:
```bash
pnpm changeset
```

**2. Version Packages**:
```bash
pnpm changeset:version
```

**3. Publish**:
```bash
pnpm changeset publish
```

### Release Workflow

**1. Development**:
- Create feature branch
- Implement changes
- Add tests
- Create changeset

**2. Review**:
- Open pull request
- Code review
- CI checks pass
- Merge to main

**3. Pre-Release**:
```bash
# Update versions
pnpm changeset:version

# Create PR with version bumps
git checkout -b release/v1.2.3
git add .
git commit -m "chore: version packages"
git push origin release/v1.2.3
```

**4. Release**:
```bash
# After PR merge
git tag v1.2.3
git push origin v1.2.3
```

**5. Publish**:
- GitHub Actions automatically publishes
- VS Code Marketplace
- JetBrains Marketplace
- npm (CLI)

### Release Checklist

- [ ] All tests pass
- [ ] Changelog updated
- [ ] Version bumped
- [ ] Documentation updated
- [ ] Release notes written
- [ ] Tag created
- [ ] Published to marketplaces
- [ ] GitHub release created
- [ ] Announcement posted

## Troubleshooting

### Build Failures

**TypeScript Errors**:
```bash
pnpm check-types
```

**Dependency Issues**:
```bash
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

**Cache Issues**:
```bash
rm -rf .turbo
turbo build --force
```

### Packaging Issues

**VSIX Too Large**:
- Check `.vscodeignore`
- Remove unnecessary files
- Optimize assets

**Missing Dependencies**:
- Check `package.json` dependencies
- Ensure bundling includes needed files

### Deployment Issues

**Authentication Failed**:
- Verify PAT token
- Check token permissions
- Regenerate if expired

**Version Conflict**:
- Ensure version is unique
- Check marketplace version
- Bump version number

---

*Last updated: 2025-10-21*
