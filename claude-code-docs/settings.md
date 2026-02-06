# Claude Code Settings Documentation

## Overview

Claude Code offers comprehensive configuration through a scope-based system with global, project-level, and local settings. Configuration is managed via the `/config` command and several JSON configuration files.

## Configuration Scopes

Claude Code uses a **four-level scope hierarchy** to determine where configurations apply:

| Scope | Location | Who it affects | Shared with team |
|-------|----------|---|---|
| **Managed** | System-level `managed-settings.json` | All users on machine | Yes (IT-deployed) |
| **User** | `~/.claude/` directory | You, across all projects | No |
| **Project** | `.claude/` in repository | All collaborators | Yes (git-committed) |
| **Local** | `.claude/*.local.*` files | You, this repo only | No (gitignored) |

### Scope Precedence (highest to lowest)
1. Managed (can't be overridden)
2. Command line arguments
3. Local project settings
4. Shared project settings
5. User settings (lowest)

## Settings Files

### File Locations

| Feature | User | Project | Local |
|---------|------|---------|-------|
| **Settings** | `~/.claude/settings.json` | `.claude/settings.json` | `.claude/settings.local.json` |
| **Subagents** | `~/.claude/agents/` | `.claude/agents/` | — |
| **MCP servers** | `~/.claude.json` | `.mcp.json` | `~/.claude.json` |
| **Plugins** | `~/.claude/settings.json` | `.claude/settings.json` | `.claude/settings.local.json` |
| **CLAUDE.md** | `~/.claude/CLAUDE.md` | `CLAUDE.md` or `.claude/CLAUDE.md` | `CLAUDE.local.md` |

### Managed Settings Paths
- **macOS**: `/Library/Application Support/ClaudeCode/`
- **Linux/WSL**: `/etc/claude-code/`
- **Windows**: `C:\Program Files\ClaudeCode\`

> **Note**: Managed settings require administrator privileges and are designed for IT deployment.

## settings.json Configuration

### Basic Example

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp"
  },
  "companyAnnouncements": [
    "Welcome to Acme Corp! Review our code guidelines at docs.acme.com"
  ]
}
```

## Available Settings

### Core Settings

| Key | Description | Example |
|-----|-------------|---------|
| `apiKeyHelper` | Script to generate auth value (sent as `X-Api-Key` and `Authorization: Bearer` headers) | `/bin/generate_temp_api_key.sh` |
| `model` | Override default model for Claude Code | `"claude-sonnet-4-5-20250929"` |
| `env` | Environment variables applied to every session | `{"FOO": "bar"}` |
| `forceLoginMethod` | Restrict login: `claudeai` or `console` | `"claudeai"` |
| `forceLoginOrgUUID` | Auto-select organization UUID during login | `"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"` |
| `language` | Claude's preferred response language | `"japanese"` |
| `outputStyle` | Adjust system prompt style | `"Explanatory"` |

### UI & Display Settings

| Key | Description | Default |
|-----|-------------|---------|
| `cleanupPeriodDays` | Delete inactive sessions after N days | `30` |
| `showTurnDuration` | Show turn duration messages | `true` |
| `spinnerTipsEnabled` | Show tips in spinner | `true` |
| `terminalProgressBarEnabled` | Enable terminal progress bar | `true` |
| `prefersReducedMotion` | Reduce UI animations for accessibility | `false` |
| `spinnerVerbs` | Customize action verbs | `{"mode": "append", "verbs": ["Pondering"]}` |

### Advanced Settings

| Key | Description |
|-----|-------------|
| `otelHeadersHelper` | Script for dynamic OpenTelemetry headers |
| `statusLine` | Custom status line context display |
| `fileSuggestion` | Custom script for `@` file autocomplete |
| `respectGitignore` | Respect `.gitignore` in file picker (default: `true`) |
| `alwaysThinkingEnabled` | Enable extended thinking by default |
| `plansDirectory` | Where plan files are stored (default: `~/.claude/plans`) |
| `autoUpdatesChannel` | `"stable"` or `"latest"` (default) |
| `awsAuthRefresh` | Script to refresh AWS credentials |
| `awsCredentialExport` | Script outputting JSON with AWS credentials |

## Permissions Configuration

### Permission Rules Format

Rules follow `Tool` or `Tool(specifier)` format:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ],
    "ask": [
      "Bash(git push *)"
    ]
  }
}
```

### Permission Rules Quick Reference

| Rule | Effect |
|------|--------|
| `Bash` | All bash commands |
| `Bash(npm run *)` | Commands starting with `npm run` |
| `Read(./.env)` | Reading `.env` file |
| `WebFetch(domain:example.com)` | Fetch requests to example.com |
| `Edit(./src/**)` | Edit files in src directory |
| `MCP(tool_name)` | Specific MCP tool usage |

### Permission Settings Keys

| Key | Description | Example |
|-----|-------------|---------|
| `allow` | Rules to allow tool use | `["Bash(git diff *)"]` |
| `ask` | Rules requiring confirmation | `["Bash(git push *)"]` |
| `deny` | Rules to block tool use | `["WebFetch", "Read(./.env)"]` |
| `additionalDirectories` | Extra working directories | `["../docs/"]` |
| `defaultMode` | Default permission mode | `"acceptEdits"` |
| `disableBypassPermissionsMode` | Prevent bypassing permissions | `"disable"` |

## Sandboxing Configuration

Bash commands can be isolated from filesystem and network:

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["docker"],
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"],
      "allowUnixSockets": ["/var/run/docker.sock"],
      "allowLocalBinding": true
    }
  }
}
```

### Sandbox Settings

| Key | Description | Example |
|-----|-------------|---------|
| `enabled` | Enable bash sandboxing | `true` |
| `autoAllowBashIfSandboxed` | Auto-approve sandboxed commands | `true` |
| `excludedCommands` | Commands running outside sandbox | `["git", "docker"]` |
| `allowUnsandboxedCommands` | Allow bypass of sandbox | `true` |
| `network.allowedDomains` | Domains accessible from sandbox | `["github.com", "*.npmjs.org"]` |
| `network.allowUnixSockets` | Accessible Unix socket paths | `["~/.ssh/agent-socket"]` |
| `network.allowLocalBinding` | Allow localhost binding (macOS) | `true` |

## Attribution Settings

Customize git commit and PR attribution:

```json
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: AI <ai@example.com>",
    "pr": ""
  }
}
```

**Default commit attribution**:
```
🤖 Generated with Claude Code

   Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

## File Suggestion Configuration

Custom command for `@` file autocomplete:

```json
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  }
}
```

The command receives JSON with `query` field and outputs newline-separated file paths (max 15).

## Plugin Configuration

### Plugin Settings

```json
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    }
  }
}
```

### Managed Marketplace Restrictions

**For `managed-settings.json` only** - Allowlist plugin marketplaces:

```json
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/approved-plugins" },
    { "source": "npm", "package": "@acme-corp/compliance-plugins" },
    { "source": "url", "url": "https://plugins.example.com/marketplace.json" }
  ]
}
```

**Disable all marketplace additions**:
```json
{
  "strictKnownMarketplaces": []
}
```

### Supported Marketplace Sources

1. **GitHub**: `{ "source": "github", "repo": "owner/repo", "ref": "branch", "path": "subdir" }`
2. **Git**: `{ "source": "git", "url": "https://...", "ref": "branch", "path": "subdir" }`
3. **URL**: `{ "source": "url", "url": "https://...", "headers": {...} }`
4. **NPM**: `{ "source": "npm", "package": "@org/package" }`
5. **File**: `{ "source": "file", "path": "/absolute/path" }`
6. **Directory**: `{ "source": "directory", "path": "/absolute/path" }`
7. **Host Pattern**: `{ "source": "hostPattern", "hostPattern": "^github\\.example\\.com$" }`

## Environment Variables

### API & Authentication

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API key for Claude SDK |
| `ANTHROPIC_AUTH_TOKEN` | Custom Authorization header value |
| `ANTHROPIC_CUSTOM_HEADERS` | Custom headers (Name: Value format) |
| `ANTHROPIC_MODEL` | Override model name |

### Cloud Provider Authentication

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_FOUNDRY_API_KEY` | Microsoft Foundry authentication |
| `ANTHROPIC_FOUNDRY_BASE_URL` | Foundry resource URL |
| `AWS_BEARER_TOKEN_BEDROCK` | Bedrock API key |
| `CLAUDE_CODE_USE_BEDROCK` | Use Bedrock backend |
| `CLAUDE_CODE_USE_FOUNDRY` | Use Microsoft Foundry |
| `CLAUDE_CODE_USE_VERTEX` | Use Google Vertex AI |

### Behavior & Features

| Variable | Purpose | Example |
|----------|---------|---------|
| `CLAUDE_CODE_ENABLE_TELEMETRY` | Enable OpenTelemetry | `1` |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | Disable background tasks | `1` |
| `CLAUDE_CODE_TASK_LIST_ID` | Share task list across sessions | ID string |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max output tokens | `32000` |
| `CLAUDE_CODE_SHELL` | Override shell detection | `bash` |
| `MAX_THINKING_TOKENS` | Extended thinking budget | `10000` |
| `DISABLE_TELEMETRY` | Opt out of Statsig telemetry | `1` |
| `DISABLE_AUTOUPDATER` | Disable auto-updates | `1` |
| `DISABLE_PROMPT_CACHING` | Disable prompt caching | `1` |

### MCP Configuration

| Variable | Purpose |
|----------|---------|
| `MCP_TIMEOUT` | MCP server startup timeout (ms) |
| `MCP_TOOL_TIMEOUT` | MCP tool execution timeout (ms) |
| `MCP_CLIENT_SECRET` | OAuth client secret for MCP servers |
| `MCP_OAUTH_CALLBACK_PORT` | Fixed OAuth redirect port |
| `ENABLE_TOOL_SEARCH` | MCP tool search: `auto`, `true`, `false` |

### Proxy & Network

| Variable | Purpose |
|----------|---------|
| `HTTP_PROXY` | HTTP proxy server |
| `HTTPS_PROXY` | HTTPS proxy server |
| `NO_PROXY` | Domains to bypass proxy |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS` | Proxy performs DNS resolution |

## Hooks Configuration

Configure custom commands at lifecycle events:

```json
{
  "hooks": {
    "before_bash": "~/.claude/hooks/before-bash.sh",
    "after_edit": "~/.claude/hooks/after-edit.sh"
  },
  "disableAllHooks": false,
  "allowManagedHooksOnly": false
}
```

**Managed-only option**: Force only managed and SDK hooks:
```json
{
  "allowManagedHooksOnly": true
}
```

## Subagent Configuration

Configure custom AI subagents:

- **User subagents**: `~/.claude/agents/` (available across projects)
- **Project subagents**: `.claude/agents/` (project-specific, shareable)

Subagents are Markdown files with YAML frontmatter defining prompts and tool permissions.

## Company Announcements

Display rotating announcements to users:

```json
{
  "companyAnnouncements": [
    "Welcome to Acme Corp! Review guidelines at docs.acme.com",
    "Reminder: Code reviews required for all PRs",
    "New security policy in effect"
  ]
}
```

## Key Points

✓ **Settings are hierarchical** - more specific scopes override broader ones
✓ **Auto-backups** - timestamped backups of config files (5 most recent retained)
✓ **JSON schema support** - add `$schema` for IDE autocomplete validation
✓ **Permissions are rule-based** - evaluated in order: deny → ask → allow (first match wins)
✓ **Managed settings are immutable** - enforce organizational policies that can't be overridden
✓ **Local settings are gitignored** - safe for personal overrides
