# Webview UI (webview-ui/) Documentation

The `webview-ui/` directory contains the React-based user interface for Kilo Code. This is what users see and interact with when using the extension.

## Directory Structure

```
webview-ui/
├── audio/                  # Audio assets
├── public/                 # Public static assets
├── src/                    # React application source
│   ├── components/         # React components
│   ├── context/            # React context providers
│   ├── hooks/              # Custom React hooks
│   ├── i18n/               # Internationalization
│   ├── kilocode/           # Kilo-specific features
│   ├── lib/                # Library utilities
│   ├── oauth/              # OAuth authentication
│   ├── services/           # Service layer
│   ├── stories/            # Storybook stories
│   ├── utils/              # Utility functions
│   ├── vite-plugins/       # Vite plugins
│   ├── App.tsx             # Main application component
│   ├── index.css           # Global styles
│   └── main.tsx            # Application entry point
├── index.html              # HTML template
├── package.json            # Dependencies and scripts
├── tsconfig.json           # TypeScript configuration
├── tailwind.config.js      # Tailwind CSS configuration
└── vite.config.ts          # Vite build configuration
```

## Technology Stack

### Core Technologies
- **React 18**: UI framework
- **TypeScript**: Type-safe JavaScript
- **Vite**: Build tool and dev server
- **Tailwind CSS**: Utility-first CSS framework

### UI Libraries
- **Radix UI**: Accessible component primitives
  - Alert dialogs
  - Checkboxes
  - Collapsibles
  - Dropdowns
  - Popovers
  - Tooltips
  - And more...
- **Lucide React**: Icon library
- **VS Code Codicons**: VS Code icon set
- **VS Code Webview UI Toolkit**: VS Code-specific components

### Additional Libraries
- **TanStack React Query**: Data fetching and caching
- **i18next**: Internationalization
- **Mermaid**: Diagram rendering
- **KaTeX**: Math rendering
- **DOMPurify**: HTML sanitization
- **date-fns**: Date manipulation
- **PostHog**: Analytics

## Main Application Files

### `main.tsx`
**Purpose**: Application entry point

**Responsibilities**:
- Render React root
- Initialize providers
- Set up error boundaries

### `App.tsx`
**Purpose**: Main application component

**Structure**:
- Provider hierarchy setup
- Route configuration
- Theme management
- Message handling with extension

### `index.css`
**Purpose**: Global styles

**Contains**:
- Tailwind CSS imports
- CSS variables
- Global resets
- Custom utility classes

### `index.html`
**Purpose**: HTML template

**Features**:
- Root div for React app
- Script tag for main.tsx
- CSP meta tags for security

## Source Directory (`src/`)

### `components/`
**Purpose**: React components organized by feature

#### `components/common/`
**Purpose**: Shared/reusable components

**Key Components**:
- Button variants
- Input fields
- Cards
- Modals
- Tooltips
- Loading indicators

**Example Components**:
```typescript
- Button.tsx
- Input.tsx
- Card.tsx
- Modal.tsx
- Tooltip.tsx
- Spinner.tsx
```

#### `components/ui/`
**Purpose**: UI component library

**Radix UI Integration**:
- Pre-configured Radix components
- Styled with Tailwind CSS
- Accessible by default

**Components**:
- `alert-dialog.tsx`
- `button.tsx`
- `checkbox.tsx`
- `dialog.tsx`
- `dropdown-menu.tsx`
- `input.tsx`
- `select.tsx`
- `slider.tsx`
- `tooltip.tsx`

**Hooks**:
- `hooks/use-toast.tsx`: Toast notifications
- `hooks/use-debounce.tsx`: Debouncing
- `hooks/kilocode/`: Kilo-specific hooks

#### `components/chat/` (implied)
**Purpose**: Chat interface components

**Features**:
- Message list
- Message input
- Message bubble
- Code blocks
- Thinking indicators
- Tool usage display

#### `components/mcp/`
**Purpose**: Model Context Protocol UI

**Components**:
- MCP server list
- Server configuration
- Tool browser
- Connection status

#### `components/cloud/`
**Purpose**: Cloud service UI

**Components**:
- Login/authentication
- User profile
- Credits display
- Subscription management

#### `components/modes/`
**Purpose**: Mode selection and configuration

**Components**:
- Mode picker
- Mode editor
- Custom mode creation

#### `components/marketplace/`
**Purpose**: Marketplace UI

