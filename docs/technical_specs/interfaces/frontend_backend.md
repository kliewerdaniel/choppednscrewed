# Frontend-Backend Interface Specification

## Overview
This document specifies the interface between the trippy-chat frontend and the backend services, including data structures, communication patterns, and error handling.

## Communication Protocols

### REST API
- Primary communication method for non-real-time operations
- JSON over HTTP/HTTPS
- Standard HTTP methods (GET, POST, PUT, DELETE)
- Status codes follow REST conventions

### WebSocket
- Used for real-time, bidirectional communication
- Ideal for streaming responses and live updates
- Falls back to polling if WebSocket unavailable

## Message Formats

### Chat Messages
Structures exchanged between frontend and backend for chat operations.

#### User Message (Frontend → Backend)
```typescript
interface UserMessage {
  id: string; // UUID
  content: string;
  timestamp: string; // ISO 8601
  metadata?: {
    messageId: string; // For tracking
    clientTimestamp: string; // When user sent it
  }
}
```

#### Assistant Message (Backend → Frontend)
```typescript
interface AssistantMessage {
  id: string; // UUID
  content: string;
  timestamp: string; // ISO 8601
  metadata: {
    personasUsed: string[]; // Persona IDs
    knowledgeSources: string[]; // Entity/relationship IDs
    tokensUsed: {
      prompt: number;
      completion: number;
      total: number;
    };
    processingTimeMs: number;
    modelUsed: string; // e.g., "qwen3.5-9b"
  };
  attachments?: Array<{
    id: string;
    type: 'image' | 'document' | 'code';
    url: string;
    preview?: string; // Base64 or URL
  }>;
  suggestions?: string[]; // Follow-up questions or actions
}
```

#### Streaming Chunks (WebSocket)
```typescript
interface StreamChunk {
  type: 'content' | 'metadata' | 'error' | 'done';
  messageId: string; // Links to parent message
  sequence: number; // Order in stream
  content?: string; // For 'content' type
  metadata?: {
    personasUsed?: string[];
    knowledgeSources?: string[];
    tokensUsed?: {
      prompt: number;
      completion: number;
      total: number;
    };
  };
  error?: {
    code: string;
    message: string;
  };
  isFinal: boolean; // True for 'done' type
}
```

## State Synchronization

### Conversation State
Shared state that both frontend and backend maintain.

```typescript
interface ConversationState {
  conversationId: string;
  title: string;
  createdAt: string; // ISO 8601
  updatedAt: string; // ISO 8601
  messageCount: number;
  userId: string;
  isActive: boolean;
  metadata: Record<string, any>;
  lastReadByUser?: string; // ISO 8601
  lastReadByAssistant?: string; // ISO 8601
}
```

### User Preferences
Settings that persist across sessions.

```typescript
interface UserPreferences {
  theme: 'light' | 'dark' | 'system';
  language: string; // e.g., 'en', 'es'
  timezone: string; // IANA timezone
  notifications: {
    email: boolean;
    desktop: boolean;
    sound: boolean;
  };
  chat: {
    autoScroll: boolean;
    showTimestamps: boolean;
    density: 'compact' | 'comfortable';
    enableKnowledgeBase: boolean;
    enablePersonas: boolean;
    defaultTemperature: number;
    defaultMaxTokens: number;
  };
  personaSettings: {
    activePersonas: string[]; // Persona IDs
    traitWeights: Record<string, number>; // Trait name -> weight
  };
}
```

## API Interaction Patterns

### Sending a Message
1. Frontend adds message to local state optimistically
2. Frontend sends POST /chat/message to backend
3. Backend validates, processes, and stores message
4. Backend returns message with ID and metadata
5. Frontend updates local message with server-provided ID
6. If streaming enabled, backend initiates WebSocket stream
7. Frontend displays chunks as they arrive
8. Backend sends completion signal via WebSocket
9. Frontend updates final message metadata

### Loading Conversation History
1. Frontend sends GET /chat/conversations/{id}?limit=N&offset=M
2. Backend returns paginated messages
3. Frontend inserts messages into local store
4. Frontend sends loading state indicators
5. On scroll near top, frontend loads more if available

### Searching Knowledge Base
1. Frontend debounces user input (300ms)
2. Frontend sends POST /knowledge/search
3. Backend returns ranked results
4. Frontend displays results in sidebar or inline
5. User can click to pin result to context
6. Pinned results included in next chat request

