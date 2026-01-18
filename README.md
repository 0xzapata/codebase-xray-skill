# Codebase X-Ray Skill

This repository contains an **AI Agent Skill** designed to help LLMs (like Claude, OpenCode, etc.) effectively use the [`llm-tldr`](https://github.com/parcadei/llm-tldr) tool.

## What is this?

This is a **meta-instruction set** for AI agents. It teaches them how to:
- Navigate large codebases without reading every file.
- Use structural analysis (AST, CFG, DFG) instead of raw text.
- Save 95% of context window tokens during exploration.
- Avoid common pitfalls like hallucinating function names or reading massive files unnecessarily.

## For AI Agents

The core instruction set is located in [SKILL.md](./SKILL.md).

## Usage

If you are using an agentic workflow (like Claude Code, OpenCode, or similar):

1.  **Install the Tool**: Ensure `llm-tldr` is installed in your environment:
    ```bash
    pip install llm-tldr
    ```

2.  **Load the Skill**: Provide the content of `SKILL.md` to your agent as a system prompt, memory, or "skill" definition.

3.  **Trigger**: Ask your agent to "Analyze this codebase" or "Debug this flow", and it will follow the optimized X-Ray workflow.

## License

MIT
