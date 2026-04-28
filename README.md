# Shared Proto Schema Repository

Central repository for protobuf schema definitions shared between NestJS and Spring Boot microservices.

**Acts as the single point of truth for inter-service gRPC communication**

---

## 🚀 Quick Start

### For First-Time Setup
1. Read [README_SETUP.md](README_SETUP.md) - Understand the architecture
2. Initialize Buf registry (instructions in setup docs)
3. Follow [QUICK_CHECKLIST.md](QUICK_CHECKLIST.md) to migrate services

### For Developers
- **Modifying schemas?** → See [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md) workflow section
- **Need code examples?** → See [EXAMPLES_NESTJS.md](EXAMPLES_NESTJS.md) or [EXAMPLES_SPRINGBOOT.md](EXAMPLES_SPRINGBOOT.md)
- **Choosing a registry?** → See [REGISTRY_COMPARISON.md](REGISTRY_COMPARISON.md)

---

## 📚 Complete Documentation

| Document | Purpose |
|----------|---------|
| **[INDEX.md](INDEX.md)** | Start here - documentation index & overview |
| **[MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)** | Detailed step-by-step migration instructions |
| **[REGISTRY_COMPARISON.md](REGISTRY_COMPARISON.md)** | Why Buf Schema Registry is recommended |

---

## 🏗️ Project Structure

```
proto/                                    # Central schema repository
subscription/v1/                 
|            ├──subscription.proto
|            └── buf.lock                 # Dependency lock file
├── buf.yaml                              # Linting & validation rules
├── .github/
│   └── workflows/
│       ├── buf-publish.yml            # Validates & publishes to Buf registry
│       └── generate-artifacts.yml     # Auto-generates & publishes packages
├── INDEX.md                       # Documentation index
├── MIGRATION_GUIDE.md             # Migration walkthrough
└── REGISTRY_COMPARISON.md         # Registry comparison
```

---

## 🎯 Services Using This Repository

### Subscription Service (NestJS)
- **Role:** gRPC Server
- **Location:** `/Users/smartboy/subscription-service`
- **Consumes:** `@org/proto-contracts` (npm package)
- **See:** [EXAMPLES_NESTJS.md](EXAMPLES_NESTJS.md)

### Billing Service (Spring Boot)
- **Role:** gRPC Client
- **Location:** `/Users/smartboy/billing-service`
- **Consumes:** `com.org:proto-contracts` (Maven artifact)
- **See:** [EXAMPLES_SPRINGBOOT.md](EXAMPLES_SPRINGBOOT.md)

---

## 🔄 Proto Development Workflow

### Making Changes
```bash
# 1. Edit schema
nano proto/subscription.proto

# 2. Validate locally
buf lint
buf format -w
buf breaking --against '.git#branch=main'

# 3. Commit & push
git add proto/subscription.proto
git commit -m "feat: add field to subscription"
git push origin main

# 4. CI/CD automatically:
#    - Publishes to Buf Schema Registry
#    - Generates TypeScript & Java code
#    - Publishes npm package
#    - Publishes Maven artifact
```

### Consuming Updates
**NestJS:**
```bash
npm update @org/proto-contracts
```

**Spring Boot:**
```bash
./gradlew build  # Gradle auto-detects new version
```

---

## 📋 Key Features

✅ **Single Source of Truth** - One schema definition for all services
✅ **Multi-Language Support** - Auto-generates TypeScript & Java
✅ **Breaking Change Detection** - Buf catches incompatible changes
✅ **API Governance** - Track schema versions and evolution
✅ **CI/CD Integration** - Automatic validation and publishing
✅ **Team Collaboration** - Code review and ownership tracking

---

## 🛠️ Modern Tech Stack

| Tool | Purpose | Status |
|------|---------|--------|
| **Buf** | Schema registry & governance | ✅ Configured |
| **GitHub Actions** | CI/CD automation | ✅ Configured |
| **GitHub Packages** | npm & Maven artifact hosting | ✅ Ready |
| **gRPC** | Inter-service communication | ✅ Implemented |

---

## 📖 Understanding the Setup

### Before This Approach
```
subscription-service/        billing-service/
  src/proto/                    src/main/proto/
    subscription.proto ←→ subscription.proto  (duplicated!)
```

**Problems:**
- ❌ Duplicate schema definitions
- ❌ Risk of desynchronization
- ❌ No central governance
- ❌ Manual code generation
- ❌ No breaking change detection

### After This Approach
```
proto/ (Central Repository)
  subscription.proto ← SINGLE SOURCE OF TRUTH
        ↓
    Buf Registry
        ↓
   ┌─────┴─────┐
   ↓           ↓
 npm         Maven
   ↓           ↓
NestJS    Spring Boot
```

**Benefits:**
- ✅ One definition, everywhere
- ✅ Automatic synchronization
- ✅ Central governance
- ✅ Automatic code generation
- ✅ Breaking change detection

---

## 🚀 Getting Started

**Step 1: Understand the Setup**
```bash
cd /Users/smartboy/proto
cat README_SETUP.md
```

**Step 2: Follow Migration Checklist**
```bash
cat QUICK_CHECKLIST.md
```

**Step 3: Implement Changes**
- [EXAMPLES_NESTJS.md](EXAMPLES_NESTJS.md) for server
- [EXAMPLES_SPRINGBOOT.md](EXAMPLES_SPRINGBOOT.md) for client

**Step 4: Test Integration**
```bash
# Terminal 1
cd /Users/smartboy/subscription-service/subscription-service
npm install && npm run start:dev

# Terminal 2
cd /Users/smartboy/billing-service/billing-service
./gradlew bootRun

# Terminal 3 - Test
curl -X POST http://localhost:8080/api/v1/billing/process/sub_123
```

---

## 📚 Documentation Map

```
START HERE
    ↓
[INDEX.md] - Overview & Quick Links
    ↓
┌───────────────────────────────────────┐
│                                       │
↓                                       ↓
[README_SETUP.md]              [REGISTRY_COMPARISON.md]
Understand architecture        Why this approach?
    ↓                                   
[MIGRATION_GUIDE.md]          
Detailed walkthrough
    ↓
[QUICK_CHECKLIST.md]
Track progress
    ↓
┌───────────────────────────────────────┐
│                                       │
↓                                       ↓
[EXAMPLES_NESTJS.md]          [EXAMPLES_SPRINGBOOT.md]
Code for server                Code for client
```

---

## ❓ FAQ

**Q: Why Buf instead of just GitHub Packages?**
A: Buf adds schema governance, breaking change detection, and multi-language support. GitHub Packages only distributes artifacts. See [REGISTRY_COMPARISON.md](REGISTRY_COMPARISON.md).

**Q: What if a service needs a different proto version?**
A: Git tags in Buf registry allow pinning. See [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md).

**Q: How do I revert a breaking change?**
A: Buf prevents breaking changes by default. See rollback section in [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md).

**Q: Can I test proto changes locally?**
A: Yes! Use `buf lint`, `buf breaking`, and `buf generate` locally before pushing. See [README_SETUP.md](README_SETUP.md).

---

## 🔗 Links

- **Buf Documentation:** https://docs.buf.build
- **gRPC:** https://grpc.io
- **Protocol Buffers:** https://developers.google.com/protocol-buffers

---

## 📧 Support

For questions about:
- **Setup:** See [README_SETUP.md](README_SETUP.md)
- **Migration:** See [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)
- **Registries:** See [REGISTRY_COMPARISON.md](REGISTRY_COMPARISON.md)

---

**Version:** 1.0.0 | **Status:** Ready for Migration | **Last Updated:** April 28, 2026