### Executing Tools
1. Frontend sends POST /tools/execute with natural language
2. Backend parses intent and selects appropriate tool
3. Backend validates permissions and safety
4. Backend executes tool in sandbox
5. Frontend receives real-time updates via WebSocket
6. Frontend displays results, logs, and any file changes
7. User can approve, modify, or reject changes
8. Approved changes trigger knowledge base updates

## Error Handling

### Error Propagation
Backend errors follow standard HTTP status codes:
- 400: Bad Request (validation errors)
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 409: Conflict
- 429: Too Many Requests
- 500: Internal Server Error
- 503: Service Unavailable

Frontend maps these to user-friendly messages:
- Validation errors: Show inline field errors
- Auth errors: Redirect to login
- 404: Show "Not found" page
- 409: Show conflict resolution UI
- 429: Show rate limit notice with retry-after
- 5xx: Show generic error with retry option

### Error Recovery
1. Network errors: Retry with exponential backoff
2. Service errors: Show error boundary with retry button
3. Validation errors: Prevent submission, show fixes needed
4. Partial failures: Show what succeeded vs failed
5. Data conflicts: Show merge conflict resolution UI

## Security Considerations

### Data Validation
All data received from frontend must be validated:
- Type checking (string, number, boolean, etc.)
- Length limits (prevent DoS via huge payloads)
- Format validation (email, URLs, UUIDs)
- Content sanitization (prevent XSS, injection)
- Business rule validation (e.g., date ranges)

### Authentication
- JWT tokens for stateless authentication
- Short-lived access tokens (15-30 minutes)
- Refresh token rotation for long sessions
- HTTPS-only transmission of tokens
- Proper CORS configuration

### Authorization
- Role-based access control (RBAC)
- Resource-level permissions (conversations, KB entries)
- User isolation (users can only access their own data)
- Admin override capabilities
- Audit logging of sensitive operations

### Input Sanitization
All user-generated content must be sanitized:
- HTML escaping for display contexts
- SQL parameterization for database queries
- Command injection prevention for tool execution
- Path traversal prevention for file operations
- Schema validation for structured data

## Performance Considerations

### Payload Optimization
- Use compact JSON representations
- Implement pagination for large datasets
- Use delta updates for state synchronization
- Compress large payloads (gzip/brotli)
- Field selection (allow clients to request specific fields)

### Caching Strategies
- Client-side caching of static resources
- HTTP caching headers for API responses
- WebSocket message deduplication
- Optimistic UI updates to reduce perceived latency
- Prefetching of likely-needed data

### Connection Management
- WebSocket heartbeat/ping to detect disconnections
- Automatic reconnection with exponential backoff
- Request queuing during connection loss
- Offline mode with local queueing and sync
- Connection pooling for HTTP requests

## Versioning and Compatibility

### API Versioning
- Version in URL path: `/api/v1/`
- Backward compatibility within major versions
- Deprecation notices for upcoming changes
- Clear migration guides for breaking changes
- Version negotiation via headers (Accept-Version)

### Feature Flags
- Runtime toggling of experimental features
- Gradual rollout capabilities
- A/B testing infrastructure
- Emergency kill switches for problematic features
- User opt-in/opt-out for beta features

## Internationalization

### Text Direction
- Support for LTR (English, etc.) and RTL (Arabic, Hebrew) layouts
- Locale-aware number and date formatting
- Proper handling of Unicode characters
- Font fallback for unsupported scripts

### Localization
- Externalized strings for translation
- Context-aware translation (same word, different meanings)
- Pluralization handling for different languages
- Date/time formatting per locale
- Currency and number formatting

## Accessibility

### Screen Reader Support
- ARIA labels for interactive elements
- Live regions for dynamic content updates
- Proper heading hierarchy
- Landmark regions for navigation
- Skip links for keyboard navigation

### Keyboard Navigation
- Tab order logical and predictable
- Escape closes modals and dropdowns
- Enter activates buttons and links
- Arrow keys navigate menus and grids
- Shortcuts for common actions (documented)

### Visual Accessibility
- Sufficient color contrast ratios
- Resizable text without breaking layout
- No reliance on color alone for meaning
- Focus indicators visible and clear
- Reduced motion preferences respected

## Monitoring and Observability

### Frontend Telemetry
- Performance metrics (FCP, LCP, CLS)
- Error tracking (JavaScript exceptions)
- Usage analytics (feature adoption)
- Performance timings for key interactions
- Memory usage monitoring

### Backend Integration
- Correlation IDs for request tracing
- Structured logging for debugging
- Performance metrics collection
- Health check endpoints
- Alerting on error rates and latency