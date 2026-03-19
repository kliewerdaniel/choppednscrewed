# API Endpoints Specification

## Overview
This document specifies the API endpoints for the enhanced chatbot system, detailing the interface between the trippy-chat frontend and the backend services.

## Base URL
```
http://localhost:8000/api/v1
```

## Authentication
All endpoints require authentication via Bearer token:
```
Authorization: Bearer <token>
```
Token is obtained through the `/auth/login` endpoint.

## Endpoints

### Authentication
#### POST /auth/login
Authenticate user and receive access token.

**Request Body:**
```json
{
  "username": "string",
  "password": "string"
}
```

**Response:**
```json
{
  "access_token": "string",
  "token_type": "bearer",
  "expires_in": 3600
}
```

#### POST /auth/refresh
Refresh access token using refresh token.

**Request Body:**
```json
{
  "refresh_token": "string"
}
```

**Response:** Same as login endpoint.

### Chat
#### POST /chat/message
Send a message to the chatbot and receive a response.

**Request Body:**
```json
{
  "message": "string",
  "conversation_id": "string (optional)",
  "context": {
    "use_knowledge_base": boolean (default: true),
    "use_personas": boolean (default: true),
    "max_context_tokens": integer (default: 4096)
  },
  "parameters": {
    "temperature": number (default: 0.7),
    "max_tokens": integer (default: 2000),
    "top_p": number (default: 0.9)
  }
}
```

**Response:**
```json
{
  "message_id": "string",
  "conversation_id": "string",
  "content": "string",
  "role": "assistant",
  "timestamp": "ISO 8601 datetime",
  "metadata": {
    "personas_used": ["string"],
    "knowledge_sources": ["string"],
    "tokens_used": {
      "prompt": integer,
      "completion": integer,
      "total": integer
    },
    "processing_time_ms": integer
  }
}
```

#### GET /chat/conversations
List user's conversations.

**Query Parameters:**
- `limit`: integer (default: 20)
- `offset`: integer (default: 0)
- `sort_by`: "created_at" | "updated_at" (default: "updated_at")
- `sort_order`: "asc" | "desc" (default: "desc")

**Response:**
```json
{
  "conversations": [
    {
      "conversation_id": "string",
      "title": "string",
      "created_at": "ISO 8601 datetime",
      "updated_at": "ISO 8601 datetime",
      "message_count": integer,
      "last_message_preview": "string"
    }
  ],
  "pagination": {
    "total": integer,
    "limit": integer,
    "offset": integer,
    "has_more": boolean
  }
}
```

#### GET /chat/conversations/{conversation_id}
Get a specific conversation with messages.

**Query Parameters:**
- `limit`: integer (default: 50)
- `offset`: integer (default: 0)
- `sort_order`: "asc" | "desc" (default: "asc")

**Response:**
```json
{
  "conversation_id": "string",
  "title": "string",
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "messages": [
    {
      "message_id": "string",
      "content": "string",
      "role": "user" | "assistant",
      "timestamp": "ISO 8601 datetime",
      "metadata": {
        "personas_used": ["string"],
        "knowledge_sources": ["string"],
        "tokens_used": {
          "prompt": integer,
          "completion": integer,
          "total": integer
        }
      }
    }
  ],
  "pagination": {
    "total": integer,
    "limit": integer,
    "offset": integer,
    "has_more": boolean
  }
}
```

#### DELETE /chat/conversations/{conversation_id}
Delete a conversation.

**Response:** 204 No Content

### Knowledge Base
#### POST /knowledge/search
Search the knowledge base.

**Request Body:**
```json
{
  "query": "string",
  "filters": {
    "entity_types": ["string"],
    "relationship_types": ["string"],
    "topics": ["string"],
    "date_range": {
      "start": "ISO 8601 date",
      "end": "ISO 8601 date"
    },
    "confidence_min": number (default: 0.0)
  },
  "limit": integer (default: 10),
  "include_vectors": boolean (default: false)
}
```

**Response:**
```json
{
  "results": [
    {
      "id": "string",
      "type": "entity" | "relationship" | "topic",
      "content": "string",
      "score": number (0-1),
      "metadata": {
        "created_at": "ISO 8601 datetime",
        "updated_at": "ISO 8601 datetime",
        "confidence": number (0-1),
        "source": "string",
        "related_ids": ["string"]
      }
    }
  ],
  "query_info": {
    "processed_query": "string",
    "total_results": integer,
    "search_time_ms": integer
  }
}
```

#### POST /knowledge/entities
Create a new entity in the knowledge base.

**Request Body:**
```json
{
  "name": "string",
  "entity_type": "string",
  "properties": {
    "key": "value"
  },
  "aliases": ["string"],
  "confidence": number (default: 1.0),
  "source": "string (optional)"
}
```

**Response:**
```json
{
  "id": "string",
  "name": "string",
  "entity_type": "string",
  "properties": {
    "key": "value"
  },
  "aliases": ["string"],
  "confidence": number,
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "source": "string"
}
```

