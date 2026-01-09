# 📘 Phase 3: Intermediate Concepts

> **Goal:** Master npm ecosystem, event-driven patterns, and secure configuration  
> **Duration:** 3-4 days  
> **Prerequisites:** Completed Phase 2

---

## 📚 What You'll Learn

| Topic | Concepts |
|-------|----------|
| npm & Third-party Modules | package.json, installing packages, semantic versioning |
| Custom Modules | Module patterns, factory, singleton |
| Event Emitter | Creating custom events, pub/sub pattern |
| Timers | setTimeout, setInterval, setImmediate, process.nextTick |
| Env Validation & Security | Validating config, secrets management |

---

## 1️⃣ npm & Third-party Modules

### Understanding package.json

```json
{
  "name": "my-backend-app",
  "version": "1.0.0",
  "description": "A Node.js backend application",
  "main": "src/index.js",
  "type": "module",
  "scripts": {
    "start": "node src/index.js",
    "dev": "node --watch src/index.js",
    "test": "node --test"
  },
  "dependencies": {
    "express": "^4.18.2",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### Key Fields Explained

| Field | Purpose |
|-------|---------|
| `main` | Entry point when package is imported |
| `type` | `"module"` for ESM, `"commonjs"` for CJS |
| `scripts` | Custom npm commands |
| `dependencies` | Packages needed to run |
| `devDependencies` | Packages only needed for development |
| `engines` | Required Node.js version |

### Semantic Versioning

```
  ^4.18.2
   │  │  │
   │  │  └── Patch: Bug fixes (safe to update)
   │  └───── Minor: New features (backward compatible)
   └──────── Major: Breaking changes (careful!)

^4.18.2  →  Allows 4.x.x (minor + patch updates)
~4.18.2  →  Allows 4.18.x (only patch updates)
4.18.2   →  Exact version only
```

### Installing Packages

```bash
# Install a package (adds to dependencies)
npm install express

# Install dev dependency
npm install --save-dev nodemon

# Install specific version
npm install lodash@4.17.21

# Install from package.json
npm install

# Remove a package
npm uninstall express

# Update packages
npm update

# Check for outdated packages
npm outdated
```

### package-lock.json

**Why it exists:**
- Locks exact versions of all dependencies (including sub-dependencies)
- Ensures everyone gets the same versions
- **Always commit this file!**

### 🎯 Real-World Best Practices

```json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "nodemon src/index.js",
    "build": "tsc",
    "test": "jest",
    "lint": "eslint src/",
    "prepare": "husky install"
  }
}
```

### 💼 Interview Relevance

**Common Questions:**
- What's the difference between dependencies and devDependencies?
- Explain semantic versioning
- What is package-lock.json and why is it important?
- How do you handle security vulnerabilities in packages?

---

## 2️⃣ Creating Custom Modules

### Module Patterns

#### Factory Pattern
Returns a new instance each time:

```javascript
// logger.js - Factory Pattern
export function createLogger(prefix) {
  return {
    log(message) {
      console.log(`[${prefix}] ${new Date().toISOString()}: ${message}`);
    },
    error(message) {
      console.error(`[${prefix}] ERROR: ${message}`);
    }
  };
}
```

```javascript
// app.js
import { createLogger } from './logger.js';

const authLogger = createLogger('AUTH');
const dbLogger = createLogger('DATABASE');

authLogger.log('User logged in');
dbLogger.log('Connection established');
```

#### Singleton Pattern
Returns the same instance every time:

```javascript
// database.js - Singleton Pattern
class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }
    this.connection = null;
    Database.instance = this;
  }

  connect(url) {
    if (this.connection) {
      console.log('Already connected');
      return this.connection;
    }
    console.log('Connecting to:', url);
    this.connection = { url, connected: true };
    return this.connection;
  }

  query(sql) {
    if (!this.connection) throw new Error('Not connected');
    console.log('Executing:', sql);
    return [];
  }
}

