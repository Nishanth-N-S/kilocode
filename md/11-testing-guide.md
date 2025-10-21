# Testing Guide

This guide covers testing strategies, frameworks, and best practices for Kilo Code.

## Table of Contents

1. [Testing Philosophy](#testing-philosophy)
2. [Test Types](#test-types)
3. [Unit Testing](#unit-testing)
4. [Integration Testing](#integration-testing)
5. [End-to-End Testing](#end-to-end-testing)
6. [Component Testing](#component-testing)
7. [Test Coverage](#test-coverage)
8. [Continuous Integration](#continuous-integration)

## Testing Philosophy

### Test Pyramid

```
      /\
     /  \    E2E Tests (Few)
    /────\
   /      \  Integration Tests (Some)
  /────────\
 /          \ Unit Tests (Many)
/────────────\
```

**Strategy**:
- **Many unit tests**: Fast, focused, isolated
- **Some integration tests**: Test component interaction
- **Few E2E tests**: Test critical user workflows

### Testing Principles

1. **Fast**: Tests should run quickly
2. **Independent**: Tests don't depend on each other
3. **Repeatable**: Same result every time
4. **Self-validating**: Pass or fail, no manual checking
5. **Timely**: Write tests with or before code

## Test Types

### Overview

| Type | Framework | Location | Purpose |
|------|-----------|----------|---------|
| Unit | Vitest | Co-located `__tests__/` | Test individual functions |
| Integration | Vitest | `src/__tests__/` | Test component interaction |
| Component | Vitest + Testing Library | `webview-ui/src/__tests__/` | Test React components |
| E2E (Extension) | VS Code Test Runner | `apps/vscode-e2e/` | Test extension in VS Code |
| E2E (Browser) | Playwright | `apps/playwright-e2e/` | Test web UI |

## Unit Testing

### Framework: Vitest

**Configuration**: `vitest.config.ts`

```typescript
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
  },
})
```

### Writing Unit Tests

**File Structure**:
```
src/
├── utils/
│   ├── formatters.ts
│   └── __tests__/
│       └── formatters.test.ts
```

**Example Test**:
```typescript
import { describe, it, expect, beforeEach, afterEach } from 'vitest'
import { formatDate, formatCurrency } from '../formatters'

describe('formatters', () => {
  describe('formatDate', () => {
    it('formats valid date', () => {
      const date = new Date('2024-01-15')
      expect(formatDate(date)).toBe('Jan 15, 2024')
    })
    
    it('handles null date', () => {
      expect(formatDate(null)).toBe('Invalid date')
    })
    
    it('uses custom format', () => {
      const date = new Date('2024-01-15')
      expect(formatDate(date, 'yyyy-MM-dd')).toBe('2024-01-15')
    })
  })
  
  describe('formatCurrency', () => {
    it('formats USD', () => {
      expect(formatCurrency(1234.56, 'USD')).toBe('$1,234.56')
    })
    
    it('handles zero', () => {
      expect(formatCurrency(0, 'USD')).toBe('$0.00')
    })
  })
})
```

### Mocking

**Mock Functions**:
```typescript
import { vi } from 'vitest'

const mockFn = vi.fn()
mockFn.mockReturnValue('mocked value')
mockFn.mockResolvedValue('async value')

// Assert calls
expect(mockFn).toHaveBeenCalled()
expect(mockFn).toHaveBeenCalledWith('arg')
expect(mockFn).toHaveBeenCalledTimes(1)
```

**Mock Modules**:
```typescript
vi.mock('./api', () => ({
  fetchData: vi.fn().mockResolvedValue({ data: 'mock' })
}))

import { fetchData } from './api'

it('uses mock', async () => {
  const result = await fetchData()
  expect(result).toEqual({ data: 'mock' })
})
```

**Mock VS Code API**:
```typescript
vi.mock('vscode', () => ({
  window: {
    showInformationMessage: vi.fn(),
    createOutputChannel: vi.fn(() => ({
      appendLine: vi.fn(),
      show: vi.fn(),
    })),
  },
  workspace: {
    getConfiguration: vi.fn(() => ({
      get: vi.fn(),
      update: vi.fn(),
    })),
  },
}))
```

### Testing Async Code

**Async/Await**:
```typescript
it('fetches data', async () => {
  const data = await fetchData()
  expect(data).toBeDefined()
})
```

**Promises**:
```typescript
it('returns promise', () => {
  return fetchData().then(data => {
    expect(data).toBeDefined()
  })
})
```

**Error Handling**:
```typescript
it('throws error', async () => {
  await expect(failingFunction()).rejects.toThrow('Error message')
})
```

### Setup and Teardown

**Before/After Each**:
```typescript
describe('Database tests', () => {
  let db: Database
  
  beforeEach(async () => {
    db = await createTestDatabase()
  })
  
  afterEach(async () => {
    await db.close()
  })
  
  it('saves data', async () => {
    await db.save({ id: 1, name: 'Test' })
    const result = await db.find(1)
    expect(result.name).toBe('Test')
  })
})
```

**Before/After All**:
```typescript
describe('Suite setup', () => {
  beforeAll(async () => {
    await setupTestEnvironment()
  })
  
  afterAll(async () => {
    await cleanupTestEnvironment()
  })
  
  // Tests...
})
```

## Integration Testing

### Testing Component Interaction

**Example**: Testing service with dependencies

```typescript
import { describe, it, expect, beforeEach } from 'vitest'
import { CloudService } from '../CloudService'
import { ApiClient } from '../ApiClient'

describe('CloudService Integration', () => {
  let service: CloudService
  let apiClient: ApiClient
  
  beforeEach(() => {
    apiClient = new ApiClient('test-key')
    service = new CloudService(apiClient)
  })
  
  it('authenticates and fetches user', async () => {
    await service.authenticate('token')
    const user = await service.getCurrentUser()
    
    expect(user).toBeDefined()
    expect(user.id).toBeTruthy()
  })
})
```

### Testing Message Passing

```typescript
import { describe, it, expect } from 'vitest'
import { ClineProvider } from '../ClineProvider'
import { WebviewMessageHandler } from '../webviewMessageHandler'

describe('Message Flow', () => {
  it('handles user message', async () => {
    const provider = new ClineProvider(context)
    const handler = new WebviewMessageHandler(provider)
    
    const message = { type: 'userMessage', text: 'Hello' }
    await handler.handle(message)
    
    const state = provider.getState()
    expect(state.messages).toHaveLength(1)
  })
})
```

## End-to-End Testing

### VS Code Extension Tests

**Location**: `apps/vscode-e2e/`
**Framework**: VS Code Test Runner

**Example Test**:
```typescript
import * as assert from 'assert'
import * as vscode from 'vscode'

suite('Extension Test Suite', () => {
  vscode.window.showInformationMessage('Start tests')
  
  test('Extension loads', async () => {
    const ext = vscode.extensions.getExtension('kilocode.kilo-code')
    assert.ok(ext)
    
    await ext?.activate()
    assert.ok(ext?.isActive)
  })
  
  test('Command registered', async () => {
    const commands = await vscode.commands.getCommands()
    assert.ok(commands.includes('kilocode.openChat'))
  })
  
  test('Opens chat view', async () => {
    await vscode.commands.executeCommand('kilocode.openChat')
    
    // Wait for webview to open
    await new Promise(resolve => setTimeout(resolve, 1000))
    
    // Assert webview is visible
    // (implementation depends on your webview setup)
  })
})
```

**Running**:
```bash
pnpm --filter @roo-code/vscode-e2e test
```

### Playwright Tests

**Location**: `apps/playwright-e2e/`
**Framework**: Playwright

**Example Test**:
```typescript
import { test, expect } from '@playwright/test'

test.describe('Chat Interface', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/')
    await page.click('[data-testid=login-button]')
    // Login flow...
  })
  
  test('sends message', async ({ page }) => {
    await page.fill('[data-testid=message-input]', 'Hello AI')
    await page.click('[data-testid=send-button]')
    
    // Wait for response
    await page.waitForSelector('.message.assistant')
    
    const response = await page.locator('.message.assistant').last().textContent()
    expect(response).toBeTruthy()
  })
  
  test('displays code block', async ({ page }) => {
    await page.fill('[data-testid=message-input]', 'Write a function')
    await page.click('[data-testid=send-button]')
    
    await page.waitForSelector('.code-block')
    const codeBlock = page.locator('.code-block').first()
    await expect(codeBlock).toBeVisible()
  })
})
```

**Running**:
```bash
# All browsers
pnpm playwright

# Specific browser
pnpm playwright --project=chromium

# Debug mode
pnpm playwright --debug

# UI mode
pnpm playwright --ui
```

## Component Testing

### React Testing Library

**Example**:
```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import { describe, it, expect, vi } from 'vitest'
import { ChatInput } from '../ChatInput'

describe('ChatInput', () => {
  it('renders input field', () => {
    render(<ChatInput onSend={vi.fn()} />)
    expect(screen.getByRole('textbox')).toBeInTheDocument()
  })
  
  it('calls onSend when submitted', () => {
    const onSend = vi.fn()
    render(<ChatInput onSend={onSend} />)
    
    const input = screen.getByRole('textbox')
    fireEvent.change(input, { target: { value: 'Hello' } })
    fireEvent.submit(input.closest('form')!)
    
    expect(onSend).toHaveBeenCalledWith('Hello')
  })
  
  it('clears input after send', () => {
    render(<ChatInput onSend={vi.fn()} />)
    
    const input = screen.getByRole('textbox') as HTMLInputElement
    fireEvent.change(input, { target: { value: 'Hello' } })
    fireEvent.submit(input.closest('form')!)
    
    expect(input.value).toBe('')
  })
  
  it('disables send when empty', () => {
    render(<ChatInput onSend={vi.fn()} />)
    
    const button = screen.getByRole('button')
    expect(button).toBeDisabled()
  })
})
```

### Testing Hooks

```typescript
import { renderHook, act } from '@testing-library/react'
import { describe, it, expect } from 'vitest'
import { useCounter } from '../useCounter'

describe('useCounter', () => {
  it('increments counter', () => {
    const { result } = renderHook(() => useCounter(0))
    
    expect(result.current.count).toBe(0)
    
    act(() => {
      result.current.increment()
    })
    
    expect(result.current.count).toBe(1)
  })
  
  it('decrements counter', () => {
    const { result } = renderHook(() => useCounter(5))
    
    act(() => {
      result.current.decrement()
    })
    
    expect(result.current.count).toBe(4)
  })
})
```

### Testing with Context

```typescript
import { render, screen } from '@testing-library/react'
import { ThemeProvider } from '../context/ThemeContext'

const renderWithTheme = (ui: React.ReactElement, theme = 'dark') => {
  return render(
    <ThemeProvider value={{ theme, setTheme: vi.fn() }}>
      {ui}
    </ThemeProvider>
  )
}

describe('ThemedButton', () => {
  it('uses dark theme', () => {
    renderWithTheme(<ThemedButton />, 'dark')
    expect(screen.getByRole('button')).toHaveClass('dark')
  })
})
```

## Test Coverage

### Running Coverage

```bash
# All packages
pnpm test:coverage

# Specific package
pnpm --filter @roo-code/cloud test:coverage
```

### Coverage Reports

**Formats**:
- Text: Console output
- HTML: `coverage/index.html`
- JSON: `coverage/coverage.json`
- LCOV: `coverage/lcov.info` (for CI)

**View HTML Report**:
```bash
open coverage/index.html
```

### Coverage Goals

**Targets**:
- **Statements**: 80%+
- **Branches**: 75%+
- **Functions**: 80%+
- **Lines**: 80%+

**Critical Code**: 95%+ coverage
- Core business logic
- Security-critical code
- Data transformation

**Less Critical**: 60%+ coverage
- UI components (visual testing)
- Experimental features

### Coverage Configuration

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        '**/__tests__/**',
        '**/*.test.ts',
        '**/*.spec.ts',
        '**/types/**',
      ],
      thresholds: {
        statements: 80,
        branches: 75,
        functions: 80,
        lines: 80,
      },
    },
  },
})
```

## Continuous Integration

### GitHub Actions

**Workflow**: `.github/workflows/test.yml`

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          
      - name: Install pnpm
        run: npm install -g pnpm
        
      - name: Install dependencies
        run: pnpm install
        
      - name: Run linting
        run: pnpm lint
        
      - name: Run type checking
        run: pnpm check-types
        
      - name: Run unit tests
        run: pnpm test
        
      - name: Run E2E tests
        run: pnpm --filter @roo-code/playwright-e2e test
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

### Test Matrix

**Multiple Environments**:
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [18, 20]
```

### Caching

```yaml
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.pnpm-store
    key: ${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}
```

## Best Practices

### 1. Test Naming

**Pattern**: `should <expected behavior> when <condition>`

```typescript
it('should return user when ID exists', () => {})
it('should throw error when ID is invalid', () => {})
it('should cache results after first call', () => {})
```

### 2. AAA Pattern

**Arrange, Act, Assert**:
```typescript
it('calculates total', () => {
  // Arrange
  const items = [{ price: 10 }, { price: 20 }]
  
  // Act
  const total = calculateTotal(items)
  
  // Assert
  expect(total).toBe(30)
})
```

### 3. One Assertion Per Test

```typescript
// Good: Focused test
it('saves user', () => {
  const saved = saveUser(user)
  expect(saved).toBe(true)
})

it('returns saved user', () => {
  const result = saveUser(user)
  expect(result.id).toBeDefined()
})

// Avoid: Multiple unrelated assertions
it('saves user', () => {
  const saved = saveUser(user)
  expect(saved).toBe(true)
  expect(result.id).toBeDefined()
  expect(result.name).toBe(user.name)
})
```

### 4. Test Data Builders

```typescript
function createTestUser(overrides = {}) {
  return {
    id: '1',
    name: 'Test User',
    email: 'test@example.com',
    ...overrides
  }
}

it('formats user name', () => {
  const user = createTestUser({ name: 'John Doe' })
  expect(formatName(user)).toBe('John Doe')
})
```

### 5. Avoid Test Interdependence

```typescript
// Bad: Tests depend on each other
let userId: string

it('creates user', () => {
  userId = createUser()
})

it('finds user', () => {
  const user = findUser(userId) // Depends on previous test
})

// Good: Independent tests
it('creates user', () => {
  const userId = createUser()
  expect(userId).toBeDefined()
})

it('finds user', () => {
  const userId = createUser() // Create own data
  const user = findUser(userId)
  expect(user).toBeDefined()
})
```

---

*Last updated: 2025-10-21*
