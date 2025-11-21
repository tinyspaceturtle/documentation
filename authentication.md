# Authentication

HeroBrain uses a secure, token-based authentication system. All API requests must include valid authentication credentials.

## Overview

HeroBrain supports two authentication methods:

1. **API Keys** - For programmatic access and server-side applications
2. **Clerk JWTs** - For authenticated user requests from client applications

## API Key Authentication

### Creating API Keys

API keys are created through the HeroBrain dashboard or API.

#### Via Dashboard

1. Sign in to [herobrain.ai](https://herobrain.ai)
2. Navigate to **API Platform** → **API Keys**
3. Click **"Create API Key"**
4. Configure:
   - **Name**: Descriptive name (e.g., "Production API Key")
   - **Environment**: `development` or `production`
   - **Scopes**: Permissions for the key
5. Click **Create**
6. **⚠️ Copy the key immediately** - it won't be shown again

#### Via API

```bash
curl -X POST "https://api.herobrain.ai/api/v1/api-keys" \
  -H "Authorization: Bearer hb_existing_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production API Key",
    "environment": "production",
    "scopes": ["read:memories", "write:memories"]
  }'
```

### Using API Keys

Include your API key in the `Authorization` header:

```
Authorization: Bearer hb_prod_abc123def456789012345678901234567890
```

#### Example Request

```bash
curl -X POST "https://api.herobrain.ai/api/v1/memories" \
  -H "Authorization: Bearer hb_prod_abc123def456789012345678901234567890" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Test memory",
    "type": "episodic",
    "tenantId": "your_tenant_id"
  }'
```

### API Key Scopes

| Scope | Description | Example Endpoints |
|-------|-------------|-------------------|
| `read:memories` | Read memories and search | `GET /memories`, `POST /memories/search` |
| `write:memories` | Create, update, delete memories | `POST /memories`, `PATCH /memories/:id`, `DELETE /memories/:id` |
| `read:intelligence` | Access intelligence features | `POST /intelligence/*` |
| `write:webhooks` | Manage webhooks | `POST /webhooks`, `DELETE /webhooks/:id` |

### API Key Security

#### Best Practices

1. **Use environment-specific keys**: Create separate keys for development and production
2. **Limit scopes**: Only grant necessary permissions
3. **Rotate regularly**: Create new keys and revoke old ones
4. **Monitor usage**: Check API key usage in the dashboard
5. **Store securely**: Use environment variables or secure key management systems

#### Environment Variables

```bash
# .env
HEROBRAIN_API_KEY=hb_prod_abc123def456789012345678901234567890
HEROBRAIN_TENANT_ID=your_tenant_id_here
```

```typescript
// config.ts
export const config = {
  herobrain: {
    apiKey: process.env.HEROBRAIN_API_KEY,
    tenantId: process.env.HEROBRAIN_TENANT_ID,
    baseUrl: 'https://api.herobrain.ai'
  }
};
```

### Managing API Keys

#### List API Keys

```bash
curl "https://api.herobrain.ai/api/v1/api-keys?tenantId=your_tenant_id" \
  -H "Authorization: Bearer hb_your_api_key_here"
```

#### Revoke an API Key

```bash
curl -X POST "https://api.herobrain.ai/api/v1/api-keys/revoke" \
  -H "Authorization: Bearer hb_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "keyId": "key_abc123def456",
    "tenantId": "your_tenant_id"
  }'
```

#### Monitor Usage

```bash
curl "https://api.herobrain.ai/api/v1/api-keys/usage?keyId=key_abc123def456&days=30" \
  -H "Authorization: Bearer hb_your_api_key_here"
```

## Clerk JWT Authentication

### Overview

For client-side applications where users are authenticated via Clerk, you can use Clerk JWT tokens directly with HeroBrain APIs.

### Client-Side Authentication Flow

```typescript
// 1. User signs in with Clerk
import { useAuth } from '@clerk/nextjs';

function MyComponent() {
  const { getToken, userId } = useAuth();

  const createMemory = async () => {
    // 2. Get Clerk JWT token
    const token = await getToken();

    // 3. Make API request with Clerk token
    const response = await fetch('https://api.herobrain.ai/api/v1/memories', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        content: 'User created memory',
        type: 'episodic',
        tenantId: userId // Clerk user ID becomes tenant ID
      })
    });

    const result = await response.json();
    console.log(result);
  };

  return (
    <button onClick={createMemory}>
      Create Memory
    </button>
  );
}
```

### Server-Side Authentication (Next.js API Routes)

```typescript
// pages/api/memories.js
import { withAuth } from '@clerk/nextjs/api';

export default withAuth(async (req, res) => {
  const { userId } = req.auth;

  // Get Clerk token for HeroBrain API
  const token = await fetch('https://api.herobrain.ai/auth/clerk-token', {
    headers: {
      'Authorization': `Bearer ${req.headers.authorization}`
    }
  });

  // Use token with HeroBrain
  const response = await fetch('https://api.herobrain.ai/api/v1/memories', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      content: req.body.content,
      type: 'episodic',
      tenantId: userId
    })
  });

  const result = await response.json();
  res.status(200).json(result);
});
```

## Rate Limiting

### Limits by Plan

| Plan | Requests/Month | Requests/Minute | Burst Limit |
|------|----------------|-----------------|-------------|
| Free | 1,000 | 10 | 20 |
| Pro | 100,000 | 100 | 200 |
| Enterprise | Custom | Custom | Custom |

### Rate Limit Headers

All API responses include rate limit information:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1638360000
X-RateLimit-Retry-After: 60  // Only present when rate limited
```

### Handling Rate Limits

```typescript
async function makeApiCall() {
  const response = await fetch('/api/memories', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(memoryData)
  });

  if (response.status === 429) {
    // Rate limited
    const retryAfter = response.headers.get('X-RateLimit-Retry-After');
    const resetTime = response.headers.get('X-RateLimit-Reset');

    console.log(`Rate limited. Retry after ${retryAfter} seconds`);
    console.log(`Limit resets at ${new Date(parseInt(resetTime) * 1000)}`);

    // Wait and retry
    await new Promise(resolve => setTimeout(resolve, parseInt(retryAfter) * 1000));
    return makeApiCall();
  }

  return response.json();
}
```

## Security Best Practices

### API Key Security

1. **Never commit API keys to version control**
2. **Use environment variables in production**
3. **Rotate keys regularly** (recommended: monthly)
4. **Monitor key usage** for suspicious activity
5. **Use different keys for different environments**

### Environment-Specific Keys

```typescript
// config.ts
const config = {
  development: {
    apiKey: process.env.HEROBRAIN_DEV_API_KEY,
    baseUrl: 'https://api.herobrain.ai'
  },
  production: {
    apiKey: process.env.HEROBRAIN_PROD_API_KEY,
    baseUrl: 'https://api.herobrain.ai'
  }
};

