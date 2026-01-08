---
name: Azure AI Services
description: This skill should be used when the user asks about "Azure AI Search", "Cognitive Search", "AI Foundry", "Azure OpenAI", "speech to text", "text to speech", "Azure AI", "vector search", "semantic search", or mentions Azure AI and cognitive services. Provides best practices and MCP tool guidance for Azure AI services.
---

# Azure AI Services Best Practices

## Overview

Azure provides a comprehensive suite of AI services for search, language, speech, and machine learning. This skill covers Azure AI Search, Azure AI Services (Speech), and Microsoft Foundry.

**MCP Tools Available:** When the Azure MCP server is enabled, use these tools:
- `azure_search_index_list` - List search indexes
- `azure_search_index_get` - Get index details
- `azure_search_query` - Query search index
- `azure_speech_transcribe` - Transcribe audio
- `azure_speech_synthesize` - Text to speech
- `azure_foundry_model_list` - List AI models
- `azure_foundry_deployment_list` - List deployments
- `azure_foundry_agent_list` - List AI agents

**If Azure MCP is not enabled:** Prompt the user to enable it via `/mcp` or run `/azure:setup`.

## Azure AI Search

### Overview

Azure AI Search (formerly Cognitive Search) provides:
- Full-text search with linguistic analysis
- Vector search for semantic similarity
- Hybrid search combining both
- AI enrichment for content extraction

### Service Tiers

| Tier | Use Case | Replicas | Indexes |
|------|----------|----------|---------|
| Free | Dev/test | 1 | 3 |
| Basic | Small production | 3 | 15 |
| Standard | Production | 12 | 50-200 |
| Storage Optimized | Large data | 12 | 200 |

### Index Design

**Field types:**
- `Edm.String` - Text fields
- `Collection(Edm.Single)` - Vector embeddings
- `Edm.Int32/Int64` - Numeric
- `Edm.DateTimeOffset` - Dates
- `Edm.Boolean` - Boolean
- `Edm.GeographyPoint` - Geospatial

**Field attributes:**
- `searchable` - Full-text search
- `filterable` - Filter expressions
- `sortable` - Sort results
- `facetable` - Faceted navigation
- `retrievable` - Return in results

### Vector Search

**Embedding dimensions:** Match your model output
- OpenAI ada-002: 1536 dimensions
- Azure OpenAI: varies by model
- Custom models: as configured

**Vector search algorithms:**
- HNSW (Hierarchical Navigable Small World) - balanced performance
- Exhaustive KNN - highest accuracy, slower

**Configuration:**
```json
{
  "vectorSearch": {
    "algorithms": [{
      "name": "hnsw-config",
      "kind": "hnsw",
      "hnswParameters": {
        "m": 4,
        "efConstruction": 400,
        "efSearch": 500,
        "metric": "cosine"
      }
    }],
    "profiles": [{
      "name": "vector-profile",
      "algorithm": "hnsw-config"
    }]
  }
}
```

### Hybrid Search

Combine keyword and vector search:
1. Run both queries in parallel
2. Normalize scores (RRF - Reciprocal Rank Fusion)
3. Merge and rank results

Best for:
- Document search with semantic understanding
- Product search with descriptions
- Knowledge bases

### AI Enrichment

**Built-in skills:**
- Entity recognition
- Key phrase extraction
- Language detection
- Sentiment analysis
- OCR (image text extraction)
- Image analysis

**Custom skills:**
- Azure Functions for custom logic
- ML model inference
- External API calls

### Best Practices

1. **Use semantic ranking** for improved relevance
2. **Enable autocomplete** for search-as-you-type
3. **Configure synonyms** for domain vocabulary
4. **Use scoring profiles** to boost important fields
5. **Implement faceted navigation** for filtering
6. **Cache search results** for common queries

## Azure Speech Services

### Speech to Text

**Capabilities:**
- Real-time transcription
- Batch transcription for audio files
- Custom speech models for domain vocabulary
- Speaker diarization (who spoke when)

**Supported formats:**
- WAV, MP3, OGG, FLAC
- Up to 4 hours per file (batch)
- Multiple languages supported

