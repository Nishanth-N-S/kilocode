# Development Guide

This guide covers development practices, workflows, and guidelines for contributing to Kilo Code.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Development Environment](#development-environment)
3. [Project Structure](#project-structure)
4. [Development Workflow](#development-workflow)
5. [Coding Standards](#coding-standards)
6. [Git Workflow](#git-workflow)
7. [Testing](#testing)
8. [Debugging](#debugging)
9. [Performance](#performance)
10. [Security](#security)

## Getting Started

### Prerequisites

**Required**:
- Node.js 20.19.2 (specified in `.nvmrc`)
- pnpm 10.8.1
- Git
- VS Code (recommended)

**For JetBrains Development**:
- Java 17 (JDK)
- Gradle

**For CLI Development**:
- Terminal with good Unicode support

### Initial Setup

**1. Clone Repository**:
```bash
git clone https://github.com/Kilo-Org/kilocode.git
cd kilocode
```

**2. Install Dependencies**:
```bash
pnpm install
```

This automatically:
- Checks Node.js version
- Initializes git submodules
- Installs all dependencies
- Sets up git hooks

**3. Build**:
```bash
pnpm build
```

**4. Start Development**:
```bash
# Press F5 in VS Code
# Or run:
code --extensionDevelopmentPath=.
```

## Development Environment

### Recommended Setup

#### VS Code Extensions

**Required**:
- ESBuild Problem Matchers

**Recommended**:
- ESLint
- Prettier
- TypeScript + JavaScript
- GitLens

**See**: `.vscode/extensions.json` for full list

#### VS Code Settings

The repository includes recommended settings in `.vscode/settings.json`:
- Format on save
- Auto-import organization
- TypeScript validation
- ESLint auto-fix

### Alternative Setups

#### Devcontainer (Windows)

**Benefits**:
- Consistent environment
- All tools pre-installed
- No local setup needed

**Usage**:
1. Install Docker Desktop
2. Install Dev Containers extension
3. Open in container (F1 → "Reopen in Container")

#### Nix Flake (NixOS/Nix users)

**Benefits**:
- Reproducible environment
- Automatic activation with direnv
- Version pinning

**Setup**:
```bash
direnv allow
pnpm install
```

## Project Structure

### Monorepo Organization

```
kilocode/
├── src/              # Main extension
├── webview-ui/       # React UI
├── cli/              # CLI application
├── jetbrains/        # JetBrains plugin
├── packages/         # Shared libraries
└── apps/             # Supporting apps
```

**See**: [02-repository-structure.md](./02-repository-structure.md) for details

### Important Directories

- **`src/core/`**: Core business logic
- **`src/services/`**: Service implementations
- **`webview-ui/src/components/`**: UI components
- **`packages/types/`**: Shared types

## Development Workflow

### Day-to-Day Development

**1. Create Feature Branch**:
```bash
git checkout -b feature/my-feature
```

**2. Make Changes**:
- Edit code
- Add tests
- Update documentation

**3. Lint and Format**:
```bash
pnpm lint
pnpm format
```

**4. Test**:
```bash
pnpm test
```

**5. Commit**:
```bash
git add .
git commit -m "feat: add my feature"
```

Git hooks will:
- Check types
- Run linting
- Format staged files

**6. Push and Create PR**:
```bash
git push origin feature/my-feature
```

### Hot Reloading

#### Extension Development

In development mode (`NODE_ENV=development`):
- Core code changes trigger automatic reload
- No need to restart debugger

In production mode:
- Must stop debugger
- Kill background tasks
- Restart debugger

#### Webview Development

- Changes reflect immediately
- No reload needed
- React Fast Refresh enabled

### Building

**Development Build**:
```bash
pnpm build
```

**Production Build**:
```bash
pnpm vsix:production
```

**Output**: `bin/kilo-code-{version}.vsix`

### Testing Changes

**1. Unit Tests**:
```bash
pnpm test
```

**2. Integration Tests**:
```bash
pnpm --filter @roo-code/vscode-e2e test
```

**3. E2E Tests**:
```bash
pnpm --filter @roo-code/playwright-e2e test
```

**4. Manual Testing**:
- Press F5 to launch extension
- Test in development window
- Check Output panel for logs

## Coding Standards

### TypeScript

**Style Guide**: Standard TypeScript conventions

**Rules**:
- Use `interface` for object types
- Use `type` for unions, primitives
- Prefer `const` over `let`
- Avoid `any`, use `unknown` if needed
- Use optional chaining (`?.`)
- Use nullish coalescing (`??`)

**Example**:
```typescript
interface UserConfig {
  name: string
  age?: number
}

function getConfig(user: UserConfig): string {
  return user.name ?? 'Unknown'
}
```

### React

**Component Style**: Functional components with hooks

**Example**:
```typescript
interface Props {
  title: string
  onClose: () => void
}

export function MyComponent({ title, onClose }: Props) {
  const [count, setCount] = useState(0)
  
  return (
    <div>
      <h1>{title}</h1>
      <button onClick={onClose}>Close</button>
    </div>
  )
}
```

### File Naming

- **Components**: `PascalCase.tsx`
- **Utilities**: `camelCase.ts`
- **Constants**: `UPPER_CASE.ts` or `camelCase.ts`
- **Tests**: `*.test.ts` or `*.spec.ts`

### Imports

**Order**:
1. External packages
2. Internal packages
3. Relative imports

**Example**:
```typescript
import * as vscode from 'vscode'
import { CloudService } from '@roo-code/cloud'
import { formatText } from './utils'
```

### Comments

**Use comments for**:
- Complex logic explanation
- TODO items
- Public API documentation

**Don't comment**:
- Obvious code
- Redundant information

**Example**:
```typescript
// Good: Explains why
// Use exponential backoff to avoid rate limits
await retryWithBackoff(operation)

// Bad: Explains what (obvious)
// Increment counter
counter++
```

### Error Handling

**Pattern**:
```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('Operation failed:', error)
  vscode.window.showErrorMessage('Failed to complete operation')
  throw error // Re-throw if caller should handle
}
```

## Git Workflow

### Branch Naming

**Format**: `type/description`

**Types**:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation
- `refactor/` - Code refactoring
- `test/` - Test additions
- `chore/` - Maintenance

**Example**: `feature/add-code-review-mode`

### Commit Messages

**Format**: [Conventional Commits](https://www.conventionalcommits.org/)

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting
- `refactor`: Code restructuring
- `test`: Tests
- `chore`: Maintenance

**Examples**:
```
feat(chat): add code review mode
fix(webview): resolve message rendering bug
docs: update API documentation
refactor(core): simplify tool execution
```

### Pull Requests

**Requirements**:
1. Branch from `main`
2. Add tests
3. Update documentation
4. Pass CI checks
5. Get review approval

**PR Template**:
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
How was this tested?

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] Code follows style guide
- [ ] All tests pass
```

### Changesets

**Create Changeset**:
```bash
pnpm changeset
```

**Answer prompts**:
1. Which packages changed?
2. What type of change? (major/minor/patch)
3. Describe changes

**Commit changeset file** with your PR

## Testing

### Test Strategy

**Test Pyramid**:
1. Unit tests (most)
2. Integration tests
3. E2E tests (least)

### Writing Tests

**Unit Test Example**:
```typescript
import { describe, it, expect } from 'vitest'
import { formatDate } from './utils'

describe('formatDate', () => {
  it('formats date correctly', () => {
    const date = new Date('2024-01-01')
    expect(formatDate(date)).toBe('Jan 1, 2024')
  })
  
  it('handles invalid date', () => {
    expect(formatDate(null)).toBe('Invalid date')
  })
})
```

**Component Test Example**:
```typescript
import { render, screen } from '@testing-library/react'
import { Button } from './Button'

describe('Button', () => {
  it('renders text', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })
  
  it('calls onClick', () => {
    const onClick = vi.fn()
    render(<Button onClick={onClick}>Click</Button>)
    screen.getByText('Click').click()
    expect(onClick).toHaveBeenCalled()
  })
})
```

### Running Tests

```bash
# All tests
pnpm test

# Watch mode
pnpm test:watch

# Coverage
pnpm test:coverage

# Specific package
pnpm --filter @roo-code/cloud test
```

## Debugging

### VS Code Debugging

**Launch Configurations**: `.vscode/launch.json`

**Available Configs**:
- "Run Extension": Launch extension in development
- "Run Extension (Release)": Launch in production mode
- "Extension Tests": Run extension tests
- "Attach to Extension Host": Attach to running extension

**Usage**:
1. Set breakpoints
2. Press F5 or select config
3. Debug in new window

### Logging

**Extension Logs**:
```typescript
import { createOutputChannelLogger } from './utils/outputChannelLogger'

const logger = createOutputChannelLogger(outputChannel)
logger.info('Message')
logger.error('Error', error)
```

**View logs**: Output panel → "Kilo Code"

**Webview Logs**:
```typescript
console.log('Debug message')
```

**View logs**: Right-click webview → Inspect Element → Console

### Common Issues

**Extension not loading**:
- Check Output panel for errors
- Check Developer Tools (Help → Toggle Developer Tools)
- Verify `package.json` is valid

**Webview not updating**:
- Hard reload: Reload Window (Cmd/Ctrl + R)
- Check browser console for errors
- Verify message passing

**Build errors**:
- Clean and rebuild: `pnpm clean && pnpm build`
- Check for TypeScript errors: `pnpm check-types`
- Reinstall dependencies: `rm -rf node_modules && pnpm install`

## Performance

### Optimization Guidelines

**1. Lazy Loading**:
```typescript
// Lazy load heavy dependencies
const heavy = await import('./heavy-module')
```

**2. Memoization**:
```typescript
const memoized = useMemo(() => expensiveCalc(data), [data])
```

**3. Debouncing**:
```typescript
const debouncedSave = debounce(save, 500)
```

**4. Virtual Scrolling**:
For large lists, use virtual scrolling

**5. Code Splitting**:
Split large bundles using dynamic imports

### Performance Monitoring

**Built-in Tools**:
- VS Code Performance Monitor
- Chrome DevTools
- React DevTools

**Metrics to Track**:
- Extension activation time
- Command response time
- Webview render time
- Memory usage

## Security

### Security Guidelines

**1. Input Validation**:
```typescript
function validateInput(input: unknown): string {
  if (typeof input !== 'string') {
    throw new Error('Invalid input')
  }
  return input
}
```

**2. Sanitization**:
```typescript
import DOMPurify from 'dompurify'

const clean = DOMPurify.sanitize(userInput)
```

**3. Secret Management**:
- Never commit secrets
- Use VS Code Secret Storage
- Environment variables for local dev

**4. API Security**:
- Validate API responses
- Handle rate limits
- Use HTTPS only

### Security Checklist

- [ ] No hardcoded secrets
- [ ] Input validation
- [ ] Output sanitization
- [ ] HTTPS for API calls
- [ ] Error messages don't leak info
- [ ] Dependencies up to date
- [ ] Security vulnerabilities checked

---

*Last updated: 2025-10-21*
