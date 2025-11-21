# SDKs

HeroBrain provides official SDKs for JavaScript/TypeScript and Python to make integration easier and more productive.

## JavaScript/TypeScript SDK

### Installation

```bash
npm install @herobrain/sdk
```

```typescript
import { HeroBrain } from '@herobrain/sdk';

// Initialize the client
const hb = new HeroBrain({
  apiKey: 'hb_your_api_key_here',
  tenantId: 'your_tenant_id_here'
});
```

### Configuration Options

```typescript
const hb = new HeroBrain({
  apiKey: 'hb_your_api_key_here',        // Required: Your API key
  tenantId: 'your_tenant_id_here',        // Required: Your tenant ID
  baseUrl: 'https://api.herobrain.ai',    // Optional: API base URL
  timeout: 30000,                         // Optional: Request timeout in ms
  retryAttempts: 3,                       // Optional: Number of retries
  retryDelay: 1000                        // Optional: Delay between retries
});
```

### Memory Operations

#### Create a Memory

```typescript
const memory = await hb.memories.create({
  content: 'Met with Sarah about Q4 goals',
  type: 'episodic',
  importance: 0.8,
  tags: ['meeting', 'q4'],
  metadata: {
    attendees: ['Sarah Chen'],
    project: 'Q4 Planning'
  }
});

console.log(memory.data);
// {
//   id: 'mem_abc123def456',
//   content: 'Met with Sarah about Q4 goals',
//   type: 'episodic',
//   importance: 0.8,
//   createdAt: '2025-11-19T10:00:00Z'
// }
```

#### Retrieve a Memory

```typescript
const memory = await hb.memories.get('mem_abc123def456');
console.log(memory.data);
```

#### Update a Memory

```typescript
const updated = await hb.memories.update('mem_abc123def456', {
  importance: 0.9,
  tags: ['meeting', 'q4', 'important']
});
console.log(updated.data);
```

#### Delete a Memory

```typescript
await hb.memories.delete('mem_abc123def456');
```

#### List Memories

```typescript
const memories = await hb.memories.list({
  limit: 20,
  page: 1,
  type: 'episodic',
  sort: 'importance',
  order: 'desc'
});

console.log(memories.data.memories);
console.log(memories.data.pagination);
```

#### Search Memories

```typescript
const results = await hb.memories.search({
  query: 'Q4 planning meetings',
  limit: 10,
  minRelevance: 0.7,
  type: 'episodic'
});

console.log(results.data.results);
// [
//   {
//     id: 'mem_abc123',
//     content: 'Met with Sarah about Q4 goals',
//     relevance: 0.89,
//     type: 'episodic'
//   }
// ]
```

#### Bulk Operations

```typescript
// Create multiple memories
const result = await hb.memories.createBulk([
  {
    content: 'Meeting notes from design review',
    type: 'episodic',
    importance: 0.7
  },
  {
    content: 'Updated design system guidelines',
    type: 'semantic',
    importance: 0.8
  }
]);

console.log(`Created ${result.data.created} memories`);

// Delete multiple memories
await hb.memories.deleteBulk(['mem_123', 'mem_456']);
```

### API Key Management

#### Create an API Key

```typescript
const apiKey = await hb.apiKeys.create({
  name: 'Production API Key',
  environment: 'production',
  scopes: ['read:memories', 'write:memories']
});

console.log(apiKey.data.plainKey); // ⚠️ Only shown once!
```

#### List API Keys

```typescript
const keys = await hb.apiKeys.list();
console.log(keys.data.keys);
```

#### Revoke an API Key

```typescript
await hb.apiKeys.revoke('key_abc123def456');
```

### Webhook Management

#### Create a Webhook

```typescript
const webhook = await hb.webhooks.create({
  name: 'Memory Sync',
  url: 'https://your-app.com/webhooks/herobrain',
  events: ['memory.created', 'memory.updated'],
  secret: 'your-webhook-secret'
});

console.log(webhook.data.id);
```

#### List Webhooks

```typescript
const webhooks = await hb.webhooks.list();
console.log(webhooks.data);
```

#### Test a Webhook

```typescript
const testResult = await hb.webhooks.test('webhook_123');
console.log(testResult.data.delivered); // true/false
```

#### Delete a Webhook

```typescript
await hb.webhooks.delete('webhook_123');
```

### Intelligence Features

#### Decision Chain Analysis

```typescript
const analysis = await hb.intelligence.decisionChain({
  topic: 'pricing strategy',
  timeRange: {
    from: '2025-01-01T00:00:00Z',
    to: '2025-11-19T23:59:59Z'
  }
});

console.log(analysis.data.chain);
```

