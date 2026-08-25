# Lab — Python to JavaScript Code Translation

## Overview

In this lab, you'll use **IBM Bob** to translate a Python data processing script to JavaScript (Node.js), maintaining full functionality while applying language-specific best practices.

You'll work across all three of Bob's modes — **Ask**, **Plan**, and **Agent** — and use Bob's real capabilities like `/init`, `/review`, `@` context mentions, and `⌘+L` inline selection to see how Bob handles a realistic translation workflow end to end.

> **Bob Differentiator — Intelligent Model Selection**
> Bob automatically routes each task to the most appropriate underlying model. Complex reasoning tasks (like mapping Python context managers to Node.js async streams) get a frontier model; simpler syntax questions use a lighter one. This happens transparently behind every prompt.

**Duration:** 45 minutes  
**Difficulty:** Intermediate

---

## What You'll Translate

A Python data processing script ([`data_processor.py`](./data_processor.py)) that:

- Reads CSV files using `csv.DictReader`
- Performs statistical calculations (mean, min, max, sum, count)
- Exports results to JSON
- Uses type hints, context managers, and list comprehensions

**Target:** Equivalent JavaScript (Node.js) implementation

---

## Learning Objectives

By the end of this lab you will:

- ✅ Use `/init` to give Bob persistent project context
- ✅ Use **Ask mode** to analyse source code with `@` file mentions
- ✅ Use **Plan mode** to design a translation strategy
- ✅ Use **Agent mode** to implement the translation
- ✅ Use `/review` to validate generated code quality
- ✅ Use `@terminal` and `@problems` to debug without switching context
- ✅ Map Python-specific features to JavaScript equivalents
- ✅ Handle the sync → async shift between Python and Node.js

---

## Prerequisites

Before starting, ensure you have:

- [ ] Python 3.8+ installed
- [ ] Node.js 14+ installed
- [ ] IBM Bob installed and running in your IDE
- [ ] Basic familiarity with both Python and JavaScript

---

## Lab Structure

```
Lab Timeline (45 minutes)
├── Setup:  Initialize Bob with /init           (3 min)
├── Step 1: Analyse Python Code — Ask Mode     (10 min)
├── Step 2: Plan Translation Strategy — Plan Mode (10 min)
├── Step 3: Implement the Translation — Agent Mode (17 min)
└── Step 4: Verify & Review                    (5 min)
```

---

## Setup: Initialize Bob with `/init` (3 minutes)

Before you start, give Bob persistent knowledge of this project so it doesn't have to re-discover files in every conversation.

### Run `/init`

In the Bob chat input, type:

```
/init
```

Bob will scan the workspace, then generate two things:

1. **`AGENTS.md`** — a root-level summary of the project structure, files, and conventions
2. **`.bob/` folder** — mode-specific context files:
   - `.bob/rules-ask/AGENTS-ask.md` — context for Ask mode
   - `.bob/rules-plan/AGENTS-plan.md` — context for Plan mode
   - `.bob/rules-agent/AGENTS-agent.md` — context for Agent mode

From this point forward, Bob reads these files at the start of every conversation. You won't have to re-explain what `data_processor.py` does each time you open a new chat.

> **Why this matters:** LLMs are stateless — each chat starts fresh. `/init` is how you give Bob a project memory that persists across sessions and modes.

---

## Step 1: Analyse Python Code with Ask Mode (10 minutes)

### 1.1 — Switch to Ask Mode

Select **Ask Mode** from the mode selector in the bottom-left of the chat panel.

### 1.2 — Reference the file with an `@` mention

Instead of copying and pasting code, use Bob's context mentions to reference the file directly:

```
Analyse @data_processor.py and explain:
1. What is the overall purpose of this script?
2. What are the main class methods and what does each do?
3. Which Python-specific features are used (type hints, context managers, comprehensions)?
4. What are the key data structures?
```

> **Bob feature — `@` context mentions:** Typing `@` in the chat input opens a dropdown of your project files. Select `data_processor.py` to pin its contents into the conversation. No copy-paste, no file paths to memorise.

