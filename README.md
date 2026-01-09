# 📘 Phase 1: Foundations

> **Goal:** Understand how Node.js organizes code and handles basic async operations  
> **Duration:** 2-3 days  
> **Prerequisites:** Basic JavaScript knowledge

---

## 📚 What You'll Learn

| Topic | Concepts |
|-------|----------|
| CommonJS Modules | `require()`, `module.exports`, `exports` |
| Built-in Modules | `fs`, `path`, `os` |
| Callbacks | Error-first pattern, async basics |
| Environment Variables | `process.env` basics |

---

## 1️⃣ CommonJS Modules (CJS)

### What is it?
CommonJS is Node.js's original module system. It lets you split your code into separate files and share functionality between them.

### The Core Concept

```javascript
// ─────────────────────────────────────
// math.js - Exporting functionality
// ─────────────────────────────────────
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

// Export functions for other files to use
module.exports = {
  add,
  subtract
};
```

```javascript
// ─────────────────────────────────────
// app.js - Importing functionality
// ─────────────────────────────────────
const math = require('./math');  // .js extension is optional

console.log(math.add(5, 3));      // 8
console.log(math.subtract(10, 4)); // 6
```

### Different Export Styles

```javascript
// Style 1: Export an object
module.exports = { add, subtract };

// Style 2: Export a single function
module.exports = function greet(name) {
  return `Hello, ${name}!`;
};

// Style 3: Add properties to exports
exports.add = add;
exports.subtract = subtract;
// ⚠️ Don't do: exports = { add } - This breaks the reference!
```

### 🎯 Why This Matters in Real Projects

| Scenario | How Modules Help |
|----------|------------------|
| **Express APIs** | Separate routes into different files |
| **Database Logic** | Keep database queries isolated |
| **Utilities** | Create reusable helper functions |
| **Config** | Manage configuration in dedicated files |

### 💼 Interview Relevance

**Common Questions:**
- What's the difference between `module.exports` and `exports`?
- How does Node.js resolve module paths?
- What happens when you require the same module twice?

**Expected Answer:** `exports` is a reference to `module.exports`. If you reassign `exports`, you break this reference. Always use `module.exports` for clarity.

---

## 2️⃣ Built-in Modules

Node.js comes with powerful built-in modules. No npm install needed!

### `fs` - File System

```javascript
const fs = require('fs');

// ─────────────────────────────────────
// SYNC (Blocking) - Avoid in production
// ─────────────────────────────────────
const data = fs.readFileSync('config.txt', 'utf8');
console.log(data);

// ─────────────────────────────────────
// ASYNC (Non-blocking) - Preferred!
// ─────────────────────────────────────
fs.readFile('config.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Failed to read file:', err.message);
    return;
  }
  console.log(data);
});

// ─────────────────────────────────────
// Writing files
// ─────────────────────────────────────
fs.writeFile('output.txt', 'Hello World', (err) => {
  if (err) throw err;
  console.log('File saved!');
});
```

### `path` - Path Manipulation

```javascript
const path = require('path');

// Join path segments (handles OS differences)
const filePath = path.join(__dirname, 'data', 'users.json');
// Windows: C:\project\data\users.json
// Linux:   /project/data/users.json

// Get file information
console.log(path.basename('/users/config.json'));  // 'config.json'
console.log(path.extname('report.pdf'));           // '.pdf'
console.log(path.dirname('/users/data/file.txt')); // '/users/data'

// Resolve to absolute path
console.log(path.resolve('src', 'index.js'));
```

### `os` - Operating System Info

```javascript
const os = require('os');

console.log(os.platform());   // 'win32', 'linux', 'darwin'
console.log(os.cpus().length); // Number of CPU cores
console.log(os.totalmem());    // Total system memory
console.log(os.homedir());     // User's home directory
console.log(os.tmpdir());      // Temp directory path
```

### 🎯 Why This Matters in Real Projects

| Module | Real Use Case |
|--------|---------------|
| `fs` | Reading config files, logging, file uploads |
| `path` | Safe file path construction, avoiding hardcoded paths |
| `os` | Health checks, system-aware configurations |

### 💼 Interview Relevance

**Common Questions:**
- Why should we avoid sync file operations?
- What's the difference between `__dirname` and `process.cwd()`?
- How do you handle file paths across different operating systems?

---

## 3️⃣ Callbacks & Error-First Pattern

### What is a Callback?
A callback is a function passed as an argument to another function, to be executed later.

### The Error-First Convention

Node.js established a convention: **the first argument of a callback is always the error**.

```javascript
const fs = require('fs');

fs.readFile('data.txt', 'utf8', (err, data) => {
  // err is the FIRST argument (null if no error)
  // data is the SECOND argument (result if success)
  
  if (err) {
    console.error('Error:', err.message);
    return; // Stop execution here!
  }
  
  console.log('File content:', data);
});
```

### Why Error-First?

```javascript
// ✅ Consistent pattern across ALL Node.js async operations
fs.readFile('file.txt', (err, data) => { /* ... */ });
fs.writeFile('file.txt', content, (err) => { /* ... */ });
fs.mkdir('folder', (err) => { /* ... */ });

// You always know: check err first, then use result
```

### Creating Your Own Async Functions

```javascript
function fetchUserData(userId, callback) {
  // Simulate database call
  setTimeout(() => {
    if (!userId) {
      // Error-first: pass error as first argument
      callback(new Error('User ID is required'), null);
      return;
    }
    
    const user = { id: userId, name: 'John Doe' };
    // Success: pass null as error, data as second argument
    callback(null, user);
  }, 1000);
}

// Usage
fetchUserData(123, (err, user) => {
  if (err) {
    console.error('Failed:', err.message);
    return;
  }
  console.log('User:', user);
});
```

