# Environment Configuration

## Overview
This document specifies the environment variables and configuration options for the enhanced chatbot system across different deployment environments.

## Configuration Loading Order
Configuration is loaded in the following order (later sources override earlier ones):
1. Default values (hardcoded in code)
2. `.env` file in project root
3. Environment variables
4. Command-line arguments
5. External configuration service (Consul, etcd, etc.) - optional

## Environment Files

### .env.template
Template file showing all available configuration options:
```env
# === APPLICATION SETTINGS ===
NODE_ENV=development
LOG_LEVEL=info
PORT=3000
API_PREFIX=/api/v1

# === DATABASE CONFIGURATION ===
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=chatbot
POSTGRES_USER=chatbot_user
POSTGRES_PASSWORD=changeme
POSTGRES_POOL_MAX=20
POSTGRES_IDLE_TIMEOUT_MS=30000

REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0
REDIS_TLS=FALSE

# Knowledge Graph Storage (Neo4j example)
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=changeme
NEO4J_MAX_CONNECTION_POOL_SIZE=50
NEO4J_CONNECTION_TIMEOUT=30000
NEO4J_MAX_TRANSACTION_RETRY_TIME=30000

# === LLAMA.CPP CONFIGURATION ===
LLAMA_MODEL_PATH=/app/models/Qwen3.5-9B-ultimate-irrefusable-heretic-Q4_K_M.gguf
LLAMA_SERVER_HOST=localhost
LLAMA_SERVER_PORT=8080
LLAMA_CONTEXT_SIZE=8192
LLAMA_GPU_LAYERS=35
LLAMA_BATCH_SIZE=8
LLAMA_PARALLEL_SEQUENCES=4
LLAMA_THREADS=8
LLAMA_SEED=-1  # -1 for random
LLAMA_F16_KV=TRUE
LLAMA_LOG_DISABLE=FALSE

# === AUTHENTICATION ===
JWT_SECRET_KEY=your-super-secret-jwt-key-change-in-production
JWT_EXPIRES_IN=3600  # seconds
JWT_REFRESH_EXPIRES_IN=604800  # 7 days in seconds
BCRYPT_SALT_ROUNDS=12

# === EXTERNAL SERVICES ===
# For proactive knowledge expansion
WIKIPEDIA_API_RATE_LIMIT=50  # requests per minute
WIKIPEDIA_CACHE_TTL=3600  # seconds
GOOGLE_SEARCH_API_KEY=  # Optional, for web search
GOOGLE_SEARCH_CX=  # Optional, Custom Search ID

# === FEATURE FLAGS ===
FEATURE_KNOWLEDGE_BASE=enabled
FEATURE_PERSONAS=enabled
FEATURE_TOOLS=enabled
FEATURE_STREAMING=enabled
FEATURE_WEBSOCKETS=enabled
FEATURE_FILE_ATTACHMENTS=enabled
FEATURE_CODE_EXECUTION=disabled  # Enable with caution
FEATURE_INTERNET_ACCESS=disabled  # For tool internet access

# === RATE LIMITING ===
RATE_LIMIT_AUTH=10  # per minute
RATE_LIMIT_API=60  # per minute
RATE_LIMIT_TOOLS=10  # per minute
RATE_LIMIT_WS_CONNECTIONS=5  # concurrent per user

# === CACHING ===
CACHE_DEFAULT_TTL=300  # 5 minutes
CACHE_QUERY_RESULTS=600  # 10 minutes
CACHE_EMBEDDINGS=3600  # 1 hour
CACHE_PERSONAS=1800  # 30 minutes

# === MONITORING ===
ENABLE_METRICS=true
METRICS_PORT=9090
ENABLE_TRACING=false
TRACING_ENDPOINT=  # e.g., http://jaeger-collector:14268/api/traces
LOG_FORMAT=json  # or text

# === FILE STORAGE ===
STORAGE_TYPE=local  # local, s3, gcs, azure
LOCAL_STORAGE_PATH=./uploads
AWS_ACCESS_KEY_ID=  # For S3
AWS_SECRET_ACCESS_KEY=  # For S3
AWS_REGION=us-east-1
S3_BUcket=chatbot-attachments

# === EMAIL SETTINGS ===
SMTP_HOST=  # For email notifications
SMTP_PORT=587
SMTP_USER=
SMTP_PASSWORD=
SMTP_FROM=noreply@chatbot.example.com

# === WEBHOOKS ===
WEBHOOK_URL=  # For notifications
WEBHOOK_SECRET=  # For signature verification

# === DEVELOPMENT ONLY ===
ENABLE_HOT_RELOAD=true
SHOW_ERROR_DETAILS=true
ENABLE_DEBUG_ENDPOINTS=true
MOCK_LLAMA_RESPONSES=false
```

