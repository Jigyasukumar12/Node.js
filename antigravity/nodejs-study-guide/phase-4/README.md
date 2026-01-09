# 📘 Phase 4: Project-Ready Skills

> **Goal:** Master advanced async patterns, streams, and production-level configuration  
> **Duration:** 4-5 days  
> **Prerequisites:** Completed Phase 3

---

## 📚 What You'll Learn

| Topic | Concepts |
|-------|----------|
| Streams | Readable, Writable, Transform, piping |
| Module Caching | How caching works, circular dependencies |
| Advanced Async | Retry logic, concurrency control, queues |
| Production Config | Multi-environment, secrets, deployment patterns |
| Interview Prep | Common questions and answers |

---

## 1️⃣ Streams

### What are Streams?
Streams process data piece by piece, instead of loading everything into memory. Essential for handling large files, network requests, and real-time data.

### Stream Types

| Type | Example | Methods |
|------|---------|---------|
| **Readable** | Reading a file | `.read()`, `.pipe()`, events: `data`, `end`, `error` |
| **Writable** | Writing to file | `.write()`, `.end()`, events: `drain`, `finish`, `error` |
| **Duplex** | TCP socket | Both read and write |
| **Transform** | Compression | Modify data as it passes through |

### Reading Files with Streams

```javascript
import { createReadStream } from 'fs';

// ─────────────────────────────────────
// Reading a large file (memory efficient)
// ─────────────────────────────────────
const stream = createReadStream('large-file.txt', { 
  encoding: 'utf8',
  highWaterMark: 64 * 1024  // 64KB chunks
});

stream.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
});

stream.on('end', () => {
  console.log('Finished reading');
});

stream.on('error', (err) => {
  console.error('Read error:', err.message);
});
```

### Writing with Streams

```javascript
import { createWriteStream } from 'fs';

const stream = createWriteStream('output.txt');

stream.write('Line 1\n');
stream.write('Line 2\n');
stream.write('Line 3\n');
stream.end(); // Signal we're done

stream.on('finish', () => {
  console.log('Write completed');
});

stream.on('error', (err) => {
  console.error('Write error:', err.message);
});
```

### Piping Streams

```javascript
import { createReadStream, createWriteStream } from 'fs';

// ─────────────────────────────────────
// Copy file using pipe (simplest way)
// ─────────────────────────────────────
const source = createReadStream('input.txt');
const destination = createWriteStream('output.txt');

source.pipe(destination);

source.on('error', (err) => console.error('Read error:', err));
destination.on('error', (err) => console.error('Write error:', err));
destination.on('finish', () => console.log('Copy complete'));
```

### Transform Streams

```javascript
import { Transform } from 'stream';
import { createReadStream, createWriteStream } from 'fs';

// ─────────────────────────────────────
// Custom transform: uppercase converter
// ─────────────────────────────────────
const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    const upperCased = chunk.toString().toUpperCase();
    callback(null, upperCased);
  }
});

createReadStream('input.txt')
  .pipe(upperCaseTransform)
  .pipe(createWriteStream('output.txt'));
```

### Using pipeline (Recommended)

```javascript
import { pipeline } from 'stream/promises';
import { createReadStream, createWriteStream } from 'fs';
import { createGzip, createGunzip } from 'zlib';

// ─────────────────────────────────────
// Compress a file
// ─────────────────────────────────────
async function compressFile(input, output) {
  await pipeline(
    createReadStream(input),
    createGzip(),
    createWriteStream(output)
  );
  console.log('Compression complete');
}

// ─────────────────────────────────────
// Decompress a file
// ─────────────────────────────────────
async function decompressFile(input, output) {
  await pipeline(
    createReadStream(input),
    createGunzip(),
    createWriteStream(output)
  );
  console.log('Decompression complete');
}

await compressFile('data.txt', 'data.txt.gz');
await decompressFile('data.txt.gz', 'data-restored.txt');
```

### 🎯 Real-World Use Cases

| Use Case | Why Streams? |
|----------|--------------|
| **File uploads** | Process as data arrives |
| **Log parsing** | Handle GB-sized log files |
| **Video streaming** | Send chunks to client |
| **CSV processing** | Parse row by row |
| **API responses** | Stream large datasets |

