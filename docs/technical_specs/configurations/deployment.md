# Deployment Configuration

## Overview
This document outlines the deployment configurations for the enhanced chatbot system, covering various deployment strategies from development to production.

## Deployment Strategies

### 1. Local Development
For developers working on the system locally.

#### Requirements
- Node.js 18+ (for frontend)
- Python 3.12+ (for backend services)
- Docker and Docker Compose (for llama.cpp and dependencies)
- Git

#### Setup Steps
1. Clone the repository
2. Install frontend dependencies: `cd trippy-chat && npm install`
3. Install backend dependencies: `cd backend && pip install -r requirements.txt`
4. Start llama.cpp server using Docker Compose: `docker-compose up -d llama-server`
5. Configure environment variables (see environment.md)
6. Start backend services: `uvicorn main:app --reload`
7. Start frontend: `npm run dev`

#### Docker Compose for Development
```yaml
version: '3.8'
services:
  llama-server:
    image: ghcr.io/ggerganov/llama.cpp:server
    volumes:
      - ./models:/models
    ports:
      - "8080:8080"
    command: >
      llama-server
      -m /models/Qwen3.5-9B-ultimate-irrefusable-heretic-Q4_K_M.gguf
      -c 8192
      -ngl 35
      -nb 8
      -np 4
      --port 8080
      --host 0.0.0.0
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: chatbot
      POSTGRES_USER: chatbot_user
      POSTGRES_PASSWORD: secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
volumes:
  postgres_data:
```

### 2. Staging Environment
For testing pre-release versions with production-like settings.

#### Infrastructure
- Kubernetes cluster (or Docker Swarm)
- Separate namespace for staging
- Resource limits and requests configured
- Monitoring and logging enabled

#### Key Differences from Production
- Smaller resource allocations
- Debug logging enabled
- Feature flags for experimental features
- Separate database instances

### 3. Production Environment
For serving end-users with high availability and performance requirements.

#### Architecture
![Production Architecture](architecture.png) <!-- This would be a diagram in a real doc -->

##### Components
1. **Load Balancer**: Distributes traffic across frontend instances
2. **Frontend**: Trippy-chat Next.js application (scaled horizontally)
3. **API Gateway**: Routes requests to appropriate backend services
4. **Backend Services**:
   - Chat Service: Handles message processing and orchestration
   - Knowledge Base Service: Manages KG operations
   - Persona Service: Manages persona lifecycle and activation
   - Tool Service: Executes mistral-vibe and other tools
5. **Inference Cluster**: Llama.cpp servers for LLM inference
6. **Data Stores**:
   - PostgreSQL: Primary relational data (users, conversations, etc.)
   - Redis: Caching, session storage, pub/sub
   - Neo4j or JanusGraph: Knowledge graph storage (for complex traversals)
   - MongoDB: Document storage for flexible metadata
   - MinIO/S3: Object storage for attachments and model files
7. **Monitoring**: Prometheus, Grafana, ELK stack
8. **CI/CD**: GitHub Actions or GitLab CI for automated deployments

#### Kubernetes Manifests Overview
```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: chatbot-production

# ConfigMaps and Secrets
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: chatbot-config
  namespace: chatbot-production
data:
  MODEL_PATH: "/models/Qwen3.5-9B-ultimate-irrefusable-heretic-Q4_K_M.gguf"
  CONTEXT_SIZE: "8192"
  GPU_LAYERS: "35"
  BATCH_SIZE: "8"
  PARALLEL_SEQUENCES: "4"

---
apiVersion: v1
kind: Secret
metadata:
  name: chatbot-secrets
  namespace: chatbot-production
type: Opaque
data:
  POSTGRES_PASSWORD: <base64-encoded>
  REDIS_PASSWORD: <base64-encoded>
  JWT_SECRET: <base64-encoded>

# Deployments
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: chatbot-production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: trippy-chat
        image: ghcr.io/yourorg/trippy-chat:latest
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: chatbot-config
        - secretRef:
            name: chatbot-secrets
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-chat
  namespace: chatbot-production
spec:
  replicas: 4
  selector:
    matchLabels:
      app: backend-chat
  template:
    metadata:
      labels:
        app: backend-chat
    spec:
      containers:
      - name: chat-service
        image: ghcr.io/yourorg/chatbot-backend:latest
        ports:
        - containerPort: 8000
        envFrom:
        - configMapRef:
            name: chatbot-config
        - secretRef:
            name: chatbot-secrets
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"

# Services
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: chatbot-production
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer

---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: chatbot-production
spec:
  selector:
    app: backend-chat
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: ClusterIP

# Ingress
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: chatbot-ingress
  namespace: chatbot-production
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: chatbot.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api/
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
```

### 4. Hybrid Cloud Deployment
For organizations with specific data residency or latency requirements.

#### Pattern
- Keep sensitive knowledge graph data on-premise or in private cloud
- Use public cloud for frontend and stateless backend services
- Llama.cpp inference can be split: sensitive queries go to private cloud, general queries to public cloud
- Use VPN or dedicated interconnect for secure communication between environments

## Configuration Management

### Environment-Specific Configurations
Configurations are managed through:
1. Environment variables (for secrets and environment-specific values)
2. ConfigMap/Kubernetes objects (for non-secret configuration)
3. Feature flags (for toggling capabilities)
4. External configuration service (like Consul or etcd for dynamic updates)

### Secrets Management
- Use Kubernetes Secrets, HashiCorp Vault, or cloud provider secret managers
- Rotate secrets regularly
- Audit access to secrets
- Never commit secrets to version control

## Scaling Strategies

