# Lab — Python to JavaScript Code Translation

## Before You Begin

### 1 — Clone the lab repository

Open a terminal, navigate to where you want to work, and run:

```bash
git clone https://github.com/viktor-demo/bob-lab-python-to-js.git bob-lab
```

This creates a new folder called `bob-lab` with all the lab files inside:

```
bob-lab/
├── data.csv
├── data_processor.py
└── README.md
```

### 2 — Open the folder in your IDE

Open the cloned folder as your workspace root:
- **VS Code:** **File → Open Folder** → select `bob-lab`
- **Alternative:** `code bob-lab` from the terminal (if the `code` CLI is installed)

### 3 — Open IBM Bob

The Bob panel should appear on the right side of your IDE. Confirm it shows the project files. You're ready to start.

---

## Overview

In this lab, you'll use **IBM Bob** to translate a Python data processing script to JavaScript (Node.js), maintaining full functionality while applying language-specific best practices.

You'll work across all three of Bob's modes — **Ask**, **Plan**, and **Agent** — and use Bob's real capabilities like `/init`, `/review`, `@` context mentions, and Move to Chat (`Cmd+L` / `Ctrl+L`) to see how Bob handles a realistic translation workflow end to end.

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

### 1.4 — Highlight a specific section with Move to Chat

Open `data_processor.py` in your editor. Highlight the `calculate_statistics` method (lines 66–115), then use the **Move to Chat** shortcut to send it directly to Bob:

- **macOS:** `Cmd` + `L`
- **Windows / Linux:** `Ctrl` + `L`
- **Alternative (all platforms):** right-click the selection → **Move to Chat**, or use the lightbulb (💡) menu

Then ask:

```
Explain exactly how this method works. What Python idioms are used here that have no direct JavaScript equivalent?
```

> **Bob feature — Move to Chat (`Cmd+L` / `Ctrl+L`):** Selects any code in the editor and sends it to the Bob chat panel with full context (file path + line numbers) — no copy-paste, no tab switching. You can also trigger it via the right-click context menu → **Move to Chat**, or via the lightbulb (💡) code actions menu that appears on selection.

---

## Step 2: Plan Translation Strategy with Plan Mode (10 minutes)

### 2.1 — Switch to Plan Mode

Select **Plan Mode** from the mode selector. Plan mode is read-only — Bob won't create or edit files here. It's designed for thinking through architecture and strategy before you write a single line of code.

> **Note:** In this step Bob responds in chat — no files are created. That's intentional. The output of Plan mode is understanding and alignment, not a document. You carry this analysis into Agent mode in Step 3.

### 2.2 — Create a feature-by-feature mapping

```
Analyse @data_processor.py and respond in chat with a translation plan for converting it to Node.js.
Do not create any files — written analysis only.

Include:
1. Feature-by-feature mapping (Python → JavaScript) for every major construct
2. npm packages required and why
3. Recommendation: CommonJS or ES Modules? async/await or streams?
4. Method name equivalents (Python snake_case → JavaScript camelCase)
5. Suggested file and project structure
```

Bob's response will cover all five areas in one shot — including the single npm dependency (`csv-parser`) and the CommonJS + async/await recommendation. Read through it before moving to Step 3.

**Key mappings to note:**

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

### 2.3 — Plan error handling equivalences (Optional)

```
Respond in chat only — no files. Map the Python exception handling in @data_processor.py
to its JavaScript equivalent.
How should FileNotFoundError, csv.Error, and IOError translate to Node.js patterns?
```

> **Optional:** Bob's answer to 2.2 already covers the key decisions. This prompt goes deeper on error handling specifically — useful if your team wants to understand how Python's typed exceptions (`FileNotFoundError`, `csv.Error`, `IOError`) map to Node.js patterns before implementing.

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

Type the following rough prompt into the chat input — but **don't send it yet**:

```
translate data_processor.py to javascript
```

