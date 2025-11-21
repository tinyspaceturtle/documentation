# Getting Started with HeroBrain

Welcome to HeroBrain! This guide will help you get up and running with our memory infrastructure platform in 15 minutes or less.

## Quick Start

### 1. Sign Up

Visit [herobrain.ai](https://herobrain.ai) and create your account.

### 2. Get Your API Key

1. Go to the API Platform in your dashboard
2. Click "Create API Key"
3. Give it a name like "Development Key"
4. Copy the API key (you won't see it again!)

### 3. Make Your First API Call

```bash
# Create your first memory
curl -X POST "https://api.herobrain.ai/api/v1/memories" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Just got started with HeroBrain API!",
    "type": "episodic",
    "tenantId": "your_tenant_id_here"
  }'
```

You should receive a response like:

```json
{
  "success": true,
  "data": {
    "id": "mem_abc123def456",
    "content": "Just got started with HeroBrain API!",
    "type": "episodic",
    "createdAt": "2025-11-19T10:00:00Z"
  },
  "metadata": {
    "request_id": "req_1234567890",
    "timestamp": "2025-11-19T10:00:00Z",
    "version": "v1"
  }
}
```

### 4. Search Your Memories

```bash
# Search for memories
curl -X POST "https://api.herobrain.ai/api/v1/memories/search" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "getting started",
    "tenantId": "your_tenant_id_here"
  }'
```

## Installation

### JavaScript/TypeScript SDK

```bash
npm install @herobrain/sdk
```

```typescript
import { HeroBrain } from '@herobrain/sdk';

const hb = new HeroBrain({
  apiKey: 'hb_your_api_key_here',
  tenantId: 'your_tenant_id_here'
});
```

### Python SDK (Coming Soon)

```bash
pip install herobrain-sdk
```

```python
from herobrain import HeroBrain

hb = HeroBrain(
    api_key='hb_your_api_key_here',
    tenant_id='your_tenant_id_here'
)
```

## Core Concepts

### Memories

Memories are the fundamental unit of data in HeroBrain. Each memory represents a piece of information your AI can recall.

**Key Properties:**
- **Content**: The actual text/content of the memory
- **Type**: How the memory should be categorized (episodic, semantic, procedural, etc.)
- **Importance**: How important this memory is (0.0 to 1.0)
- **Metadata**: Additional structured data

### Memory Types

- `episodic`: Personal experiences and events
- `semantic`: Factual knowledge and concepts
- `procedural`: How-to knowledge and processes
- `task`: Tasks and action items

### Tenants

Each user has their own tenant (data silo). The `tenantId` ensures your data stays private and isolated.

## Basic Operations

### Creating Memories

```typescript
// With the SDK
const memory = await hb.memories.create({
  content: "Met with the design team about the new mobile app",
  type: "episodic",
  importance: 0.7,
  metadata: {
    attendees: ["Alice", "Bob", "Charlie"],
    project: "Mobile App Redesign"
  }
});
```

```bash
# With curl
curl -X POST "https://api.herobrain.ai/api/v1/memories" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Met with the design team about the new mobile app",
    "type": "episodic",
    "importance": 0.7,
    "metadata": {
      "attendees": ["Alice", "Bob", "Charlie"],
      "project": "Mobile App Redesign"
    },
    "tenantId": "your_tenant_id_here"
  }'
```

### Searching Memories

```typescript
// Semantic search
const results = await hb.memories.search({
  query: "mobile app design meeting",
  limit: 5
});

console.log(results.data.results);
// [
//   {
//     id: "mem_abc123",
//     content: "Met with the design team about the new mobile app",
//     relevance: 0.89,
//     type: "episodic"
//   }
// ]
```

```bash
# With curl
curl -X POST "https://api.herobrain.ai/api/v1/memories/search" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "mobile app design meeting",
    "limit": 5,
    "tenantId": "your_tenant_id_here"
  }'
```

### Listing Memories

```typescript
// Get recent memories
const memories = await hb.memories.list({
  limit: 10,
  sort: "createdAt",
  order: "desc"
});

console.log(memories.data.memories);
```

```bash
# With curl
curl "https://api.herobrain.ai/api/v1/memories?tenantId=your_tenant_id_here&limit=10&sort=createdAt&order=desc" \
  -H "Authorization: Bearer hb_your_api_key_here"
```

## Common Use Cases

### 1. Meeting Notes & Action Items

```typescript
// After a meeting, capture notes and action items
const meetingMemory = await hb.memories.create({
  content: "Product strategy meeting: Focus on user acquisition for Q4",
  type: "episodic",
  importance: 0.8,
  metadata: {
    meeting_type: "strategy",
    attendees: ["CEO", "CTO", "Product Manager"],
    action_items: [
      "Create user acquisition campaign",
      "Update product roadmap",
      "Schedule follow-up meeting"
    ]
  }
});

// Create individual task memories for action items
for (const action of meetingMemory.metadata.action_items) {
  await hb.memories.create({
    content: action,
    type: "task",
    importance: 0.6,
    metadata: {
      source: "meeting",
      meeting_id: meetingMemory.id,
      status: "pending"
    }
  });
}
```

### 2. Knowledge Base

```typescript
// Store important facts and procedures
await hb.memories.create({
  content: "Our production database is PostgreSQL with connection string: postgresql://user:pass@host:5432/db",
  type: "semantic",
  importance: 0.9,
  metadata: {
    category: "infrastructure",
    access_level: "team"
  }
});

await hb.memories.create({
  content: "To deploy to production: 1) Run tests, 2) Create PR, 3) Get approval, 4) Merge to main",
  type: "procedural",
  importance: 0.8,
  metadata: {
    category: "deployment",
    applies_to: "all-services"
  }
});
```

### 3. Customer Interactions

```typescript
// After a customer call
await hb.memories.create({
  content: "Customer John Doe called about billing issue. Increased plan limit, issue resolved.",
  type: "episodic",
  importance: 0.7,
  metadata: {
    customer_id: "cus_12345",
    customer_name: "John Doe",
    issue_type: "billing",
    resolution: "plan-upgrade",
    sentiment: "satisfied"
  }
});
```

### 4. AI-Powered Search

```typescript
// Find all customer issues
const issues = await hb.memories.search({
  query: "customer billing issues",
  type: "episodic",
  limit: 20
});

// Find procedural knowledge
const procedures = await hb.memories.search({
  query: "how to deploy",
  type: "procedural"
});

// Find high-priority tasks
const tasks = await hb.memories.search({
  query: "urgent tasks",
  type: "task",
  minRelevance: 0.8
});
```

## Advanced Features

### Webhooks

Get notified when memories are created, updated, or searched:

```typescript
// Register a webhook
const webhook = await hb.webhooks.create({
  name: "Memory Sync",
  url: "https://your-app.com/webhooks/herobrain",
  events: ["memory.created", "memory.updated"],
  secret: "your-webhook-secret"
});
```

### Intelligence Features

Use AI to analyze patterns and relationships in your memories:

```typescript
// Find decision-making patterns
const decisions = await hb.intelligence.decisionChain({
  topic: "pricing strategy",
  timeRange: {
    from: "2025-01-01T00:00:00Z",
    to: "2025-11-19T23:59:59Z"
  }
});

// Detect implicit patterns
const patterns = await hb.intelligence.implicitPatterns({
  minOccurrences: 3
});
```

### Bulk Operations

Handle multiple memories at once:

```typescript
// Create multiple memories
const memories = await hb.memories.createBulk([
  {
    content: "Meeting notes from design review",
    type: "episodic"
  },
  {
    content: "Updated design system guidelines",
    type: "semantic"
  }
]);

// Delete multiple memories
await hb.memories.deleteBulk(["mem_123", "mem_456"]);
```

## Error Handling

All HeroBrain APIs return consistent error responses:

```typescript
try {
  const memory = await hb.memories.create({
    content: "", // Invalid: empty content
    type: "episodic"
  });
} catch (error) {
  if (error.response?.data?.success === false) {
    const errors = error.response.data.errors;
    console.log("Validation errors:", errors);
    // Handle validation errors
  }
}
```

Common error codes:
- `VALIDATION_ERROR`: Invalid request data
- `AUTH_REQUIRED`: Missing or invalid API key
- `RATE_LIMIT_EXCEEDED`: Too many requests
- `RESOURCE_NOT_FOUND`: Memory doesn't exist

## Best Practices

### 1. Use Appropriate Memory Types

- `episodic` for events and experiences
- `semantic` for facts and knowledge
- `procedural` for how-to guides
- `task` for action items

### 2. Set Importance Scores

- 0.1-0.3: Minor details
- 0.4-0.6: Regular information
- 0.7-0.8: Important but not critical
- 0.9-1.0: Critical information

### 3. Use Rich Metadata

```typescript
metadata: {
  source: "meeting",           // Where it came from
  contact_name: "John Doe",    // People involved
  project: "Q4 Launch",        // Related projects
  sentiment: "positive",       // Emotional context
  urgency: "high",            // Priority level
  category: "sales"           // Topic category
}
```

### 4. Implement Proper Error Handling

```typescript
async function createMemory(data) {
  try {
    const result = await hb.memories.create(data);
    return result.data;
  } catch (error) {
    if (error.response?.status === 429) {
      // Rate limited, retry later
      await delay(1000);
      return createMemory(data);
    }
    throw error;
  }
}
```

### 5. Use Webhooks for Real-time Updates

```typescript
// In your webhook handler
app.post('/webhooks/herobrain', (req, res) => {
  const signature = req.headers['x-herobrain-signature'];
  const body = req.body;

  // Verify webhook signature
  if (verifySignature(body, signature, WEBHOOK_SECRET)) {
    switch (body.event) {
      case 'memory.created':
        // Index in your search system
        indexMemory(body.data);
        break;
      case 'memory.updated':
        // Update your index
        updateMemory(body.data);
        break;
    }
  }

  res.sendStatus(200);
});
```

## Testing

Use our test tenant for development:

```typescript
const hb = new HeroBrain({
  apiKey: 'hb_your_api_key_here',
  tenantId: '00000000-0000-0000-0000-000000000001' // Test tenant
});

// Contains 975 sample memories for testing
```

## Rate Limits

- **Free tier**: 1,000 requests/month
- **Pro tier**: 100,000 requests/month
- **Enterprise**: Custom limits

Monitor your usage:

```typescript
const usage = await hb.usage.metrics();
console.log(`Used ${usage.data.apiCalls.thisMonth} of ${usage.data.apiCalls.limit} requests`);
```

## Support

- **Documentation**: https://docs.herobrain.ai
- **API Reference**: https://docs.herobrain.ai/api-reference
- **Community**: https://community.herobrain.ai
- **Support**: support@herobrain.ai

## Next Steps

1. **Explore the API**: Check out all available endpoints in our [API Reference](api-reference.md)
2. **Try Intelligence Features**: Use our AI-powered analysis tools
3. **Set Up Webhooks**: Get real-time notifications for memory changes
4. **Build Something**: Create your first AI application with HeroBrain
5. **Join the Community**: Share what you're building and get help

Welcome to the future of AI memory! 🚀