export default new Database(); // Export instance, not class
```

```javascript
// user-service.js
import db from './database.js';
db.connect('mongodb://localhost'); // First connection

// order-service.js
import db from './database.js';
db.connect('mongodb://localhost'); // Same connection reused!
```

#### Revealing Module Pattern

```javascript
// cache.js - Revealing Module Pattern
function createCache() {
  // Private variables
  const store = new Map();
  let hitCount = 0;
  let missCount = 0;

  // Private function
  function isExpired(item) {
    return item.expiry && Date.now() > item.expiry;
  }

  // Public interface
  return {
    get(key) {
      const item = store.get(key);
      if (!item || isExpired(item)) {
        missCount++;
        return null;
      }
      hitCount++;
      return item.value;
    },

    set(key, value, ttlMs = null) {
      store.set(key, {
        value,
        expiry: ttlMs ? Date.now() + ttlMs : null
      });
    },

    stats() {
      return { hitCount, missCount, size: store.size };
    }
  };
}

export const cache = createCache();
```

### Organizing Large Modules

```
📁 src/
├── modules/
│   ├── user/
│   │   ├── index.js          (re-exports)
│   │   ├── user.service.js
│   │   ├── user.repository.js
│   │   └── user.validation.js
│   └── order/
│       ├── index.js
│       ├── order.service.js
│       └── order.repository.js
└── index.js
```

```javascript
// modules/user/index.js - Barrel file
export { UserService } from './user.service.js';
export { UserRepository } from './user.repository.js';
export { validateUser } from './user.validation.js';
```

```javascript
// Anywhere in app
import { UserService, validateUser } from './modules/user/index.js';
```

---

## 3️⃣ Event Emitter

### What is Event Emitter?
A pattern where objects emit named events that trigger listener functions.

### Basic Usage

```javascript
import { EventEmitter } from 'events';

const emitter = new EventEmitter();

// Subscribe to an event
emitter.on('userCreated', (user) => {
  console.log('New user created:', user.name);
});

// Subscribe once (auto-removes after first call)
emitter.once('userCreated', (user) => {
  console.log('First user ever!');
});

// Emit an event
emitter.emit('userCreated', { id: 1, name: 'John' });
emitter.emit('userCreated', { id: 2, name: 'Jane' }); // "First user" won't log
```

### Creating Custom Event Emitters

```javascript
// order-system.js
import { EventEmitter } from 'events';

class OrderSystem extends EventEmitter {
  constructor() {
    super();
    this.orders = [];
  }

  createOrder(order) {
    order.id = Date.now();
    order.status = 'pending';
    this.orders.push(order);
    
    // Emit event for other parts of system
    this.emit('orderCreated', order);
    return order;
  }

  completeOrder(orderId) {
    const order = this.orders.find(o => o.id === orderId);
    if (!order) throw new Error('Order not found');
    
    order.status = 'completed';
    this.emit('orderCompleted', order);
    return order;
  }
}

export const orderSystem = new OrderSystem();
```

```javascript
// app.js
import { orderSystem } from './order-system.js';

// Set up listeners (like microservices)
orderSystem.on('orderCreated', (order) => {
  console.log('📧 Sending confirmation email for order:', order.id);
});

orderSystem.on('orderCreated', (order) => {
  console.log('📊 Updating analytics for new order');
});

orderSystem.on('orderCompleted', (order) => {
  console.log('📦 Scheduling shipment for order:', order.id);
});

// Create order - all listeners fire automatically
orderSystem.createOrder({ product: 'Laptop', quantity: 1 });
```

### Error Handling

```javascript
// Always handle 'error' events!
emitter.on('error', (err) => {
  console.error('Event error:', err.message);
});

