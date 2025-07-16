# AWS Amplify Gen 2 Core Concepts

## Overview

AWS Amplify Gen 2 represents a fundamental shift in how we build fullstack applications. Unlike Gen 1's CLI-driven approach, Gen 2 embraces a **code-first** methodology where your backend infrastructure is defined, versioned, and deployed using TypeScript code.

## Core Principles

### 1. Code-First Infrastructure
Define your entire backend using TypeScript, not configuration files:

```typescript
// amplify/backend.ts
import { defineBackend } from '@aws-amplify/backend';

const backend = defineBackend({
  auth,    // Authentication & Authorization
  data,    // GraphQL API & Database
  storage, // File Storage & CDN
});
```

### 2. Per-Developer Cloud Sandboxes
Each developer gets isolated AWS environments:
- **Real AWS resources** for local development
- **Cost-effective** - pay only for what you use
- **Conflict-free** - no shared development resources
- **Production parity** - same services in dev and prod

### 3. Fullstack Type Safety
Automatic type generation across your entire stack:
- Backend schema → Frontend types
- Real-time synchronization
- IntelliSense support everywhere
- Catch errors at compile time

## Key Concepts

### Resource Definition Pattern

Each backend feature is defined in its own resource file:

```typescript
// amplify/data/resource.ts - Data layer
export const data = defineData({
  schema: a.schema({
    Product: a.model({
      name: a.string().required(),
      price: a.integer().required(),
    }),
  }),
});

// amplify/auth/resource.ts - Authentication
export const auth = defineAuth({
  loginWith: { email: true },
  groups: ["Admins", "Users"],
});

// amplify/storage/resource.ts - File storage
export const storage = defineStorage({
  name: "myFiles",
  access: (allow) => ({
    "public/*": [allow.guest.to(["read"])],
  }),
});
```

### Automatic Configuration Generation

Gen 2 automatically generates `amplify_outputs.json` containing all necessary configuration:

```json
{
  "auth": {
    "aws_region": "us-east-1",
    "user_pool_id": "us-east-1_abc123",
    "user_pool_client_id": "xyz789"
  },
  "data": {
    "aws_region": "us-east-1",
    "url": "https://xyz.appsync-api.us-east-1.amazonaws.com/graphql"
  },
  "storage": {
    "aws_region": "us-east-1",
    "bucket_name": "amplify-myapp-dev-bucket"
  }
}
```

### Type-Safe Client Generation

Frontend clients are automatically typed based on your backend schema:

```typescript
import { generateClient } from 'aws-amplify/data';
import type { Schema } from '../amplify/data/resource';

const client = generateClient<Schema>();

// Fully typed operations
const { data: products } = await client.models.Product.list();
//     ^? Product[] - IntelliSense knows the exact type
```

## Modern Development Workflow

### 1. Start Development
```bash
# Start your personal cloud sandbox
npx ampx sandbox

# In another terminal
npm run dev
```

### 2. Make Changes
```bash
# Enable auto-deployment on code changes
npx ampx sandbox --watch
```

### 3. Real-time Collaboration
- Each team member has isolated environments
- Schema changes don't affect others
- Easy to test different features in parallel

### 4. Deploy to Production
```bash
# Connect to Amplify Console for branch-based deployments
# Automatic staging and production environments
```

## Data Layer Deep Dive

### Declarative Schema Definition
```typescript
const schema = a.schema({
  // Models (DynamoDB tables)
  Product: a.model({
    name: a.string().required(),
    description: a.string(),
    price: a.integer().required(),
    
    // Relationships
    images: a.hasMany("ProductImage", "productId"),
    reviews: a.hasMany("Review", "productId"),
    
    // Enums for type safety
    status: a.ref("ProductStatus"),
  }),
  
  // Enums
  ProductStatus: a.enum(["DRAFT", "ACTIVE", "ARCHIVED"]),
  
  // Custom queries
  listActiveProducts: a
    .query()
    .returns(a.ref("Product").array())
    .handler(a.handler.custom({ entry: "./listActive.js" })),
});
```

### Real-time Subscriptions
```typescript
// Automatic real-time updates
const subscription = client.models.Product.observeQuery().subscribe({
  next: ({ items, isSynced }) => {
    setProducts(items);
  }
});
```

### Authorization Patterns
```typescript
Product: a.model({
  name: a.string().required(),
})
.authorization((allow) => [
  allow.guest().to(["read"]),           // Public read
  allow.authenticated().to(["read"]),   // Auth read
  allow.group("Admins"),               // Admin full access
  allow.owner(),                       // Owner-based access
])
```

