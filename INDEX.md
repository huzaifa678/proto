# Proto Registry Documentation Index

Complete guide for managing shared protobuf schemas across NestJS and Spring Boot microservices using Buf Schema Registry.

---

## 📚 Documentation Files

### 1. **[README_SETUP.md](README_SETUP.md)** - START HERE ⭐
**Overview of the complete setup**
- Current architecture (dual protos problem)
- Target architecture (centralized Buf)
- Step-by-step setup instructions
- Migration path for each service
- Key differences from old approach

**Best for:** Understanding the big picture and getting oriented

---

### 2. **[QUICK_CHECKLIST.md](QUICK_CHECKLIST.md)** - USE WHILE MIGRATING ✓
**Actionable checklist for completing migration**
- Setup verification steps
- Per-service migration checklist
- Integration testing scenarios
- Troubleshooting guide
- Progress tracking

**Best for:** Hands-on migration work, keeping track of what's done

---

### 3. **[MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)** - DETAILED WALKTHROUGH
**Complete step-by-step migration instructions**
- Current vs target state visualization
- Detailed steps for each service
- Code changes with before/after
- Handling proto changes going forward
- Troubleshooting with solutions
- Rollback plan

**Best for:** Deep understanding of each migration step

---

### 4. **[EXAMPLES_NESTJS.md](EXAMPLES_NESTJS.md)** - NESTJS IMPLEMENTATION
**Subscription Service (gRPC Server) code examples**
- Installation & package setup
- Service implementation
- gRPC controller setup
- Module configuration
- Main application setup
- Testing approaches

**Best for:** Writing NestJS gRPC server code

---

### 5. **[EXAMPLES_SPRINGBOOT.md](EXAMPLES_SPRINGBOOT.md)** - SPRING BOOT IMPLEMENTATION
**Billing Service (gRPC Client) code examples**
- Gradle configuration
- gRPC client service
- Spring service using client
- REST controller
- Application configuration
- Testing & deployment

**Best for:** Writing Spring Boot gRPC client code

---

### 6. **[REGISTRY_COMPARISON.md](REGISTRY_COMPARISON.md)** - DECISION REFERENCE
**Comparison of registry approaches**
- Buf Schema Registry (recommended)
- GitHub Packages
- Artifactory/Nexus
- Self-hosted Git
- Cost analysis
- Modern architecture diagram

**Best for:** Understanding why this approach is best

---

## 🏗️ Configuration Files (Already Created)

### Root Level
```
/Users/smartboy/proto/
├── buf.yaml                  # Linting & validation rules
├── buf.gen.yaml             # Code generation configuration
├── buf.lock                 # Dependency lock file
├── package.json             # npm configuration with Buf scripts
├── .github/
│   └── workflows/
│       ├── buf-publish.yml         # Validates & publishes to Buf registry
│       └── generate-artifacts.yml  # Auto-generates & publishes packages
└── proto/
    └── subscription.proto   # Central schema definition
```

### Key Features
- ✅ Lint validation on every commit
- ✅ Breaking change detection
- ✅ Multi-language code generation (TS + Java)
- ✅ Automatic artifact publishing
- ✅ Version management

---

## 🎯 Current Setup Status

### Central Repo (`/Users/smartboy/proto`)
| Component | Status | Details |
|-----------|--------|---------|
| Buf configuration | ✅ Ready | `buf.yaml`, `buf.gen.yaml` configured |
| Schema definition | ✅ Ready | `proto/subscription.proto` matches both services |
| GitHub workflows | ✅ Ready | CI/CD pipelines configured |
| Dependencies | ⏳ Pending | Need `BUF_TOKEN` in GitHub Secrets |
| Initial push | ⏳ Pending | Run `buf push` to initialize registry |

### Subscription Service (NestJS)
| Component | Status | Current | Target |
|-----------|--------|---------|--------|
| Proto files | ✅ | `src/proto/subscription.proto` | Remove (use npm package) |
| Code gen | ✅ | `src/pb/` (local) | Use `@org/proto-contracts` |
| package.json | ⏳ | `proto:gen` script | `proto:install` script |
| Imports | ⏳ | Local imports | `@org/proto-contracts` imports |
| gRPC Server | ✅ | Already implemented | Keep as-is, just change imports |

