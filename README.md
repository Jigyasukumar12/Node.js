# 🚀 Node.js Study Guide
## Modules | Async Code | Environment Variables

> **Focus:** Project-Oriented + Interview-Ready  
> **Level:** Beginner → Intermediate → Project-Ready  
> **Approach:** Practical usage, patterns, and real backend use cases

---

## 📚 STEP 1: Complete Study List

### 1️⃣ Node.js Modules

| Subtopic | Description |
|----------|-------------|
| **CommonJS (CJS) Modules** | `require()`, `module.exports`, `exports` |
| **ES Modules (ESM)** | `import`, `export`, dynamic imports |
| **Module Resolution** | How Node finds modules (relative, absolute, node_modules) |
| **Built-in Modules** | `fs`, `path`, `os`, `http`, `url`, `crypto`, `events` |
| **Third-party Modules** | npm, installing packages, `package.json`, `node_modules` |
| **Creating Custom Modules** | Structuring code into reusable modules |
| **Module Caching** | How Node caches modules and implications |
| **Circular Dependencies** | Understanding and avoiding circular imports |
| **Module Patterns** | Factory, Singleton, Revealing Module patterns |
| **Package.json Deep Dive** | `main`, `type`, `exports`, `scripts`, `dependencies` |

---

### 2️⃣ Asynchronous Code in Node.js

| Subtopic | Description |
|----------|-------------|
| **Callbacks** | Traditional async pattern, callback hell, error-first callbacks |
| **Promises** | Creating, chaining, `Promise.all`, `Promise.race`, `Promise.allSettled` |
| **Async/Await** | Modern syntax, error handling with try/catch |
| **Event Emitter** | Custom events, `on`, `emit`, `once`, `removeListener` |
| **Streams** | Readable, Writable, Duplex, Transform streams |
| **Timers** | `setTimeout`, `setInterval`, `setImmediate`, `process.nextTick` |
| **File System Async** | Async file operations with `fs.promises` |
| **Error Handling** | Handling async errors, unhandled rejections |
| **Concurrent Operations** | Running multiple async ops, controlling concurrency |
| **Real-world Patterns** | Retry logic, debouncing, throttling, queue processing |

---

### 3️⃣ Environment Variables

| Subtopic | Description |
|----------|-------------|
| **process.env Basics** | Accessing environment variables in Node |
| **Setting Env Variables** | CLI, OS-level, cross-platform |
| **.env Files** | Using `dotenv` package |
| **Configuration Management** | Separating config from code |
| **Environment-based Config** | development, staging, production configs |
| **Secrets Management** | Best practices for sensitive data |
| **Validation** | Validating required env variables at startup |
| **Type Conversion** | Converting string env vars to proper types |
| **Default Values** | Providing fallbacks for optional env vars |
| **Security Best Practices** | Never commit secrets, gitignore patterns |

---

## 📊 STEP 2: Phase Division

```
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1: Foundations                                           │
│  ─────────────────────                                          │
│  • CommonJS Modules (require/exports)                           │
│  • Built-in Modules (fs, path, os)                              │
│  • Callbacks & Error-first Pattern                              │
│  • process.env Basics                                           │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 2: Modern Patterns                                       │
│  ────────────────────────                                       │
│  • ES Modules (import/export)                                   │
│  • Promises & Promise Methods                                   │
│  • Async/Await                                                  │
│  • dotenv & Configuration Management                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 3: Intermediate Concepts                                 │
│  ──────────────────────────────                                 │
│  • Third-party Modules & npm                                    │
│  • Module Patterns & Custom Modules                             │
│  • Event Emitter                                                │
│  • Timers in depth                                              │
│  • Env Validation & Security                                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 4: Project-Ready Skills                                  │
│  ───────────────────────────────                                │
│  • Streams                                                      │
│  • Module Caching & Circular Dependencies                       │
│  • Advanced Async Patterns (retry, queue, concurrency)          │
│  • Production Config Patterns                                   │
│  • Interview Preparation Summary                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 Phase READMEs

| Phase | Focus | Link |
|-------|-------|------|
| **Phase 1** | Foundations | [phase-1/README.md](./phase-1/README.md) |
| **Phase 2** | Modern Patterns | [phase-2/README.md](./phase-2/README.md) |
| **Phase 3** | Intermediate Concepts | [phase-3/README.md](./phase-3/README.md) |
| **Phase 4** | Project-Ready Skills | [phase-4/README.md](./phase-4/README.md) |

---

## 🎯 How to Use This Guide

1. **Start with Phase 1** - Don't skip! These are the building blocks
2. **Complete all practice tasks** - Theory without practice = wasted time
3. **Build something real** - After Phase 4, build a complete project
4. **Review for interviews** - Each phase has interview relevance sections

---

## 💡 Prerequisites

- JavaScript fundamentals (variables, functions, arrays, objects)
- Basic command line knowledge
- Node.js installed (v18+ recommended)
- A code editor (VS Code recommended)

---

Start with [Phase 1 Foundations](./phase-1/README.md) →
