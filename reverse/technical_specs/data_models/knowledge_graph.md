# Knowledge Graph Data Models

## Overview
This document defines the data models for the unified knowledge graph system that serves as both long-term memory (knowledgebase) and short-term context.

## Core Entities

### Entity
Represents a distinct object, concept, or item in the knowledge domain.

```json
{
  "id": "string (UUID)",
  "name": "string",
  "entity_type": "string",
  "properties": {
    "key": "value (string | number | boolean | array | object)"
  },
  "aliases": ["string"],
  "confidence": "number (0.0-1.0)",
  "source": "string (optional, e.g., 'user_conversation', 'document', 'api')",
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "created_by": "string (user_id or system)",
  "tags": ["string"]
}
```

#### Entity Types (examples)
- `person`: A human individual
- `organization`: A company, institution, or group
- `location`: A geographical place
- `concept`: An abstract idea or principle
- `event`: Something that happens at a specific time and place
- `product`: A tangible or intangible offering
- `technology`: A tool, technique, or system
- `document`: A piece of written, printed, or electronic matter

### Relationship
Represents a connection between two entities.

```json
{
  "id": "string (UUID)",
  "source_entity_id": "string (UUID)",
  "target_entity_id": "string (UUID)",
  "relationship_type": "string",
  "properties": {
    "key": "value (string | number | boolean | array | object)"
  },
  "confidence": "number (0.0-1.0)",
  "source": "string (optional)",
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "created_by": "string (user_id or system)",
  "tags": ["string"],
  "valid_from": "ISO 8601 datetime (optional)",
  "valid_to": "ISO 8601 datetime (optional)"
}
```

#### Relationship Types (examples)
- `WORKS_FOR`: Person → Organization
- `LOCATED_IN`: Entity → Location
- `PART_OF`: Entity → Entity (composition)
- `SUBCLASS_OF`: Concept → Concept (taxonomy)
- `RELATED_TO`: Entity → Entity (generic association)
- `CREATED_BY`: Entity → Person
- `MENTIONED_IN`: Entity → Document
- `DEVELOPED_BY`: Technology → Organization
- `OCCURRED_AT`: Event → Location
- `HAS_SYMPTOM`: Disease → Symptom (medical)

### Topic
Represents a categorical grouping for organizing knowledge.

```json
{
  "id": "string (UUID)",
  "name": "string",
  "description": "string (optional)",
  "parent_topic_id": "string (UUID, optional)",
  "properties": {
    "key": "value"
  },
  "confidence": "number (0.0-1.0)",
  "source": "string (optional)",
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "created_by": "string (user_id or system)",
  "tags": ["string"]
}
```

### Conversation
Represents a chat session between a user and the system.

```json
{
  "id": "string (UUID)",
  "title": "string",
  "user_id": "string",
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "ended_at": "ISO 8601 datetime (optional)",
  "message_count": "integer",
  "metadata": {
    "key": "value"
  },
  "tags": ["string"]
}
```

### Message
Represents a single message within a conversation.

```json
{
  "id": "string (UUID)",
  "conversation_id": "string (UUID)",
  "content": "string",
  "role": "user" | "assistant" | "system",
  "created_at": "ISO 8601 datetime",
  "metadata": {
    "personas_used": ["string"],
    "knowledge_sources": ["string (entity IDs)"],
    "tokens_used": {
      "prompt": "integer",
      "completion": "integer",
      "total": "integer"
    },
    "processing_time_ms": "integer"
  },
  "attachments": [
    {
      "id": "string",
      "type": "image" | "document" | "audio" | "video",
      "url": "string",
      "description": "string (optional)"
    }
  ]
}
```

### Persona
Represents an AI agent with specific traits, expertise, and behavioral patterns.

```json
{
  "persona_id": "string (UUID)",
  "name": "string",
  "description": "string",
  "traits": {
    "trait_name": "integer (1-9)" // e.g., "creativity": 8, "analytical": 7
  },
  "expertise": ["string"], // e.g., ["machine_learning", "web_development"]
  "activation_cost": "number (0.0-1.0)", // Relative cost to activate this persona
  "historical_performance": {
    "metric_name": "number" // e.g., "relevance_score": 0.85, "user_satisfaction": 4.2
  },
  "metadata": {
    "key": "value"
  },
  "status": "active" | "stable" | "experimental" | "pruned",
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "created_by": "string (user_id or system)"
}
```

### User
Represents a system user.

