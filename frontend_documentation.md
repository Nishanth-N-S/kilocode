# Frontend Documentation for AI Text Editor Application

## Executive Overview

This documentation is designed to help your frontend development team build a **production-level AI-powered text editing web application** inspired by Kilocode's architecture. This application will enable students and scholars to interact with AI for text manipulation—appending, modifying, and removing text—with sophisticated prompting, context management, and conversational capabilities.

As your senior software architect, I've analyzed Kilocode's frontend architecture and extracted the key patterns and structures your team should implement. This is not just a reference—it's a blueprint for building a scalable, maintainable, and feature-rich application.

---

## Table of Contents

1. [Technology Stack](#technology-stack)
2. [Architecture Overview](#architecture-overview)
3. [Core Frontend Components](#core-frontend-components)
4. [State Management](#state-management)
5. [Communication Layer](#communication-layer)
6. [UI Component Library](#ui-component-library)
7. [Context Management System](#context-management-system)
8. [Real-time Interaction Flow](#real-time-interaction-flow)
9. [Key Features to Implement](#key-features-to-implement)
10. [File Structure](#file-structure)
11. [Implementation Guidelines](#implementation-guidelines)
12. [Testing Strategy](#testing-strategy)
13. [Performance Optimization](#performance-optimization)
14. [Accessibility & Internationalization](#accessibility--internationalization)

---

## 1. Technology Stack

### Core Technologies

```json
{
  "framework": "React 18.3+ with TypeScript",
  "buildTool": "Vite 6.x",
  "stateManagement": "React Context API + Custom Hooks",
  "styling": "Tailwind CSS 4.x + CSS Modules",
  "uiComponents": "Radix UI (headless components)",
  "icons": "Lucide React + VS Code Codicons",
  "dataFetching": "@tanstack/react-query 5.x",
  "markdown": "react-markdown + remark-gfm",
  "virtualization": "react-virtuoso (for large lists)",
  "testing": "Vitest + Testing Library"
}
```

### Key Dependencies

```bash
npm install react react-dom typescript
npm install @radix-ui/react-dialog @radix-ui/react-dropdown-menu @radix-ui/react-tooltip
npm install tailwindcss tailwind-merge class-variance-authority
npm install @tanstack/react-query axios
npm install lucide-react
npm install react-markdown remark-gfm rehype-highlight
npm install react-virtuoso
npm install react-textarea-autosize
npm install i18next react-i18next
npm install zod
```

---

## 2. Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Web Application                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Chat View  │  │ History View │  │ Settings View│      │
│  │   (Main UI)  │  │              │  │              │      │
│  └──────┬───────┘  └──────────────┘  └──────────────┘      │
│         │                                                     │
│  ┌──────▼──────────────────────────────────────────┐        │
│  │      Extension State Context Provider           │        │
│  │  (Global State: User Prefs, API Config, etc.)  │        │
│  └──────┬──────────────────────────────────────────┘        │
│         │                                                     │
│  ┌──────▼──────────────────────────────────────────┐        │
│  │         WebSocket/API Communication Layer        │        │
│  │    (Bidirectional messaging with backend)       │        │
│  └──────┬──────────────────────────────────────────┘        │
│         │                                                     │
└─────────┼─────────────────────────────────────────────────────┘
          │
          ▼
    Backend Server
```

### Component Hierarchy

```
App.tsx (Root)
├── ErrorBoundary
├── TooltipProvider
├── TranslationProvider (i18n)
└── ExtensionStateContextProvider (Global State)
    ├── ChatView (Main Editor Interface)
    │   ├── TaskHeader (Document/Task Info)
    │   ├── ChatTextArea (Input Component)
    │   ├── Virtuoso (Message List - virtualized)
    │   │   └── ChatRow[] (Individual Messages)
    │   │       ├── TextBlock
    │   │       ├── ToolUseBlock (AI actions)
    │   │       └── ThinkingBlock (AI reasoning)
    │   └── BottomControls (Actions & Status)
    ├── HistoryView (Past Sessions)
    ├── SettingsView (Configuration)
    └── ModesView (AI Modes/Personas)
```

---

## 3. Core Frontend Components

### 3.1 Main Application Shell (`App.tsx`)

**Purpose**: Root component managing view routing and global providers.

**Key Responsibilities**:
- Tab-based navigation (Chat, History, Settings, Modes)
- Global error boundary
- Theme management
- WebSocket connection initialization
- Dialog state management (confirmation dialogs, modals)

**Implementation Pattern**:

```typescript
// App.tsx structure
const App = () => {
  const [activeTab, setActiveTab] = useState<'chat' | 'history' | 'settings' | 'modes'>('chat')
  const { didHydrateState } = useExtensionState()
  
  // Initialize WebSocket communication
  useEffect(() => {
    vscode.postMessage({ type: 'webviewDidLaunch' })
  }, [])
  
  if (!didHydrateState) {
    return <LoadingScreen />
  }
  
  return (
    <ErrorBoundary>
      <TooltipProvider>
        <TranslationProvider>
          {activeTab === 'chat' && <ChatView />}
          {activeTab === 'history' && <HistoryView />}
          {activeTab === 'settings' && <SettingsView />}
          {activeTab === 'modes' && <ModesView />}
        </TranslationProvider>
      </TooltipProvider>
    </ErrorBoundary>
  )
}
```

### 3.2 ChatView (`ChatView.tsx`)

**Purpose**: Main text editing interface with AI conversation.

**Key Features**:
- Real-time text streaming from AI
- Virtualized message list (handles 1000+ messages efficiently)
- Image attachment support
- Context mentions (@file, @folder, @url)
- Auto-approval controls
- Checkpoint/undo system

**Structure**:

```typescript
interface ChatViewProps {
  isHidden: boolean
  showAnnouncement: boolean
}

const ChatView: React.FC<ChatViewProps> = ({ isHidden, showAnnouncement }) => {
  const {
    clineMessages,        // Array of conversation messages
    isStreaming,          // AI is currently responding
    inputValue,           // Current input text
    selectedImages,       // Attached images
    autoApprovalSettings  // Auto-approval rules
  } = useExtensionState()
  
  const virtuosoRef = useRef<VirtuosoHandle>(null)
  
  // Auto-scroll to bottom on new message
  useEffect(() => {
    if (isStreaming) {
      virtuosoRef.current?.scrollToIndex({ index: 'LAST', behavior: 'smooth' })
    }
  }, [clineMessages.length, isStreaming])
  
  return (
    <div className="chat-view">
      <TaskHeader />
      
      <Virtuoso
        ref={virtuosoRef}
        data={clineMessages}
        itemContent={(index, message) => (
          <ChatRow
            key={message.ts}
            message={message}
            isExpanded={true}
            onEdit={handleEditMessage}
          />
        )}
      />
      
      <ChatTextArea
        value={inputValue}
        onChange={handleInputChange}
        onSubmit={handleSubmit}
        placeholder="Describe the text changes you want..."
      />
      
      <BottomControls />
    </div>
  )
}
```

### 3.3 ChatRow Component

**Purpose**: Renders individual messages (user or AI) with rich content.

**Message Types**:
1. **User Messages**: Text + optional images
2. **AI Text Responses**: Markdown-formatted text
3. **Tool Use Blocks**: AI actions (e.g., "append_text", "replace_text", "format_document")
4. **Thinking Blocks**: AI reasoning process
5. **Error Messages**: Failure notifications

**Example Implementation**:

```typescript
interface ChatRowProps {
  message: ClineMessage
  isExpanded: boolean
  onEdit?: (ts: number, newText: string) => void
}

const ChatRow: React.FC<ChatRowProps> = ({ message, isExpanded, onEdit }) => {
  if (message.type === 'ask') {
    return <UserMessage message={message} onEdit={onEdit} />
  }
  
  if (message.type === 'say') {
    return (
      <div className="ai-message">
        {message.say === 'text' && (
          <Markdown content={message.text} />
        )}
        {message.say === 'tool_use' && (
          <ToolUseBlock tool={message.tool} isExpanded={isExpanded} />
        )}
        {message.say === 'thinking' && (
          <ThinkingBlock content={message.text} />
        )}
      </div>
    )
  }
  
  return null
}
```

### 3.4 ChatTextArea (Input Component)

**Purpose**: Multi-line text input with advanced features.

**Features**:
- Auto-resize (react-textarea-autosize)
- Keyboard shortcuts (Cmd+Enter to send)
- Image paste support
- Context mentions autocomplete (@file, @folder)
- Slash commands (/format, /summarize)

**Implementation**:

```typescript
const ChatTextArea: React.FC<ChatTextAreaProps> = ({
  value,
  onChange,
  onSubmit,
  placeholder
}) => {
  const textareaRef = useRef<HTMLTextAreaElement>(null)
  const [mentions, setMentions] = useState<Mention[]>([])
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' && (e.metaKey || e.ctrlKey)) {
      e.preventDefault()
      onSubmit()
    }
  }
  
  const handlePaste = async (e: React.ClipboardEvent) => {
    const items = e.clipboardData.items
    for (const item of items) {
      if (item.type.startsWith('image/')) {
        const file = item.getAsFile()
        if (file) {
          const base64 = await fileToBase64(file)
          vscode.postMessage({ type: 'selectImages', images: [base64] })
        }
      }
    }
  }
  
  return (
    <div className="chat-input-container">
      <TextareaAutosize
        ref={textareaRef}
        value={value}
        onChange={onChange}
        onKeyDown={handleKeyDown}
        onPaste={handlePaste}
        placeholder={placeholder}
        minRows={3}
        maxRows={15}
      />
      <MentionDropdown mentions={mentions} />
    </div>
  )
}
```

### 3.5 ToolUseBlock Component

**Purpose**: Display AI tool/action executions (e.g., text modifications).

**Tool Types for Text Editor**:
- `append_text`: Add text at cursor/end
- `insert_text_at_position`: Insert at specific location
- `replace_text`: Find and replace
- `delete_text`: Remove text selection
- `format_text`: Apply formatting (bold, italic, lists)
- `restructure_document`: Reorganize sections

**Example**:

```typescript
interface ToolUseBlockProps {
  tool: {
    name: string
    input: Record<string, any>
    approvalState?: 'pending' | 'approved' | 'rejected'
  }
  isExpanded: boolean
}

const ToolUseBlock: React.FC<ToolUseBlockProps> = ({ tool, isExpanded }) => {
  const [isApproved, setIsApproved] = useState(tool.approvalState === 'approved')
  
  const renderToolContent = () => {
    switch (tool.name) {
      case 'append_text':
        return (
          <div className="tool-append">
            <h4>Appending Text</h4>
            <CodeBlock code={tool.input.text} />
            <span className="position">At: {tool.input.position || 'end'}</span>
          </div>
        )
      
      case 'replace_text':
        return (
          <div className="tool-replace">
            <h4>Replacing Text</h4>
            <DiffViewer
              oldText={tool.input.old_text}
              newText={tool.input.new_text}
            />
          </div>
        )
      
      // ... other tool types
    }
  }
  
  return (
    <div className={`tool-block ${tool.name}`}>
      <ToolHeader
        name={tool.name}
        isApproved={isApproved}
        onApprove={() => handleApprove(tool)}
        onReject={() => handleReject(tool)}
      />
      {isExpanded && renderToolContent()}
    </div>
  )
}
```

---

## 4. State Management

### 4.1 Global State Architecture

Use **React Context API** with **custom hooks** for global state.

**State Structure** (`ExtensionStateContext.tsx`):

```typescript
interface ExtensionStateContextType {
  // User session
  userId?: string
  sessionId?: string
  
  // Current document/task
  currentTaskId?: string
  currentDocument?: {
    id: string
    title: string
    content: string
    lastModified: Date
  }
  
  // Conversation
  clineMessages: ClineMessage[]
  isStreaming: boolean
  inputValue: string
  selectedImages: string[]
  
  // API configuration
  apiConfiguration: ProviderSettings
  selectedModel: string
  
  // Settings
  autoApprovalSettings: {
    alwaysAllowReadOnly: boolean
    alwaysAllowWrite: boolean
    allowedMaxCost?: number
  }
  
  // History
  historyItems: HistoryItem[]
  
  // UI state
  theme: 'light' | 'dark'
  language: string
  showTimestamps: boolean
  
  // Methods
  setInputValue: (value: string) => void
  sendMessage: (text: string, images?: string[]) => void
  updateDocument: (content: string) => void
  // ... other methods
}
```

**Implementation**:

```typescript
// context/ExtensionStateContext.tsx
const ExtensionStateContext = createContext<ExtensionStateContextType | undefined>(undefined)

export const ExtensionStateContextProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, setState] = useState<ExtensionStateContextType>({
    // Initial state
    clineMessages: [],
    isStreaming: false,
    inputValue: '',
    // ...
  })
  
  // Handle messages from backend
  useEffect(() => {
    const messageHandler = (event: MessageEvent) => {
      const message = event.data
      
      switch (message.type) {
        case 'state':
          // Full state update from backend
          setState(prevState => ({ ...prevState, ...message.state }))
          break
        
        case 'messageUpdated':
          // Update specific message
          setState(prevState => ({
            ...prevState,
            clineMessages: updateMessage(prevState.clineMessages, message.message)
          }))
          break
        
        // ... other message types
      }
    }
    
    window.addEventListener('message', messageHandler)
    return () => window.removeEventListener('message', messageHandler)
  }, [])
  
  const sendMessage = useCallback((text: string, images?: string[]) => {
    // Send to backend
    vscode.postMessage({
      type: 'newTask',
      text,
      images
    })
  }, [])
  
  return (
    <ExtensionStateContext.Provider value={{ ...state, sendMessage }}>
      {children}
    </ExtensionStateContext.Provider>
  )
}

export const useExtensionState = () => {
  const context = useContext(ExtensionStateContext)
  if (!context) throw new Error('useExtensionState must be used within provider')
  return context
}
```

### 4.2 Local State for Components

Use **useState** and **useReducer** for component-local state.

**Example**: Message expansion state

```typescript
const ChatRow: React.FC<ChatRowProps> = ({ message }) => {
  const [isExpanded, setIsExpanded] = useState(true)
  const [isEditing, setIsEditing] = useState(false)
  
  // ...
}
```

---

## 5. Communication Layer

### 5.1 WebSocket/Messaging Protocol

For a web application, use **WebSocket** for real-time bidirectional communication.

**Message Types (Frontend → Backend)**:

```typescript
type WebviewMessage = 
  | { type: 'newTask', text: string, images?: string[] }
  | { type: 'askResponse', askTs: number, response: 'approve' | 'reject' }
  | { type: 'cancelTask' }
  | { type: 'saveApiConfiguration', config: ProviderSettings }
  | { type: 'updateDocument', documentId: string, content: string }
  | { type: 'deleteTaskWithId', taskId: string }
  // ... more message types
```

**Message Types (Backend → Frontend)**:

```typescript
type ExtensionMessage =
  | { type: 'state', state: Partial<ExtensionStateContextType> }
  | { type: 'messageUpdated', message: ClineMessage }
  | { type: 'action', action: 'chatButtonClicked' | 'settingsButtonClicked' }
  | { type: 'streamingUpdate', text: string, isComplete: boolean }
  // ... more message types
```

**WebSocket Setup** (`utils/websocket.ts`):

```typescript
class WebSocketService {
  private ws: WebSocket | null = null
  private messageHandlers: ((msg: any) => void)[] = []
  
  connect(url: string) {
    this.ws = new WebSocket(url)
    
    this.ws.onopen = () => {
      console.log('Connected to backend')
    }
    
    this.ws.onmessage = (event) => {
      const message = JSON.parse(event.data)
      this.messageHandlers.forEach(handler => handler(message))
    }
    
    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error)
    }
  }
  
  send(message: WebviewMessage) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(message))
    }
  }
  
  onMessage(handler: (msg: any) => void) {
    this.messageHandlers.push(handler)
  }
}

export const wsService = new WebSocketService()
```

### 5.2 API Client (REST endpoints)

Use **Axios** with **React Query** for REST API calls (e.g., fetching history, models).

**Setup**:

```typescript
// api/client.ts
import axios from 'axios'

export const apiClient = axios.create({
  baseURL: process.env.VITE_API_BASE_URL || 'http://localhost:3000/api',
  headers: {
    'Content-Type': 'application/json'
  }
})

// Add auth token interceptor
apiClient.interceptors.request.use(config => {
  const token = localStorage.getItem('authToken')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})
```

**React Query Hooks**:

```typescript
// hooks/useHistory.ts
import { useQuery } from '@tanstack/react-query'
import { apiClient } from '@/api/client'

export const useHistory = () => {
  return useQuery({
    queryKey: ['history'],
    queryFn: async () => {
      const { data } = await apiClient.get('/history')
      return data.items
    },
    refetchInterval: 30000 // Refetch every 30 seconds
  })
}
```

---

## 6. UI Component Library

### 6.1 Design System

Use **Radix UI** (headless) + **Tailwind CSS** for a consistent design.

**Components to Build**:

1. **Button** (primary, secondary, ghost)
2. **Input / TextArea**
3. **Dialog / Modal**
4. **Dropdown Menu**
5. **Tooltip**
6. **Popover**
7. **Checkbox / Radio**
8. **Select**
9. **Progress Bar**
10. **Badge / Chip**

**Example: Button Component** (`components/ui/button.tsx`):

```typescript
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-blue-600 text-white hover:bg-blue-700',
        secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
        ghost: 'hover:bg-gray-100',
        destructive: 'bg-red-600 text-white hover:bg-red-700'
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4 text-base',
        lg: 'h-12 px-6 text-lg'
      }
    },
    defaultVariants: {
      variant: 'default',
      size: 'md'
    }
  }
)

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export const Button: React.FC<ButtonProps> = ({ 
  variant, 
  size, 
  className, 
  ...props 
}) => {
  return (
    <button
      className={cn(buttonVariants({ variant, size }), className)}
      {...props}
    />
  )
}
```

### 6.2 Markdown Renderer

Use **react-markdown** with syntax highlighting for displaying AI responses.

```typescript
// components/Markdown.tsx
import ReactMarkdown from 'react-markdown'
import remarkGfm from 'remark-gfm'
import { Prism as SyntaxHighlighter } from 'react-syntax-highlighter'
import { oneDark } from 'react-syntax-highlighter/dist/esm/styles/prism'

export const Markdown: React.FC<{ content: string }> = ({ content }) => {
  return (
    <ReactMarkdown
      remarkPlugins={[remarkGfm]}
      components={{
        code({ node, inline, className, children, ...props }) {
          const match = /language-(\w+)/.exec(className || '')
          return !inline && match ? (
            <SyntaxHighlighter
              style={oneDark}
              language={match[1]}
              PreTag="div"
              {...props}
            >
              {String(children).replace(/\n$/, '')}
            </SyntaxHighlighter>
          ) : (
            <code className={className} {...props}>
              {children}
            </code>
          )
        }
      }}
    >
      {content}
    </ReactMarkdown>
  )
}
```

---

## 7. Context Management System

**Context** refers to additional information the AI needs to perform text operations (e.g., style guides, glossaries, user preferences).

### 7.1 Context Types

1. **Document Context**: Current document metadata
2. **Style Guide**: Writing style preferences
3. **Glossary**: Custom terms and definitions
4. **User Instructions**: Custom prompt additions
5. **Workspace Files**: Related documents

### 7.2 Mention System (@-mentions)

Allow users to reference context in input:

- `@style-guide`: Include style guide rules
- `@glossary`: Include term definitions
- `@document:file.txt`: Reference another document
- `@url:https://example.com`: Fetch and include web content

**Implementation**:

```typescript
// utils/mentions.ts
export const parseMentions = (text: string): Mention[] => {
  const mentionRegex = /@(\w+)(?::(.+?))?(?:\s|$)/g
  const mentions: Mention[] = []
  let match
  
  while ((match = mentionRegex.exec(text)) !== null) {
    mentions.push({
      type: match[1],
      value: match[2],
      start: match.index,
      end: match.index + match[0].length
    })
  }
  
  return mentions
}
```

**Autocomplete UI**:

```typescript
const MentionDropdown: React.FC<{ mentions: Mention[] }> = ({ mentions }) => {
  const [suggestions, setSuggestions] = useState<string[]>([])
  
  const fetchSuggestions = async (query: string) => {
    // Fetch from backend
    const response = await apiClient.get('/mentions/search', { params: { q: query } })
    setSuggestions(response.data)
  }
  
  return (
    <Popover open={suggestions.length > 0}>
      <PopoverContent>
        {suggestions.map(suggestion => (
          <div key={suggestion} onClick={() => handleSelect(suggestion)}>
            {suggestion}
          </div>
        ))}
      </PopoverContent>
    </Popover>
  )
}
```

---

## 8. Real-time Interaction Flow

### 8.1 User Sends Message

1. User types in `ChatTextArea` and clicks Send (or Cmd+Enter)
2. Frontend calls `sendMessage(text, images)`
3. Message sent to backend via WebSocket:
   ```typescript
   wsService.send({ type: 'newTask', text, images })
   ```
4. Frontend immediately adds a "pending" user message to UI
5. UI scrolls to bottom

### 8.2 AI Streams Response

1. Backend processes request and starts streaming
2. Frontend receives streaming updates:
   ```typescript
   { type: 'streamingUpdate', text: 'partial response...', isComplete: false }
   ```
3. Frontend appends text to last message in real-time
4. When `isComplete: true`, finalize message

### 8.3 Tool Execution (AI Action)

1. Backend sends:
   ```typescript
   {
     type: 'messageUpdated',
     message: {
       type: 'say',
       say: 'tool_use',
       tool: {
         name: 'append_text',
         input: { text: 'New paragraph...', position: 'end' }
       }
     }
   }
   ```
2. Frontend displays `ToolUseBlock` component
3. If auto-approval is OFF, show "Approve" / "Reject" buttons
4. User approves → Frontend sends:
   ```typescript
   wsService.send({ type: 'askResponse', askTs: message.ts, response: 'approve' })
   ```
5. Backend executes tool and sends updated document:
   ```typescript
   { type: 'documentUpdated', document: { id, content: 'updated content...' } }
   ```
6. Frontend updates document view

---

## 9. Key Features to Implement

### 9.1 Text Editing Operations

**Core Operations**:
1. **Append Text**: Add at end or specific position
2. **Insert Text**: Add at cursor or line number
3. **Replace Text**: Find/replace with regex support
4. **Delete Text**: Remove selection
5. **Format Text**: Apply Markdown/HTML formatting
6. **Restructure**: Reorganize sections, headings

**Implementation Tip**: Each operation should be a tool that the AI calls, and the frontend displays the tool use with a preview.

### 9.2 Document History & Versioning

- **Checkpoints**: Save snapshots before major changes
- **Undo/Redo**: Revert to previous versions
- **Diff View**: Show changes between versions

**UI Component**:

```typescript
const CheckpointRestoreDialog: React.FC = () => {
  const { currentDocument, checkpoints } = useExtensionState()
  
  return (
    <Dialog>
      <DialogTitle>Restore Checkpoint</DialogTitle>
      <DialogContent>
        {checkpoints.map(checkpoint => (
          <div key={checkpoint.id}>
            <span>{checkpoint.timestamp}</span>
            <Button onClick={() => restoreCheckpoint(checkpoint.id)}>
              Restore
            </Button>
          </div>
        ))}
      </DialogContent>
    </Dialog>
  )
}
```

### 9.3 Multi-Document Support

Allow users to work on multiple documents simultaneously.

**State**:

```typescript
interface DocumentState {
  openDocuments: Document[]
  activeDocumentId: string
}
```

**Tab UI**:

```typescript
const DocumentTabs: React.FC = () => {
  const { openDocuments, activeDocumentId, switchDocument } = useExtensionState()
  
  return (
    <div className="document-tabs">
      {openDocuments.map(doc => (
        <button
          key={doc.id}
          className={doc.id === activeDocumentId ? 'active' : ''}
          onClick={() => switchDocument(doc.id)}
        >
          {doc.title}
        </button>
      ))}
    </div>
  )
}
```

### 9.4 Collaboration Features (Future)

- **Real-time co-editing**: Multiple users edit same document
- **Comments/Suggestions**: Leave feedback on text
- **Share Sessions**: Invite others to join AI session

### 9.5 Export & Import

- **Export**: PDF, DOCX, Markdown, plain text
- **Import**: Upload documents to edit with AI

---

## 10. File Structure

```
src/
├── api/
│   ├── client.ts             # Axios setup
│   └── endpoints/
│       ├── documents.ts
│       ├── history.ts
│       └── settings.ts
├── components/
│   ├── chat/
│   │   ├── ChatView.tsx
│   │   ├── ChatRow.tsx
│   │   ├── ChatTextArea.tsx
│   │   ├── ToolUseBlock.tsx
│   │   ├── ThinkingBlock.tsx
│   │   └── MessageList.tsx
│   ├── history/
│   │   ├── HistoryView.tsx
│   │   └── HistoryItem.tsx
│   ├── settings/
│   │   ├── SettingsView.tsx
│   │   ├── ApiConfiguration.tsx
│   │   └── PreferencesPanel.tsx
│   ├── modes/
│   │   ├── ModesView.tsx
│   │   └── ModeCard.tsx
│   ├── ui/
│   │   ├── button.tsx
│   │   ├── dialog.tsx
│   │   ├── input.tsx
│   │   ├── tooltip.tsx
│   │   └── ...
│   └── common/
│       ├── ErrorBoundary.tsx
│       ├── LoadingScreen.tsx
│       └── Markdown.tsx
├── context/
│   └── ExtensionStateContext.tsx
├── hooks/
│   ├── useExtensionState.ts
│   ├── useHistory.ts
│   ├── useWebSocket.ts
│   └── useAutoSave.ts
├── lib/
│   ├── utils.ts
│   └── constants.ts
├── services/
│   └── websocket.ts
├── types/
│   ├── messages.ts
│   ├── state.ts
│   └── models.ts
├── utils/
│   ├── mentions.ts
│   ├── fileUtils.ts
│   └── formatting.ts
├── i18n/
│   └── locales/
│       ├── en.json
│       └── es.json
├── App.tsx
├── main.tsx
└── index.css
```

---

## 11. Implementation Guidelines

### 11.1 Component Best Practices

1. **Single Responsibility**: Each component does one thing well
2. **Composition over Inheritance**: Build complex UIs from simple components
3. **Props Typing**: Always use TypeScript interfaces
4. **Memoization**: Use `React.memo` for expensive renders
5. **Lazy Loading**: Code-split large components

**Example**:

```typescript
// Good: Small, focused component
const UserMessage: React.FC<{ message: ClineMessage }> = React.memo(({ message }) => {
  return (
    <div className="user-message">
      <p>{message.text}</p>
      {message.images?.map(img => <img key={img} src={img} />)}
    </div>
  )
})
```

### 11.2 State Management Rules

1. **Global State**: User auth, API config, current document
2. **Local State**: UI-only state (dropdowns, modals)
3. **Server State**: Use React Query for caching
4. **Avoid Prop Drilling**: Use Context for deep nesting

### 11.3 Performance Optimization

1. **Virtualization**: Use `react-virtuoso` for message lists
2. **Debouncing**: Debounce input changes
3. **Code Splitting**: Lazy load routes
4. **Memoization**: `useMemo`, `useCallback` for expensive operations

**Example**:

```typescript
// Debounce input
const [debouncedValue] = useDebounce(inputValue, 300)

useEffect(() => {
  // Fetch suggestions with debounced value
  fetchSuggestions(debouncedValue)
}, [debouncedValue])
```

### 11.4 Error Handling

1. **Error Boundaries**: Catch React errors
2. **API Errors**: Display user-friendly messages
3. **WebSocket Reconnection**: Auto-reconnect with exponential backoff

**Example**:

```typescript
const useWebSocket = (url: string) => {
  const [isConnected, setIsConnected] = useState(false)
  const reconnectAttempts = useRef(0)
  
  const connect = useCallback(() => {
    const ws = new WebSocket(url)
    
    ws.onopen = () => {
      setIsConnected(true)
      reconnectAttempts.current = 0
    }
    
    ws.onerror = () => {
      setIsConnected(false)
      
      // Exponential backoff
      const delay = Math.min(1000 * Math.pow(2, reconnectAttempts.current), 30000)
      setTimeout(() => {
        reconnectAttempts.current++
        connect()
      }, delay)
    }
  }, [url])
  
  useEffect(() => {
    connect()
  }, [connect])
  
  return { isConnected }
}
```

---

## 12. Testing Strategy

### 12.1 Unit Tests (Vitest)

Test individual components and functions.

```typescript
// __tests__/ChatRow.test.tsx
import { render, screen } from '@testing-library/react'
import { ChatRow } from '@/components/chat/ChatRow'

describe('ChatRow', () => {
  it('renders user message correctly', () => {
    const message = {
      type: 'ask',
      ask: 'user',
      text: 'Hello AI',
      ts: Date.now()
    }
    
    render(<ChatRow message={message} />)
    expect(screen.getByText('Hello AI')).toBeInTheDocument()
  })
})
```

### 12.2 Integration Tests

Test component interactions.

```typescript
it('sends message when Enter is pressed', async () => {
  const onSubmit = vi.fn()
  render(<ChatTextArea onSubmit={onSubmit} />)
  
  const textarea = screen.getByRole('textbox')
  await userEvent.type(textarea, 'Test message{Control>}{Enter}')
  
  expect(onSubmit).toHaveBeenCalledWith('Test message')
})
```

### 12.3 E2E Tests (Playwright)

Test full user flows.

```typescript
test('user can send message and receive AI response', async ({ page }) => {
  await page.goto('http://localhost:3000')
  
  // Type and send message
  await page.fill('[data-testid="chat-input"]', 'Append "Hello World"')
  await page.click('[data-testid="send-button"]')
  
  // Wait for AI response
  await page.waitForSelector('[data-testid="ai-message"]')
  
  // Verify tool use block appears
  const toolBlock = await page.locator('[data-testid="tool-block"]')
  expect(toolBlock).toContainText('append_text')
})
```

---

## 13. Performance Optimization

### 13.1 Bundle Size

- **Code splitting**: Lazy load routes
- **Tree shaking**: Remove unused code
- **Compression**: Enable gzip/brotli

```typescript
// Lazy load routes
const SettingsView = lazy(() => import('@/components/settings/SettingsView'))
const HistoryView = lazy(() => import('@/components/history/HistoryView'))
```

### 13.2 Rendering Performance

- **Virtualization**: Only render visible messages
- **Memoization**: Prevent unnecessary re-renders
- **Debouncing**: Limit state updates

### 13.3 Network Optimization

- **WebSocket**: Use binary messages for images
- **Compression**: Compress large text payloads
- **Caching**: Cache API responses with React Query

---

## 14. Accessibility & Internationalization

### 14.1 Accessibility (a11y)

1. **Keyboard Navigation**: All features accessible via keyboard
2. **Screen Reader Support**: ARIA labels on interactive elements
3. **Focus Management**: Logical focus order
4. **Color Contrast**: WCAG AA compliance

**Example**:

```typescript
<button
  aria-label="Send message"
  aria-disabled={isStreaming}
  onClick={handleSend}
>
  <SendIcon />
</button>
```

### 14.2 Internationalization (i18n)

Use **i18next** for translations.

**Setup**:

```typescript
// i18n/config.ts
import i18n from 'i18next'
import { initReactI18next } from 'react-i18next'

i18n
  .use(initReactI18next)
  .init({
    resources: {
      en: { translation: require('./locales/en.json') },
      es: { translation: require('./locales/es.json') }
    },
    lng: 'en',
    fallbackLng: 'en'
  })
```

**Usage**:

```typescript
import { useTranslation } from 'react-i18next'

const ChatView = () => {
  const { t } = useTranslation()
  
  return (
    <h1>{t('chat.title')}</h1>
  )
}
```

---

## Final Notes for Your Team

### Priority Tasks (MVP)

1. ✅ **Set up project** (React + Vite + TypeScript + Tailwind)
2. ✅ **Build UI component library** (buttons, inputs, dialogs)
3. ✅ **Implement ChatView** with message list
4. ✅ **Implement WebSocket communication**
5. ✅ **Build state management** (ExtensionStateContext)
6. ✅ **Add tool use blocks** (append, replace, delete text)
7. ✅ **Implement history view**
8. ✅ **Add settings panel**

### Success Criteria

- Users can converse with AI to edit text
- AI actions are clearly visualized (tool use blocks)
- Application is responsive and performant (handles 1000+ messages)
- WebSocket connection is stable with auto-reconnect
- State management is predictable and debuggable

### Resources

- **Kilocode GitHub**: https://github.com/Kilo-Org/kilocode (reference architecture)
- **Radix UI Docs**: https://www.radix-ui.com/
- **React Query Docs**: https://tanstack.com/query/latest
- **Tailwind CSS**: https://tailwindcss.com/

---

**Good luck building an exceptional AI text editor! This documentation should give your team a solid foundation. Refer back to Kilocode's implementation for additional patterns and edge cases.**

---

*Document Version: 1.0*  
*Last Updated: [Current Date]*  
*Author: Senior Software Architect*
