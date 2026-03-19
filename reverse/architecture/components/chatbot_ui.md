# Chatbot UI Component (Trippy-Chat Frontend)

## Overview
This document details the trippy-chat frontend implementation, which serves as the user interface for the chatbot application.

## Technology Stack
- **Framework**: Next.js 16.1.7
- **Language**: TypeScript
- **Styling**: Tailwind CSS v4
- **UI Components**: Radix UI primitives
- **Animation**: Framer Motion

## Key Features
1. Real-time chat interface
2. Knowledge base integration toggle
3. System prompt selection
4. Responsive design
5. Dark/Light mode support

## File Structure
```
trippy-chat/
├── src/
│   ├── app/
│   │   ├── page.tsx              # Main chat interface
│   │   ├── layout.tsx            # Root layout
│   │   └── api/                  # API routes
│   ├── components/
│   │   └── ui/                   # Reusable UI components
│   └── lib/                      # Utility functions and services
├── public/
│   └── system-prompts.json       # System prompt configurations
└── package.json
```

## Core Components
1. **Chat Interface** (`src/app/page.tsx`)
   - Main chat window
   - Message display and input handling
   - Integration with Llama API service

2. **Knowledge Base Toggle** (`src/components/ui/knowledge-base-toggle.tsx`)
   - Switch to enable/disable knowledge base
   - Visual indicator for KB status

3. **System Prompt Selector** (`src/components/ui/system-prompt-selector.tsx`)
   - Dropdown to select different AI personas
   - Persists user preference

4. **Avatar Component** (`src/components/ui/avatar.tsx`)
   - Displays user and assistant avatars

5. **Message Components** (`src/components/ui/trippy-message.tsx`, `trippychat-container.tsx`)
   - Individual message rendering
   - Chat container layout

## API Routes
1. **Knowledge Base Search** (`src/app/api/knowledge-base/search/route.ts`)
   - Searches the knowledgebase for relevant context
   - Returns formatted results for LLM consumption

2. **Knowledge Base Enhance** (`src/app/api/knowledge-base/enhance/route.ts`)
   - Enhances user queries with knowledge base context
   - Prepends relevant information to prompts

3. **System Prompts** (`src/app/api/system-prompts/route.ts`, `config/route.ts`)
   - Fetches available system prompts
   - Manages prompt configuration

## Services
1. **Llama API Service** (`src/lib/llama-api.ts`)
   - Handles communication with llama.cpp server
   - Manages system prompts and context
   - Implements caching for improved performance

2. **Knowledge Base Service** (`src/lib/knowledge-base.ts`)
   - Interfaces with knowledgebase search APIs
   - Formats retrieved information for LLM consumption

3. **System Prompt Manager** (`src/lib/system-prompt-manager.ts`)
   - Loads and manages system prompts from JSON
   - Tracks current active prompt

## Styling Approach
- Uses Tailwind CSS utility-first methodology
- Implements custom CSS variables for theme colors
- Leverages class-variance-authority for component variants
- Utilizes clsx for conditional class styling

## State Management
- Primarily uses React hooks (useState, useEffect)
- Context API for global state (theme, KB status)
- SWR or React Query for data fetching (inferred from patterns)

## Performance Optimizations
- Lazy loading of components
- Efficient re-rendering with React.memo
- Optimized image loading
- Code splitting via Next.js built-in mechanisms

## Security Considerations
- Input sanitization for chat messages
- CORS configuration for API endpoints
- Environment variable management for secrets
- Rate limiting considerations for API calls

## Accessibility Features
- ARIA labels for interactive elements
- Keyboard navigation support
- Screen reader friendly markup
- Sufficient color contrast ratios

## Deployment Configuration
- Next.js build optimizations
- Vercel deployment configuration (inferred)
- Environment-specific configurations