// If no 'error' listener, Node crashes!
emitter.emit('error', new Error('Something went wrong'));
```

### 🎯 Real-World Use Cases

| Use Case | Example |
|----------|---------|
| **Logging** | Emit events for each log level |
| **Notifications** | Trigger emails/SMS on events |
| **Caching** | Invalidate cache on data change |
| **Metrics** | Record metrics on operations |
| **Webhooks** | Trigger external calls on events |

### 💼 Interview Relevance

**Common Questions:**
- What is the Observer pattern?
- What's the difference between `on` and `once`?
- How do you handle errors in EventEmitter?
- How would you implement a pub/sub system?

---

## 4️⃣ Timers in Depth

### Timer Functions

```javascript
// ─────────────────────────────────────
// setTimeout - Execute once after delay
// ─────────────────────────────────────
const timeoutId = setTimeout(() => {
  console.log('Runs after 2 seconds');
}, 2000);

// Cancel before it runs
clearTimeout(timeoutId);
```

```javascript
// ─────────────────────────────────────
// setInterval - Execute repeatedly
// ─────────────────────────────────────
let count = 0;
const intervalId = setInterval(() => {
  count++;
  console.log(`Heartbeat ${count}`);
  
  if (count >= 5) {
    clearInterval(intervalId); // Stop after 5 times
  }
}, 1000);
```

### Node.js Specific Timers

```javascript
// ─────────────────────────────────────
// setImmediate - Execute after I/O
// ─────────────────────────────────────
setImmediate(() => {
  console.log('Runs after current I/O operations');
});

// ─────────────────────────────────────
// process.nextTick - Execute ASAP (before I/O)
// ─────────────────────────────────────
process.nextTick(() => {
  console.log('Runs before any I/O or timers');
});
```

### Execution Order

```javascript
console.log('1: Synchronous');

setTimeout(() => console.log('2: setTimeout'), 0);
setImmediate(() => console.log('3: setImmediate'));
process.nextTick(() => console.log('4: nextTick'));

console.log('5: Synchronous');

// Output:
// 1: Synchronous
// 5: Synchronous
// 4: nextTick        ← Runs first (before I/O)
// 2: setTimeout      ← Timer queue
// 3: setImmediate    ← Check queue (after I/O)
```

### Practical Timer Patterns

```javascript
// Debounce - Wait for pause in calls
function debounce(fn, delay) {
  let timeoutId;
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}

