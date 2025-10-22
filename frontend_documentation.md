# Frontend Documentation for AI Text Manipulation Website

## Overview

This document serves as a comprehensive guide for frontend developers to build an AI-powered text manipulation website inspired by Kilocode's architecture. The website will enable scholars, students, and educational users to append, modify, and remove text using AI with advanced prompting and context management capabilities.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Technology Stack](#technology-stack)
3. [Project Structure](#project-structure)
4. [Core Components](#core-components)
5. [State Management](#state-management)
6. [User Interface Components](#user-interface-components)
7. [AI Interaction Flow](#ai-interaction-flow)
8. [Context Management](#context-management)
9. [Communication with Backend](#communication-with-backend)
10. [Key Features to Implement](#key-features-to-implement)
11. [Development Guidelines](#development-guidelines)

---

## Architecture Overview

The frontend is built as a modern React-based Single Page Application (SPA) that provides an intuitive interface for users to interact with AI for text manipulation tasks. The application follows a component-based architecture with centralized state management.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interface Layer                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   Chat View  │  │ Settings View│  │  Document Editor    │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                    State Management Layer                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │   Context API + React Query for API State Management │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                    Communication Layer                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  WebSocket   │  │  REST API    │  │  Message Queue      │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            ↕
                      Backend Services
```

---

## Technology Stack

### Core Technologies

1. **React 18+**
   - Primary UI framework
   - Hooks-based functional components
   - Context API for state management

2. **TypeScript**
   - Type-safe code
   - Enhanced IDE support
   - Better maintainability

3. **Vite**
   - Fast build tool
   - Hot Module Replacement (HMR)
   - Optimized production builds

4. **Tailwind CSS + Radix UI**
   - Tailwind CSS 4.0 for utility-first styling
   - Radix UI for accessible, unstyled components
   - Custom component library built on top

### Supporting Libraries

```json
{
  "dependencies": {
    "react": "^18.x",
    "react-dom": "^18.x",
    "typescript": "^5.x",
    "@tanstack/react-query": "^5.x",
    "@radix-ui/react-dialog": "^1.x",
    "@radix-ui/react-dropdown-menu": "^2.x",
    "@radix-ui/react-tooltip": "^1.x",
    "@radix-ui/react-progress": "^1.x",
    "@tailwindcss/vite": "^4.x",
    "axios": "^1.x",
    "clsx": "^2.x",
    "class-variance-authority": "^0.7.x",
    "react-markdown": "^9.x",
    "rehype-highlight": "^7.x",
    "dompurify": "^3.x",
    "date-fns": "^4.x",
    "zustand": "^4.x"
  }
}
```

---

## Project Structure

```
frontend/
├── public/
│   ├── index.html
│   └── assets/
│       ├── icons/
│       └── images/
├── src/
│   ├── components/
│   │   ├── common/           # Reusable UI components
│   │   │   ├── Button/
│   │   │   ├── Input/
│   │   │   ├── Modal/
│   │   │   └── Toast/
│   │   ├── chat/             # Chat interface components
│   │   │   ├── ChatView.tsx
│   │   │   ├── MessageList.tsx
│   │   │   ├── MessageItem.tsx
│   │   │   ├── ChatInput.tsx
│   │   │   └── SuggestionChips.tsx
│   │   ├── editor/           # Text editor components
│   │   │   ├── DocumentEditor.tsx
│   │   │   ├── Toolbar.tsx
│   │   │   ├── TextHighlight.tsx
│   │   │   └── VersionHistory.tsx
│   │   ├── settings/         # Settings and configuration
│   │   │   ├── SettingsView.tsx
│   │   │   ├── ApiConfiguration.tsx
│   │   │   └── UserPreferences.tsx
│   │   ├── context/          # Context manipulation tools
│   │   │   ├── ContextPanel.tsx
│   │   │   ├── MentionSelector.tsx
│   │   │   └── ContextManager.tsx
│   │   └── history/          # Task history
│   │       ├── HistoryView.tsx
│   │       └── HistoryItem.tsx
│   ├── context/              # React Context providers
│   │   ├── AppStateContext.tsx
│   │   ├── UserContext.tsx
│   │   └── ThemeContext.tsx
│   ├── hooks/                # Custom React hooks
│   │   ├── useChat.ts
│   │   ├── useTextManipulation.ts
│   │   ├── useContextManager.ts
│   │   └── useWebSocket.ts
│   ├── services/             # API and service layers
│   │   ├── api/
│   │   │   ├── client.ts
│   │   │   ├── chat.ts
│   │   │   ├── documents.ts
│   │   │   └── user.ts
│   │   ├── websocket/
│   │   │   └── wsClient.ts
│   │   └── storage/
│   │       └── localStorage.ts
│   ├── types/                # TypeScript type definitions
│   │   ├── api.ts
│   │   ├── chat.ts
│   │   ├── document.ts
│   │   └── user.ts
│   ├── utils/                # Utility functions
│   │   ├── formatting.ts
│   │   ├── validation.ts
│   │   └── textProcessing.ts
│   ├── styles/               # Global styles
│   │   ├── globals.css
│   │   └── themes.css
│   ├── App.tsx               # Root component
│   ├── main.tsx              # Entry point
│   └── vite-env.d.ts
├── package.json
├── tsconfig.json
├── vite.config.ts
└── tailwind.config.js
```

---

## Core Components

### 1. App Component (`App.tsx`)

The root component that manages the overall application state and routing.

```typescript
import React, { useState, useEffect } from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AppStateProvider } from './context/AppStateContext';
import ChatView from './components/chat/ChatView';
import HistoryView from './components/history/HistoryView';
import SettingsView from './components/settings/SettingsView';
import DocumentEditor from './components/editor/DocumentEditor';

type Tab = 'chat' | 'history' | 'settings' | 'editor';

const queryClient = new QueryClient();

const App: React.FC = () => {
  const [currentTab, setCurrentTab] = useState<Tab>('chat');
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  useEffect(() => {
    // Initialize application state
    // Load user preferences
    // Set up WebSocket connection
  }, []);

  const renderView = () => {
    switch (currentTab) {
      case 'chat':
        return <ChatView />;
      case 'history':
        return <HistoryView />;
      case 'settings':
        return <SettingsView />;
      case 'editor':
        return <DocumentEditor />;
      default:
        return <ChatView />;
    }
  };

  return (
    <QueryClientProvider client={queryClient}>
      <AppStateProvider>
        <div className={`app-container ${theme}`}>
          <nav className="tab-navigation">
            {/* Tab buttons */}
          </nav>
          <main className="main-content">
            {renderView()}
          </main>
        </div>
      </AppStateProvider>
    </QueryClientProvider>
  );
};

export default App;
```

### 2. ChatView Component

The main interface for AI text manipulation interactions.

```typescript
import React, { useState, useRef, useEffect } from 'react';
import { useChat } from '../../hooks/useChat';
import MessageList from './MessageList';
import ChatInput from './ChatInput';
import ContextPanel from '../context/ContextPanel';

interface ChatViewProps {
  documentId?: string;
}

const ChatView: React.FC<ChatViewProps> = ({ documentId }) => {
  const {
    messages,
    isLoading,
    sendMessage,
    editMessage,
    deleteMessage
  } = useChat(documentId);

  const [selectedContext, setSelectedContext] = useState<string[]>([]);
  const [inputValue, setInputValue] = useState('');

  const handleSendMessage = async (text: string, context?: any[]) => {
    await sendMessage({
      text,
      context: selectedContext,
      documentId
    });
    setInputValue('');
  };

  return (
    <div className="chat-view">
      <div className="chat-header">
        <h2>AI Text Assistant</h2>
        <ContextPanel 
          selectedContext={selectedContext}
          onContextChange={setSelectedContext}
        />
      </div>
      
      <MessageList 
        messages={messages}
        onEdit={editMessage}
        onDelete={deleteMessage}
      />
      
      <ChatInput 
        value={inputValue}
        onChange={setInputValue}
        onSend={handleSendMessage}
        isLoading={isLoading}
      />
    </div>
  );
};

export default ChatView;
```

### 3. DocumentEditor Component

Rich text editor for viewing and manipulating documents.

```typescript
import React, { useState, useEffect } from 'react';
import { useTextManipulation } from '../../hooks/useTextManipulation';

interface DocumentEditorProps {
  documentId: string;
}

const DocumentEditor: React.FC<DocumentEditorProps> = ({ documentId }) => {
  const {
    content,
    isLoading,
    saveDocument,
    applyAIChanges
  } = useTextManipulation(documentId);

  const [editorContent, setEditorContent] = useState('');
  const [selectedText, setSelectedText] = useState('');

  useEffect(() => {
    if (content) {
      setEditorContent(content);
    }
  }, [content]);

  const handleTextSelection = () => {
    const selection = window.getSelection();
    if (selection) {
      setSelectedText(selection.toString());
    }
  };

  const handleAIAction = async (action: 'append' | 'modify' | 'remove') => {
    // Trigger AI action on selected text
    await applyAIChanges(action, selectedText);
  };

  return (
    <div className="document-editor">
      <div className="editor-toolbar">
        <button onClick={() => handleAIAction('append')}>
          Append with AI
        </button>
        <button onClick={() => handleAIAction('modify')}>
          Modify with AI
        </button>
        <button onClick={() => handleAIAction('remove')}>
          Remove with AI
        </button>
      </div>
      
      <div 
        className="editor-content"
        contentEditable
        onMouseUp={handleTextSelection}
        dangerouslySetInnerHTML={{ __html: editorContent }}
      />
    </div>
  );
};

export default DocumentEditor;
```

---

## State Management

### AppStateContext

Global application state managed using React Context API.

```typescript
import React, { createContext, useContext, useState, useCallback } from 'react';

interface AppState {
  // User state
  user: User | null;
  isAuthenticated: boolean;
  
  // UI state
  theme: 'light' | 'dark';
  sidebarOpen: boolean;
  
  // Document state
  currentDocument: Document | null;
  documents: Document[];
  
  // AI state
  currentModel: string;
  apiConfiguration: APIConfig;
  
  // Chat state
  activeChat: string | null;
  chatHistory: ChatMessage[];
}

interface AppStateContextType extends AppState {
  // Actions
  setUser: (user: User | null) => void;
  setTheme: (theme: 'light' | 'dark') => void;
  toggleSidebar: () => void;
  setCurrentDocument: (doc: Document | null) => void;
  setApiConfiguration: (config: APIConfig) => void;
  updateChatHistory: (messages: ChatMessage[]) => void;
}

const AppStateContext = createContext<AppStateContextType | undefined>(undefined);

export const AppStateProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, setState] = useState<AppState>({
    user: null,
    isAuthenticated: false,
    theme: 'light',
    sidebarOpen: true,
    currentDocument: null,
    documents: [],
    currentModel: 'gpt-4',
    apiConfiguration: {},
    activeChat: null,
    chatHistory: []
  });

  const setUser = useCallback((user: User | null) => {
    setState(prev => ({ ...prev, user, isAuthenticated: !!user }));
  }, []);

  const setTheme = useCallback((theme: 'light' | 'dark') => {
    setState(prev => ({ ...prev, theme }));
    // Apply theme to document
    document.documentElement.setAttribute('data-theme', theme);
  }, []);

  const toggleSidebar = useCallback(() => {
    setState(prev => ({ ...prev, sidebarOpen: !prev.sidebarOpen }));
  }, []);

  const setCurrentDocument = useCallback((doc: Document | null) => {
    setState(prev => ({ ...prev, currentDocument: doc }));
  }, []);

  const setApiConfiguration = useCallback((config: APIConfig) => {
    setState(prev => ({ ...prev, apiConfiguration: config }));
  }, []);

  const updateChatHistory = useCallback((messages: ChatMessage[]) => {
    setState(prev => ({ ...prev, chatHistory: messages }));
  }, []);

  const value: AppStateContextType = {
    ...state,
    setUser,
    setTheme,
    toggleSidebar,
    setCurrentDocument,
    setApiConfiguration,
    updateChatHistory
  };

  return (
    <AppStateContext.Provider value={value}>
      {children}
    </AppStateContext.Provider>
  );
};

export const useAppState = () => {
  const context = useContext(AppStateContext);
  if (!context) {
    throw new Error('useAppState must be used within AppStateProvider');
  }
  return context;
};
```

---

## User Interface Components

### 1. Chat Components

#### MessageItem Component

```typescript
import React from 'react';
import ReactMarkdown from 'react-markdown';
import { formatDistanceToNow } from 'date-fns';

interface MessageItemProps {
  message: ChatMessage;
  onEdit?: (messageId: string, newText: string) => void;
  onDelete?: (messageId: string) => void;
}

const MessageItem: React.FC<MessageItemProps> = ({ message, onEdit, onDelete }) => {
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(message.text);

  const handleSaveEdit = () => {
    if (onEdit) {
      onEdit(message.id, editText);
    }
    setIsEditing(false);
  };

  return (
    <div className={`message-item ${message.role}`}>
      <div className="message-header">
        <span className="message-role">{message.role}</span>
        <span className="message-timestamp">
          {formatDistanceToNow(new Date(message.timestamp), { addSuffix: true })}
        </span>
      </div>
      
      <div className="message-content">
        {isEditing ? (
          <div className="message-edit">
            <textarea 
              value={editText}
              onChange={(e) => setEditText(e.target.value)}
            />
            <button onClick={handleSaveEdit}>Save</button>
            <button onClick={() => setIsEditing(false)}>Cancel</button>
          </div>
        ) : (
          <ReactMarkdown>{message.text}</ReactMarkdown>
        )}
      </div>
      
      {message.role === 'user' && (
        <div className="message-actions">
          <button onClick={() => setIsEditing(true)}>Edit</button>
          <button onClick={() => onDelete?.(message.id)}>Delete</button>
        </div>
      )}
    </div>
  );
};

export default MessageItem;
```

#### ChatInput Component

```typescript
import React, { useState, useRef } from 'react';

interface ChatInputProps {
  onSend: (text: string) => void;
  isLoading: boolean;
  placeholder?: string;
}

const ChatInput: React.FC<ChatInputProps> = ({ 
  onSend, 
  isLoading, 
  placeholder = "Ask AI to help with your text..." 
}) => {
  const [input, setInput] = useState('');
  const textareaRef = useRef<HTMLTextAreaElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (input.trim() && !isLoading) {
      onSend(input);
      setInput('');
      if (textareaRef.current) {
        textareaRef.current.style.height = 'auto';
      }
    }
  };

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      handleSubmit(e);
    }
  };

  const handleInput = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
    setInput(e.target.value);
    // Auto-resize textarea
    if (textareaRef.current) {
      textareaRef.current.style.height = 'auto';
      textareaRef.current.style.height = `${textareaRef.current.scrollHeight}px`;
    }
  };

  return (
    <form className="chat-input" onSubmit={handleSubmit}>
      <textarea
        ref={textareaRef}
        value={input}
        onChange={handleInput}
        onKeyDown={handleKeyDown}
        placeholder={placeholder}
        disabled={isLoading}
        rows={1}
      />
      <button 
        type="submit" 
        disabled={!input.trim() || isLoading}
      >
        {isLoading ? 'Sending...' : 'Send'}
      </button>
    </form>
  );
};

export default ChatInput;
```

### 2. Context Management Components

#### ContextPanel Component

```typescript
import React, { useState } from 'react';
import { useContextManager } from '../../hooks/useContextManager';

interface ContextPanelProps {
  selectedContext: string[];
  onContextChange: (context: string[]) => void;
}

const ContextPanel: React.FC<ContextPanelProps> = ({ 
  selectedContext, 
  onContextChange 
}) => {
  const { availableDocuments, recentSelections } = useContextManager();
  const [isOpen, setIsOpen] = useState(false);

  const toggleContext = (contextId: string) => {
    if (selectedContext.includes(contextId)) {
      onContextChange(selectedContext.filter(id => id !== contextId));
    } else {
      onContextChange([...selectedContext, contextId]);
    }
  };

  return (
    <div className="context-panel">
      <button 
        className="context-toggle"
        onClick={() => setIsOpen(!isOpen)}
      >
        Context ({selectedContext.length})
      </button>
      
      {isOpen && (
        <div className="context-menu">
          <h3>Add Context</h3>
          
          <div className="context-section">
            <h4>Documents</h4>
            {availableDocuments.map(doc => (
              <label key={doc.id} className="context-item">
                <input
                  type="checkbox"
                  checked={selectedContext.includes(doc.id)}
                  onChange={() => toggleContext(doc.id)}
                />
                <span>{doc.name}</span>
              </label>
            ))}
          </div>
          
          <div className="context-section">
            <h4>Recent Selections</h4>
            {recentSelections.map(item => (
              <div key={item.id} className="context-item">
                <button onClick={() => toggleContext(item.id)}>
                  {item.name}
                </button>
              </div>
            ))}
          </div>
        </div>
      )}
      
      {selectedContext.length > 0 && (
        <div className="selected-context">
          {selectedContext.map(id => (
            <span key={id} className="context-chip">
              {id}
              <button onClick={() => toggleContext(id)}>×</button>
            </span>
          ))}
        </div>
      )}
    </div>
  );
};

export default ContextPanel;
```

### 3. Settings Components

#### SettingsView Component

```typescript
import React, { useState } from 'react';
import { useAppState } from '../../context/AppStateContext';
import ApiConfiguration from './ApiConfiguration';
import UserPreferences from './UserPreferences';

const SettingsView: React.FC = () => {
  const { apiConfiguration, setApiConfiguration, theme, setTheme } = useAppState();
  const [activeTab, setActiveTab] = useState<'api' | 'preferences'>('api');

  return (
    <div className="settings-view">
      <h1>Settings</h1>
      
      <div className="settings-tabs">
        <button 
          className={activeTab === 'api' ? 'active' : ''}
          onClick={() => setActiveTab('api')}
        >
          API Configuration
        </button>
        <button 
          className={activeTab === 'preferences' ? 'active' : ''}
          onClick={() => setActiveTab('preferences')}
        >
          Preferences
        </button>
      </div>
      
      <div className="settings-content">
        {activeTab === 'api' && (
          <ApiConfiguration 
            config={apiConfiguration}
            onConfigChange={setApiConfiguration}
          />
        )}
        
        {activeTab === 'preferences' && (
          <UserPreferences 
            theme={theme}
            onThemeChange={setTheme}
          />
        )}
      </div>
    </div>
  );
};

export default SettingsView;
```

---

## AI Interaction Flow

### Request Flow

```
User Input → Input Validation → Context Assembly → 
API Request → WebSocket Connection → Stream Response → 
UI Update → Result Display
```

### Detailed Flow Diagram

```typescript
/**
 * AI Interaction Flow Implementation
 */

// 1. User submits a text manipulation request
const handleUserRequest = async (userInput: string, context: Context[]) => {
  // 2. Validate input
  if (!validateInput(userInput)) {
    showError('Invalid input');
    return;
  }

  // 3. Assemble context (documents, previous messages, etc.)
  const assembledContext = assembleContext(context);

  // 4. Create API request
  const request = {
    prompt: userInput,
    context: assembledContext,
    operation: detectOperation(userInput), // 'append', 'modify', 'remove'
    model: selectedModel,
    stream: true
  };

  // 5. Send request to backend
  const response = await apiClient.post('/api/chat/stream', request);

  // 6. Handle streaming response
  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  
  let buffer = '';
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    buffer += decoder.decode(value, { stream: true });
    
    // 7. Update UI with partial responses
    updateChatWithStreamedContent(buffer);
  }

  // 8. Finalize and display complete result
  finalizeResponse(buffer);
};
```

---

## Context Management

### Context Types

1. **Document Context**: Selected documents or text passages
2. **Conversation Context**: Previous chat messages
3. **User Context**: User preferences and settings
4. **Task Context**: Current operation and goals

### Context Assembly

```typescript
interface ContextItem {
  id: string;
  type: 'document' | 'conversation' | 'selection' | 'reference';
  content: string;
  metadata: {
    source: string;
    timestamp: number;
    relevance: number;
  };
}

class ContextManager {
  private contexts: Map<string, ContextItem> = new Map();
  private maxContextSize: number = 100000; // characters

  addContext(item: ContextItem): void {
    this.contexts.set(item.id, item);
    this.pruneContext();
  }

  removeContext(id: string): void {
    this.contexts.delete(id);
  }

  getContext(): ContextItem[] {
    return Array.from(this.contexts.values())
      .sort((a, b) => b.metadata.relevance - a.metadata.relevance);
  }

  private pruneContext(): void {
    const contexts = this.getContext();
    let totalSize = 0;
    const pruned: ContextItem[] = [];

    for (const context of contexts) {
      if (totalSize + context.content.length <= this.maxContextSize) {
        pruned.push(context);
        totalSize += context.content.length;
      } else {
        break;
      }
    }

    this.contexts = new Map(pruned.map(c => [c.id, c]));
  }

  getTotalSize(): number {
    return Array.from(this.contexts.values())
      .reduce((sum, c) => sum + c.content.length, 0);
  }
}
```

---

## Communication with Backend

### API Client Setup

```typescript
import axios, { AxiosInstance } from 'axios';

class APIClient {
  private client: AxiosInstance;
  private baseURL: string = process.env.VITE_API_URL || 'http://localhost:3000';

  constructor() {
    this.client = axios.create({
      baseURL: this.baseURL,
      timeout: 30000,
      headers: {
        'Content-Type': 'application/json'
      }
    });

    // Request interceptor for auth
    this.client.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('auth_token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor for error handling
    this.client.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 401) {
          // Handle unauthorized
          this.handleUnauthorized();
        }
        return Promise.reject(error);
      }
    );
  }

  private handleUnauthorized(): void {
    localStorage.removeItem('auth_token');
    window.location.href = '/login';
  }

  // Chat endpoints
  async sendMessage(message: ChatRequest): Promise<ChatResponse> {
    const response = await this.client.post('/api/chat', message);
    return response.data;
  }

  // Document endpoints
  async getDocuments(): Promise<Document[]> {
    const response = await this.client.get('/api/documents');
    return response.data;
  }

  async getDocument(id: string): Promise<Document> {
    const response = await this.client.get(`/api/documents/${id}`);
    return response.data;
  }

  async saveDocument(document: Document): Promise<Document> {
    const response = await this.client.post('/api/documents', document);
    return response.data;
  }

  // User endpoints
  async getUserProfile(): Promise<User> {
    const response = await this.client.get('/api/user/profile');
    return response.data;
  }
}