### ⚠️ Callback Hell (What to Avoid)

```javascript
// 😱 This is hard to read and maintain
fs.readFile('file1.txt', (err, data1) => {
  if (err) return console.error(err);
  fs.readFile('file2.txt', (err, data2) => {
    if (err) return console.error(err);
    fs.readFile('file3.txt', (err, data3) => {
      if (err) return console.error(err);
      // Deep nesting = callback hell!
    });
  });
});
```

### 💼 Interview Relevance

**Common Questions:**
- What is callback hell and how do you avoid it?
- Why does Node.js use error-first callbacks?
- What's the problem with returning inside a callback?

**Expected Answer:** Callback hell can be avoided using Promises or async/await (Phase 2). Error-first is a convention that ensures consistent error handling across the ecosystem.

---

## 4️⃣ Environment Variables Basics

### What are Environment Variables?
Configuration values set **outside** your code, passed to your application at runtime.

### Accessing via `process.env`

```javascript
// process.env is a global object containing all env variables
console.log(process.env.NODE_ENV);  // 'development' or 'production'
console.log(process.env.PORT);       // '3000' (always a string!)
console.log(process.env.PATH);       // System PATH variable
```

### Setting Environment Variables

```bash
# Windows (PowerShell)
$env:PORT="3000"; node app.js

# Windows (CMD)
set PORT=3000 && node app.js

# Linux/Mac
PORT=3000 node app.js

# Or inline (Linux/Mac)
NODE_ENV=production node app.js
```

### Basic Usage Pattern

```javascript
// app.js
const port = process.env.PORT || 3000;  // Default if not set
const env = process.env.NODE_ENV || 'development';

console.log(`Running on port ${port} in ${env} mode`);
```

### ⚠️ Important: Env Vars are Always Strings!

```javascript
// 🚫 Common mistake
process.env.PORT = 3000;
console.log(typeof process.env.PORT); // 'string', not 'number'!

// ✅ Convert when needed
const port = parseInt(process.env.PORT, 10) || 3000;
const isDebug = process.env.DEBUG === 'true';  // String comparison
```

### 🎯 Why This Matters in Real Projects

| Use Case | Example |
|----------|---------|
| **Different Environments** | Dev vs Production database URLs |
| **Secrets** | API keys, database passwords |
| **Feature Flags** | Enable/disable features |
| **Configuration** | Port numbers, timeout values |

### 💼 Interview Relevance

**Common Questions:**
- Why use environment variables instead of hardcoding?
- How do you handle different configs for dev vs production?
- What's the security benefit of environment variables?

---

## 🧪 Practice Tasks

### Task 1: Module Organization
**Create a simple calculator module**

```
📁 task-1/
├── calculator.js   (export add, subtract, multiply, divide)
└── app.js          (import and use all operations)
```

**Requirements:**
- Export all 4 operations from calculator.js
- Handle division by zero (return 'Error: Division by zero')
- Use the calculator in app.js to perform various calculations

---

### Task 2: File Operations
**Create a simple note-taking app (CLI)**

```
📁 task-2/
└── notes.js
```

**Requirements:**
- Use `fs` to save notes to `notes.txt`
- Accept action via command line: `node notes.js add "My note"`
- Actions: `add`, `list`, `clear`
- Use error-first callbacks

**Example usage:**
```bash
node notes.js add "Buy groceries"
node notes.js add "Call mom"
node notes.js list
node notes.js clear
```

---

### Task 3: System Info Reporter
**Create a system info module**

```
📁 task-3/
├── sysinfo.js     (module with functions)
└── report.js      (generates report)
```

**Requirements:**
- Create functions: `getPlatform()`, `getCpuCount()`, `getMemoryGB()`, `getTempPath()`
- Export all functions from sysinfo.js
- In report.js, generate a formatted system report
- Save the report to `system-report.txt` using fs

---

### Task 4: Environment-Aware App
**Create an app that behaves differently based on environment**

```
📁 task-4/
├── config.js      (read env variables)
└── app.js         (use config)
```

**Requirements:**
- Read `NODE_ENV`, `PORT`, and `APP_NAME` from environment
- Provide default values if not set
- Log different messages based on NODE_ENV
- Test by running with different env variables

**Example:**
```bash
# PowerShell
$env:NODE_ENV="production"; $env:PORT="8080"; node app.js
```

---

### Task 5: Combined Challenge
**Build a "Daily Logger" application**

```
📁 task-5/
├── logger.js      (module for logging)
├── fileManager.js (module for file operations)
├── app.js         (main application)
└── logs/          (directory for log files)
```

**Requirements:**
- `logger.js`: Export functions `logInfo()`, `logError()`, `logWarning()`
- `fileManager.js`: Export functions to read/write/append to files
- Log format: `[TIMESTAMP] [LEVEL] Message`
- Create daily log files: `log-2024-01-15.txt`
- Use `path.join()` for file paths
- Read LOG_LEVEL from environment (default: 'info')

---

## ✅ Phase 1 Checklist

Before moving to Phase 2, make sure you can:

- [ ] Create and export modules using CommonJS
- [ ] Import modules using `require()`
- [ ] Read/write files using `fs` with callbacks
- [ ] Construct file paths safely with `path.join()`
- [ ] Understand error-first callback pattern
- [ ] Access environment variables via `process.env`
- [ ] Provide default values for env variables

---

*Next: [Phase 2: Modern Patterns →](../phase-2/README.md)*