#### Expertise Mapping

```typescript
const expertise = await hb.intelligence.expertiseGraph({
  minConfidence: 0.7
});

console.log(expertise.data.expertiseAreas);
```

#### Pattern Detection

```typescript
const patterns = await hb.intelligence.implicitPatterns({
  minOccurrences: 3
});

console.log(patterns.data.patterns);
```

#### Burnout Analysis

```typescript
const analysis = await hb.intelligence.burnoutAnalysis({
  timeRange: {
    from: '2025-10-01T00:00:00Z',
    to: '2025-11-19T23:59:59Z'
  }
});

console.log(analysis.data.riskLevel); // 'low' | 'medium' | 'high'
```

### Usage & Billing

#### Get Usage Metrics

```typescript
const usage = await hb.usage.metrics();
console.log(usage.data);
// {
//   apiCalls: { today: 150, thisMonth: 3200, limit: 100000 },
//   memories: { count: 1250, limit: 1000000 },
//   storage: { used: 52428800, limit: 1073741824 }
// }
```

#### Get Usage History

```typescript
const history = await hb.usage.history({
  days: 30
});

console.log(history.data.history);
```

### Error Handling

The SDK provides comprehensive error handling:

```typescript
try {
  const memory = await hb.memories.create({
    content: '',
    type: 'invalid'
  });
} catch (error) {
  if (error.response?.data?.success === false) {
    const errors = error.response.data.errors;
    console.log('Validation errors:', errors);
    // Handle specific errors
  } else if (error.code === 'ECONNRESET') {
    // Network error, retry
  } else {
    // Other error
  }
}
```

### Advanced Configuration

#### Custom Request Interceptors

```typescript
const hb = new HeroBrain({
  apiKey: 'hb_your_key',
  tenantId: 'your_tenant',
  interceptors: {
    request: (config) => {
      // Add custom headers
      config.headers['X-Custom-Header'] = 'value';
      return config;
    },
    response: (response) => {
      // Log all responses
      console.log(`${response.config.method} ${response.config.url} - ${response.status}`);
      return response;
    }
  }
});
```

#### Logging

```typescript
const hb = new HeroBrain({
  apiKey: 'hb_your_key',
  tenantId: 'your_tenant',
  logging: {
    level: 'debug', // 'error' | 'warn' | 'info' | 'debug'
    logger: customLogger // Optional custom logger
  }
});
```

### TypeScript Support

The SDK is fully typed with TypeScript:

```typescript
import type {
  Memory,
  MemoryCreateRequest,
  SearchRequest,
  SearchResponse,
  ApiKey,
  Webhook
} from '@herobrain/sdk';

// Full type safety
const memory: Memory = await hb.memories.create({
  content: 'Typed content',
  type: 'episodic' as const,
  importance: 0.8
});
```

## Python SDK

### Installation

```bash
pip install herobrain-sdk
```

### Basic Usage

```python
from herobrain import HeroBrain

# Initialize the client
hb = HeroBrain(
    api_key='hb_your_api_key_here',
    tenant_id='your_tenant_id_here'
)
```

### Memory Operations

```python
# Create a memory
memory = hb.memories.create(
    content='Met with Sarah about Q4 goals',
    type='episodic',
    importance=0.8,
    metadata={
        'attendees': ['Sarah Chen'],
        'project': 'Q4 Planning'
    }
)

print(memory.data.id)

# Search memories
results = hb.memories.search(
    query='Q4 planning meetings',
    limit=10
)

for result in results.data.results:
    print(f"{result.relevance}: {result.content}")
```

### Complete Python Example

```python
import asyncio
from herobrain import HeroBrain

async def main():
    hb = HeroBrain(
        api_key='hb_your_api_key_here',
        tenant_id='your_tenant_id_here'
    )

    # Create memories
    memories_data = [
        {
            'content': 'Completed Q4 planning meeting',
            'type': 'episodic',
            'importance': 0.8
        },
        {
            'content': 'Updated project timeline',
            'type': 'task',
            'importance': 0.7
        }
    ]

    for mem_data in memories_data:
        memory = await hb.memories.create(**mem_data)
        print(f"Created memory: {memory.data.id}")

    # Search
    results = await hb.memories.search(
        query='Q4 planning',
        limit=5
    )

    print(f"Found {len(results.data.results)} memories")

if __name__ == '__main__':
    asyncio.run(main())
```

## SDK Comparison