export const currentConfig = config[process.env.NODE_ENV || 'development'];
```

### Key Rotation Strategy

```typescript
// Rotate API keys monthly
async function rotateApiKey(oldKeyId: string) {
  // 1. Create new key
  const newKey = await hb.apiKeys.create({
    name: `Production ${new Date().toISOString().split('T')[0]}`,
    environment: 'production',
    scopes: ['read:memories', 'write:memories']
  });

  // 2. Update your application configuration
  updateEnvironmentVariable('HEROBRAIN_API_KEY', newKey.data.plainKey);

  // 3. Revoke old key
  await hb.apiKeys.revoke({
    keyId: oldKeyId
  });

  console.log('API key rotated successfully');
}
```

## Testing Authentication

### Test API Key

```bash
# Test with a simple request
curl "https://api.herobrain.ai/api/v1/memories?tenantId=00000000-0000-0000-0000-000000000001&limit=1" \
  -H "Authorization: Bearer hb_your_api_key_here"
```

### Test Clerk JWT

```typescript
// In browser console on your app
const token = await window.Clerk.session.getToken();
console.log('Clerk token:', token);

// Test with API
fetch('https://api.herobrain.ai/api/v1/memories', {
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    content: 'Test from Clerk JWT',
    type: 'episodic',
    tenantId: window.Clerk.user.id
  })
})
.then(res => res.json())
.then(console.log);
```

## Troubleshooting

### Common Authentication Errors

#### 401 Unauthorized

```json
{
  "success": false,
  "errors": [
    {
      "code": "AUTH_REQUIRED",
      "message": "Authentication required"
    }
  ]
}
```

**Solutions:**
- Check that the `Authorization` header is present
- Verify the API key format: `Bearer hb_...`
- Ensure the key hasn't expired

#### 403 Forbidden

```json
{
  "success": false,
  "errors": [
    {
      "code": "TENANT_MISMATCH",
      "message": "Resource belongs to different tenant"
    }
  ]
}
```

**Solutions:**
- Verify the `tenantId` matches the API key's tenant
- Check that you're using the correct tenant ID for your account

#### 429 Too Many Requests

```json
{
  "success": false,
  "errors": [
    {
      "code": "RATE_LIMIT_EXCEEDED",
      "message": "Too many requests"
    }
  ]
}
```

**Solutions:**
- Check the `X-RateLimit-Retry-After` header
- Implement exponential backoff
- Consider upgrading your plan

### Debugging Authentication

#### Check API Key Validity

```bash
# Get API key details
curl "https://api.herobrain.ai/api/v1/api-keys?tenantId=your_tenant_id" \
  -H "Authorization: Bearer hb_your_api_key_here"
```

#### Validate JWT Token

```typescript
// Decode JWT to check contents
function decodeJWT(token: string) {
  const payload = JSON.parse(atob(token.split('.')[1]));
  console.log('JWT payload:', payload);
  console.log('Expires:', new Date(payload.exp * 1000));
  console.log('Tenant ID:', payload.sub);
}
```

#### Test Token with API

```bash
# Test authentication
curl -I "https://api.herobrain.ai/api/v1/memories?tenantId=your_tenant_id" \
  -H "Authorization: Bearer your_token_here"
```

## Support

If you're having authentication issues:

1. **Check the API status**: https://status.herobrain.ai
2. **Review your code**: Ensure correct header format
3. **Verify your keys**: Check expiration and scopes
4. **Contact support**: support@herobrain.ai with your request ID

## Migration Guide

### From API Keys to Clerk JWTs

If you're migrating from API key authentication to Clerk JWTs:

```typescript
// Before (API Key)
const hb = new HeroBrain({
  apiKey: 'hb_prod_abc123...',
  tenantId: 'user_123'
});

// After (Clerk JWT)
const { getToken, userId } = useAuth();

// Get fresh token for each request
const token = await getToken();
const response = await fetch('/api/memories', {
  headers: { 'Authorization': `Bearer ${token}` },
  body: JSON.stringify({ ...data, tenantId: userId })
});
```

### From Development to Production

```typescript
// config.ts
const configs = {
  development: {
    apiKey: process.env.HEROBRAIN_DEV_KEY,
    baseUrl: 'https://api.herobrain.ai'
  },
  production: {
    apiKey: process.env.HEROBRAIN_PROD_KEY,
    baseUrl: 'https://api.herobrain.ai'
  }
};

export const config = configs[process.env.NODE_ENV || 'development'];
```
