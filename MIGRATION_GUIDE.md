# Migration Guide: From Dual Protos to Centralized Buf Registry

## Current State (Today)

```
Project Structure:
├── subscription-service/              (NestJS - gRPC Server)
│   ├── subscription-service/
│   │   ├── src/proto/
│   │   │   └── subscription.proto     ← Local copy
│   │   ├── src/pb/                    ← Generated code
│   │   └── package.json
│   │       └── "proto:gen" script
│   └── ...
│
├── billing-service/                   (Spring Boot - gRPC Client)
│   ├── billing-service/
│   │   ├── src/main/proto/
│   │   │   └── subscription.proto     ← Duplicate copy
│   │   ├── src/main/generated/        ← Generated code
│   │   ├── build.gradle
│   │   └── protobuf { } config
│   └── ...
│
└── proto/                             (NEW - Central Repo)
    ├── proto/subscription.proto
    ├── buf.yaml
    ├── buf.gen.yaml
    └── buf.lock
```

## Target State (After Migration)

```
Project Structure:
├── subscription-service/              (NestJS - gRPC Server)
│   ├── subscription-service/
│   │   ├── package.json
│   │   │   └── npm install @org/proto-contracts
│   │   └── src/
│   │       └── (no more proto/ folder)
│   └── ...
│
├── billing-service/                   (Spring Boot - gRPC Client)
│   ├── billing-service/
│   │   ├── build.gradle
│   │   │   └── implementation 'com.org:proto-contracts:1.0.0'
│   │   └── (no more proto/ folder)
│   └── ...
│
└── proto/                             (Central Source of Truth)
    ├── proto/subscription.proto       ← SINGLE COPY
    ├── buf.yaml
    ├── buf.gen.yaml
    ├── buf.lock
    └── .github/workflows/
        ├── buf-publish.yml            ← Validates & publishes to Buf
        └── generate-artifacts.yml     ← Auto-generates & publishes packages
```

---

## Step-by-Step Migration


### Step 1: Initialize Buf Registry (10 mins)

```bash
# From /Users/smartboy/proto directory

# 1. Install Buf CLI (if not already installed)
brew install bufbuild/buf/buf

# 2. Create Buf account at https://buf.build
# 3. Create a workspace
# 4. Get your auth token

# 5. Authenticate locally
buf registry login

# 6. Push initial version
buf push
# Output: Pushed buf.build/YOUR_ORG/schemas@v1
```

### Step 2: Update GitHub Secrets (5 mins)

```
GitHub > Settings > Secrets and Variables > Actions

Add:
- BUF_TOKEN: <your-buf-token>
- GITHUB_TOKEN: (already exists)
```

### Step 3: Migration - Subscription Service (NestJS)

#### 3a. Update package.json

**Before:**
```json
{
  "name": "subscription-service",
  "scripts": {
    "proto:gen": "protoc --plugin=./node_modules/.bin/protoc-gen-ts_proto --ts_proto_out=src/pb/ --ts_proto_opt=nestJs=true,outputClientImpl=grpc-js,outputServices=grpc-js src/proto/*.proto"
  },
  "dependencies": {
    "@grpc/grpc-js": "^1.14.3",
    "@grpc/proto-loader": "^0.8.0"
  }
}
```

**After:**
```json
{
  "name": "subscription-service",
  "scripts": {
    "proto:install": "npm install @org/proto-contracts"
  },
  "dependencies": {
    "@org/proto-contracts": "^1.0.0",
    "@grpc/grpc-js": "^1.14.3",
    "@grpc/proto-loader": "^0.8.0"
  }
}
```

#### 3b. Install proto package

```bash
cd subscription-service/subscription-service
npm install @org/proto-contracts
```

#### 3c. Update imports in code

**Before:**
```typescript
import { GetSubscriptionRequest } from '../pb/subscription';
```

**After:**
```typescript
import { GetSubscriptionRequest } from '@org/proto-contracts/subscription';
```

#### 3d. Remove local proto directory

```bash
rm -rf subscription-service/subscription-service/src/proto
rm -rf subscription-service/subscription-service/src/pb
```

#### 3e. Verify

```bash
# Should work without issues
npm run start:dev
```

### Step 4: Migration - Billing Service (Spring Boot)

#### 4a. Update build.gradle

**Before:**
```gradle
plugins {
    id 'com.google.protobuf' version '0.9.5'
}

protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:3.21.0"
    }
    generateProtoTasks {
        all().each { task ->
            task.builtins {
                java { }
                grpc { }
            }
        }
    }
}
```

**After:**
```gradle
plugins {
    id 'com.google.protobuf' version '0.9.5'
}

dependencies {
    implementation 'com.org:proto-contracts:1.0.0'
    // ... rest of dependencies
}
```

#### 4b. Update imports in code

**Before:**
```java
import subscription.GetSubscriptionRequest;
import subscription.SubscriptionServiceGrpc;
```

**After:**
```java
// Same imports - they come from the Maven artifact now
import subscription.GetSubscriptionRequest;
import subscription.SubscriptionServiceGrpc;
```

#### 4c. Remove local proto directory

```bash
rm -rf billing-service/billing-service/src/main/proto
rm -rf billing-service/billing-service/src/main/generated
```

#### 4d. Verify

```bash
cd billing-service/billing-service
./gradlew build
```

### Step 5: Verify Integration

#### From Subscription Service
```bash
# Subscription service should start gRPC server
cd subscription-service/subscription-service
npm install
npm run start:dev

# Should output: Subscription gRPC server started on port 50051
```

#### From Billing Service
```bash
# Billing service should connect and work
cd billing-service/billing-service
./gradlew bootRun

# Test endpoint
curl -X POST http://localhost:8080/api/v1/billing/process/sub_123
```

---

## Handling Proto Changes Going Forward

### Developer Updates Proto
```bash
echo "Adding new RPC method..."

buf lint
buf breaking --against '.git#branch=main'

git push origin main
```

### CI/CD Automatically
1. **buf-publish.yml workflow:**
   - Validates schema (buf lint)
   - Checks for breaking changes (buf breaking)
   - Publishes to Buf Schema Registry

2. **generate-artifacts.yml workflow:**
   - Generates TypeScript for NestJS
   - Publishes to npm
   - Generates Java for Spring Boot
   - Publishes to Maven

### Services Consume Updates
```bash
pnpm update @org/proto-contracts
pnpm run start:dev

./gradlew clean build  
./gradlew bootRun
```

---