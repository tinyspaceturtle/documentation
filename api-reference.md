# HeroBrain API Reference

The HeroBrain API is a REST API that enables developers to programmatically interact with memory data. It provides endpoints for creating, reading, updating, and searching memories, as well as managing API keys and webhooks.

## Base URL

```
https://api.herobrain.ai
```

## Authentication

All API requests require authentication. HeroBrain supports two authentication methods:

### Bearer Token Authentication

Include your API key in the `Authorization` header:

```
Authorization: Bearer hb_your_api_key_here
```

### Clerk JWT Authentication

For server-side requests from authenticated users:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Rate Limiting

API requests are rate limited based on your plan:

- **Free tier**: 1,000 requests per month
- **Pro tier**: 100,000 requests per month
- **Enterprise**: Custom limits

Rate limit headers are included in all responses:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1638360000
```

## Response Format

All API responses follow a consistent format:

### Success Response

```json
{
  "success": true,
  "data": {
    // Response data varies by endpoint
  },
  "metadata": {
    "request_id": "req_1234567890",
    "timestamp": "2025-11-19T10:30:00Z",
    "version": "v1"
  }
}
```

### Error Response

```json
{
  "success": false,
  "errors": [
    {
      "code": "VALIDATION_ERROR",
      "message": "The 'content' field is required",
      "field": "content"
    }
  ],
  "metadata": {
    "request_id": "req_1234567890",
    "timestamp": "2025-11-19T10:30:00Z",
    "version": "v1"
  }
}
```

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `VALIDATION_ERROR` | 400 | Request validation failed |
| `AUTH_REQUIRED` | 401 | Authentication required |
| `AUTH_INVALID` | 401 | Invalid authentication token |
| `TENANT_MISMATCH` | 403 | Resource belongs to different tenant |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `RESOURCE_NOT_FOUND` | 404 | Requested resource not found |
| `DUPLICATE_RESOURCE` | 409 | Resource already exists |
| `INSUFFICIENT_QUOTA` | 402 | Usage quota exceeded |
| `DATABASE_ERROR` | 500 | Database operation failed |
| `EXTERNAL_SERVICE_ERROR` | 502 | External service error |

## Core Resources

### Memories

The memory resource represents individual pieces of information stored in HeroBrain.

#### Memory Object

```json
{
  "id": "mem_abc123def456",
  "content": "Met with Sarah about Q4 goals",
  "type": "episodic",
  "importance": 0.85,
  "tags": ["meeting", "q4"],
  "metadata": {
    "source": "email",
    "contact_name": "Sarah Chen",
    "sentiment": "positive"
  },
  "createdAt": "2025-11-19T10:00:00Z",
  "updatedAt": "2025-11-19T10:00:00Z"
}
```

#### Memory Types

- `episodic`: Personal experiences and events
- `semantic`: Factual knowledge and concepts
- `procedural`: How-to knowledge and processes
- `working`: Temporary working memory
- `prospective`: Future plans and intentions
- `emotional`: Emotional experiences and states
- `contextual`: Current context and environment
- `identity`: Personal identity and traits
- `relational`: Relationships and social connections
- `event`: Calendar events and occurrences
- `predictive`: Predictions and forecasts
- `task`: Tasks and action items
- `decision`: Decision-making processes

## Endpoints

### Create a Memory

Creates a new memory in your tenant.

```
POST /api/v1/memories
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | Yes | The memory content (1-10000 characters) |
| `type` | string | Yes | Memory type (see valid types above) |
| `importance` | number | No | Importance score (0.0-1.0, default: 0.5) |
| `tags` | string[] | No | Array of tags |
| `metadata` | object | No | Additional metadata |
| `tenantId` | string | Yes | Your tenant ID |

#### Example Request

```bash
curl -X POST "https://api.herobrain.ai/api/v1/memories" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Met with Sarah Chen about Q4 goals. Focus on revenue growth.",
    "type": "episodic",
    "importance": 0.8,
    "tags": ["meeting", "q4"],
    "metadata": {
      "source": "meeting",
      "contact_name": "Sarah Chen",
      "sentiment": "positive"
    },
    "tenantId": "user_12345"
  }'
```

#### Example Response

```json
{
  "success": true,
  "data": {
    "id": "mem_abc123def456",
    "content": "Met with Sarah Chen about Q4 goals. Focus on revenue growth.",
    "type": "episodic",
    "importance": 0.8,
    "tags": ["meeting", "q4"],
    "metadata": {
      "source": "meeting",
      "contact_name": "Sarah Chen",
      "sentiment": "positive"
    },
    "createdAt": "2025-11-19T10:00:00Z",
    "updatedAt": "2025-11-19T10:00:00Z"
  },
  "metadata": {
    "request_id": "req_1234567890",
    "timestamp": "2025-11-19T10:00:00Z",
    "version": "v1"
  }
}
```

### Retrieve a Memory

Retrieves a specific memory by ID.

```
GET /api/v1/memories/{id}
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | The memory ID |
| `tenantId` | string | Yes | Your tenant ID (query parameter) |

#### Example Request

```bash
curl "https://api.herobrain.ai/api/v1/memories/mem_abc123def456?tenantId=user_12345" \
  -H "Authorization: Bearer hb_your_api_key_here"