export const apiClient = new APIClient();
```

### WebSocket Client

```typescript
class WebSocketClient {
  private ws: WebSocket | null = null;
  private reconnectAttempts: number = 0;
  private maxReconnectAttempts: number = 5;
  private reconnectDelay: number = 1000;
  private messageHandlers: Map<string, Function> = new Map();

  connect(url: string): void {
    this.ws = new WebSocket(url);

    this.ws.onopen = () => {
      console.log('WebSocket connected');
      this.reconnectAttempts = 0;
    };

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    this.ws.onclose = () => {
      console.log('WebSocket disconnected');
      this.attemptReconnect(url);
    };
  }

  private handleMessage(data: any): void {
    const handler = this.messageHandlers.get(data.type);
    if (handler) {
      handler(data);
    }
  }

  on(type: string, handler: Function): void {
    this.messageHandlers.set(type, handler);
  }

  send(data: any): void {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(data));
    }
  }

  private attemptReconnect(url: string): void {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      setTimeout(() => {
        console.log(`Reconnecting... Attempt ${this.reconnectAttempts}`);
        this.connect(url);
      }, this.reconnectDelay * this.reconnectAttempts);
    }
  }

  disconnect(): void {
    if (this.ws) {
      this.ws.close();
      this.ws = null;
    }
  }
}