### 💼 Interview Relevance

**Common Questions:**
- What's the advantage of streams over reading entire files?
- What are the different types of streams?
- How would you process a 10GB file in Node.js?
- What is backpressure in streams?

---

## 2️⃣ Module Caching

### How Module Caching Works

```javascript
// counter.js
let count = 0;

export function increment() {
  return ++count;
}

export function getCount() {
  return count;
}
```

```javascript
// app.js
import { increment, getCount } from './counter.js';
import { increment as inc2 } from './counter.js'; // Same module!

console.log(increment()); // 1
console.log(increment()); // 2
console.log(inc2());      // 3 (same counter!)
console.log(getCount());  // 3
```

**Key insight:** The module is executed only once. All imports share the same state.

### Cache Location

```javascript
// CommonJS: Check the cache
console.log(require.cache);
// {
//   '/path/to/module.js': Module { ... }
// }

// Clear cache (rarely needed)
delete require.cache[require.resolve('./module.js')];
```

### Circular Dependencies

```javascript
// ─────────────────────────────────────
// ⚠️ Circular Dependency Example
// ─────────────────────────────────────

// a.js
import { bValue } from './b.js';
export const aValue = 'A';
console.log('In a.js, bValue is:', bValue);

// b.js
import { aValue } from './a.js';
export const bValue = 'B';
console.log('In b.js, aValue is:', aValue);

// app.js
import './a.js';

// Output (ESM):
// In b.js, aValue is: undefined  ← Problem!
// In a.js, bValue is: B
```

### Avoiding Circular Dependencies

```javascript
// ─────────────────────────────────────
// Solution 1: Restructure your code
// ─────────────────────────────────────

// shared.js (new file with shared logic)
export const sharedValue = 'shared';

// a.js
import { sharedValue } from './shared.js';
export const aValue = sharedValue + '-A';

// b.js
import { sharedValue } from './shared.js';
export const bValue = sharedValue + '-B';


// ─────────────────────────────────────
// Solution 2: Lazy loading
// ─────────────────────────────────────

// a.js
export const aValue = 'A';

export function getB() {
  // Import when needed, not at module load
  const { bValue } = await import('./b.js');
  return bValue;
}
```

### 💼 Interview Relevance

**Common Questions:**
- How does module caching work in Node.js?
- What happens when you import the same module multiple times?
- How do you handle circular dependencies?

---

## 3️⃣ Advanced Async Patterns

### Retry Logic

```javascript
async function fetchWithRetry(fn, options = {}) {
  const {
    maxRetries = 3,
    delayMs = 1000,
    backoff = 2  // Exponential backoff multiplier
  } = options;

  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      console.log(`Attempt ${attempt} failed: ${error.message}`);
      
      if (attempt < maxRetries) {
        const waitTime = delayMs * Math.pow(backoff, attempt - 1);
        console.log(`Waiting ${waitTime}ms before retry...`);
        await new Promise(r => setTimeout(r, waitTime));
      }
    }
  }

  throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
}

// Usage
const result = await fetchWithRetry(
  () => fetch('https://api.example.com/data').then(r => r.json()),
  { maxRetries: 5, delayMs: 500 }
);
```

### Concurrency Control

```javascript
async function processWithConcurrency(items, fn, concurrency = 3) {
  const results = [];
  const executing = new Set();

  for (const item of items) {
    const promise = fn(item).then(result => {
      executing.delete(promise);
      return result;
    });
    
    results.push(promise);
    executing.add(promise);

    if (executing.size >= concurrency) {
      await Promise.race(executing);
    }
  }

  return Promise.all(results);
}

// Usage: Process 100 items, max 5 at a time
const urls = Array.from({ length: 100 }, (_, i) => `https://api.example.com/${i}`);