**Components**:
- Extension browser
- MCP server marketplace
- Installation UI
- Search and filters

#### `components/human-relay/`
**Purpose**: Human-in-the-loop interactions

**Components**:
- Approval requests
- Confirmation dialogs
- Input prompts

### `context/`
**Purpose**: React context providers for global state

**Key Contexts**:

#### `ExtensionStateContext.tsx`
**Purpose**: Extension state management
**Provides**:
- Current task state
- Configuration
- User settings
- API keys

#### `ThemeContext.tsx`
**Purpose**: Theme management
**Provides**:
- Current theme (light/dark)
- Theme toggle
- VS Code theme synchronization

#### `AuthContext.tsx`
**Purpose**: Authentication state
**Provides**:
- User authentication status
- Login/logout functions
- User information

#### `I18nContext.tsx`
**Purpose**: Internationalization
**Provides**:
- Current language
- Translation functions
- Language switching

### `hooks/`
**Purpose**: Custom React hooks

**Common Hooks**:

#### `useExtensionState.ts`
**Purpose**: Access extension state
**Returns**: Current extension state and updaters

#### `usePostMessage.ts`
**Purpose**: Send messages to extension
**Usage**: 
```typescript
const postMessage = usePostMessage()
postMessage({ type: 'startTask', text: 'Hello' })
```

#### `useVSCodeTheme.ts`
**Purpose**: Sync with VS Code theme
**Returns**: Current theme (light/dark)

#### `useSettings.ts`
**Purpose**: Access and update settings
**Returns**: Settings object and update function

#### `kilocode/` Hooks
**Purpose**: Kilo-specific custom hooks
- Feature-specific logic
- Kilo service integrations

### `services/`
**Purpose**: Service layer for business logic

**Services**:

#### `messageService.ts`
**Purpose**: Handle messages to/from extension
**Functions**:
- `sendMessage()`: Send message to extension
- `onMessage()`: Listen for messages
- `requestData()`: Request data from extension

#### `apiService.ts`
**Purpose**: API communication
**Functions**:
- API key validation
- Model selection
- Provider configuration

#### `storageService.ts`
**Purpose**: Local storage utilities
**Functions**:
- Save/load preferences
- Cache management
- Session storage

### `utils/`
**Purpose**: Utility functions

**Key Utils**:

#### `markdown.ts`
**Purpose**: Markdown parsing and rendering
**Features**:
- Code block extraction
- Syntax highlighting
- Link handling

#### `codeBlock.ts`
**Purpose**: Code block utilities
**Features**:
- Language detection
- Copy to clipboard
- Apply code action

#### `timeline/`
**Purpose**: Timeline utilities
**Features**:
- Task timeline rendering
- Event sorting
- Timeline visualization

#### `kilocode/`
**Purpose**: Kilo-specific utilities
**Features**:
- Kilo-specific formatting
- Custom transformations

### `i18n/`
**Purpose**: Internationalization setup

**Files**:
- `config.ts`: i18next configuration
- Translation loading
- Language detection

**Supported Languages**:
- English (default)
- Spanish
- French
- German
- Japanese
- And more...

### `kilocode/`
**Purpose**: Kilo Code specific features

**Features**:
- Kilo-specific UI components
- Custom modes
- Kilo branding
- Feature flags

### `lib/`
**Purpose**: Library utilities

**Contains**:
- `utils.ts`: Common utility functions
- `cn()`: Class name merger (clsx + tailwind-merge)
- Type guards
- Validators

### `oauth/`
**Purpose**: OAuth authentication flow

**Features**:
- OAuth callback handling
- Token management
- Provider integration

### `stories/`
**Purpose**: Storybook stories

**Organization**:
- Component stories
- Documentation
- Examples
- Interactive demos

### `vite-plugins/`
**Purpose**: Custom Vite plugins

**Plugins**:
- Custom transformations
- Build optimizations
- Development utilities

## Build Configuration

### `vite.config.ts`
**Purpose**: Vite configuration

**Key Settings**:
- Build target: ES2020
- Output directory: `../src/webview-ui`
- Entry point: `src/main.tsx`
- Plugins: React, Tailwind CSS
- Asset handling
- Code splitting

**Build Output**:
- Bundled to `../src/webview-ui/`
- Includes HTML, CSS, JS
- Assets are inlined or copied

### `tsconfig.json`
**Purpose**: TypeScript configuration

