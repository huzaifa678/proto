# Proto Schema Registry Setup

## Overview
This repository uses **Buf Schema Registry (BSR)** as the **single source of truth** for protobuf schemas shared between:
- **NestJS (Subscription Service)** - gRPC Server
- **Spring Boot (Billing Service)** - gRPC Client

Currently, both services maintain separate proto files in their own directories. This setup consolidates them into a single registry.

## Current Architecture (Before Migration)

```
subscription-service/         billing-service/
  src/proto/                     src/main/proto/
    subscription.proto             subscription.proto  ← Duplicated!
  package.json                  build.gradle
    proto:gen script              protobuf plugin
```

## New Modern Architecture (With Buf)

```
proto/ (this repo)
  subscription.proto            ← SINGLE SOURCE OF TRUTH
  buf.yaml
  buf.gen.yaml

Buf Schema Registry
  ↓ (auto-generates)
  
npm package                     Maven artifact
@org/proto-contracts      ←→   com.org:proto-contracts
  
Consumed by:
subscription-service/          billing-service/
  npm install                    Maven dependency
  @org/proto-contracts          com.org:proto-contracts
```

---

## Migration Steps

### Phase 1: Setup Buf (20 mins)

#### 1. **Create Buf Account & Token**
```bash
# Visit https://buf.build and create account
# Create a workspace and get your BUF_TOKEN
# Add to GitHub Secrets: Settings > Secrets > BUF_TOKEN
```

#### 2. **Install Buf CLI & Push**
```bash
# Install buf (https://docs.buf.build/installation)
brew install bufbuild/buf/buf

# From /Users/smartboy/proto directory:
cd /Users/smartboy/proto
buf registry login
buf push
```

### Phase 2: Update NestJS Service (Subscription Service)

#### Current Setup
```bash
# subscription-service/subscription-service/package.json
"proto:gen": "protoc --plugin=./node_modules/.bin/protoc-gen-ts_proto --ts_proto_out=src/pb/ --ts_proto_opt=nestJs=true,outputClientImpl=grpc-js,outputServices=grpc-js src/proto/*.proto"
```

#### New Setup with Buf
```bash
cd subscription-service
npm install @org/proto-contracts

rm subscription-service/src/proto/subscription.proto
```

**Update package.json in subscription-service:**
```json
{
  "scripts": {
    "proto:install": "npm install @org/proto-contracts",
    "start:dev": "ts-node -r tsconfig-paths/register --transpile-only src/main.ts"
  },
  "dependencies": {
    "@org/proto-contracts": "^1.0.0",
    "@grpc/grpc-js": "^1.14.3",
    "@grpc/proto-loader": "^0.8.0"
  }
}
```

**Usage in subscription-service:**
```typescript
// src/service/subscription.service.ts
import { SubscriptionService as SubscriptionGrpcService } from '@org/proto-contracts/subscription';
import * as grpc from '@grpc/grpc-js';

// Define gRPC service implementation
const grpcService: SubscriptionGrpcService = {
  getSubscription: async (call) => {
    // Implementation
  },
  getUserActiveSubscriptions: async (call) => {
    // Implementation
  }
};
```

### Phase 3: Update Spring Boot Service (Billing Service)

#### Current Setup
```gradle
// billing-service/build.gradle
plugins {
    id 'com.google.protobuf' version '0.9.5'
}

// Local proto file management
```

#### New Setup with Buf

**Update build.gradle:**
```gradle
plugins {
    id 'com.google.protobuf' version '0.9.5'
}

dependencies {
    // Add Buf-generated artifact
    implementation 'com.org:proto-contracts:1.0.0'
    
    // Existing gRPC deps
    implementation 'io.grpc:grpc-netty-shaded'
    implementation 'io.grpc:grpc-stub'
    implementation 'io.grpc:grpc-protobuf'
}
```

**Remove local proto file:**
```bash
rm src/main/proto/subscription.proto
```

**Usage in billing-service:**
```java
// src/main/java/com/project/billing/client/SubscriptionGrpcClient.java
@Service
public class SubscriptionGrpcClient {
    
    private SubscriptionServiceGrpc.SubscriptionServiceBlockingStub stub;
    
    @Autowired
    public SubscriptionGrpcClient(
        @Value("${subscription.service.host}") String host,
        @Value("${subscription.service.port}") int port) {
        
        ManagedChannel channel = ManagedChannelBuilder
            .forAddress(host, port)
            .usePlaintext()
            .build();
        
        this.stub = SubscriptionServiceGrpc.newBlockingStub(channel);
    }
    
    public SubscriptionResponse getSubscription(String subscriptionId) {
        GetSubscriptionRequest request = GetSubscriptionRequest.newBuilder()
            .setSubscriptionId(subscriptionId)
            .build();
        
        return stub.getSubscription(request);
    }
}
```

## Registry Options (Modern Recommendations)

### Option 1: **Buf Schema Registry (RECOMMENDED)** ⭐
- **Pros**: Versioning, schema linting, breaking change detection, SLA
- **Cost**: Free tier available, paid tiers for enterprise
- **Best for**: Large teams, strict versioning needs

### Option 2: **GitHub Packages** (Current)
- **Pros**: Already integrated with GitHub, free for public repos
- **Cons**: Limited to npm/maven artifacts, not dedicated schema registry
- **Setup**: Already in your `package.json`

### Option 3: **Artifactory/Nexus**
- **Pros**: Self-hosted, universal package management
- **Cons**: Infrastructure overhead, older approach

## Workflow Automation

The included GitHub Actions workflows handle:
1. **buf-publish.yml**: Validates and publishes to Buf on proto changes
2. **generate-artifacts.yml**: Auto-generates and publishes artifacts

## Development Workflow

### Making Changes
```bash
buf lint
buf breaking --against .

git push origin main
```

### Dependencies
```bash
buf lint
buf breaking --against '.git#branch=main'

buf generate

buf format -w
```

## Best Practices

**Use Buf for:**
- Schema versioning and governance
- Breaking change detection
- Multi-language code generation
- API documentation

**Registry Selection:**
- Use **Buf Schema Registry** for schema management
- Use **GitHub Packages** for language-specific artifacts (npm, maven)
- Buf handles the schema; repos handle the compiled code

**Microservice Integration:**
- NestJS: Consume from npm package + use Buf SDK
- Spring Boot: Consume from Maven + use Buf SDK
- Both can reference the same schema module in Buf

## Commands

```bash
# Authenticate
buf registry login

# Push schema
buf push

# Generate code locally
buf generate

# Lint
buf lint

# Check for breaking changes
buf breaking --against '.git#branch=main'

# Format
buf format -w

# Export schema
buf export buf.build/your-org/schemas
```
