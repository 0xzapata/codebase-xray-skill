---
name: codebase-xray
description: Use when analyzing large codebases, debugging complex execution flows, finding definitions/usages, or needing to understand code structure without reading raw files.
---

# Codebase X-Ray (llm-tldr)

## Overview
Uses the `llm-tldr` tool to analyze code structure, dependencies, and flow. This reduces token usage by 95% compared to reading raw files by providing structural summaries (AST, call graphs, CFG, DFG) instead of full text.

## When to Use
*   **Analyze codebase**: "Understand this project" or "Map the file structure"
*   **Trace execution**: "Who calls function X?" or "What does function Y call?"
*   **Debug logic**: "Why is this variable null?" or "Trace the data flow"
*   **Find code**: "Where is the auth logic?" (semantic search)
*   **Refactor**: "What breaks if I change this function?" (impact analysis)

## Prerequisites
**Tool Installation**
If `tldr` command is not found, install it via pip:
```bash
pip install llm-tldr
# OR
pip3 install llm-tldr

# OR Install from Source (Development)
git clone https://github.com/parcadei/llm-tldr.git
cd llm-tldr
pip install -e .
```

## Quick Reference

| Goal | Command |
|------|---------|
| **Initialize (REQUIRED)** | `tldr warm .` |
| **Map Structure** | `tldr structure . --lang [python|ts|...]` |
| **Understand Function** | `tldr context <name> --project . --depth 2` |
| **Trace Callers** | `tldr impact <name> .` |
| **Debug Logic** | `tldr slice <file> <func> <line>` |
| **Semantic Search** | `tldr semantic "description of logic" .` |

## Core Workflow

### 1. Initialize (Mandatory)
Always start by warming the cache to build indexes and start the daemon.
```bash
tldr warm .
```

### 2. Locate Entry Points
Find where to look without reading files.
```bash
# If you know the concept (e.g., "authentication")
tldr semantic "verify user token" .

# If you want a map of the territory
tldr structure . --lang python
```

### 3. Analyze Context
Read the map, not the terrain. Get the function + its dependencies.
```bash
tldr context process_payment --project . --depth 2
```
*Returns signature, docstring, and immediate callers/callees.*

### 4. Deep Dive (Only if needed)
Inspect specific logic paths or data flows.
```bash
# "Why is this line executed?" (Control Flow)
tldr cfg src/payment.py process_payment

# "Where did this variable come from?" (Data Flow)
tldr dfg src/payment.py process_payment

# "What code affects line 50?" (Program Slicing)
tldr slice src/payment.py process_payment 50
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Reading raw files (cat/read) for overview | Use `tldr structure` or `tldr extract` to save 90% tokens. |
| Guessing function names | Use `tldr semantic` to find code by behavior. |
| Skipping `tldr warm` | Always run it first to ensure indexes are up-to-date. |
| Reading full file to debug one line | Use `tldr slice` to see only the relevant lines. |

## Supported Languages
Python, TypeScript, JavaScript, Go, Rust, Java, C, C++, Ruby, PHP, Kotlin, Swift, C#, Scala, Lua, Luau, Elixir.

## Advanced Configuration

### Automated Reindexing (Git Hooks)
To keep the index fresh automatically, add `tldr warm .` to your git hooks.
*   **Hook**: `.git/hooks/post-checkout` and `.git/hooks/post-merge`
*   **Command**:
    ```bash
    #!/bin/sh
    tldr warm . > /dev/null 2>&1 &
    ```
*   **Benefit**: Index updates in the background whenever you switch branches or pull code.

### Gitignore Rules
Add these to your `.gitignore` to keep your repo clean:
```text
.tldr/          # Cache and daemon socket
.tldrignore     # Auto-generated ignore rules
```

## Embedding Model Setup
The first time you run `tldr semantic`, it downloads the embedding model (~1.3GB).
*   **Model**: `BAAI/bge-large-en-v1.5` (High quality)
*   **Offline Mode**: Once downloaded, it works fully offline.
*   **Smaller Model**: Use `--model all-MiniLM-L6-v2` (80MB) if bandwidth/storage is tight.
    ```bash
    tldr semantic "query" . --model all-MiniLM-L6-v2
    ```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| **Daemon not responding** | Stale socket file | `rm /tmp/tldr-*.sock` then `tldr warm .` |
| **"Lock file exists"** | Previous crash | `rm /tmp/tldr-*.lock` |
| **Semantic search slow** | First-run model load | Wait for download to complete (up to 2 mins). |
| **Missing files in tree** | .tldrignore | Check `.tldrignore` or use `--no-ignore`. |