const results = await processWithConcurrency(
  urls,
  async (url) => {
    const response = await fetch(url);
    return response.json();
  },
  5  // Max 5 concurrent requests
);
```

### Simple Queue Implementation

```javascript
class AsyncQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.queue = [];
    this.running = 0;
    this.results = [];
  }

  add(fn) {
    return new Promise((resolve, reject) => {
      this.queue.push({ fn, resolve, reject });
      this.processNext();
    });
  }

  async processNext() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }

    this.running++;
    const { fn, resolve, reject } = this.queue.shift();

    try {
      const result = await fn();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.processNext();
    }
  }
}

// Usage
const queue = new AsyncQueue(2); // Max 2 concurrent

const tasks = [
  () => fetch('/api/1').then(r => r.json()),
  () => fetch('/api/2').then(r => r.json()),
  () => fetch('/api/3').then(r => r.json()),
  () => fetch('/api/4').then(r => r.json()),
];

const results = await Promise.all(tasks.map(task => queue.add(task)));
```

### Timeout Wrapper

```javascript
function withTimeout(promise, ms, message = 'Operation timed out') {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => reject(new Error(message)), ms);
  });
  
  return Promise.race([promise, timeout]);
}

// Usage
try {
  const result = await withTimeout(
    fetch('https://slow-api.example.com'),
    5000,
    'API request timed out after 5 seconds'
  );
} catch (error) {
  console.error(error.message);
}
```

### Debouncing Async Functions

```javascript
function debounceAsync(fn, delay) {
  let timeoutId;
  let pendingPromise = null;

  return async (...args) => {
    clearTimeout(timeoutId);
    
    return new Promise((resolve, reject) => {
      timeoutId = setTimeout(async () => {
        try {
          const result = await fn(...args);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      }, delay);
    });
  };
}

// Usage: Search API that waits for user to stop typing
const debouncedSearch = debounceAsync(async (query) => {
  const response = await fetch(`/search?q=${query}`);
  return response.json();
}, 300);
```

---

## 4️⃣ Production Configuration Patterns

### Complete Config System

```
📁 config/
├── index.js          (main entry point)
├── schema.js         (validation schema)
├── loader.js         (loads and validates)
└── environments/
    ├── base.js       (shared config)
    ├── development.js
    ├── production.js
    └── test.js
```

```javascript
// config/schema.js
export const schema = {
  NODE_ENV: {
    required: true,
    enum: ['development', 'production', 'test']
  },
  PORT: {
    required: true,
    type: 'number',
    default: 3000
  },
  DATABASE_URL: {
    required: true,
    type: 'string'
  },
  JWT_SECRET: {
    required: true,
    type: 'string',
    minLength: 32
  },
  CACHE_TTL: {
    required: false,
    type: 'number',
    default: 3600
  },
  ENABLE_METRICS: {
    required: false,
    type: 'boolean',
    default: false
  }
};
```

```javascript
// config/loader.js
import 'dotenv/config';

export function loadConfig(schema) {
  const config = {};
  const errors = [];

  for (const [key, rules] of Object.entries(schema)) {
    let value = process.env[key];

    // Apply default
    if (value === undefined && rules.default !== undefined) {
      value = String(rules.default);
    }

    // Check required
    if (rules.required && value === undefined) {
      errors.push(`Missing required env var: ${key}`);
      continue;
    }

    // Type conversion
    if (value !== undefined) {
      if (rules.type === 'number') {
        value = parseInt(value, 10);
        if (isNaN(value)) {
          errors.push(`${key} must be a number`);
          continue;
        }
      } else if (rules.type === 'boolean') {
        value = value === 'true' || value === '1';
      }
    }

    // Validation
    if (rules.enum && !rules.enum.includes(value)) {
      errors.push(`${key} must be one of: ${rules.enum.join(', ')}`);
    }
    if (rules.minLength && value?.length < rules.minLength) {
      errors.push(`${key} must be at least ${rules.minLength} characters`);
    }

    config[key] = value;
  }

  if (errors.length > 0) {
    console.error('❌ Configuration errors:');
    errors.forEach(e => console.error(`   - ${e}`));
    process.exit(1);
  }

  return config;
}
```

```javascript
// config/environments/base.js
export default {
  app: {
    name: 'MyApp',
    version: '1.0.0'
  },
  cors: {
    credentials: true
  },
  rateLimit: {
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100
  }
};
```

```javascript
// config/environments/development.js
export default {
  logging: {
    level: 'debug',
    prettyPrint: true
  },
  database: {
    logging: true
  }
};
```

```javascript
// config/environments/production.js
export default {
  logging: {
    level: 'warn',
    prettyPrint: false
  },
  database: {
    logging: false,
    ssl: true
  },
  security: {
    helmet: true,
    rateLimit: {
      max: 50
    }
  }
};
```

```javascript
// config/index.js
import { schema } from './schema.js';
import { loadConfig } from './loader.js';
import base from './environments/base.js';

const envConfig = loadConfig(schema);
const env = envConfig.NODE_ENV;

// Dynamic import based on environment
const { default: envSpecific } = await import(`./environments/${env}.js`);

// Deep merge configs
function deepMerge(target, source) {
  for (const key of Object.keys(source)) {
    if (source[key] instanceof Object && key in target) {
      deepMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

export const config = Object.freeze(
  deepMerge(
    deepMerge({ ...base }, envSpecific),
    {
      env,
      isProduction: env === 'production',
      isDevelopment: env === 'development',
      server: {
        port: envConfig.PORT
      },
      database: {
        url: envConfig.DATABASE_URL
      },
      auth: {
        jwtSecret: envConfig.JWT_SECRET
      },
      cache: {
        ttl: envConfig.CACHE_TTL
      }
    }
  )
);

console.log(`✅ Config loaded for: ${env}`);
```

### Secrets in Production

```javascript
// ─────────────────────────────────────
// Pattern: Load secrets from secure storage
// ─────────────────────────────────────

async function loadSecrets() {
  if (process.env.NODE_ENV === 'production') {
    // In production, load from AWS Secrets Manager, Vault, etc.
    // const client = new SecretsManagerClient({ region: 'us-east-1' });
    // const secret = await client.send(new GetSecretValueCommand({ SecretId: 'my-app/prod' }));
    // return JSON.parse(secret.SecretString);
  }
  
  // In development, use .env file
  return {
    JWT_SECRET: process.env.JWT_SECRET,
    DB_PASSWORD: process.env.DB_PASSWORD,
    API_KEY: process.env.API_KEY
  };
}

export const secrets = await loadSecrets();
```

---

## 5️⃣ Interview Preparation Summary

### Top Node.js Interview Questions

#### Modules
1. **What's the difference between CommonJS and ES Modules?**
   - CJS: `require()`/`module.exports`, synchronous, dynamic imports allowed
   - ESM: `import`/`export`, asynchronous, static analysis, top-level await

2. **How does module caching work?**
   - Modules are cached after first load
   - Subsequent imports return cached exports
   - Same instance shared across all imports

3. **How do you handle circular dependencies?**
   - Extract shared code into separate module
   - Use dynamic imports
   - Restructure to break the cycle

#### Async Code
4. **What's the difference between Promise.all and Promise.allSettled?**
   - `Promise.all`: Fails fast if any promise rejects
   - `Promise.allSettled`: Waits for all, returns status of each

5. **How do you handle errors in async/await?**
   - Use try/catch blocks
   - Create error handling middleware
   - Use .catch() for Promise chains

6. **What's the execution order of setTimeout, setImmediate, and process.nextTick?**
   - Sync code → nextTick → Promise microtasks → Timers → I/O → setImmediate

7. **What are streams and when would you use them?**
   - Process data in chunks
   - Use for large files, network requests
   - Types: Readable, Writable, Duplex, Transform

#### Environment Variables
8. **Why use environment variables instead of hardcoding?**
   - Different values per environment
   - Keep secrets out of code
   - Easy configuration changes without code changes

9. **How do you validate environment variables?**
   - Check at startup before app runs
   - Validate types and required fields
   - Provide clear error messages

10. **What security practices should you follow?**
    - Never commit .env files
    - Use different secrets per environment
    - Rotate secrets regularly
    - Use secret management in production

### Code Challenges You Might Face

```javascript
// 1. Implement Promise.all from scratch
function myPromiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}

// 2. Implement retry with exponential backoff
// (See section 3 above)

// 3. Read a large file and count word frequencies
async function countWords(filepath) {
  const { createReadStream } = await import('fs');
  const { createInterface } = await import('readline');
  
  const counts = new Map();
  
  const rl = createInterface({
    input: createReadStream(filepath),
    crlfDelay: Infinity
  });

  for await (const line of rl) {
    const words = line.toLowerCase().match(/\b\w+\b/g) || [];
    for (const word of words) {
      counts.set(word, (counts.get(word) || 0) + 1);
    }
  }

  return counts;
}
```

---

## 🧪 Practice Tasks

### Task 1: Large File Processor
**Process a large CSV file using streams**

```
📁 task-1/
├── generator.js     (generates test CSV)
├── processor.js     (stream-based processor)
└── data/
```

**Requirements:**
- Generate a 100MB+ CSV file (use streams)
- Process it line by line with streams
- Calculate statistics (count, sum, average per column)
- Write results to output.json
- Compare memory usage with non-stream version

---

### Task 2: API Client with Retry
**Build a robust HTTP client**

```
📁 task-2/
├── http-client.js
└── app.js
```

**Requirements:**
- Implement fetch with retry and backoff
- Add timeout support
- Concurrent request limiting (max 3 at a time)
- Request queuing
- Error classification (retryable vs permanent)

---

### Task 3: File Watcher Service
**Create a file watching service using events**

```
📁 task-3/
├── watcher.js       (EventEmitter-based)
├── handlers/
│   ├── logger.js
│   └── sync.js
└── app.js
```

**Requirements:**
- Watch a directory for changes
- Emit events: `fileAdded`, `fileChanged`, `fileRemoved`
- Logger handler: log all events
- Sync handler: copy changes to backup folder
- Handle errors gracefully

---

### Task 4: Task Queue System
**Build a job queue with persistence**

```
📁 task-4/
├── queue.js         (core queue logic)
├── worker.js        (processes jobs)
├── jobs/
│   └── pending.json
└── app.js
```

**Requirements:**
- Persist jobs to JSON file
- Configurable concurrency
- Job retry on failure (max 3 times)
- Job status tracking (pending, running, completed, failed)
- Graceful shutdown (finish running jobs)

---

### Task 5: Production-Ready Config
**Complete configuration system**

```
📁 task-5/
├── config/
│   ├── index.js
│   ├── schema.js
│   ├── loader.js
│   └── environments/
│       ├── base.js
│       ├── development.js
│       ├── production.js
│       └── test.js
├── .env
├── .env.example
├── .gitignore
└── app.js
```

**Requirements:**
- Schema-based validation
- Type conversion
- Environment-specific overrides
- Deep merging of configs
- Immutable config object
- Clear startup logging

---

## ✅ Phase 4 Checklist

After this phase, you should be able to:

- [ ] Use streams for processing large files
- [ ] Implement stream pipelines with transforms
- [ ] Understand and handle module caching
- [ ] Avoid and resolve circular dependencies
- [ ] Implement retry logic with backoff
- [ ] Control concurrency in async operations
- [ ] Build production-ready configuration
- [ ] Answer common Node.js interview questions

---

## 🎉 What's Next?

Congratulations! You've completed the study guide. Here's what to do next:

### Build a Real Project
Combine everything you've learned:
- **REST API** with Express or Fastify
- **Configuration** using patterns from Phase 4
- **Logging** with custom EventEmitter
- **File handling** with streams
- **Environment** management for dev/prod

### Suggested Project Ideas
1. **Task Manager API** - CRUD with file-based storage
2. **Log Aggregator** - Parse and analyze log files
3. **Config Server** - Centralized configuration service
4. **File Sync Service** - Watch and sync directories
5. **Job Queue** - Background task processing

### Keep Learning
- Express.js / Fastify for web servers
- Testing with Jest / Vitest
- Database integration (MongoDB, PostgreSQL)
- Authentication & Authorization
- Deployment (Docker, PM2)

---

*Good luck with your interviews and projects! 🚀*
