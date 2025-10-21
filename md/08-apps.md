# Apps Directory (apps/) Documentation

The `apps/` directory contains supporting applications and tools that complement the main Kilo Code extension. These applications serve various purposes from documentation to testing and evaluation.

## Directory Structure

```
apps/
├── kilocode-docs/      # Documentation website (Docusaurus)
├── playwright-e2e/     # Playwright end-to-end tests
├── storybook/          # Component development and documentation
├── vscode-e2e/         # VS Code extension integration tests
├── vscode-nightly/     # Nightly build configuration
├── web-evals/          # Web-based evaluation interface
└── web-roo-code/       # Web version of Roo Code
```

## Applications Overview

### `apps/kilocode-docs/`

**Name**: `kilocode-docs`
**Purpose**: Official documentation website for Kilo Code
**Technology**: Docusaurus, React, MDX

**Structure**:
```
kilocode-docs/
├── blog-posts/         # Blog content
├── docs/               # Documentation pages
│   ├── getting-started/
│   ├── features/
│   ├── api/
│   └── guides/
├── i18n/               # Internationalization
├── src/
│   ├── components/     # Custom React components
│   ├── css/            # Custom styles
│   └── pages/          # Custom pages
├── static/             # Static assets
│   ├── img/            # Images
│   └── videos/         # Videos
├── docusaurus.config.js # Docusaurus configuration
└── package.json
```

**Key Features**:
- **Documentation**: Comprehensive user guides
- **API Reference**: API documentation
- **Blog**: News and updates
- **Search**: Full-text search
- **Versioning**: Version-specific docs
- **i18n**: Multi-language support

**Scripts**:
```bash
# Development server
pnpm docs:start

# Build for production
pnpm docs:build

# Deploy
pnpm docs:deploy
```

**Configuration** (`docusaurus.config.js`):
- Site metadata
- Theme configuration
- Plugin setup
- Navbar and footer
- SEO settings

**Adding Documentation**:
1. Create MDX file in `docs/`
2. Add frontmatter:
```markdown
---
title: My Page
sidebar_position: 1
---

# Content here
```
3. File will auto-appear in sidebar

**Blog Posts**:
1. Create MDX file in `blog-posts/`
2. Name: `YYYY-MM-DD-title.mdx`
3. Add frontmatter:
```markdown
---
title: Post Title
authors: [author-name]
tags: [tag1, tag2]
---
```

**Deployment**:
- Static site generated to `build/`
- Deployed to hosting (Vercel, Netlify, etc.)
- URL: https://kilocode.ai/docs

---

### `apps/playwright-e2e/`

**Name**: `@roo-code/playwright-e2e`
**Purpose**: Browser-based end-to-end tests using Playwright
**Technology**: Playwright, TypeScript

**Structure**:
```
playwright-e2e/
├── tests/              # Test files
│   ├── auth/           # Authentication tests
│   ├── chat/           # Chat interface tests
│   ├── settings/       # Settings tests
│   └── tools/          # Tool execution tests
├── helpers/            # Test helpers
│   ├── fixtures.ts     # Test fixtures
│   ├── mocks.ts        # Mock data
│   └── utils.ts        # Utility functions
├── scripts/            # Build and run scripts
├── types/              # Type definitions
├── playwright.config.ts # Playwright configuration
└── package.json
```

**Purpose**: Test user workflows and interactions

**Test Categories**:

#### Authentication Tests
```typescript
test('user can log in', async ({ page }) => {
  await page.goto('/login')
  await page.fill('[name=email]', 'user@example.com')
  await page.fill('[name=password]', 'password')
  await page.click('button[type=submit]')
  await expect(page).toHaveURL('/dashboard')
})
```

#### Chat Tests
```typescript
test('user can send message', async ({ page }) => {
  await page.goto('/chat')
  await page.fill('[data-testid=message-input]', 'Hello')
  await page.click('[data-testid=send-button]')
  await expect(page.locator('.message').last()).toContainText('Hello')
})
```

#### Tool Tests
```typescript
test('file read tool works', async ({ page }) => {
  // Test file reading functionality
})
```

**Configuration** (`playwright.config.ts`):
```typescript
export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  use: {
    baseURL: 'http://localhost:3000',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
})
```

