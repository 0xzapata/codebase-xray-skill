# Codebase X-Ray Skill

## Description
Expert guide for using `llm-tldr` to analyze codebases efficiently. Use this skill when you need to understand large projects, trace execution flows, debug complex logic, or find relevant code without reading thousands of lines of raw text. It reduces token usage by 95% by providing structural summaries.

## Triggers
- "Analyze this codebase"
- "How does function X work?"
- "Find where Y is defined"
- "Trace the callers of Z"
- "Debug why variable A is null"
- "Get context for B"

## Critical Workflow
1.  **Initialize**: Always run `tldr warm .` first to build/update indexes.
2.  **Search/Locate**: Use `tldr semantic "query"` or `tldr structure` to find entry points.
3.  **Analyze**: Use `tldr context` for understanding or `tldr slice` for debugging.

## Tool Capabilities & Usage

### 1. Project Setup (Do this first!)
*   **Command**: `tldr warm .`
*   **Purpose**: Indexes the codebase (AST, Call Graph, etc.) and starts the background daemon.
*   **Note**: Takes ~5-10s for initial run, <1s for updates.

### 2. Exploration (Map the Territory)
*   **File Tree**: `tldr tree .`
    *   *Use when*: You need to see the file layout.
*   **Code Structure**: `tldr structure . --lang python` (or ts, go, etc.)
    *   *Use when*: You need a high-level map of classes and functions in all files.

### 3. Context & Understanding (Read the Signposts)
*   **Smart Context**: `tldr context <function_name> --project . --depth 2`
    *   *Use when*: You want to understand a function *and* its immediate dependencies.
    *   *Why*: Returns signature, docstring, and what it calls/is called by.
*   **File Outline**: `tldr extract <file_path>`
    *   *Use when*: You want to see all functions/classes in a file without reading the body.

### 4. Navigation (Follow the Roads)
*   **Call Graph (Forward)**: `tldr calls .` or check `calls` in `extract` output.
    *   *Use when*: Tracing what a function *does*.
*   **Impact Analysis (Backward)**: `tldr impact <function_name> .`
    *   *Use when*: Refactoring. "Who uses this function?" or "What breaks if I change this?"

### 5. Deep Analysis & Debugging (Inspect the Engine)
*   **Control Flow**: `tldr cfg <file> <function>`
    *   *Use when*: Understanding complex logic (loops, nested ifs).
*   **Data Flow**: `tldr dfg <file> <function>`
    *   *Use when*: Tracing where a variable is defined or used.
*   **Program Slicing**: `tldr slice <file> <function> <line_number>`
    *   *Use when*: "Why is this line crashing?" Returns *only* the code affecting that specific line.

### 6. Search (Find the Needle)
*   **Semantic Search**: `tldr semantic "natural language description" .`
    *   *Use when*: You know *what* it does, but not the function name. (e.g., "verify jwt token").
*   **Text Search**: `tldr search "exact_pattern" .`
    *   *Use when*: Searching for specific variable names or error strings.

## Best Practices for Agents
1.  **Token Economy**: **Never** read a 500+ line file raw if `tldr context` or `tldr extract` can give you the answer.
2.  **Breadth First**: Start with `tldr structure` or `tldr arch` to understand layers before diving into specific files.
3.  **Surgical Debugging**: Don't guess. Use `tldr slice` to isolate the bug's origin.
4.  **Cross-Language**: Remember `tldr` supports 17 languages. Specify `--lang` if auto-detection might be ambiguous.

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