**Best practices:**
1. Use custom speech models for domain-specific terms
2. Implement speaker diarization for multi-party audio
3. Handle silence and noise appropriately
4. Provide phrase hints for better accuracy

### Text to Speech

**Voices:**
- Neural voices (natural sounding)
- Custom neural voices (your own voice)
- Multiple languages and styles

**SSML support:**
```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis">
  <voice name="en-US-JennyNeural">
    <prosody rate="+10%" pitch="+5%">
      Hello, how can I help you today?
    </prosody>
  </voice>
</speak>
```

**Best practices:**
1. Use neural voices for natural output
2. Apply SSML for pronunciation control
3. Cache common phrases
4. Use viseme data for lip-sync

## Microsoft Foundry

### Overview

Microsoft Foundry (AI Foundry) provides:
- Model catalog and deployment
- AI agent creation
- Prompt flow orchestration
- Model fine-tuning

### Model Management

**Available models:**
- Azure OpenAI models (GPT-4, GPT-3.5)
- Open source models (Llama, Mistral)
- Microsoft models
- Custom fine-tuned models

**Deployment options:**
- Managed compute (serverless)
- Dedicated capacity
- Real-time vs batch inference

### AI Agents

**Agent capabilities:**
- Multi-turn conversations
- Tool/function calling
- RAG (Retrieval Augmented Generation)
- Code interpretation

**Best practices:**
1. Define clear agent personas
2. Implement guardrails for safety
3. Use structured outputs for reliability
4. Monitor and log interactions
5. Implement human escalation paths

### Prompt Flow

**Components:**
- Prompt nodes - LLM calls
- Tool nodes - External functions
- Python nodes - Custom logic
- Flow control - Branching, loops

**Best practices:**
1. Version control prompt flows
2. Implement evaluation datasets
3. A/B test prompt variations
4. Monitor latency and costs

## Common Operations with MCP

### Search Operations

```
1. List indexes with azure_search_index_list
2. Get index schema with azure_search_index_get
3. Query index with azure_search_query
4. Use vector and hybrid search modes
```

### Speech Operations

```
1. Transcribe audio with azure_speech_transcribe
2. Convert text to speech with azure_speech_synthesize
3. Specify language and voice options
```

### Foundry Operations

```
1. List models with azure_foundry_model_list
2. List deployments with azure_foundry_deployment_list
3. List agents with azure_foundry_agent_list
4. Manage model configurations
```

## Cost Optimization

### AI Search
1. Right-size service tier
2. Reduce replica count during low traffic
3. Use smaller documents when possible
4. Implement caching for common queries

### Speech Services
1. Use batch transcription for large volumes
2. Cache synthesized audio
3. Choose appropriate voice tier
4. Optimize audio quality vs cost

### Foundry
1. Use appropriate model for task (don't oversize)
2. Implement caching for common prompts
3. Batch requests when possible
4. Monitor token usage

## Security Best Practices

- [ ] Use managed identity for authentication
- [ ] Enable private endpoints
- [ ] Configure CORS appropriately
- [ ] Implement rate limiting
- [ ] Log all API calls
- [ ] Apply content filtering
- [ ] Implement responsible AI practices

## Integration Patterns

### RAG (Retrieval Augmented Generation)

1. Index documents in AI Search
2. Generate embeddings for queries
3. Search for relevant documents
4. Pass context to LLM
5. Generate grounded response

### Voice-Enabled Applications

1. Capture audio input
2. Transcribe with Speech to Text
3. Process with AI/LLM
4. Synthesize response with Text to Speech
5. Play audio output

## MCP Tool Reference

| Operation | MCP Tool | Description |
|-----------|----------|-------------|
| List search indexes | `azure_search_index_list` | Get all indexes |
| Get index | `azure_search_index_get` | Get index details |
| Query search | `azure_search_query` | Search index |
| Transcribe audio | `azure_speech_transcribe` | Speech to text |
| Synthesize speech | `azure_speech_synthesize` | Text to speech |
| List models | `azure_foundry_model_list` | Get AI models |
| List deployments | `azure_foundry_deployment_list` | Get deployments |
| List agents | `azure_foundry_agent_list` | Get AI agents |
