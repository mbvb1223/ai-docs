# Angular CLI MCP Server
> Source: https://angular.dev/ai/mcp

## Overview

The Angular CLI includes an experimental Model Context Protocol (MCP) server that enables AI assistants in development environments to interact with Angular CLI capabilities, supporting code generation, package management, and more.

## Available Tools

### Standard Tools (Enabled by Default)

| Tool | Purpose | Local-Only | Read-Only |
|------|---------|-----------|-----------|
| `ai_tutor` | Interactive AI-powered Angular tutor for v20+ projects | Yes | Yes |
| `find_examples` | Locates authoritative code examples from official curated database | Yes | Yes |
| `get_best_practices` | Retrieves Angular Best Practices Guide emphasizing modern patterns | Yes | Yes |
| `list_projects` | Lists applications and libraries from angular.json | Yes | Yes |
| `onpush_zoneless_migration` | Provides step-by-step plan for OnPush change detection migration | Yes | Yes |
| `search_documentation` | Searches official Angular documentation at angular.dev | No | Yes |

### Experimental Tools

| Tool | Purpose | Local-Only | Read-Only |
|------|---------|-----------|-----------|
| `build` | Executes one-off non-watched build via ng build | Yes | No |
| `devserver.start` | Asynchronously launches watched development server | Yes | Yes |
| `devserver.stop` | Halts running development server | Yes | Yes |
| `devserver.wait_for_build` | Returns build output logs from running dev server | Yes | Yes |
| `e2e` | Executes end-to-end tests | Yes | Yes |
| `modernize` | Performs code migrations to latest best practices | Yes | No |
| `test` | Runs unit tests | Yes | Yes |

## Getting Started

Execute this command in your terminal:

```bash
ng mcp
```

When run interactively, this displays configuration instructions for your development environment.

## IDE Configuration Examples

### Cursor

Create `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

Global configuration location: `~/.cursor/mcp.json`

### Firebase Studio

Create `.idx/mcp.json`:

```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

### Gemini CLI

Create `.gemini/settings.json`:

```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

### JetBrains IDEs

Navigate to `Settings | Tools | AI Assistant | Model Context Protocol (MCP)`. Add new server and select "As JSON":

```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

Reference: [JetBrains MCP documentation](https://www.jetbrains.com/help/ai-assistant/mcp.html#connect-to-an-mcp-server)

### VS Code

Create `.vscode/mcp.json` (note: uses `servers` property):

```json
{
  "servers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

### Other IDEs

Create appropriate `mcp.json` file based on your IDE documentation:

```json
{
  "mcpServers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp"]
    }
  }
}
```

## Command Options

| Option | Type | Description | Default |
|--------|------|-------------|---------|
| `--read-only` | boolean | Registers only non-modifying tools | false |
| `--local-only` | boolean | Registers only offline-capable tools | false |
| `--experimental-tool` / `-E` | string | Enables specific experimental tools (space-separated) | -- |

### Read-Only Mode Example (VS Code)

```json
{
  "servers": {
    "angular-cli": {
      "command": "npx",
      "args": ["-y", "@angular/cli", "mcp", "--read-only"]
    }
  }
}
```

Enable all devserver tools: `-E devserver`

Enable multiple tools: `-E tool_a tool_b`

## Feedback and Contributions

Share feedback and feature requests via the [angular/angular GitHub repository](https://github.com/angular/angular/issues).