export const wsClient = new WebSocketClient();
```

---

## Key Features to Implement

### 1. Text Manipulation Operations

#### Append Operation
```typescript
const appendText = async (target: string, addition: string, position: 'before' | 'after') => {
  const request = {
    operation: 'append',
    target,
    content: addition,
    position,
    context: getActiveContext()
  };
  
  return await apiClient.post('/api/text/append', request);
};
```

#### Modify Operation
```typescript
const modifyText = async (target: string, modification: string) => {
  const request = {
    operation: 'modify',
    target,
    instruction: modification,
    context: getActiveContext()
  };
  
  return await apiClient.post('/api/text/modify', request);
};
```

#### Remove Operation
```typescript
const removeText = async (target: string, criteria: string) => {
  const request = {
    operation: 'remove',
    target,
    criteria,
    context: getActiveContext()
  };
  
  return await apiClient.post('/api/text/remove', request);
};
```

### 2. Real-time Collaboration

```typescript
const useCollaboration = (documentId: string) => {
  const [users, setUsers] = useState<User[]>([]);
  const [changes, setChanges] = useState<Change[]>([]);

  useEffect(() => {
    // Connect to collaboration room
    wsClient.send({
      type: 'join_room',
      documentId
    });

    // Listen for other users' changes
    wsClient.on('user_change', (data) => {
      applyRemoteChange(data.change);
    });

    // Broadcast local changes
    const broadcastChange = (change: Change) => {
      wsClient.send({
        type: 'broadcast_change',
        documentId,
        change
      });
    };

    return () => {
      wsClient.send({
        type: 'leave_room',
        documentId
      });
    };
  }, [documentId]);

  return { users, changes, broadcastChange };
};
```

### 3. Version History

```typescript
interface Version {
  id: string;
  content: string;
  timestamp: number;
  author: string;
  description: string;
}

