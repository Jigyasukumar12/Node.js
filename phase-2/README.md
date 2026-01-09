# 📘 Phase 2: Modern Patterns

> **Goal:** Master modern JavaScript async patterns and ES Modules  
> **Duration:** 3-4 days  
> **Prerequisites:** Completed Phase 1

---

## 📚 What You'll Learn

| Topic | Concepts |
|-------|----------|
| ES Modules | `import`, `export`, dynamic imports |
| Promises | Creating, chaining, `Promise.all/race/allSettled` |
| Async/Await | Modern syntax, error handling |
| Configuration | dotenv, environment-based config |

---

## 1️⃣ ES Modules (ESM)

### What Changed?
ES Modules are the official JavaScript module system (standardized in ES6). Node.js now fully supports them!

### Enabling ES Modules

**Option 1:** Use `.mjs` extension
```javascript
// math.mjs - treated as ES Module
export function add(a, b) {
  return a + b;
}
```

**Option 2:** Set `"type": "module"` in package.json (Recommended)
```json
{
  "name": "my-app",
  "type": "module"
}
```

### Basic Syntax

```javascript
// ─────────────────────────────────────
// utils.js - Named Exports
// ─────────────────────────────────────
export function formatDate(date) {
  return date.toISOString().split('T')[0];
}

export function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

export const VERSION = '1.0.0';
```

```javascript
// ─────────────────────────────────────
// app.js - Named Imports
// ─────────────────────────────────────
import { formatDate, capitalize, VERSION } from './utils.js';
// Note: .js extension is REQUIRED in ESM!

console.log(formatDate(new Date()));
console.log(capitalize('hello'));
console.log(VERSION);
```

### Default Exports

```javascript
// ─────────────────────────────────────
// database.js - Default Export
// ─────────────────────────────────────
class Database {
  connect() { /* ... */ }
  query(sql) { /* ... */ }
}

export default Database;
```

```javascript
// ─────────────────────────────────────
// app.js - Default Import
// ─────────────────────────────────────
import Database from './database.js';
// Can use any name for default imports

const db = new Database();
db.connect();
```

### Combining Default and Named

```javascript
// ─────────────────────────────────────
// api.js
// ─────────────────────────────────────
export default class ApiClient { /* ... */ }
export const BASE_URL = 'https://api.example.com';
export function handleError(err) { /* ... */ }

// ─────────────────────────────────────
// app.js
// ─────────────────────────────────────
import ApiClient, { BASE_URL, handleError } from './api.js';
```

### Dynamic Imports (Code Splitting)

```javascript
// Load modules conditionally at runtime
async function loadFeature(featureName) {
  if (featureName === 'charts') {
    const { renderChart } = await import('./features/charts.js');
    renderChart();
  } else if (featureName === 'reports') {
    const { generateReport } = await import('./features/reports.js');
    generateReport();
  }
}
```

### Key Differences: CJS vs ESM

| Feature | CommonJS | ES Modules |
|---------|----------|------------|
| Syntax | `require()` / `module.exports` | `import` / `export` |
| Loading | Synchronous | Asynchronous |
| File Extension | `.js` optional | `.js` required |
| this | `module.exports` | `undefined` |
| Dynamic | `require(variable)` works | Need `import()` |
| Top-level await | ❌ Not allowed | ✅ Allowed |

### 🎯 Why This Matters in Real Projects

| Scenario | Benefit |
|----------|---------|
| **Modern Frameworks** | Most new packages use ESM |
| **Tree Shaking** | Bundlers can remove unused code |
| **Browser Compatibility** | Same syntax works in browsers |
| **Top-level Await** | Cleaner initialization code |

### 💼 Interview Relevance

**Common Questions:**
- What are the differences between CommonJS and ES Modules?
- How do you use ESM in Node.js?
- What is dynamic import and when would you use it?

---

## 2️⃣ Promises

### What is a Promise?
A Promise represents a value that may be available now, later, or never.