#### GET /knowledge/entities/{entity_id}
Get a specific entity.

**Response:**
```json
{
  "id": "string",
  "name": "string",
  "entity_type": "string",
  "properties": {
    "key": "value"
  },
  "aliases": ["string"],
  "confidence": number,
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "source": "string",
  "relationships": [
    {
      "id": "string",
      "type": "string",
      "target_entity_id": "string",
      "properties": {
        "key": "value"
      },
      "confidence": number,
      "created_at": "ISO 8601 datetime"
    }
  ]
}
```

#### PUT /knowledge/entities/{entity_id}
Update an existing entity.

**Request Body:** Same as creation but partial updates allowed.

**Response:** Same as get entity.

#### DELETE /knowledge/entities/{entity_id}
Delete an entity.

**Response:** 204 No Content

#### POST /knowledge/relationships
Create a new relationship between entities.

**Request Body:**
```json
{
  "source_entity_id": "string",
  "target_entity_id": "string",
  "relationship_type": "string",
  "properties": {
    "key": "value"
  },
  "confidence": number (default: 1.0),
  "source": "string (optional)"
}
```

**Response:**
```json
{
  "id": "string",
  "source_entity_id": "string",
  "target_entity_id": "string",
  "relationship_type": "string",
  "properties": {
    "key": "value"
  },
  "confidence": number,
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "source": "string"
}
```

### Personas
#### GET /personas
List available personas.

**Query Parameters:**
- `active_only`: boolean (default: false)
- `expertise`: ["string"] (filter by expertise)
- `limit`: integer (default: 50)
- `offset`: integer (default: 0)

**Response:**
```json
{
  "personas": [
    {
      "persona_id": "string",
      "name": "string",
      "description": "string",
      "traits": {
        "trait_name": integer (1-9)
      },
      "expertise": ["string"],
      "activation_cost": number,
      "historical_performance": {
        "metric_name": number
      },
      "metadata": {
        "key": "value"
      },
      "status": "active" | "stable" | "experimental" | "pruned",
      "created_at": "ISO 8601 datetime",
      "updated_at": "ISO 8601 datetime"
    }
  ],
  "pagination": {
    "total": integer,
    "limit": integer,
    "offset": integer,
    "has_more": boolean
  }
}
```

#### POST /personas
Create a new persona.

**Request Body:**
```json
{
  "persona_id": "string",
  "name": "string",
  "description": "string",
  "traits": {
    "trait_name": integer (1-9)
  },
  "expertise": ["string"],
  "activation_cost": number,
  "historical_performance": {
    "metric_name": number
  },
  "metadata": {
    "key": "value"
  }
}
```

**Response:** Same as get persona.

#### GET /personas/{persona_id}
Get a specific persona.

**Response:** Same as list item.

#### PUT /personas/{persona_id}
Update a persona.

**Request Body:** Partial updates allowed.

**Response:** Same as get persona.