| Feature | JavaScript/TypeScript | Python |
|---------|----------------------|--------|
| Memory CRUD | ✅ | ✅ |
| Search | ✅ | ✅ |
| Bulk Operations | ✅ | ✅ |
| API Key Management | ✅ | ✅ |
| Webhooks | ✅ | ✅ |
| Intelligence Features | ✅ | ✅ |
| Usage Tracking | ✅ | ✅ |
| TypeScript Types | ✅ | ❌ |
| Async/Await | ✅ | ✅ |
| Error Handling | ✅ | ✅ |
| Rate Limiting | ✅ | ✅ |
| Retry Logic | ✅ | ✅ |
| Streaming Support | ✅ | ✅ |

## MCP (Model Context Protocol)

HeroBrain also provides an MCP server for AI agent integration.

### Installation

```bash
npm install -g @herobrain/mcp
```

### Configuration for Claude Desktop

Add to your Claude Desktop configuration:

```json
{
  "mcpServers": {
    "herobrain": {
      "command": "npx",
      "args": ["-y", "@herobrain/mcp"],
      "env": {
        "TENANT_ID": "your-tenant-id",
        "API_KEY": "your-api-key"
      }
    }
  }
}
```

### Available MCP Tools

1. **search_memories**: Semantic search across memories
2. **create_memory**: Create a new memory
3. **get_memory**: Retrieve a specific memory
4. **list_memories**: List memories with pagination
5. **update_memory**: Update an existing memory

### MCP Usage in Claude

```
Search for memories about Q4 planning meetings
```

Claude will automatically use the HeroBrain MCP tools to search your memories and provide contextually relevant responses.

## Testing with SDKs

### Test Data

Use the test tenant for development:

```typescript
const hb = new HeroBrain({
  apiKey: 'hb_your_key',
  tenantId: '00000000-0000-0000-0000-000000000001' // Test tenant with 975 memories
});
```

### Mock Client

For unit testing, use the mock client:

```typescript
import { MockHeroBrain } from '@herobrain/sdk/test-utils';

const mockHb = new MockHeroBrain();

// Returns mock data without API calls
const memories = await mockHb.memories.list();
```

## Migration Guide

### From REST API to SDK

```typescript
// Before (REST API)
const response = await fetch('https://api.herobrain.ai/api/v1/memories', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    content: 'Test memory',
    type: 'episodic',
    tenantId: tenantId
  })
});
const result = await response.json();

// After (SDK)
const result = await hb.memories.create({
  content: 'Test memory',
  type: 'episodic'
});
```

### From Other SDKs

```python
# From requests to HeroBrain SDK
# Before
import requests

response = requests.post('https://api.herobrain.ai/api/v1/memories',
    headers={'Authorization': f'Bearer {api_key}'},
    json={
        'content': 'Test',
        'type': 'episodic',
        'tenantId': tenant_id
    })
data = response.json()

# After
from herobrain import HeroBrain

hb = HeroBrain(api_key=api_key, tenant_id=tenant_id)
data = hb.memories.create(content='Test', type='episodic')
```

## Best Practices

### Error Handling

```typescript
async function safeMemoryOperation() {
  try {
    const memory = await hb.memories.create(memoryData);
    return memory.data;
  } catch (error) {
    if (error.response?.status === 429) {
      // Rate limited, implement backoff
      await delay(1000);
      return safeMemoryOperation();
    }

    if (error.response?.data?.errors) {
      // Validation errors
      const validationErrors = error.response.data.errors;
      throw new ValidationError(validationErrors);
    }

    // Network or other errors
    throw new NetworkError(error.message);
  }
}
```

### Connection Pooling

```typescript
// For high-traffic applications
const hb = new HeroBrain({
  apiKey: 'hb_key',
  tenantId: 'tenant_id',
  connectionPool: {
    maxConnections: 10,
    keepAlive: true
  }
});
```

### Batch Operations for Performance

```typescript
// Instead of individual creates
const memories = [];
for (const item of items) {
  memories.push({
    content: item.content,
    type: item.type,
    importance: item.importance
  });
}

// Use batch create
const result = await hb.memories.createBulk(memories);
console.log(`Created ${result.data.created} memories`);
```

### Memory Management

```typescript
// For memory-constrained environments
const hb = new HeroBrain({
  apiKey: 'hb_key',
  tenantId: 'tenant_id',
  memoryLimit: '50MB', // Limit response caching
  streaming: true      // Use streaming for large responses
});
```

## Support

- **SDK Issues**: https://github.com/herobrain/sdk/issues
- **Documentation**: https://docs.herobrain.ai/sdks
- **Community**: https://community.herobrain.ai
- **Email**: sdk@herobrain.ai

## Changelog

### v1.0.0 (November 19, 2025)
- Initial release with full API coverage
- TypeScript and Python SDKs
- MCP server integration
- Comprehensive error handling
- Rate limiting and retry logic
- Streaming support for large responses