**Running Tests**:
```bash
# Run all tests
pnpm playwright

# Run specific test file
pnpm playwright tests/auth/login.spec.ts

# Run with UI
pnpm playwright --ui

# Debug mode
pnpm playwright --debug
```

**Helpers and Fixtures**:
```typescript
// helpers/fixtures.ts
export const test = base.extend({
  authenticatedPage: async ({ page }, use) => {
    // Set up authenticated session
    await use(page)
  },
})
```

**CI Integration**:
- Runs on GitHub Actions
- Parallel execution
- Screenshot and video capture on failure
- Test reports

---

### `apps/storybook/`

**Name**: `@roo-code/storybook`
**Purpose**: Component development environment and documentation
**Technology**: Storybook, React, TypeScript

**Structure**:
```
storybook/
├── .storybook/         # Storybook configuration
│   ├── main.ts         # Main config
│   ├── preview.ts      # Preview config
│   └── theme.ts        # Theme customization
├── generated-theme-styles/ # Generated CSS
├── stories/            # Component stories
│   ├── Button.stories.tsx
│   ├── Input.stories.tsx
│   └── ...
├── src/                # Source files
└── scripts/            # Build scripts
```

**Purpose**: 
- Develop UI components in isolation
- Document component APIs
- Test component variants
- Visual regression testing

**Component Stories**:
```typescript
// stories/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react'
import { Button } from '../src/components/Button'

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger'],
    },
  },
}

export default meta
type Story = StoryObj<typeof Button>

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Click me',
  },
}

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Click me',
  },
}
```

**Features**:
- **Interactive Props**: Modify props in real-time
- **Multiple Variants**: Show different states
- **Dark/Light Mode**: Theme switching
- **Accessibility**: a11y checks
- **Documentation**: MDX docs for components

**Running Storybook**:
```bash
# Development server
pnpm --filter @roo-code/storybook dev

# Build static site
pnpm --filter @roo-code/storybook build
```

**Addons**:
- `@storybook/addon-essentials`: Core addons
- `@storybook/addon-interactions`: Interaction testing
- `@storybook/addon-a11y`: Accessibility testing
- `@storybook/addon-themes`: Theme support

---

### `apps/vscode-e2e/`

**Name**: `@roo-code/vscode-e2e`
**Purpose**: VS Code extension integration tests
**Technology**: VS Code Test Runner, Mocha, TypeScript

**Structure**:
```
vscode-e2e/
├── src/
│   ├── test/           # Test suites
│   │   ├── extension.test.ts
│   │   ├── commands.test.ts
│   │   ├── webview.test.ts
│   │   └── integration.test.ts
│   └── suite/          # Test suite setup
├── package.json
└── tsconfig.json
```

**Purpose**: Test extension integration with VS Code

**Test Categories**:

#### Extension Activation
```typescript
test('extension activates', async () => {
  const ext = vscode.extensions.getExtension('kilocode.kilo-code')
  await ext?.activate()
  assert.ok(ext?.isActive)
})
```

#### Command Tests
```typescript
test('open chat command works', async () => {
  await vscode.commands.executeCommand('kilocode.openChat')
  // Assert webview is open
})
```

#### Webview Tests
```typescript
test('webview receives messages', async () => {
  // Test message passing
})
```

#### Integration Tests
```typescript
test('complete workflow', async () => {
  // Test end-to-end user workflow
})
```

**Running Tests**:
```bash
# Run all tests
pnpm --filter @roo-code/vscode-e2e test

# Run with coverage
pnpm --filter @roo-code/vscode-e2e test:coverage
```

**Test Setup**:
- Launches VS Code instance
- Loads extension
- Runs tests in extension host
- Cleans up after tests

---

### `apps/vscode-nightly/`

**Name**: `@roo-code/vscode-nightly`
**Purpose**: Nightly build configuration
**Technology**: Build scripts, VS Code extension API

**Structure**:
```
vscode-nightly/
├── build/              # Build output
├── package.json        # Nightly package.json
└── scripts/            # Build scripts
```

**Purpose**: Create nightly builds with:
- Latest features
- Pre-release testing
- Bleeding-edge updates

**Configuration**:
```json
{
  "name": "kilo-code-nightly",
  "displayName": "Kilo Code (Nightly)",
  "version": "0.0.0-nightly",
  "preview": true
}
```