**Expected outcome:** Bob explains the `DataProcessor` class, its five public methods, and highlights Python patterns like `with open()` context managers and list comprehensions.

### 1.3 — Identify translation challenges

```
What challenges might we face translating @data_processor.py to Node.js?
Focus on: file I/O, CSV parsing, type hints, list comprehensions, and sync vs async.
```

**Expected challenges Bob will surface:**

| Python pattern | Translation challenge |
|---|---|
| `with open()` context manager | Node.js uses async streams or `fs.promises` |
| `csv.DictReader` | No built-in equivalent — needs `csv-parser` npm package |
| Type hints | No runtime equivalent — use JSDoc |
| List comprehensions | Replace with `Array.map()` / `Array.filter()` |
| Sync file I/O | Node.js I/O is async by default |

### 1.4 — Highlight a specific section with ⌘+L

Open `data_processor.py` in your editor. Highlight the `calculate_statistics` method (lines 66–115), then press **`⌘+L`** to add the selection directly to the Bob chat. Then ask:

```
Explain exactly how this method works. What Python idioms are used here that have no direct JavaScript equivalent?
```

> **Bob feature — `⌘+L` inline selection:** Highlights any text in the editor and sends it to Bob as a focused context snippet — faster than switching tabs or copy-pasting.

---

## Step 2: Plan Translation Strategy with Plan Mode (10 minutes)

### 2.1 — Switch to Plan Mode

Select **Plan Mode** from the mode selector. Plan mode is read-only — Bob won't create or edit files here. It's designed for thinking through architecture and strategy before you write a single line of code.

### 2.2 — Create a feature-by-feature mapping

```
Create a detailed translation plan for converting @data_processor.py to Node.js.
Include:
1. Feature-by-feature mapping (Python → JavaScript) for every major construct
2. npm packages required and why
3. Recommendation: CommonJS or ES Modules? async/await or streams?
4. Method name equivalents (Python snake_case → JavaScript camelCase)
5. Suggested file and project structure
```

**Expected mapping Bob will produce:**

| Python | JavaScript (Node.js) | Notes |
|---|---|---|
| `class DataProcessor` | `class DataProcessor` | Syntax is similar |
| `def __init__(self, filename: str)` | `constructor(filename)` | No `self`, no type hints |
| `with open(file)` | `createReadStream()` + `csv-parser` | Must be async |
| `csv.DictReader` | `csv-parser` (npm) | Stream-based |
| `[x for x in list if cond]` | `.filter(...).map(...)` | More verbose |
| Type hints | JSDoc `@param` / `@returns` | Recommended |
| `if __name__ == '__main__'` | `if (require.main === module)` | CommonJS guard |
| `_is_numeric(value)` | `_isNumeric(value)` | camelCase convention |

### 2.3 — Confirm the dependency list

```
What is the minimal set of npm packages needed for this translation?
Explain why each is required and whether any could be replaced with Node.js built-ins.
```

You should land on a single external dependency: `csv-parser`.

### 2.4 — Plan error handling equivalences

```
Map the Python exception handling in data_processor.py to its JavaScript equivalent.
How should FileNotFoundError, csv.Error, and IOError translate to Node.js patterns?
```

> **Why Plan mode matters here:** By thinking through the full mapping before writing code, you avoid mid-implementation surprises — like discovering midway that Python's sync CSV reading needs a completely different async pattern in Node.js.

---

## Step 3: Implement the Translation with Agent Mode (17 minutes)

### 3.1 — Switch to Agent Mode

Select **Agent Mode**. This is where Bob reads, writes, and executes. You'll approve each action Bob proposes before it takes effect.

> **Optional — Auto-approve Read actions:** For this lab, you can hover over the auto-approve toolbar above the chat input and enable **Read** (medium risk). This lets Bob read files without prompting you each time. Leave **Edit** and **Execute** on manual approval so you review each file write and terminal command.

### 3.2 — Create `package.json`

```
Create a package.json for a Node.js project called "data-processor" version 1.0.0.
Single dependency: csv-parser.
Main entry point: data_processor.js.
Add a "start" script: node data_processor.js
```