### Environment-Specific Files
- `.env.development` - Development environment overrides
- `.env.staging` - Staging environment overrides  
- `.env.production` - Production environment overrides
- `.env.test` - Test environment overrides

## Configuration Categories

### Application Settings
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| NODE_ENV | Environment name (development, staging, production) | development | No |
| LOG_LEVEL | Logging level (trace, debug, info, warn, error, fatal) | info | No |
| PORT | Frontend server port | 3000 | No |
| API_PREFIX | API route prefix | /api/v1 | No |

### Database Configuration
#### PostgreSQL
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| POSTGRES_HOST | Database host | localhost | Yes |
| POSTGRES_PORT | Database port | 5432 | No |
| POSTGRES_DB | Database name | chatbot | Yes |
| POSTGRES_USER | Database username | chatbot_user | Yes |
| POSTGRES_PASSWORD | Database password |  | Yes |
| POSTGRES_POOL_MAX | Max connection pool size | 20 | No |
| POSTGRES_IDLE_TIMEOUT_MS | Connection idle timeout | 30000 | No |

#### Redis
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| REDIS_HOST | Redis host | localhost | Yes |
| REDIS_PORT | Redis port | 6379 | No |
| REDIS_PASSWORD | Redis password |  | No |
| REDIS_DB | Redis database number | 0 | No |
| REDIS_TLS | Use TLS for Redis connection | false | No |

#### Neo4j (Knowledge Graph)
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| NEO4J_URI | Neo4j connection URI | bolt://localhost:7687 | Yes |
| NEO4J_USER | Neo4j username | neo4j | Yes |
| NEO4J_PASSWORD | Neo4j password |  | Yes |
| NEO4J_MAX_CONNECTION_POOL_SIZE | Max connection pool | 50 | No |
| NEO4J_CONNECTION_TIMEOUT | Connection timeout (ms) | 30000 | No |
| NEO4J_MAX_TRANSACTION_RETRY_TIME | Max retry time (ms) | 30000 | No |

### Llama.cpp Configuration
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| LLAMA_MODEL_PATH | Path to GGUF model file |  | Yes |
| LLAMA_SERVER_HOST | Llama server host | localhost | No |
| LLAMA_SERVER_PORT | Llama server port | 8080 | No |
| LLAMA_CONTEXT_SIZE | Context window size | 8192 | No |
| LLAMA_GPU_LAYERS | Layers offloaded to GPU | 35 | No |
| LLAMA_BATCH_SIZE | Batch size for prompt processing | 8 | No |
| LLAMA_PARALLEL_SEQUENCES | Number of parallel sequences | 4 | No |
| LLAMA_THREADS | Number of threads for processing | 8 | No |
| LLAMA_SEED | Random seed (-1 for random) | -1 | No |
| LLAMA_F16_KV | Use FP16 for key/value cache | true | No |
| LLAMA_LOG_DISABLE | Disable llama.cpp logging | false | No |

### Authentication
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| JWT_SECRET_KEY | Secret for signing JWT tokens |  | Yes |
| JWT_EXPIRES_IN | Access token expiry (seconds) | 3600 | No |
| JWT_REFRESH_EXPIRES_IN | Refresh token expiry (seconds) | 604800 | No |
| BCRYPT_SALT_ROUNDS | Bcrypt salt rounds for password hashing | 12 | No |

### External Services
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| WIKIPEDIA_API_RATE_LIMIT | Wikipedia API rate limit (req/min) | 50 | No |
| WIKIPEDIA_CACHE_TTL | Wikipedia cache TTL (seconds) | 3600 | No |
| GOOGLE_SEARCH_API_KEY | Google Custom Search API key |  | No* |
| GOOGLE_SEARCH_CX | Google Custom Search ID |  | No* |
| *Required only if FEATURE_INTERNET_ACCESS=enabled |