### Promise States
```
┌─────────────┐
│   PENDING   │ ─── Initial state
└──────┬──────┘
       │
       ├──────────────┐
       ▼              ▼
┌─────────────┐ ┌─────────────┐
│  FULFILLED  │ │  REJECTED   │
│  (resolved) │ │  (error)    │
└─────────────┘ └─────────────┘
```

### Creating Promises

```javascript
// ─────────────────────────────────────
// Basic Promise Creation
// ─────────────────────────────────────
function fetchUser(userId) {
  return new Promise((resolve, reject) => {
    // Simulate async operation
    setTimeout(() => {
      if (userId <= 0) {
        reject(new Error('Invalid user ID'));
        return;
      }
      
      resolve({ id: userId, name: 'John Doe' });
    }, 1000);
  });
}

// Usage
fetchUser(123)
  .then(user => console.log('User:', user))
  .catch(err => console.error('Error:', err.message));
```

### Promise Chaining

```javascript
fetchUser(123)
  .then(user => {
    console.log('Got user:', user.name);
    return fetchUserPosts(user.id); // Return another promise
  })
  .then(posts => {
    console.log('Got posts:', posts.length);
    return fetchComments(posts[0].id);
  })
  .then(comments => {
    console.log('Got comments:', comments);
  })
  .catch(err => {
    // Catches error from ANY step above
    console.error('Something failed:', err.message);
  })
  .finally(() => {
    // Always runs, regardless of success/failure
    console.log('Cleanup complete');
  });
```

### Promise Static Methods

```javascript
// ─────────────────────────────────────
// Promise.all - Wait for ALL to complete
// ─────────────────────────────────────
const userPromise = fetchUser(1);
const postsPromise = fetchPosts(1);
const settingsPromise = fetchSettings(1);

Promise.all([userPromise, postsPromise, settingsPromise])
  .then(([user, posts, settings]) => {
    console.log('All data loaded:', user, posts, settings);
  })
  .catch(err => {
    // If ANY promise fails, catch is called
    console.error('One of them failed:', err);
  });
```

```javascript
// ─────────────────────────────────────
// Promise.allSettled - Get all results regardless of failure
// ─────────────────────────────────────
Promise.allSettled([fetchUser(1), fetchUser(-1), fetchUser(2)])
  .then(results => {
    results.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        console.log(`User ${index}: ${result.value.name}`);
      } else {
        console.log(`User ${index} failed: ${result.reason.message}`);
      }
    });
  });
```

```javascript
// ─────────────────────────────────────
// Promise.race - First to complete wins
// ─────────────────────────────────────
const timeout = new Promise((_, reject) => 
  setTimeout(() => reject(new Error('Timeout!')), 5000)
);

Promise.race([fetchUser(1), timeout])
  .then(user => console.log('Got user before timeout:', user))
  .catch(err => console.error(err.message));
```

```javascript
// ─────────────────────────────────────
// Promise.any - First SUCCESS wins (ignores rejections)
// ─────────────────────────────────────
Promise.any([
  fetchFromServer1(),
  fetchFromServer2(),
  fetchFromServer3()
])
  .then(result => console.log('First successful:', result))
  .catch(err => console.log('All failed'));
```

### 💼 Interview Relevance

**Common Questions:**
- What's the difference between `Promise.all` and `Promise.allSettled`?
- How do you implement a timeout with promises?
- What happens if one promise in `Promise.all` fails?

---

## 3️⃣ Async/Await

### The Modern Way
Async/await is syntactic sugar over Promises, making async code look synchronous.

### Basic Syntax

```javascript
// ─────────────────────────────────────
// Compare Promise .then() vs async/await
// ─────────────────────────────────────

// Promise style
function getUser(id) {
  return fetchUser(id)
    .then(user => fetchUserPosts(user.id))
    .then(posts => posts);
}

// Async/await style (cleaner!)
async function getUser(id) {
  const user = await fetchUser(id);
  const posts = await fetchUserPosts(user.id);
  return posts;
}
```

### Error Handling

```javascript
// ─────────────────────────────────────
// try/catch for async/await
// ─────────────────────────────────────
async function loadUserData(userId) {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchPosts(user.id);
    return { user, posts };
  } catch (error) {
    console.error('Failed to load data:', error.message);
    throw error; // Re-throw if needed
  } finally {
    console.log('Cleanup');
  }
}
```