### Billing Service (Spring Boot)
| Component | Status | Current | Target |
|-----------|--------|---------|--------|
| Proto files | ✅ | `src/main/proto/subscription.proto` | Remove (use Maven package) |
| Code gen | ✅ | `src/main/generated/` (local) | Auto-generated from Maven artifact |
| build.gradle | ⏳ | Local protobuf config | Add `com.org:proto-contracts` dependency |
| Imports | ⏳ | Auto-generated locally | Same, but from Maven artifact |
| gRPC Client | ✅ | Already implemented | Keep as-is, just update imports |

---

## 🚀 Quick Start (Next 5 Minutes)

### 1. Read Overview
```bash
# Understand the current problem and solution
open README_SETUP.md
```

### 2. Initialize Buf Registry
```bash
cd /Users/smartboy/proto

# Install Buf CLI if needed
brew install bufbuild/buf/buf

# Login to Buf
buf registry login

# Push schema
buf push
```

### 3. Add GitHub Secret
```
GitHub Settings > Secrets
- Name: BUF_TOKEN
- Value: <your-buf-token-from-previous-step>
```

### 4. Follow Checklist
```bash
# Use interactive checklist to track progress
open QUICK_CHECKLIST.md
```

---

## 📋 Migration Steps (Next 1.5 Hours)

### Phase 1: Central Repository (15 mins)
- ✅ Configuration files created
- ⏳ Initialize Buf registry
- ⏳ Add GitHub secrets

**Files:** `buf.yaml`, `buf.gen.yaml`, GitHub workflows

### Phase 2: Subscription Service (20 mins)
- ⏳ Update `package.json`
- ⏳ Remove `src/proto/` directory
- ⏳ Update all imports
- ⏳ Test gRPC server

**Reference:** [EXAMPLES_NESTJS.md](EXAMPLES_NESTJS.md), [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)

### Phase 3: Billing Service (20 mins)
- ⏳ Update `build.gradle`
- ⏳ Remove `src/main/proto/` directory
- ⏳ Update all imports
- ⏳ Test gRPC client

**Reference:** [EXAMPLES_SPRINGBOOT.md](EXAMPLES_SPRINGBOOT.md), [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)

### Phase 4: Integration Testing (20 mins)
- ⏳ Start both services
- ⏳ Test gRPC communication
- ⏳ Test REST endpoints
- ⏳ Verify no errors

**Reference:** [QUICK_CHECKLIST.md](QUICK_CHECKLIST.md)

---

## 🏆 Benefits After Migration

### For Developers
✅ Single schema definition - no duplicates to sync
✅ Automatic code generation - one command for all languages
✅ Clear versioning - know exactly which version you're using
✅ IDE support - better autocomplete and type checking

### For Teams
✅ Breaking change detection - catches incompatible changes
✅ API governance - track schema evolution
✅ Audit trail - see who changed what
✅ Collaboration - comment on schemas, review changes

### For Operations
✅ CI/CD automation - no manual build steps
✅ Consistent deployments - all services use same version
✅ Reduced errors - compilation happens in CI
✅ Easy rollback - versions always available

---

## 📞 Support & Troubleshooting

### Common Issues

**Issue:** `npm install @org/proto-contracts` fails
```bash
# Solution: Configure GitHub token
npm set //npm.pkg.github.com/:_authToken=$GITHUB_TOKEN
```

**Issue:** `./gradlew build` can't find proto artifact
```bash
# Solution: Make sure Maven artifact is published
# Check GitHub Packages after buf push
```

**Issue:** gRPC connection refused
```bash
# Solution: Verify services are running
# Subscription: npm run start:dev (port 50051)
# Billing: ./gradlew bootRun (port 8080)
```