### Feature Flags
| Variable | Description | Valid Values | Default |
|----------|-------------|--------------|---------|
| FEATURE_KNOWLEDGE_BASE | Enable knowledge base integration | enabled/disabled | enabled |
| FEATURE_PERSONAS | Enable persona system | enabled/disabled | enabled |
| FEATURE_TOOLS | Enable tool execution (mistral-vibe) | enabled/disabled | enabled |
| FEATURE_STREAMING | Enable streaming responses | enabled/disabled | enabled |
| FEATURE_WEBSOCKETS | Enable WebSocket connections | enabled/disabled | enabled |
| FEATURE_FILE_ATTACHMENTS | Enable file uploads/attachments | enabled/disabled | enabled |
| FEATURE_CODE_EXECUTION | Enable code execution tools | enabled/disabled | disabled |
| FEATURE_INTERNET_ACCESS | Allow tools internet access | enabled/disabled | disabled |

### Rate Limiting
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| RATE_LIMIT_AUTH | Auth endpoint requests/min per IP | 10 | No |
| RATE_LIMIT_API | API requests/min per user | 60 | No |
| RATE_LIMIT_TOOLS | Tool execution requests/min per user | 10 | No |
| RATE_LIMIT_WS_CONNECTIONS | Max concurrent WS connections per user | 5 | No |

### Caching
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| CACHE_DEFAULT_TTL | Default cache TTL (seconds) | 300 | No |
| CACHE_QUERY_RESULTS | Query result cache TTL (seconds) | 600 | No |
| CACHE_EMBEDDINGS | Embedding cache TTL (seconds) | 3600 | No |
| CACHE_PERSONAS | Persona cache TTL (seconds) | 1800 | No |

### Monitoring
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| ENABLE_METRICS | Enable Prometheus metrics | true/false | true |
| METRICS_PORT | Port for metrics endpoint | 9090 | No |
| ENABLE_TRACING | Enable distributed tracing | true/false | false |
| TRACING_ENDPOINT | Tracing collector endpoint |  | No* |
| LOG_FORMAT | Log format (json/text) | json | No |
| *Required only if ENABLE_TRACING=true |

### File Storage
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| STORAGE_TYPE | Storage backend (local/s3/gcs/azure) | local | No |
| LOCAL_STORAGE_PATH | Path for local storage | ./uploads | No* |
| AWS_ACCESS_KEY_ID | AWS access key ID |  | ** |
| AWS_SECRET_ACCESS_KEY | AWS secret access key |  | ** |
| AWS_REGION | AWS region | us-east-1 | ** |
| S3_BUCKET | S3 bucket name |  | ** |
| *Required only if STORAGE_TYPE=local |
| **Required only if STORAGE_TYPE=s3 |

### Email Settings
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| SMTP_HOST | SMTP server host |  | No* |
| SMTP_PORT | SMTP server port | 587 | No* |
| SMTP_USER | SMTP username |  | No* |
| SMTP_PASSWORD | SMTP password |  | No* |
| SMTP_FROM | From address for emails | noreply@chatbot.example.com | No* |
| *Required only if email notifications are enabled |

### Webhooks
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| WEBHOOK_URL | URL for outgoing webhooks |  | No* |
| WEBHOOK_SECRET | Secret for webhook signature verification |  | No* |
| *Required only if webhooks are enabled |

### Development Only
| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| ENABLE_HOT_RELOAD | Enable hot reload during development | true/false | true |
| SHOW_ERROR_DETAILS | Show detailed error messages | true/false | true |
| ENABLE_DEBUG_ENDPOINTS | Enable debug/test endpoints | true/false | true |
| MOCK_LLAMA_RESPONSES | Mock llama.cpp responses for testing | true/false | false |

## Configuration Validation

The system validates configuration at startup and reports:
- Missing required variables
- Invalid values (wrong type, out of range)
- Conflicting settings
- Deprecated configuration options

Validation errors prevent the application from starting.

## Configuration Examples

### Development Environment (.env.development)
```env
NODE_ENV=development
LOG_LEVEL=debug
POSTGRES_HOST=localhost
POSTGRES_PASSWORD=dev_password
REDIS_HOST=localhost
REDIS_PASSWORD=
LLAMA_MODEL_PATH=/Users/developer/models/Qwen3.5-9B-ultimate-irrefusable-heretic-Q4_K_M.gguf
LLAMA_GPU_LAYERS=20  # Lower for dev machines with less VRAM
FEATURE_CODE_EXECUTION=disabled
ENABLE_HOT_RELOAD=true
SHOW_ERROR_DETAILS=true
ENABLE_DEBUG_ENDPOINTS=true
```