### Parallel Execution

```javascript
// ❌ WRONG: Sequential (slow)
async function loadData() {
  const users = await fetchUsers();    // Wait 1 second
  const products = await fetchProducts(); // Wait 1 second
  const orders = await fetchOrders();     // Wait 1 second
  // Total: 3 seconds
}

// ✅ CORRECT: Parallel (fast)
async function loadData() {
  const [users, products, orders] = await Promise.all([
    fetchUsers(),
    fetchProducts(),
    fetchOrders()
  ]);
  // Total: 1 second (parallel)
}
```

### fs.promises - Modern File Operations

```javascript
import { readFile, writeFile, mkdir } from 'fs/promises';
import path from 'path';

async function processFile(filename) {
  try {
    // Read file
    const content = await readFile(filename, 'utf8');
    
    // Process content
    const processed = content.toUpperCase();
    
    // Ensure output directory exists
    await mkdir('output', { recursive: true });
    
    // Write result
    const outputPath = path.join('output', 'result.txt');
    await writeFile(outputPath, processed);
    
    console.log('File processed successfully!');
  } catch (error) {
    if (error.code === 'ENOENT') {
      console.error('File not found:', filename);
    } else {
      throw error;
    }
  }
}
```

### Top-Level Await (ESM only)

```javascript
// config.js (ES Module)
import { readFile } from 'fs/promises';

// Can use await at the top level!
const configData = await readFile('config.json', 'utf8');
export const config = JSON.parse(configData);
```

### 🎯 Real-World Example: API Call

```javascript
async function fetchWithRetry(url, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      return await response.json();
    } catch (error) {
      console.log(`Attempt ${attempt} failed:`, error.message);
      if (attempt === maxRetries) throw error;
      // Wait before retrying (exponential backoff)
      await new Promise(r => setTimeout(r, 1000 * attempt));
    }
  }
}
```

### 💼 Interview Relevance

**Common Questions:**
- What's the difference between Promises and async/await?
- How do you run multiple async operations in parallel?
- How do you handle errors in async/await?
- Can you use await outside an async function?

---

## 4️⃣ dotenv & Configuration Management

### Why dotenv?
Setting environment variables manually is tedious. dotenv loads variables from a `.env` file automatically.

### Setup

```bash
npm init -y
npm install dotenv
```

### Using dotenv

```
📁 project/
├── .env              (your secrets - NEVER commit!)
├── .env.example      (template - safe to commit)
├── .gitignore        (must include .env)
├── config.js
└── app.js
```

```env
# .env
NODE_ENV=development
PORT=3000
DATABASE_URL=mongodb://localhost:27017/myapp
API_KEY=sk-secret-key-12345
DEBUG=true
```

```env
# .env.example (safe to commit)
NODE_ENV=development
PORT=3000
DATABASE_URL=mongodb://localhost:27017/myapp
API_KEY=your-api-key-here
DEBUG=false
```

```javascript
// config.js
import 'dotenv/config';  // Load .env file

export const config = {
  nodeEnv: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT, 10) || 3000,
  databaseUrl: process.env.DATABASE_URL,
  apiKey: process.env.API_KEY,
  debug: process.env.DEBUG === 'true'
};

// Validate required variables
const required = ['DATABASE_URL', 'API_KEY'];
for (const key of required) {
  if (!process.env[key]) {
    throw new Error(`Missing required env variable: ${key}`);
  }
}
```

```javascript
// app.js
import { config } from './config.js';

console.log('Environment:', config.nodeEnv);
console.log('Port:', config.port);
console.log('Debug mode:', config.debug);
```

### Environment-Based Configuration