#### DELETE /personas/{persona_id}
Delete a persona (marks as pruned, doesn't actually delete).

**Response:** 204 No Content

#### POST /personas/{persona_id}/activate
Activate a persona for use.

**Response:**
```json
{
  "persona_id": "string",
  "status": "active",
  "activated_at": "ISO 8601 datetime"
}
```

#### POST /personas/{persona_id}/deactivate
Deactivate a persona.

**Response:**
```json
{
  "persona_id": "string",
  "status": "stable", // or previous status
  "deactivated_at": "ISO 8601 datetime"
}
```

### Tools (Mistral-Vibe Integration)
#### POST /tools/execute
Execute a tool command via natural language.

**Request Body:**
```json
{
  "command": "string", // Natural language description of desired action
  "context": {
    "working_directory": "string (optional)",
    "file_constraints": ["string"] (optional, glob patterns),
    "safety_level": "low" | "medium" | "high" (default: "medium")
  },
  "parameters": {
    "auto_approve": boolean (default: false),
    "max_execution_time_ms": integer (default: 30000)
  }
}
```

**Response:**
```json
{
  "execution_id": "string",
  "status": "completed" | "failed" | "timeout",
  "result": {
    "type": "file_change" | "command_output" | "information",
    "content": "varies by type"
  },
  "logs": ["string"],
  "artifacts": [
    {
      "type": "file" | "output" | "image",
      "path": "string",
      "content": "string or base64"
    }
  ],
  "execution_time_ms": integer,
  "safety_checks_passed": boolean
}
```

#### GET /tools/executions/{execution_id}
Get details of a specific tool execution.

**Response:** Same as execute endpoint response.

#### GET /tools/available
List available tools and their capabilities.

**Response:**
```json
{
  "tools": [
    {
      "name": "string",
      "description": "string",
      "parameters": {
        "parameter_name": {
          "type": "string" | "number" | "boolean" | "array" | "object",
          "description": "string",
          "required": boolean,
          "default": "any"
        }
      },
      "return_type": "string",
      "examples": ["string"],
      "safety_level": "low" | "medium" | "high"
    }
  ]
}
```

### System
#### GET /system/health
Check system health and component status.

**Response:**
```json
{
  "status": "healthy" | "degraded" | "unhealthy",
  "timestamp": "ISO 8601 datetime",
  "components": {
    "llama_cpp": {
      "status": "healthy" | "degraded" | "unhealthy",
      "details": {
        "model_loaded": string,
        "context_size": integer,
        "gpu_utilization": number (0-1),
        "average_response_time_ms": integer
      }
    },
    "knowledge_base": {
      "status": "healthy" | "degraded" | "unhealthy",
      "details": {
        "entity_count": integer,
        "relationship_count": integer,
        "last_updated": "ISO 8601 datetime",
        "query_avg_time_ms": integer
      }
    },
    "persona_system": {
      "status": "healthy" | "degraded" | "unhealthy",
      "details": {
        "active_personas": integer,
        "total_personas": integer,
        "avg_performance_score": number
      }
    }
  }
}
```

#### GET /system/metrics
Get system performance metrics.

**Response:**
```json
{
  "timestamp": "ISO 8601 datetime",
  "metrics": {
    "requests_per_minute": number,
    "average_response_time_ms": number,
    "error_rate": number (0-1),
    "knowledge_base": {
      "queries_per_minute": number,
      "cache_hit_rate": number (0-1),
      "storage_size_mb": number
    },
    "llama_cpp": {
      "tokens_per_second": number,
      "average_context_utilization": number (0-1),
      "batch_efficiency": number (0-1)
    }
  }
}
```

## WebSocket Endpoints

### WS /ws/chat/{conversation_id}
Real-time chat communication.

**Messages from Client to Server:**
```json
{
  "type": "message",
  "content": "string",
  "metadata": {
    "use_knowledge_base": boolean,
    "use_personas": boolean,
    "parameters": {
      "temperature": number,
      "max_tokens": integer
    }
  }
}
```

**Messages from Server to Client:**
```json
{
  "type": "message_start",
  "message_id": "string"
}

// Streaming content chunks
{
  "type": "content_chunk",
  "message_id": "string",
  "content": "string"
}

// Final message metadata
{
  "type": "message_complete",
  "message_id": "string",
  "metadata": {
    "personas_used": ["string"],
    "knowledge_sources": ["string"],
    "tokens_used": {
      "prompt": integer,
      "completion": integer,
      "total": integer
    },
    "processing_time_ms": integer
  }
}

// Error messages
{
  "type": "error",
  "message": "string",
  "code": "string"
}
```

### WS /ws/tools/{execution_id}
Real-time tool execution updates.

**Messages from Server to Client:**
```json
{
  "type": "execution_start",
  "execution_id": "string"
}

// Progress updates
{
  "type": "execution_progress",
  "execution_id": "string",
  "percentage": number (0-100),
  "message": "string"
}

// Log messages
{
  "type": "execution_log",
  "execution_id": "string",
  "level": "info" | "warning" | "error",
  "message": "string",
  "timestamp": "ISO 8601 datetime"
}

// Final result
{
  "type": "execution_complete",
  "execution_id": "string",
  "status": "completed" | "failed" | "timeout",
  "result": {
    // Same as REST endpoint result
  }
}

// Error messages
{
  "type": "execution_error",
  "execution_id": "string",
  "message": "string",
  "code": "string"
}
```

## Error Handling

### Error Response Format
All endpoints return errors in this format:
```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": {
      // Additional error-specific information
    }
  }
}
```

### Common Error Codes
- `VALIDATION_ERROR`: Invalid request parameters
- `AUTHENTICATION_ERROR`: Missing or invalid credentials
- `AUTHORIZATION_ERROR`: Insufficient permissions
- `RESOURCE_NOT_FOUND`: Requested resource doesn't exist
- `RESOURCE_CONFLICT`: Conflict with existing resource
- `RATE_LIMIT_EXCEEDED`: Too many requests
- `INTERNAL_ERROR`: Unexpected server error
- `SERVICE_UNAVAILABLE`: Downstream service unavailable
- `TIMEOUT_ERROR`: Operation timed out

## Rate Limiting
- Auth endpoints: 10 requests/minute per IP
- General API: 60 requests/minute per user
- Tool execution: 10 requests/minute per user
- WebSocket connections: 5 concurrent per user

## Versioning
API versions are specified in the URL path (`/api/v1/`). Breaking changes will increment the version number. Minor changes that don't break compatibility will be versioned as v1.1, v1.2, etc.

## Security Considerations
1. All data transmitted over HTTPS/WSS in production
2. Input validation and sanitization on all endpoints
3. SQL injection prevention through parameterized queries
4. XSS prevention through proper output encoding
5. CSRF protection for state-changing operations
6. Rate limiting to prevent abuse
7. File access restrictions for tool execution
8. Sandboxed execution environment for potentially unsafe operations