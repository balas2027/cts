# Secure Code Analyzer - CLI Commands & Flow Guide

## Overview

Complete command reference for the Secure Code Analyzer CLI with analysis modes, report generation, and workflow diagrams.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Command Reference](#command-reference)
3. [Analysis Modes](#analysis-modes)
4. [Report Formats](#report-formats)
5. [Workflow Flows](#workflow-flows)
6. [Examples](#examples)

---

## Quick Start

### Basic Installation

```bash
pip install -r requirements.txt
```

### Run CLI

```bash
python cli_analyzer.py
```

---

## Command Reference

### 1. Interactive Mode (Default)

Run without arguments to enter interactive mode:

```bash
python cli_analyzer.py
```

**Flow:**

- Select Analysis Mode (Taint/Regex/Hybrid)
- Select Action (File/Directory/GitHub)
- Enter file/directory path or GitHub URL
- View results in terminal
- (Optional) Export report

---

### 2. Analyze Single File

#### With All Options

```bash
python cli_analyzer.py \
  -f <file_path> \
  -m <mode> \
  -o <output_file> \
  --format <format>
```

#### Minimal

```bash
python cli_analyzer.py -f path/to/file.js
```

**Parameters:**
| Parameter | Short | Long | Values | Default | Description |
|-----------|-------|------|--------|---------|-------------|
| File | `-f` | `--file` | `<path>` | - | File to analyze (JS/PHP/TS) |
| Mode | `-m` | `--mode` | `taint`, `regex`, `hybrid` | `taint` | Analysis model |
| Output | `-o` | `--output` | `<path>` | - | Output report path |
| Format | - | `--format` | `text`, `json` | `text` | Output format |

**Examples:**

```bash
# JavaScript with default mode
python cli_analyzer.py -f vulnerable.js

# PHP with taint analysis and HTML report
python cli_analyzer.py -f vulnerable.php -m taint -o report.html

# TypeScript with regex mode and JSON report
python cli_analyzer.py -f app.ts -m regex -o report.json
```

---

### 3. Analyze Directory

#### Command

```bash
python cli_analyzer.py \
  -d <directory_path> \
  -m <mode> \
  -o <output_file>
```

#### Minimal

```bash
python cli_analyzer.py -d ./src
```

**Examples:**

```bash
# Scan entire src directory (default: taint mode)
python cli_analyzer.py -d ./src

# Scan with hybrid mode and generate report
python cli_analyzer.py -d ./app -m hybrid -o security_report.html

# Scan and export as JSON
python cli_analyzer.py -d ./backend -m taint -o results.json --format json
```

---

### 4. Analyze GitHub Repository

#### Command

```bash
python cli_analyzer.py \
  -g <github_url> \
  -m <mode> \
  -o <output_file>
```

#### Minimal

```bash
python cli_analyzer.py -g https://github.com/user/repo
```

**Examples:**

```bash
# Analyze public repository
python cli_analyzer.py -g https://github.com/example/vulnerable-app

# Analyze with specific mode and generate report
python cli_analyzer.py -g https://github.com/user/project -m hybrid -o github_audit.html
```

---

## Analysis Modes

### 1. **Taint Mode** (Recommended - Default)

**Command:** `-m taint`

**Description:** Data flow analysis tracking untrusted data sources through the application.

**Use Cases:**

- Complex data flow vulnerabilities
- Indirect injection attacks
- Information disclosure via data paths

**Pros:**

- ✅ Detects complex attack vectors
- ✅ Tracks data flow across functions
- ✅ Lower false positives

**Cons:**

- ⏱️ Slower analysis
- 🔍 More detailed output

**Detection Examples:**

- SQL injection via POST parameters
- XSS through database-stored values
- Command injection via chained operations

---

### 2. **Regex Mode**

**Command:** `-m regex`

**Description:** Traditional pattern matching using OWASP TOP 10 rules.

**Use Cases:**

- Quick scanning
- Known vulnerability patterns
- Legacy code analysis

**Pros:**

- ✅ Fast analysis
- ✅ Quick results
- ✅ Lightweight

**Cons:**

- ⚠️ May miss complex flows
- ⚠️ More false positives possible

**Detection Examples:**

- SQL keywords without sanitization
- Script tags in output
- Weak cryptography (MD5, SHA1)
- Hardcoded secrets and credentials

---

### 3. **Hybrid Mode**

**Command:** `-m hybrid`

**Description:** Combines both Taint and Regex analysis for comprehensive coverage.

**Use Cases:**

- Comprehensive security audits
- Complete vulnerability detection
- Industry compliance requirements

**Pros:**

- ✅ Catches all vulnerability types
- ✅ Combines best of both methods
- ✅ Most thorough

**Cons:**

- ⏱️ Slowest analysis
- 📊 Most output to review

---

## Mode Comparison Matrix

| Feature             | Taint        | Regex        | Hybrid       |
| ------------------- | ------------ | ------------ | ------------ |
| **Speed**           | Medium       | Fast         | Slow         |
| **False Positives** | Low          | Medium       | Low-Medium   |
| **Complex Flows**   | ✅ Excellent | ❌ Limited   | ✅ Excellent |
| **Simple Patterns** | ✅ Good      | ✅ Excellent | ✅ Excellent |
| **CPU Usage**       | Medium       | Low          | High         |
| **Memory Usage**    | Medium       | Low          | High         |
| **Best For**        | Production   | Quick Scans  | Audits       |

---

## Report Formats

### 1. Text/HTML Report (Interactive Terminal)

**Trigger:** Default when no `--format json` specified

```bash
python cli_analyzer.py -f app.js -o report.html
```

**Output Includes:**

- Severity breakdown (Critical/High/Medium/Low)
- Vulnerability categories
- Security score (0-100)
- Detailed findings table
- Interactive formatting

---

### 2. HTML Report

**File Extension:** `.html`

```bash
python cli_analyzer.py -f app.js -o security_report.html
```

**Features:**

- 📊 Formatted tables
- 📈 Charts and statistics
- 🎨 Color-coded severity
- 📱 Responsive design

---

### 3. JSON Report

**File Extension:** `.json`

```bash
python cli_analyzer.py -f app.js -o report.json

# Or use JSON format output
python cli_analyzer.py -f app.js --format json > report.json
```

**Use Cases:**

- CI/CD pipeline integration
- Automated processing
- API integration
- Tool chaining

---

### 4. PDF Report

**File Extension:** `.pdf`

```bash
python cli_analyzer.py -f app.js -o full_audit.pdf
```

**Features:**

- 📄 Professional formatting
- 📋 Executive summary
- 🔍 Detailed findings
- ✍️ Signature space for compliance

---

### 5. Text Report

**File Extension:** `.txt`

```bash
python cli_analyzer.py -f app.js -o findings.txt
```

**Use Cases:**

- Simple text documentation
- Email reports
- Quick reference

---

## Workflow Flows

### 1. Complete Analysis Workflow

```
┌─────────────────────────────────────────────────────────────┐
│         START SECURE CODE ANALYZER                          │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
          ┌────────────────────────────┐
          │   Run: python cli_analyzer │
          │       .py [OPTIONS]        │
          └────┬───────────────────────┘
               │
        ┌──────┴──────┬────────────┬──────────┐
        │             │            │          │
        ▼             ▼            ▼          ▼
    ┌────────┐   ┌────────┐  ┌────────┐  ┌─────────┐
    │ File   │   │Directory│  │GitHub  │  │Interactive│
    │ (-f)   │   │(-d)    │  │(-g)    │  │(default)  │
    └────┬───┘   └────┬───┘  └────┬───┘  └─────┬─────┘
         │            │           │            │
         └────────────┴───────────┴────────────┘
                      │
                      ▼
        ┌──────────────────────────────┐
        │  Choose Analysis Mode        │
        │  1. Taint (Recommended)      │
        │  2. Regex (Fast)             │
        │  3. Hybrid (Complete)        │
        └──────────────┬───────────────┘
                       │
        ┌──────────────┴──────────────┐
        │                             │
        ▼                             ▼
    ┌─────────────┐           ┌──────────────┐
    │ Run Analysis│           │Specify Mode  │
    │ (Default)   │           │ -m taint     │
    │             │           │ -m regex     │
    └──────┬──────┘           │ -m hybrid    │
           │                  └──────┬───────┘
           └──────────────┬──────────┘
                          │
                          ▼
            ┌─────────────────────────────┐
            │    Execute Analysis         │
            │  ┌─────────────────────┐   │
            │  │ Parse Code Files    │   │
            │  ├─────────────────────┤   │
            │  │ Build AST Tree      │   │
            │  ├─────────────────────┤   │
            │  │ Apply Rules         │   │
            │  ├─────────────────────┤   │
            │  │ Detect Flows/Patterns   │
            │  ├─────────────────────┤   │
            │  │ Calculate Score     │   │
            │  └─────────────────────┘   │
            └────────────┬────────────────┘
                         │
                         ▼
        ┌──────────────────────────────┐
        │  View Terminal Results       │
        │  ├─ Severity Breakdown       │
        │  ├─ Category Statistics      │
        │  ├─ Vulnerability List      │
        │  ├─ Security Score (0-100)  │
        │  └─ Recommendations          │
        └────────────┬─────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
    ┌─────────────┐       ┌───────────────┐
    │  Export?    │───N───│  END (View    │
    │ (Y/N/JSON)  │       │  Results Only)│
    └──────┬──────┘       └───────────────┘
           │
      ┌────┴──────────────┬─────────────┬──────────┐
      │                   │             │          │
      ▼                   ▼             ▼          ▼
  ┌────────┐         ┌────────┐   ┌────────┐  ┌──────────┐
  │ JSON   │    ┌───►│ HTML   │   │ PDF    │  │ TXT      │
  │(-o)    │    │    │(-o)    │   │(-o)    │  │(-o)      │
  └────────┘    │    └────────┘   └────────┘  └──────────┘
                │
        Or use --format json
                │
            ┌───┴─────────────────────────┐
            │                             │
            ▼                             ▼
        ┌─────────────┐         ┌──────────────┐
        │ Save to     │         │ Output to    │
        │ File        │         │ STDOUT (API) │
        └─────────────┘         └──────────────┘
            │                             │
            └─────────────┬───────────────┘
                         │
                         ▼
            ┌─────────────────────────┐
            │  Report Generated       │
            │  ✓ Successfully saved   │
            └──────────────┬──────────┘
                           │
                           ▼
            ┌─────────────────────────┐
            │  END - Ready for Review │
            │  Next: Export it or     │
            │  Share findings         │
            └─────────────────────────┘
```

---

### 2. Interactive Mode Flow

```
┌────────────────────────────┐
│ python cli_analyzer.py     │
│ (No Parameters)            │
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────────────┐
│  DISPLAY BANNER                    │
│  Secure Code Analyzer              │
│  OWASP Top 10                      │
│  Mode: [Current]                   │
└────────────┬───────────────────────┘
             │
             ▼
┌────────────────────────────────────┐
│  SELECT ANALYSIS MODE              │
│  1. Taint Analysis ✓ Recommended   │
│  2. Regex Patterns                 │
│  3. Hybrid Mode                    │
│                                    │
│  Enter choice (1-3, default=1):    │
└────────────┬───────────────────────┘
             │
   ┌─────────┴──────────┬──────────┐
   │                    │          │
   ▼                    ▼          ▼
 [1]                  [2]         [3]
Taint               Regex         Hybrid
   │                 │             │
   └─────────────────┴─────────────┘
             │
             ▼
┌─────────────────────────────────────┐
│  SELECT ANALYSIS ACTION             │
│  1. Analyze single file             │
│  2. Analyze directory               │
│  3. Analyze GitHub repository       │
│  4. Exit                            │
│                                     │
│  Enter choice (1-4):                │
└────────────┬────────────────────────┘
             │
   ┌─────────┼──────────┬──────────┐
   │         │          │          │
   ▼         ▼          ▼          ▼
 [1]       [2]        [3]        [4]
File     Directory   GitHub      Exit
   │         │          │          │
   │         │          │      (Program
   │         │          │       Ends)
   └─────────┴──────────┘
             │
             ▼
┌────────────────────────────────┐
│  ENTER SOURCE PATH             │
│  File: path/to/code.js         │
│  Dir : ./src                   │
│  URL : github.com/user/repo    │
└────────────┬───────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│  ⏳ ANALYZING...                │
│  ├─ Parsing code                │
│  ├─ Building AST                │
│  ├─ Applying rules              │
│  └─ Calculating scores          │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│  📊 DISPLAY RESULTS              │
│  ├─ Summary Panel               │
│  ├─ Severity Table              │
│  ├─ Category Breakdown          │
│  ├─ List of Vulnerabilities    │
│  └─ Security Score              │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│  EXPORT REPORT?                  │
│  json / html / txt / n           │
│                                  │
│  Enter format (or n):            │
└────────────┬─────────────────────┘
         ┌───┼───────┬─────────┐
         │   │       │         │
         ▼   ▼       ▼         ▼
        [J] [H]     [T]       [N]
       JSON HTML   TXT       None
         │   │       │         │
         └───┴───────┘         │
             │                 │
             ▼                 ▼
    ┌──────────────────┐  ┌──────────┐
    │ Enter output     │  │ END      │
    │ file path:       │  │ Back to  │
    │ report.json      │  │ Menu or  │
    └────────┬─────────┘  │ Exit     │
             │            └──────────┘
             ▼
    ┌──────────────────┐
    │ ✓ Report Saved   │
    │ report.json      │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ Back to Main     │
    │ Menu or Exit     │
    └──────────────────┘
```

---

### 3. Mode Selection Decision Tree

```
┌──────────────────────────────────────────┐
│  Choose Analysis Mode                    │
└────────┬───────────────────────────────┬─┘
         │                               │
         ▼                               ▼
  ┌──────────────────┐         ┌─────────────────────┐
  │ Need Fast Scan?  │         │ Production Audit?   │
  │ Yes/No           │         │ Yes/No              │
  └────┬─────────────┘         └────┬────────────────┘
       │                             │
    Yes│                             │Yes
       ▼                             ▼
  ┌──────────────┐             ┌──────────────┐
  │ REGEX Mode   │             │ TAINT Mode   │
  │ (-m regex)   │             │ (-m taint)   │
  │              │             │              │
  │ Fast Results │             │ Deep Analysis│
  │ Simple Scans │             │ Accurate     │
  └──────────────┘             └──────────────┘
       │                             │
    No│                             │No
       └──── ┬──────────────────────┘
            │
            ▼
    ┌───────────────────────┐
    │ Need All Detections?  │
    │ Yes/No                │
    └────┬──────────┬───────┘
         │          │
      Yes│          │No
         ▼          ▼
   ┌──────────┐  ┌──────────┐
   │HYBRID    │  │ Choose   │
   │(-m hybrid)   │ Taint or │
   │          │  │ Regex    │
   │ Complete │  │ based on │
   │ Coverage │  │ needs    │
   └──────────┘  └──────────┘
```

---

### 4. Report Generation Flow

```
┌────────────────────────────────────┐
│  Analysis Complete                 │
│  Vulnerabilities: [N]              │
└────────────────┬───────────────────┘
                 │
         ┌───────┴────────┐
         │                │
         ▼                ▼
    Analysis Summary   Export?
    Ready
         │                │
         └────────┬───────┘
                  │
                  ▼
      ┌───────────────────────┐
      │ Export Format?        │
      │ 1. JSON               │
      │ 2. HTML               │
      │ 3. PDF                │
      │ 4. TXT                │
      │ 5. None (Terminal)    │
      └───┬───┬───┬───┬───┬──┘
          │   │   │   │   │
      ┌───┘   │   │   │   │
      │       │   │   │   │
      ▼       ▼   ▼   ▼   ▼
    JSON    HTML PDF TXT None
      │       │   │   │   │
      │       │   │   │   └─► Display Terminal
      │       │   │   │       Results Only
      │       │   │   │
      └───────┴───┴───┘
              │
              ▼
    ┌─────────────────────────┐
    │ Enter Output File Path  │
    │ Ex: report.format       │
    └────────┬────────────────┘
             │
             ▼
    ┌─────────────────────────┐
    │ Generate Report         │
    │ ├─ Format Data          │
    │ ├─ Apply Styling        │
    │ ├─ Calculate Stats      │
    │ └─ Save to File         │
    └────────┬────────────────┘
             │
             ▼
    ┌─────────────────────────┐
    │ ✓ Report Saved          │
    │ report.json (or format) │
    │ Size: XX KB             │
    │ Location: /path/to/file │
    └─────────────────────────┘
```

---

## Examples

### Example 1: Basic File Analysis

```bash
python cli_analyzer.py -f vulnerable.js
```

**What happens:**

1. Uses default Taint mode
2. Analyzes JavaScript file
3. Displays results in terminal with formatted output
4. Shows severity breakdown and vulnerabilities

**Output:**

```
╔══════════════════════════════════════════════════════════════╗
║         🔒 Secure Code Analyzer - OWASP Top 10              ║
║         Mode: 🔍 Taint Analysis (Data Flow Tracking)        ║
╚══════════════════════════════════════════════════════════════╝

┌─ Summary ─────────────────────────┐
│ Source: vulnerable.js            │
│ Total Issues: 5                  │
│ Security Score: 45/100           │
├─ Issues by Severity ─────────────┤
│ Critical: 2    High: 1    Medium: 2    Low: 0
```

---

### Example 2: Directory Scan with Report Export

```bash
python cli_analyzer.py -d ./src -m hybrid -o security_audit.html
```

**What happens:**

1. Scans entire `./src` directory recursively
2. Uses **Hybrid mode** (most comprehensive)
3. Analyzes all JS/TS/PHP files
4. Generates formatted HTML report
5. Saves to `security_audit.html`

---

### Example 3: GitHub Repository Analysis

```bash
python cli_analyzer.py -g https://github.com/example/vulnerable-app -m taint -o github_report.json
```

**What happens:**

1. Clones GitHub repository
2. Analyzes all code files in repo
3. Uses **Taint mode** (recommended for production)
4. Exports results as JSON
5. Reports cloned repository location
6. Cleans up temporary files

---

### Example 4: CI/CD Integration (JSON Output)

```bash
python cli_analyzer.py -f app.js --format json > results.json
```

**What happens:**

1. Analyzes `app.js`
2. Outputs only raw JSON (no terminal formatting)
3. Pipes to file for processing
4. Suitable for automated pipelines

**Usage in CI/CD:**

```yaml
# GitHub Actions Example
- name: Security Scan
  run: python cli_analyzer.py -f app.js --format json > vuln-report.json

- name: Check Results
  run: |
    CRITICAL=$(jq '.critical_count' vuln-report.json)
    if [ $CRITICAL -gt 0 ]; then exit 1; fi
```

---

### Example 5: Quick Regex Scan (Fast)

```bash
python cli_analyzer.py -f code.php -m regex -o quick_scan.txt
```

**What happens:**

1. Fast pattern-based scan using Regex mode
2. Outputs to text file
3. Suitable for quick checks
4. Lower resource usage

---

### Example 6: Interactive Complete Audit

```bash
python cli_analyzer.py
```

**User Flow:**

```
Select Analysis Mode:
1. 🔍 Taint Analysis (Data Flow Tracking) - Recommended    ← User enters: 1
2. 📋 Regex Patterns (Traditional Matching)
3. ⚡ Hybrid Mode (Both Taint + Regex)

Select Analysis Action:
1. Analyze single file
2. Analyze directory                                        ← User enters: 2
3. Analyze GitHub repository
4. Exit

Enter directory path to analyze:
./myapp/src                                                 ← User enters path

[Analysis runs...]

Export report? (json/html/txt/n):
html                                                        ← User enters: html

Enter output file path:
./reports/audit_2024.html                                   ← User enters: path

✓ Report saved to: ./reports/audit_2024.html
```

---

## Model Selection Guide

### When to Use Each Mode:

#### 🟢 Use TAINT Mode When:

- ✔️ Analyzing production code
- ✔️ Need accurate results
- ✔️ Complex application logic
- ✔️ Security compliance required
- ✔️ Tracking data flow is important

```bash
python cli_analyzer.py -f app.js -m taint
```

#### 🟡 Use REGEX Mode When:

- ✔️ Quick security scans needed
- ✔️ Simple code patterns
- ✔️ Fast feedback in development
- ✔️ Limited resources available
- ✔️ Known vulnerability patterns

```bash
python cli_analyzer.py -f app.js -m regex
```

#### 🔵 Use HYBRID Mode When:

- ✔️ Complete coverage needed
- ✔️ Security audit for clients
- ✔️ Compliance documentation
- ✔️ Finding all possible issues
- ✔️ Time not critical

```bash
python cli_analyzer.py -f app.js -m hybrid
```

---

## Performance Considerations

### Execution Time (Approximate)

| Mode   | Small File (< 1KB) | Medium File (10KB) | Large Directory |
| ------ | ------------------ | ------------------ | --------------- |
| Regex  | < 1s               | 1-2s               | 5-10s           |
| Taint  | 2-3s               | 5-10s              | 30-60s          |
| Hybrid | 3-5s               | 10-15s             | 60-120s         |

### Resource Usage

| Mode   | CPU    | Memory | Disk    |
| ------ | ------ | ------ | ------- |
| Regex  | Low    | ~50MB  | Minimal |
| Taint  | Medium | ~150MB | Minimal |
| Hybrid | High   | ~250MB | Minimal |

---

## Troubleshooting

### Issue: "File not found"

```bash
# Make sure the file path is correct (absolute or relative)
python cli_analyzer.py -f ./test_samples/vulnerable.js

# Or use absolute path
python cli_analyzer.py -f C:\projects\cts\test_samples\vulnerable.js
```

### Issue: Output Path Handling

The CLI now intelligently handles output paths:

**Directory Path Provided:**

```
Export report? (json/html/txt/n): html
Enter output file path: C:\projects\cts\test_samples
ℹ️ Path is a directory. Generating filename...
File will be saved as: security_report_20240214_143022.html
✓ Report saved to: C:\projects\cts\test_samples\security_report_20240214_143022.html
```

**File Path with Wrong Extension:**

```
Export report? (json/html/txt/n): html
Enter output file path: C:\projects\cts\report.txt
⚠️  Correcting file extension from '.txt' to '.html'
✓ Report saved to: C:\projects\cts\report.html
```

**Automatic Directory Creation:**

```
Export report? (json/html/txt/n): json
Enter output file path: C:\projects\reports\audit\final_report.json
✓ Report saved to: C:\projects\reports\audit\final_report.json
# Creates missing directories automatically
```

**Recommended Formats:**

- **HTML Report:** `C:\projects\cts\security_audit.html`
- **JSON Report:** `C:\projects\cts\results.json`
- **Text Report:** `C:\projects\cts\findings.txt`

### Issue: No Output in JSON Mode

```bash
# JSON mode suppresses all terminal output
# Check file was created:
ls -la output.json

# Or check with jq:
jq . output.json
```

### Issue: Out of Memory with Large Directory

```bash
# Use faster Regex mode for large scans:
python cli_analyzer.py -d ./large_project -m regex

# Or analyze subdirectories separately:
python cli_analyzer.py -d ./large_project/src -m hybrid
python cli_analyzer.py -d ./large_project/lib -m hybrid
```

---

## Summary Table

| Task                   | Command                                                                 |
| ---------------------- | ----------------------------------------------------------------------- |
| Analyze file (default) | `python cli_analyzer.py -f app.js`                                      |
| Analyze file (taint)   | `python cli_analyzer.py -f app.js -m taint`                             |
| Analyze file (regex)   | `python cli_analyzer.py -f app.js -m regex`                             |
| Analyze file (hybrid)  | `python cli_analyzer.py -f app.js -m hybrid`                            |
| Save as HTML           | `python cli_analyzer.py -f app.js -o report.html`                       |
| Save as JSON           | `python cli_analyzer.py -f app.js -o report.json`                       |
| Save as PDF            | `python cli_analyzer.py -f app.js -o report.pdf`                        |
| Save as TXT            | `python cli_analyzer.py -f app.js -o report.txt`                        |
| Analyze directory      | `python cli_analyzer.py -d ./src -o report.html`                        |
| Analyze GitHub         | `python cli_analyzer.py -g https://github.com/user/repo -o report.html` |
| Interactive mode       | `python cli_analyzer.py`                                                |
| JSON to stdout         | `python cli_analyzer.py -f app.js --format json`                        |

---

## Full Parameter Reference

```bash
python cli_analyzer.py \
  [-f FILE_PATH]                    # File to analyze
  [-d DIRECTORY_PATH]               # Directory to analyze (recursive)
  [-g GITHUB_URL]                   # GitHub repository URL
  [-o OUTPUT_FILE]                  # Output report path
  [-m {taint|regex|hybrid}]         # Analysis mode (default: taint)
  [--format {text|json}]            # Output format (default: text)
```

**All parameters are optional.**

- Without flags: Enters interactive mode
- With `-f`, `-d`, or `-g`: Performs specified analysis
- With `-m`: Changes analysis model
- With `-o`: Saves report to file
- With `--format json`: Outputs machine-readable JSON to stdout

---

## Authentication & Advanced Usage

### GitHub Private Repositories

For private repositories, set GitHub token:

```bash
export GITHUB_TOKEN=your_token_here
python cli_analyzer.py -g https://github.com/private-org/repo
```

### API Integration Example

```python
import subprocess
import json

# Run analysis and get JSON output
result = subprocess.run(
    ['python', 'cli_analyzer.py', '-f', 'app.js', '--format', 'json'],
    capture_output=True,
    text=True
)

vulnerabilities = json.loads(result.stdout)
print(f"Found {len(vulnerabilities)} issues")
```

---

## Next Steps

1. **First Time?** Start with: `python cli_analyzer.py -f test_samples/vulnerable.js`
2. **Production Audit?** Use: `python cli_analyzer.py -d ./app -m taint -o audit.html`
3. **CI/CD Integration?** Use: `python cli_analyzer.py -f app.js --format json`
4. **Need Help?** Run: `python cli_analyzer.py -h`

---

_Last Updated: 2024 | Secure Code Analyzer v1.0_