```json
{
  "id": "string (UUID)",
  "username": "string",
  "email": "string",
  "display_name": "string",
  "avatar_url": "string (optional)",
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "last_login_at": "ISO 8601 datetime (optional)",
  "is_active": "boolean",
  "roles": ["string"], // e.g., ["admin", "user", "developer"]
  "preferences": {
    "key": "value"
  },
  "metadata": {
    "key": "value"
  }
}
```

## Knowledge Graph Extensions

### Episodic Memory
Stores specific interactions or experiences for personalization.

```json
{
  "id": "string (UUID)",
  "user_id": "string (UUID)",
  "conversation_id": "string (UUID)",
  "summary": "string",
  "key_entities": ["string (entity IDs)"],
  "key_relationships": ["string (relationship IDs)"],
  "emotional_valence": "number (-1.0 to 1.0)", // Negative to positive
  "importance_score": "number (0.0-1.0)",
  "created_at": "ISO 8601 datetime",
  "accessed_count": "integer",
  "last_accessed_at": "ISO 8601 datetime",
  "tags": ["string"]
}
```

### Semantic Memory
Represents organized knowledge that is not tied to specific experiences.

This is the core knowledge graph itself, composed of entities, relationships, and topics.

### Procedural Memory
Stores knowledge about how to perform tasks or execute procedures.

```json
{
  "id": "string (UUID)",
  "name": "string",
  "description": "string",
  "steps": [
    {
      "step_number": "integer",
      "description": "string",
      "required_entities": ["string (entity IDs)"],
      "required_tools": ["string (tool IDs)"]
    }
  ],
  "prerequisites": ["string (knowledge IDs)"],
  "difficulty_level": "string (beginner|intermediate|advanced)",
  "estimated_time_minutes": "integer",
  "confidence": "number (0.0-1.0)",
  "source": "string (optional)",
  "created_at": "ISO 8601 datetime",
  "updated_at": "ISO 8601 datetime",
  "created_by": "string (user_id or system)",
  "tags": ["string"]
}
```

## Indexing and Search Models

### Search Query
Represents a request to search the knowledge graph.

```json
{
  "query_text": "string",
  "query_type": "keyword" | "semantic" | "hybrid",
  "filters": {
    "entity_types": ["string"],
    "relationship_types": ["string"],
    "topics": ["string"],
    "confidence_min": "number (0.0-1.0)",
    "date_range": {
      "start": "ISO 8601 datetime",
      "end": "ISO 8601 datetime"
    },
    "entity_ids": ["string (UUID)"],
    "relationship_ids": ["string (UUID)"]
  },
  "limit": "integer",
  "offset": "integer",
  "include_vectors": "boolean",
  "include_paths": "boolean", // Include relationship paths in results
  "max_path_length": "integer (default: 3)"
}
```

### Search Result
Represents a single result from a knowledge graph search.

```json
{
  "id": "string (UUID)",
  "result_type": "entity" | "relationship" | "topic" | "path",
  "content": "string", // Textual representation
  "score": "number (0.0-1.0)", // Relevance score
  "metadata": {
    // For entities
    "entity_type": "string",
    "properties": {
      "key": "value"
    },
    "aliases": ["string"],
    "confidence": "number (0.0-1.0)",
    // For relationships
    "source_entity": {
      "id": "string",
      "name": "string"
    },
    "target_entity": {
      "id": "string",
      "name": "string"
    },
    "relationship_type": "string",
    "properties": {
      "key": "value"
    },
    "confidence": "number (0.0-1.0)",
    // For paths
    "path": [
      {
        "entity": {
          "id": "string",
          "name": "string",
          "entity_type": "string"
        },
        "relationship": {
          "type": "string",
          "properties": {
            "key": "value"
          }
        }
      }
    ],
    "path_length": "integer"
  },
  "highlighted_text": "string (optional)" // Query terms highlighted
}
```

## Caching Models

### Query Cache
Stores results of frequent queries for quick retrieval.

```json
{
  "cache_key": "string (hash of query parameters)",
  "query": "Search Query (as above)",
  "results": ["Search Result (as above)"],
  "created_at": "ISO 8601 datetime",
  "expires_at": "ISO 8601 datetime",
  "hit_count": "integer",
  "last_hit_at": "ISO 8601 datetime"
}
```

### Embedding Cache
Stores vector embeddings for entities and text to avoid recomputation.

```json
{
  "cache_key": "string (hash of input text + model version)",
  "input_text": "string",
  "model_version": "string",
  "embedding": ["number"], // Vector of floats
  "created_at": "ISO 8601 datetime",
  "expires_at": "ISO 8601 datetime"
}
```