```

### List Memories

Returns a paginated list of memories.

```
GET /api/v1/memories
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tenantId` | string | Yes | Your tenant ID |
| `page` | number | No | Page number (default: 1) |
| `limit` | number | No | Items per page (default: 50, max: 100) |
| `type` | string | No | Filter by memory type |
| `tags` | string | No | Filter by tags (comma-separated) |
| `sort` | string | No | Sort field: `createdAt`, `importance`, `updatedAt` |
| `order` | string | No | Sort order: `asc`, `desc` (default: `desc`) |

#### Example Request

```bash
curl "https://api.herobrain.ai/api/v1/memories?tenantId=user_12345&page=1&limit=20&type=episodic&sort=importance&order=desc" \
  -H "Authorization: Bearer hb_your_api_key_here"
```

#### Example Response

```json
{
  "success": true,
  "data": {
    "memories": [
      {
        "id": "mem_abc123def456",
        "content": "Met with Sarah Chen about Q4 goals...",
        "type": "episodic",
        "importance": 0.8,
        "createdAt": "2025-11-19T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "pages": 8,
      "hasNext": true,
      "hasPrevious": false
    }
  },
  "metadata": {
    "request_id": "req_1234567890",
    "timestamp": "2025-11-19T10:00:00Z",
    "version": "v1"
  }
}
```

### Update a Memory

Updates an existing memory.

```
PATCH /api/v1/memories/{id}
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | The memory ID |
| `content` | string | No | Updated content |
| `importance` | number | No | Updated importance (0.0-1.0) |
| `tags` | string[] | No | Updated tags |
| `metadata` | object | No | Updated metadata |

#### Example Request

```bash
curl -X PATCH "https://api.herobrain.ai/api/v1/memories/mem_abc123def456" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "importance": 0.9,
    "tags": ["meeting", "q4", "important"]
  }'
```

### Delete a Memory

Deletes a memory permanently.

```
DELETE /api/v1/memories/{id}
```

#### Example Request

```bash
curl -X DELETE "https://api.herobrain.ai/api/v1/memories/mem_abc123def456?tenantId=user_12345" \
  -H "Authorization: Bearer hb_your_api_key_here"
```

### Search Memories

Performs semantic search across memories.

```
POST /api/v1/memories/search
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search query |
| `tenantId` | string | Yes | Your tenant ID |
| `limit` | number | No | Max results (default: 20, max: 100) |
| `type` | string | No | Filter by memory type |
| `minRelevance` | number | No | Minimum relevance score (0.0-1.0) |
| `tags` | string[] | No | Filter by tags |

#### Example Request

```bash
curl -X POST "https://api.herobrain.ai/api/v1/memories/search" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Q4 planning meetings",
    "tenantId": "user_12345",
    "limit": 10,
    "minRelevance": 0.7
  }'
```

#### Example Response

```json
{
  "success": true,
  "data": {
    "results": [
      {
        "id": "mem_abc123def456",
        "content": "Met with Sarah Chen about Q4 goals...",
        "type": "episodic",
        "relevance": 0.89,
        "importance": 0.8,
        "createdAt": "2025-11-19T10:00:00Z"
      }
    ],
    "query": "Q4 planning meetings",
    "latency": 145
  },
  "metadata": {
    "request_id": "req_1234567890",
    "timestamp": "2025-11-19T10:00:00Z",
    "version": "v1"
  }
}
```

## API Key Management

### Create an API Key

Creates a new API key for programmatic access.

```
POST /api/v1/api-keys
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Key name (1-255 characters) |
| `tenantId` | string | Yes | Your tenant ID |
| `environment` | string | No | `development` or `production` (default: `production`) |
| `scopes` | string[] | No | Permissions: `read:memories`, `write:memories` |
| `expiresAt` | number | No | Expiration timestamp |

#### Example Request

```bash
curl -X POST "https://api.herobrain.ai/api/v1/api-keys" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production API Key",
    "tenantId": "user_12345",
    "environment": "production",
    "scopes": ["read:memories", "write:memories"]
  }'
```

#### Example Response

```json
{
  "success": true,
  "data": {
    "id": "key_abc123def456",
    "name": "Production API Key",
    "keyPrefix": "hb_prod_abc123",
    "plainKey": "hb_prod_abc123def456789012345678901234567890",
    "environment": "production",
    "scopes": ["read:memories", "write:memories"],
    "rateLimit": {
      "requestsPerMinute": 60,
      "requestsPerHour": 1000,
      "requestsPerDay": 10000
    },
    "createdAt": "2025-11-19T10:00:00Z"
  },
  "metadata": {
    "request_id": "req_1234567890",
    "timestamp": "2025-11-19T10:00:00Z",
    "version": "v1"
  }
}
```

**⚠️ Important:** The `plainKey` field is only returned once. Store it securely immediately.

### List API Keys

Retrieves all API keys for your tenant.

```
GET /api/v1/api-keys
```

#### Example Request