const VersionHistory: React.FC<{ documentId: string }> = ({ documentId }) => {
  const [versions, setVersions] = useState<Version[]>([]);
  const [selectedVersion, setSelectedVersion] = useState<Version | null>(null);

  useEffect(() => {
    loadVersions();
  }, [documentId]);

  const loadVersions = async () => {
    const data = await apiClient.get(`/api/documents/${documentId}/versions`);
    setVersions(data);
  };

  const restoreVersion = async (versionId: string) => {
    await apiClient.post(`/api/documents/${documentId}/restore`, { versionId });
    // Reload document
  };

  return (
    <div className="version-history">
      <h3>Version History</h3>
      <ul>
        {versions.map(version => (
          <li key={version.id}>
            <div>{version.description}</div>
            <div>{new Date(version.timestamp).toLocaleString()}</div>
            <button onClick={() => restoreVersion(version.id)}>
              Restore
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

### 4. Smart Suggestions

```typescript
const useSuggestions = (documentId: string, selectedText: string) => {
  const [suggestions, setSuggestions] = useState<Suggestion[]>([]);

  useEffect(() => {
    if (selectedText) {
      fetchSuggestions();
    }
  }, [selectedText]);

  const fetchSuggestions = async () => {
    const response = await apiClient.post('/api/suggestions', {
      documentId,
      selectedText,
      context: getActiveContext()
    });
    setSuggestions(response.data);
  };

  const applySuggestion = async (suggestion: Suggestion) => {
    await apiClient.post('/api/text/apply-suggestion', {
      documentId,
      suggestionId: suggestion.id
    });
  };

  return { suggestions, applySuggestion };
};
```

---

## Development Guidelines

### 1. Code Style and Best Practices

- Use TypeScript for all components
- Follow React hooks patterns
- Implement proper error boundaries
- Use functional components over class components
- Implement proper memoization with `useMemo` and `useCallback`
- Keep components small and focused (single responsibility)

### 2. Performance Optimization

```typescript
// Memoize expensive computations
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);

// Memoize callbacks
const handleClick = useCallback(() => {
  // Handle click
}, [dependency]);

// Use React.memo for pure components
const MessageItem = React.memo<MessageItemProps>(({ message }) => {
  return <div>{message.text}</div>;
});
```

### 3. Error Handling

```typescript
class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean; error: Error | null }
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Log to error reporting service
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 4. Testing Strategy

```typescript
// Component test example
import { render, screen, fireEvent } from '@testing-library/react';
import { ChatInput } from './ChatInput';

describe('ChatInput', () => {
  it('should send message on submit', () => {
    const onSend = jest.fn();
    render(<ChatInput onSend={onSend} isLoading={false} />);
    
    const input = screen.getByPlaceholderText(/ask ai/i);
    const button = screen.getByRole('button', { name: /send/i });
    
    fireEvent.change(input, { target: { value: 'Test message' } });
    fireEvent.click(button);
    
    expect(onSend).toHaveBeenCalledWith('Test message');
  });

  it('should disable submit when loading', () => {
    render(<ChatInput onSend={jest.fn()} isLoading={true} />);
    
    const button = screen.getByRole('button', { name: /sending/i });
    expect(button).toBeDisabled();
  });
});
```

### 5. Accessibility

```typescript
// Ensure proper ARIA attributes
<button 
  aria-label="Send message"
  aria-disabled={isLoading}
  onClick={handleSend}
>
  <SendIcon />
</button>

// Use semantic HTML
<nav aria-label="Main navigation">
  <ul>
    <li><a href="/chat">Chat</a></li>
    <li><a href="/documents">Documents</a></li>
  </ul>
</nav>

// Keyboard navigation
const handleKeyDown = (e: React.KeyboardEvent) => {
  if (e.key === 'Escape') {
    closeModal();
  }
  if (e.key === 'Enter' && !e.shiftKey) {
    submitForm();
  }
};
```

### 6. Responsive Design

```css
/* Mobile-first approach */
.chat-view {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

/* Tablet */
@media (min-width: 768px) {
  .chat-view {
    flex-direction: row;
  }
  
  .sidebar {
    width: 300px;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .chat-view {
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

---

## Production Deployment Checklist

### Build Configuration

```javascript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@services': path.resolve(__dirname, './src/services'),
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom'],
          'ui-vendor': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
        }
      }
    }
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true
      }
    }
  }
});
```

### Environment Variables

```bash
# .env.production
VITE_API_URL=https://api.yourdomain.com
VITE_WS_URL=wss://ws.yourdomain.com
VITE_APP_VERSION=1.0.0
```

### Performance Monitoring

```typescript
// Add performance monitoring
import { reportWebVitals } from './utils/webVitals';

reportWebVitals((metric) => {
  // Send to analytics
  console.log(metric);
});
```

---

## Conclusion

This frontend documentation provides a comprehensive guide for building an AI-powered text manipulation website. The architecture is designed to be scalable, maintainable, and user-friendly, with a focus on real-time collaboration and intelligent text processing.

### Next Steps for Development:

1. Set up the development environment
2. Implement the core App structure and routing
3. Build out the ChatView and DocumentEditor components
4. Implement state management with Context API
5. Connect to backend APIs
6. Add real-time features with WebSocket
7. Implement text manipulation operations
8. Add comprehensive testing
9. Optimize for production
10. Deploy and monitor

For backend integration details, refer to the `backend_documentation.md` file.