```javascript
// config.js - Advanced pattern
import 'dotenv/config';

const baseConfig = {
  appName: 'MyApp',
  port: parseInt(process.env.PORT, 10) || 3000,
};

const envConfigs = {
  development: {
    dbUrl: 'mongodb://localhost:27017/dev',
    logLevel: 'debug',
    apiUrl: 'http://localhost:3001'
  },
  production: {
    dbUrl: process.env.DATABASE_URL,
    logLevel: 'error',
    apiUrl: 'https://api.myapp.com'
  },
  test: {
    dbUrl: 'mongodb://localhost:27017/test',
    logLevel: 'silent',
    apiUrl: 'http://localhost:3001'
  }
};

const env = process.env.NODE_ENV || 'development';
export const config = { ...baseConfig, ...envConfigs[env] };
```

### 🛡️ Security Best Practices

```gitignore
# .gitignore - ALWAYS include these
.env
.env.local
.env.*.local
*.pem
*.key
```

```javascript
// Never log secrets!
console.log('Config loaded for:', config.nodeEnv);
// ❌ console.log('API Key:', config.apiKey);
```

### 💼 Interview Relevance

**Common Questions:**
- How do you manage configuration in Node.js?
- Why shouldn't you commit .env files?
- How do you handle different configs for different environments?

---

## 🧪 Practice Tasks

### Task 1: ES Module Refactor
**Convert a CommonJS project to ES Modules**

```
📁 task-1/
├── package.json      (add "type": "module")
├── utils/
│   ├── strings.js    (export: capitalize, slugify, truncate)
│   └── arrays.js     (export: unique, shuffle, chunk)
└── app.js            (import and use all utils)
```

**Requirements:**
- Use named exports in utility files
- Create `index.js` that re-exports all utilities
- Use proper `.js` extensions in imports

---

### Task 2: Promise-Based File Processor
**Create a file processor using promises**

```
📁 task-2/
├── processor.js
└── data/
    └── input.txt
```

**Requirements:**
- Read a text file
- Count words, lines, and characters
- Write stats to `stats.json`
- Use `fs/promises` (not callbacks)
- Handle file not found gracefully

---

### Task 3: Parallel Data Fetcher
**Fetch data from multiple sources in parallel**

```
📁 task-3/
├── fetcher.js
└── app.js
```

**Requirements:**
- Create functions that simulate API calls (use setTimeout)
  - `fetchUser(id)` - returns after 1 second
  - `fetchPosts(userId)` - returns after 1.5 seconds
  - `fetchComments(postId)` - returns after 0.5 seconds
- Use `Promise.all` to fetch user, posts, and comments in parallel
- Add a timeout: if it takes more than 3 seconds, reject
- Use `Promise.allSettled` to handle partial failures

---

### Task 4: Configuration System
**Build a complete configuration system**

```
📁 task-4/
├── .env
├── .env.example
├── .gitignore
├── config/
│   └── index.js
└── app.js
```

**Requirements:**
- Use dotenv
- Create configs for: development, production, test
- Validate required variables at startup
- Type-convert appropriately (string → number, boolean)
- Create `.env.example` template

**Config should include:**
- PORT, NODE_ENV, LOG_LEVEL
- DATABASE_HOST, DATABASE_PORT, DATABASE_NAME
- API_SECRET (required!)

---

### Task 5: Async File Sync Tool
**Build a folder sync utility**

```
📁 task-5/
├── sync.js
├── source/
│   ├── file1.txt
│   └── file2.txt
└── backup/
```

**Requirements:**
- Use ES Modules
- Read all files from `source/` directory
- Copy each file to `backup/` with timestamp prefix
  - Example: `2024-01-15_file1.txt`
- Use async/await with `fs/promises`
- Run copy operations in parallel
- Log progress for each file
- Handle errors gracefully

---

## ✅ Phase 2 Checklist

Before moving to Phase 3, make sure you can:

- [ ] Create and use ES Modules with import/export
- [ ] Understand when to use default vs named exports
- [ ] Create and chain Promises
- [ ] Use Promise.all, Promise.race, Promise.allSettled
- [ ] Write async/await code with proper error handling
- [ ] Use fs/promises for modern file operations
- [ ] Set up dotenv for environment configuration
- [ ] Create environment-based config patterns

---

*Next: [Phase 3: Intermediate Concepts →](../phase-3/README.md)*