```bash
curl "https://api.herobrain.ai/api/v1/api-keys?tenantId=user_12345" \
  -H "Authorization: Bearer hb_your_api_key_here"
```

### Revoke an API Key

Revokes an API key, making it unusable.

```
POST /api/v1/api-keys/revoke
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `keyId` | string | Yes | The API key ID to revoke |
| `tenantId` | string | Yes | Your tenant ID |

#### Example Request

```bash
curl -X POST "https://api.herobrain.ai/api/v1/api-keys/revoke" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "keyId": "key_abc123def456",
    "tenantId": "user_12345"
  }'
```

## Webhooks

### Create a Webhook

Registers a webhook endpoint for event notifications.

```
POST /api/v1/webhooks
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Webhook name |
| `url` | string | Yes | HTTPS endpoint URL |
| `events` | string[] | Yes | Events to subscribe to |
| `secret` | string | Yes | Secret for signature verification |
| `status` | string | No | `active` or `inactive` (default: `active`) |

#### Supported Events

- `memory.created`
- `memory.updated`
- `memory.deleted`
- `search.performed`
- `api_key.created`
- `api_key.revoked`

#### Example Request

```bash
curl -X POST "https://api.herobrain.ai/api/v1/webhooks" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Memory Sync",
    "url": "https://your-app.com/webhooks/herobrain",
    "events": ["memory.created", "memory.updated"],
    "secret": "your-webhook-secret"
  }'
```

#### Example Webhook Payload

```json
{
  "event": "memory.created",
  "data": {
    "id": "mem_abc123def456",
    "content": "Met with Sarah about Q4 goals...",
    "type": "episodic",
    "tenantId": "user_12345",
    "createdAt": "2025-11-19T10:00:00Z"
  },
  "metadata": {
    "request_id": "req_1234567890",
    "timestamp": "2025-11-19T10:00:01Z",
    "version": "v1"
  }
}
```

### List Webhooks

Retrieves all webhooks for your tenant.

```
GET /api/v1/webhooks
```

### Test a Webhook

Sends a test payload to your webhook endpoint.

```
POST /api/v1/webhooks/{id}/test
```

## Intelligence Features

### Cross-Memory Reasoning

Analyzes relationships between memories to reconstruct decision-making processes.

```
POST /api/v1/intelligence/decision-chain
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `topic` | string | Yes | Topic to analyze |
| `tenantId` | string | Yes | Your tenant ID |
| `timeRange` | object | No | Date range to analyze |

#### Example Request

```bash
curl -X POST "https://api.herobrain.ai/api/v1/intelligence/decision-chain" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "Q4 strategy",
    "tenantId": "user_12345",
    "timeRange": {
      "from": "2025-10-01T00:00:00Z",
      "to": "2025-11-19T23:59:59Z"
    }
  }'
```

### Pattern Detection

Discovers implicit patterns in your memory data.

```
POST /api/v1/intelligence/implicit-patterns
```

#### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tenantId` | string | Yes | Your tenant ID |
| `minOccurrences` | number | No | Minimum pattern occurrences (default: 3) |

## Usage & Billing

### Get Usage Metrics

Retrieves current usage statistics.

```
GET /api/v1/usage/metrics/{tenantId}
```

#### Example Response

```json
{
  "success": true,
  "data": {
    "apiCalls": {
      "today": 150,
      "thisMonth": 3200,
      "limit": 100000
    },
    "memories": {
      "count": 1250,
      "limit": 1000000
    },
    "storage": {
      "used": 52428800,
      "limit": 1073741824
    }
  },
  "metadata": {
    "request_id": "req_1234567890",
    "timestamp": "2025-11-19T10:00:00Z",
    "version": "v1"
  }
}
```

## SDKs

### JavaScript/TypeScript SDK

```bash
npm install @herobrain/sdk
```

```typescript
import { HeroBrain } from '@herobrain/sdk';

const hb = new HeroBrain({
  apiKey: 'hb_your_api_key_here',
  tenantId: 'user_12345'
});

// Create a memory
const memory = await hb.memories.create({
  content: 'Met with Sarah about Q4 goals',
  type: 'episodic',
  importance: 0.8
});

// Search memories
const results = await hb.memories.search({
  query: 'Q4 planning',
  limit: 10
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
    tenant_id='user_12345'
)

# Create a memory
memory = hb.memories.create(
    content='Met with Sarah about Q4 goals',
    type='episodic',
    importance=0.8
)

# Search memories
results = hb.memories.search(
    query='Q4 planning',
    limit=10
)
```

## Testing

### Test Mode

Use the following test tenant ID for development:

```
tenantId: "00000000-0000-0000-0000-000000000001"
```

### Mock Data

The test tenant contains 975 sample memories across all memory types with realistic content for testing search and intelligence features.

## Support

- **Documentation:** https://docs.herobrain.ai
- **API Status:** https://status.herobrain.ai
- **Support:** support@herobrain.ai
- **Community:** https://community.herobrain.ai

## Changelog

### v1.0.0 (November 19, 2025)
- Initial production release
- Core memory CRUD operations
- Semantic search
- API key management
- Webhook support
- Intelligence features (beta)
- TypeScript and Python SDKs
