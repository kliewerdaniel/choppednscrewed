# Llama.cpp Integration for Local Inference

## Overview
This document details the llama.cpp integration used for local LLM inference in the chatbot application. The system uses a quantized Qwen3.5-9B model served via llama.cpp's OpenAI-compatible API server.

## Technology Stack
- **Inference Engine**: llama.cpp (C/C++)
- **Model Format**: GGUF (Q4_K_M quantization)
- **API Layer**: OpenAI-compatible REST API served by llama-server
- **Client Communication**: HTTP fetch requests from TypeScript frontend
- **Model**: Qwen3.5-9B-ultimate-irrefusable-heretic-Q4_K_M.gguf

## Key Features
1. Local LLM inference without external API dependencies
2. GPU acceleration support (Metal for Apple Silicon)
3. Efficient memory usage through quantization
4. OpenAI API compatibility for easy integration
5. Configurable context windows and caching
6. Speculative decoding support
7. Grammar-constrained generation capabilities

## Server Setup and Configuration
The llama.cpp server is launched with the following typical configuration:
```bash
llama-server -m Qwen3.5-9B-ultimate-irrefusable-heretic-Q4_K_M.gguf \
             -c 8192 \                  # Context window size
             -ngl 35 \                  # Layers offloaded to GPU (Metal)
             -nb 8 \                    # Batch size for prompt processing
             -np 4 \                    # Number of parallel sequences
             --port 8080 \              # API server port
             --host 127.0.0.1 \         # Bind to localhost
             -t 8                       # Number of threads for processing
```

## API Endpoints Used
1. **Chat Completions** (`POST /v1/chat/completions`)
   - Primary endpoint for generating responses
   - Used by `LlamaAPI.queryModel()` and `LlamaAPI.queryWithKnowledgeBase()`

2. **Model Listing** (`GET /v1/models`)
   - Used for health checking via `LlamaAPI.isServerRunning()`

3. **Embeddings** (`POST /v1/embeddings`)
   - Available but not currently used in the implementation

4. **Completion** (`POST /v1/completions`)
   - Available but chat endpoint is preferred for conversational use

## Integration Details
### LlamaAPI Service (`trippy-chat/src/lib/llama-api.ts`)
The service provides two main methods:

1. `queryModel(prompt: string)`: Simple query without conversation history
2. `queryWithKnowledgeBase(prompt: string, messages: Message[])`: Query with conversation history and knowledge base context

Both methods:
- Connect to `http://localhost:8080/v1/chat/completions`
- Use Bearer token authentication (set to "no-key" as llama-server doesn't require auth by default)
- Send requests with system prompt, conversation history, and user query
- Configure temperature (0.7) for balanced creativity/consistency
- Set max_tokens to 4000 for extended responses
- Enable caching with large cache size and 1-hour TTL
- Handle errors with appropriate exception throwing

### Model Configuration
- **Model Name**: Qwen3.5-9B-ultimate-irrefusable-heretic-Q4_K_M.gguf
- **Quantization**: Q4_K_M (4-bit quantization with K-meshing)
- **Size**: Approximately 4.9 GB
- **Context Window**: Configured for 8192 tokens in practice
- **Capabilities**: Strong reasoning, instruction following, multi-turn conversation

## Performance Characteristics
1. **Inference Speed**
   - ~15-30 tokens/second on Apple Silicon M1/M2 (varies with load)
   - First token latency: ~1-2 seconds
   - Subsequent tokens: ~50-100ms each

2. **Memory Usage**
   - Model footprint: ~5-6 GB RAM/VRAM
   - Context memory: ~8KB per token in context window
   - Cache overhead: Configurable, typically 10-20% of model size

3. **Scalability Features**
   - GPU acceleration via Metal framework
   - CPU offloading for layers exceeding GPU memory
   - Batch processing for efficient prompt handling
   - Parallel sequence processing (-np flag)
   - Speculative decoding support for faster generation

## Quality and Capabilities
1. **Language Understanding**
   - Strong performance on multilingual tasks
   - Good instruction following capabilities
   - Effective reasoning and problem-solving

2. **Knowledge Base**
   - Trained on diverse datasets up to 2024
   - Good factual recall for general knowledge
   - Capable of specialized knowledge when prompted

3. **Conversation Abilities**
   - Maintains context over long conversations
   - Handles role-play and persona adoption well
   - Adapts tone based on system prompts

## Integration with Knowledge Base
The llama.cpp inference is enhanced by the knowledge base system:
1. Knowledge base retrieves relevant information for user queries
2. Retrieved information is formatted and prepended to the system prompt
3. Enhanced context improves response accuracy and relevance
4. Token usage is managed to stay within context window limits

## Error Handling and Fallbacks
1. **Server Unavailability**
   - Detected via health check endpoint
   - User notified of service issues
   - Automatic retry mechanisms in UI layer

2. **Generation Errors**
   - Timeout handling for long generations
   - Truncation for responses exceeding max_tokens
   - Fallback to simpler responses on API errors

3. **Resource Constraints**
   - Graceful degradation when GPU memory insufficient
   - Automatic CPU fallback for offloaded layers
   - Context window overflow handling

## Security Considerations
1. **Local Execution**
   - All processing happens locally, no data leaves machine
   - Model weights stored locally in GGUF format
   - No external API calls for inference

2. **API Security**
   - Localhost binding by default (127.0.0.1)
   - Optional authentication can be enabled
   - Rate limiting can be implemented at reverse proxy level

3. **Model Safety**
   - Depends on model's inherent safety training
   - System prompts can add additional constraints
   - Output filtering can be implemented in application layer

## Maintenance and Updates
1. **Model Updates**
   - Drop-in replacement of .gguf file
   - Restart server to load new model
   - Version compatibility maintained through GGUF format

2. **Server Maintenance**
   - Regular updates from llama.cpp repository
   - Configuration optimization based on usage patterns
   - Monitoring of resource utilization

3. **Performance Tuning**
   - Adjust ngl (GPU layers) based on available VRAM
   - Tune context window (-c) for use case requirements
   - Optimize batch size (-nb) and parallel sequences (-np)
   - Thread count (-t) adjustment for CPU-bound scenarios

## Extensibility
1. **Multiple Models**
   - Server can switch models on-the-fly
   - Model selection via API or restart with different model
   - LoRA adapter support for specialized variants

2. **Advanced Features**
   - Speculative drafting with smaller models
   - Grammar-constrained generation (JSON, SQL, etc.)
   - Embedding generation for semantic search
   - Reranking capabilities for improved retrieval

3. **Integration Points**
   - WebSocket support for streaming responses
   - Custom metadata in API requests
   - Callback mechanisms for long generations
   - Function calling capabilities (planned/emerging)

## Deployment Considerations
1. **Hardware Requirements**
   - Minimum: 8GB RAM for Q4 quantized model
   - Recommended: 16GB+ RAM for comfortable operation
   - GPU: Any modern GPU for acceleration (Metal/CUDA/Vulkan)
   - Storage: 5-10GB for model + workspace

2. **Operating System Support**
   - Cross-platform: Linux, macOS, Windows
   - Optimized builds for Apple Silicon (ARM NEON + Metal)
   - x86_64 optimizations (AVX/AVX2/AVX512/AMX)
   - ARM64 support for mobile/embedded devices

3. **Production Readiness**
   - Process management (systemd, Docker, etc.)
   - Logging and monitoring integration
   - Backup and recovery procedures
   - Performance benchmarking and tuning guides