**Settings**:
- JSX: react-jsx
- Module: ESNext
- Target: ES2020
- Strict mode enabled
- Path aliases configured

### `tailwind.config.js`
**Purpose**: Tailwind CSS configuration

**Customization**:
- Theme colors
- Breakpoints
- Custom utilities
- Plugins

### `package.json`
**Purpose**: Package configuration

**Key Scripts**:
- `dev`: Start development server
- `build`: Build for production
- `build:nightly`: Build nightly version
- `preview`: Preview production build
- `lint`: Run ESLint
- `test`: Run tests
- `format`: Format with Prettier

## Communication with Extension

### Message Protocol

**From Webview to Extension**:
```typescript
vscode.postMessage({
  type: 'messageType',
  payload: { /* data */ }
})
```

**From Extension to Webview**:
```typescript
window.addEventListener('message', (event) => {
  const message = event.data
  // Handle message
})
```

**Message Types**:
- `startTask`: Start a new task
- `sendMessage`: Send user message
- `cancelTask`: Cancel current task
- `updateSettings`: Update settings
- `getState`: Request current state
- And many more...

### State Synchronization

**Pattern**:
1. Webview requests data via message
2. Extension responds with current state
3. Webview updates UI
4. User interacts with UI
5. Webview sends update to extension
6. Extension processes and updates state
7. Extension broadcasts state change
8. Webview receives and updates UI

## Styling

### Tailwind CSS

**Usage**:
```tsx
<div className="flex items-center gap-2 p-4 rounded-lg bg-background">
  <Button variant="primary" size="sm">Click me</Button>
</div>
```

**Custom Classes**:
- Component variants via `class-variance-authority`
- Responsive utilities
- Dark mode support

### CSS Variables

**Theme Variables**:
```css
:root {
  --background: ...;
  --foreground: ...;
  --primary: ...;
  --secondary: ...;
  /* VS Code theme colors */
}
```

**Usage**:
```css
.my-component {
  background-color: var(--background);
  color: var(--foreground);
}
```

## Testing

### Test Setup
- **Framework**: Vitest
- **Testing Library**: React Testing Library
- **Coverage**: Enabled

### Test Files
- Co-located with components in `__tests__/` directories
- Named: `*.test.tsx` or `*.spec.tsx`

### Running Tests
```bash
pnpm test          # Run all tests
pnpm test:watch    # Watch mode
pnpm test:coverage # With coverage
```

## Development

### Starting Dev Server

```bash
cd webview-ui
pnpm dev
```

**Features**:
- Hot module replacement
- Fast refresh
- VS Code webview simulation

### Building

```bash
pnpm build           # Production build
pnpm build:nightly   # Nightly build
```

**Output**:
- Files are written to `../src/webview-ui/`
- Extension bundles these files

### Debugging

**In VS Code Extension Host**:
1. Press F5 to start extension
2. Open Kilo Code webview
3. Right-click in webview
4. Select "Inspect Element"
5. Use Chrome DevTools

**Standalone (for development)**:
1. Run `pnpm dev`
2. Open http://localhost:5173
3. Use browser DevTools

## Key Features

### Accessibility
- Radix UI components are accessible by default
- Keyboard navigation
- Screen reader support
- ARIA attributes

### Internationalization
- i18next for translations
- Language detection
- Dynamic language switching
- RTL support (if configured)

### Performance
- React Query for efficient data fetching
- Code splitting
- Lazy loading
- Memoization

### Security
- Content Security Policy (CSP)
- DOMPurify for HTML sanitization
- Input validation
- XSS prevention

## Common Patterns

### Component Structure
```tsx
interface ComponentProps {
  // Props
}

export function Component({ ...props }: ComponentProps) {
  // Hooks
  const state = useExtensionState()
  
  // Event handlers
  const handleClick = () => {
    // ...
  }
  
  // Render
  return (
    <div>
      {/* JSX */}
    </div>
  )
}
```

### Context Usage
```tsx
import { useExtensionState } from '@/context/ExtensionStateContext'

function MyComponent() {
  const { state, updateState } = useExtensionState()
  // Use state
}
```

### Message Sending
```tsx
import { usePostMessage } from '@/hooks/usePostMessage'

function MyComponent() {
  const postMessage = usePostMessage()
  
  const startTask = () => {
    postMessage({ type: 'startTask', text: 'Hello' })
  }
}
```

---

*Last updated: 2025-10-21*