### Staging Environment (.env.staging)
```env
NODE_ENV=staging
LOG_LEVEL=info
POSTGRES_HOST=staging-postgres.internal
POSTGRES_PASSWORD=staging_secure_password
REDIS_HOST=staging-redis.internal
REDIS_PASSWORD=staging_redis_password
LLAMA_MODEL_PATH=/models/Qwen3.5-9B-ultimate-irrefusable-heretic-Q4_K_M.gguf
LLAMA_GPU_LAYERS=35
FEATURE_CODE_EXECUTION=disabled
ENABLE_HOT_RELOAD=false
SHOW_ERROR_DETAILS=false
ENABLE_DEBUG_ENDPOINTS=false
```

### Production Environment (.env.production)
```env
NODE_ENV=production
LOG_LEVEL=warn
POSTGRES_HOST=prod-postgres-cluster.aws.example.com
POSTGRES_PASSWORD=${PROD_POSTGRES_PASSWORD}  # From secrets manager
REDIS_HOST=prod-redis-cluster.aws.example.com
REDIS_PASSWORD=${PROD_REDIS_PASSWORD}
LLAMA_MODEL_PATH=/mnt/models/Qwen3.5-9B-ultimate-irrefusable-heretic-Q4_K_M.gguf
LLAMA_GPU_LAYERS=35
LLAMA_BATCH_SIZE=16  # Increased for production throughput
LLAMA_PARALLEL_SEQUENCES=8
FEATURE_CODE_EXECUTION=disabled
FEATURE_INTERNET_ACCESS=disabled
ENABLE_HOT_RELOAD=false
SHOW_ERROR_DETAILS=false
ENABLE_DEBUG_ENDPOINTS=false
ENABLE_METRICS=true
ENABLE_TRACING=true
TRACING_ENDPOINT=https://tracing.example.com/api/traces
STORAGE_TYPE=s3
AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
S3_BUCKET=prod-chatbot-attachments
```

## Secrets Management Recommendations

### In Development
- Use `.env` file added to `.gitignore`
- Never commit real secrets to version control
- Use placeholder values in `.env.template`

### In Staging/Production
- Use secrets managers:
  - AWS Secrets Manager or Parameter Store
  - HashiCorp Vault
  - Kubernetes Secrets
  - Google Cloud Secret Manager
  - Azure Key Vault
- Reference secrets in environment variables or configuration service
- Implement secret rotation policies
- Audit secret access and usage

## Configuration Change Management

### Runtime Reloadable Configuration
Some configuration can be changed without restarting:
- Log level (via API or signal)
- Feature flags (via API or configuration service)
- Rate limits (via API or configuration service)
- Cache TTL values (via API or configuration service)

### Restart Required Configuration
These require application restart:
- Database connections
- External service endpoints
- Authentication secrets
- File storage configuration
- Llama.cpp model path and parameters

### Migration Strategy
When configuration schema changes:
1. Add new variables with sensible defaults
2. Deprecate old variables with warnings
3. Remove old variables after grace period
4. Provide migration documentation
5. Validate configuration during startup

## Best Practices

1. **Never hardcode secrets** - Always use environment variables or secrets managers
2. **Use consistent naming** - Prefix with service or feature name (e.g., LLAMA_, POSTGRES_)
3. **Document all variables** - Keep .env.template up to date
4. **Validate in CI** - Add configuration validation to CI pipeline
5. **Use configuration per environment** - Don't try to use one config for all environments
6. **Secure configuration transmission** - Use secure channels for deploying secrets
7. **Monitor configuration changes** - Alert on unexpected configuration modifications
8. **Test configuration changes** - Validate in staging before production deployment
9. **Keep configuration minimal** - Only configure what differs from defaults
10. **Use configuration validation** - Fail fast on invalid configuration

## Troubleshooting Configuration Issues

### Common Problems
1. **Missing required variable** - Check .env file and environment
2. **Invalid value type** - Ensure correct format (number, boolean, etc.)
3. **Connection refused** - Verify host, port, and network connectivity
4. **Authentication failed** - Check credentials and permissions
5. **Permission denied** - Verify file/directory permissions
6. **Address already in use** - Check for conflicting services on same port

### Diagnostic Steps
1. Check application logs for configuration errors
2. Validate environment variables with `env | grep -E "(LLAMA|POSTGRES|REDIS|JWT)"`
3. Test connections manually (e.g., `psql`, `redis-cli`, `curl`)
4. Verify file paths and permissions
5. Check that required services are running
6. Look for typos in variable names
7. Verify values are in expected ranges
8. Check for conflicting configuration sources

This configuration specification provides a comprehensive framework for managing the enhanced chatbot system across all deployment environments, ensuring consistency, security, and operational excellence.