Review and approve the file write when Bob proposes it.

### 3.3 — Translate the full class

```
Using the translation plan we designed, translate the entire DataProcessor class
from @data_processor.py to JavaScript (Node.js).

Requirements:
- CommonJS modules (require / module.exports)
- async/await for all file operations
- JSDoc comments matching the Python docstrings
- camelCase method names (e.g. loadData, calculateStatistics, exportResults)
- _isNumeric() private helper matching Python's _is_numeric()
- Identical statistical output (mean, min, max, count, sum)
- Same error behaviour: throw on file not found, etc.
- Main execution block with async IIFE at the bottom
```

Bob will create `data_processor.js`. Review the proposed file before approving.

### 3.4 — Understand a specific translation decision

Highlight the `loadData()` method in the newly created `data_processor.js`, press **`⌘+L`**, and ask:

```
Why did you use createReadStream + csv-parser instead of fs.promises.readFile here?
What are the trade-offs between the two approaches for this use case?
```

This demonstrates that Bob can explain its own implementation decisions — useful for developers who need to justify or defend the generated code.

### 3.5 — Run the JavaScript version

Install dependencies and run the script. In your terminal:

```bash
npm install
node data_processor.js
```

### 3.6 — Fix errors with `@terminal`

If you see any errors in the terminal output, stay in Agent mode and use `@terminal` instead of copy-pasting the error:

```
@terminal — fix the error shown above
```

> **Bob feature — `@terminal`:** Sends the most recent terminal output directly into the conversation as context. Bob reads the exact error, not your description of it. This is significantly faster than copy-pasting stack traces.

### 3.7 — Fix any IDE diagnostics with `@problems`

If your IDE shows red underlines in `data_processor.js`, use:

```
@problems fix all errors in data_processor.js
```

> **Bob feature — `@problems`:** Pulls in the full IBM Bob Problems panel diagnostics — every error and warning Bob's analysis has flagged — and fixes them in one shot without you having to enumerate them.

---

## Step 4: Verify & Review (5 minutes)

### 4.1 — Run the Python version to get the reference output

```bash
python data_processor.py
```

Expected console output:

```
🚀 Data Processor - Python Version
==================================================
✅ Loaded 4 rows from sample_data.csv
📊 Found 2 numeric fields: age, score

STATISTICS SUMMARY
==================================================
age:
  Mean:  26.25  Min: 22.00  Max: 30.00  Count: 4  Sum: 105.00
score:
  Mean:  90.90  Min: 87.30  Max: 95.50  Count: 4  Sum: 363.60

✅ Results exported to statistics.json
✅ Processing complete!
```

### 4.2 — Run the JavaScript version

```bash
node data_processor.js
```

The statistical output must be numerically identical.

### 4.3 — Run `/review` on the translated file

Switch to **Ask Mode** and run:

```
/review data_processor.js
```

`/review` performs a thorough static analysis of the file including:

- **Bug detection** — off-by-one errors, unhandled promise rejections
- **Security checks** — unsafe file path handling, injection risks
- **Performance issues** — blocking operations inside async contexts
- **Style consistency** — naming conventions, JSDoc completeness

Address any issues Bob flags before wrapping up.

### 4.4 — Compare both implementations side by side

Use `@` mentions to pin both files at once:

```
Compare @data_processor.py and @data_processor.js side by side.
For each method, confirm: is the JavaScript version functionally equivalent to Python?
Are there any edge cases the Python version handles that the JavaScript version does not?
```

### 4.5 — Verify functional equivalence

Both versions should produce:

- ✅ Identical numeric statistics (mean, min, max, count, sum)
- ✅ Same `statistics.json` structure
- ✅ Same error messages on missing files
- ✅ Equivalent console output format

---

## Bob Features Used in This Lab — Summary