### Horizontal Pod Autoscaler (HPA)
Configure based on CPU utilization, memory consumption, or custom metrics (request latency, queue depth).

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-chat-hpa
  namespace: chatbot-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-chat
  minReplicas: 4
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods
    pods:
      metric:
        name: request_latency_seconds
        target:
          type: AverageValue
          averageValue: "2"
```

### Vertical Pod Autoscaler (VPA)
For optimizing resource requests and limits based on actual usage.

### Cluster Autoscaler
Automatically adjust the number of nodes in the cluster based on pod scheduling requirements.

## Monitoring and Logging

### Metrics to Collect
- **Latency**: Request latency, LLM inference time, knowledge graph query time
- **Throughput**: Requests per second, concurrent users, token generation rate
- **Errors**: Error rates by type and component
- **Resources**: CPU, memory, disk, network utilization per service
- **Business Metrics**: Conversation length, user satisfaction, feature adoption

### Logging Structure
Structured JSON logs with standard fields:
```json
{
  "timestamp": "ISO 8601",
  "level": "info|warn|error",
  "service": "service-name",
  "trace_id": "UUID",
  "span_id": "UUID",
  "message": "string",
  "fields": {
    // Service-specific fields
  }
}
```

### Health Checks
- Liveness probe: Checks if the application is running
- Readiness probe: Checks if the application is ready to serve traffic
- Startup probe: Checks if the application has finished initialization

## Backup and Disaster Recovery

### Data Backup Strategy
- **PostgreSQL**: Daily full backups, hourly WAL archiving
- **Redis**: RDB snapshots every 6 hours, AOF persistence
- **Knowledge Graph**: Daily exports to cold storage, transaction log replication
- **Object Storage**: Versioned buckets with lifecycle policies
- **Configuration**: Git-backed with encryption for secrets

### Disaster Recovery Plan
1. **RTO (Recovery Time Objective)**: <4 hours for critical services
2. **RPO (Recovery Point Objective)**: <1 hour for user data
3. **Failover Process**: Automated failover to secondary region
4. **Testing**: Quarterly DR drills
5. **Documentation**: Runbooks stored in accessible location

## Security Considerations in Deployment

### Network Security
- Private networks for inter-service communication
- Network policies to restrict traffic between namespaces
- TLS encryption for all service-to-service communication
- Web Application Firewall (WAF) for ingress traffic

### Image Security
- Base images scanned for vulnerabilities
- Images signed and verified
- Regular updates to base images
- Minimal base images (distroless or alpine)

### Runtime Security
- Pod security policies (or OPA/Gatekeeper)
- Read-only root containers where possible
- Drop unnecessary Linux capabilities
- Run as non-root user
- Seccomp and AppArmor profiles

### Compliance
- Audit logging for all data access
- Data encryption at rest and in transit
- Regular penetration testing
- Compliance with GDPR, HIPAA, or other relevant regulations

## Performance Tuning

### Database
- Connection pooling
- Read replicas for read-heavy workloads
- Proper indexing strategy
- Query optimization and monitoring

### Caching
- Multi-level caching (local, Redis, CDN)
- Cache warming strategies
- Cache invalidation policies
- Monitoring cache hit rates

### Llama.cpp Inference
- GPU utilization monitoring
- Batch size optimization
- Context window management
- Quantization selection based on quality/speed tradeoff
- Model sharding for very large models

### Network
- Keep-alive connections
- HTTP/2 where possible
- CDN for static assets
- Geographic load balancing

## Update and Rollback Procedures

### Deployment Strategy
- Blue-green deployments for zero-downtime updates
- Canary releases for risk mitigation
- Rolling updates for stateless services
- Database migration strategies (backward compatible, then cleanup)

### Rollback Triggers
- Error rate exceeding threshold
- Latency degradation beyond SLA
- Failed health checks
- Manual trigger by operations team

### Rollback Process
1. Halt new deployments
2. Route traffic to previous stable version
3. Investigate cause of failure
4. Fix and redeploy
5. Return to normal operations

## Cost Optimization

### Resource Rightsizing
- Regular review of resource requests and limits
- Use spot instances for fault-tolerant workloads
- Schedule scaling for predictable workloads
- Delete unused resources

### Storage Optimization
- Lifecycle policies for object storage
- Compression and deduplication
- Archive old data to cheaper storage tiers
- Regular cleanup of temporary files

### Licensing
- Use open-source alternatives where possible
- Track and optimize commercial license usage
- Negotiate enterprise agreements based on actual usage

## Troubleshooting Common Issues

### Deployment Failures
1. Image pull failures: Check registry credentials and network
2. Configuration errors: Validate ConfigMaps and Secrets
3. Resource constraints: Increase limits or adjust node sizes
4. CrashLoopBackOff: Check application logs for startup errors

### Performance Issues
1. High latency: Check database queries, external service calls, resource saturation
2. Low throughput: Check thread pools, connection pools, async processing
3. Memory leaks: Monitor memory growth over time, use profiling tools
4. CPU spikes: Profile CPU usage, look for inefficient algorithms

### Data Issues
1. Inconsistent data: Check transaction boundaries, replication lag
2. Missing data: Verify backup and restore processes
3. Corrupted data: Run data validation checks, consider point-in-time recovery

## Future Enhancements

### Deployment Improvements
- Service mesh (Istio/Linkerd) for advanced traffic management
- GitOps workflow with ArgoCD or Flux
- AI-driven autoscaling based on predictive analytics
- Multi-cluster deployments for global low-latency access
- Serverless options for sporadic workloads

### Monitoring Advancements
- Distributed tracing with OpenTelemetry
- Anomaly detection using machine learning
- Business intelligence dashboards
- User experience monitoring (Real User Monitoring)

### Security Advancements
- Zero-trust network architecture
- Confidential computing for sensitive workloads
- Automated compliance checking
- Advanced threat detection and response