> **Optional — Prompt Enhancement:** Before sending, click the ✨ sparkle button that appears in the chat input field. Bob will rewrite your rough prompt into a detailed, specific one — adding requirements like async/await, JSDoc, camelCase naming, error handling, and module format. Compare the before and after to see the difference. This feature must be enabled first: go to **Bob Settings → General → Enable prompt enhancement**.
>
> Whether you use enhancement or not, make sure the final prompt sent to Bob includes these requirements:

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

### 3.4 — Understand the file I/O translation

Highlight the `loadData()` method in `data_processor.js` and press **`Cmd+L`** (macOS) or **`Ctrl+L`** (Windows/Linux) to send it to Bob, then ask:

```
Why did you use createReadStream + csv-parser instead of fs.promises.readFile here?
What are the trade-offs between the two approaches for this use case?
```

This demonstrates that Bob can explain its own implementation decisions — useful when you need to justify or defend generated code to a colleague or reviewer.

> **Optional — Bonus prompts:** Steps 3.5, 3.6, and 3.7 are independent of each other. Run any or all of them depending on time. Each covers a distinct translation challenge and can be revisited after the lab.

### 3.5 — Understand the statistical calculations translation

```
Explain how you translated the calculate_statistics method.
How did you convert Python's list comprehensions and built-in functions to JavaScript?
```

### 3.6 — Understand the JSON export translation

```
Explain how you translated the export_results method.
What's the difference between Python's synchronous file writing and JavaScript's async approach?
```

### 3.7 — Understand the main execution logic

```
Explain how you translated Python's if __name__ == '__main__' pattern to JavaScript.
Why did you use an async IIFE (Immediately Invoked Function Expression)?
```

### 3.8 — Extend with Inline Chat (Optional)

Now that `data_processor.js` exists, try a surgical edit without ever leaving the file.

1. Open `data_processor.js` in your editor
2. Place your cursor inside the `calculateStatistics()` method, after the existing stats are built
3. Trigger **Inline Chat**:
   - **macOS:** `Cmd` + `K`
   - **Windows / Linux:** `Ctrl` + `K`
   - **Alternative (all platforms):** right-click in the editor → **Inline Chat**, or use the lightbulb (💡) menu
4. Type the following directly in the inline chat interface:

```
Add a median calculation to the stats for each numeric field.
Expected results for data.csv: age median = 26.5, score median = 90.4
```

Bob will show an inline diff with the new `median` field added to each stats object. Review the change in-place, then:
- **Accept:** `Cmd+Enter` (macOS) / `Ctrl+Enter` (Windows/Linux)
- **Reject:** close the inline chat without accepting

> **Bob feature — Inline Chat (`Cmd+K` / `Ctrl+K`):** Opens a chat interface directly in the editor at your cursor — no panel switching, no copy-paste. The result appears as an inline diff you accept or reject before it's written. Ideal for targeted, single-method edits when you don't need the full chat panel context.

### 3.9 — Run the JavaScript version

Install dependencies and run the script. In your terminal:

```bash
npm install
node data_processor.js
```

### 3.10 — Fix errors with `@terminal`

If you see any errors in the terminal output, stay in Agent mode and use `@terminal` instead of copy-pasting the error:

```
@terminal — fix the error shown above
```

> **Bob feature — `@terminal`:** Sends the most recent terminal output directly into the conversation as context. Bob reads the exact error, not your description of it. This is significantly faster than copy-pasting stack traces.

### 3.11 — Fix any IDE diagnostics with `@problems`

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
| `Cmd+L` / `Ctrl+L` (Move to Chat) | Steps 1, 3 | Sends highlighted code to Bob with file path + line numbers |
| ✨ Prompt Enhancement | Step 3.3 (optional) | Rewrites a rough prompt into a detailed one before sending |
| `Cmd+K` / `Ctrl+K` (Inline Chat) | Step 3.8 (optional) | Chat interface directly in the editor — inline diff, no panel switching |
| `@terminal` | Step 3.10 | Sends recent terminal output to Bob for debugging |
| `@problems` | Step 3.11 | Sends IDE diagnostics to Bob for bulk fixing |
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