## Authentication & Authorization

### Simple Setup
```typescript
export const auth = defineAuth({
  loginWith: {
    email: true,
    // Optional: social providers
    externalProviders: {
      google: { /* config */ },
      facebook: { /* config */ },
    },
  },
  groups: ["Admins", "Users"],
  multifactor: {
    mode: "OPTIONAL",
    sms: { enable: true },
    totp: { enable: true },
  },
});
```

### Frontend Integration
```typescript
import { Authenticator } from '@aws-amplify/ui-react';

function App() {
  return (
    <Authenticator>
      {({ signOut, user }) => (
        <main>
          <h1>Hello {user?.username}</h1>
          <button onClick={signOut}>Sign out</button>
        </main>
      )}
    </Authenticator>
  );
}
```

## File Storage

### Path-Based Access Control
```typescript
export const storage = defineStorage({
  name: "myAppStorage",
  access: (allow) => ({
    // Public files
    "public/*": [
      allow.guest.to(["read"]),
      allow.authenticated.to(["read", "write"]),
    ],
    
    // User-specific files
    "private/{entity_id}/*": [
      allow.entity("identity").to(["read", "write", "delete"]),
    ],
    
    // Admin-only files
    "admin/*": [
      allow.groups(["Admins"]).to(["read", "write", "delete"]),
    ],
  }),
});
```

### Frontend Usage
```typescript
import { uploadData, getUrl } from 'aws-amplify/storage';

// Upload with progress
const upload = await uploadData({
  path: `public/images/${file.name}`,
  data: file,
  options: {
    onProgress: ({ transferredBytes, totalBytes }) => {
      const progress = (transferredBytes / totalBytes) * 100;
      setProgress(progress);
    },
  },
});

// Get CDN URL
const { url } = await getUrl({ path: upload.path });
```

## Advanced Customization

### CDK Integration
Access underlying AWS resources for advanced customization:

```typescript
// amplify/backend.ts
const backend = defineBackend({ auth, data, storage });

// Access CDK constructs
const dataStack = Stack.of(backend.data);
const userPool = backend.auth.resources.userPool;

// Add custom resources
new aws_lambda.Function(dataStack, "CustomFunction", {
  runtime: aws_lambda.Runtime.NODEJS_20_X,
  handler: "index.handler",
  code: aws_lambda.Code.fromAsset("./custom-function"),
});
```

### Custom Business Logic
```typescript
// Custom resolvers for complex operations
const schema = a.schema({
  calculateShipping: a
    .query()
    .arguments({ 
      items: a.ref("CartItem").array(),
      destination: a.string()
    })
    .returns(a.ref("ShippingQuote"))
    .handler(a.handler.custom({
      entry: "./resolvers/calculateShipping.js"
    })),
});
```

## Production Deployment

### Branch-Based Environments
- **Automatic staging** environments for feature branches
- **Production deployment** from main branch
- **Environment-specific** configuration
- **Zero-downtime** deployments

### Build Configuration
```yaml
# amplify.yml (auto-generated)
version: 1
backend:
  phases:
    build:
      commands:
        - npm ci
        - npx ampx generate outputs --app-id $AWS_APP_ID --branch $AWS_BRANCH
frontend:
  phases:
    preBuild:
      commands:
        - npm ci
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: .next
    files:
      - '**/*'
```

## Best Practices

### 1. Resource Organization
- One feature per resource file
- Clear naming conventions
- Logical grouping of related models

### 2. Type Safety
- Always use generated types
- Avoid `any` types
- Validate data at boundaries

### 3. Authorization
- Start with least privilege
- Use field-level auth for sensitive data
- Test auth rules thoroughly

### 4. Performance
- Use appropriate indexes
- Implement pagination
- Optimize query patterns

### 5. Development Workflow
- Use sandbox for development
- Enable watch mode for faster iteration
- Clean up sandboxes regularly

## Monitoring & Debugging

### Built-in Observability
```bash
# View real-time logs
npx ampx sandbox logs --follow

# Function-specific logs
npx ampx sandbox logs --function myFunction

# Check resource status
npx ampx sandbox status
```

### CloudWatch Integration
- Automatic metrics for all resources
- Error tracking and alerting
- Performance monitoring
- Cost optimization insights

This comprehensive overview covers the core concepts and patterns that make Amplify Gen 2 a powerful platform for modern fullstack development.