| Feature | Where used | What it does |
|---|---|---|
| `/init` | Setup | Scans project, writes `AGENTS.md` — gives Bob persistent memory across sessions |
| **Ask mode** | Step 1 | Read-only analysis and explanation — no file changes |
| **Plan mode** | Step 2 | Architecture and strategy design — no file changes |
| **Agent mode** | Step 3 | Reads, writes, and executes — full implementation |
| `@filename` mention | Steps 1, 3, 4 | Pins a file's contents into the conversation as context |
| `⌘+L` selection | Steps 1, 3 | Sends a highlighted code selection directly to Bob |
| `@terminal` | Step 3.6 | Sends recent terminal output to Bob for debugging |
| `@problems` | Step 3.7 | Sends IDE diagnostics to Bob for bulk fixing |
| `/review` | Step 4.3 | Runs a full code review: bugs, security, performance, style |

---

## Translation Patterns Reference

### 1 — Class and constructor

```python
# Python
class DataProcessor:
    def __init__(self, filename: str) -> None:
        self.filename = filename
```

```js
// JavaScript
class DataProcessor {
    constructor(filename) {
        this.filename = filename;
    }
}
```

### 2 — List comprehensions → Array methods

```python
# Python
values = [float(row[field]) for row in self.data if self._is_numeric(row[field])]
```

```js
// JavaScript
const values = this.data
    .filter(row => this._isNumeric(row[field]))
    .map(row => parseFloat(row[field]));
```

### 3 — File I/O

```python
# Python — synchronous, automatic close via context manager
with open(self.filename, 'r', encoding='utf-8') as file:
    reader = csv.DictReader(file)
    self.data = [row for row in reader]
```

```js
// JavaScript — asynchronous, stream-based
async loadData() {
    return new Promise((resolve, reject) => {
        const results = [];
        createReadStream(this.filename)
            .pipe(csv())
            .on('data', row => results.push(row))
            .on('end', () => { this.data = results; resolve(); })
            .on('error', reject);
    });
}
```

### 4 — Type hints → JSDoc

```python
# Python
def calculate_statistics(self) -> Dict[str, Dict[str, float]]:
```

```js
// JavaScript
/**
 * @returns {Object.<string, {mean: number, min: number, max: number, count: number, sum: number}>}
 */
calculateStatistics() {
```

### 5 — Main execution guard

```python
# Python
if __name__ == '__main__':
    main()
```

```js
// JavaScript
if (require.main === module) {
    (async () => {
        try {
            const processor = new DataProcessor('data.csv');
            await processor.loadData();
            processor.calculateStatistics();
            await processor.exportResults('statistics.json');
        } catch (error) {
            console.error('❌ Error:', error.message);
            process.exit(1);
        }
    })();
}
```

---

## Language Comparison

| Feature | Python | JavaScript (Node.js) |
|---|---|---|
| Typing | Optional type hints | JSDoc or TypeScript |
| Async model | Sync by default | Async by default |
| File I/O | Built-in `open()`, sync | `fs` module, async |
| CSV | Built-in `csv` module | `csv-parser` (npm) |
| Array processing | List comprehensions | `.map()`, `.filter()`, `.reduce()` |
| Classes | `class` + `self` | `class` + `this` (ES6+) |
| Modules | `import` / `from` | `require` / `module.exports` |
| Entry point guard | `if __name__ == '__main__'` | `if (require.main === module)` |

---

## Troubleshooting

**`ModuleNotFoundError: No module named 'csv'`**  
`csv` is a Python built-in. Verify your Python version: `python --version` (must be 3.x).

**`Cannot find module 'csv-parser'`**  
Run `npm install` before executing the script.

**`async/await` syntax error in Node.js**  
Requires Node.js 14 or later: `node --version`.

**File not found errors**  
Use `path.join(__dirname, 'data.csv')` to resolve paths relative to the script file.

```js
const path = require('path');
const filePath = path.join(__dirname, 'data.csv');
```

---

## Next Steps

- Translate a REST client: Python `requests` → JavaScript `axios`
- Translate a CLI tool: Python `argparse` → JavaScript `commander`
- Add TypeScript types to the JavaScript output for full type safety
- Re-run `/init` after adding the JavaScript file — watch how `AGENTS.md` updates to include both languages

---

*Made with IBM Bob*
