# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a documentation repository containing crawled Symfony AI documentation stored as markdown files. It serves as a local reference for Symfony AI components, bundles, and guides.

## Structure

```
symfony-ai-docs/
├── index.md                 # Main entry point with overview
├── components/              # Core Symfony AI components
│   ├── platform.md          # Unified AI model abstraction
│   ├── agent.md             # AI agents with tools
│   ├── chat.md              # Message storage
│   ├── store.md             # Vector stores for RAG
│   └── mate.md              # MCP server dev tool
├── bundles/                 # Symfony integration bundles
│   ├── ai-bundle.md         # Main AI Bundle
│   └── mcp-bundle.md        # Model Context Protocol Bundle
└── cookbook/                # Practical guides
    ├── chatbot-with-memory.md
    └── rag-implementation.md
```

## Working with This Repository

- **No build/test commands** - This is a static documentation repository
- **Source**: Documentation was crawled from https://symfony.com/doc/current/ai/
- **License**: Content is under [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)

## When Updating Documentation

If re-crawling from Symfony's official docs:
1. Use WebFetch to retrieve pages from `https://symfony.com/doc/current/ai/`
2. Extract content as clean markdown
3. Preserve code examples and section structure
4. Maintain the existing directory hierarchy