**Build Process**:
1. Copy extension code
2. Update version to nightly
3. Update package.json
4. Bundle extension
5. Create VSIX
6. Publish to marketplace

**Scripts**:
```bash
# Build nightly version
pnpm vsix:nightly

# Bundle nightly
pnpm bundle:nightly
```

**Publishing**:
- Automated via GitHub Actions
- Published daily
- Pre-release channel
- Separate marketplace entry

---

### `apps/web-evals/`

**Name**: `@roo-code/web-evals`
**Purpose**: Web interface for evaluation system
**Technology**: React, TypeScript, Vite

**Structure**:
```
web-evals/
├── public/             # Static assets
├── src/
│   ├── components/     # React components
│   │   ├── TestList.tsx
│   │   ├── ResultsView.tsx
│   │   └── Metrics.tsx
│   ├── api/            # API client
│   ├── hooks/          # React hooks
│   └── App.tsx
├── scripts/            # Build scripts
└── package.json
```

**Purpose**: UI for evaluation framework

**Features**:

#### Test Management
- List test cases
- Create new tests
- Edit existing tests
- Delete tests

#### Test Execution
- Run individual tests
- Run test suites
- Monitor progress
- View logs

#### Results Visualization
- Test results table
- Success/failure rates
- Performance metrics
- Comparison charts

#### Metrics Dashboard
- Overall statistics
- Trend analysis
- Model comparison
- Performance tracking

**Components**:

```typescript
// TestList - Display all tests
function TestList() {
  const tests = useTests()
  return (
    <div>
      {tests.map(test => (
        <TestCard key={test.id} test={test} />
      ))}
    </div>
  )
}

// ResultsView - Show test results
function ResultsView({ testId }) {
  const results = useResults(testId)
  return <ResultsTable results={results} />
}
```

**Running**:
```bash
# Development server
pnpm --filter @roo-code/web-evals dev

# Build
pnpm --filter @roo-code/web-evals build
```

**API Integration**:
- REST API client
- Real-time updates via WebSocket
- Test execution API
- Results retrieval

---

### `apps/web-roo-code/`

**Name**: `@roo-code/web-roo-code`
**Purpose**: Web-based version of Roo Code
**Technology**: React, TypeScript, Vite

**Structure**:
```
web-roo-code/
├── public/             # Static assets
├── src/
│   ├── components/     # React components
│   ├── services/       # Service layer
│   ├── hooks/          # Custom hooks
│   └── App.tsx
└── package.json
```

**Purpose**: Browser-based Kilo Code experience

**Features**:
- Chat interface
- Code generation
- File browsing
- Tool execution (limited)
- Settings management

**Limitations** (compared to VS Code extension):
- No direct file system access
- Limited terminal access
- Browser-based tools only
- Cloud-based execution

**Use Cases**:
- Quick tasks without VS Code
- Mobile/tablet access
- Embedded in other applications
- Demo and trial

**Running**:
```bash
# Development
pnpm --filter @roo-code/web-roo-code dev

# Build
pnpm --filter @roo-code/web-roo-code build
```

---

## Common Patterns

### Build Scripts

All apps use similar scripts:
```json
{
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint src",
    "test": "vitest"
  }
}
```

### TypeScript Configuration

Apps extend base config:
```json
{
  "extends": "@roo-code/config-typescript/base.json",
  "compilerOptions": {
    "outDir": "dist"
  }
}
```

### Dependency Management

Apps use workspace dependencies:
```json
{
  "dependencies": {
    "@roo-code/types": "workspace:^",
    "@roo-code/cloud": "workspace:^"
  }
}
```

## Development Workflow

### Running Multiple Apps

```bash
# Run specific app
pnpm --filter kilocode-docs dev
pnpm --filter @roo-code/storybook dev

# Run all apps (not recommended)
# Better to run individually
```

### Building Apps

```bash
# Build all apps
pnpm turbo run build --filter='./apps/*'

# Build specific app
pnpm --filter kilocode-docs build
```

### Testing Apps

```bash
# Test all apps
pnpm turbo run test --filter='./apps/*'

# Test specific app
pnpm --filter @roo-code/playwright-e2e test
```

---

*Last updated: 2025-10-21*
