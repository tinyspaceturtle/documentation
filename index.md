# HeroBrain Documentation

Welcome to HeroBrain, the memory infrastructure platform for AI applications. Build AI that remembers everything.

## 🚀 Quick Start

Get started in 15 minutes:

1. **[Sign up](https://herobrain.ai)** for a free account
2. **[Create an API key](authentication.md)** in your dashboard
3. **[Make your first API call](getting-started.md)**

```bash
curl -X POST "https://api.herobrain.ai/api/v1/memories" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Just getting started with HeroBrain!",
    "type": "episodic",
    "tenantId": "your_tenant_id"
  }'
```

## 📚 Documentation

### For Developers

| Guide | Description |
|-------|-------------|
| **[Getting Started](getting-started.md)** | Quick start guide (15 minutes) |
| **[API Reference](api-reference.md)** | Complete API documentation |
| **[Authentication](authentication.md)** | API keys, JWTs, security |
| **[SDKs](sdks.md)** | JavaScript, Python, MCP libraries |

### Core Concepts

- **Memories**: Individual pieces of information
- **Tenants**: Isolated data containers per user
- **Memory Types**: Episodic, semantic, procedural, task
- **Intelligence**: AI-powered pattern recognition

### Memory Types

| Type | Purpose | Example |
|------|---------|---------|
| `episodic` | Personal experiences | "Met with Sarah about Q4 goals" |
| `semantic` | Factual knowledge | "PostgreSQL is our database" |
| `procedural` | How-to knowledge | "Deploy: npm run build && npm run deploy" |
| `task` | Action items | "Send proposal by Friday" |

## 🛠️ SDKs & Tools

### Official SDKs

```bash
# JavaScript/TypeScript
npm install @herobrain/sdk

# Python
pip install herobrain-sdk
```

### MCP (Model Context Protocol)

Integrate HeroBrain with AI agents:

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

Compatible with Claude Desktop, Cursor, VS Code, and other MCP clients.

## 📊 Key Features

### Memory Management
- **CRUD Operations**: Create, read, update, delete memories
- **Bulk Operations**: Handle multiple memories efficiently
- **Metadata**: Rich structured data with each memory
- **Tagging**: Organize memories with custom tags

### Intelligent Search
- **Semantic Search**: AI-powered understanding
- **Hybrid Search**: Combine semantic + keyword matching
- **Relevance Scoring**: Results ranked by importance
- **Filtering**: By type, tags, date ranges, metadata

### Intelligence Features
- **Pattern Recognition**: Discover implicit patterns
- **Decision Archaeology**: Reconstruct decision processes
- **Expertise Mapping**: Identify knowledge areas
- **Cross-Memory Reasoning**: Connect related memories

### Developer Experience
- **REST API**: Clean, consistent HTTP API
- **Rate Limiting**: Fair usage with clear limits
- **Webhooks**: Real-time event notifications
- **Usage Analytics**: Monitor API usage and costs

## 💰 Pricing

| Plan | API Calls/Month | Memories | Intelligence | Price |
|------|----------------|----------|--------------|-------|
| **Free** | 1,000 | 1,000 | Basic | $0 |
| **Pro** | 100,000 | 1,000,000 | Advanced | $15/month |
| **Enterprise** | Custom | Unlimited | Full | Contact sales |

## 🌟 Use Cases

### Customer Support AI
```typescript
// Store customer interactions
await hb.memories.create({
  content: "Customer reported billing issue - resolved by updating payment method",
  type: "episodic",
  metadata: {
    customer_id: "cus_123",
    issue_type: "billing",
    resolution: "payment_update",
    sentiment: "satisfied"
  }
});

// Later, AI can recall similar issues
const similar = await hb.memories.search({
  query: "billing issues resolved",
  limit: 5
});
```

### Meeting Intelligence
```typescript
// Capture meeting notes and action items
const meeting = await hb.memories.create({
  content: "Q4 planning: Focus on user acquisition, launch mobile app",
  type: "episodic",
  importance: 0.9,
  metadata: {
    attendees: ["Alice", "Bob", "Charlie"],
    action_items: ["Update roadmap", "Design mobile mockups", "Plan user testing"]
  }
});

// AI finds related decisions and context
const context = await hb.intelligence.decisionChain({
  topic: "mobile app development",
  timeRange: { from: "2025-01-01T00:00:00Z" }
});
```

### Knowledge Management
```typescript
// Store important procedures and facts
await hb.memories.create({
  content: "To deploy to production: 1) Run tests, 2) Create PR, 3) Get approval, 4) Merge",
  type: "procedural",
  importance: 0.8,
  metadata: {
    category: "deployment",
    applies_to: "web-applications"
  }
});

// Search for relevant knowledge
const procedures = await hb.memories.search({
  query: "how to deploy web application",
  type: "procedural"
});
```

## 🔧 API Examples

### Create Memory
```bash
curl -X POST "https://api.herobrain.ai/api/v1/memories" \
  -H "Authorization: Bearer hb_your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Met with design team about new branding",
    "type": "episodic",
    "importance": 0.7,
    "metadata": {"project": "rebrand"}
  }'
```

### Search Memories
```bash
curl -X POST "https://api.herobrain.ai/api/v1/memories/search" \
  -H "Authorization: Bearer hb_your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "design team meetings",
    "limit": 10
  }'
```

### Intelligence Analysis
```typescript
const patterns = await hb.intelligence.implicitPatterns({
  minOccurrences: 3
});

console.log("Discovered patterns:", patterns.data.patterns);
```

## 🔗 Integrations

### Popular Frameworks

**Next.js/React**
```typescript
'use client';
import { HeroBrain } from '@herobrain/sdk';

export default function MemoryComponent() {
  const [memories, setMemories] = useState([]);

  useEffect(() => {
    const hb = new HeroBrain({
      apiKey: process.env.NEXT_PUBLIC_HEROBRAIN_KEY,
      tenantId: userId
    });

    hb.memories.list().then(result => {
      setMemories(result.data.memories);
    });
  }, []);

  return (
    <div>
      {memories.map(memory => (
        <div key={memory.id}>{memory.content}</div>
      ))}
    </div>
  );
}
```

**Express.js**
```typescript
const express = require('express');
const { HeroBrain } = require('@herobrain/sdk');

const app = express();
const hb = new HeroBrain({
  apiKey: process.env.HEROBRAIN_KEY,
  tenantId: process.env.HEROBRAIN_TENANT
});

app.post('/api/memories', async (req, res) => {
  try {
    const memory = await hb.memories.create(req.body);
    res.json(memory.data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

**Python FastAPI**
```python
from fastapi import FastAPI
from herobrain import HeroBrain

app = FastAPI()
hb = HeroBrain(
    api_key=os.getenv("HEROBRAIN_KEY"),
    tenant_id=os.getenv("HEROBRAIN_TENANT")
)

@app.post("/memories")
async def create_memory(content: str, type: str):
    memory = await hb.memories.create(
        content=content,
        type=type
    )
    return memory.data
```

## 📈 Performance

### API Performance
- **P95 Latency**: <200ms for memory operations
- **Search Latency**: <500ms for semantic search
- **Throughput**: 10,000+ requests/minute
- **Uptime**: 99.9% SLA

### Search Quality
- **Relevance Scores**: 75-95% accuracy
- **Context Preservation**: Full conversation history
- **Multi-Modal**: Text, images, audio support (coming soon)

## 🔒 Security & Compliance

- **End-to-End Encryption**: Data encrypted at rest and in transit
- **Tenant Isolation**: Complete data separation between users
- **SOC 2 Compliant**: Enterprise-grade security practices
- **GDPR Ready**: Data portability and deletion capabilities

## 🎯 Success Stories

### Startup: AI Customer Support
*"HeroBrain reduced our support ticket resolution time by 60% by giving our AI access to complete customer interaction history."*

### Enterprise: Meeting Intelligence
*"Our executive team now gets AI-powered summaries of all relevant discussions and decisions, even from meetings they couldn't attend."*

### Developer: Knowledge Base
*"Built an AI assistant that knows our entire codebase, architecture decisions, and development processes."*

## 📞 Support

- **Documentation**: https://docs.herobrain.ai
- **API Status**: https://status.herobrain.ai
- **Community**: https://community.herobrain.ai
- **Email**: support@herobrain.ai
- **Chat**: Available in dashboard

## 🚀 What's Next

- **Multi-Modal Memories**: Images, audio, video
- **Advanced Intelligence**: Predictive analytics, trend detection
- **Team Collaboration**: Shared memory spaces
- **Mobile SDKs**: iOS and Android libraries
- **Enterprise Features**: SSO, audit logs, advanced analytics

---

**Ready to build?** [Get started now](getting-started.md) or [explore the API](api-reference.md).