// Throttle - Limit call frequency
function throttle(fn, limit) {
  let inThrottle;
  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Delay utility
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

await delay(1000); // Wait 1 second
```

---

## 5️⃣ Environment Validation & Security

### Validating at Startup

```javascript
// config/validate.js
export function validateEnv() {
  const required = [
    'NODE_ENV',
    'PORT',
    'DATABASE_URL',
    'JWT_SECRET',
    'API_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    console.error('❌ Missing required environment variables:');
    missing.forEach(key => console.error(`   - ${key}`));
    process.exit(1);
  }

  console.log('✅ All required environment variables present');
}
```

### Type-Safe Configuration

```javascript
// config/index.js
import 'dotenv/config';

function parseBoolean(value, defaultValue = false) {
  if (value === undefined) return defaultValue;
  return value === 'true' || value === '1';
}

function parseNumber(value, defaultValue) {
  if (value === undefined) return defaultValue;
  const parsed = parseInt(value, 10);
  if (isNaN(parsed)) return defaultValue;
  return parsed;
}

export const config = Object.freeze({
  nodeEnv: process.env.NODE_ENV || 'development',
  isProduction: process.env.NODE_ENV === 'production',
  isDevelopment: process.env.NODE_ENV === 'development',
  
  server: {
    port: parseNumber(process.env.PORT, 3000),
    host: process.env.HOST || '0.0.0.0'
  },

  database: {
    url: process.env.DATABASE_URL,
    poolSize: parseNumber(process.env.DB_POOL_SIZE, 10)
  },

  auth: {
    jwtSecret: process.env.JWT_SECRET,
    jwtExpiresIn: process.env.JWT_EXPIRES_IN || '7d'
  },

  features: {
    enableCache: parseBoolean(process.env.ENABLE_CACHE, true),
    debugMode: parseBoolean(process.env.DEBUG)
  }
});

// Freeze prevents modification
```

### Security Best Practices

```javascript
// 1. Never log secrets
console.log('Starting server...'); // ✅
console.log('API Key:', config.apiKey); // ❌

// 2. Use different secrets per environment
// dev: JWT_SECRET=dev-secret-not-for-prod
// prod: JWT_SECRET=super-long-random-string-32-chars

// 3. Rotate secrets regularly
// 4. Use secret management in production (AWS Secrets Manager, Vault)
```

### Complete .gitignore for Node.js

```gitignore
# Environment files
.env
.env.local
.env.*.local
.env.production
.env.staging

# Dependencies
node_modules/

# Build output
dist/
build/

# Logs
logs/
*.log
npm-debug.log*

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/

# Secrets
*.pem
*.key
```

---

## 🧪 Practice Tasks

### Task 1: npm Project Setup
**Create a properly structured npm project**

```
📁 task-1/
├── package.json
├── .gitignore
├── src/
│   └── index.js
└── README.md
```

**Requirements:**
- Initialize with `npm init`
- Add scripts: start, dev, test
- Install: lodash (dependency), nodemon (devDependency)
- Use lodash to manipulate some arrays/objects
- Create proper .gitignore

---

### Task 2: Custom Logger Module
**Build a configurable logger using module patterns**

```
📁 task-2/
├── lib/
│   └── logger.js
├── app.js
└── package.json
```

**Requirements:**
- Use Factory pattern
- Support log levels: debug, info, warn, error
- Configurable prefix and timestamp format
- Optional file output
- Filter logs by level (e.g., "production" mode only shows warn+error)

---

### Task 3: Order Event System
**Build an order processing system using EventEmitter**

```
📁 task-3/
├── src/
│   ├── order-system.js    (EventEmitter class)
│   ├── email-service.js   (listener)
│   ├── inventory-service.js (listener)
│   └── analytics-service.js (listener)
└── app.js
```

**Requirements:**
- OrderSystem emits: `orderCreated`, `orderPaid`, `orderShipped`, `orderCancelled`
- EmailService: sends emails on all events
- InventoryService: updates stock on orderCreated, restores on orderCancelled
- AnalyticsService: logs all events with timestamps
- Proper error handling

---

### Task 4: Job Scheduler
**Create a simple job scheduler using timers**

```
📁 task-4/
├── scheduler.js
└── app.js
```

**Requirements:**
- Schedule functions to run at intervals
- Support job IDs and cancellation
- Methods: `schedule(name, fn, intervalMs)`, `cancel(name)`, `list()`
- Add a "run once after delay" option
- Log when jobs run

---

### Task 5: Secure Config System
**Build production-ready configuration**

```
📁 task-5/
├── .env
├── .env.example
├── .gitignore
├── config/
│   ├── index.js
│   ├── validate.js
│   └── environments/
│       ├── development.js
│       └── production.js
└── app.js
```

**Requirements:**
- Environment-based configs (dev, prod)
- Validate required vars at startup
- Type conversion with defaults
- Freeze config object
- Never log sensitive values
- Create .env.example template

---

## ✅ Phase 3 Checklist

Before moving to Phase 4, make sure you can:

- [ ] Create and manage package.json properly
- [ ] Understand semantic versioning
- [ ] Implement Factory and Singleton patterns
- [ ] Create and use EventEmitter
- [ ] Use setTimeout, setInterval, setImmediate, process.nextTick
- [ ] Validate environment variables at startup
- [ ] Create type-safe configuration
- [ ] Follow security best practices for secrets

---

*Next: [Phase 4: Project-Ready Skills →](../phase-4/README.md)*