### Help Resources
- **Setup questions:** [README_SETUP.md](README_SETUP.md)
- **Code examples:** [EXAMPLES_NESTJS.md](EXAMPLES_NESTJS.md), [EXAMPLES_SPRINGBOOT.md](EXAMPLES_SPRINGBOOT.md)
- **Migration help:** [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)
- **Decision rationale:** [REGISTRY_COMPARISON.md](REGISTRY_COMPARISON.md)
- **Tracking progress:** [QUICK_CHECKLIST.md](QUICK_CHECKLIST.md)

---

## 🔄 Proto Change Workflow (Going Forward)

### 1. Update Schema
```bash
cd /Users/smartboy/proto
# Edit proto/subscription.proto
```

### 2. Validate Locally
```bash
buf lint
buf breaking --against '.git#branch=main'
buf format -w
```

### 3. Push to Git
```bash
git add proto/subscription.proto
git commit -m "feat: add new subscription status field"
git push origin main
```

### 4. CI/CD Takes Over
- buf-publish.yml: Validates and publishes to Buf
- generate-artifacts.yml: Generates code and publishes packages

### 5. Services Update
```bash
# Subscription Service
npm update @org/proto-contracts

# Billing Service
./gradlew build  # Auto-detects new version
```

---

## 📊 Architecture Overview

```
┌─────────────────────────────────────────┐
│  Buf Schema Registry                    │
│  (buf.build/YOUR_ORG/schemas)          │
│                                         │
│  - Versioning & Governance              │
│  - Breaking Change Detection            │
│  - API Documentation                    │
│  - Team Collaboration                   │
└────────┬────────────────────────────────┘
         │
    ┌────┴────────────────────────┐
    │                              │
    ▼                              ▼
npm Registry              Maven Central/GitHub
@org/proto-contracts     com.org:proto-contracts

    │                              │
    ▼                              ▼
Subscription Service         Billing Service
(NestJS gRPC Server)        (Spring Boot gRPC Client)
    │                              │
    └──────────── gRPC ────────────┘
```

---

## ✅ Verification Checklist

Before considering migration complete:

- [ ] Buf registry initialized and working
- [ ] Central proto schema in `proto/subscription.proto`
- [ ] NestJS service imports from `@org/proto-contracts`
- [ ] Spring Boot service imports from Maven artifact
- [ ] Both services start without proto-related errors
- [ ] gRPC communication works between services
- [ ] REST endpoints function correctly
- [ ] CI/CD pipelines execute successfully
- [ ] New proto versions auto-publish to registries
- [ ] Team understands proto change workflow

---

## 📖 Document Usage Guide

| Document | When to Use | Read Time |
|----------|------------|-----------|
| README_SETUP.md | Understanding the full setup | 10 min |
| QUICK_CHECKLIST.md | Tracking migration progress | 5 min (interactive) |
| MIGRATION_GUIDE.md | Detailed step-by-step instructions | 20 min |
| EXAMPLES_NESTJS.md | Writing NestJS server code | 15 min |
| EXAMPLES_SPRINGBOOT.md | Writing Spring Boot client code | 15 min |
| REGISTRY_COMPARISON.md | Justifying approach to team | 10 min |
| This file | Finding the right document | 5 min |

---

## 🎓 Next Steps

1. **Understand** - Read [README_SETUP.md](README_SETUP.md)
2. **Plan** - Review [MIGRATION_GUIDE.md](MIGRATION_GUIDE.md)
3. **Execute** - Follow [QUICK_CHECKLIST.md](QUICK_CHECKLIST.md)
4. **Code** - Use [EXAMPLES_NESTJS.md](EXAMPLES_NESTJS.md) and [EXAMPLES_SPRINGBOOT.md](EXAMPLES_SPRINGBOOT.md)
5. **Verify** - Run integration tests from checklist
6. **Deploy** - Push changes and monitor CI/CD
7. **Maintain** - Use proto change workflow for future updates

---

**Version:** 1.0.0
**Last Updated:** April 28, 2026
**Status:** Ready for Migration
